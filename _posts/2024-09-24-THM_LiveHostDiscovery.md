---
title: TryHackMe - Nmap Live Host Directory
description: A walkthrouhg of TryHackMe's room "Nmap Live Host Discovery".
categories: [Walkthrough THM, Network Security Module]
tags: [Nmap, Tools]
pin: false
---

## Foreword
Hey there! 
Welcome to this walkthrough of the room "Nmap Live Host Discovery"! 

This is the first room in the Nmap series on TryHackMe.

I strongly recommend getting the basics refreshed before starting this room. Unless you feel that you got the the nitty gritty of ARP, ICMP, TCP/UDP and the OSI model straightened out, either have a look at **ADD ROOMS** or quickly have a look at 
* [Another OSI Model Breakdown]({% post_url 2024-09-19-OSI_Model_Basics %}).
* [ARP The Basics]({% post_url 2024-09-20-ARP_The_Basics%}).
* [ICMP - The Two Edged Sword]({% post_url 2024-09-23-ICMP_The_Two_Edged_Sword %}). 

Note that I won't do a copy-paste from TryHackMe. You can read that yourself, I will provide my own take on the information given and guide you to the correct answers in my own words. 

When you feel ready lets go ahead and discover this room!

## Task 1: Introduction
So, we want to target a network. It's a dark hole, all we have is an IP address and possibly a subnet mask... Who (or rather what) comes to the rescue? Yes you guessed it! __Nmap!__ The good ol' friend that we always consult in these situations. The friend that does all the repetitive work for us and just hands us the report we ask for! 

There are two main things that Nmap help with
* Which systems are up and running
* What services are running on these systems?

>Let's get the terminology for system and services straightened out right away:
>* A _system_ typically refers to any device or host that is connected to the network. For example computers, routers, switches and IoT devices.
>* A _service_ refers to a network application or protocol that runs on a specific port of a system. For example HTTP, HTTPS, SMTP, IMAP, FTP and databases such as MySQl. 

This room focuses primarily on finding out which systems are up and running. 

How is this different from the "normal" port scan that we have gotten used to? Provided with a subnet, it is an absolute waste of time to do a port scan if the system (or host) is offline.

This room will present three different approaches Nmap can use to discover live hosts.
1. __ARP scan:__ Uses ARP Requests to find live hosts
2. __ICMP scan:__ Uses ICMP Requests to find live hosts
3. __TCP/UDP ping scan:__ This scan sends packets to TCP and UDP ports to determine live hosts.

## Task 2: Subnetworks
This is a little recap section about subnets. If you want the longer version have a look **HERE**. 

**Key terms:**
* __Network segment:__ A group of computers connected using a shared medium. Like a ethernet switch or WiFi access point. **_A physical connection_**.
* __Subnetwork:__ _usually_ the equivalent of one or more network segments connected together and configured to use the same router. **_A logical connection_**.

>In the example picture given on THM, Network B has the IP address 10.2.0.0/16. Where /16 (which also can be expressed as 255.255.0.0) indicates that 16 out of the 32 bits make out the network part of the IP address. The remaining 16 bits make out the host part of hte IP address. This will generate 2<sup>16</sup>=65 536 unique hosts.
Whereas, Network A has 10.1.100.0/24. Where the /24 (which also can be expressed as 255.255.255.0) indicates that 24 out of the 32 bits make out the network part of the IP address, and the remaining 8 bits make out the host part of the IP address.

Back to Nmap. Sometimes we are given a group of hosts or a subnet, and for reconnaissance we might want to discover more information about them. If we are connected to the same subnet as the one we want to gather information on, we could use ARP queries to discover live hosts (recall that ARP __only__ works within a subnet). Any response to the ARP-request would imply that the host is online. 

Make sure to click the "view site" button to start the simulator to aid answering the questions for tasks 2,4 and 5.

---

"Send a packet with the following:  
From computer1  
To computer1 (to indicate it is broadcast)  
Packet Type: “ARP Request”  
Data: computer6 (because we are asking for computer6 MAC address using ARP Request)  "

---

**Q: How many devices can see the ARP request?**
<details><summary>Answer (hint: The switch is not a device)</summary>
4.
Why? Remember that ARP only works within its own subnet. The router will provide its MAC address as a reply to the ARP request, and not forward any packets.
</details>

---

**Q: Did computer 6 receive the ARP Request? (Y/N)**
<details><summary>Answer</summary>  
N. Why? Again, ARP only works within its own subnet, ARP requests are never forwarded by a router, it is simply seen as a device in the subnet.
</details>

---

"From computer4  
To computer4 (to indicate it is broadcast)  
Packet Type: “ARP Request”  
Data: computer6 (because we are asking for computer6 MAC address using ARP Request")

---

**Q: How Many devices can see teh ARP request?**
<details><summary>Answer (hint: the switch is still not counted as a device)</summary>
4
</details>

---

**Q: Did computer6 reply to the ARP Request? (Y/N)**
<details><summary>Answer</summary> Y, because it is in the same subnet now! </details>

## Task 3: Enumerating Targets

Nmap requires you to specify the targets to scan. You can provide a list, a range or a subnet. Some examples of this syntax are:
* List: `nmap Machine_IP, scanme.nmap.org example.com` will scan all three IP addresses.
* Range: `nmap 10.11.12.15-20` will scan all 6 IP addresses from 10.11.12.15 through 10.11.12.10.
* Subnet: `nmap MACHINE_IP/30` will scan 4 IP addresses (30 out of 32 bits are used for the network part of the IP address, thus there are 2 bits left for hosts, 2²=4).
* Input text file: `nmap -iL list_of_hosts.txt`.

If you want to see the list of hosts that Nmap will scan you can use `nmap -sL TARGETS`. This option will give you a detailed list of the hosts that Nmap will scan without scanning them; however, Nmap will attempt a reverse-DNS resolution on all the targets to obtain their names. Names might reveal various information to the pentester. (If you don’t want Nmap to the DNS server, you can add -n.)

---

**Q: What is the first IP address Namp would scan if you provided `10.10.12.13/29` as your target?**
<details><summary>Answer</summary> 10.10.12.8. THM expect you to use "nmap -sL TARGETS" for this. </details>

> Side note: You can actually find this out without using nmap.   
You know that the first 29 bits are reserved, let's convert this to binary: 00001010.00001010.00001100.00001101. We know that the 29 first bits will remain the same -> 00001010.00001010.00001100.00001XXX. Where the first visited address would be: 00001010.00001010.00001100.00001000

---

**Q: How many IP addresses will Nmap scan if you provide the following range `10.10.0-255.101-125`?** 
<details><summary>Answer</summary>
6400. You can use nmap -sL -n 10.10.0-255.101-125 and the last line will say "Nmap done: 6400 IP addresses (0 hosts up) scanned in ...". You can calculate this if you want, but I won't do that here!
</details>

## Task 4: Discovering Live Hosts
As already mentioned, we will use different protocols to discover live hosts. Lets recap on which layer they operate and what their main purpose is:
* ARP: Operates in the Link Layer with the main purpose to get the MAC address of a specific IP address within the subnet.
* IMCP: Operates in the Network Layer and in this case it is the ping command we are interested in, using the ICMP Echo and ICMP Reply messages. (Note, if you want to ping a system on the same subnet, an ARP query should precede the ICMP Echo)
* TCP/UDP: Operates in the Transport Layer and have the main purpose to transport IP packages between hosts. For network scanning purposes, the scanner can send a special packet to the TCP and UDP ports to check if they will respond. This can be very handy if the ICMP Echo is blocked.

Open the network simulator again (click the view site).

---

Send a packet with the following:

From computer1  
To computer3  
Packet Type: “Ping Request”  

**Q: What is the type of packet that computer 1 sent before the ping?**
<details><summary>Answer</summary> 
ARP Request. Have a look in the network log and you will se that an ARP request is sent. Why is an ARP request sent first? Because you are sending an IP packet, and you need to know the receiving device.
</details>

---

**Q: What is the type of packet that computer1 received before being able to send the ping?**
<details><summary>Answer</summary> 
ARP Response. 
</details>

---

**Q:How many computers responded to the ping request?**
<details><summary>Answer</summary>
1. Since the ping is sent to a specific host, no others are expected to respond.
</details>

---

Send a packet with the following:

From computer2  
To computer5  
Packet Type: “Ping Request”

**Q: What is the name of the first device that responded to the first ARP Request?**
<details><summary>Answer</summary>
Router. Since the target device for the PING is outside of the subnet, computer 1 knows that the router has to handle the rest. 
</details>

---

**Q: What is the name of the first device that responded to the second ARP Request?**
<details><summary>Answer</summary>  
computer5. This request is in a new subnet, and the router needs to send a ARP request to find the MAC address of computer5 in its subnet.
</details>

---

**Q:Send another Ping Request. Did it require new ARP Requests? (Y/N)**
<details><summary>Answer</summary>
N. Since the first ARP response (the MAC/IP pair) was stored in the ARP table at computer1 and the router stored the MAC/IP pair for computer5.
</details>

## Task 5: Nmap Host Discovery Using ARP
I think you got the basics of ARP covered by now, so I won't go into as much detail on that, refer to the TryHackMe text on this task for a recap! 

It could, however, be worth mentioning the approach Namp takes when no host discovery options are provided:
1. When a _privileged_ user (root or a sudoer) tries to scan targets on a _local network_ Nmap uses the _**ARP request**_ . 
2. When a _privileged_ user tries to scan targets _outside the local network_ Nmap uses the ICMP echo requests, TCP ACK to port 80, TCP SYn to port 443 and ICMP timestamp request
3. When an _unprivileged_ user tries to scan targets _outside the local network_, Nmap resorts to a TCP 3-way handshake by sending SYN packets to ports 80 and 443.

Nmap, by default, uses the most appropriate of these ping scans to find live hosts and only proceeds the scan on these live hosts.

You can also use Nmap to discover online hosts without port-scanning the live systems bu using `nmap -sn TARGETS` (  `-sn`: Ping Scan - disable port scan)

---

To specify that you want to perform a Nmap **ARP scan**  you use the flag `-PR`. To only discover live hosts without the port scan you can, as we just learned, use `-sn`. Leading to `nmap -PR -sn Targets`. Recall from task 3 that we can use lists, ranges and subnets as variable to the nmap scan. It would seem reasonable to use ARP-scan on a subnet, right? Run `nmap -PR -sn -MACHINE_IP/24` and you will use ARP scan on the subnet.

> Did you know that there is a scanner built around ARP queries? I'ts called arp-scan and only sends ARP queries.
{: .prompt-tip}

---

From computer1
To computer1 (to indicate it is broadcast)
Packet Type: “ARP Request”
Data: try all the possible eight devices (other than computer1) in the network: computer2, computer3, computer4, computer5, computer6, switch1, switch2, and router.

**Q: How many devices are you able to discover using ARP requests?**
<details><summary>Answer</summary>
3. - It's like a mantra, "ARP only operates on a local network, ARP only operates on a local network, ARP only..."
</details>

## Task 6: Nmap Host Discovery Using ICMP
There are a few messages that are part of the ICMP protocol the ones that are important for this task are:
* Echo Request/Echo Reply
* Timestamp
* Address Mask Request

Echo Request/Reply and Timestamp are shortly covered in [ICMP - The Two Edged Sword]({% post_url 2024-09-23-ICMP_The_Two_Edged_Sword %}).


If you want to use the _ICMP Echo request_ to discover the live hosts you add the `-PE` flag. Note that the ICMP Echo request is sent as an IP packet, and in order to know where to send the IP packets an ARP request is sent in order to know the destination device. So if you scan a local subnet with ICMP Echo request, these packets don't need to be sent since Nmap can confirm that the hosts are up based on the ARP responses.

The ICMP Echo requests are often blocked, therefore ICMP Timestamp or ICMP Address Mask requests can be used instead. To use the ICMP Timestamp request add the `-PP` option and to use the address mask request use `-PM`.

Why do we need to know this many variants of scanning? Sometimes different types of ICMP packets are blocked, so it is important to know different approaches to achieve the same results. 

---

**Q: What is the option required to tell Nmap to use ICMP Timestamp to discover live hosts?**
<details><summary>Answer</summary> 
-PP
</details>

---

**Q: What is the option required to tell Nmap to use ICMP Address Mask to discover live hosts?**

<details><summary>Answer</summary>
-PM
</details>

---

**Q: What is the option required to tell Nmap to use ICMP Echo to discover live hosts?**
<details><summary>Answer</summary>
-PE
</details>

## Task 7: Nmap Host Discovery Using TCP and UDP

### TCP SYN Ping
Let’s first clarify that a TCP SYN scan and a TCP SYN Ping scan are not the same thing. A TCP SYN scan (or Stealth Scan) is used to find open ports on a target, while a TCP SYN Ping scan is used to find live hosts in a set of IP addresses. In general, the TCP SYN scan is used to identify open ports on a live host. The TCP SYN Ping scan only checks specific ports to see if the host is live. If the host responds, there is no need to check other ports for host discovery, and the scan continues to the next IP address (if any).

If we want to use the Nmap TCP SYN Ping scan add hte `-PS` followed by the port number range or a combination of them. For example `-PS21` to target port 21 (add , to add more specific ports) or use `-PS21-25` to scan ports 21 through 25. If nothing is specified, port 80 will be used.

**Privileged users** (root and sudoers) can send TCP SYN packets without completing the TCP three-way-handshake even if the port is open. Unprivileged users have to complete the handshake if the port is open.

### TCP ACK Ping
Sometimes the firewall will block the TCP SYN requests, then you can test sending a packet with the ACK flag sent instead, using the TCP ACK Ping scan. **You must run nmap as a privileged user to accomplish this.**
Here you would use the option `-PA` instead. You can specify the ports in the same ways as for TCP SYN Ping scan above. If nothing is specified, port 80 will be used.

### UDP Ping
By design, UDP "doesn't care" if the packets reach it's destination. However, we might be lucky and receive an ICMP port unreachable packet if the port is closed, which in turn would indicate that the system is up and running! Note that there are firewalls and rate limiting that can prevent this from being sent!

However, to use the UDP Ping scan we use the `-PU` option.

> There is a tool called Masscan that uses a similar approach to discover the available system. 
{: .prompt-tip}

---

**Q: Which TCP ping scan does not require a privileged account?**
<details><summary>Answer</summary>
TCP SYN Ping
</details>

---

**Q:Which TCP ping scan requires a privileged account?**
<details><summary>Answer</summary> 
TCP ACK Ping
</details>

---

**Q:What option do you need to add to Nmap to run a TCP SYN ping scan on the telnet port?**
<details><summary>Answer</summary>
-PS23
</details>

## Task 8: Using Reverse-DNS Lookup
Nmap’s default behavior is to use reverse-DNS online hosts. Because the host names can reveal a lot, this can be a helpful step. However, if you don’t want to send such DNS queries, you use `-n` to skip this step.

By default, Nmap will look up online hosts; however, you can use the option `-R` to query the DNS server even for offline hosts. If you want to use a specific DNS server, you can add the `--dns-servers DNS_SERVER` option.

---

**Q: We want Nmap to issue a reverse DNS lookup for all the possibles hosts on a subnet, hoping to get some insights from the names. What option should we add?**
<details><summary>Answer</summary>
-R
</details>

## Task 9: Summary
Have a look at the table given on THM, maybe even do a little copy-paste of it into your nice-to-have document. It might save you some time later on!

Over and out!