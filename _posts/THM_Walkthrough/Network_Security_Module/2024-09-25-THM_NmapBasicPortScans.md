---
title: TryHackMe - Nmap Basic Port Scans
description: A walkthrough of TryHackMe's room "Basic Port Scans".
categories: [Walkthrough THM, Network Security Module]
tags: [Nmap, Tools]
pin: false
---

Hey there! 
Welcome to this walkthrough of the room "Nmap Basic Port Scans"!

This is the second room in the Nmap series on TryHackMe.

## Task 1: Introduction

In the previous room, [Live Host Discovery]({% post_url THM_Walkthrough/Network_Security_Module/2024-09-24-THM_LiveHostDiscovery%}), the main focus was on finding the online systems. This room will focus on the next step, when we want to check the ports on a live target.

This room will cover:
1. TCP connect port scan
2. TCP SYN port scan
3. UDP port scan

## Task 2: TCP and UDP ports
Just as an IP address identifies a specific device on a network, a TCP or UDP port identifies a specific service running on that device. Think of the IP address as the street address of a hotel, and the port as the room number within that hotel. The server, which is like the hotel, offers various services (or rooms) through these ports. Each service follows a particular set of rules, known as a network protocol. Some ports are associated with a specific protocol, an HTTP server would try to bind to TCP port 80 by default and if the server supports SSL/TLS it would listen to port 443. Only one service can listen to a port on a host. 

Nmap uses the following six states when it comes to classifying the port state: 

1. **Open:** Indicates that a service is listening on the specified port.
2. **Closed:** Indicates that no service is listening on the specified port, although the port is accessible. By accessible, we mean that it is reachable and is not blocked by a firewall or other security appliances/programs.
3. **Filtered:** Means that Nmap cannot determine if the port is open or closed because the port is not accessible. This state is usually due to a firewall preventing Nmap from reaching that port. Nmap’s packets may be blocked from reaching the port; alternatively, the responses are blocked from reaching Nmap’s host.
4. **Unfiltered:** Means that Nmap cannot determine if the port is open or closed, although the port is accessible. This state is encountered when using an ACK scan `-sA`.
5. **Open\|Filtered:** This means that Nmap cannot determine whether the port is open or filtered.
6. **Closed\|Filtered:** This means that Nmap cannot decide whether a port is closed or filtered.

---

**Q: Which service uses UDP port 53 by default?**
> Use your google skills for the first two questions if yu don't know by heart.
{: .prompt-tip}
<details><summary>Answer</summary>
DNS
</details>

---

**Q: Which service uses TCP port 22 by default?**
<details><summary>Answer</summary>
SSH
</details>

---
**Q: How many port states does Nmap consider?**
<details><summary>Answer</summary>
6
</details>

---

**Q: Which port state is the most interesting to discover as a pentester?**
<details><summary>Answer</summary>
open
</details>

## Task 3: TCP Flags
Nmap supports different types of TCP port scans. It is then reasonable to take a look at the TCP header in order to really understand the difference between them.
![alt text](/assets/images/TCP_Header.png){: h="600" w="600" center }
_Tcp header__

We will focus on the flags on the third row of the header. These are:

1. **URG:** The Urgent flag. As the name hints, this flag indicates that the incoming data is urgent. Any TCP segment with the URG flag set will be processed immediately regardless of its sequence number.
2. **ACK:** The Acknowledge flag is used to acknowledge a TCP segment.
3. **PSH:** This flag is used to indicate that the data should be delivered to the receiving application immediately, without waiting for more data to arrive.
4. **RST:** Reset flag is used to reset the connection.
5. **SYN:** The SYN flag is used to initiate the three-way-handshake. 
6. **FIN:** Sender has no more data to send.

---
**Q: What three letters represent the Reset flag?**
<details><summary>Answer</summary>
RST
</details>

---

**Q: Which flag needs to be set when you initiate a TCP connection (first packet of TCP 3-way handshake)?**
<details><summary>Answer</summary>
SYN
</details>

## TCP Connect Scan
The *TCP connect scan* (`-sT`) works by completing the three-way handshake. Nmap sends a TCP SYN packet and waits for a response. If a SYN/ACK is received, it indicates that the port is open. Nmap then sends an ACK to complete the handshake and immediately terminates the connection by sending a RST (reset) packet.

This is the only scan option when nmap is not run as by a privileged user. 

You can use `-F` to enable fast mode and reduce the number of scanned ports to the 100 most common ports. You can also use the `-r` flag to scan the ports in order, instead of randomly.

**Q: Launch the VM. Open the AttackBox and execute `nmap -sT 10.10.61.180` via the terminal. A new service has been installed on this VM since our last scan. Which port number was closed in the scan above (see THM for image) but is now open on this target VM?**
> Hint: The ports are displayed in order... 
{: .prompt-tip}

<details><summary>Answer</summary>
110 
</details>

---

**Q: What is Nmap’s guess about the newly installed service?**
<details><summary>Answer</summary>
POP3
</details>

## TCP SYN scan
When running nmap as a privileged user, the default scan mode is the SYN scan (`-sS`). The SYN scan does not have to complete the TCP three-way-handshake, it tears down the connection once it receives a response from the server. This reduces the chances of the scan being logged. 

---

**Q: Launch the VM. Some new server software has been installed since the last time we scanned it. On the AttackBox, use the terminal to execute `nmap -sS 10.10.142.62`. What is the new open port?**
<details><summary>Answer</summary>
6667
</details>

---

**Q: What is Nmap’s guess of the service name?**
<details><summary>Answer</summary>
IRC
</details>

## Task 6: UDP Scan
The UDP scan (`-sU`) does not attempt any handshake due to the nature of how UDP works. Since there is no guarantee that a service listening on a UDP port would respond to packets sent, nmap turns to the ICMP protocol for help. If a UDP packet is sent to a closed port, an ICMP "port unreachable" error is returned, indicating that the port is closed. T However, if there is no response, it is impossible to determine whether the port is open or filtered, so Nmap sets the port state to `open|filtered`.

---

**Q: Launch the VM. On the AttackBox, use the terminal to execute nmap -sU -F -v 10.10.109.145. A new service has been installed since the last scan. What is the UDP port that is now open?**
<details><summary>Answer</summary>
53
</details>

---

**Q: What is the service name according to Nmap?**
<details><summary>Answer</summary>
domain
</details>


## Task 7: Fine-Tuning Scope and Performance
As we have seen before, you can specify ports using lists and ranges like this:
* port list: `-p22,80, 443`
* port range: `-p1-1023` will scan all ports from 1 to 1023.

A few new tips include:
* `-p-` to scan all 65535 ports.
* `-F` to scan only the 100 most common ports.
* `--top-ports 10` to scan the ten most common ports (update the 10 to any number)

You can also control the scan timing using the `T<0-5>`. Where the number chosen indicates the speed of the scan, 0 is the slowest and 5 is the fastest.

Nmap names these as follows:
* paranoid (0)
* sneaky (1)
* polite (2)
* normal (3)
* aggressive (4)
* insane (5)

To avoid triggering IDS (Intrusion Detection System) `-T0` or `-T1` would be advised. This will however take a long time, so in cases where that is not an issue (like in the CTFs) you might want to go for `-T4`. Note that using the most aggressive speed might affect the accuracy of the scan results.

Another option would be to control the packet rate using `--min-rate <number>` and `--max-rate <number>`. For example `--max-rate 10` which ensures that your scanner is not sending more than ten packets per second.

Additionally, you can manage the parallelization of probes using the `--min-parallelism <numprobes>` and `--max-parallelism <numprobes>` options. Nmap sends probes (a series of packets) to targets to determine which hosts are active and which ports are open. Parallelization in this context refers to the number of probes that Nmap can execute simultaneously. For example, setting `--min-parallelism=512` ensures that Nmap maintains at least 512 probes running in parallel, which are used for both host discovery and checking open ports.

---

**Q: What is the option to scan all the TCP ports between 5000 and 5500?**
<details><summary>Answer</summary>
-p5000-5500
</details>

---

**Q: How can you ensure that Nmap will run at least 64 probes in parallel?**
<details><summary>Answer</summary>
--min-parallelism=64
</details>

---

**Q: What option would you add to make Nmap very slow and paranoid?**
<details><summary>Answer</summary>
-T0
</details>

## Task 8: Summary

There were three basic types of scans covered in this room.

* TCP Connect Scan -> `nmap -sT MACHINE_IP`
* TCP SYN Scan -> `sudo nmap -sS MACHINE_IP`
* UDP Scan -> `sudo nmap -sU MACHINE_IP`

A few port, time and probe settings were also covered.

Happy learning and see you in the next post!