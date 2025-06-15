---
tags: Nmap
---
# DNS Enumeration

Common DNS record types:
- NS: Nameserver records contain the name of the authoritative servers hosting the DNS records for a domain
- A: Host record contains the IPv4 address of a hostname
- AAAA: contains IPv6 address of a hostname
- MX: Mail Exchange records contain the names of the servers responsible for handling email for the domain. A domain can contain multiple MX records.
- PTR: Pointer records contain the records associated with an IP address (reverse lookup)
- CNAME: Canonical Name records contain aliases for other host records
- TXT: Text records can contain any various data for various purposes (e.g. for domain ownership verification)

`host` command:
- `host www.megacorpone.com`: find IP address of host
- `host -t mx megacorpone.com`: use `-t` option to specify record type
	- Note: for MX records, the lowest priority number server will be used first to forward mail addressed to the domain

DNS bruteforcing for valid hostnames
- `for ip in $(cat wordlist.txt); do host $ip.megacorpone.com; done | grep -v "not found"`
- wordlists can be found in `/usr/share/seclists/Discovery/DNS/`
- look for any range of IP addresses (e.g. `51.222.169.X`)

Scan range of IP addresses with reverse lookup (PTR)
- `for ip in $(seq 200 254); do host 51.222.169.$ip; done | grep -v "not found"`

### Automated DNS enumeration

[DNSRecon](https://github.com/darkoperator/dnsrecon)
- scan main DNS record types for a domain name
	- `dnsrecon -d megacorpone.com -t std`
		- `-d`: domain name
		- `-t`: type of enumeration (`std` is standard)
- subdomain brute force with wordlist
	- `dnsrecon -d megacorpone.com -D ~/list.txt -t brt`

DNSEnum
- subdomain brute force, netranges, reverse lookup on netranges
	- `dnsenum megacorpone.com`

### Windows DNS enumeration

nslookup
- resolve A record
	- `nslookup mail.megacorptwo.com`
- more granular queries
	- `nslookup -type=TXT info.megacorptwo.com 192.168.50.151`
		- format: `nslookup -type=<record type> <host> <DNS server>`
- also available on Linux

# TCP/UDP Port Scanning

### One methodology
1. Scan for ports 80 and 443
2. Run a full port scan in the background
3. After full port scan is complete, further narrow scans

### netcat for basic port scanning
- TCP CONNECT scan
	- `nc -nvv -w 1 -z 192.168.50.152 3388-3390`
		- `-n`: numeric-only IP address, no DNS
		- `-vv`: verbose (two v more verbose than one v)
		- `-w`: specify connection timeout (in seconds)
		- `-z`: zero-I/O mode (scanning only, no sending data)
- UDP scan (send empty UDP packet)
	- `nc -nv -u -z -w 1 192.168.50.149 120-123`
		- `-u`: UDP scan
	- Response received depends on how the application is programmed to respond to an empty UDP packet
		- If response if an **ICMP port unreachable**, it could mean that the port is closed OR filtered by a firewall
		- Note: Firewalls and routers may drop UDP packets, so there may be false positives
		- Note: Using a protocol-specific UDP port scanner may help to obtain more accurate results

### Port scanning with [Nmap](https://nmap.org/)

- Default scan scans the 1000 most popular ports using stealth/SYN scanning (if the user has the required raw socket privileges) or TCP connect scanning (if the user does not have raw socket privileges)

Basic usage
- `nmap 192.168.50.149
- `nmap -p 1-65535 192.168.50.149`
	- `-p`: specify port range

#### TCP scanning

Stealth/SYN scanning
- -> SYN
  SYN-ACK <- (if open)
  scanner does not send final ACK to complete handshake
- `sudo nmap -sS 192.168.50.149`
- Note: Because the TCP three-way handshake is never completed, the information will not be passed to the application layer (so will not appear in application logs). However, modern firewalls may still log it.

TCP connect scanning (takes longer time)
- full three-way handshake is performed
- `nmap -sT 192.168.50.149`

Note: We can infer the underlying OS and role of the target host from the result

#### UDP scaning
- Nmap uses a combination of two different methods:
	- For most ports, the standard "ICMP port unreachable" method by sending an empty packet to a given port is used
	- For common ports, a protocol-specific packet is sent in an attempt to get a response from the application bound to the port
- `sudo nmap -sU 192.168.50.149`

#### Joint TCP and UDP scanning
- Using UDP scan and TCP SYN scan in conjunction
- `sudo nmap -sU -sS 192.168.50.149`

#### Network sweeping
- applying scan to a full network range
	- for each host, nmap sends an **ICMP echo request**, a **TCP SYN packet to port 443**, a **TCP ACK packet to port 80** and an **ICMP stamp request** to verify if the host is available
- `nmap -sn 192.168.50.1-253`

#### Nmap with grep
Save output to file with greppable format
- `nmap -v -sn 192.168.50.1-253 -oG ping-sweep.txt`
	- `-v`: verbose
	- `-oG`: save output with greppable format to specified file
- sweep a specific port: `nmap -p 80 192.168.50.1-253 -oG web-sweep.txt`
Use grep to find live hosts from output file
- `grep Up ping-sweep.txt | cut -d " " -f 2`
- `grep open web-sweep.txt | cut -d " " -f 2`

#### Probe top ports
- `nmap -sT -A --top-ports=20 192.168.50.1-253 -oG top-port-sweep.txt`
	- `-A`: enable OS version detection, script scanning and traceroute
	- `--top-ports=<#>`: scan for top # ports
- Top ports are defined using the `/usr/share/nmap/nmap-services` file
	- Columns: 
	  name of service    port # / protocol    port frequency

#### OS fingerprinting
Attempts to guess the target's OS by inspecting returned packets
- `sudo nmap -O 192.168.50.14 --osscan-guess`
	- `-O`: enable OS fingerprinting option
	- `--osscan-guess`: force Nmap to print guessed result even if it is not fully accurate (by default, Nmap only prints very accurate results)

#### Service scanning
Attempts to guess service running on ports by inspecting service banners
- `nmap -sT -A 192.168.50.14`
	- `-A`: run service scan with extra options
- use `-sV` parameter to run plain service scan: `nmap -sV 192.168.50.14`
- Note: Banners can be modified by system administrators and intentionally set to fake service names to mislead potential attackers

#### [Nmap Scripting Engine (NSE)](https://nmap.org/book/nse.html)
NSE scripts are located in `/usr/share/nmap/scripts/`

Using a script
- `nmap --script http-headers 192.168.50.6`

Viewing information about a script
- `nmap --script-help http-headers`

### Port scanning in Windows
- if there is internet access, may install Windows Nmap version
- else, use **built-in PowerShell functions**

#### Test-NetConnection
- checks if an IP responds to ICMP and whether a specified TCP port on the target host is open
- Basic usage: `Test-NetConnection -Port 445 192.168.50.151`
	- returned value in `TcpTestSucceeded` parameter indicates whether the specified port is open (`True`)
- Script to scan first 1024 ports on the target: `1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("192.168.50.151", $_)) "TCP port $_ is open"} 2>$null`

# SMB Enumeration
[SMB: Server Message Block](https://en.wikipedia.org/wiki/Server_Message_Block)

Theory
- NetBIOS service: TCP port # 139
- SMB: TCP port # 445
- Modern implementations of SMB can work without NetBIOS, but *NetBIOS over TCP* (NBT) is required for backward compatibility and these are often enabled together
- Hence, enumeration of these two services often go hand-in-hand

Scanning with Nmap
- `nmap -v -p 139,445 -oG smb.txt 192.168.50.1-254`
- `cat smb.txt | grep Up`

Crackmapexec:
- `crackmapexec smb 10.10.11.174`
	- can get hostname, domain name 
- `crackmapexec smb 10.10.11.174 --shares`
	- try different authentications if cannot list shares
		- `crackmapexec smb 10.10.11.174 --shares -u '' -p ''`
		- `crackmapexec smb 10.10.11.174 --shares -u 'DoesNotExist' -p ''`
			- any username that does not exist will fall back to anonymous access

Scanning with nbtscan (a more specialised tool)
- `sudo nbtscan -r 192.168.50.0/24`
	- `-r`: specify the originating UDP port as 137
- The scan reveals the NetBIOS name of the host, which may provide information about the role of the host in the organisation

Nmap scripts for SMB enumeration
- `ls -1 /usr/share/nmap/scripts/smb*`
- `nmap -v -p 139,445 --script smb-os-discovery 192.168.50.152`
	- The `smb-os-discovery` script identifies a potential target OS
	- Note: The `smb-os-discovery` script works only if SMBv1 is enabled on the target, which is not the default case on modern versions of Windows. However, some legacy systems may be running SMBv1.
- Note: Compared to Nmap's OS fingerprinting options above, OS enumeration via NSE scripting provides extra information such as the domain and other details related to Active Directory Domain Services.
- Note: This approach will likely go unnoticed, because it blends in with normal enterprise network activity.

### SMB enumeration from Windows

net view to list SMB shares
- lists domains, resources, and computers belonging to a given host
- `net view \\<hostname> /all`
	- `/all`: list the administrative shares (ending with `$`)

# SMTP Enumeration
[SMTP: Simple Mail Transfer Protocol](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol) (port # 25)

Useful commands in SMTP:
- `VRFY`: ask the server to verify an email address
- `EXPN`: ask the server for the membership of a mailing list

`VRFY` Example:
```
kali@kali:~$ nc -nv 192.168.50.8 25 
(UNKNOWN) [192.168.50.8] 25 (smtp) open 
220 mail ESMTP Postfix (Ubuntu) 
VRFY root 
252 2.0.0 root 
VRFY idontexist
550 5.1.1 : Recipient address rejected: User unknown in local recipient table
```

`VRFY` Python script:
```python
#!/usr/bin/python
import socket
import sys
if len(sys.argv) != 3:
 print("Usage: vrfy.py <username> <target_ip>")
 sys.exit(0)
# Create a Socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# Connect to the Server
ip = sys.argv[2]
connect = s.connect((ip,25))
# Receive the banner
banner = s.recv(1024)
print(banner)
# VRFY a user
user = (sys.argv[1]).encode()
s.send(b'VRFY ' + user + b'\r\n')
result = s.recv(1024)
print(result)
# Close the socket
s.close()
```
Usage: `python3 smtp.py <username> <target IP>`

### SMTP enumeration from Windows

Obtaining SMTP information about target from Windows
- `Test-NetConnection -Port 25 192.168.50.8`
- Check `TcpTestSucceeded` value (`True`)

To interact with SMTP server, install Microsoft version of Telnet client if haven't already (requires admin privileges)
- `dism /online /Enable-Feature /FeatureName:TelnetClient`
Or use telnet binary `telnet.exe`

Connect to target machine and perform enumeration as above
- `telnet 192.168.50.8 25`

# SNMP Enumeration
SNMP: Simple Network Management Protocol (port # 161)

Theory:
- UDP, stateless protocol used to manage the network
- Susceptible to IP spoofing and replay attacks
- SNMP information and credentials can be easily intercepted over a local network (The commonly used SNMP protocols 1, 2, and 2c offer no traffic encryption)
- Traditional SNMP protocols have weak authentication schemes and are commonly left configured with default public and private community strings
- Note: Community strings are used in SNMPv1 and SNMPv2c. SNMPv3 requires username/password authentication and an encryption key.

[SNMP Management Information Base (MIB) Tree](https://www.ibm.com/docs/en/aix/7.1?topic=management-information-base)
- A database containing information usually related to network management
- It is organized like a tree:
	- Branches represent different organizations or network functions
	- Leaves correspond to specific variable values that can then be accessed and probed by an external user

Example of MIB values corresponding to specific Microsoft Windows SNMP parameters:
**OID                             Description**
1.3.6.1.2.1.25.1.6.0       System Processes 
1.3.6.1.2.1.25.4.2.1.2    Running Programs *(vulnerable aps? AV?)*
1.3.6.1.2.1.25.4.2.1.4    Processes Path 
1.3.6.1.2.1.25.2.3.1.4    Storage Units 
1.3.6.1.2.1.25.6.3.1.2    Software Name *(use with Running Programs 
                   to cross-check exact software version running on target host)*
1.3.6.1.4.1.77.1.2.25    User Accounts *(enumerate user accounts)*
1.3.6.1.2.1.6.13.1.3      TCP Local Ports *(ports that are only listening 
                  locally can reveal a new service that was unknown before)*

### Scanning for open SNMP ports with nmap
- `sudo nmap -sU --open -p 161 192.168.50.1-254 -oG open-snmp.txt`
	- `--open`: limit output display to only open ports

### Bruteforce attack with [onesixtyone](http://www.phreedom.org/software/onesixtyone/)
1. Build text files containing community strings and the IP addresses to be scanned
```
kali@kali:~$ echo public > community 
kali@kali:~$ echo private >> community 
kali@kali:~$ echo manager >> community 
kali@kali:~$ for ip in $(seq 1 254); do echo 192.168.50.$ip; done > ips
```
2. Scan: `onesixtyone -c community -i ips`

### Probing SNMP values using snmpwalk
Prerequisite: SNMP read-only community string is known (usually it's "public")

Basic
- `snmpwalk -c public -v1 -t 10 192.168.50.151`
	- `-c [string]`: specify community string
	- `-v`: specify SNMP version
	- `-t [time in seconds]`: set timeout (in seconds)
- Target email addresses can be revealed using this method

Parse a specific [OID](https://en.wikipedia.org/wiki/Object_identifier) branch of MIB Tree
- `snmpwalk -c public -v1 192.168.50.151 1.3.6.1.4.1.77.1.2.25`
- See above for interesting OIDs

 Note: SNMP walk is available for Windows as an .exe too