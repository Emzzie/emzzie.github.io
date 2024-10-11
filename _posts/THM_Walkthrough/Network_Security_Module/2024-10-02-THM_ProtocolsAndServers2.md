---
title: TryHackMe - Protocols and Servers 2
description: A walkthrough of TryHackMe's room "Protocols and Servers 2".
categories: [Walkthrough THM, Network Security Module]
tags: [Basics, Protocols]
pin: false
---

Access the lab [here](https://tryhackme.com/r/room/protocolsandservers2)

## Task 1: Introduction
In the previous room, ["Protocols and Servers"]({% post_url THM_Walkthrough/Network_Security_Module/2024-09-27-THM_ProtocolsAndServers %}) we covered the basics of the following protocols:

* Telnet
* HTTP
* FTP
* SMTP
* POP3
* IMAP

We discussed the fact that all of these protocols transmit data in clear text. This is inherently a vulnerability, making them susceptible to various types of attacks, including:
* Sniffing Attack (Network Packet Capture)
* Man-in-the-Middle (MITM) Attack
* Password Attack (Authentication Attack)

Additionally, like all software, implementations of these protocols may have bugs or other vulnerabilities that can be exploited.

### The CIA Security Triad
Remember the cornerstones of security, the CIA triad? **C**onfidentiality, **I**ntegrity, **A**vailability. Lets quickly recap each one:

* **Confidentiality**: Ensures that only authorized individuals can access information. 
* **Integrity**: Ensures that the information can only be altered by authorized individuals.
* **Availability**: Ensures that information is accessible to authorized users whenever it is needed.

### The DAD model
The DAD model (**D**isclosure, **A**lteration and **D**estruction) is essentially what the CIA triad is trying to prevent:  
Confidentiality ↔ **Disclosure** (Failing confidentiality leads to disclosure).  
Integrity ↔ **Alteration** (Failing integrity leads to unauthorized alteration).  
Availability ↔ **Destruction** (Failing availability leads to destruction or denial of access). 

### Threats to Confidentiality and Integrity

Referring back to the attacks we mentioned earlier, **network packet capture**, when data is transmitted in clear text, violates confidentiality and can lead to the disclosure of information. A successful **password attack** (such as a brute force attack, dictionary attack, or credential stuffing) can also result in disclosure. On the other hand, **Man-in-the-Middle** (MITM) attacks compromise the integrity of the system, as they can alter the communicated data.

These three attacks — network packet capture, password attacks, and MITM — are essential to understand because they exploit weaknesses that are directly related to how protocols and servers are built.

Exploiting a **vulnerability** can have different effects depending of the exploit and the targeted system. The exploits vary from Denial of Service (DoS), which can affect the system's availability, to Remote Code Execution (RCE), which can lead to severe damage. The existence of a vulnerability creates a risk while damage can occur if the vulnerability is exploited. 

This room will focus on how we can upgrade the protocols or replace them to protect against disclosure and alteration. 

## Task 2: Sniffing Attack
A sniffing attack is essentially using software (or network packet capturing tools) to eavesdrop on a network to collect information about a target. This attack is especially effective against protocols that communicate in clear text since all data can be interpreted without much effort. As long as the attacker has access to the network traffic the attack will be successful when using clear text.

Before we look into how this can be fixed, let's have a look at how a sniffing attack can be conducted.

### What you need
All you need to perform a sniffing attack is access to the network you want to eavesdrop, an Ethernet network card and a program to capture network packets.

A few of these tools include:
* Tcpdump: Free open source command-line interface (CLI) program.
* Wireshark: Free open source graphical interface (GUI) program
* Tshark: A CLI alternative to Wireshark.

### Snapshot Tcpdump
**Purpose:** Tcpdump is a command-line packet analysis tool for capturing and analyzing network traffic.

**Command Layout:** the command `sudo tcpdump port 110 -A` is used to capture packets from POP3 port and display in ASCII format (the `-A` -> ASCII)

The example given on THM is the following:

```terminal
pentester@TryHackMe$ sudo tcpdump port 110 -A
[...]
09:05:15.132861 IP 10.20.30.1.58386 > 10.20.30.148.pop3: Flags [P.], seq 1:13, ack 19, win 502, options [nop,nop,TS val 423360697 ecr 3958275530], length 12
E..@.V@.@.g.
...
......n......"............
.;....}.USER frank

09:05:15.133465 IP 10.20.30.148.pop3 > 10.20.30.1.58386: Flags [.], ack 13, win 510, options [nop,nop,TS val 3958280553 ecr 423360697], length 0
E..4..@.@.O~
...
....n....".........?P.....
...i.;..
09:05:15.133610 IP 10.20.30.148.pop3 > 10.20.30.1.58386: Flags [P.], seq 19:43, ack 13, win 510, options [nop,nop,TS val 3958280553 ecr 423360697], length 24
E..L..@.@.Oe
...
....n....".........<-.....
...i.;..+OK Password required.

09:05:15.133660 IP 10.20.30.1.58386 > 10.20.30.148.pop3: Flags [.], ack 43, win 502, options [nop,nop,TS val 423360698 ecr 3958280553], length 0
E..4.W@.@.g.
...
......n......".....??.....
.;.....i
09:05:22.852695 IP 10.20.30.1.58386 > 10.20.30.148.pop3: Flags [P.], seq 13:28, ack 43, win 502, options [nop,nop,TS val 423368417 ecr 3958280553], length 15
E..C.X@.@.g.
...
......n......".....6......
.<.....iPASS D2xc9CgD
[...]
```

Where you clearly can see that this captured both the username and password.

### Snapshot Wireshark
**Purpose:** Wireshark is a graphical network protocol analyzer that allows users to capture and interactively browse the traffic running on a computer network.

**User interface:** 

![Wireshark UI](/assets/images/wireshark_UI.png)
*Wireshark UI, source TryHackMe room Protocols And Servers 2*

As can be seen in the image, the username and password can be viewed in clear text here as well. Notice the "pop" written in the green input field. That is for filtering the packets to only show packets from the POP protocol. 

### Mitigation
As demonstrated with Tcpdump and Wireshark, the transmission of usernames and passwords in clear text poses a significant security risk. This vulnerability is common across all the protocols mentioned in the introduction - HTTP, FTP, SMTP, POP3, and IMAP - as well as any other protocols that transmit data unencrypted. To address this issue, Transport Layer Security (TLS) has been implemented to provide an encryption layer over these network protocols. Additionally, for remote access, Telnet has been replaced by the more secure **S**ecure **SH**ell (SSH).

---

**Q: What do you need to add to the command `sudo tcpdump` to capture only Telnet traffic?**
<details><summary>Answer</summary>
port 23
</details>

---

**Q: What is the simplest display filter you can use with Wireshark to show only IMAP traffic?**
<details><summary>Answer</summary>
IMAP
</details>

---

## Task 3: Man-in-the-Middle (MITM) Attack
When sniffing, you passively listen to network traffic. In contrast, a Man-in-the-Middle (MITM) attack involves actively intercepting and altering communication between two parties without their knowledge. For example, if you intercept a request from party A intended for party B, you can impersonate A, modify the request, and then forward it to B. In this scenario, you effectively become the "man in the middle."

There are two tools commonly used:
* Ettercap - offers three interfaces; Traditional command line, GUI and Ncurses
* Bettercap - offers three interfaces; Web UI, interactive mode, Scripting

---

**Q: How many different interfaces does Ettercap offer?**

![alt text](/assets/images/ettecap_answer.png)

<details><summary>Answer</summary>
3
</details>

---

**Q: In how many ways can you invoke Bettercap?**

![alt text](/assets/images/bettercap_answer.png)

<details><summary>Answer</summary>
3
</details>

---

## Task 4: Transport Layer Security (TLS)
In order to protect the confidentiality and integrity of data transmitted over the network, Secure Sockets Layer (SSL) and, nowadays more commonly used, Transport Layer Security (TLS) were introduced. They have the main purpose of preventing attacks like password sniffing and Man-in-the-Middle attacks. 

Once internet became more commonly used and new areas of online activities involving payment transfers emerged, the SSL was introduced in 1994 to increase the security. In 1999 the more secure alternative TLS was introduced. Today TLS has essentially replaced SSL.

SSL/TLS adds an encryption layer to the presentation layer, which ensures that data is encrypted before being transmitted. This prevents any captured data from being readable in clear text without a decryption key, thus protecting both confidentiality and integrity of the data.

The advantage of SSL/TLS is that it can be applied to existing protocols and make them secure. For example HTTP becomes HTTPS, which uses port 443 instead of por 80. In a similar manner "S" is added at the end of the protocols to indicate that they use SSL/TLS (FTPS, SMTPS, POP3S, IMAPS). 

| Protocol | Default port | Secured Protocol | Default port with TLS|
|----------|--------------|------------------|----------------------|
| HTTP     |    80        |        HTTPS     |          443         |
| FTP     |    21        |        FTPS     |          990         |
| SMTP     |    25        |        SMTPS     |          465         |
| POP3     |    110        |        POP3S     |          995         |
| IMAP     |    143        |        IMAPS     |          993         |

To illustrate how SSL/TLS works in practice, let's first have a look at HTTP and how a web browser and web server initially established a connection.

1. TCP connection with remote web server
2. Send HTTP requests such as `GET` and `POST`

HTTPS requires an additional step here;
1. Establish a TCP connection
2. **Establish SSL/TLS connection**
3. Send HTTP requests to the web server

### Establishing SSL/TLS connection
The SSL/TLS connection is established in four general steps:
1. **Client Hello**: 
    * The client starts by sending a _ClientHello_to the server to indicate that it wishes to establish a connection.
    * This message includes details like:
        - Encryption algorithms
        - Connection preferences (like which version of TLS it wants to use)
    * Think of this as a way to say "Hello, here is how I can communicate securely"
2. **ServerHello and Certificate**
    * The server responds with a message called ServerHello
    * In this message the server:
        - Chooses connection settings
        - Provides a digital certificate. This certificate is signed by a trusted third party called a _Certificate Authority_ (CA)
    * The digital certificate helps the client confirm that it is communicating with the intended server and not an imposter.
    * The server may also send a _ServerKeyExchange_ message if extra details are needed to create a master key
    * The server indicates that it is done sharing information and is ready to proceed by sending a _ServerHelloDone_
3. **ClientKeyExchange and ChangeChipherSpec**
    * The client responds with a __ClientKeyExchange__ message containing information that both the client and server will use to create a shared master key, which will be used to encrypt teh entire session.
    * The client then sends a _ChangeCipherSpec_ message indicating that it is switching to encrypted communication
4. The server responds with a _ChangeCipherSpec_ message letting the client know that it is ready to switch to encrypted communication as well. 

Well that was a mouthful! Have a look at [this](https://www.youtube.com/watch?v=25_ftpJ-2ME) video where David Bombal interviews Ed from PracticalNetworking diving a little deeper into the TLS hand shake! 

---

**Q: DNS can also be secured using TLS. What is the three-letter acronym of the DNS protocol that uses TLS?**
<details><summary>Answer</summary>
You're gonna have to show off your google skills here... Search for "DNS with TLS" and the first hit you get reveals the answer!
</details>

---

## Task 5: Secure Shell (SSH)
We have previously gotten acquainted with Telnet, which allows you to connect to a system over network and execute commands on them. SSH has the same purpose as Telnet but, unlike Telnet, it confirms the identity of the remote server, encrypts messages and allows both sides to detect modifications in the messages as well. 

To use SSH, you need an SSH server and an SSH client. By connecting to port 22, the default port that SSH servers listen to, the client can authenticate using either username and password or a private and public key.

On linux you would use the command `ssh pentester@MACHINE_IP` which will try to connect to the server of the IP address with the username `pentester`. SSH will them prompt you to provide a password for the username and once you are authenticated, you will have access to the server's terminal.

When connecting to an SSH server for the first time, you will need to manually verify the fingerprint of the server's public key to protect against Man-in-the-Middle (MITM) attacks. 

In the case of SSH, we typically don't have a trusted third party to verify the server's public key for us. Therefore, it is important to verify this information manually during the first connection to ensure that the server you're connecting to is legitimate. This helps prevent MITM attacks, which could otherwise compromise your communication. 

### Secure Copy Protocol
We can use SSH to transfer files by using SCP (Secure Copy Protocol). The syntax is as follows `scp pentester@MACHINE_IP:/path/to/file /path/to/whereYouWantToPutIt`, just like a normal `cp` command.

Note that FTP can be used with SSL/TLS as well by using the FTPS protocol on port 990. You can also secure FTP using the SSH protocol, SFTP, in which case the service would listen to port 22 just like SSH.

---

**Q: Use SSH to connect to 10.10.106.215 as mark with the password XBtc49AB. Using uname -r, find the Kernel release?**
<details><summary>Answer</summary>
5.15.0-119-generic 

Log in via ssh mark@MACHINE_IP, you might get a prompt asking if you want to proceed, enter "yes" and when asked for the password enter it. Then you simply type uname-r into the terminal and you should get the Kernel release.
</details>

---

**Q: Use SSH to download the file book.txt from the remote system. How many KBs did scp display as download size?**
<details><summary>Answer</summary>
415.

 Use   
scp mark@MACHINE_IP:/home/mark/book.txt .   
to download the boom.txt to the directory you ran the command from. Note that you cannot be logged in as mark via SSH from the previous question, enter "exit" to close the connection.
</details>

## Task 6: Password Attack
Another significant risk in network security are password attacks. Passwords are commonly used for authentication in protocols making them vulnerable to attacks. A password can be guessed either by a well educated guess or by sifting through a dictionary (dictionary attack), like a list of leaked passwords, Rockyou.txt is a great example of such dictionary. Sometimes passwords are even brute-forced, where every possible combination is tested. Fun fact: Burp-suite can actually help you with a few types of password attacks on websites. 

The tool we will focus on here is Hydra. I won't go into as much detail as they do in the room though.
The main syntax is as follows:
`hydra -l username -P wordlist.txt server service`

**`-l username`:** -l should precede the username, i.e. the login name of the target.  
**`-P wordlist.txt`**: -P precedes the wordlist.txt file, which is a text file containing the list of passwords you want to try with the provided username.  
**`server`** is the hostname or IP address of the target server.  
**`service`** indicates the service which you are trying to launch the dictionary attack.

You can adopt several measures to mitigate password attacks. Enforcing a strong password policy, implementing account lockout mechanism and adding delays to authentication attempts can help reduce the risk. The Two-factor authentication (2FA) and CAPTHCA provides additional security by making it harder for automated tools. 

---

**Q: We learned that one of the email accounts is `lazie`. What is the password used to access the IMAP service on MACHINE_IP?**
<details><summary>Answer</summary>
hydra -l lazie -P /usr/share/wordlists/rockyou.txt imap://MACHINE_IP

This should give you the password.

</details>

## Summary
This miniseries on Protocols and Servers has looked at a few protocols and their usage along with the basics of how they work under the hood. 

The three common attacks on these protocols we have covered are:
* Sniffing Attack
* MITM Attack
* Password Attack. 

Make sure to have the protocols and ports somewhere to be found, until they stick, it's nice to have around.