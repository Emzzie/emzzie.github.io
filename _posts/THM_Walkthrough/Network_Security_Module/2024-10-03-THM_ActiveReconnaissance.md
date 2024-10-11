---
title: TryHackMe - Active Reconnaissance
description: A walkthrough of TryHackMe's room "Active Reconnaissance".
categories: [Walkthrough THM, Network Security Module]
tags: [Basics]
pin: false
---

Lab access: [https://tryhackme.com/r/room/activerecon](https://tryhackme.com/r/room/activerecon)

## Task 1: Introduction
So far in this module we have only looked at the passive reconnaissance, not it's time to look at active reconnaissance and the tools that can be used to gather information, such as the web browser, `ping`, `traceroute`, telnet and `nc`.

In passive reconnaissance we don't interact with the target at all, only gather publicly available information about it. In active reconnaissance we actually interact with the target in one way or another to gather information. It is worth noting that active reconnaissance can be illegal, so only perform this if you have legal authorization from the client. 

Active reconnaissance utilizes some kind of direct connection to the machine, which might leave a digital trace in the form of client IP address and connection details among other things. Not all connections are suspicious however, browsing a web page for example does not seem suspicious, even if the intent of visiting the web page is. 

## Web Browser
The web browser can actually a pretty convenient tool and it is available on all systems. There are several ways you can use a web browser to gather information about a target.

The browser uses TCP on port 80 by default when HTTP is used and on port 443 when HTTPS is used. Since this is by default the web browser does not show them in the address bar. If a local HTTPS server listens on for example 8080, then it is possible to connect to it via the address bar of the web browser using https://127.0.0.1:8080/. 

On a all web browsers allow you to enter "Developer Tools" by pressing `Ctrl+Shift+I`(or simply right click on a web page and find the Inspect option). This will provide you with access to the assets like javascript code, cookies, console and much more. Here is a good place to look for comments in the available code and console warnings among other things.

You can also utilize a few add-ons to the web-browsers such as **FoxyProxy** that allows you to change the proxy server used to access the website, **Wappalyzer** to provide the technologies used on the website (such as languages, servers and databases used) and **user-agent switcher and manager** to make it look like you are accessing the webpage from another operating system.


---

**Q: Browse to the [following website](https://static-labs.tryhackme.cloud/sites/networking-tcp/) and ensure that you have opened your Developer Tools on AttackBox Firefox, or the browser on your computer. Using the Developer Tools, figure out the total number of questions.**
<details><summary>Answer</summary>
8
</details>

---

## Task 3: Ping
Ping is a tool used to send packets to a remote system which will return a reply if it is online. It sends a ICMP Echo request to the remote system which replies with a ICMP Echo reply. For more in-depth on PING have a look at [ICMP The Two Edged Sword]({% post_url Basics/2024-09-23-ICMP_The_Two_Edged_Sword %}).

The general syntax is `ping MACHINE_IP` or `ping HOST_NAME`. This will keep sending ping packets until manually cancelled using `Ctrl+C`. It could be a bit more convenient to use `ping -c 10 MACHINE_IP` to specify that you only want to send 10 packets and then exit out of it automatically.

An example of how it would look with only five sent requests is as follows:

```terminal
user@AttackBox$ ping -c 5 MACHINE_IP
PING MACHINE_IP (MACHINE_IP) 56(84) bytes of data.
64 bytes from MACHINE_IP: icmp_seq=1 ttl=64 time=0.636 ms
64 bytes from MACHINE_IP: icmp_seq=2 ttl=64 time=0.483 ms
64 bytes from MACHINE_IP: icmp_seq=3 ttl=64 time=0.396 ms
64 bytes from MACHINE_IP: icmp_seq=4 ttl=64 time=0.416 ms
64 bytes from MACHINE_IP: icmp_seq=5 ttl=64 time=0.445 ms

--- MACHINE_IP ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4097ms
rtt min/avg/max/mdev = 0.396/0.475/0.636/0.086 ms
```

This response suggest that the target host is online since we have 0% packet loss. There is, however, a possibility that you get the following result instead:

```terminal
user@AttackBox$ ping -c 5 MACHINE_IP
PING MACHINE_IP (MACHINE_IP) 56(84) bytes of data.
From ATTACKBOX_IP icmp_seq=1 Destination Host Unreachable
From ATTACKBOX_IP icmp_seq=2 Destination Host Unreachable
From ATTACKBOX_IP icmp_seq=3 Destination Host Unreachable
From ATTACKBOX_IP icmp_seq=4 Destination Host Unreachable
From ATTACKBOX_IP icmp_seq=5 Destination Host Unreachable

--- MACHINE_IP ping statistics ---
5 packets transmitted, 0 received, +5 errors, 100% packet loss, time 4098ms
pipe 4
```

This does not automatically mean that the host is offline, it could also mean that a firewall is configured to block ping packets (for example MS Windows firewall blocks ping by default), that the destination computer is not connected to the network etc. Anyhow, this gives you some information that can be useful in further reconnaissance.

**Q: Which option would you use to set the size of the data carried by the ICMP echo request?**  
**Hint:** have a look in the man-pages for `ping`
<details><summary>Answer</summary>
-s
</details>

---

**Q: What is the size of the ICMP header in bytes?**  
**Hint:** have a look in the man-pages for `ping`
<details><summary>Answer</summary>
8
</details>

---

**Q: Does MS Windows Firewall block ping by default? (Y/N)**
<details><summary>Answer</summary>
Y
</details>

---

**Q: Deploy the VM for this task and using the AttackBox terminal, issue the command **ping -c 10** MACHINE_IP. How many ping replies did you get back?**
<details><summary>Answer</summary>
10
</details>

## Task 4: Traceroute
Traceroute is another tool using the ICMP protocol (traceroute is also covered in [ICMP The Two Edged Sword]({% post_url Basics/2024-09-23-ICMP_The_Two_Edged_Sword %})) and is used to trace the route taken by packets from your system to another host. The main syntax is `traceroute MACHINE_IP`. It utilizes that the IP header has a a "time to live"-header (TTL). For each router the packet passes the TTL will be decreased by 1 until it either reaches its target or TTL is 0. When TTL is 0 the router that the packet currently resided in will return a ICMP Time Exceeded message. Traceroute increases the TTL for each packet that is sent until it reaches it's destination. It could look something like this:

    Your Computer ----> [TTL=1] ----> Router 1
    Your Computer <---- [Time Exceeded] <---- Router 1

    Your Computer ----> [TTL=2] ----> Router 1 ----> Router 2
    Your Computer <---- [Time Exceeded] <---- Router 2

    Your Computer ----> [TTL=3] ----> Router 1 ----> Router 2 ----> Router 3
    Your Computer <---- [Time Exceeded] <---- Router 3

The presence of multiple IP addresses in a traceroute output indicates different paths, interfaces, or nodes at a particular hop responding to the traceroute probes.
If we don't get an expected time exceeded message within the right time frame, we get a * to indicate that. 

---

To answer these questions you need to have a look at the two traces in the lab.

**Q: In Traceroute A, what is the IP address of the last router/hop before reaching tryhackme.com?**
<details><summary>Answer</summary>
172.67.69.208
</details>

---

**Q: In Traceroute B, what is the IP address of the last router/hop before reaching tryhackme.com?**
<details><summary>Answer</summary>
104.26.11.229
</details>

---

**Q: In Traceroute B, how many routers are between the two systems?**
<details><summary>Answer</summary>
26, each line represents one router.
</details>

---

**Q: Start the attached VM from Task 3 if it is not already started. On the AttackBox, run traceroute MACHINE_IP. Check how many routers/hops are there between the AttackBox and the target VM.**
<details><summary>Answer</summary>
no answer needed.
</details>

## Task 5: Telnet
The TELNET (Teletype Network), developed in 1969 allows communication via a command-line interface and uses port 23 by default. Since TELNET sends all data in clear text it is not considered to be secure. A secure alternative would be SSH (Secure SHell).

Despite the lack of security, TELNET can be useful for testing and banner grabbing. You can use `telnet MACHINE_IP` to connect to a service running on TCP and send simple requests like `GET / HTTP/1.1` to retrieve the default page. You do, however, need to input a value for the host like `host: example` after the requests to get a valid response.

For example:
```terminal
Pentester Terminal
pentester@TryHackMe$ telnet MACHINE_IP 80
Trying MACHINE_IP...
Connected to MACHINE_IP.
Escape character is '^]'.
GET / HTTP/1.1
host: telnet

HTTP/1.1 200 OK
Server: nginx/1.6.2
Date: Tue, 17 Aug 2021 11:13:25 GMT
Content-Type: text/html
Content-Length: 867
Last-Modified: Tue, 17 Aug 2021 11:12:16 GMT
Connection: keep-alive
ETag: "611b9990-363"
Accept-Ranges: bytes



...

```
Note how we can see the installed web server and it's version! These could be of use when looking for vulnerabilities.

---

**Q: Start the attached VM from Task 3 if it is not already started. On the AttackBox, open the terminal and use the telnet client to connect to the VM on port 80. What is the name of the running server?**
<details><summary>Answer</summary>
Apache
</details>

---

**Q: What is the version of the running server (on port 80 of the VM)?**
<details><summary>Answer</summary>
2.4.61
</details>

## Task 6: Netcat
Netcat is a versatile command-line utility used for network communications. It is sometimes referred to the "Swiss Army knife of networking" as it can create almost any kind of connection. Its features vary from port scanning and banner grabbing to file transfer and establishing backdoors. The room ["What the Shell?"](https://tryhackme.com/r/room/introtoshells) covers a little more on netcat.

In this case, we will focus more on using netcat to connect to a server, for which we will use `nc MACHINE_IP PORT_NUMBER` once connected, you can enter similar requests as for TELNET, `GET / HTTP/1.1` with `host: netcat` and you will receive a similar output.  

Note that there is MUCH more to netcat than this short task covers. Be curious and discover it's full potential! 

---

**Q: Start the VM and open the AttackBox. Once the AttackBox loads, use Netcat to connect to the VM port 21. What is the version of the running server?**
<details><summary>Answer</summary>
0.17, use the GET command!
</details>

## Task 7: Putting it all together
In this room we covered traceroute, ping and telnet to perform active reconnaissance, both to check if a target is online and the route taken along with some banner grabbing using telnet and netcat.


