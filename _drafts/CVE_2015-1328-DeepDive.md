---
title: CVE 2015-1328 Deep dive
description: Let's explore the OverlayFS filesystem vulnerability exploited in this CVE
categories: [Walkthrough THM]
tags: [Nmap, Tools]
pin: false
---


**DRAFT**



Here’s a draft for your **CVE-2015-1328** deep dive blog post. I'll cover all the key areas you want to focus on, explaining the vulnerability, the exploit's technical details, and how it works. Let me know if you'd like any adjustments!

---

## CVE-2015-1328 Deep Dive: A Detailed Look into Linux Kernel Privilege Escalation

### Introduction
In 2015, a critical vulnerability in the Linux kernel was discovered, allowing local users to escalate their privileges to root. **CVE-2015-1328** affects the OverlayFS file system, a widely-used file system used for containerization and other layering purposes. The exploit leverages improper handling of user namespaces and OverlayFS to gain root privileges. In this post, we will break down the exploit, explore the technical details, and explain how the attack works.

### 1. Technical Background

Before diving into the specifics of **CVE-2015-1328**, it's crucial to understand a few key concepts:

#### OverlayFS
OverlayFS is a union mount file system that layers one file system on top of another. It allows changes to be written to the upper layer while leaving the lower layer untouched. OverlayFS is particularly useful in containers where you want to make isolated changes without affecting the host system.

https://www.tuxera.com/blog/linux-data-integrity-overlayfs/
https://www.youtube.com/watch?v=DfENwtNRlD4

#### Linux Namespaces
Namespaces in Linux are a feature that provides isolation of system resources between processes. The key idea is that processes within a namespace cannot see or affect processes outside of it. The exploit abuses the **user namespace** and **mount namespace**.

- **User Namespace**: Provides isolation for user and group IDs. It allows a process to have root privileges inside the namespace while maintaining its original non-root identity outside it.
- **Mount Namespace**: Isolates the set of file system mount points. This allows processes to create their own mount points without affecting the global file system.

In **CVE-2015-1328**, improper handling of the OverlayFS mount operation in the context of these namespaces creates the vulnerability.

### 2. Vulnerability Overview

The core of this vulnerability lies in how the Linux kernel handles **OverlayFS mount operations** within the mount namespace. Specifically, it does not properly check permissions for certain mount operations when used with user namespaces. This leads to a situation where unprivileged users can manipulate OverlayFS mounts and escape the restrictions of user namespaces, effectively bypassing security measures and gaining root privileges.

The flaw stems from missing permission checks when mounting OverlayFS, particularly when an attacker has control over what gets mounted. By manipulating the `upperdir` and `workdir` in OverlayFS, an attacker can trick the system into writing files like `/etc/ld.so.preload`, which is a key part of this exploit.

Yes, the **OverlayFS** is mounted in the **new mount namespace**, and this is a key reason why the exploit works.

### Why It Works:

1. **Mount Namespace Isolation**: 
   When the attacker creates a new mount namespace using `unshare(CLONE_NEWNS)`, they gain a private view of the file system. This allows them to manipulate mount points within that namespace without affecting the global mount points outside it. Essentially, whatever the attacker does in the new mount namespace is isolated and invisible to other processes on the system (outside of the namespace).

2. **Permission Bypass**:
   Normally, mounting file systems like OverlayFS would require root privileges. However, because the exploit runs in a **user namespace** (created using `unshare(CLONE_NEWUSER)`), the attacker can run certain operations with root privileges **within the namespace**, even though they are not root in the global system.

3. **Improper Permission Checks in OverlayFS**:
   The core flaw of **CVE-2015-1328** is that the kernel does not properly check permissions when mounting **OverlayFS** within the user namespace. This allows an unprivileged user to perform mount operations that should typically be restricted to root. The attacker can then control the upper and lower layers of the OverlayFS mount in their private namespace, leading to the ability to write files in sensitive locations (like `/etc`).

4. **Manipulating the File System in Isolation**:
   In the new mount namespace, the attacker mounts **OverlayFS** and controls the lower directory (a critical system path like `/proc/sys/kernel` or `/sys/kernel/security`). At the same time, they control the upper directory (an attacker-controlled directory such as `/tmp/ns_sploit/upper`). 

   This mount manipulation is key. Because it’s happening in a private namespace, the attacker is able to overlay the system’s real file system with their own controlled view. This makes it possible to hide their changes from the real system while performing operations like modifying files such as `/etc/ld.so.preload`, which will affect processes outside the namespace once the exploit completes.

### Critical Moment: Escaping the Namespace

Once the attacker has mounted OverlayFS in the new mount namespace and injected their malicious shared library (via `/etc/ld.so.preload`), they execute code (like spawning a root shell). At this point, the changes made inside the namespace start affecting the system globally. The preloaded malicious library now runs in the context of processes in the global namespace, allowing the attacker to escalate privileges to root **outside the namespace**.

### In Summary:

- **OverlayFS is mounted in the new mount namespace** to avoid affecting the system-wide file system.
- The **exploit works** because the kernel does not properly enforce permission checks when mounting **OverlayFS** inside a user/mount namespace.
- This combination of namespaces and file system manipulation allows the attacker to isolate their actions, manipulate critical files, and then escalate privileges once they escape the namespace.

By mounting **OverlayFS** in the new mount namespace, the attacker bypasses normal permission checks and escalates privileges on the host system, making this vulnerability highly dangerous.

### 3. Detailed Code Analysis

Let’s look at the core exploit code, step by step.

#### Hijacking `getuid()` to Gain Root Privileges
In the exploit, a custom shared library overrides the `getuid()` function to escalate privileges. Here's a breakdown of that code:

```c
uid_t getuid(void) {
    _real_getuid = (uid_t (*)(void)) dlsym((void *) -1, "getuid");
    readlink("/proc/self/exe", path, sizeof(path));
    if (geteuid() == 0 && !strcmp(path, "/bin/su")) {
        unlink("/etc/ld.so.preload");
        unlink("/tmp/ofs-lib.so");
        setresuid(0, 0, 0);
        setresgid(0, 0, 0);
        execle("/bin/sh", "sh", "-i", NULL);
    }
    return _real_getuid();
}
```

**What’s Happening Here?**
- This function overrides the legitimate `getuid()` function used to check a user's identity. 
- It first stores the reference to the original `getuid()` using `dlsym()` so it can be called later.
- The key logic is in the check `geteuid() == 0`, which checks if the effective user ID is **root**. It only proceeds if this condition is true and the running executable is `/bin/su`.
- Once the process is running as root, the exploit removes files like `/etc/ld.so.preload`, which is used by the dynamic linker to preload libraries for all executed programs. This is often manipulated in privilege escalation attacks.

#### Escalating Privileges with OverlayFS

The exploit then uses OverlayFS to overwrite critical system files. Here’s how it works:

```c
if (mount("overlay", "/tmp/ns_sploit/o", "overlay", MS_MGC_VAL, "lowerdir=/proc/sys/kernel,upperdir=/tmp/ns_sploit/upper") != 0) {
    // If the mount fails, it tries another method using AppArmor
    if (mount("overlay", "/tmp/ns_sploit/o", "overlay", MS_MGC_VAL, "lowerdir=/sys/kernel/security/apparmor,upperdir=/tmp/ns_sploit/upper,workdir=/tmp/ns_sploit/work") != 0) {
        fprintf(stderr, "no FS_USERNS_MOUNT for overlayfs on this kernel\n");
        exit(-1);
    }
}
```

**Mounting Trick**
- The attacker mounts a new file system using OverlayFS, with the **lower directory** pointing to a critical kernel directory (e.g., `/proc/sys/kernel`) and the **upper directory** pointing to an attacker-controlled directory (`/tmp/ns_sploit/upper`).
- By manipulating the mount, the attacker gets control over the kernel's view of file systems and can inject malicious content.

### 4. Exploitation Process: Step-by-Step Walkthrough

Here’s how the exploit works step-by-step:

1. **Namespace Trick**: The attacker creates a new user namespace and mount namespace using the `unshare()` and `clone()` system calls.
   
2. **OverlayFS Manipulation**: The attacker mounts OverlayFS with a controlled upper directory. This allows the exploit to write to critical system directories (e.g., `/etc`) without normal permissions.

3. **Writing to `/etc/ld.so.preload`**: The attacker gains the ability to write to `/etc/ld.so.preload`, a file that the system uses to preload shared libraries. By adding a malicious shared library here, every process executed on the system will load the attacker’s code, granting them full control.

4. **Root Shell**: Finally, the exploit spawns a root shell by overriding `getuid()` and running `/bin/sh` with root privileges.

### 5. Mitigations and Patches

After the vulnerability was discovered, Linux kernel maintainers quickly released patches to address the issue:

- **Proper Permission Checks**: The kernel was updated to include proper permission checks when mounting OverlayFS inside user namespaces.
- **Namespace Restrictions**: Kernel updates also introduced stricter restrictions on how unprivileged users can interact with namespaces.

For users, upgrading to a kernel version that includes the patch for **CVE-2015-1328** is critical.

### 6. Real-World Impact

This vulnerability was highly impactful for systems using Linux kernel versions without the patch. Systems that relied on containers or used namespaces for isolation were particularly vulnerable.

The ability to bypass privilege boundaries using user and mount namespaces opened the door for attackers to gain root access on systems they otherwise would not have had permission to control.

### 7. Related Vulnerabilities

This vulnerability is part of a larger class of **namespace-related vulnerabilities** in the Linux kernel. In recent years, there have been multiple instances where Linux namespaces have been exploited for privilege escalation, particularly in container environments. Understanding how these exploits work provides valuable insight into the delicate nature of namespace isolation.

### Conclusion

**CVE-2015-1328** is a prime example of how seemingly minor oversights in permission handling can lead to critical vulnerabilities. The misuse of OverlayFS within user namespaces gave attackers a pathway to escalate privileges and gain root access. By understanding the intricacies of this vulnerability, we can better appreciate the importance of rigorous security checks in kernel code and the ongoing challenges in securing containerized and isolated environments.

If you're a security researcher or just interested in learning more about Linux kernel exploits, CVE-2015-1328 offers a fascinating case study in privilege escalation.

### References

- [Linux Kernel Patch for CVE-2015-1328](#)
- [OverlayFS Documentation](#)
- [Understanding Linux Namespaces](#)

---

This draft provides a comprehensive and detailed overview of **CVE-2015-1328**. Each section is designed to help readers understand the technical details, step-by-step exploitation, and the broader implications of the vulnerability. Let me know if you'd like to tweak any specific part or add more details!