---
tags: socat
---
## A Simple Port Forwarding Scenario

![[Pasted image 20240819135703.png]]

## Setting up the scenario environment

(skip the exploitation of Confluence server to obtain reverse shell here)

Enumeration:
- Check network interfaces:
  `ip addr`
- Check routes:
  `ip route`
	- shows hosts in which subnet you can access through each interface
- Check configuration file of Confluence to look for any credentials and IP address of database server

Issue: 
- No PostgreSQL client installed in CONFLUENCE01 machine and we do not have enough privileges to install it
- PostgreSQL client is installed in the Kali machine, but can't connect directly to PGDATABASE01 since it's only routable from CONFLUENCE01

Solution: 
- There's no firewall between Kali and CONFLUENCE01, so we can bind ports on the WAN interface of CONFLUENCE01 and connect to them from Kali
- Create a port forward on CONFLUENCE01 that listens on a port on the WAN interface, then forwards all packets received to PGDATABASE01 on the internal subnet

## Port forwarding with Socat

![[Pasted image 20240819221037.png]]

Prerequisite:
- Socat already installed in CONFLUENCE01, or
- Able to download and run a statically-linked binary version of Socat in CONFLUENCE01 instead

Steps:
1. On CONFLUENCE01, use Socat to set up port forwarding:
  `socat -ddd TCP-LISTEN:2345,fork TCP:10.4.50.215:5432`
	- `-ddd`: verbose
	- `TCP-LISTEN:2345`: listen to this TCP port
	- `fork`: fork into a new subprocess when it receives a connection instead of dying after a single connection
	- `TCP:10.4.50.215:5432`: forward all received traffic to this IP and TCP port
2. Use the port forwarding from Kali machine:
   `psql -h 192.168.50.63 -p 2345 -U postgres`
	- Use the IP and port of CONFLUENCE01

From here, we can continue enumeration in the PostgreSQL database.

### Alternatives to Socat

- [rinetd](https://github.com/samhocevar/rinetd): 
	- runs as a daemon
	- better for long-term port forwarding
	- unwieldly for temporary port forwarding solutions
- Combine Netcat and a [FIFO](https://man7.org/linux/man-pages/man7/fifo.7.html) named pipe file
- If we have root privileges, could use iptables 
	- In Linux, also requires enabling forwarding on the interface we want to forward on by writing “1” to `/proc/sys/net/ipv4/conf/[interface]/forwarding`

