# Cybersecurity portfolio

A public record of my cybersecurity project work for **WQE7001 Cyber Security**: an isolated lab I built and broke on purpose, a service scan of the target inside it, and what I learned from both.

**Live site:** <https://25070270CYS.github.io/cybersecurity-portfolio/>

## What is in this repository

| Path | What it is |
| --- | --- |
| [`index.md`](index.md) | Home page of the published site: about me, skills, certifications, projects, learning journey, career objective, contact |
| [`_config.yml`](_config.yml) | Jekyll and GitHub Pages settings |
| [`projects/virtual-lab-setup/`](projects/virtual-lab-setup/) | Project 1 — building an isolated virtual lab |
| [`projects/nmap-service-scan/`](projects/nmap-service-scan/) | Project 2 — discovering and interpreting exposed services |
| [`assets/images/`](assets/images/) | Images used by the site |
| [`.gitignore`](.gitignore) | Keeps secrets, lab images and large binaries out of the repository |

## Projects

- **Virtual lab setup** — [write-up](projects/virtual-lab-setup/). VMware host with Kali Linux and Metasploitable 2 on a host-only network, so attacks never leave the machine.
- **Nmap service scan** — [write-up](projects/nmap-service-scan/). Host discovery, port scanning and version detection, plus what each open port means and how it should be fixed.

## How the evidence is handled

- Every screenshot and log comes from my own machine or from a legal practice platform.
- Evidence is sanitised before it is committed: no home IP addresses, hostnames, usernames, tokens, email addresses or student ID.
- Virtual machines, ISOs and full packet captures are excluded by `.gitignore` and described in the write-ups instead.

## Submission

This repository is marked at the release tag [`v1.0-submission`](releases/tag/v1.0-submission).
