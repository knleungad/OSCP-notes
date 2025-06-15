### Local enumeration on compromised machine
Like the steps in [[Attacking a Public Machine]]

Additional Information to pay attention to:
- Any AV
- Network interfaces
- DNS cache -> identify internal machines
	- can guess the function of the cached servers from their names
		- E.g. dcsrv1.beyond.com is likely the domain controller

Store identified internal machines in tracker:
- ip(s)
- is it external/internal/both
- name

### Enumerate AD environment and its objects

Use BloodHound with SharpHound.ps1 collected (see [[Active Directory - Automated Enumeration]])

Information to obtain:
- computers
- domain users
	- if user is displayed as an SID, it means it is from local machine -> check if it is local admin (RID 500) (see [[Enumerating Windows]])
- domain admins
- domain groups
- GPOs
	- click on the node of a user > Node info > Outbound Object Control
- vectors for privilege escalation or lateral movement

### Enumerate services and sessions

Query all active sessions in BloodHound (see [[Active Directory - Automated Enumeration]])
- if we get privileged access on a machine, we can potentially extract the NTLM hash of users with active session on that machine

Identify Kerberoastable users in domain (in BloodHound)
- *krbtgt* user is not Kerberoastable. Skip it in the context of Kerberoasting

Inspect Kerberoastable user's node (*Node info* menu). Identify SPN, which we can access after compromising plaintext password of user

#### Set up a SOCKS5 proxy to perform enumeration

crackmapexec (for Windows)
- enumerate smb shares and permissions for compromised user:
  `proxychains -q crackmapexec smb 172.16.6.240-241 172.16.6.254 -u john -d beyond.com -p "dqsTwTpZPn#nL" --shares`
	- If a share has SMB signing set to false, we can potentially form relay attacks if we can force an authentication request

nmap
- enumerate the shares identified by crackmapexec:
  `sudo proxychains -q nmap -sT -oN nmap_servers -Pn -p 21,80,443 172.16.6.240 172.16.6.241 172.16.6.254`
	- must specify `-sT` for TCP scan or it won't work
	- enumerate the commonly-used ports of those services

For browsing http / https ports, better use Chisel for port forwarding as it is more stable
-> browsing to port 80 of localhost is equivalent to browsing to the same port on the target machine
- may need to add the DNS name to `/etc/hosts` on Kali:
  `127.0.0.1    internalsrv1.beyond.com`






