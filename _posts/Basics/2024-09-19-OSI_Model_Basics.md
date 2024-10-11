---
title: Another OSI Model Breakdown
description: A quick breakdown of the OSI Model.
author: Emzzie
date:  2024-09-19 10:00 +0800
categories: [Basics]
tags: [Basics, OSI]
pin: false
---
This is a VERY quick overview of the OSI (**O**pen **S**ystem **I**nterconnection) Model, a well known way of breaking down the computer networking into seven layers.    

For the visual learners out there [here](https://www.youtube.com/watch?v=KHMwhjQrCmo) is an awesome three minute explanation.

Otherwise lets get to it!  

## Layer 1  (The Physical Layer): 
Specifies how the physical communication will take place, which cables are used if any, which voltages etc. 

## Layer 2 (The Data Link Layer): 
This layer handles the node-to-node transfer. There are two layers here, the Media Access Control (MAC) and the Logical Link Control (LLC).    

## Layer 3 (The Network Layer): 
This layer is responsible for packet forwarding, including routing through different routes. Making it possible for data to be sent from one part of the world to another.   

## Layer 4 (The Transport Layer): 
This layer coordinates and transfers data between systems and hosts. Both TCP and UDP act on this level.   

## Layer 5 (The Session Layer): 
This layer makes sure that devices can communicate with each other in a structured manner (in a session). Just like we know the different rules of engagement when having a conversation or when going to a lecture, the sessions created need to take the devices type of communication into consideration.   

## Layer 6 (The Presentation Layer): 
This layer acts like a translator between network formatted data and human readable data. The most common example of this being decryption and encryption of data.   

## Layer 7 (The Application Layer): 
This layer handles what you see on the screen, you see this as text instead of strange jibberish