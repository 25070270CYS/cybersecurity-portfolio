---
layout: default
title: "Cybersecurity portfolio"
---
# Yan Shen — cybersecurity portfolio

This site is my working record for **WQE7001 Cyber Security**: the lab I built, the evidence that came out of it, and an honest account of what I can and cannot do yet. Everything here was done on my own machines or on a legal practice platform.

## About me

I am Yan Shen, a postgraduate student in cyber security at Universiti Malaya. I came to the field through an interest in how systems fail: most of the incidents I read about were not sophisticated attacks but ordinary mistakes — an unpatched service, a reused password, a log nobody read. That is the kind of problem I want to work on. The course gave me a lab of my own, and this site is where I keep the evidence: a virtual network I can break safely, and the scans and captures I used to understand it. I am early in my career, and I would rather show what I have actually done than list tools I have only read about.

## Technical skills

| Skill | Level | Where I have used it |
| --- | --- | --- |
| Linux command line, file permissions and log locations | working knowledge | [Virtual lab setup](projects/virtual-lab-setup/) |
| Windows administration: processes, registry, accounts | working knowledge | Course labs (PowerShell, Task Manager, registry) |
| Virtualisation and isolated lab networks (VMware) | working knowledge | [Virtual lab setup](projects/virtual-lab-setup/) |
| Network scanning and service enumeration (Nmap) | working knowledge | [Nmap service scan](projects/nmap-service-scan/) |
| Packet analysis with Wireshark (TCP/UDP, DNS, HTTP) | working knowledge | Course labs |
| Firewall ACLs and IDS rules (Packet Tracer, Snort) | beginner | Course labs |
| Hashing, encryption and certificates (OpenSSL) | beginner | Course labs |
| Incident response and log analysis | beginner | Course labs |
| Git, GitHub and publishing a static site | working knowledge | [This repository](https://github.com/25070270CYS/cybersecurity-portfolio) |

## Certifications and training

| Training | Provider | Status | Evidence |
| --- | --- | --- | --- |
| Cyber Security module — WQE7001 | Universiti Malaya | In progress, expected November 2026 | The two projects on this site |

## Projects

### 1. Virtual lab setup — [full write-up](projects/virtual-lab-setup/)

A VMware host running Kali Linux and Metasploitable 2 on a host-only network, so that attacks I practise cannot reach the internet or my home network. It shows virtualisation, network isolation, and the discipline of defining scope before touching a target.

### 2. Nmap service scan — [full write-up](projects/nmap-service-scan/)

A full 65,535-port scan and version detection against the lab target: 30 open ports, none of them filtered, and seven that a default scan never shows. The write-up explains what each service gives an attacker and how it should be closed or monitored. It shows a repeatable method — and the habit of checking a result rather than pasting it.

## Learning journey

I started this course able to follow instructions and not much else. What has changed is that I can usually tell when a result is wrong, and I now have a method for finding out why.

### Two labs, and what I took from each

The Wireshark labs (9.2.6 and 10.2.7) changed how I read a capture. Following a TCP handshake and then the same conversation in DNS made the layers stop being a diagram: I could point at the sequence number that shows a retransmission, or at the query that explains a delay the browser would have blamed on the server. The Nmap lab (9.3.8) was the other one. Running a scan is a five-minute exercise; deciding what the output means took me far longer, and that is the part I kept. A port is not a finding. A product and a version is a finding, and only then can you say what it exposes.

### What went wrong, and how I fixed it

My first `nmap -sV` against the lab target hung. The statistics line sat at `0 hosts completed`, `1 undergoing ARP Ping Scan`, and a minute later it had not moved. I assumed the GNS3 link was broken, which sent me looking in the wrong place. Before changing anything else I pinged the target from Kali, and Kali from the target: 4/4 replies each way, well under a millisecond. The network was fine, so the fault had to be in what nmap was doing besides scanning — reverse DNS, or ARP discovery. Adding `-n` removed DNS and the same scan finished in 11.36 seconds.

That was not the end of it. The full 65,535-port scan then found seven ports the default scan had missed, and the vulnerability scripts used one of them, distccd, to run a command on the target. Only at the end did the whole picture appear: the guest has no default route, so no resolver could ever have answered. The lesson was not "use `-n`". It was that I should have proved the network worked before touching the scanner, and that a tool timing out is a claim about the tool until you check.

### What I still find difficult

Reading scanner output critically. Of the vulnerability checks, five scripts failed, two timed out, and two were throttled by the service they were probing. My first instinct was to read the absence of a `VULNERABLE` line as a clean result, which is exactly backwards: a check that did not run has told me nothing. I am also slow at prioritising. I can describe all thirty open ports, but choosing which three a defender should fix first still takes longer than it should.

### My next goal

By 30 June 2027 I want three incident write-ups on this site, each reconstructing a timeline from logs rather than from a scanner, because that is what the work I am aiming at actually looks like. I will measure it by counting published write-ups — three, each with a dated timeline and at least one conclusion I could not have reached from a tool's default output.

## Career objective

My goal is a first role as a **Security Operations Engineer**, on the detection side of the work: triaging alerts, reading logs, and knowing what is normal for a network before deciding what is hostile. This portfolio is the evidence that I have started; the steps from here are concrete.

1. **Finish this course with evidence rather than claims.** By November 2026 the site should hold two projects a reviewer can check in ten minutes: a lab I built and isolated, and a scan whose every number I can explain.
2. **Build detection skill deliberately.** By June 2027, three incident write-ups reconstructed from logs — timelines, not tool output — and one entry-level certification to put the theory on paper.
3. **Get into a queue I can learn from.** From mid-2027, apply for graduate and entry-level security operations roles, and treat every interview as a gap list: whatever I cannot explain, I study next.

## Contact

- **Email:** yanshen87698@gmail.com
- **GitHub:** <https://github.com/25070270CYS>
- **LinkedIn:** <https://www.linkedin.com/in/yanshen25070270>

My LinkedIn profile links back to this site. I do not publish my phone number or home address here.
