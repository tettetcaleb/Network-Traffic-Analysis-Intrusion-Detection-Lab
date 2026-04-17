# Network-Traffic-Analysis-Intrusion-Detection-Lab

tcpdump

Basics of tcpdump

- First is to list all available network interfaces ![Screenshot](Network_screenshots/Screenshot-2026-04-15-210201.png)

- Next is to monitor one of the network interfaces. in this case i choose to monitor the traffic on my local loop back interface.![Screenshot](Network_screenshots/Screenshot-2026-04-15-210521.png)

- Next is me stimulating real traffic and showing the 3 way handshake in real time ![Screenshot](Network_screenshots/Screenshot-2026-04-15-211243.png)

Capturing network traffic using tcpdump
  
 -selected an interface and displayed its traffic using tcpdump ![Screenshot](Network_screenshots/Screenshot-2026-04-15-214518.png)

 -Filternig and capturing network through a specific range of IP Adresses ![Screenshot](Network_screenshots/Screenshot-2026-04-17-064300.png)

 Analyzing Network Traffic
 
 - What Im going to do right now is analize a PCAP file that has been infected with LockBit![Screenshot](Network_screenshots/Screenshot-2026-04-17-075550.png)
 - 
 - In this PCAP file i noticed there was alot of traffic between Ip addressews on port 80. I then filtered to show only port 80 traffic. ![Screenshot](Network_screenshots/Screenshot-2026-04-17-080708.png)

 - Since we are only dealing with HTTP I also searched for any get or post request using the grep command![Screenshot](Network_screenshots/Screenshot-2026-04-17-081228.png)

 - After filtering i noticed that the endpoint downloaded an executable file called .audiologg.exe
   
 - I then looked up this file and saw that it was associated with microsoft. But to be sure  looked up its IP address. what i noticed was that it was based in vietnam with no association to microsoft![Screenshot](Network_screenshots/Screenshot-2026-04-17-082115.png)

 - I then performed another grep filtering to show any packet that contains .audiologg.exe file.
 
