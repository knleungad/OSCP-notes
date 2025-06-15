---
tags: unix-privesc-check
---
## Files and User Privileges in Linux

In Unix derivatives (including Linux), most resources, including files, directories, devices, and network communications are represented in the filesystem.

Every file abides by user and group permissions based on:
- read (r)
	- file: allows reading file content
	- directory: allows consulting the list of its contents
- write (w)
	- file: allows changing its content
	- directory: allows creating or deleting files
- execute (x)
	- file: allows file to be run
	- directory: allows crossing through the directory to access its contents (e.g. `cd`)

Each file or directory has specific permissions for these three categories of users:
- owner
- owner group
- others group

Format:
E.g. 
```
kali@kali:~$ ls -l /etc/shadow 
-rw-r----- 1 root shadow 1751 May 2 09:31 /etc/shadow
```
- first `-`: describes file type (directory or file) (irrelevant to permissions)
- next three characters: file owner (root) permissions
- next three characters: owner group permissions
- final three characters: others group permissions

## Manual Enumeration
- Good for identifying more peculiar privesc methods that are often overlooked by automated tools
- automated enumeration cannot replace manual investigation because the customized settings of our target environments are likely to be exactly those that are misconfigured

### Gather user context information

Identify user context:
`id`
- UID
- GID
- groups

Enumerate all users:
`cat /etc/passwd`
- user accounts used by services (with home folder `/usr/sbin/nologin`), which indicates the services running on the target machine
- standard user accounts have configured home folder `/home/[username]` -> identify potential high privilege user accounts for privesc
- Data format:
  E.g. `joe:x:1000:1000:joe,,,:/home/joe:/bin/bash`
	- Login Name: `joe`
	- Encrypted Password: `x` 
		- x means that the entire password hash is in `/etc/shadow` file
	- UID / Real User ID: `1000` 
		- Aside from the root user that has always a UID of 0, Linux starts counting regular user IDs from 1000
	- GID: `1000`
	- Comment: `joe,,,`
		- description about user, typically only repeats username
	- Home Folder: `/home/joe`
	- Login Shell: `/bin/bash`
		- full default interactive shell, if one exists

Discover hostname:
`hostname`
- check for identifiable abbreviations that may indicate the target's information (e.g. location, role)

### Gather OS information
Useful for kernel exploitation

> Kernel exploits:
	specified by a particular operating system and version combination
	Since attacking a target with a mismatched kernel exploit can lead to system instability or even a crash, we must gather precise information about the target

Gather info about the operating system:
`cat /etc/issue` (OS version)
`cat /etc/os-release` (release-specific info)
`uname -a` (kernel version and architecture)

### Explore running processes
some may allow us to elevate privileges

List system processes:
`ps aux`
- `ax`: list all processes with or without a tty
- `u`: list in a user-readable format

### Review available network interfaces, routes and open ports
- help determine if compromised target is connected to multiple networks and therefore could be used as a pivot
- presence of specific virtual interfaces could indicate the existence of virtualisation or AV

We can also investigate port bindings to see if a running service is only available on a loopback address, rather than on a routable one. Investigating a privileged program or service listening on the loopback interface could expand our attack surface and increase our probability of a privilege escalation attack’s success

List TCP/IP configuration of every network adapter:
`ifconfig` or `ip a` (compact)

Display network routing tables:
`route` or `routel`

Display active network connections and listening ports:
`netstat` 
or 
`ss -anp`
- `-a`: list all connections
- `-n`: avoid hostname resolution (which may stall command execution)
- `-p`: list the process name the connection belongs to

### Firewall rules
- if a network service is not remotely accessible because it is blocked by the firewall, it is generally accessible locally via the loopback interface. If we can interact with these services locally, we may be able to exploit them to escalate our privileges on the local system.
- information about inbound and outbound port filtering to facilitate port forwarding and tunneling when it’s time to pivot to an internal network
- On Linux-based systems, we must have root privileges to list firewall rules with `iptables`. However, depending on how the firewall is configured, we may be able to glean information about the rules as a standard user.

Gleaning firewall rules as standard user:
- the *iptables-persistent* package on Debian Linux saves firewall rules in specific files under `/etc/iptables` by default
- search for files created by the `iptables-save` command. If a system administrator had ever run this command, we could search the configuration directory (`/etc`) or grep the file system for iptables commands to locate the file
`cat /etc/iptables/rules.v4`
- look out for non-default rules

### Scheduled tasks
- Linux job scheduler: cron
- Systems acting as servers often periodically execute various automated, scheduled tasks. When these systems are misconfigured, or the user-created files are left with insecure permissions, we can modify these files that will be executed by the scheduling system at a high privilege level.

List scheduled tasks:
`ls -lah /etc/cron*`
`cat /etc/crontab`
- carefully inspect any scheduled tasks added by system admins for insecure file permissions
- most jobs here will run as root
- wildcard `*`: bash replaces it with all the filenames in the current directory, so may be able to exploit it to run privesc command (with reference to GTFOBins)

View current user's scheduled jobs:
`crontab -l`
`sudo crontab -l` (to view jobs run by root user. works if the current use may is granted sudo permission for listing cron jobs)

See [[Insecure File Permissions]]

### Local applications
Should use automation for this step, to search for a matching exploit. Here is the manual method.

List all applications installed by dpkg in Debian:
`dpkg -i`

### Files that the current user has access to
- May modify scripts or binaries that are executed under the context of a privileged account (e.g. cron scripts)
- Sensitive files that are readable by an unprivileged user may also contain important information such as hard-coded credentials for a database or a service account running with higher privileges

Enumerate writeable files:
`find / -writable -type d 2>/dev/null`
- `/`: search whole root directory
- `-writeable`: specify attribute we are interested in
- `-type d`: locate directories
- `2>/dev/null`: filter errors

### Unmounted drives
could contain valuable info

List all mounted systems:
`mount`
- system administrator might have used custom configurations or scripts to mount drives that are not listed in the `/etc/fstab` file

List all drives that will be mounted at boot time:
`cat /etc/fstab`

List all available disks:
`lsblk`
- might reveal partitions that are not mounted
- we might be able to mount these partitions and search for information there (if the system (mis)configuration allows us)

### Loaded kernel modules
Need to match vulnerabilities with corresponding exploits

List loaded kernel modules:
`lsmod`

More info about specific module:
`/sbin/modinfo [module]`

### SUID-marked binaries
- Two special rights pertaining to executable files:
	- setuid: allows the current user to execute the file with the rights of the owner
	- setgid: allows the current user to execute the file with the rights of the owner's group
- If these rights are set, either an uppercase or lowercase "s" will appear in permissions
- Exploitation of SUID binaries will vary based on several factors

Search for SUID-marked binaries:
`find / -perm -u=s -type f 2>/dev/null`
- `/`: search at root directory
- `-type f`: search for files
- `-perm -u=s`: search for those which have SUID bit set
- `2>/dev/null`: discard error messages

### Abusing sudo

List commands that you can sudo:
`sudo -l`

Example output:
```
User webadmin may run the following commands on serv:
    (ALL : ALL) /bin/nice /notes/*
```
means you can use the binary `/bin/nice` on the files `/notes/*`

### List of techniques
- https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md
- https://book.hacktricks.xyz/linux-hardening/privilege-escalation
- https://pentestmonkey.net/tools/audit/unix-privesc-check

See [[Insecure System Components]]

### Interesting file locations

- `/opt`: new programs may be saved there
- `/etc/knockd.conf`: ssh information
	- Port knocking: you need to knock on specific ports to unlock the SSH access
	- `for X in 4000 5000 6000; do nmap -Pn -host-timeout 201 -max-retries 0 -p $X [IP here]; done`

## Automated Enumeration

Tool: [unix-privesc-check](https://pentestmonkey.net/tools/audit/unix-privesc-check) (for finding misconfigs)
- Located in `/usr/bin/unix-privesc-check` in Kali
- Review tool's details:
  `unix-privesc-check`
- Use standard mode:
  `./unix-privesc-check standard > output.txt`

Other tools: LinEnum, LinPeas











