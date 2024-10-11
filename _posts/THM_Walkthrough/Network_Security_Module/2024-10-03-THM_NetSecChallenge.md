---
title: TryHackMe - Net Sec Challenge
description: A walkthrough of TryHackMe's room "Net Sec Challenge".
categories: [Walkthrough THM]
tags: [Nmap, Tools]
pin: false
---

## Introduction
This is a challenge to test the skills acquired in this network security module, all questions can be solved using nmap, telnet, and hydra.

Let's first quickly summarize what we learned in table form:

## Nmap recap
### Live host discovery:

| Scan Type              | Example Command                             | Advantage                                     |
|------------------------|---------------------------------------------|-----------------------------------------------|
| ARP Scan               | `sudo nmap -PR -sn MACHINE_IP/24`           | Quickly discovers hosts on a local network    |
| ICMP Echo Scan         | `sudo nmap -PE -sn MACHINE_IP/24`           | Identifies live hosts using ICMP requests     |
| ICMP Timestamp Scan    | `sudo nmap -PP -sn MACHINE_IP/24`           | Can reveal host uptime information             |
| ICMP Address Mask Scan | `sudo nmap -PM -sn MACHINE_IP/24`           | Helps identify subnet structure                |
| TCP SYN Ping Scan      | `sudo nmap -PS22,80,443 -sn MACHINE_IP/30`  | Efficiently checks for open TCP ports         |
| TCP ACK Ping Scan      | `sudo nmap -PA22,80,443 -sn MACHINE_IP/30`  | Useful for mapping firewall rules              |
| UDP Ping Scan          | `sudo nmap -PU53,161,162 -sn MACHINE_IP/30` | Identifies open UDP ports                      |


### Basic port scan

| Port Scan Type   | Example Command           | Advantage                                      |
|------------------|---------------------------|------------------------------------------------|
| TCP Connect Scan | `nmap -sT MACHINE_IP`     | Reliable method for identifying open ports     |
| TCP SYN Scan     | `sudo nmap -sS MACHINE_IP` | Stealthy and less likely to be logged by firewalls |
| UDP Scan         | `sudo nmap -sU MACHINE_IP` | Identifies open UDP ports, often overlooked    |

### Advanced port scan

| Scan Type                | Example Command                                      | Advantage                                      |
|--------------------------|------------------------------------------------------|------------------------------------------------|
| TCP Null Scan            | `sudo nmap -sN MACHINE_IP`                           | Bypasses firewalls that only block SYN packets |
| TCP FIN Scan             | `sudo nmap -sF MACHINE_IP`                           | Useful for bypassing certain packet filters    |
| TCP Xmas Scan            | `sudo nmap -sX MACHINE_IP`                           | Identifies open ports using unusual flags      |
| TCP Maimon Scan          | `sudo nmap -sM MACHINE_IP`                           | Evades firewalls that block SYN/ACK packets    |
| TCP ACK Scan             | `sudo nmap -sA MACHINE_IP`                           | Maps firewall rules and identifies stateful filters |
| TCP Window Scan          | `sudo nmap -sW MACHINE_IP`                           | Determines open ports based on window size     |
| Custom TCP Scan          | `sudo nmap --scanflags URGACKPSHRSTSYNFIN MACHINE_IP`| Highly customizable to evade detection         |
| Spoofed Source IP        | `sudo nmap -S SPOOFED_IP MACHINE_IP`                 | Hides the attacker's real IP address           |
| Spoofed MAC Address      | `--spoof-mac SPOOFED_MAC`                            | Avoids detection by imitating a trusted device |
| Decoy Scan               | `nmap -D DECOY_IP,ME MACHINE_IP`                     | Hides the real source of the scan              |
| Idle (Zombie) Scan       | `sudo nmap -sI ZOMBIE_IP MACHINE_IP`                 | Avoids detection and attribution of the scan   |
| Fragment IP data into 8 bytes | `-f`                                           | Bypasses some firewalls by fragmenting packets |
| Fragment IP data into 16 bytes| `-ff`                                          | Further fragmentation to evade detection       |

### Options

| Option               | Purpose                                | Advantage                                     |
|----------------------|----------------------------------------|-----------------------------------------------|
| -n                   | No DNS lookup                          | Speeds up the scan by skipping DNS resolution |
| -R                   | Reverse-DNS lookup for all hosts      | Provides hostnames for better context         |
| -sn                  | Host discovery only                    | Focuses on live hosts without port scanning   |
| -p-                  | All ports                              | Comprehensive scan of all available ports     |
| -p1-1023             | Scan ports 1 to 1023                   | Targets commonly used ports for quicker scans |
| -F                   | 100 most common ports                  | Faster assessment of commonly used services    |
| -r                   | Scan ports in consecutive order        | Provides more predictable scan results        |
| -T<0-5>             | `-T0` being the slowest and `-T5` the fastest | Allows control over scan speed                 |
| --max-rate 50        | Rate <= 50 packets/sec                 | Prevents overwhelming the network              |
| --min-rate 15        | Rate >= 15 packets/sec                 | Ensures a minimum pace of packet sending      |
| --min-parallelism 100| At least 100 probes in parallel        | Maximizes speed for large scans                |

## Hydra recap

| Option               | Explanation                                               |
|----------------------|----------------------------------------------------------|
| -l username          | Provide the login name                                   |
| -P WordList.txt     | Specify the password list to use                         |
| server service       | Set the server address and service to attack             |
| -s PORT              | Use in case of non-default service port number           |
| -V or -vV           | Show the username and password combinations being tried   |
| -d                   | Display debugging output if the verbose output is not helping |

## Challenge questions

### Question 1
**What is the highest port number being open less than 10,000?**

For port scanning we use nmap. Looking at the options we have available and the fact that we are interested in the port number less than 10 000 we can either take a chance, and run

`sudo nmap MACHINE_IP` and hope that scanning the top 1000 will be enough or play it safe and run
`sudo nmap -p1-10000 MACHINE_IP`.

Either way, you should receive the following, which gives you the port number. 

Note the `sudo`, remember that the default scan type when using sudo is syn-stealth scan, which is much faster than the non-sudo default TCP-connect scan.

![alt text](/assets/images/NetSecChallenge_q1.png)

### Question 2

**There is an open port outside the common 1000 ports; it is above 10,000. What is it?**

Once again we will consult nmap. 
You could simply have run 

`nmap -p- MACHINE_IP` in the first question, and you can still do so now.

My approach was to scan the ports above 10000. We know that there are 65,535 available and scanning 10000 less is pretty nice. Further more we know that there is __*one*__ port outside the common ports. So I tried

`nmap -p 10000-65535 MACHINE_IP -v`, let's break this down:

`-p 10000-65535` will specify to scan ports between 10000 and 65535  
`-v` will display the ports.

This will scan all ports above 100000, as soon as you see a port you can quit the scan manually. 

### Question 3
**How many TCP ports are open**

Here you can simply count the number of open ports found in question 1 and add the port above 10,000 to that and you will get the correct number of open ports! Hint: There are only TCP ports open...

### Question 4
**What is the flag hidden in the HTTP server header?**

Now we have moved away from port scanning, we have found a few open ports. 
We now know that HTTP has the default port of 80, we have also learned that we can use telnet to connect to it.

`telnet MACHINE_IP 80`

We also learned that we can send basic GET requests by using `GET / HTTP/1.1` and `host: telnet`

It will look something like this:
```terminal
$ telnet MACHINE_IP 80
Trying MACHINE_IP...
Connected to MACHINE_IP.
Escape character is '^]'.
GET /HTTP/1.1
host: telnet

HTTP/1.0 400 Bad Request
Content-Type: text/html
Content-Length: 345
Connection: close
Date: Fri, 04 Oct 2024 12:37:47 GMT
Server: lighttpd THM{XXXXXXXXXXX}
```

### Question 5
**What is the flag hidden in the SSH server header?**

We can use telnet here as well and instead of using the port for HTTP we use the port for SSH giving `telnet MACHINE_IP 22` which will result in something like this:

```terminal
$ telnet MACHINE_IP 22
Trying MACHINE_IP...
Connected to MACHINE_IP.
Escape character is '^]'.
SSH-2.0-OpenSSH_8.2p1 THM{XXXXXXXX} 
```

### Question 6
**We have an FTP server listening on a nonstandard port. What is the version of the FTP server?**

Let's first look at all the ports we have found. There is only one ftp server in the non-standard range, and it uses the port from question 2. 
Since we know the port number, we can let nmap scan only that port. 
`nmap -p PORT MACHINE_IP -sV` we use `-sV` to get the versions.

### Question 7
**We learned two usernames using social engineering: `eddie` and `quinn`. What is the flag hidden in one of these two account files and accessible via FTP?**

We now have two names and an open FTP port but no password. Let's see if they were using insecure passwords with _**hydra**_. 

One approach is to run hydra twice once with `eddie` and once with `quinn`. The approach I'm gonna take is to add them to a text file called `users.txt` and run hydra with the users.txt file like this:
`hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ftp://MACHINE_IP:PORT`

This will give the following result:
```shell
$ hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ftp://MACHINE_IP:PORT   
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-10-04 10:17:08
[DATA] max 16 tasks per 1 server, overall 16 tasks, 28688798 login tries (l:2/p:14344399), ~1793050 tries per task
[DATA] attacking ftp://MACHINE_IP:PORT/
[PORT][ftp] host: MACHINE_IP   login: eddie   password: ------
[PORT][ftp] host: MACHINE_IP   login: quinn   password: ------
1 of 1 target successfully completed, 2 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-10-04 10:17:32

```
Now that we have usernames and passwords, lets log in to the FTP server and see which files we can find!

```shell
$ ftp user_name@MACHINE_IP -p PORT 
Connected to MACHINE_IP.
220 (vsFTPd 3.0.5)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||30777|)
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002           18 Sep 20  2021 ftp_flag.txt
226 Directory send OK.

```
use `get ftp_flag.txt` and the file will be downloaded to your current folder.

### Question 8

**Browsing to http://MACHINE_IP:8080 displays a small challenge that will give you a flag once you solve it. What is the flag?**

On the website we are met with the following:
![alt text](/assets/images/netSecCHallenge_q8.png)

So the task is to scan as covertly as possible without being detected. 

I tried a few options with FIN and XMAS scan, including `f`, `-ff` decoys, and a few `-T<>` options, all were 100% detected (or did not generate the flag). 

Only `sudo nmap -sN MACHINE_IP`  gave 0% chance of being detected (depending on what you have done before, make sure to reset the packet count between all scans.) 

## Summary
In this challenge we tested our knowledge about nmap, hydra and getting banners using telnet. Which concludes the Network Security Module!

See you in the next module, over and out!