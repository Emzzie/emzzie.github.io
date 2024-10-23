---
title: Active Directory - Some common terms
description: A nice-to-have post with some common abbreviations and terms in Active Directory
author: Emzzie
date:  2024-10-22 10:00 +0800
categories: [Misc, ActiveDirectory, Windows]
tags: [Basics, ActiveDirectory]
pin: false
---

I could have made a walkthrough of the THM-lab [Active Directory Basics](https://tryhackme.com/r/room/winadbasics) but I wanted this to be a little broader, and keep building on this glossary and include some of the other terms related to Active Directory as I encounter them to make it a little bit more useful as a quick lookup page.

## General

### Active Directory (AD)
Centralized directory service for managing users, computers, and resources. Advantages: centralized identity management and security policy control.

### Domain Controller (DC)
Server hosting the AD database, responsible for authenticating users, enforcing security policies, and managing access to resources within a Windows domain.

### Active Directory Domain Services (AD DS)
The core AD service that stores directory data and manages communication between users and domains. This service acts as a catalogue that holds the information of all of the "objects" that exist on your network. 

### Object
A distinct entity within the directory that represents a resource, such as a user, group, computer, printer, or organizational unit (OU).
Key aspects of an object include:
* __Identity__: Each object has a unique identity making it distinctly recognized within the directory.
* __Attributes__: Attributes provide information about the object, for example username, email address, phone number and group membership
* __Classes__: Each object belongs to a specific class which defines the object and the attributes that is available to it.
* __Access Control__: Objects can have security descriptors that define permissions and access rights, determining who can perform certain actions on the object. 

### Common objects
#### Users 
The most common object types in Active Directory. Users are one of the objects known as _security principals_, they can be authenticated by the domain and can be assigned privileges over
* __resources__: like files or printers. There are in general two types of entities:
* __people__: The most common representation of a user is a person like employees.
* __services__: You can also define users to be services, every single service requires a user to run, but service users are different from regular users and they will only have privileges needed to run their specific service.

#### Machines/Computer Objects
Represent computers that are joined to the AD domain. All machine accounts will have the computer name followed by a dollar sign. I.e `DC01` -> `DC01$`.

#### Group Objects
* **Security Groups**
Used for assigning permissions to resources. 
Most common groups:

| Group Name           | Description                                                                                     |
|----------------------|-------------------------------------------------------------------------------------------------|
| Domain Admins        | Users of this group have administrative privileges over the entire domain. <br> By default, they can administer any computer on the domain, including the DCs. |
| Server Operators     | Users in this group can administer Domain Controllers. <br> They cannot change any administrative group memberships. |
| Backup Operators     | Users in this group are allowed to access any file, ignoring their permissions. <br> They are used to perform backups of data on computers. |
| Account Operators    | Users in this group can create or modify other accounts in the domain.                        |
| Domain Users         | Includes all existing user accounts in the domain.                                            |
| Domain Computers     | Includes all existing computers in the domain.                                               |
| Domain Controllers    | Includes all existing DCs on the domain.                                                     |


* **Distribution Groups**
Used primarily for email distribution lists.

#### Organizational Units (OU)
Containers used to organize users, groups, computers and other OUs. They help delegating administrative control and applying group policies.

#### Shared Folders 
Represent folders that can be accessed by users or groups within the domain

#### Domain Objects
Represents the domain itself, which can contain multiple 

#### Group Policy Objects

**Organizational Units (OUs)**
Containers used to organize users, computers, and other objects within Active Directory. OUs help apply policies and manage resources more efficiently.

**Group Policy Objects (GPOs)**
Collections of settings used to manage and configure the operating systems, applications, and user environments applied to OUs. 

#### Security Groups vs OUs
OUs are handy for **applying policies** to users and computers. 
Security Groups are used to **grant permissions over resources**.


### Forests and trees

#### Tree
A hierarchical arrangement of domains, where child domains inherit the namespace of the parent domain.

#### Forest 
A collection of one or more domain trees, which share a common AD infrastructure and allow for seamless resource sharing.


### Trust Relationships
Logical connections between two domains or forests that allow users in one domain to access resources in another domain.

### LDAP (Lightweight Directory Access Protocol)
A protocol used to access and manage directory information services like AD. It's essential for querying and modifying AD data.

### FSMO Roles (Flexible Single Master Operations)
A set of roles in AD responsible for specific tasks to ensure the directory functions properly. Includes roles like Schema Master, RID Master, and PDC Emulator.
