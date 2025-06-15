---
tags: xfreerdp, net.exe, PowerView
---
## Enumeration Using Legacy Windows Tools

Connect to client by RDP:
`xfreerdp /f /u:stephanie /d:corp.com /v:192.168.50.75`
- `/f` for fullscreen. Ctrl+Alt+Enter to toggle fullscreen.

`rdesktop [target ip]`

### Using net.exe for enumeration
Installed by default on all Windows OS
Note that this method will miss some info

Enumerate domain users:
`net user /domain`

Query information about a user:
`net user jeffadmin /domain`
- look out for **Domain Admins**

Enumerate domain groups:
`net group /domain`
- look out for custom groups!

Enumerate group members:
`net group "Sales Department" /domain`

## Using PowerShell and .NET classes for enumeration

LDAP: protocol used to communicate with AD (but other directory services use it as well)

LDAP ADsPath prototype:
`LDAP://HostName[:PortNumber][/DistinguishedName]`
- `HostName`: can be a computer name, IP address or a domain name
	- Note that a domain may have multiple DCs, so setting the domain name could potentially resolve to the IP address of any DC in the domain
	- Should look for the Primary DC (PDC), which holds the most updated information. It holds the *PdcRoleOwner* property
- `PortNumber`: optional
- `DistinguishedName`: DN, name that uniquely identifies an object in AD, including the domain itself. Reads from right to left.
	- E.g. DN of the domain user *stephanie*: `CN=Stephanie,CN=Users,DC=corp,DC=com`
		- `CN`: Common Name, the identified of an object in the domain
		- `DC`: Distinguished Component, represents the top of an LDAP tree, the distinguished name of the domain itself

Invoke the Domain Class and the GetCurrentDomain method:
(PS) `[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`
- the PdcRoleOwner property is the hostname of the PDC

Powershell script for generating LDAP of PDC:
```Powershell
$PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name 
$DN = ([adsi]'').distinguishedName 
$LDAP = "LDAP://$PDC/$DN" 
$LDAP
```

Bypass PS execution policy before running script:
(PS) `powershell -ep bypass`
`.\enumeration.ps1`

## Adding Search Functionality to our Script

PS script for enumerating all users in domain:
```powershell
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = $domainObj.PdcRoleOwner.Name
$DN = ([adsi]'').distinguishedName
$LDAP = "LDAP://$PDC/$DN"
$direntry = New-Object System.DirectoryServices.DirectoryEntry($LDAP)
$dirsearcher = New-Object System.DirectoryServices.DirectorySearcher($direntry)
$dirsearcher.filter="samAccountType=805306368"
$result = $dirsearcher.FindAll()
Foreach($obj in $result)
{
 Foreach($prop in $obj.Properties)
 {
 $prop
 }
 Write-Host "-------------------------------" 
}
```
- `samAccountType`: attribute applied to all user, computer, group objects
	- 0x30000000 (decimal 805306368) will enumerate all users in the domain
	- Possible values: https://learn.microsoft.com/en-us/windows/win32/adschema/a-samaccounttype
- Filter based on any property of any object type
	- set the filter (e.g. `$dirsearcher.filter="name=jeffadmin"`) and change the property (e.g. `$prop.memberof` to show only groups that jeffadmin is a member of)

Function for AD enumeration:
```powershell
function LDAPSearch {
 param (
 [string]$LDAPQuery
 )
 $PDC =[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
 $DistinguishedName = ([adsi]'').distinguishedName
 $DirectoryEntry = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$PDC/$DistinguishedName")
 $DirectorySearcher = New-Object System.DirectoryServices.DirectorySearcher($DirectoryEntry, $LDAPQuery)
 return $DirectorySearcher.FindAll()
}
```

Import the function: `Import-Module .\function.ps1`
Usage:
- `LDAPSearch -LDAPQuery "(samAccountType=805306368)"`
- `LDAPSearch -LDAPQuery "(objectclass=group)"`
	- This enumerates more groups than net.exe because this also enumerates Domain Local groups instead of only Domain Global groups
- Enumerate every group in the domain and display the user members:
  `foreach ($group in $(LDAPSearch -LDAPQuery "(objectCategory=group)")) { $group.properties | select {$_.cn}, {$_.member} }`
- `$sales = LDAPSearch -LDAPQuery "(&(objectCategory=group)(cn=Sales Department))"`
  `$sales.properties.member`

## AD Enumeration with [PowerView](https://powersploit.readthedocs.io/en/latest/Recon/)

Importing PowerView:
`Import-Module .\PowerView.ps1`

Get basic information about domain:
`Get-NetDomain`

Get all users in domain:
`Get-NetUser`

Get all groups in domain:
`Get-NetGroup`

Choose the attribute we are interested in:
`Get-NetUser | select cn`
`Get-NetUser | select cn,pwdlastset,lastlogon`
`Get-NetGroup | select cn`
`Get-NetGroup "Sales Department" | select member`





