After an nmap scan on a machine with Active Directory (AD), my usual approach involves prioritizing enumeration of key services and identifying potential misconfigurations. Here's how I typically prioritize and proceed:

1. SMB (Port 445): This is often my first point of focus due to its common vulnerabilities. I look for open shares (`smbclient`, `enum4linux`, or `smbmap`), weak permissions on shared files, and potential misconfigurations like Null Sessions or anonymous access. Sometimes, you'll find sensitive information in shared folders or credentials in configuration files.
    
2. LDAP (Port 389/636): I enumerate users and groups via LDAP, especially if **anonymous** bind is allowed. Tools like `ldapsearch` or `enum4linux` help in gathering information about the domain, users, groups, and organizational units. A weak configuration may expose sensitive data, and if you can bind as an authenticated user, it opens more enumeration opportunities.
    
3. Kerberos (Port 88): I use tools like `kerbrute` or `GetNPUsers.py` from `Impacket` to perform username enumeration and check for accounts with no pre-authentication required (AS-REP roasting). I also enumerate service principal names (SPNs) using `GetUserSPNs.py` for Kerberoasting.
    
4. RPC (Port 135/139): I attempt to enumerate users and shares using tools like `rpcclient`. Sometimes, this provides a way to query information from the domain controller, including user accounts and group memberships.
    
5. DNS (Port 53): Check if zone transfers are allowed (`dig axfr` or `host -l`). Misconfigured DNS can leak valuable information about the domain, hosts, and network architecture.
    

Tricks that often work:

- Password Spraying: Once I have usernames from LDAP/Kerberos, I often test weak or common passwords using tools like `crackmapexec`.
    
- Privilege Escalation: If I have a low-privileged account, I focus on privilege escalation, particularly checking for misconfigurations in GPOs, weak permissions on AD objects, or vulnerable services.
    
- Group Policy Preferences: Check for GPP password vulnerabilities (stored in SYSVOL with reversible encryption).
    
- Exploiting SMB vulnerabilities: Like EternalBlue or any open SMB shares with misconfigured permissions.

Source: https://www.reddit.com/r/oscp/comments/1fa9qpa/which_methodology_you_trust_the_most_on_foothold/



## bloodhound.py
location: `~/Documents/Tools/bloodhound.py`

Use from Kali:
`python3 bloodhound.py --dns-tcp -ns 10.10.11.174 -d domainname.htb -u username -p 'password' -c all`
- `-c all`: add all collections (for more info)

## ldapsearch

Use from Kali

Get the DN:
`ldapsearch -H LDAP://support.htb -x -s base namingcontexts`
- The DN is in the format `dc=[],dc=[]`

`ldapsearch -H LDAP://support.htb -D 'ldap@support.htb' -w 'password' -b 'dc=support,dc=htb'`
- `-h`: specify hostname
- `-D`: specify user
- `-w`: password of user
- `-b`: specify the DN, e.g. hostname is support.htb, becomes `dc=support,dc=htb` (LDAP format)

Anonymous authentication:
`ldapsearch -H LDAP://support.htb -x -b 'dc=support,dc=htb'`

Perform query:
`ldapsearch -H LDAP://support.htb -x -b 'dc=support,dc=htb' '[query here]'`
- Query for users: `(objectClass=Person)`

Keywords to search in output:
- `memberof`: shows the groups
- `CN=[username]`: find a specific user
- `objectClass: Person`: show the users
- `sAMAccountName`: the username

Interesting fields: 
- info (maybe they put the password in there)
- description (maybe they put the password in there)

## nxc/crackmapexec

Test credentials to make sure they're correct:
`nxc smb [target ip] -u [username] -p [password]`

Enumerate SMB shares:
`nxc smb [target ip] -u [username] -p [password] --shares`

Enumerate SMB users:
`nxc smb [target ip] -u [username] -p [password] --users`

## rpcclient

Anonymous login:
`rpcclient -U '' -N [target ip]`

After login:
`enumdomusers`: get domain users

## GetADUsers.py
from Impacket

Get list of active users in domain:
`GetADUsers.py -all active.htb/[username] -dc-ip 10.10.10.100`

-> next step: Kerberoast
## GetNPUsers.py
from Impacket (no need to provide credentials)

`GetNPUsers.py -dc-ip 10.10.10.161 -request 'htb.local/'`


## certipy
For enumerating Active Directory Certificate Service (ADCS) vulnerability

Location: `~/Documents/Tools/certipy/`

`certipy find -dc-ip [target ip] -vulnerable -u [username] -p [password] -stdout`
- look for vulnerable templates under `Certificate Templates` output (result is user)
	- if there is vulnerability, search the code in Google for exploit 
- can also run without the `-vulnerable` to view all templates
	- look out for which non-default groups/users have `Enrollment Rights` permission under a template - we may be able to exploit if we are in that group
- instead of `-p` for password, can also do `-hashes` to use hash
	- e.g. NTLM hash: `-hashes :[NTLM hash here]`

Abuse GenericWrite privilege on user
`certipy shadow auto -target certified.htb -dc-ip 10.10.11.41 -u judith.mader -p judith09 -account management_svc`
- `-account`: specify the target user
- the command adds a certificate to the target user (used for authentication), then grabs the TGT which gives us NTLM hash of the target user