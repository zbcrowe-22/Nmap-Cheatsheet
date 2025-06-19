# Nmap (Network Mapper) Deep‑Dive Cheat‑Sheet

## Table of Contents
1. [Introduction](#introduction)
2. [Installing Nmap](#installing-nmap)
3. [Quick-Start Command Crib](#quick-start-command-crib)
4. [Timing & Performance (-T0…-T5)](#timing--performance--t0t5)
5. [Common Scan Types](#common-scan-types)
6. [NSE (Nmap Scripting Engine) Basics](#nse-nmap-scripting-engine-basics)
7. [Popular Script Categories & Examples](#popular-script-categories--examples)
8. [Script Arguments & Advanced Usage](#script-arguments--advanced-usage)
9. [Combining Options – Practical Recipes](#combining-options--practical-recipes)
10. [Output Formats](#output-formats)
11. [Further Reading](#further-reading)

## Introduction

Nmap (Network Mapper) is an open‑source tool for network discovery, security auditing, and service enumeration. Beyond simple port scanning, Nmap includes a powerful NSE (Nmap Scripting Engine) and granular timing controls, allowing you to balance speed, stealth, and reliability.

This cheat‑sheet condenses the most essential commands and flags so you can scan effectively without memorizing the entire man page.

## Installing Nmap

| OS | Command / Action |
|----|------------------|
| **Windows** | Download the installer from <https://nmap.org/download.html>; includes **Zenmap** (GUI), **Npcap** driver, and optional **Ndiff** & **Ncat**. |
| **macOS** | `brew install nmap` (Homebrew) **or** download the official **.dmg** from the Nmap site. |
| **Linux (Debian / Ubuntu)** | ```bash\nsudo apt update && sudo apt install nmap\n``` |
| **Linux (Fedora / RHEL / CentOS)** | ```bash\nsudo dnf install nmap\n``` |
| **Docker** | ```bash\ndocker run --rm -it instrumentisto/nmap -sV scanme.nmap.org\n``` |


## Quick Start Command Crib 

| Goal | Command |
|------|---------|
| **Fast TCP scan of top 1000 ports** | `nmap -T4 <target>` |
| **Verbose version detection** | `nmap -sV -vv <target>` |
| **OS detection + traceroute** | `nmap -A <target>` |
| **UDP scan (top 100 ports)** | `nmap -sU --top-ports 100 <target>` |
| **Save results to all formats (normal, grepable, XML)** | `nmap -oA results <target>` |
| **Run default “safe” NSE scripts** | `nmap --script default <target>` |

## Timing & Performance (-T0 … -T5)

| Level | Nick-name | Intended Use | Parallelism&nbsp;(rough) | Timeout aggressiveness |
|-------|-----------|--------------|--------------------------|------------------------|
| **T0** | Paranoid  | IDS evasion; very slow serial probes | **1** probe | Long (rare timeouts) |
| **T1** | Sneaky    | Light IDS evasion                   | 2 – 3  | High |
| **T2** | Polite    | Low-bandwidth links; friendly scanning | 5 – 10 | Moderate |
| **T3** | Normal    | *Default* (if none specified)       | ~15    | Balanced |
| **T4** | Aggressive| Fast scans on reliable LANs         | 25 – 100 | Shorter timeouts |
| **T5** | Insane    | Very fast on exceptionally stable networks | 300+  | Very short — risk of false negatives |


## Common Scan Types

| Flag | Name | Description |
|------|------|-------------|
| **-sS** | SYN (half-open) scan | Stealthy; sends SYN and waits for SYN-ACK. Default for root. |
| **-sT** | TCP connect scan | Full three-way handshake (no raw sockets). Default when unprivileged. |
| **-sU** | UDP scan | Sends empty UDP datagrams; slower, may need `--version-intensity 0`. |
| **-sV** | Version detection | Performs banner probes after port discovery. |
| **-sC** | Default scripts | Shortcut for `--script default`. |
| **-A**  | Aggressive | Combines `-sV`, OS detection, `--traceroute`, and default scripts. |

## NSE Basics

The Nmap Scripting Engine lets you automate tasks like vulnerability checks, brute‑forcing, and service enumeration.

*   Scripts live in /usr/share/nmap/scripts/ (Linux) or C:\Program Files (x86)\Nmap\scripts\ (Windows).

*   Categories: auth, default, discovery, exploit, safe, vuln, etc.

*   Run scripts with:

    nmap --script <category|name> <target>

*   Update scripts: sudo nmap --script-updatedb

## Popular Script Categories and Examples

| Category | What it does | Handy scripts |
|----------|--------------|---------------|
| **safe** | Non-intrusive information gathering | `ssh-hostkey`, `http-title`, `banner` |
| **default** | Scripts that run automatically with **`-sC`** or **`-A`**; generally safe | Mix of service probes & light vuln tests |
| **vuln** | Checks for known CVEs (can be disruptive) | `ftp-vsftpd-backdoor`, `http-shellshock`, `smb-vuln-ms17-010` |
| **brute** | Password guessing / brute-force | `ssh-brute`, `http-brute`, `ftp-brute` |
| **discovery** | Network-mapping helpers | `broadcast-dhcp-discover`, `snmp-info`, `ssl-cert` |
| **exploit** | Actively exploits a vulnerability (use with caution) | `http-vuln-cve2017-5638` |

Example – enumerate SSL certs on port 443:
nmap -p 443 --script ssl-cert <target>

## Script Arguments and Advanced Usage

1. Pass args with --script-args:

nmap --script ftp-anon,ftp-brute --script-args userdb=/path/users.txt,passdb=/path/passwords.txt <target>

2. Set global NSE variables:

nmap --script smb-enum-shares --script-args=unsafe=1 <target>

3. Increase intensity for version detection (0–9):

nmap -sV --version-intensity 9 <target>

## Combining Options - Practical Recipes

| Scenario | Command |
|----------|---------|
| **Fast LAN discovery + OS detect** | `nmap -T4 -F -O 192.168.1.0/24` |
| **Full TCP + top 100 UDP + default scripts** | `nmap -sS -sU --top-ports 100 -sC -T4 <target>` |
| **Thorough vuln scan, quiet timing** | `nmap -sS --script vuln -T2 <target>` |
| **Gather SSL/TLS info & Heartbleed check** | `nmap -p 443 --script ssl-cert,ssl-enum-ciphers,ssl-heartbleed <target>` |
| **Output grepable + XML** | `nmap -oG scan.grep -oX scan.xml <target>` |

## Output Formats 

| Flag | Format | File example |
|------|--------|--------------|
| **-oN** | Normal | `scan.txt` |
| **-oG** | Grepable | `scan.grep` |
| **-oX** | XML | `scan.xml` |
| **-oA** | All of the above | `scan.{txt,grep,xml}` |

Parse XML with tools like xsltproc or import into databases for long‑term tracking.


## Further Reading

* Official Book: Gordon “Fyodor” Lyon – Nmap Network Scanning (https://nmap.org/book/)

* Docs & Reference: https://nmap.org/docs.html

* Script Gallery: https://nmap.org/nsedoc/

* SecLists (common wordlists): https://github.com/danielmiessler/SecLists