# Evidence — virtual lab setup

Five screenshots, matching the table in the write-up:

- `01-isolation-check.png` — `ping 8.8.8.8` and `ip route`, showing no default gateway
- `02-gns3-topology.png` — the GNS3 topology, with both guests, `Switch1` and `R1`
- `03-kali-address-and-ping.png` — Kali's `ifconfig` and the ping to the target
- `04-target-address.png` — the target's `ifconfig`
- `05-target-to-kali-ping.png` — the ping back from the target to Kali

Before committing anything:

- Crop or blur hostnames, usernames, your home network's real addresses, MAC addresses, email addresses, tokens and licence keys.
- Keep only what the write-up actually refers to.
- No personal data, no third-party or employer data.
- No VM images, ISOs or full packet captures — those are excluded by `.gitignore`; describe them instead.
