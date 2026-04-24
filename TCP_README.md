 # 🔍 tcpdump — LockBit Ransomware PCAP Analysis

## Overview
Analyzed a PCAP captured during an active LockBit ransomware infection using tcpdump. Traced the attacker's C2 communication, identified a malicious executable download, decoded the obfuscated URL, and confirmed the threat via VirusTotal.

---

## 🔧 Investigation Steps

### 1. Loading & Parsing the PCAP
Loaded the PCAP into tcpdump and immediately observed a high volume of HTTP traffic between internal hosts and an external IP on port 80.

```bash
tcpdump -tt -r 2021-09-14.pcap port 80
```

 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-080708.png)

---

### 2. Filtering HTTP GET & POST Requests
Filtered the capture for only HTTP GET and POST requests using grep, surfacing exactly what the infected machine was requesting from the network.

```bash
tcpdump -tt -r 2021-09-14.pcap port 80 and host 10.0.0.168 | grep -E "GET|POST"
```

 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-081228.png)

Two requests immediately stood out — repeated GET requests for a file called `audiodg.exe` being served from an external IP. Legitimate Windows system files are not downloaded from remote HTTP servers.

---

### 3. IP Reputation — Vietnam-Based C2 Server
Looked up the server IP (`103.232.55.148`) in DomainTools. The IP resolved to a Vietnamese hosting provider with no association to Microsoft — confirming the `audiodg.exe` filename was being used as a masquerade for malware.

 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-082115.png)

---

### 4. Extracting Raw Packet Data
Isolated all packets referencing the malicious filename and inspected the raw payload to extract the encoded download URL.

 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-085052.png)

---

### 5. Decoding the URL in CyberChef
Copied the percent-encoded URL from the packet payload and decoded it in CyberChef using URL Decode + Defang URL, revealing the full C2 download path:

```
hxxp://103[.]232[.]55[.]148/service/[.]audiodg[.]exe
```

 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-085614.png)

---

### 6. VirusTotal Confirmation — LockBit C2
Submitted the defanged URL to VirusTotal. Multiple vendors flagged it as malicious, confirming it as a known threat indicator tied to the LockBit ransomware campaign.

**9/95 vendors flagged as malicious**

 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-085702.png)

---

## 🧠 Attack Summary

| Stage | Finding |
|-------|---------|
| C2 Communication | Repeated HTTP GET requests to `103.232.55.148` on port 80 |
| Masquerading | Executable named `audiodg.exe` to blend in with legitimate Windows processes |
| C2 Infrastructure | IP hosted by Vietnamese provider — no Microsoft affiliation |
| Payload URL | `hxxp://103[.]232[.]55[.]148/service/[.]audiodg[.]exe` |
| Threat Intel | Confirmed malicious by VirusTotal — LockBit ransomware campaign |

---

## 📋 Skills Demonstrated
- PCAP analysis with tcpdump
- HTTP traffic filtering and grep-based IOC extraction
- IP reputation lookup (DomainTools WHOIS)
- URL decoding and defanging (CyberChef)
- Threat intelligence lookup (VirusTotal)
- LockBit C2 traffic identification

---

## 🛠️ Tools & Technologies

![tcpdump](https://img.shields.io/badge/tcpdump-darkgreen?style=for-the-badge)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![CyberChef](https://img.shields.io/badge/CyberChef-darkgreen?style=for-the-badge)
![VirusTotal](https://img.shields.io/badge/VirusTotal-394EFF?style=for-the-badge&logo=virustotal&logoColor=white)
![DomainTools](https://img.shields.io/badge/DomainTools-orange?style=for-the-badge)
