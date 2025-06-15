---
tags:
  - PowerView
  - SysInternals
  - setspn.exe
---
## Enumerating Operating Systems

Use PowerView to enumerate computer objects in domain:
(PV) `Get-NetComputer`
(PV) `Get-NetComputer | select operatingsystem,dnshostname,operatingsystemversion`

## Getting an Overview - Permissions and Logged on Users

determine if our current user has administrative permissions on any computers in the domain：
(PV) `Find-LocalAdminAccess`

Check for logged in users on machine:
(PV) `Get-NetSession -ComputerName files04 -Verbose`
- Check that the IP address matches as well, otherwise it's not working. Can try a different tool instead. 
- 5 possible query levels:
	- 0: name of the computer establishing the session
	- 1, 2: more info but requires admin privileges
	- 10: default, return information such as the name of the computer and name of the user establishing the connection
	- 502: return information such as the name of the computer and name of the user establishing the connection
- The permissions required to enumerate sessions with `NetSessionEnum` are defined in the `SrvsvcSessionInfo` registry key, which is located in the `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanServer\DefaultSecurity` hive
	- Check permissions on current machine (note that current machine may have different permissions than the other machines in the environment, but is still valuable for reference):
	  `Get-Acl -Path HKLM:SYSTEM\CurrentControlSet\Services\LanmanServer\DefaultSecurity\ | fl`
		- Users that have either **FullControl** or **ReadKey** can read the `SrvsvcSessionInfo` registry key
	- Due to changes in Windows registry permissions, *NetSessionEnum* won't work on default Windows 11 OS, some Windows 10 OS and Windows Server operating systems since Windows Server 2019 build 1809

> Alternate tool: *PsLoggedOn* application from the *SysInternals Suite*.
> Limitation: it relies on the *Remote Registry* service in order to scan the associated key. This has not been enabled by default on Windows workstations since Windows 8

Check for logged in users with PsLoggedOn:
`.\PsLoggedOn.exe \\files04`
- if the output shows no users, it could be because the Remote Registry service is not running on that machine
- note that PsLoggedOn also uses the NetSessionEnum API, which in this case requires a logon in order to work. Hence, it may show that the domain user that we are using is logged on to the machine

## Enumeration Through Service Principal Names
- Services launched by the system itself run in the context of a Service Account (e.g. LocalSystem, LocalService, and NetworkService).
- For more complex applications, a domain user account may be used to provide the needed context while still maintaining access to resources inside the domain
- A unique service instance identifier known as *Service Principal Name (SPN)* associates a service to a specific service account in Active Directory

Tool: setspn.exe (installed on Windows by default)

Enumerate SPNs linked to a certain domain user:
`setspn -L iis_service`\

Enumerate SPNs in domain:
(PV) `Get-NetUser -SPN | select samaccountname,serviceprincipalname`

Revolve obtained network paths:
`nslookup.exe web04.corp.com`
- from the result, it’s clear that the hostname resolves to an internal IP address.

## Enumerating Object Permissions

- An object in AD may have a set of permissions applied to it with multiple Access Control Entries (ACE).
- These ACEs make up the Access Control List (ACL).
- Each ACE defines whether access to the specific object is allowed or denied.

ACL validation process:
1. A domain user wants to access an object in AD (e.g. a domain share)
2. The user sends an *access token*, which contains the user identity and permissions
3. The target object validates the token against the ACL
4. If the ACL allows the user access to the object, access is granted. Otherwise, the request is denied

Some interesting permission types for ACE:
- `GenericAll`: Full permissions on object (highest)
- `GenericWrite`: Edit certain attributes on the object 
- `WriteOwner`: Change ownership of the object 
- `WriteDACL`: Edit ACE's applied to object 
- `AllExtendedRights`: Change password, reset password, etc. 
- `ForceChangePassword`: Password change for object 
- `Self` (Self-Membership): Add ourselves to for example a group

Enumerate ACEs:
(PV) `Get-ObjectAcl -Identity stephanie`
- Things to notice in the output:
	- `SecurityIdentifier` (the object who has the rights)
		- Convert to an actual domain object name:
		  (PV) `Convert-SidToName S-1-5-21-1987370270-658905905-1781884369-1104`
	- `ActiveDirectoryRights` (the type of rights applied to the object)
- Note: `ObjectSID` is the object we are enumerating

Enumerate objects that have "GenericAll" permission on a specific object:
(PV) `Get-ObjectAcl -Identity "Management Department" | ?{$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights`
- Typically, a normal domain user should not have "GenericAll" permission on other objects in the AD

## Enumerating Domain Shares
Domain shares often contain critical information about the environment, which we can use to our advantage.

Find shares in domain:
(PV) `Find-DomainShare`
- add flag `-CheckShareAccess` to display only shares available to us
- note that it may take some time
- The `SYSVOL` file share may include files and folders that reside on the domain controller itself. It is typically used for various domain policies and scripts. By default, it is mapped to `%SystemRoot%\SYSVOL\Sysvol\domainname` on the domain controller and every domain user has access to it 

List items in share:
`ls \\dc1.corp.com\sysvol\corp.com\`

Group Policy Preferences:
- GPP-stored passwords are encrypted by AES, but the private key is posted in [MSDN](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be?redirectedfrom=MSDN#endNote2).

Use gpp-decrypt ruby script in Kali to decrypt a GPP encrypted string:
`gpp-decrypt "+bsY0V3d4/KgX3VJdO/vyepPfAN1zMFTiQDApgR92JE"`

Or use tool directly on the file Groups.xml (sample location on target: `\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\Groups.xml`):
`python3 gpp-decrypt.py -f ~/Downloads/temp/Active/Groups.xml`
(Tool location: `~/Documents/Tools/gpp-decrypt/`)











