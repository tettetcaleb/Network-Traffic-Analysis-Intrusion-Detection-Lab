# Network-Traffic-Analysis-Intrusion-Detection-Lab

tcpdump

Section 1: tcpdump Basics

1.1 Listing Available Network Interfaces
The first thing I did was list all the available network interfaces on my machine so I could figure out which one to monitor.
 ![Screenshot](Network_screenshots/Screenshot-2026-04-15-210201.png)

1.2 Monitoring a Network Interface
I decided to monitor the loopback interface (lo) to watch traffic being sent between processes on my local machine.
![Screenshot](Network_screenshots/Screenshot-2026-04-15-210521.png)

1.3 Simulating Traffic and Watching the Three-Way Handshake
I then simulated real traffic to watch the TCP three-way handshake happen in real time — the SYN, SYN-ACK, and ACK packets all showed up exactly as expected.
 ![Screenshot](Network_screenshots/Screenshot-2026-04-15-211243.png)

Section 2: Capturing Network Traffic

2.1 Capturing Traffic on a Selected Interface
I selected an interface and used tcpdump to display its live traffic, giving me a real-time view of what was moving across the network.
 ![Screenshot](Network_screenshots/Screenshot-2026-04-15-214518.png)

 2.2 Filtering Traffic by IP Address Range
I then applied a filter to only show packets coming from or going to a specific range of IP addresses. This is useful when you want to zero in on traffic from particular hosts instead of seeing everything at once.
 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-064300.png)

 Section 3: Analyzing a LockBit Ransomware PCAP
 This was the main part of this section ofthe lab. I got my hands on a PCAP file that was captured during a LockBit ransomware infection and started digging through it to trace what happened.

 
 3.1 Opening the PCAP File
I loaded the PCAP file into tcpdump and immediately noticed a huge amount of traffic between IP addresses on port 80.
 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-075550.png)

   
 3.2 Filtering for Port 80 Traffic
Since most of the traffic was on port 80 (HTTP), I filtered the capture to only show that. HTTP traffic is unencrypted so it's a good place to look for malware activity since you can actually read what's being sent.
 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-080708.png)

3.3 Grepping for HTTP GET and POST Requests
I used grep to filter the output further and only show HTTP GET and POST requests. This is where things got interesting — I could see exactly what the infected machine was requesting from the network.
![Screenshot](Network_screenshots/Screenshot-2026-04-17-081228.png)

3.4 Investigating the Source IP
At first glance, audiologg.exe looked like it might be associated with Microsoft. But I had to confirm it. I looked up the IP address it was being served from and found it was geolocated in Vietnam with zero connection to Microsoft. Legitimate Microsoft files don't get served from random servers in Vietnam, so this was a clear red flag.
 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-082115.png)


3.5 Extracting and Decoding the Malicious URL
I ran another grep to isolate all packets that contained the audiologg.exe filename, then opened them up to inspect the raw data. I copied out the encoded URL I found inside the packet and threw it into CyberChef to decode it, which gave me the full download URL the malware was using.
 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-085052.png)
 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-085614.png)

 3.6 Confirming on VirusTotal
Finally, I took the decoded URL and looked it up on VirusTotal. It came back flagged as malicious by multiple vendors that confirmed it. The URL is a known threat indicator tied to the LockBit ransomware campaign.
 ![Screenshot](Network_screenshots/Screenshot-2026-04-17-085702.png)

 Wireshark

Capture and display filters

1.1 Capturing Traffic with Filters
I used Wireshark to capture live traffic from www.example.com while excluding common ports such as 80 and 25. This allowed me to focus on less conventional traffic and significantly reduce noise in the capture.![Screenshot](Network_screenshots/Screenshot-2026-04-18-065202.png)

1.2 Using Display Filters

I applied a display filter to isolate only HTTP traffic from the captured packets. Unlike capture filters, display filters in Wireshark are applied after the fact, making it easy to pivot and re-examine the same capture from different angles without needing to recapture. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-065619.png)

Statistics

2.1 Capture File Properties

I used Capture File Properties to review metadata about the packet capture, including file size, total packet count, capture duration, and the interface it was recorded on. This is my standard starting point before diving into any capture.
   ![Screenshot](Network_screenshots/Screenshot-2026-04-18-070014.png)
  
2.2 Resolved Addresses

I examined the Resolved Addresses window to see which DNS lookups had been performed and how IP addresses mapped to domain names. This is useful for quickly identifying hosts without having to manually look up every IP address in the capture.  ![Screenshot](Network_screenshots/Screenshot-2026-04-18-070508.png)

2.3 Protocol Hierarchy

I used the Protocol Hierarchy view to get a percentage breakdown of every protocol present in the capture, organized by protocol layer. This gave me an at-a-glance picture of what types of traffic dominated the capture before I started drilling down. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-070848.png)

2.4 Conversations

I opened the Conversations view to see all two-way communication pairs in the capture. This showed me which hosts were talking to each other and the volume of data exchanged between them  useful for quickly identifying the most active or unusual connections.  ![Screenshot](Network_screenshots/Screenshot-2026-04-18-071142.png)

2.5 Endpoints

I then used the Endpoints feature, which is similar to Conversations but focused on individual hosts rather than communication pairs. This gave me a ranked list of every device that sent or received traffic, along with their total byte and packet counts.![Screenshot](Network_screenshots/Screenshot-2026-04-18-071325.png)

2.6 Following a Stream

One significant advantage Wireshark has over tcpdump is the ability to follow a specific traffic stream with a single click. Wireshark reconstructs the complete back and forth conversation between two hosts in a readable format a process that would require manual effort and additional tools in tcpdump. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-071851.png)

Section 3: Analyzing a Malware PCAP

For this section I analyzed a PCAP file sourced from Malware-Traffic-Analysis.net. My approach followed a structured methodology: start broad, identify anomalies, then drill down to extract actionable intelligence.

3.1 Getting a High-Level Overview

I began with Capture File Properties to understand the scope of the capture before touching anything else  how long it ran, how many packets were recorded, and what interfaces were involved.  ![Screenshot](Network_screenshots/Screenshot-2026-04-18-082528.png)

3.2 Identifying the Most Active Conversations

I used the Conversations view to identify which IP address pairs were generating the most traffic. This is one of the fastest ways to surface suspicious hosts that are communicating more than expected. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-082726.png)

3.3 Reviewing the Protocol Hierarchy

I checked the Protocol Hierarchy to identify which protocols were most prevalent in the capture. This helped me prioritize where to look  any unencrypted or unusual protocol usage at this stage is worth investigating further. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-082941.png)

3.4 Adding Source and Destination Port Columns

I configured Wireshark to display source and destination ports as visible columns in the packet list, since these are hidden by default. Having port information immediately visible eliminates the need to open individual packets just to check what service is being contacted. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-083240.png)

3.5 Filtering for HTTP Traffic and Identifying Suspicious Requests

I applied an HTTP display filter and began working through the results. Two things immediately stood out: the Host field in the HTTP header contained a raw IP address rather than a domain name, and the request was targeting a .DAT file. Both of these are commonly associated with malware delivery and warranted closer inspection.![Screenshot](Network_screenshots/Screenshot-2026-04-18-083750.png)

I followed the TCP stream for the suspicious request to reconstruct the full conversation. The User-Agent header identified the curl utility as the client, and I spotted an MZ file signature in the response body  the magic bytes that indicate a Windows executable. This led me to conclude that the victim machine had downloaded a malicious executable from a remote web server using curl. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-084253.png)

3.7 Extracting the File and Computing Its Hash

After confirming the filename, server IP, and download URL, I extracted the file directly from the PCAP using Wireshark's export functionality. I then computed its SHA256 hash to use as a unique fingerprint for threat intelligence lookups. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-085018.png)

3.8 Verifying the File on VirusTotal and Malware Bazaar

I submitted the hash to both VirusTotal and Malware Bazaar. Both platforms flagged the file as malicious and identified it as belonging to the Qakbot malware family, confirming my suspicion from the earlier traffic analysis. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-085304.png)  ![Screenshot](Network_screenshots/Screenshot-2026-04-18-085604.png)

3.9 Researching Qakbot TTPs

With the malware family confirmed, I researched Qakbot's known post-compromise tactics, techniques, and procedures (TTPs) to identify additional indicators I could hunt for in the PCAP. One well documented Qakbot behavior is performing ARP scans to enumerate other active hosts on the local network.

3.10 Confirming the ARP Scan

I filtered for ARP traffic and observed that the destination IP addresses in the ARP requests were descending sequentially a textbook indicator of an automated network enumeration scan. This confirmed that the infected host was actively mapping the local network in search of additional targets. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-090647.png)

3.11 Analyzing ICMP Traffic

I applied an ICMP filter to check for ping activity. The results showed two ICMP echo replies, meaning the attacker successfully identified two live machines on the network during their enumeration phase. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-091234.png)

3.12 Checking for Port Scanning Activity

I investigated whether the attacker had followed up their host discovery with port scanning against the identified machines. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-091320.png)

3.13 Uncovering Compromised SMTP Credentials

While reviewing the Protocol Hierarchy, I had noted the presence of SMTP traffic. I filtered for it and found an authentication login packet. Following the TCP stream revealed a base64-encoded authentication exchange, which I decoded in CyberChef. The output exposed a plaintext email address and password , compromised credentials the attacker had attempted to use for authentication. The login ultimately failed, but the credentials themselves represent a confirmed indicator of compromise.![Screenshot](Network_screenshots/Screenshot-2026-04-18-091754.png) ![Screenshot](Network_screenshots/Screenshot-2026-04-18-092314.png)

3.14 Extracting and Verifying SMB Files

Finally, I exported files transferred over SMB from within the PCAP. Several DLL files were present, which I extracted and hashed using SHA256. Every DLL returned the same hash as the initial Qakbot DLL I had identified earlier  confirming that the malware was actively propagating copies of itself across the network via SMB file shares. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-093059.png) ![Screenshot](Network_screenshots/Screenshot-2026-04-18-093907.png)

Wireshark PCAP Analysis Summary

I triaged a malware PCAP using a broad to narrow approach starting with Conversations and Protocol Hierarchy to identify suspicious hosts, then drilling into HTTP traffic where I found a curl-based executable download with an MZ signature. I extracted the file, hashed it, and confirmed it as Qakbot via VirusTotal. Further analysis uncovered an ARP scan, live host discovery, decoded SMTP credentials via CyberChef, and Qakbot DLLs propagating over SMB.
Next steps as a SOC analyst: isolate the host, block the C2 IP and malicious URL at the firewall, hunt for the file hash across all endpoints in the SIEM, reset compromised credentials, and document the full attack chain in an incident report.

   
