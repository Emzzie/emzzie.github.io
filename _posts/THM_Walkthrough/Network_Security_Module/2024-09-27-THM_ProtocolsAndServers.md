---
title: TryHackMe - Protocols and Servers
description: A walkthrough of TryHackMe's room "Protocols and Servers".
categories: [Walkthrough THM, Network Security Module]
tags: [Basics, Protocols]
pin: false
---

Let's continue our path in the Network Security Module with some Protocols and Servers with the room "Protocols And Servers", found [here](https://tryhackme.com/r/room/protocolsandservers).

## Task 1: Introduction
This room will cover the basics of the following protocols:
* HTTP
* FTP
* POP3
* SMTP
* IMAP

These protocols are essential for everyday computer and email use. Although they operate at the Application layer, they are often hidden behind graphical user interfaces (GUIs). In this room we will take a peak behind the scenes and use these protocols hands on! So let's get to it!

## Task 2: Telnet
Telnet, or Teletype Network Protocol, is mainly listening to connections on port 23. It is a protocol used for connecting to a virtual terminal on another computer. Developed in 1969, it was widely used during the 1970s and 1980s by academic and research institutions to access mainframe computers and supercomputers remotely. Telnet became an important tool for sharing information among researchers and academics, as it was also used for email and file transfer.

Today, however, Telnet is rarely used because **all** data is sent in **cleartext**, including usernames, passwords, and any other transmitted information. This lack of encryption makes it too insecure for modern use. Instead SSH has taken its place as the secure alternative. (Which will be explored further in the next room)

---

**Q: To which port will the `telnet` command with the default parameters try to connect?**
<details><summary>Answer</summary>
23
</details>

---

## Task 3: Hypertext Transfer Protocol (HTTP)
Hypertext Transfer Protocol HTTP is the protocol used for transferring web pages, so you reading this right now utilizes the HTTP protocol! Your web browser connects to the web server and uses HTTP to request the HTML (Hypertext Markup Language) pages and other resources resources. 

HTTP sends and receives data in cleartext, making it possible to use tools like Telnet to communicate with a web server and act as a "web browser". 
Have a look at the room on [HTTP i Detail](https://tryhackme.com/r/room/howwebsiteswork) if you haven't already, to get a deeper dive into HTTP and it's commands. 

To retrieve a web page when having a telnet connection to a server, you can simply type `GET /index.html HTTP/1.1` to retrieve the default page (index.html) and then provide a host, `host: telnet` and press Enter twice.

This will look something like:
```terminal
pentester@TryHackMe$ telnet MACHINE_IP 80
Trying MACHINE_IP...
Connected to MACHINE_IP.
Escape character is '^]'.
GET /index.html HTTP/1.1
host: telnet

```

Then you will see a response from the server with a HTTP header and the contents of index.html.

In order to use the HTTP protocol, an HTTP server (web server) and an HTTP client (web browser) is needed. Where the web browser requests specific files that the server provides.

A few common choices for HTTP servers include:
* Apache
* Internet Information Services (IID)
* nginx

Currently, the most common web browsers are:
* Chrome by Google
* Edge by Microsoft
* Firefox by Mozilla
* Safari by Apple

In general, these web browsers are free to install.

---

**Q:Launch the attached VM. From the AttackBox terminal, connect using Telnet to 10.10.240.126 80 and retrieve the file flag.thm. What does it contain?**
<details><summary>Answer</summary>
Rules from THM states that writeups are not allowed to include flags. So I'll give the way to retrieve it...

1. Connect using telnet: telnet MACHINE_IP 80
2. Remember how we got the index.html to show? Exchange the index.html with flag.thm: GET /flag.thm HTTP/1.1 [ENTER] host: telnet
3. Press enter twice. And the flag will show.
</details>

## Task 4: File Transfer Protocl (FTP)

File Transfer Protocol (FTP) is a standard network protocol used to transfer files between a client and a server over the Internet or within a local network. It operates on a client-server model where a client initiates a connection to the server. FTP, however, transmits both commands and data, including usernames and passwords, in plain text, which makes it vulnerable to interception by attackers.

There are two modes of operation:
* **Active Mode:** The server connects back to the client to send data
* **Passive Mode:** The client establishes both the command and data connections, making it firewall friendly. 

Due to FTP sending credentials in plain text, it’s important to understand the inherent security risk. This is why FTP is generally considered insecure unless used within a protected network or alongside encryption (like in FTPS).

Some of the most frequently used FTP commands include:
* `USER` and `PASS`: Used for authentication.
* `LIST`: Lists files in the current directory.
* `RETR`: Downloads files from the server.
* `STOR`: Uploads files to the server.
* `QUIT`: Ends the FTP session

You can interact with an FTP server using Telnet. However, this is not very practical since FTP creates a separate data channel for file transfers, which Telnet cannot easily handle.

```Shell
telnet MACHINE_IP 21  
```

A better solution is to use a dedicated FTP client that can handle both control and data channels effectively.

```Shell
ftp MACHINE_IP
```

There are a few examples of FTP server software these include:
* vsftpd
* ProFTPD
* uFTP

---

**Q: Using an FTP client, connect to the VM and try to recover the flag file. What is the flag?**

**Username: frank**  
**Password: D2xc9CgD**

**Answer**  
Note that it states use an _ftp client_, so lets use `ftp MACHINE_IP`. It might take a few minutes before hte FTP server is up and running, so be patient and try agian if you don't get a login prompt. Log in using the credentials above. 

You can now use `ls` to show the available files. The `get` command allows you to download a file to the directory you ran `ftp` from, use `get ftp_flag.thm` to get the flag file. 

It should look something like this:

```terminal
└─$ ftp MACHINE_IP
Connected to MACHINE_IP.
220 (vsFTPd 3.0.5)
Name (MACHINE_IP:kali): frank
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||61323|)
150 Here comes the directory listing.
drwx------   10 1001     1001         4096 Sep 15  2021 Maildir
-rw-rw-r--    1 1001     1001         4006 Sep 15  2021 README.txt
-rw-rw-r--    1 1001     1001           39 Sep 15  2021 ftp_flag.thm
226 Directory send OK.
ftp> get ftp_flag.thm
local: ftp_flag.thm remote: ftp_flag.thm
229 Entering Extended Passive Mode (|||24064|)
150 Opening BINARY mode data connection for ftp_flag.thm (39 bytes).
100% |***********************************|    39       98.92 KiB/s    00:00 ETA
226 Transfer complete.
39 bytes received in 00:00 (0.70 KiB/s)
ftp> quit
221 Goodbye.

```
## Task 5: Simple Mail Transfer Protocol (SMTP)
Simple Mail Transfer Protocol (SMTP) is a standard communication protocol for sending email messages across the internet. It uses a client-server model where a mail client sends messages to a mail server which then relays them to teh recipient's server. It is SMTP that ensures that email gets from the sender to the recipient's mail server.

There are four key components in the email transmission. 
* **Mail User Agent (MUA)**: This is the email client you interact with directly like outlook, gmail etc. It is responsible for composing, sending and receiving emails.
* **Mail Submission Agent (MSA)**: The MSA is responsible for receiving the client's (MUA's) outgoing mail and ensuring that it follows the standards required for proper delivery. 
* **Mail Transfer Agent (MTA)**: The MTA accepts email from the MSA and transfers it to the recipient's mail server. It can be seen as the backbone of email delivery. There are a few examples that include **Postfix**, **sendmail** and **exim**. The MTA communicates with other MTAs using SMTP, usually on port 25.
* **Mail Delivery Agent (MDA)**: The Mail Delivery Agent (MDA) handles the email once it has been delivered to the recipients server by the MTA. It is, for example, responsible for placing the email in the recipients mail box where the MUA can interact with it.

Common SMTP Commands
SMTP communication between client and server is text-based, using various commands to initiate and complete the process of sending an email:

* **HELO / EHLO**: Introduces the client to the SMTP server.
* **MAIL FROM**: Specifies the sender’s email address.
* **RCPT TO**: Specifies the recipient’s email address.
* **DATA**: Indicates the start of the message body, followed by the subject, content, and headers.
* **QUIT**: Ends the SMTP session.

SMTP sends email messages in plain text by default. Sending data in plain text over the internet makes it susceptible to interception. This is the case for both login credentials and email contents are visible if the transmission is intercepted. To enhance security, SMTP is often used with TLS/SSL (known as SMTPS) on port 587 or 465.

---

**Q: Using the AttackBox terminal, connect to the SMTP port of the target VM. What is the flag that you can get?**

**Answer**: Once again this is a flag, so it cannot be explicitly posted, but this one is easy! You can simply access the SMTP server using telnet as follows and the flag is displayed directly:

```shell
└─$ telnet MACHINE_IP 25 
Trying MACHINE_IP...
Connected to 1MACHINE_IP.
Escape character is '^]'.
220 bento.localdomain ESMTP Postfix FLAG_SHOWN_HERE
```

## Task 6: Post Office Protocol 3 (POP3)
The Post Office Protocol 3 (POP3) works on the recipient's side of the transmission. It is used to communicate between the MDA (Mail Delivery Agent) and and the MUA (Mail User Agent) and downloading all the messages in the inbox to your device. Once all messages have been retrieved by your device, they are deleted from the server. This approach allows you to access the emails offline, they are stored locally and can be read without internet connection. However, there is a small caveat, there is limited support for synchronizing across multiple devices. Reading, deleting and messages on one device won't be reflected on others. Additionally, it's important to note that POP3 also transmits your credentials and emails in clear text, making it vulnerable to eavesdropping if not used with additional encryption, such as SSL/TLS. POP3 typically operates over port 110 for unencrypted connections and port 995 for secure connections using SSL/TLS.

The main commands commonly used with POP3 are:
* **USER**: Specifies username of the email account to log in
* **PASS**: After the `USER` command this command provides the password to authenticate the user.
* **STAT**: Provides a status summary of the mailbox, showing the number of messages and the total size in bytes.
* **LIST**: Lists all messages in the mailbox and their sizes.
* **RETR**: This command retrieves a specific email message from the server by providing the message number.

---

**Q: Connect to the VM (MACHINE_IP) at the POP3 port. Authenticate using the username frank and password D2xc9CgD. What is the response you get to STAT?**  
Use the following:
```shell
└─$ telnet MACHINE_IP 110
Trying MACHINE_IP...
Connected to MACHINE_IP.
Escape character is '^]'.
+OK Hello there.
USER frank
+OK Password required.
PASS D2xc9CgD
+OK logged in.
STAT

```
<details><summary>Answer</summary>
The response from STAT is: 
+OK 0 0
</details>

---

**Q: How many email messages are available to download via POP3 on MACHINE_IP?**
<details><summary>Answer</summary>
Sinste STAT shows the number of messages and theirs size, and we got 0 0  in the previous question, the answer here is also 0.
</details>

## Internet Message Access Protocol (IMAP)
Just like POP3, the Internet Message Access Protocol (IMAP) works on the recipient's side to retrieve messages from the MDA (Mail Delivery Agent) to the MUA (Mail User Agent). However, IMAP is more sophisticated, allowing you to keep emails synchronized across multiple devices and mail clients. 
This means that actions like reading, deleting, or moving messages are reflected in real-time on all devices, making it a better option for users who access their emails from various locations. IMAP typically operates over port 143 for unencrypted connections and port 993 for secure connections using SSL/TLS.

Additionally, IMAP enables users to manage mail folders on the server, allowing for greater organization and flexibility. Unlike POP3, which typically downloads emails and removes them from the server, IMAP keeps emails stored on the server until the user decides to delete them. It also supports searching through emails on the server without needing to download them first, enhancing user experience and efficiency.

Some common commands include:

* **LOGIN**: Authenticates the user by providing a username and password.
* **SELECT**: Selects a mailbox (folder) to operate on. This is necessary before performing actions like retrieving messages.
* **LIST**: Lists the mailboxes (folders) available on the server.
* **SEARCH**: Searches for messages that match specific criteria, such as unread messages or messages from a specific sender.
* **EXAMINE**: Opens a mailbox in read-only mode, allowing the user to check the status of messages without modifying any flags or deleting messages.

Just like POP3 and ICMP, IMAP sends login credentials and contents in clear text, unless used with additional encryption. 

---

**Q: What is the default port used by IMAP?**
<details><summary>Answer</summary>
143
</details>

---

## Task 8: Summary

This room covered some common protocols that are mainly communicating in clear text. 

Here is a table from THM with the covered protocols and their default ports.

| Protocol | TCP Port | Application(s)      | Data Security |
|----------|----------|---------------------|---------------|
| FTP      | 21       | File Transfer       | Cleartext     |
| HTTP     | 80       | Worldwide Web       | Cleartext     |
| IMAP     | 143      | Email (MDA)        | Cleartext     |
| POP3     | 110      | Email (MDA)        | Cleartext     |
| SMTP     | 25       | Email (MTA)        | Cleartext     |
| Telnet   | 23       | Remote Access       | Cleartext     |