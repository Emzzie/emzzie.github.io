---
title: TryHackMe - Nmap Post Port Scans
description: A walkthrough of TryHackMe's room "Nmap Post Port Scans".
categories: [Walkthrough THM, Network Security Module]
tags: [Nmap, Tools]
pin: false
---

## Task 1: Introduction
This is the last room in the Nmap series on TryHackMe. This room will focus more on the steps after port-scanning, like which service or OS is running, running nmap scripts and saving scan results.

## Task 2: Service Detection
So far, we have only focused on finding live hosts and open ports on them. Another piece in the puzzle is to find out which service is running on a port.

If the `-sV` flag is set, Nmap will collect and determine service and version information for the open ports. You can control the intensity with `--version-intensity LEVEL`. Intensity in this case refers to how many packets are sent to find the service and version running. The intensity levels vary from 0 to 9, where 0 is the least intrusive and might miss some details, while 9 is the most intrusive but also very thorough.

It is worth mentioning that -sV will force Nmap to fulfill the three-way-handshake and establish a connection. So the stealth SYN scan will not be possible when using the `-sV` flag. Furthermore, when adding the `-sV` flag, you get another column `VERSION` available, any version mentioned here is retrieved from the service and not a guess. The `SERVICE` column, however, is a best guess.

---

**Q: Start the target machine for this task and launch the AttackBox. Run `nmap -sV --version-light 10.10.191.164`via the AttackBox. What is the detected version for port 143?**
<details><summary>Answer</summary>
Dovecot imapd
</details>

---

**Q: Which service did not have a version detected with `--version-light`?**
<details><summary>Answer</summary>
Rpcbind.
</details>

## Task 3: OS Detection and Traceroute

### OS Detection
Nmap is not limited to finding versions, it can also detect the Operating System running on the host using the `-O`. The OS detection is not always reliable and there are many things that affect its accuracy, for example Nmap needs to find both one open and one closed port to make a reliable guess.

### Traceroute
By adding the option `--traceroute` Nmap will attempt to find the routers between you and the target. In contrast to the `traceroute` command found on linux which increases the TTL of the packets, the Nmap version starts with packets of high TTL and decreases it.

---
start the machine from task 2 and answer the following question.

**Q: Run nmap with -O option against MACHINE_IP. What OS did Nmap detect?**
<details><summary>Answer</summary>
Linux
</details>

---

## Task 4: Nmap Scripting Engine (NSE)
You can extend the capabilities of Nmap beyond basic network scanning by utilizing Nmap scripts, which are part of the Nmap Scripting Engine (NSE). These scripts allow you to automate a variety of networking tasks such as HTTP Enumeration, DNS Enumeration, FTP Misconfiguration checks etc. By using the `-sC` flag you run a set of default scripts. THe main scripts included in the default set are:
* **Banner Grabbing:** Retrieves service banners to identify software versions.
* **DNS Service Discovery:** Identifies DNS servers and their configurations.
* **HTTP Enumeration:** Enumerates directories and files on web servers.
* **SSL/TLS Information:** Gathers information about SSL/TLS configurations.
* **SMB Security Mode:** Checks the security mode of SMB servers.
* **FTP Anonymous Login:** Tests for anonymous FTP login.
* **NTP Monlist:** Checks for NTP monlist vulnerabilities.
* **SNMP Information:** Gathers information from SNMP services.
* **SSH Host Key:** Retrieves SSH host keys

You can also specify a script via the `--script "SCRIPT_NAME"` option, you can find all the included nmap scripts at `/usr/share/nmap/scripts`. 

I think it is worth mentioning that using a `-sC` scan is most likely going to trigger firewalls and IDS systems alerts due to the significant network traffic generated. 

**Q: Knowing that Nmap scripts are saved in `/usr/share/nmap/scripts` on the AttackBox. What does the script `http-robots.txt` check for?**
<details><summary>Answer</summary>
disallowed entries, note that the ending of the script might be .nse, show the script via less or cat, or open in your texteditor to read the description.
</details>

---

**Q: Can you figure out the name for the script that checks for the remote code execution vulnerability MS15-034 (CVE2015-1635)?**
> Hint: you can search for names with `find -iname SEARCH_QUERY`. Use * as wildcard arount the word you want to find. `iname` makes the search non-case sensitive.
{: .prompt-info}
<details><summary>Answer</summary>
http-vuln-cve2015-1635
</details>

---

**Q: Launch the AttackBox if you haven't already. After you ensure you have terminated the VM from Task 2, start the target machine for this task. On the AttackBox, run Nmap with the default scripts -sC against 10.10.35.196. You will notice that there is a service listening on port 53. What is its full version value?**
<details><summary>Answer</summary>
9.18.28-1~deb12u2-Debian
</details>

---

**Q: Based on its description, the script ssh2-enum-algos “reports the number of algorithms (for encryption, compression, etc.) that the target SSH2 server offers.” What is the name of the server host key algorithm that relies on SHA2-512 and is supported by 10.10.35.196?**
<details><summary>Answer</summary>
rsa-sha2-512, look for server_host_key_algorithms and see if you can find sha2-512 anywhere there.
</details>

## Task 5: Saving the output
There are a few ways to save the output of a file. The three main formats are 
1. **Normal:**  `-oN FILENAME`, saves the scan so that it looks like it does in the terminal when you run the scan.
2. **Greppable:** `-oG FILENAME`, saves the scan in a way that you will get sufficient information when running `grep` on specific keywords or terms.
3. **XML**: `-oX FILENAME`, saves the scan in a way that it is convenient for other programs to process the output.

---

Terminate the target machine of the previous task and start the target machine for this task. On the AttackBox terminal, issue the command scp pentester@MACHINE_IP:/home/pentester/* . to download the Nmap reports in normal and grepable formats from the target virtual machine.

Note that the username pentester has the password THM17577

**Q: Check the attached Nmap logs. How many systems are listening on the HTTPS port?**
<details><summary>Answer</summary>
3
</details>

---

**Q: What is the IP address of the system listening on port 8089?**
<details><summary>Answer</summary>
172.17.20.147
</details>

---


## Task 6: Summary
This concludes this Nmap series, you have learned to scan for live hosts, find open ports in both stealthy ways and not so stealthy ways. You have then learned to utilize the power of Nmap with scripts and version detection and finally you have discovered the way to save these results in a good manner. Great job!

There are many cheat sheets out there, have a look around and find one you like, it is always nice to have a cheat sheet to freshen up the memory every now and then!
