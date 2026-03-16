# Cybersecurity-Home-Lab-SIEM-IDS-Network-Defense-

![Lab Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Splunk](https://img.shields.io/badge/SIEM-Splunk-orange)
![Suricata](https://img.shields.io/badge/IDS-Suricata-blue)
![Kali Linux](https://img.shields.io/badge/Attack-Kali%20Linux-red)
![Ubuntu](https://img.shields.io/badge/Server-Ubuntu%2022-purple)

A fully functional cybersecurity home lab simulating real-world enterprise attack and defense scenarios. Built from scratch to develop hands-on SOC Analyst skills including threat detection, log analysis, incident response, and Linux server hardening.

---

## 📋 Table of Contents

- [Lab Overview](#lab-overview)
- [Environment & Architecture](#environment--architecture)
- [Tools & Technologies](#tools--technologies)
- [What I Built](#what-i-built)
- [Key Accomplishments](#key-accomplishments)
- [Real Threat Detection](#real-threat-detection)
- [Incident Response](#incident-response)
- [Splunk Dashboard](#splunk-dashboard)
- [Skills Demonstrated](#skills-demonstrated)
- [Screenshots](#screenshots)

---

## 🧪 Lab Overview

This home lab was designed to simulate a real enterprise security environment where I could:

- Launch real network attacks using Kali Linux and Nmap
- Detect those attacks using Suricata IDS with Emerging Threats rulesets
- Ingest and analyze alerts in Splunk SIEM in real time
- Build a professional attack monitoring dashboard
- Detect and respond to **real external threat actors** that were discovered targeting the server during the lab
- Harden the server against active attacks using industry standard tools

---

## 🖥️ Environment & Architecture

```
┌─────────────────────────────────────────────────────┐
│                  HOME LAB NETWORK                   │
│                                                     │
│  ┌─────────────┐         ┌─────────────────────┐   │
│  │  Kali Linux  │────────▶│  Ubuntu Server 22   │   │
│  │  172.17.0.6  │ Attack  │  172.17.1.4         │   │
│  │  (Attacker)  │         │  Splunk + Suricata  │   │
│  └─────────────┘         └─────────────────────┘   │
│                                    ▲                │
│  ┌─────────────┐                   │ Forwarder      │
│  │ Windows 10  │───────────────────┤                │
│  │ 172.17.0.5  │                   │                │
│  │ (Endpoint)  │         ┌─────────┴───────┐       │
│  └─────────────┘         │ Win Server 2022  │       │
│                           │ 172.17.0.4      │       │
│                           │ Domain Controller│       │
│                           └─────────────────┘       │
└─────────────────────────────────────────────────────┘
```

| Machine | IP Address | Role |
|---|---|---|
| Ubuntu Server 22 | 172.17.1.4 | Splunk SIEM + Suricata IDS |
| Kali Linux | 172.17.0.6 | Attack Machine |
| Windows 10 | 172.17.0.5 | Endpoint / Log Source |
| Windows Server 2022 | 172.17.0.4 | Domain Controller / Log Source |

---

## 🛠️ Tools & Technologies

| Category | Tool |
|---|---|
| SIEM | Splunk Enterprise 9.3.0 |
| IDS | Suricata |
| Attack Simulation | Kali Linux, Nmap |
| Log Forwarding | Splunk Universal Forwarder |
| Firewall | UFW (Uncomplicated Firewall) |
| Brute Force Protection | Fail2Ban |
| Rulesets | Emerging Threats (ET Open), aleksibovellan/nmap |
| OS | Ubuntu Server 22, Windows 10, Windows Server 2022 |

---

## 🔧 What I Built

### 1. Suricata IDS Pipeline
- Installed and configured Suricata on Ubuntu Server 22
- Configured EVE JSON logging (`/var/log/suricata/eve.json`)
- Ran `suricata-update` to pull Emerging Threats open ruleset
- Enabled the `aleksibovellan/nmap` ruleset for dedicated Nmap scan detection
- Wrote custom local rules in `/etc/suricata/rules/local.rules`

### 2. Splunk SIEM Integration
- Configured `inputs.conf` to monitor `eve.json` in real time:
```ini
[monitor:///var/log/suricata/eve.json]
disabled = false
index = main
sourcetype = suricata
```
- Built SPL queries to surface and analyze alert data
- Resolved sourcetype misconfiguration (`json` → `suricata`)

### 3. Custom Nmap Detection Rules
Written in Suricata rule syntax to detect:
- `-sS` SYN Scan
- `-sT` TCP Connect Scan
- `-sU` UDP Scan
- `-sX` Xmas Scan
- `-sN` NULL Scan
- `-sA` ACK Scan
- `-f` Fragmented Packet Scan

### 4. Splunk Universal Forwarders
- Deployed forwarders on Windows 10 and Windows Server 2022
- Configured to forward Security, System, and Application event logs
- Verified pipeline: `Windows Endpoint → Forwarder → Splunk Indexer → SPL Search`

### 5. Linux Server Hardening
- Configured SSH hardening (`PermitRootLogin no`, `MaxAuthTries 3`)
- Deployed UFW firewall with IP whitelisting for lab machines only
- Installed and configured Fail2Ban with systemd backend
- Whitelisted lab machine IPs to prevent accidental lockout

---

## 🎯 Key Accomplishments

- ✅ Built a full end-to-end SIEM + IDS pipeline from scratch
- ✅ Detected **58,000+ real-time security events** in Splunk
- ✅ Caught Kali Linux Nmap scans with specific signature matching
- ✅ Built a 6-panel professional Splunk attack monitoring dashboard
- ✅ Detected real external threat actors targeting the server
- ✅ Responded to a live attack and reduced malicious traffic to zero
- ✅ Hardened Ubuntu server with firewall, Fail2Ban, and SSH hardening

---

## 🚨 Real Threat Detection

During the lab, the Splunk Live Alert Feed began showing **unrecognized external IPs** actively scanning and probing the server on port 22. These were not from the lab environment.

### Threat Intelligence Signatures Triggered:

| Signature | Severity | Description |
|---|---|---|
| ET CINS Active Threat Intelligence Poor Reputation IP | Medium | Known malicious IP from threat intel feed |
| ET DROP Dshield Block Listed Source | Medium | IP on Dshield blocklist |
| ET COMPROMISED Known Compromised or Hostile Host | Medium | Known compromised host |
| ET INFO SSH-2.0-Go version string | Low | Automated SSH scanning tool |

### Attacker IPs Detected:
- `78.128.114.130` — Dshield block listed + CINS poor reputation
- `139.19.117.130` — Known compromised/hostile host
- `159.65.51.52` — Repeated SSH scanning
- `167.71.132.200` — SSH scanning

This was **not a simulated exercise** — these were real external attackers hitting the server in real time, caught and identified by the Suricata + Splunk pipeline.

---

## 🛡️ Incident Response

Upon detecting the real external threat actors, I responded by:

**1. Immediate firewall deployment:**
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 172.17.0.4 to any port 22
sudo ufw allow from 172.17.0.5 to any port 22
sudo ufw allow from 172.17.0.6 to any port 22
sudo ufw allow from 172.17.0.4 to any port 8000
sudo ufw allow from 172.17.0.5 to any port 8000
sudo ufw allow from 172.17.0.6 to any port 8000
sudo ufw allow from 172.17.0.4 to any port 9997
sudo ufw allow from 172.17.0.5 to any port 9997
sudo ufw enable
```

**2. SSH hardening:**
```bash
PermitRootLogin no
MaxAuthTries 3
PubkeyAuthentication yes
PasswordAuthentication yes
```

**3. Fail2Ban deployment:**
```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 3
ignoreip = 127.0.0.1/8 172.17.0.4 172.17.0.5 172.17.0.6

[sshd]
enabled = true
port = 22
backend = systemd
```

**Result:** External attack traffic dropped to zero within minutes of firewall deployment.

---

## 📊 Splunk Dashboard

Built a full dark-themed **Nmap Attack Monitor** dashboard with 6 panels:

| Panel | Type | Description |
|---|---|---|
| Alerts Over Time | Line Chart | Attack frequency over 24 hours |
| Top Attacker IPs | Bar Chart | Most active source IPs |
| Top Attack Signatures | Pie Chart | Breakdown of alert types |
| Most Targeted Ports | Bar Chart | Most probed destination ports |
| Severity Distribution | Pie Chart | High / Medium / Low breakdown |
| Live Alert Feed | Table | Real-time alerts with timestamps |

### SPL Query Used:
```
index=main sourcetype=suricata event_type=alert
| eval severity_label=case(
    'alert.severity'=="1", "High",
    'alert.severity'=="2", "Medium",
    'alert.severity'=="3", "Low",
    1==1, "Unknown")
| eval Time=strftime(_time, "%Y-%m-%d %H:%M:%S")
| table Time, src_ip, dest_ip, dest_port, alert.signature, severity_label
| rename src_ip as "Attacker IP", dest_ip as "Target IP",
         dest_port as "Port", alert.signature as "Signature",
         severity_label as "Severity"
| sort - Time
```

---

## 🧠 Skills Demonstrated

| Category | Skills |
|---|---|
| SIEM | Splunk Enterprise, SPL queries, dashboard development, log ingestion |
| IDS | Suricata, EVE JSON, rule writing, ET Open ruleset management |
| Threat Detection | Nmap scan detection, threat intel signatures, IOC identification |
| Incident Response | Live threat identification, firewall deployment, attack mitigation |
| Linux Administration | Ubuntu Server 22, systemctl, SSH config, UFW, Fail2Ban, package management |
| Offensive Tools | Kali Linux, Nmap (-sS, -sT, -sU, -sX, -sA, -A flags) |
| Log Analysis | EVE JSON parsing, Windows Event Log forwarding, Splunk indexing |
| Network Security | Firewall rules, IP whitelisting, port management, network segmentation |

---

## 📸 Screenshots

> Add your dashboard screenshots here

| Screenshot | Description |
|---|---|
| `dashboard1.png` | Full Nmap Attack Monitor dashboard — alerts over time spike |
| `dashboard2.png` | Live Alert Feed with external threat actor IPs |
| `attacks_port22.png` | ET CINS, ET DROP, ET COMPROMISED alerts |
| `ufw_rules.png` | UFW firewall rules confirmed active |

---

## 👩🏾‍💻 About

**Ashley McGee**
Aspiring SOC Analyst | A.S. in Information Technology / Cybersecurity (Expected December 2026)
Northeast Wisconsin Technical College | Green Bay, WI

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://linkedin.com/in/ashley-mcgee-972006367)

---

*This lab was completed on March 15, 2026 as part of an ongoing series of hands-on cybersecurity projects.*
