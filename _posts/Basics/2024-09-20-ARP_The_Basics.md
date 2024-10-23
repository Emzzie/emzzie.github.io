---
title: ARP - The basics
description: Lets have a look at the basics of ARP 
categories: [Basics, Protocols]
tags: [Basics, Protocols]
pin: false
---

The Address Resolution Protocol, or ARP for short, is the protocol that maps the software defined address (Internet Protocol (IP) address) with the physical address (Media Access Control (MAC) address) of devices in a network.

## MAC Addresses
All devices capable of connecting to a network (not necessarily the internet) have a Network Interface Card (NIC) with a unique MAC address assigned by the manufacturer. All devices such as computers, routers, mobile phones and the smart devices you control via your WIFI have a MAC address. These addresses are used by the the Data Link Layer (Layer 2) of the [OSI model]({% post_url Basics/2024-09-19-OSI_Model_Basics %}) and deals directly with connected devices.

## IP Addresses
IP Addresses is what is used to identify hosts over the internet. All hosts connected to the internet have an IP address. These addresses are used by the Network Layer (Layer 3) of the [OSI model]({% post_url Basics/2024-09-19-OSI_Model_Basics %}) and deals (mostly) indirectly with connected devices. In a local network the IP addresses of a device can vary but the MAC-address can't (except through spoofing). 

You can deep dive into the vast world of IP and subnets [here](https:https://www.youtube.com/playlist?list=PLIhvC56v63IKrRHh3gvZZBAGvsvOhwrRF).

## Why do we need both IP Addresses and Mac Addresses?
Note that the addresses are used by different layers. The data link layer (Layer 2) doesn't really care what happens on the network layer (Layer 3). This means it doesn’t care if you’re using IPv4, IPv6, or even another protocol. Layer 2 handles devices using their MAC addresses, while Layer 3 handles routing traffic using IP addresses. So, basically the way Internet works makes it necessary to have both addresses. I warmly recommend [this](https://ine.com/blog/why-do-we-need-both-ip-addresses-and-mac-addresses) blog post if you want to read more on this!

## ARP and The Subnet
Recall how the entire internet is made up from subnets. You probably have WiFi setup at home, with lots of things connected to it, like computers, phones, TVs and other things. These are all part of a subnet that is (hopefully) only accessed by you and your household. The WiFi router is your path to the vast internet. What on earth does this have to do with ARP? 

Well, any time you want to send or receive IP-packets, whether it is to connect to a website or to turn on your smart tv via your phone, you need to know which device that should receive the packet. You will already have the IP-address one way or another (more on this later), but what is the MAC-address to send this data to? This is where ARP comes in. 

ARP is essentially a bridge between a known Layer 3 address and an unknown layer 2 address. Most commonly a known IP address and an unknown MAC address. Note that ARP only works within a subnet. You might wonder how the heck the IP packets get to the destination if ARP only works within a subnet. Remember how the router is your path to the internet? It is a part of your subnet but it is also part of a bigger subnet with other routers. All of those routers have their own MAC addresses.

### ARP: From Request To Response
![IP address lookup](/assets/images/IP_addresses.png){: h="300" w="300" .right }

Let's say you want to browse to Google.com. First off, your computer needs to resolve the domain name, this is done using a DNS (Domain Name System). This gives you the IP address of Google.com, lets say 172.217.12.110. This is where your request to view the page (IP packets) will be sent. Now the layer 3 knows where to send the packets. 

The next step is to figure out if that IP address is within your subnet or not. In this case it isn't, so the request needs to be sent to your default gateway (usually your wifi router), this will be 192.168.0.1, 192.168.1.1 or 192.168.254. Once your computer knows the IP address of Google, it still needs to find a way to send packets to that address within the local network — this is where ARP comes in.

Your computer now takes a look at what is called an __ARP table__ (or ARP Cache) to figure out if it has the MAC-address of the router already stored.

Let's assume that your device doesn't know the MAC address of your router. It will then send an __ARP Request__ to all devices the subnet. This is the way the computer asks "Hey, I want to send data to the device associated with this IP address, what's your MAC-address?". The device that has the IP address will respond to the request with its MAC-address. 

Once the ARP reply is received at your computer the MAC address is then stored in the ARP table and associated with the IP address. Your computer now has a MAC address to use to send the IP packets to the correct device. 

> For the fun of it, let's make an analogy. Imagine you just sold your old computer to a man named Bob, who lives on NetworkStreet (the computer represents IP packets). You need to deliver the computer to his house.
>1. __Checking Your Neighborhood__: First, you check if NetworkStreet is in your neighborhood (checking if the IP address is in your subnet). Unfortunately, you don’t live near NetworkStreet (it’s not in your subnet).
>2. __Finding the Post Office:__ Since you can’t deliver it directly, you need to go to the local post office (the router). However, you’re new to the neighborhood and don’t know where the post office is (the MAC address of the router is not in your ARP table).
>3. __Asking for Directions:__ Desperate, you go outside and shout, “Where is the post office?” (sending an ARP request).
>4. __Getting a Response:__ The post office hears you and calmly replies with its address (ARP reply). After that, you write down the address in your notebook (ARP table) so that next time, you won’t need to ask again
>5. __Delivering the Package:__ Now that you know where the post office is, you can go there and send your computer to Bob.

## Key Takeaways

* __IP to MAC Mapping:__ ARP is used to map IP addresses to a MAC address within a _local_ network. 
* __ARP Requests and Replies:__ When a device needs to communicate to another device on the same network but doesn't know it's MAC address, it sends an __ARP request__. The device with the matching IP address responds with an __ARP reply__ and provides his MAC address.
* __ARP Table:__ The devices maintain an ARP table (or cache) to speed up future communications by reducing the number of ARP requests.

## Dive Deeper 
I strongly recommend having a look at the room [Nmap Live Host Discovery](https://tryhackme.com/r/room/nmap01), task 2 you allows you to actively practice using ARP for host discovery, helping you understand how these concepts apply in real-world scenarios. It's free for all, so just create an account and get testing! I have provided a writeup for this room [here]({% post_url THM_Walkthrough/Network_Security_Module/2024-09-24-THM_LiveHostDiscovery %})

Jeremy's IT lab has a very good video on ARP, check it out [here](https://www.youtube.com/watch?v=k3oda32jmWY)

