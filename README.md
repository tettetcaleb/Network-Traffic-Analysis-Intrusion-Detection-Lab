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
 - This is an example of me using wireshark to capture traffice from www.example.com woles excluding ports such as 80 and 25 ![Screenshot](Network_screenshots/Screenshot-2026-04-18-065202.png)

 - here is me using display filters to display only http traffic ![Screenshot](Network_screenshots/Screenshot-2026-04-18-065619.png)

Statistics
- Capture file properties: This allows me to view metadata and useful data about the packet file
   ![Screenshot](Network_screenshots/Screenshot-2026-04-18-070014.png)
  
- Reserved Address : This allows me to see where a DNS lookup was peformed and an IP address was maaped to a domain  ![Screenshot](Network_screenshots/Screenshot-2026-04-18-070508.png)

- Protocol Heirachy : This provides percentages for every protocol in a packet capture, organized by protocol layers.  ![Screenshot](Network_screenshots/Screenshot-2026-04-18-070848.png)

- Conversations : This shows me all two-way communication pairs.  ![Screenshot](Network_screenshots/Screenshot-2026-04-18-071142.png)

- Endpoints : This feature similar is to Conversations but is focused on individual hosts rather than pairs![Screenshot](Network_screenshots/Screenshot-2026-04-18-071325.png)

wireshark also allows me to follow a specific stream of traffic which is way more oonvinient than tcpdump ![Screenshot](Network_screenshots/Screenshot-2026-04-18-071851.png)

Analyzing Network Traffic
Now im about to analyze a PCap file provided by Malware-Traffic-analysis.net

- First thing i do is start from a zoomed out overview and then highlight any interesting conversation by using capture file properties  ![Screenshot](Network_screenshots/Screenshot-2026-04-18-082528.png)

- Next thing i did was use conversation to see the most communication between IP addresses ![Screenshot](Network_screenshots/Screenshot-2026-04-18-082726.png)

- Next is to check the protocol Heirachy to see the most common protocols to see any low hanging fruit ![Screenshot](Network_screenshots/Screenshot-2026-04-18-082941.png)

- What i did next was to set up additional collums to show the source and destination ports which are not enabled by default. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-083240.png)

- Next is to filter to show only http traffic. which i then analize and notice that the host value in the http header is an IP address and not a doamin name and also its making a request to a DAT file which is suspecious ![Screenshot](Network_screenshots/Screenshot-2026-04-18-083750.png)

- I then opened the http stream and rebuilt the conversation and noticed that the user agent has a curl utility which i suspicoius. i also notice that is has an MZ file signature. This makes me come to the conclusion that the victim downloaded an executable file from a web server using the curl utility ![Screenshot](Network_screenshots/Screenshot-2026-04-18-084253.png)

- After finding the name of the file, IP address and url associated with it, i then extracted the file itself to gather its hashes 

- Next is to extraxt the hash of the file using sha256sum. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-085018.png)

- Now what i did was to chech the file reputaion using a tool called virus total and malware bazare which then flagged it as malicious and informed me of the malware family it belobgs to called Quakbot ![Screenshot](Network_screenshots/Screenshot-2026-04-18-085304.png)  ![Screenshot](Network_screenshots/Screenshot-2026-04-18-085604.png)

- Next is to do research into quackbot and see if i can discover any post compromise IOC's or tactics i can identify in my PCAP file 

- What i learnt about quackbot is tthat it has been observed peforming an ARP scan in order to discover other endpoints on the network. within IP and TCP traffic i saw a lot of ARP traffic. I am now going to check my PCAP for any indicators of an attacker performing an ARP scan ![Screenshot](Network_screenshots/Screenshot-2026-04-18-090647.png)

- Then i noticed that under the info collum the IP addresses were descending which is a clear indication of an ARP scan. This tells me that the attacker was attempting to find any active machines within the network to probe 

- Next i used the icmp filter to see any icmp echo request or replies when then told me that the hacker was able to make 2 replies and get 2 request ![Screenshot](Network_screenshots/Screenshot-2026-04-18-091234.png)

- Next is to chech if the attacker peformed any port scans ![Screenshot](Network_screenshots/Screenshot-2026-04-18-091320.png)

- when  i was looking at the protocol hierachy i also noticed smpt traffic. i then checked to see if i can uncover anything interesting. After using smpt filter i noticed an authentication login packet. which i then folowed thee tcp stream ![Screenshot](Network_screenshots/Screenshot-2026-04-18-091754.png)

- In the TCP stream i copied the entire conversation of  base64 data for the authentication login and pasted it in cyberchef to decrpyt. what i discovered was the email address and the password the attacker tried to use But the authentication failed howerber i identified some compromised user credentiials. ![Screenshot](Network_screenshots/Screenshot-2026-04-18-092314.png)

- next thing i did was export the SMB files. i then noticed some suspicious dlls files which i saved to a specified folder which i then run an SHA256 on them which showed the all had the same hash similar to the initial quackbot dll file ![Screenshot](Network_screenshots/Screenshot-2026-04-18-093059.png) ![Screenshot](Network_screenshots/Screenshot-2026-04-18-093907.png)

   
