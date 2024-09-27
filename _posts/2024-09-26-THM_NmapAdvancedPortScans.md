---
title: TryHackMe - Nmap Advanced Port Scans
description: A walkthrough of TryHackMe's room "Advanced Port Scans".
categories: [Walkthrough THM, Network Security Module]
tags: [Nmap, Tools]
pin: false
---

Hey there! 
Welcome to this walkthrough of the room "Nmap Advanced Port Scans"!

This is the third room in the Nmap series on TryHackMe. 

## Task 1: Introduction
In the first room of the series,[Live Host Discovery]({% post_url 2024-09-24-THM_LiveHostDiscovery%}), you learned all about host discovery. Then you explored the basic scans used by Nmap in [Basic Port Scans]({% post_url 2024-09-25-THM_NmapBasicPortScans%}). Now it's time to briefly discover a few of the more advanced scans along with spoofing IP and MAC, Decoy Scans, fragmented packets and Zombie Scan. 

## Task 2: TCP Null Scan, FIN Scan and Xmas Scan
On the man pages of port scanning on [Nmap](https://nmap.org/book/man-port-scanning-techniques.html) you will find the following on these three scans:

> These three scan types (even more are possible with the --scanflags option described in the next section) exploit a subtle loophole in the TCP RFC to differentiate between open and closed ports. Page 65 of RFC 793 says that “if the [destination] port state is CLOSED .... an incoming segment not containing a RST causes a RST to be sent in response.” Then the next page discusses packets sent to open ports without the SYN, RST, or ACK bits set, stating that: “you are unlikely to get here, but if you do, drop the segment, and return.”
>
> When scanning systems compliant with this RFC text, any packet not containing SYN, RST, or ACK bits will result in a returned RST if the port is closed and no response at all if the port is open. As long as none of those three bits are included, any combination of the other three (FIN, PSH, and URG) are OK. Nmap exploits this with three scan types:
>
> Null scan (-sN): Does not set any bits (TCP flag header is 0)
>
> FIN scan (-sF): Sets just the TCP FIN bit.
>
> Xmas scan (-sX): Sets the FIN, PSH, and URG flags, lighting the packet up like a Christmas tree.

In summary, for all these scans, we expect a RST response if the port is closed and no response if the port is open. There is, however, no way of knowing if the port is actually closed or if there is a firewall dropping the packets without sending an RST. Thus, all Namp knows about the port is that it is either open or filtered resulting in the status `open|closed`. 

---

**Q: In a null scan, how many flags are set to 1?**
<details><summary>Answer</summary>
0
</details>

---

**Q: In a FIN scan, how many flags are set to 1?**
<details><summary>Answer</summary>
1
</details>

---

**Q: In a Xmas scan, how many flags are set to 1?**
<details><summary>Answer</summary>
3
</details>

---

**Q: Start the VM and load the AttackBox. Once both are ready, open the terminal on the AttackBox and use nmap to launch a FIN scan against the target VM. How many ports appear as open|filtered?**
<details><summary>Answer</summary>
9
</details>

---

**Q: Repeat your scan launching a null scan against the target VM. How many ports appear as open|filtered?**
<details><summary>Answer</summary>
9
</details>

## Task 3: TCP Maimon Scan

The TCP Maimon Scan, `-sM`, is a network scan that exploits a design flaw in the implementation of the TCP/IP stack of older versions of the Unix-like operating system BSD (Berkeley Software Distribution). Instead of responding with an RST packet to the combination of FIN and ACK flags set simultaneously, as specified by RFC 793, these systems would drop the packet if the port was open, in turn revealing the port’s state.

> Note that this bug has been corrected in later versions of the BSD-based systems to comply with the standard
{: .prompt-info}

---

**Q: In the Maimon scan, how many flags are set?**
<details><summary>Answer</summary>
2
</details>

## Task 4: TCP ACK, Window, and Custom Scan

### TCP ACK Scan
The TCP ACK scan, `-sA`. sends a TCP packet with the ACK flag set. If the port is unfiltered, the target will return an RST packet regardless of whether the port is open or closed. If the port is filtered, there may be no response or an ICMP error message. This scan is useful for determining the presence of a firewall and identifying which ports it protects. 
> Just because a firewall isn't blocking a port doesn't neccessarily mean that there is a service listening on that port.
{: .prompt-info}

### Window Scan
The Window scan, `-sW`, is similar to the TCP ACK Scan in that it sets the ACK flag and expect a RST packet in response. However, the Window Scan examines the TCP Window field of the returned RST packets infer whether the port is open or closed.

### Custom Scan
You can experiment with any TCP flag combination by using the `--scanflags` option. Just add the flag acronyms you want to include as one big word. For example FIN and RST flag would be `--scanflags FINRST` all flags set would be `--scanflags URGACKPSHRSTSYNFIN`... 

---

**Q: In TCP Window scan, how many flags are set?**
<details><summary>Answer</summary>
1
</details>

---

**Q: You decided to experiment with a custom TCP scan that has the reset flag set. What would you add after `--scanflags?`**
<details><summary>Answer</summary>
RST
</details>

---

**Q: [...] use Nmap to launch an ACK scan against the target VM. How many ports appear unfiltered?**
> It might take some time for the machine to start, so if you don't get any unfiltered ports try again after a few minutes.
{: .prompt-info}
<details><summary>Answer</summary>
4
</details>

---

**Q: What is the new port number that appeared?**
<details><summary>Answer</summary>
443
</details>

---

**Q: Is there any service behind the newly discovered port number? (Y/N)**
> Hint: What kind of service does Nmap think it is? How could you test that?
{: .prompt-info}
<details><summary>Answer</summary>
N
</details>

## Task 5: Spoofing and Decoys
It is possible to scan a target system using a spoofed IP address, but this is only reliable if you can guarantee capturing the responses. This can be achieved through proxy rules or by using tools like Wireshark.

The attacker would use `nmap -S SPOOFED_IP MACHINE_IP` to target the MACHINE_IP with the sender set to SPOOFED_IP. However, to ensure proper functionality, you should specify the network interface (e.g., `eth0` for Ethernet or `wlan0` for a wireless interface). You can identify your network interfaces using commands like `ifconfig` or `ip addr show`. Additionally, explicitly disable the ping scan using `-Pn`. The resulting command would be `nmap -e NET_INTERFACE -Pn -S SPOOFED_IP MACHINE_IP`. 

While on the same subnet as the target machine, you would also be able to spoof the MAC address using `--spoof-mac SPOOFED_MAC`. 

As you might have seen, spoofing only works in certain situations, therefore the attacker might decide to use decoys instead. When using decoys nmap makes it appear like the scan comes from many different IP addresses and includes the attacker IP in this range of decoy IPs.

You would launch a decoy scan by specifying a specific or random IP address after `-D` and your own IP address after `ME`, you can also assign IP addresses randomly using `RND`. This would look something like this: `nmap -D 10.10.0.1,10.10.0.2,RND,RND,ME MACHINE_IP`

---
What do you need to add to the command `sudo nmap MACHINE_IP` to make the scan appear as if coming from the source IP address `10.10.10.11` instead of your IP address?
**Q: **
<details><summary>Answer</summary>
-S 10.10.10.11
</details>

---

**Q: What do you need to add to the command `sudo nmap MACHINE_IP` to make the scan appear as if coming from the source IP addresses `10.10.20.21` and `10.10.20.28` in addition to your IP address?**
> hint: Don't include your own IP only the command to include it.
{: .prompt-info}
<details><summary>Answer</summary>
-D 10.10.20.21,10.10.20.28,ME
</details>

## Task6: Fragmented Packets
Two common network security applicances include Firewalls and IDS (Intrusion Detection System). 

### Firewalls
Firewalls have the main function of controlling packet flow into a network. It either permits packets or blocks them. A firewall at the least inspects the IP header and the Layer 4 headers like TCP and UDP headers. The more sophisticated firewalls also try to examine the data carried by the individual packets.

### IDS
In contrast to firewalls, which control the packet flow, Intrusion Detection Systems (IDS) only monitor network packets and raise an alarm if there is anything out of the ordinary, such as unusual patterns or content signatures. IDS performs a thorough check on data received at the transport layer and tries to match it to known malicious patterns.

### Fragmented Packets
Fragmenting packets can make it less likely for traditional firewalls and IDS to to detect the nmap activity, although it is not guaranteed. Nmap provides the option `-f` to fragment packets into 8 bytes, `-ff` will split it into 16 byte fragments instead (reducing the number of fragments) and `--mtu` (Maximum Transmission Unit) allows you to set another default value (should be a multiple of 8).

Firewalls and IDS struggle with IP packet fragments. While they could reassemble the packets, this requires extra resources. Additionally, fragments might take different paths which makes reassembly difficult or impossible. Due to these challenges, some filters ignore all fragments, while others automatically pass all but the first fragment. The IDS can have problems if the first fragment isn’t long enough to contain the entire TCP header, or if the second packet partially overwrites it. This could cause false alarms or lead to missing a potential threat. Although fewer filtering devices have these issues now, it’s still worth trying.

---

**Q: If the TCP segment has a size of 64, and -ff option is being used, how many IP fragments will you get?**
<details><summary>Answer</summary>
4, (64 bytes divided into packets of length 16, 64/16=4)
</details>

## Task 7: Idle/Zombie Scan
In the Idle scan, you can scan a target without sending a single packet from your own IP address, using an Idle host ("zombie host").
This can be done using the following three steps:
1. Get the zombie's IP ID (for example by sending a SYN/ACK to it) and note it.
2. Send a SYN packet to a TCP port on the target host to make it look like the zombie-host wants to open communication. The target host sends a SYN/ACK to the zombie host if hte port is open. The zombie host, which doesn't expect this SYN/ACK, returns a RST, and in the process increases its IP ID.
3. Send a SYN/ACK to the zombie host again and note if the IP id was increased by 1 (there was no request from the target host, indicating a closed port) or by 2 (there was a request form the target host, indicating an open port).

You can run an idle scan using `nmap -sI ZOMBIE_IP MACHINE_IP`, where `ZOMBIE_IP` is the IP address of the idle host (zombie).

There is, however no way for nmap to know if the port is actually closed or filtered, and therefore can only give the status `closed|filtered`

---

**Q: You discovered a rarely-used network printer with the IP address 10.10.5.5, and you decide to use it as a zombie in your idle scan. What argument should you add to your Nmap command?**
<details><summary>Answer</summary>
-sI 10.10.5.5
</details>

## Task 8: Getting More Details

You might consider adding `--reason` if you want Nmap to provide more details regarding its reasoning and conclusions. Consider the two scans below to the system; however, the latter adds `--reason`.

```terminal
pentester@TryHackMe$ sudo nmap -sS 10.10.252.27

Starting Nmap 7.60 ( https://nmap.org ) at 2021-08-30 10:39 BST
Nmap scan report for ip-10-10-252-27.eu-west-1.compute.internal (10.10.252.27)
Host is up (0.0020s latency).
Not shown: 994 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
110/tcp open  pop3
111/tcp open  rpcbind
143/tcp open  imap
MAC Address: 02:45:BF:8A:2D:6B (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.60 seconds
```

```terminal
pentester@TryHackMe$ sudo nmap -sS --reason 10.10.252.27

Starting Nmap 7.60 ( https://nmap.org ) at 2021-08-30 10:40 BST
Nmap scan report for ip-10-10-252-27.eu-west-1.compute.internal (10.10.252.27)
Host is up, received arp-response (0.0020s latency).
Not shown: 994 closed ports
Reason: 994 resets
PORT    STATE SERVICE REASON
22/tcp  open  ssh     syn-ack ttl 64
25/tcp  open  smtp    syn-ack ttl 64
80/tcp  open  http    syn-ack ttl 64
110/tcp open  pop3    syn-ack ttl 64
111/tcp open  rpcbind syn-ack ttl 64
143/tcp open  imap    syn-ack ttl 64
MAC Address: 02:45:BF:8A:2D:6B (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.59 seconds
```

Let's try to get another reason, here I've redone the scan in task 4 with the `--reason` flag set, we see that it received a reset (RST) and that is why it was deemed unfiltered. Which is wat we would expect in that situation.

```terminal
└─$ sudo nmap -sA 10.10.107.134 --reason
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-26 08:29 EDT
Nmap scan report for 10.10.107.134
Host is up, received reset ttl 63 (0.15s latency).
Not shown: 982 filtered tcp ports (no-response), 14 filtered tcp ports (admin-prohibited)
PORT    STATE      SERVICE REASON
22/tcp  unfiltered ssh     reset ttl 63
25/tcp  unfiltered smtp    reset ttl 63
80/tcp  unfiltered http    reset ttl 63
443/tcp unfiltered https   reset ttl 63

Nmap done: 1 IP address (1 host up) scanned in 11.71 seconds
```

---

**Q: Launch the AttackBox if you haven't done so already. After you make sure that you have terminated the VM from Task 4, start the VM for this task. Wait for it to load completely, then open the terminal on the AttackBox and use Nmap with `nmap -sS -F --reason 10.10.107.134` to scan the VM. What is the reason provided for the stated port(s) being open?**
<details><summary>Answer</summary>
syn-ack
</details>

---

## Task 9: Summary
This room covered many different types of scans and extra settings. They might be useful in some cases and it is good to know that they exist!

