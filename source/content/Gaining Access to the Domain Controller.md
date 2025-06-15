Once obtained local admin on a machine, use Mimikatz to extract cached Kerberos password or NTLM hash of active domain admin user on the machine

Then, use the domain admin creds to perform lateral movement to domain controller.
Two methods:
- impacket-psexec with NTLM hash
- use plaintext Kerberos password directly