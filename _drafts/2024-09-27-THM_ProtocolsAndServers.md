---
title: TryHackMe - Protocols and Servers
description: A walkthrough of TryHackMe's room "Protocols and Servers".
categories: [Walkthrough THM, Network Security Module]
tags: [Basics, Protocols]
pin: false
---

Let's continue our path in the Network Security Module with some Protocols and Servers with the room Protocols And Servers, found [here](https://tryhackme.com/r/room/protocolsandservers).

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

To retrieve a web page when having a telnet connection to a server, you can simply type `GET /index.html HTTP/1.1` to retrieve the default page and then provide a host, `host: telnet` and press Enter twice.

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

**Q:**
<details><summary>Answer</summary>

</details>

---