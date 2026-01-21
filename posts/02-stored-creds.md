# Privilege Escalation – Stored Passwords (Configuration Files)

##  Enumeration

While enumerating the system, an **OpenVPN configuration file** was found in the user’s home directory:

```bash
cat /home/user/myvpn.ovpn
```
Output:
```bash
text
Copy code
client
dev tun
proto udp
remote 10.10.10.10 1194
resolv-retry infinite
nobind
persist-key
persist-tun
auth-user-pass /etc/openvpn/auth.txt
comp-lzo
verb 1
reneg-sec 0
```
## Analysis
The configuration file references an external file for authentication:

```bash
auth-user-pass /etc/openvpn/auth.txt
```
This indicates that credentials may be stored in plaintext inside the referenced file.

## Credential Discovery
Reading the authentication file revealed stored credentials:

```bash
cat /etc/openvpn/auth.txt
```
Output:
 ```bash
user
password321
```
 Result
Credentials were found stored in cleartext inside a configuration-related file.

Such credentials can potentially be reused for:

* Privilege escalation
* Lateral movement
* Access to other services (SSH, VPN, sudo, etc.)

## Mitigation
1.) Avoid storing credentials in plaintext configuration files

2.) Use secure credential storage mechanisms (e.g., keyrings, vaults)

3.) Restrict file permissions on sensitive configuration files:

```bash
chmod 600 /etc/openvpn/auth.txt

```
4.) Regularly audit system files for exposed credentials

5.) Remove unused or outdated configuration files


