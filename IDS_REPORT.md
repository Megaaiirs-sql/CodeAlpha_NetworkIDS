# Network Intrusion Detection System with Suricata

CodeAlpha Cybersecurity Internship - Task 4: Network Intrusion Detection System

## Overview

This project sets up Suricata, an open-source network IDS/IPS engine, to monitor live network traffic, detect suspicious activity using both community-maintained threat signatures and a custom-written rule, and log alerts for review.

## Tooling

- Suricata 8.0.7 (pre-installed on Kali Linux)
- suricata-update for pulling the Emerging Threats Open ruleset
- Interface monitored: eth0

## Setup and Configuration

1. Installed suricata-update and pulled the full Emerging Threats Open ruleset, loading 69,071 rules (53,114 enabled after dependency resolution), on top of Suricata's 445 built-in protocol/decoder event rules.
2. Verified the interface configuration in /etc/suricata/suricata.yaml - confirmed it was already set to monitor eth0 by default.
3. Ran Suricata in live IDS mode against real traffic:

sudo suricata -i eth0 -l /var/log/suricata/

## Detection Testing

### Test 1: Nmap scan detection

Ran an aggressive Nmap scan (nmap -sS -A scanme.nmap.org, Nmap's official public test target) while Suricata was actively monitoring. Suricata detected and logged 249 separate alerts for the signature ET SCAN Possible Nmap User-Agent Observed, correctly identifying the scanning activity in real time. It also independently flagged a protocol anomaly: SURICATA HTTP Request unrecognized authorization method.

Example alert:

[**] [1:2024364:5] ET SCAN Possible Nmap User-Agent Observed [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 10.0.0.2:50358 -> 45.33.32.156:631

### Test 2: Custom rule - ICMP ping detection

To demonstrate rule-writing capability (not just running pre-built signatures), wrote a custom rule:

alert icmp any any -> any any (msg:"CUSTOM Ping (ICMP Echo Request) Detected"; itype:8; classtype:misc-activity; sid:1000001;

## Key Takeaways

- Signature-based IDS is only as good as its rule set and its visibility into real traffic. Several early test attempts failed silently because the monitored interface didn't actually see the generated traffic (e.g. scanning a machine's own IP can route internally via loopback rather than crossing the monitored NIC) - a reminder that IDS deployment requires confirming the sensor is positioned somewhere it can actually observe the traffic of interest.
- Community rule sets (Emerging Threats Open) provide broad, well-maintained coverage - the Nmap scan was caught immediately without any custom tuning, out of tens of thousands of pre-built signatures.
- Writing custom rules is straightforward once the syntax (action, protocol, direction, options) is understood, and it's necessary for detecting organization-specific threats that generic rule sets won't cover.
- IDS output requires triage. Not every alert is equally severe - the DHCP hostname notice (Priority 1, informational) is a very different finding from a scan-detection alert (Priority 1, but indicating active probing), and a real analyst needs to read classification and context, not just alert counts.

## Author

Built as part of the CodeAlpha Cybersecurity Internship (Task 4: Network Intrusion Detection System).
