---
title: TryHackMe - Passive Reconnaissance
description: A walkthrough of TryHackMe's room "Passive Reconnaissance".
categories: [Walkthrough THM, Network Security Module]
tags: [Basics]
pin: false
---

Lab access: [https://tryhackme.com/r/room/passiverecon](https://tryhackme.com/r/room/passiverecon)

## Introduction

This is the first room in the module "Network Security" which will cover passive and active reconnaissance, nmap and look at some basic ports. 

We will start off by looking at a three essential commandline tools used in passive reconnaissance:
* `whois` - query WHOIS servers
* `nslookup` - query DNS servers
* `dig` - query DNS servers  

Furthermore, we have a few online services at our disposal:
* DNSDumpster
* shodan.io

All of these tools utilize publicly available information and is therefore seen as passive reconnaissance. We will look more into what is active and passive reconnaissance in the next task.

>It is recommended that you have a basic understanding of networking and linux for this module. 
{: .prompt-warning}

## Task 2: Passive Versus Active Recon
Reconnaissance (recon) can be defined as a preliminary survey to gather information about a target, and we will divide it into two parts in this module, _Active_ and _Passive_ reconnaissance. This is the first step in gaining initial foothold on a system. (Have a look at the Unified Kill Chain, either the [THM room](https://tryhackme.com/r/room/unifiedkillchain) or the [website](https://www.unifiedkillchain.com/)).

### Passive Reconnaissance
If you only use publicly available data in your reconnaissance, it is passive. The moment you interact with the target you stepped into the active reconnaissance sphere. 

Some activities that fall under the passive reconnaissance definition is:
* Finding DNS records of a domain from a public DNS server
* Checking job ads related to the target website
* Reading news articles about the company

### Active Reconnaissance
Unlike passive reconnaissance, active reconnaissance interacts directly with target, for example:
* Connecting to one of the company servers such as HTTP, FTP and SMTP
* Calling the company in an attempt to get information
* Entering company premises pretending to be a repairman.  

---

**Q: You visit the Facebook page of the target company, hoping to get some of their employee names. What kind of reconnaissance activity is this? (A for active, P for passive)**
<details><summary>Answer</summary>
P, you are not actively interacting with the target. Facebook is open information for all.
</details>

---

**Q: You ping the IP address of the company webserver to check if ICMP traffic is blocked. What kind of reconnaissance activity is this? (A for active, P for passive)**
<details><summary>Answer</summary>
A, you are sending data to the companys server, and therefore interacting directly with the target.
</details>

---


**Q: You happen to meet the IT administrator of the target company at a party. You try to use social engineering to get more information about their systems and network infrastructure. What kind of reconnaissance activity is this? (A for active, P for passive)**
<details><summary>Answer</summary>
A, you are, once again, interacting directly with the target.
</details>


## Task 3: Whois
WHOIS is a request and response protocol that follows RFC 3912: WHOIS Protocol Specification. It is essentially a big database holding various information related to domains. Such information includes:
* Registrar
* Contact information of registrar
* Creation, update and expiration dates
* Name server
* Etc.

To retrieve this information to your computer, we use a `whois` client or an online service. The general syntax is `whois DOMAIN_NAME`, so for example `whois tryhackme.com`. 
Which would return 

```terminal
user@TryHackMe$ whois tryhackme.com
[Querying whois.verisign-grs.com]
[Redirected to whois.namecheap.com]
[Querying whois.namecheap.com]
[whois.namecheap.com]
Domain name: tryhackme.com
Registry Domain ID: 2282723194_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.namecheap.com
Registrar URL: http://www.namecheap.com
Updated Date: 2021-05-01T19:43:23.31Z
Creation Date: 2018-07-05T19:46:15.00Z
Registrar Registration Expiration Date: 2027-07-05T19:46:15.00Z
Registrar: NAMECHEAP INC
Registrar IANA ID: 1068
Registrar Abuse Contact Email: abuse@namecheap.com
Registrar Abuse Contact Phone: +1.6613102107
Reseller: NAMECHEAP INC
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Registry Registrant ID: 
Registrant Name: Withheld for Privacy Purposes
Registrant Organization: Privacy service provided by Withheld for Privacy ehf
[...]
URL of the ICANN WHOIS Data Problem Reporting System: http://wdprs.internic.net/
>>> Last update of WHOIS database: 2021-08-25T14:58:29.57Z <<<
For more information on Whois status codes, please visit https://icann.org/epp
```

You can look through the information above to retrieve a bunch of information on registrars, contact information, domain name servers and much more.

### What does `whois` provide an ethical hacker?
First off, you could be provided with contact information that can be useful in social engineering attacks. You also get access to domain registration and expiration dates, if the owner doesn't renew the domain it might lapse and be vulnerable to domain hijacking. Furthermore, you get name server information, if they are misconfigured you could expose internal network details to map target's infrastructure. These are just some examples of how the `whois` tool can provide valuable information. 

---

Use a terminal to run `whois tryhackme.com` to get the information you need to answer the following questions.

**Q: When was TryHackMe.com registered?**
<details><summary>Answer</summary>
20180705

Creation Date: 2018-07-05T19:46:15.00Z
</details>

---

**Q: What is the registrar of TryHackMe.com?**
<details><summary>Answer</summary>
namecheap.com

Registrar WHOIS Server: whois.namecheap.com
</details>

---

**Q: Which company is TryHackMe.com using for name servers?**
<details><summary>Answer</summary>
cloudflare.com

Name Server: KIP.NS.CLOUDFLARE.COM
</details>

## Task 4: nslookup and dig
You just looked at how to retrieve various information about a domain using `whois`, one thing we were able to retrieve was the DNS servers from the registrar.

### nslookup
You can find the IP address of a server using `nslookup`. You can simply issue the command `nslookup DOMAIN_NAME` or can add options using the command `nslookup OPTIONS DOMAIN_NAME SERVER`. Where `OPTIONS`  specify the query type (e.g., A for IPv4, MX for mail servers), `DOMAIN_NAME` is the target, and `SERVER` is the DNS server to query (e.g., Cloudflare's 1.1.1.1).

Let's look at tryhackme.com again.

```terminal
user@TryHackMe$ nslookup -type=A tryhackme.com 1.1.1.1
Server:		1.1.1.1
Address:	1.1.1.1#53

Non-authoritative answer:
Name:	tryhackme.com
Address: 172.67.69.208
Name:	tryhackme.com
Address: 104.26.11.229
Name:	tryhackme.com
Address: 104.26.10.229
```

`nslookup` is useful for quick checks when you just need an IP address or a simple record lookup.

### dig
For more advanced DNS queries, you can use `dig` ("domain information groper"). Dig provides more comprehensive output DNS, it’s especially useful for network troubleshooting. It is also more commonly used in scripts due to its consistent and easily parsable output format.

Let's again look at tryhackme.com for reference

```terminal
; <<>> DiG 9.20.1-1-Debian <<>> tryhackme.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21830
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;tryhackme.com.			IN	A

;; ANSWER SECTION:
tryhackme.com.		300	IN	A	104.22.55.228
tryhackme.com.		300	IN	A	172.67.27.10
tryhackme.com.		300	IN	A	104.22.54.228

;; Query time: 32 msec
;; SERVER: 82.195.202.150#53(82.195.202.150) (UDP)
;; WHEN: Wed Oct 02 12:53:00 EDT 2024
;; MSG SIZE  rcvd: 90
```


**Q: Check the TXT records of thmlabs.com. What is the flag there?**

__*Answer*__

>Use either 
>
>nslookup: `nslookup -type=TXT thmlabs.com`
>
> or 
> 
> dig: `dig thmlabs.com TXT`
>
>to retrieve the flag

## Task 5: DNSDumpster
Unfortunately, DNS lookup tools like nslookup and dig cannot find subdomains on their own. Subdomains are common for organize and manage different section of websites. In a domain blog.example.com, _blog_ is a subdomain to _example.com_. It is possible that these subdomains are falling behind in updates and you might find vulnerable services running. 

DNSDumpster is a way to easily get data on subdomains for a domain and it is online. 

**Q: Lookup tryhackme.com on DNSDumpster. What is one interesting subdomain that you would discover in addition to www and blog?**
![alt text](/assets/images/dnsdumpster.png)

<details><summary>Answer</summary>
remote
</details>

## Task 6: shodan.io
shodan.io is your google for network-connected devices, it is a  specialized search engine designed to discover and index devices and services exposed to the internet. You can search for domain names, IP addesses, protocols and much more, and it will provide you with devices connected matching the search query. It is sometimes referred to as the "search engine for hackers". 

---

Go to [shodan.io](https://www.shodan.io/) to answer the questions below.

**Q: According to Shodan.io, what is the 2nd country in the world in terms of the number of publicly accessible Apache servers?**

**Hint:** Which type of server are you looking for? Enter that as the keyword in your search. On the left side you will see a list of the top countries.
<details><summary>Answer</summary>
Germany
</details>

---

**Q: Based on Shodan.io, what is the 3rd most common port used for Apache?**

**Hint:** Use the same search as above but look at the top ports list instead.
<details><summary>Answer</summary>
8080
</details>

---

**Q: Based on Shodan.io, what is the 3rd most common port used for nginx?**
<details><summary>Answer</summary>
5001, search for nginx and you will see it in the top ports on the left.
</details>


## Task 7: Summary
In this room, we focussed on passive reconnaissance by looking at tools like `whois`, `nslookup` and `dig`. We also looked at the online tools DNSDumpster and shodan.io.
Learning to use these tools efficiently and understanding their results can provide valuable information!

