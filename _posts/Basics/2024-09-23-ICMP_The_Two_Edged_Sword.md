---
title: IMCP - The Two Edged Sword
description: Let's explore IMCP. A fundamental tool for maintaining network integrity and a potential vector for security threats.
categories: [Basics, Protocols]
tags: [Basics, Protocols]
pin: false
---

ICMP (Internet Control Message Protocol) is a fundamental part och the Internet Protocol. It is used for error reporting and diagnostic functions within a network. It is crucial in diagnosing problems within networks and helps improve the efficiency of communication. There are two well-known tools that use ICMP, _ping_ and _traceroute_, apart from these tools, users rarely interact with ICMP. The ICMP message is actually encapsulated in an IP packet but it "lives" in the network layer of the OSI model. The ICMP messages can substantially improve the traffic flow of both TCP and UDP messages by providing feedback that allows the protocols to adapt to network conditions, optimize performance and avoid issues. Which in turn leads to more efficient and reliable communication. 

## Common ICMP Messages
Let's start with defining some of the common ICMP messages and their uses. They can in general be divided into two groups, _Error reporting_ and _Query Messages_.

### Error reporting
These messages inform a source node about issues preventing the delivery of packets. The most common messages are:

1.  __Destination Unreachable:__ This message is sent when a packet can't reach it's destination. It is worth noting that this message comes with a code that further explains the nature of the problem.

2.  __Time Exceeded:__ This message is sent when a packet takes too long to reach it's target (when time to live (TTL) has expired). 

3.  __Parameter Problem:__ This message is sent when there is a problem with the packet being sent and the packet cant be delivered without corrections.

### Informational Messages

1.  __Echo Request & Echo Reply:__ These messages are used to check the network availability. You can see it as the Echo request is saying "hey, are you there?" to the target host and the target host replying "yes, I'm here". These are the messages that _ping_ rely on.

2.  __Timestamp Request and Timestamp Reply:__ These messages measure the transit times across the network.

## ICMP in Action
We already established that ICMP is used for error reporting and diagnostics within a network. Let's explore this with a couple of examples.

Let's say that PC1 wants to send a packet to PC2 in the image below. First the packet reaches router 1. Which finds a matching route and forwards the packet to router 2. Router 2, however, cannot find a route for the packet and therefore drops it. It then sends a _ICMP Destination Unreachable_ message to PC1. **IMAGE**



In a similar manner, if the TTL expires, at Router 2 and the packet is destined for PC2, then Router 2 will return a _ICMP Time Exceeded_ message.

### Ping
Ping uses the Echo Request and Echo Reply messages to assess the accessibility of a host across an IP network. It measures the time for messages to go from the source to the destination and back. Which provides insights to the networks operational status. 

### Traceroute

Traceroute on the other hand uses ICMP Time Exceeded messages to determine the path the packets take to their destination by utilizing the TTL (Time to Live) values. For each router a packet passes through, the TTL is decreased by one until it either reaches it's destination or the value is 0 and the packet is dropped. Traceroute increases the TTL for each packet that is sent until it reaches it's destination. It could look something like this:

    Your Computer ----> [TTL=1] ----> Router 1
    Your Computer <---- [Time Exceeded] <---- Router 1

    Your Computer ----> [TTL=2] ----> Router 1 ----> Router 2
    Your Computer <---- [Time Exceeded] <---- Router 2

    Your Computer ----> [TTL=3] ----> Router 1 ----> Router 2 ----> Router 3
    Your Computer <---- [Time Exceeded] <---- Router 3

    ...


## Security Risks
Like pretty much any protocol, ICMP has it's security risks. Some network administrators completely block all ICMP messages to avoid these security risks. Most of the security risks can, however, be avoided through blocking certain types of messages and correct rules in IDS:es.

 I will briefly discuss a few security risks below.

### ICMP Tunneling
ICMP tunneling is a technique that allows data to be covertly transmitted through a network by encapsulating it within ICMP (Internet Control Message Protocol) packets. This method can be used for legitimate purposes, such as bypassing firewalls for troubleshooting, but it is often associated with malicious activities like data exfiltration or establishing covert communication channels.

A few mitigation techniques to avoid this includes monitoring and analyzing ICMP traffic for unusual patterns and using deep packet inspection (DPI) to detect and block suspicious ICMP packets. 

### Ping Flood
In this attack an attacker sends a very large number of ICMP Echo requests (ping) packets to a target with the intent to overwhelm it and causing a denial of service (DoS).

One way to avoid this is to limit the ICMP traffic allowed on routers and use firewall to prevent too many ICMP packets from reaching the target.

### Ping of Death:
In Ping of Death attack, the attacker sends a malformed or oversized ICMP packet that can crash or harm the target system. 

In general, modern systems are not that vulnerable to this attack, but older systems should ensure that the routers are patched and updated to handle oversized packets correctly.

### Smurf Attack:
A Smurf Attack is a kind of DDoS (Distributed Denial of Service) where an attacker sends a ICMP echo request on a network's broadcast address and changes the source address to the victims IP address. Which causes all devices on the network to send Echo Replies to the victim and overwhelming it.

A simple way to avoid this is to disable ICMP responses to broadcast addresses on network devices.

## General Mitigation Strategies
In general implementing firewall rules to control ICMP traffic, monitor the network traffic and rate limiting will mitigate most of these attacks. It is, as always, important to keep all the network devices updated with the latest security patches to mitigate known vulnerabilities. 

## Deep dive
This was a very brief and basic post on the ICMP. If you want to deep-dive into this I'll provide a few links below. 


[Why you should not block ICMP](https://www.rimscout.com/why-you-should-not-block-icmp/)  
[ICMP: The Good, the Bad, and the Ugly](https://blog.securityevaluators.com/icmp-the-good-the-bad-and-the-ugly-130413e56030 )  
[Basics of ICMP: What You Need to Know](https://orhanergun.net/icmp-guide)  
[The internet control message protocol (ICMP)](https://www.infosecinstitute.com/resources/network-security-101/internet-control-message-protocol-icmp/)  
[ICMP attacks](https://www.infosecinstitute.com/resources/hacking/icmp-attacks/)