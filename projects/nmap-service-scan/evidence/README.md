# Evidence — Nmap service scan

Scan output first, in the order it was produced:

- `01-full-tcp-scan.txt` — `nmap -sS -p- -n --min-rate 1000 …`
- `02-version-detection.txt` — the `-sV` run on the 26 fixed ports
- `03-version-detection-pn.txt` — the same scan with `-Pn`, as a cross-check
- `04-nse-default.txt` — `nmap -sC -sV …`
- `05-rpcinfo.txt` — `rpcinfo -p 192.168.62.130`
- `06-nse-vuln.txt` — `nmap --script vuln`; the file was produced as `05-nse-vuln.txt` and renamed to keep the order

`screenshots/` holds terminal captures of the same session, including the stalled scan that started the diagnosis:

- `01-stalled-scan.png` — the run that appeared hung
- `02-no-discovery-no-dns.png` — `-Pn -n`
- `03-dns-disabled.png` — `-n` only
- `04-full-tcp-scan.png`, `05-version-detection.png`, `06-default-scripts-1..3.png`

Before committing anything:

- Paste terminal output as plain text where you can; it is searchable and diff-friendly.
- Crop or blur hostnames, usernames, real addresses and anything that identifies your home network.
- Keep the exact command in the first line of each output file, so the evidence is reproducible.
- Do not commit full packet captures or binaries — describe them instead.
