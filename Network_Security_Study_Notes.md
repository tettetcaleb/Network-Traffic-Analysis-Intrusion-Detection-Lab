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
This is a picture of the 3 way handshake being shown in real time: ![Screenshot](Network_screenshots/Screenshot-2026-04-15-054315.png)

- **Reliable** — guarantees delivery, retransmits lost packets
- **Flow control** — prevents sender from overwhelming receiver
- **SOC Relevance** — SYN flood attacks abuse the handshake by sending thousands of SYNs without completing the connection, exhausting server resources. Incomplete handshakes in packet captures = red flag.

  ![Screenshot](Network_screenshots/Screenshot-2026-04-15-052030.png)

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

Common ports: ![Screenshot](Network_screenshots/Screenshot-2026-04-15-055542.png)

---

---

### UDP (User Datagram Protocol)
Connectionless protocol — fires packets without confirming receipt.

- **Faster** than TCP, no handshake overhead
- **No guarantee** of delivery or order
- **Used for**: VoIP, video streaming, DNS, online gaming

- **SOC Relevance** — DNS tunneling (data exfiltration technique) runs over UDP port 53. Unusual spikes in outbound UDP traffic to external IPs can indicate C2 (command and control) communication.

---

## 3. Packet Capture and Flow Analysis

Packet: A packet is a small, structured unit of data transmitted over a network, carrying both the actual information and control details to ensure it reaches its destination correctly.

Parts of a packer :
  
  -Header: is the control section at the start of a network packet that contains essential information for delivering and processing the data payload

  -Payload:  refers to the actual data being transmitted from the sender to the receiver.

  -Trailer: A packet trailer is supplemental data added to the end of a network packet, primarily used for error checking and ensuring data integrity.

Packert Capture(PCAP):

  -Intercepting Packets: involves capturing and analyzing data packets traveling across a network, which can be done using various tools and techniques for legitimate purposes like network monitoring and security analysis

  Methods of intercepting packets include:
    
    -Port mirroting (SPAN Ports): a technique used in networking to copy packets from a specified source port to a destination port without affecting the original packet processing on the network device, such as a switch or router.

   - Inline network Devices: These devices are integral to the functioning and security of a network, as they monitor, regulate, and control the flow of data packets within the network.

   - Network taps: A network tap is a hardware or software device that provides complete, real-time access to network traffic for monitoring, security, and performance analysis without impacting network performance.
  
Packet captures are stored through:

  PCAP Files:data file created by network packet capture tools like Wireshark.
  
  Advantages  of PCAP File:
    - most detailed record
    - includes packet [ayload

   Disavantages
  - resource intensive
  - requires significant storage space
  -Hard to scale

Flow Records: Flow records in networking are essential for monitoring and analyzing network traffic

  - Aggregated metadata
  - 5-tuple
      - source IP
      - source Port
      - Destination IP
      - Destination Port
      - Transport Protocol

    Advantages
      - Efficient bandwidth and storage requiremts
      - high level pattern and anamoly detection
      - easier to scale
   
    Disadvantages
      - lacks payload deteal
      - doesnt give you the whole picture


---

## 4. tcpdump

tcpdump — What It Is & How It Works
tcpdump is a command-line packet analyzer that runs on Linux/Unix systems. It intercepts and displays network packets passing through a network interface in real time, or saves them to a file for later analysis.


How It Works

tcpdump uses the libpcap library to capture raw packets directly from a network interface at the OS level, before the firewall or application layer processes them. You need root/sudo to run it because it operates in promiscuous mode — meaning it captures all traffic on the interface, not just traffic addressed to your machine.

Core Uses

1. Real-Time Traffic Monitoring

Watch live traffic on an interface to see what's happening on your network right now.
bashsudo tcpdump -i eth0
2. Capturing to a File

Save packets to a .pcap file to analyze later in Wireshark or other tools.
bashsudo tcpdump -i eth0 -w capture.pcap
3. Reading a Capture File

bashsudo tcpdump -r capture.pcap
4. Filtering Traffic

This is where tcpdump becomes powerful. You can isolate exactly what you care about using Berkeley Packet Filter (BPF) syntax.

Filter     Command

By hosttcpdump host 192.168.1.10

By source IPtcpdump src 10.0.0.5

By destination IPtcpdump dst 10.0.0.5

By porttcpdump port 443

By protocoltcpdump icmp or tcp or udp

Combine filterstcpdump src 10.0.0.5 and port 80

Exclude traffictcpdump not port 22

5. Verbose Output
See more packet detail — TTL, checksum, flags, etc.
bash
sudo tcpdump -v    # verbose
sudo tcpdump -vv   # more verbose
sudo tcpdump -vvv  # maximum detail

7. Show Packet Contents

Display the actual payload data in hex and ASCII — useful for spotting cleartext credentials or malicious strings.

bash
sudo tcpdump -X -i eth0 port 80

9. Limit Capture Count
Stop after capturing N packets.

bash
sudo tcpdump -c 100 -i eth0

Security & Forensics Uses

Incident response — capture traffic during an active breach to see what data is leaving

Detecting port scans — a flood of SYN packets to many ports is a clear nmap signature

Finding cleartext credentials — HTTP, FTP, Telnet traffic can expose passwords

C2 beaconing detection — regular outbound connections to unknown IPs at set intervals

DNS analysis — spot unusual domain lookups or DNS tunneling

Validating firewall rules — confirm traffic is actually being blocked


Key Flags Cheat Sheet
bash-i eth0       # specify interface
-w file.pcap  # write to file
-r file.pcap  # read from file
-n            # don't resolve hostnames (faster)
-nn           # don't resolve hostnames OR port names
-v / -vv      # verbosity
-X            # show hex + ASCII payload
-c 50         # capture only 50 packets
-A            # show payload in ASCII only

tcpdump vs Wireshark
|Feature       |tcpdump                           |Wireshark                                              |
|--------------|----------------------------------|-------------------------------------------------------|
|Interface     |Command line                      |GUI                                                    |
|Best for      |Remote servers, scripting,        |Deep analysis, visualization                           |
|resource use  |Very lightweight                  |Heavier                                                |
|Output        |Terminal / .pcap file             |.pcap file with visual decode                          |
|Typical use   |Capture on server,transfer,analyze    |Analyze locally                                 |

---

## 5. Wireshark

# Wireshark — Overview, Uses, and Why It's Preferred Over tcpdump

---

## What Is Wireshark?

Wireshark is a free, open-source **network protocol analyzer** (also called a packet sniffer). It captures network traffic in real time and lets you inspect each packet in granular detail — from the raw bytes at the frame level all the way up to the application layer. Originally released in 1998 as "Ethereal," it was rebranded Wireshark in 2006 and has since become the industry-standard tool for network analysis worldwide.

Wireshark works on all major platforms: **Windows, macOS, and Linux**.

---

## Core Capabilities

- **Live packet capture** — intercepts traffic on any network interface (Ethernet, Wi-Fi, loopback, etc.)
- **Offline analysis** — opens and inspects saved `.pcap` / `.pcapng` capture files
- **Deep protocol dissection** — decodes hundreds of protocols automatically (TCP/IP, HTTP, DNS, TLS, DHCP, FTP, SMTP, and many more)
- **Display filters** — lets you isolate specific traffic using a powerful, readable filter syntax (e.g., `http`, `ip.addr == 192.168.1.1`, `tcp.port == 443`)
- **Capture filters** — restricts what gets captured in the first place (BPF syntax, same as tcpdump)
- **Stream following** — reconstructs full TCP/UDP/SSL conversations for easy reading
- **Statistics & graphs** — I/O graphs, protocol hierarchy trees, endpoint/conversation lists, round-trip time analysis
- **Packet colorization** — color-codes traffic by protocol or condition for at-a-glance analysis
- **Export options** — exports packets, decrypted sessions, objects (HTTP files, images), and statistics in multiple formats

---

## Common Use Cases

### 1. Network Troubleshooting
Diagnose latency, packet loss, retransmissions, connection resets, and misconfigured services. Wireshark's visual timeline and TCP stream analysis make it easy to pinpoint exactly where a communication breaks down.

### 2. Security Analysis & Incident Response
Identify suspicious traffic patterns, unauthorized connections, malware callbacks (C2 traffic), ARP spoofing, DNS poisoning, or data exfiltration. Security analysts use Wireshark to investigate alerts and confirm whether a threat is real.

### 3. Protocol Development & Debugging
Developers writing custom protocols or debugging application-layer behavior can see exactly what bytes are being sent and received, and verify their implementation against the spec.

### 4. Learning & Education
Wireshark is widely used in university networking courses and certifications (CompTIA Network+, CEH, OSCP) because it makes abstract protocol concepts concrete and visible.

### 5. Performance Optimization
Measure handshake times, identify chattiness (excessive round-trips), find bandwidth hogs, and analyze the impact of changes to network configuration.

### 6. VoIP Analysis
Decode and even play back RTP audio streams to troubleshoot VoIP call quality issues.

---

## What Is tcpdump?

`tcpdump` is a **command-line packet capture tool** available on Unix/Linux systems. It captures packets and prints summaries to the terminal (or writes them to a `.pcap` file for later analysis). It is lightweight, scriptable, and available on almost every Unix-like system by default.

Example:
```bash
tcpdump -i eth0 -w capture.pcap port 80
```

---

## Wireshark vs. tcpdump — Why Wireshark Is Preferred

| Feature | Wireshark | tcpdump |
|---|---|---|
| Interface | Graphical (GUI) | Command-line only |
| Protocol decoding | Hundreds of protocols, full dissection | Basic summary output |
| Ease of use | Beginner-friendly | Requires CLI expertise |
| Display filters | Rich, intuitive filter language | BPF syntax only |
| Stream reconstruction | Yes (Follow TCP/UDP/SSL Stream) | No |
| Statistics & graphs | Built-in charts, timelines, trees | None (external tools needed) |
| Real-time colorization | Yes | No |
| Packet editing/replay | Via companion tools (editcap, Scapy) | Not built-in |
| Platform | Windows, macOS, Linux | Linux/Unix only (Windows via WinDump) |
| Remote capture | Yes (via `dumpcap` / `rpcapd` / SSH) | Yes |
| Learning curve | Low to medium | Medium to high |

### Key Reasons Wireshark Is Preferred

**1. Graphical Interface Makes Analysis Accessible**
tcpdump outputs raw text summaries that require experience to read quickly. Wireshark presents every packet in a structured three-pane layout — packet list, packet detail tree, and raw hex dump — making it usable for beginners and experts alike.

**2. Superior Protocol Dissection**
Wireshark fully decodes protocols and presents every field by name with its value. Instead of seeing `Flags [S]`, you see a labeled tree showing SYN=1, ACK=0, window size, and TCP options. tcpdump provides only a terse, abbreviated summary.

**3. Powerful Display Filters**
Wireshark's display filter language is human-readable and expressive. You can filter by any protocol field (`http.response.code == 404`, `dns.qry.name contains "evil"`, `tls.handshake.type == 1`). tcpdump uses Berkeley Packet Filter (BPF) syntax, which is powerful but significantly harder to write and limited to capture-time filtering.

**4. Stream Following**
With one click, Wireshark reconstructs the full back-and-forth of a TCP or UDP session as readable text. This is invaluable for seeing HTTP requests/responses, FTP sessions, or chat messages. tcpdump cannot do this natively.

**5. Built-in Statistics and Visualization**
Wireshark includes I/O graphs, protocol hierarchy statistics, endpoint and conversation tables, and RTT analysis — all without needing external tools. tcpdump must be paired with other tools (like tshark, Zeek, or Python scripts) to derive the same insights.

**6. Cross-Platform**
tcpdump is a Unix tool. Wireshark runs natively on Windows, macOS, and Linux, making it the default choice for analysts and developers working across environments.

### When tcpdump Is Still the Right Tool

Wireshark is not always the answer. tcpdump is preferred when:

- You are working on a **headless server** with no GUI
- You need to **capture remotely over SSH** and pipe output directly
- You need a **lightweight, scriptable** solution for automated capture in pipelines
- You want to **quickly grab packets** and save them for later Wireshark analysis

A common real-world workflow combines both: use `tcpdump` (or Wireshark's `dumpcap`) to capture on a remote server, transfer the `.pcap` file, and open it in Wireshark for full analysis.

---

## Quick-Reference Filter Examples (Wireshark)

```
# Show only HTTP traffic
http

# Filter by IP address
ip.addr == 192.168.1.100

# Show DNS queries only
dns.flags.response == 0

# Filter by TCP port
tcp.port == 443

# Show only TCP retransmissions
tcp.analysis.retransmission

# Show traffic between two hosts
ip.addr == 10.0.0.1 && ip.addr == 10.0.0.2

# Show packets with HTTP 4xx errors
http.response.code >= 400 && http.response.code < 500
```

---

## Summary

Wireshark is the gold standard for packet analysis because it combines the raw capture power of tcpdump with a full-featured graphical interface, deep protocol intelligence, and built-in analytical tools. For anyone doing serious network troubleshooting, security investigation, or protocol work, Wireshark provides capabilities that tcpdump simply cannot match out of the box. That said, they are complementary tools — and knowing both makes you a more effective network engineer or security analyst.

---

*Tool: Wireshark | Version: 4.x (current) | License: GNU GPLv2 | Site: [wireshark.org](https://www.wireshark.org)*

---

## 6. Intrusion Detection and Prevention Systems (IDS/IPS)

IDS — Intrusion Detection System

Monitors network traffic and alerts you when something suspicious is detected. It's purely passive. It sees the threat and tells you about it, but does nothing to stop it. Think of it as a security camera.

IPS — Intrusion Prevention System

Does everything an IDS does but can also actively block or drop malicious traffic in real time. It sits inline with your traffic, meaning all traffic flows through it. Think of it as a security guard, not just a camera.

Two detection methods:

Signature-based — compares traffic against a database of known attack patterns. Fast and accurate against known threats, blind to new ones.

Anomaly-based — learns what normal traffic looks like and alerts on anything that deviates. Catches new threats but generates more false positives.

Detection Methods

Signature-based Detection

Matches traffic against a database of known attack patterns. Great for catching known threats fast, but completely blind to anything new — if the signature doesn't exist, it won't detect it. This covers the Known Known quadrant — threats we know about and know how to detect.

Behavior-based Detection

Instead of looking for known patterns, it builds a baseline of what normal looks like and alerts on anything that deviates from it. Better at catching new and unknown threats but generates more false positives because not everything unusual is malicious. Covers the Known Unknown quadrant — we know threats exist but don't know exactly what they look like.

Rule-based Detection

You define the rules yourself based on your organization's specific policies — what's allowed, what's a violation. Highly flexible and customizable but requires constant maintenance because rules need to be updated as the environment changes. This is exactly what you'll be doing in Snort — writing your own rules.

The quadrant explained simply:

Known Known — we know the threat exists and we know what it looks like → Signature-based handles this

Known Unknown — we know threats exist but can't predict the exact form → Behavior-based handles this

Unknown Known — gaps in our knowledge about threats we should know about → training and intel sharing

Unknown Unknown — threats we don't know exist yet → hardest to defend against, no tool fully covers this



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
