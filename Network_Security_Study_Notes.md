# Network Security Study Notes
> Course: Network Security (YouTube)
> Repo: [Network-Traffic-Analysis-Intrusion-Detection-Lab](https://github.com/tettetcaleb/Network-Traffic-Analysis-Intrusion-Detection-Lab)

---

## Table of Contents
1. [Network Security Theory](#1-network-security-theory)
2. [Network Layer Protocols](#2-network-layer-protocols)
3. [Packet Capture and Flow Analysis](#3-packet-capture-and-flow-analysis)
4. [tcpdump](#4-tcpdump)
5. [Wireshark](#5-wireshark)
6. [Intrusion Detection and Prevention Systems (IDS/IPS)](#6-intrusion-detection-and-prevention-systems-idsips)
7. [Snort](#7-snort)
8. [Network Traffic Analysis Practice](#8-network-traffic-analysis-practice)

---

## 1. Network Security Theory

### What is Network Security?
The practice of protecting a computer network from unauthorized access, misuse, malfunction, or attacks. It covers both hardware and software technologies.

### Key Goals (CIA Triad)
| Goal | Meaning |
|------|---------|
| **Confidentiality** | Only authorized users can access data |
| **Integrity** | Data is not tampered with in transit |
| **Availability** | Systems and data are accessible when needed |

### SOC Relevance
A SOC analyst's job is to monitor, detect, and respond to threats against all three of these goals. Everything in this course feeds into that workflow.

Source & Destination Port — identify which application is sending and receiving (e.g. port 443 = HTTPS, port 22 = SSH). SOC use: spotting unusual ports used for data exfiltration or C2 traffic.

Sequence Number — a number assigned to each byte of data so the receiver can reassemble packets in the correct order. Also used to detect missing or duplicate packets.

Acknowledgment Number — tells the sender which byte the receiver expects next, confirming what was successfully received. This is how TCP guarantees delivery.

Data Offset — tells the receiver where the actual data starts, since the header can vary in size due to options.

Reserved — unused bits, set to zero. Reserved for future use by the protocol.

Flags — 9 control bits that manage the state of the connection. The key ones are SYN (start connection), ACK (acknowledge), FIN (close connection), RST (reset/abort), and PSH (send data immediately). SOC use: a flood of SYN packets with no ACK = SYN flood attack. Unexpected RST packets can indicate a port scan.

Window Size — how many bytes the receiver can accept before needing an acknowledgment. Controls the flow so neither side gets overwhelmed.

Checksum — verifies the header and data haven't been corrupted in transit. If it doesn't match, the packet is dropped.

Urgent Pointer — only active when the URG flag is set, points to urgent data that should be processed immediately. Rarely used in normal traffic.

Options — optional extra settings like maximum segment size or timestamps. Padded to keep the header aligned to 32-bit boundaries.
---

## 2. Network Layer Protocols

### Internet Protocol (IP)
The addressing system that identifies every device on a network.

| Version | Format | Example |
|---------|--------|---------|
| IPv4 | 32-bit, 4 octets | 192.168.1.1 |
| IPv6 | 128-bit, 8 groups | 2001:0db8::1 |

- **IP Routing** — the process of forwarding packets from source to destination across networks using routing tables.
- **SOC Relevance** — every alert has a source/destination IP. Knowing routing helps identify internal vs. external traffic and detect IP spoofing.

---

### TCP (Transmission Control Protocol)
Connection-oriented protocol — both sides must agree before data is sent.

**Three-Way Handshake:**
```
Client  →  SYN        →  Server
Client  ←  SYN-ACK    ←  Server
Client  →  ACK        →  Server
(Connection established)
```

- **Reliable** — guarantees delivery, retransmits lost packets
- **Flow control** — prevents sender from overwhelming receiver
- **SOC Relevance** — SYN flood attacks abuse the handshake by sending thousands of SYNs without completing the connection, exhausting server resources. Incomplete handshakes in packet captures = red flag.

---

### UDP (User Datagram Protocol)
Connectionless protocol — fires packets without confirming receipt.

- **Faster** than TCP, no handshake overhead
- **No guarantee** of delivery or order
- **Used for**: VoIP, video streaming, DNS, online gaming

- **SOC Relevance** — DNS tunneling (data exfiltration technique) runs over UDP port 53. Unusual spikes in outbound UDP traffic to external IPs can indicate C2 (command and control) communication.

---

## 3. Packet Capture and Flow Analysis

> *Notes coming — section in progress*

---

## 4. tcpdump

> *Notes coming — section in progress*

---

## 5. Wireshark

> *Notes coming — section in progress*

---

## 6. Intrusion Detection and Prevention Systems (IDS/IPS)

> *Notes coming — section in progress*

---

## 7. Snort

> *Notes coming — section in progress*

---

## 8. Network Traffic Analysis Practice

> *Notes coming — section in progress*

---

## Key Terms Glossary

| Term | Definition |
|------|------------|
| **Packet** | A unit of data transmitted over a network |
| **Protocol** | A set of rules governing how data is transmitted |
| **Port** | A logical endpoint for communication (e.g., port 80 = HTTP) |
| **IP Spoofing** | Faking a source IP address to disguise the origin of an attack |
| **SYN Flood** | A DoS attack that abuses the TCP handshake |
| **DNS Tunneling** | Encoding data in DNS queries to exfiltrate information |
| **C2 (Command & Control)** | A server used by attackers to communicate with compromised systems |
| **pcap** | Packet capture file format used by Wireshark and tcpdump |
| **IDS** | Intrusion Detection System — monitors and alerts on suspicious activity |
| **IPS** | Intrusion Prevention System — monitors and actively blocks threats |

---

*Last updated: April 2026*
