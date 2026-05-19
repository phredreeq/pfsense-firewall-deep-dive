# pfSense Firewall Deep Dive
## Understanding Firewall Rules, NAT, DHCP and DNS

---

## Overview
pfSense is an open source firewall and router used
by businesses, universities and home labs worldwide.
This document covers the core concepts every SOC
analyst needs to understand about firewall operation
— rules processing, NAT, DHCP and logging.

---

## Objectives
- Understand how pfSense firewall rules are processed
- Learn NAT and how pfSense manages internal traffic
- Understand DHCP lease process and security risks
- Identify pfSense logging capabilities for SOC work

---

## Environment
| Component | Role |
|---|---|
| **pfSense** | Firewall, router, DHCP server, DNS resolver |
| **WAN Interface** | Connects to internet |
| **LAN Interface** | Connects to internal VMs |
| **Kali Linux** | Internal VM — attacker platform |
| **Windows VM** | Internal VM — target machine |
| **Ubuntu Server** | Internal VM — Linux target |

---

## What pfSense Does

### Four Core Jobs

Job 1 — Router
Routes traffic between networks:
Internet and WAN → pfSense → LAN and Your VMs

Job 2 — Firewall
Controls what traffic is allowed:
Every packet is checked against rules
Each packet is either ALLOWED or BLOCKED

Job 3 — DHCP Server
Automatically assigns IP addresses to devices:
IP Address, Subnet Mask, Gateway, DNS Server

Job 4 — DNS Resolver
Resolves domain names for internal network:
All DNS queries go to pfSense first
pfSense forwards to upstream DNS if needed

---

## Firewall Rules

### How Rules Are Processed
Rules are read top to bottom — first match wins:

Example ruleset:
Rule 1 — BLOCK 192.168.1.200 to Any
Rule 2 — BLOCK Any to Port 22
Rule 3 — ALLOW 192.168.1.0/24 to Any
Rule 4 — BLOCK All

Traffic from 192.168.1.50 to port 80:
- Rule 1 — source is not 192.168.1.200 — skip
- Rule 2 — destination is not port 22 — skip
- Rule 3 — source is in 192.168.1.0/24 — ALLOWED

### Rule Components
| Component | Purpose | Example |
|---|---|---|
| Action | Allow or Block | BLOCK |
| Interface | Traffic source | LAN |
| Protocol | TCP UDP ICMP | TCP |
| Source | Origin address | 192.168.1.200 |
| Destination | Target address | Any |
| Port | Service port | 22 |
| Log | Record traffic | Yes |

### Default Rules
LAN to Any — ALLOW — VMs can reach internet
WAN to LAN — BLOCK — internet cannot reach VMs

---

## NAT — Network Address Translation

### Why NAT Exists
IPv4 has limited addresses — NAT lets many devices
share one public IP address.

Example:
192.168.10.102 Kali Linux    ↘
192.168.10.103 Windows VM    → 102.89.45.201 → Internet
192.168.10.104 Ubuntu Server ↗

All three VMs share one public IP.
pfSense tracks who is who using a NAT table.

### NAT Table Example
| Internal IP | Internal Port | External IP | External Port |
|---|---|---|---|
| 192.168.10.102 | 52341 | 142.251.216.110 | 443 |
| 192.168.10.103 | 49821 | 104.20.23.154 | 80 |
| 192.168.10.104 | 51234 | 8.8.8.8 | 53 |

### Types of NAT
| Type | Purpose | Security Risk |
|---|---|---|
| Outbound NAT | Internal to internet | Low |
| Port Forward | Internet to internal | High if misconfigured |
| 1:1 NAT | One public IP per device | Medium |

---

## DHCP — Dynamic Host Configuration Protocol

### What DHCP Assigns
Every device receives four things:
- IP Address — unique network address
- Subnet Mask — network size
- Gateway — pfSense IP — door to internet
- DNS Server — pfSense IP — name resolution

### DORA Process
The four steps of DHCP IP assignment:

Step 1 — DISCOVER
Device broadcasts to network
I need an IP address — anyone there?

Step 2 — OFFER
pfSense responds to device
I will give you 192.168.10.102

Step 3 — REQUEST
Device confirms to pfSense
Yes please — I will take 192.168.10.102

Step 4 — ACKNOWLEDGE
pfSense confirms to device
Confirmed — 192.168.10.102 is yours for 24 hours

### DHCP Security Risks
| Attack | What happens | Impact |
|---|---|---|
| DHCP Starvation | Attacker requests all IPs | Legitimate devices get none |
| Rogue DHCP Server | Attacker assigns wrong gateway | Man in the Middle attack |

---

## pfSense Logging for SOC Analysts

pfSense logs everything — essential for threat detection:

| Log Type | What it records | SOC Value |
|---|---|---|
| Firewall logs | Every allowed and blocked connection | Detect port scans and attacks |
| DHCP logs | Every IP assignment | Track device activity |
| DNS logs | Every domain query | Detect DGA and tunneling |
| System logs | Configuration changes | Detect unauthorized changes |

---

## Security Relevance

### What SOC Analysts Look for in pfSense Logs

Firewall logs:
- Many blocked connections from one IP — port scan
- Blocked connections to sensitive ports — targeted attack
- Unusual outbound traffic — data exfiltration

DHCP logs:
- Unknown MAC addresses — unauthorized device
- Rapid IP requests — DHCP starvation attack
- Duplicate IP assignments — rogue DHCP server

DNS logs:
- Random domain queries — DGA malware
- Long query strings — DNS tunneling
- High NXDOMAIN rate — infected machine

---

## Key Concepts Summary

| Concept | Plain English |
|---|---|
| Stateful firewall | Tracks connections — allows responses to outbound traffic |
| Rule order | First matching rule wins — order is critical |
| NAT | Many private IPs share one public IP |
| DORA | Four step process for automatic IP assignment |
| Port forward | Allows internet to reach internal services — risky if misconfigured |
| Rogue DHCP | Fake DHCP server assigns wrong gateway — enables MITM |

---

## 🔗 Related Projects
- [Firewall Log Analysis](https://github.com/Phredreeq/firewall-log-analysis)
- [DNS Analysis and Threat Detection](https://github.com/Phredreeq/dns-analysis-threat-detection)
- [Network Traffic Analysis](https://github.com/Phredreeq/network-traffic-analysis-wireshark)

---

## 👤 Author
Fredrick Agufenwa

Cybersecurity Student | SOC & Threat Detection
