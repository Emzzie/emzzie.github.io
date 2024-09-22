---
title: Nmap Live Host Directory
description: A walkthrouhg of TryHackMe's room "Nmap Live Host Discovery".
author: Emzzie
date:  2024-09-19 10:00 +0800
categories: [Walkthrough THM]
tags: [Nmap, Tools]
pin: false
---

## Foreword
Hey there! 
Welcome to this walktrhough of the room "Nmap Live Host Discovery"! 

I strongly recommend getting the basics refreshed before starting this room. Unless you feel that you got the the nitty gritty of ARP, ICMP, TCP/UDP and the OSI model straightend out, either have a look at **ADD ROOMS** or quickly have a look at 
* [Another OSI Model Breakdown]({% post_url 2024-09-19-OSI_Model_Basics %}).
* [ARP The Basics]({% post_url 2024-09-20-ARP_The_Basics%}).
* [ICMP - The Two Edged Sword]({% link _drafts/ICMP_The_Two_Edged_Sword.md%}). 

Note that I won't do a copy-paste from TryHackMe. You can read that yourself, I will provide my own take on the information given and guide you to the correct answers in my own words. 

When you feel ready lets go ahead and discover this room!

## Task 1: Introduction
So, we want to target a network. It's a dark hole, all we have is an IP  and possibly a subnet mask... Who (or rather what) comes to the rescue? Yes you guessed it! __Nmap!__ The good ol' friend that we always consult in these situations. The friend that does all the repetetive work for us and just hands us the report we ask for! 

There are two main things that Nmap help with
* Which systems are up and running
* What services are running on these systems?

>Let's get the teminology for system and services straightend out right away
>* A _system_ typically refers to any device or host that is connected to the network. For example computers, routers, switches and IoT devices.
>* A _service_ refers to a network application or protocol that runs on a specific port of a system. For example HTTP, HTTPS, SMTP, IMAP, FTP and databases such as MySQl. 

This room focuses primarlily on finding out which systems are up and running. 

How is this different from the "normal" port scan that we have gotten used to? Provided with a subnet, it is an absolute waste of time to do a port scan if the system  (or host) is offline.

This room will present three different approches Nmap can use to discover live hosts.
1. __ARP scan:__ Uses ARP Requests to find live hosts
2. __ICMP scan__ Uses 





