If the currently compromised public machine isn't connected to internal network, use `crackmapexec` to check currently known credentials against a different public machine:
`crackmapexec smb 192.168.50.242 -u usernames.txt -p passwords.txt --continue-on-success`
- another great CrackMapExec feature: identifies the domain name and added it to the usernames -> then we know that the machine is domain-joined and we obtained a set of domain credentials
- If the identified user is local admin, there will be written `Pwn3d!` in the output

If the nmap scan identified services like WinRM or RDP, then we can proceed to compromise that machine accordingly