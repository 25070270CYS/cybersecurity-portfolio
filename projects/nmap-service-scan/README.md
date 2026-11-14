---
layout: page
title: "Project 2 — Nmap service scan"
permalink: /projects/nmap-service-scan/
---
**Project 2 of 2** · [Back to the portfolio](../../)

## Objective

Establish what the lab target actually exposes, and turn the port list into a judgement: which services should not be running at all, which are old enough to have public exploits, and what a defender should do first. The scan itself takes seconds; the result is the interpretation and the fix list.

## Scope and authorisation

| Question | Answer |
| --- | --- |
| Who authorised the testing? | Me. Both machines are mine and run on my own laptop. |
| What is in scope? | One host: `192.168.62.130`, the Metasploitable 2 guest built in [Project 1](../virtual-lab-setup/). All 65,535 TCP ports. |
| What is out of scope? | Every other address — the host, the home LAN, the university network, the internet. |
| How is that enforced? | The lab is a GNS3 project on my own machine; `Switch1` is an internal virtual switch and the guest VMs are attached through VMware adapters. I never pass an address outside `192.168.62.0/24` to a scanner, and the guest has no route off that segment at all. |

## Environment and tools

| Component | Role | Notes |
| --- | --- | --- |
| Kali Linux 2026.2 (VMware guest) | Scanning host | `192.168.62.129/24` |
| Metasploitable 2 (VMware guest) | Target | `192.168.62.130/24` |
| GNS3 2.2.61 with `Switch1` | Virtual topology | Both guests share one L2 segment |
| Nmap 7.991 | Scanner | Host discovery, TCP SYN scan, version detection, default NSE scripts |

## Method

1. **Confirmed the target was alive with ICMP first** (`ping -c 4 192.168.62.130`), from Kali and from the target back to Kali — both directions are recorded in [Project 1](../virtual-lab-setup/). Both replied with 0% loss and sub-millisecond times, so any later silence from nmap would be nmap's problem, not the network's.
2. **Diagnosed a stall that was not a network problem.** The first runs looked hung inside host discovery; the cause turned out to be the reverse-DNS lookup timing out on a network with no resolver, not the link. `-n` removes DNS from the scan, after which discovery and scanning both finished in about 11 seconds.
3. **Full TCP scan.** SYN scan over all 65,535 ports to catch services outside nmap's default top-1,000.
4. **Version detection** on all 26 fixed ports the full scan found, so each finding becomes a product and version rather than a port number. The four high ports are handled separately: three turn out to be RPC services, which the default scripts later name.
5. **Cross-checked the result.** Ran the same scan again with `-Pn`, which skips discovery entirely (safe, because step 1 had already proved the host was up). On the 23 ports the two runs share, the output is identical, so nothing in the list depends on how the host was found.
6. **Default NSE scripts** with `-sC`, staying with the safe default set rather than intrusive script categories.
7. **Vulnerability scripts** with `--script vuln` over the same 26 ports, capped with `--script-timeout 60s` so one fragile service cannot stall the run. These go beyond detection: two of them confirmed the backdoors by running a command.
8. **Triaged every port** into six groups — unauthenticated, outdated, plaintext, should-be-local, ephemeral RPC, or unexplained — and wrote the fix list from those groups rather than from the port numbers.

```bash
# liveness first, so a scanner problem cannot be mistaken for a network problem
ping -c 4 192.168.62.130

# full TCP scan, no DNS
nmap -sS -p- -n --min-rate 1000 192.168.62.130 -oN 01-full-tcp-scan.txt

# version detection on every open port
nmap -sV -n -p 21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787 192.168.62.130 -oN 02-version-detection.txt

# the same scan with discovery switched off, as a cross-check
nmap -sV -Pn -n -p 21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787 192.168.62.130 -oN 03-version-detection-pn.txt

# safe default scripts
nmap -sC -sV -n -p 21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787 192.168.62.130 -oN 04-nse-default.txt

# map the ephemeral RPC ports back to their programs
rpcinfo -p 192.168.62.130

# vulnerability checks, one minute per script
nmap -sV -n -p 21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787 --script vuln --script-timeout 60s -oN 06-nse-vuln.txt 192.168.62.130
```

### What went wrong: a scan that looked hung

The first `nmap -sV` run printed `Parallel DNS resolution of 1 host`, then sat at `0 hosts completed (0 up), 1 undergoing ARP Ping Scan` long enough to look dead. The capture in `screenshots/01-stalled-scan.png` shows six consecutive statistic lines between 19 and 21 seconds, every one of them `0 hosts completed` — that is not a slow scan, it is a scan waiting for a reply that never arrives. The obvious suspect was the GNS3 link, so that is what I checked first: `ping -c 4` in both directions returned 4/4 replies at well under a millisecond. The network was fine.

That left the two things nmap was doing besides scanning: reverse-DNS lookups, and ARP host discovery. Adding `-n` removed DNS and the same scan finished in 11.36 seconds with the host reported up — so the stall was DNS, on a lab network with no resolver to answer it, and neither host discovery nor the virtual link was at fault. The `-Pn` run then served as an independent confirmation: identical ports, identical versions.

## Results

30 of the 65,535 TCP ports were open. 65,505 were closed and **none were filtered** — every port answered one way or the other, which means no host firewall is running and nothing in the path is filtering. That is a finding in its own right: the only thing between an attacker and these services is whether they know the port is there.

| Port | Service | Version | Why it matters |
| --- | --- | --- | --- |
| 21/tcp | ftp | vsftpd 2.3.4 | This build was modified to include a backdoor (CVE-2011-2523), and the `vuln` script confirms it by returning a root shell. Anonymous login is also enabled. |
| 22/tcp | ssh | OpenSSH 4.7p1 | Legitimate administration, but a 2008-era build and a password-guessing target if exposed more widely. |
| 23/tcp | telnet | Linux telnetd | Sends credentials and session traffic in clear text. |
| 25/tcp | smtp | Postfix smtpd | Old daemon, and it still offers SSLv2 with export-grade ciphers. The certificate it presents expired in April 2010. |
| 53/tcp | domain | ISC BIND 9.4.2 | Old resolver; zone transfer and cache-poisoning classes of attack. |
| 80/tcp | http | Apache httpd 2.2.8 | Serves the deliberately vulnerable web applications, so the application layer is the real attack surface here. |
| 111/tcp | rpcbind | RPC #100000 | Publishes the RPC program list and makes 2049 and the four high ports easy to find. |
| 139/tcp | netbios-ssn | Samba smbd 3.X–4.X | Old Samba with a long CVE history (for example CVE-2007-2447, the usermap script command injection). |
| 445/tcp | netbios-ssn | Samba smbd 3.0.20-Debian | Same daemon over direct SMB, on the usual entry point for lateral movement. Guest access is accepted and message signing is off. |
| 512/tcp | exec | netkit-rsh rexecd | Remote execution from an era with no encryption and no meaningful authentication. |
| 513/tcp | login | rlogind | Plaintext remote login. |
| 514/tcp | shell | rshd | Trust-based remote shell; anybody who can reach it and claim the right source address gets in. |
| 1099/tcp | java-rmi | GNU Classpath grmiregistry | RMI registry exposed to the network; deserialisation on this path is a known remote-code-execution route. |
| 1524/tcp | bindshell | Metasploitable root shell | Not a service at all — a backdoor that hands over a root shell. The port database calls it `ingreslock`, which is exactly why the version scan is needed. |
| 2049/tcp | nfs | NFS 2–4 | The export can be mounted by anyone who can reach it, with no credential check. |
| 2121/tcp | ftp | ProFTPD 1.3.1 | Old daemon; `mod_copy` allows copying files without authentication. The port database calls it `ccproxy-ftp`, the version scan corrects it. |
| 3306/tcp | mysql | MySQL 5.0.51a | Ancient release listening on all interfaces; weak or empty root credentials are common. |
| 3632/tcp | distccd | distccd v1 (GNU 4.2.4) | The distributed-compiler daemon, and an old build of it. It accepts work from any client without authentication (CVE-2004-2687), which means command execution as the `distcc` user. |
| 5432/tcp | postgresql | PostgreSQL 8.3.x | Old release, default credentials, network-exposed, presenting the same certificate that expired in 2010. |
| 5900/tcp | vnc | VNC protocol 3.3 | Weak password scheme and no transport encryption. |
| 6000/tcp | x11 | X11, access denied | The refusal is the only thing protecting an X server that has no real authentication. |
| 6667/tcp | irc | UnrealIRCd | The daemon reports itself as `Unreal3.2.8.1`, the build that shipped with a backdoor (CVE-2010-2075). |
| 6697/tcp | irc | UnrealIRCd | The same daemon as 6667, on the port normally used for TLS. Which listener an attacker reaches makes no difference: the backdoor is in the daemon. |
| 8009/tcp | ajp13 | Apache Jserv 1.3 | Old Tomcat AJP connector; an internal protocol that should never be reachable from a client network. |
| 8180/tcp | http | Tomcat 5.5 (Coyote 1.1) | Tomcat manager on a build from 2004, reachable with default credentials. |
| 8787/tcp | drb | Ruby DRb RMI (Ruby 1.8) | The distributed-Ruby protocol is unauthenticated: any object the service exposes can be called from the network, and the Ruby 1.8 runtime underneath it is years out of support. |
| 36116/tcp | mountd | RPC #100005, ephemeral | The NFS mount daemon, on a port rpcbind handed it at boot. It is what tells a client which exports it may mount. |
| 38103/tcp | unknown | not in rpcinfo | Open at scan time, but a direct `rpcinfo -p` an hour later still does not list it. It is not an RPC program — or an RPC program that never registered. The one row in this table I cannot name. |
| 43215/tcp | nlockmgr | RPC #100021, ephemeral | The NFS lock manager. It exists only because NFS does. |
| 47115/tcp | status | RPC #100024, ephemeral | rpc.statd, the lock-recovery service that goes with NFS. |

### What the default scripts added

Version detection says what software is listening; the default scripts say what it actually does. Six results changed the picture:

- **`ftp-anon` on 21.** Anonymous access is no longer a configuration guess: the script logged in, received code 230 and read the server status. The banner still advertises the daemon as "secure, fast, stable".
- **`sslv2` on 25.** The mail server offers SSLv2 with RC2, RC4 and DES export ciphers, and presents a certificate that expired in April 2010. Encrypted is not the same as safe.
- **`rpcinfo` on 111.** This is the map for the high ports: `mountd` on 36116, `nlockmgr` on 43215, `status` on 47115. It does not explain 38103, and a direct `rpcinfo -p` an hour later confirmed the same three ports and the same absence — which is itself the finding: not every high port is an RPC side door.
- **`smb-security-mode` on 445.** Samba 3.0.20-Debian, guest account accepted, message signing disabled. Signing off is the default that makes SMB relay attacks work.
- **`irc-info` on 6667.** The daemon reports itself as `Unreal3.2.8.1` — the backdoored build — and the output records the scanner's own address as the source of that connection.
- **`vnc-info` on 5900.** Protocol 3.3 with "VNC Authentication (2)", the fixed-key challenge-response scheme that makes VNC passwords worth attacking.

### What the vulnerability scripts confirmed

The `vuln` scripts go further than fingerprinting: two of them proved their finding by actually running a command on the target.

- **21 — vsftpd 2.3.4 backdoor.** `ftp-vsftpd-backdoor` reports `VULNERABLE (Exploitable)`, CVE-2011-2523, and the `id` it ran through the backdoor returned `uid=0(root) gid=0(root)`. This is no longer a claim about a version number.
- **3632 — distccd.** `distcc-cve2004-2687` reports `VULNERABLE (Exploitable)`, CVSS 9.3, and its `id` returned `uid=1(daemon)`. Unauthenticated command execution, exactly as the version suggested.
- **1099 — RMI registry.** `rmi-vuln-classloader` reports the default configuration allows loading classes from remote URLs, which is remote code execution by another name.
- **6667 / 6697 — UnrealIRCd.** `irc-unrealircd-backdoor` reports "Looks like trojaned version of unrealircd" — an independent confirmation of what the banner already claimed.
- **25 and 5432 — TLS.** POODLE (CVE-2014-3566) on both, CCS injection (CVE-2014-0224) on PostgreSQL, and anonymous Diffie-Hellman with 512-bit export groups (Logjam, CVE-2015-4000) on the mail server.
- **80 and 8180 — content.** `http-enum` found `/phpMyAdmin/`, `/phpinfo.php`, `/tikiwiki/` and a browsable `/doc/` directory on 80, with TRACE enabled; on 8180 the Tomcat manager answers 401 and the scanner lists a long set of `/admin/` paths, with `JSESSIONID` set without `HttpOnly`.

**Authorisation.** Every one of these checks ran against my own guest, on a host-only segment. The same command against a host you do not own is not a scan, it is an offence.

**What the output does not say.** Five checks failed rather than passed — `sslv2-drown` on 25, `http-vuln-cve2017-1001000` and `http-sql-injection` on 80, `http-sql-injection` on 8180 and `smb-vuln-regsvc-dos` on 445 — two more timed out (`ssl-ccs-injection` on 3306 and 6697), and the IRC checks never got a verdict because the daemon throttled the second connection. A failed script is an untested hypothesis, not a clean bill of health, and none of them can be read as "not vulnerable". The 27 `Error resolving vulners.com` lines are the same missing resolver that started this project: scripts that consult an external CVE database cannot work in a lab with no DNS. And `smtp-vuln-cve2010-4344` returns `NOT VULNERABLE` because the server is Postfix while the CVE is about Exim — a script named after a CVE does not mean that CVE applies to this host.

**What the default scan would have missed.** The first, top-1,000 scan found 23 ports; the full scan found 30. The seven extra are `3632` (distccd), `6697` (the same UnrealIRCd as 6667), `8787` (Ruby DRb), the three NFS-related RPC ports and the unexplained `38103`. Two of them are unauthenticated remote-execution services — distccd accepts work from any client, and DRb exposes Ruby objects with no credential check — so the default scan alone would have left the two most directly exploitable findings off the list. The lesson is not that the default scan is wrong, but that it is a shortcut.

**Triage.** The 30 ports fall into six groups, which is how the fix list is ordered:

| Group | Ports | What it means |
| --- | --- | --- |
| Unauthenticated by design (or a backdoor) | 21, 512, 513, 514, 1099, 1524, 2049, 3632, 6000, 6667, 6697, 8787 | No credential is needed to get somewhere useful — these are the first things to remove. |
| Outdated software with public exploits | 21, 23, 25, 53, 80, 139, 445, 2121, 3306, 3632, 5432, 5900, 8180, 8787 | The version string alone is a finding; patching or replacing is the fix. |
| Plaintext protocols | 21, 23, 512, 513, 514, 6667 | Anything captured on the wire is readable, including credentials. |
| Should listen on loopback only | 1099, 3306, 5432, 5900, 6000, 8009, 8787 | Administrative interfaces that have no reason to be reachable from another host. |
| Ephemeral RPC side doors | 36116, 43215, 47115 | Not fixed services: rpcbind hands out a new port at every boot, so a port list is unstable evidence until `rpcinfo` names the programs. All three disappear when NFS does. |
| Unexplained | 38103 | Open at scan time but missing from the portmapper table on two separate runs, so it is not an RPC service. It needs its own probe before it can be classified. |

**Two things worth reading carefully in the output.** The latency figure is the round trip of whatever probe found the host, not a property of the network: `0.000060s` appears under `-Pn`, where no discovery probe was sent, while the ARP-based runs report `0.000044s` and `0.00065s`, and `ping` reports 0.15–0.43 ms. Three different probes, three different numbers — quoting any one of them as "the latency" would be wrong. And the service names in the full scan come from a port-number database, not from the target: `1524` is labelled `ingreslock` and `2121` `ccproxy-ftp`, while version detection shows a root-shell backdoor and ProFTPD; `6697` is `ircs-u` until the scan reports the same UnrealIRCd as 6667. Port names are guesses; versions are evidence.

## Evidence

| File | What it shows |
| --- | --- |
| `evidence/01-full-tcp-scan.txt` | Every open TCP port out of 65,535, and that none are filtered |
| `evidence/02-version-detection.txt` | Product and version behind all 26 fixed ports; discovery on, DNS off |
| `evidence/03-version-detection-pn.txt` | The same scan with `-Pn`, as a cross-check on the port list |
| `evidence/04-nse-default.txt` | Anonymous FTP, SSLv2 on SMTP, the rpcbind table, SMB signing, IRC and VNC details |
| `evidence/05-rpcinfo.txt` | The portmapper table on its own, including the port that is not in it |
| `evidence/06-nse-vuln.txt` | Which findings are confirmed, which scripts failed, and what they proved |
| `screenshots/` | Terminal captures of each step, from the stalled scan (`01-stalled-scan.png`) to the script output |

## What I would do differently

- **Start from `-p-`.** The default scan stopped at 1,000 ports and missed seven, one of which is unauthenticated remote execution. A top-1,000 scan is a reconnaissance shortcut, not a baseline.
- **Version-scan what the full scan finds.** My first version scan covered only the default port list, which left distccd, DRb and the second IRC listener unfingerprinted; DRb only became a finding once it had a version.
- **Run the vulnerability scripts after fingerprinting, not instead of it.** They took 80 seconds against 11 for the version scan, several of them errored out, and the output is only readable because the version scan had already said what each service is.
- **Scan UDP too, at least the top ports.** The rpcbind table lists a UDP counterpart for every RPC service — `48624/udp` mountd, `41568/udp` nlockmgr, `33739/udp` status — so a TCP-only scan understates the target in exactly the way the default scan did.
- **Identify every port before publishing.** `rpcinfo` named three of the four high ports and ruled out the fourth as an RPC service; `38103` is still only a number, and `ss -lntp` on the target is the one command that would say which process owns it.
- **Save every run with `-oA`, not just `-oN`.** The XML output would have let me compare two scans programmatically instead of by eye.
- **Keep a command log as I go.** The version-detection port list above was reconstructed from the scan output; a log would have removed that step.
- **Reach for `-n` before retrying anything.** The stall looked like a network fault and was a DNS timeout; one flag separated the two, but only after `ping` had eliminated the link. Re-running the same command, which was my first instinct, taught me nothing.

## How I would fix it

- **Remove what is not needed.** 1524 and 6667 are backdoors — they are not configured away, the machine is rebuilt. 512–514, 23, 21, 3632 and 6000 should not exist on any host in 2026. The FTP daemon on 21 is the clearest case: a version check would have called it outdated, and the vulnerability script walked away with a root shell.
- **Turn off RPC and NFS together.** Disabling rpcbind and the NFS server removes 111, 2049, 36116, 43215 and 47115 — five findings in one change. `38103` has to be identified before it can be claimed by the same fix.
- **Bind administration to loopback.** MySQL, PostgreSQL, VNC, RMI, DRb and the AJP connector should listen on `127.0.0.1`, not on the network interface.
- **Patch or replace the rest.** vsftpd, Samba, Apache, Tomcat, BIND, Postfix, OpenSSH and both databases are years out of date; the version strings above are the evidence.
- **Filter at the boundary.** A host firewall allowing only the ports the lab needs — plus an ACL on `R1` once it is in the path — would have turned 65,505 closed ports and 30 open ones into a much shorter list of reachable ports. Today nothing is filtered at all.
- **Watch for change.** Alerting on new listening ports and on scan-like traffic (many SYNs, few completed handshakes) catches both a service that appears after the build and the next person running this scan.
- **Re-scan and compare.** The only proof the fixes worked is a second scan showing fewer ports, and the count is the metric to quote.
