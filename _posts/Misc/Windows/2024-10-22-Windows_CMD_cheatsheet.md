---
title: Windows Command Prompt Cheat sheet
description: A list of need-to-know CMD commands along with som nice-to-have!
categories: [Windows, cmd]
tags: [Windows, Tools]
pin: false
---

# General

| Command                         | Description                                                                                                     |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------|
| `help <command>`, `<command> /?` | List available commands along with a brief description of each |
| `more` | Displays output one screen at a time, when output is very long it is useful. <br> `<command> | more` |
| `set`                           | Displays a list of all current environment variables and their values.                                         |
| `set VARIABLE_NAME=value`      | Creates a new environment variable or modifies an existing one. <br> For example: `set MY_VAR=Hello`  <br> **(Session Scope)** |
| `echo %VARIABLE_NAME%`         | Displays the value of a specified environment variable. <br> For example: `echo %MY_VAR%`                   |
| `ver`                          | Displays windows version |
| `systeminfo`                  | Displays operation system configuration information
| `cd`              | Changes the current directory. <br> For example, `cd C:\Users` changes to the Users directory.  |
| `dir`             | Lists the files and directories in the current directory.                                   |
| `mkdir`           | Creates a new directory. <br> For example, `mkdir NewFolder` creates a folder named NewFolder. |
| `rmdir`           | Removes an empty directory. <br> For example, `rmdir OldFolder` removes the OldFolder directory. |
| `del`             | Deletes one or more files. <br> For example, `del file.txt` deletes the file named file.txt.  |
| `erase`           | Delete one or more files. Similar to `del` |
| `copy`            | Copies files from one location to another. <br> For example, `copy file1.txt D:\Backup` copies <br> file1.txt to the Backup folder on D drive. |
| `move`            | Moves files from one location to another. <br> For example, `move file.txt D:\Documents` moves <br> file.txt to the Documents folder on D drive. |
| `type`            | Dump the contents of a textfile on the screen. |
| `copy <from> <to>` | Copy files from one location to another |
| `move`            | Move files from one locatin to another |
| `*`               | Wildcard, For example `*.md` would match all files with the `.md` extension. |
| `cls`             | Clears the command prompt screen.                                                            |
| `exit`            | Exits the command prompt.                                                                    |

### Important Notes

| Note                  | Description                                                                                                     |
|-----------------------|-----------------------------------------------------------------------------------------------------------------|
| **Session Scope**     | Changes made with the `set` command are temporary and only apply to the current <br> command prompt session. They will be lost when the session ends. |
| **Permanent Changes** | Use the System Properties dialog or the `setx` command to make permanent changes <br> to environment variables.       |

## Network Specific

| Command                           | Description                                                                                                     |
|-----------------------------------|-----------------------------------------------------------------------------------------------------------------|
| `ipconfig`                        | Displays the IP address configuration for all network adapters on the computer.             |
| `ping`                            | Tests the connectivity to another network device or website. <br> For example, `ping google.com` checks the connection to Google. |
| `tracert`                         | Traces the network route traversed to reach the target. |
| `nslookup <host>`                 | Look up host using default server. |
| `netstat`                         | Displays protocol statistics and current TCP/IP network connections.        |
| `arp -a`                          | Display the ARP cache  | 
| `route print`                     | Display the local routing table |
| `netsh`                           | A versatile network configuration command. <br> Used for configuring firewall rules, network interfaces, or capturing packets. |

## Process and Task Management

| Command            | Description                                                                                                           |
|--------------------|-----------------------------------------------------------------------------------------------------------------------|
| `tasklist /svc`    | Lists all running processes and their associated services. <br> Useful for detecting suspicious processes or services. |
| `taskkill /f /im`  | Forcibly terminates processes by name or PID. <br> For example, `taskkill /f /im notepad.exe` forcibly kills Notepad.  |
| `schtasks`         | Schedules and manages tasks (like cron jobs). <br> `schtasks /query` lists all tasks.                                 |
| `wmic`             | Accesses Windows Management Instrumentation. <br> `wmic process list` lists all running processes in detail.          |
| `sc`               | Interacts with Windows services. <br> Can start, stop, or query services, e.g., `sc query` shows all running services. |


## Advanced File and Disk Management

| Command            | Description                                                                                                           |
|--------------------|-----------------------------------------------------------------------------------------------------------------------|
| `cipher /w`        | Securely wipes deleted data by overwriting free space on a disk. <br> `cipher /w:C` wipes free space on the C: drive. |
| `icacls`           | Manages file and folder permissions. <br> `icacls file.txt /grant User:F` grants full access to a user for a file.    |

## System Information and Monitoring

| Command            | Description                                                                                                           |
|--------------------|-----------------------------------------------------------------------------------------------------------------------|
| `gpresult`         | Displays Group Policy settings applied to a user or computer. <br> `gpresult /r` shows policies applied to the user.  |
| `wevtutil`         | Interacts with Windows Event Logs. <br> `wevtutil qe System` queries the System event log.                            |
| `getmac`           | Displays the MAC address of the network adapter(s). <br> Useful for identifying devices on the network.               |

## Registry and Configuration Management

| Command            | Description                                                                                                           |
|--------------------|-----------------------------------------------------------------------------------------------------------------------|
| `reg`              | Allows command-line editing of the Windows registry. <br> `reg query HKLM\Software` lists registry entries in that path. |
| `powercfg`         | Configures power settings and generates reports. <br> `powercfg /energy` generates an energy report for diagnostics.  |
