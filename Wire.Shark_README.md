# 🔍 Malware PCAP Analysis — Qakbot Detection & Investigation

![Screenshot](Network_screenshots/Screenshot-2026-04-18-085304.png)

## Overview
Analyzed a real-world malware PCAP from Malware-Traffic-Analysis.net using a structured broad-to-narrow triage methodology. Identified a Qakbot infection chain including initial payload delivery, network enumeration, credential theft, and lateral movement via SMB — then documented full attacker TTPs and recommended SOC response actions.

---

## 🏗️ Analysis Methodology

Used a structured triage approach:
1. Start broad — Conversations, Protocol Hierarchy, Endpoints
2. Identify anomalies — unusual protocols, raw IP hosts, suspicious file types
3. Drill down — follow streams, extract files, compute hashes, query threat intel

---

## 🔧 Investigation Steps

### 1. Identifying the Most Active Conversations
Opened the Conversations view to surface which IP pairs were generating the most traffic — the fastest way to identify suspicious hosts communicating at abnormal volumes.

![Screenshot](Network_screenshots/Screenshot-2026-04-18-082726.png)

---

### 2. Protocol Hierarchy Review
Reviewed the Protocol Hierarchy to identify which protocols dominated the capture. Presence of SMB, SMTP, and HTTP alongside TLS warranted closer inspection.

![Screenshot](Network_screenshots/Screenshot-2026-04-18-082941.png)

---

### 3. Adding Port Columns & Filtering HTTP Traffic
Configured Wireshark to display source and destination ports as visible columns, then filtered for HTTP traffic. Two immediate red flags surfaced:
- Host field contained a **raw IP address** instead of a domain name
- The request targeted a **`.DAT` file** — a common malware delivery pattern

![Screenshot](Network_screenshots/Screenshot-2026-04-18-083750.png)

---

### 4. Following the TCP Stream — MZ Signature Identified
Followed the TCP stream for the suspicious HTTP request. The User-Agent was `curl/7.83.1` and the response body began with `MZ` — the magic bytes indicating a **Windows PE executable** was downloaded disguised as a `.DAT` file.

![Screenshot](Network_screenshots/Screenshot-2026-04-18-084253.png)

---

### 5. File Extraction & SHA256 Hash
Exported the file directly from the PCAP and computed its SHA256 hash for threat intelligence lookup.

```bash
$ file 86607.dat
86607.dat: PE32 executable for MS Windows 4.00 (DLL), Intel i386, 6 sections

$ sha256sum 86607.dat
713207d9d9875ec88d2f3a53377bf8c2d620147a4199eb183c13a7e957056432  86607.dat
```

![Screenshot](Network_screenshots/Screenshot-2026-04-18-085018.png)

---

### 6. Threat Intelligence — Confirmed Qakbot
Submitted the hash to VirusTotal and MalwareBazaar. Both platforms flagged it as malicious and identified it as **Qakbot** (also known as QBot/Bobik) — a banking trojan and loader known for credential theft, lateral movement, and ransomware delivery.

**VirusTotal: 55/72 vendors flagged as malicious**

![Screenshot](Network_screenshots/Screenshot-2026-04-18-085304.png)

![Screenshot](Network_screenshots/Screenshot-2026-04-18-085604.png)

---

### 7. ARP Scan — Network Enumeration Confirmed
Filtered for ARP traffic. Destination IPs decremented sequentially — a textbook indicator of **automated host enumeration**. Confirmed the infected host was actively mapping the local network.

![Screenshot](Network_screenshots/Screenshot-2026-04-18-090647.png)

---

### 8. ICMP — Live Host Discovery
Filtered for ICMP traffic. Two echo replies confirmed the attacker successfully identified **two live machines** on the network during enumeration.

![Screenshot](Network_screenshots/Screenshot-2026-04-18-091234.png)

---

### 9. Port Scanning Activity
Investigated follow-up scanning against the discovered hosts, observing TCP connection attempts across multiple ports on the identified live machines.

![Screenshot](Network_screenshots/Screenshot-2026-04-18-091320.png)

---

### 10. SMTP Credential Theft
Filtered for SMTP traffic and found an authentication exchange. Following the TCP stream revealed base64-encoded credentials. Decoded in CyberChef — output exposed plaintext email and password belonging to a compromised account. Authentication ultimately failed but the credentials represent a confirmed IOC.

![Screenshot](Network_screenshots/Screenshot-2026-04-18-091754.png)

![Screenshot](Network_screenshots/Screenshot-2026-04-18-092314.png)

---

### 11. SMB Lateral Movement — Qakbot DLL Propagation
Exported files transferred over SMB. Three DLL files were present — all returned the **same SHA256 hash** as the original Qakbot DLL, confirming the malware was **propagating copies of itself across the network** via SMB shares.

![Screenshot](Network_screenshots/Screenshot-2026-04-18-093059.png)

```bash
$ sha256sum *
713207d9d9875ec88d2f3a53377bf8c2d620147a4199eb183c13a7e957056432  %5cefweioirfbtk.dll
713207d9d9875ec88d2f3a53377bf8c2d620147a4199eb183c13a7e957056432  %5cltoawuimupfxvg.dll
713207d9d9875ec88d2f3a53377bf8c2d620147a4199eb183c13a7e957056432  %5cumtqqzkklrgp.dll
```

![Screenshot](Network_screenshots/Screenshot-2026-04-18-093907.png)

---

## 🧠 Attack Chain Summary

| Stage | Technique | Evidence |
|-------|-----------|----------|
| Initial Access | curl-based HTTP download of PE executable disguised as `.DAT` | HTTP stream, MZ header |
| Execution | PE32 DLL dropped on victim host | `file` command output |
| Discovery | ARP scan of local subnet | Sequential ARP broadcasts |
| Discovery | ICMP ping sweep | 2 echo replies |
| Discovery | TCP port scan against live hosts | SYN packets to ports 445, 139, 80 |
| Credential Access | SMTP auth with stolen credentials | Base64-decoded SMTP stream |
| Lateral Movement | Qakbot DLL copied to remote hosts via SMB | Matching SHA256 hashes across 3 DLLs |

---

## 🚨 SOC Response Recommendations
- Isolate the infected host immediately
- Block the C2 IP (`128.254.207.55`) and malicious URL at the firewall
- Hunt for the file hash across all endpoints in the SIEM
- Reset compromised email credentials
- Scan all SMB shares for matching DLL hashes
- Document full attack chain in an incident report

---

## 📋 Skills Demonstrated
- Malware PCAP triage (broad-to-narrow methodology)
- HTTP stream analysis and PE file identification
- File extraction and SHA256 hashing
- Threat intelligence lookup (VirusTotal, MalwareBazaar)
- ARP scan and port scan detection
- SMTP credential extraction and base64 decoding (CyberChef)
- SMB lateral movement analysis

---

## 🛠️ Tools & Technologies

![Wireshark](https://img.shields.io/badge/Wireshark-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![VirusTotal](https://img.shields.io/badge/VirusTotal-394EFF?style=for-the-badge&logo=virustotal&logoColor=white)
![CyberChef](https://img.shields.io/badge/CyberChef-darkgreen?style=for-the-badge)
![MalwareBazaar](https://img.shields.io/badge/MalwareBazaar-red?style=for-the-badge)
