---
tags: mimikatz
---
## NTLM Authentication

NTLM authentication is used when a client authenticates to a server by IP address (instead of by hostname), or if the user attempts to authenticate to a hostname that is not registered on the Active Directory-integrated DNS server. Likewise, third-party applications may choose to use NTLM authentication instead of Kerberos.

![[Pasted image 20241012151308.png]]
1. Client computer calculates the NTLM hash from the user's password
2. Client computer sends username to application server
3. Server returns a nonce (challenge)
4. Client encrypts the nonce using the NTLM hash (response) and sends it to server
5. Server forwards the response, nonce and username to DC
6. DC validates
7. DC approves authentication

> As with any other cryptographic hash, NTLM cannot be reversed. However, it is considered a fasthashing algorithm since short passwords can be cracked quickly using modest equipment.

NTLM may be disabled in systems, but most still have it enabled

## Keberos Authentication
the default authentication protocol in Active Directory and for associated services

![[Pasted image 20241012152221.png]]
1. When a user logs in to the workstation, an Authentication Server Request (AS-REQ) is sent to DC. AS-REQ contains a timestamp encrypted using a hash derived from the username and password.
2. DC acts as a KDC. It looks up the password hash of the user in the **ntds.dit** file and attempts to decrypt the timestamp. If the decryption process is successful and the timestamp is not a duplicate, the authentication is successful. 
   DC replies client with an Authentication Server Reply (AS-REP) containing a session key and a ticket granting ticket (TGT). The session key is encrypted using the user’s password hash and may be decrypted by the client and then reused. The TGT contains information regarding the user, the domain, a timestamp, the IP address of the client, and the session key. The TGT is encrypted by a secret key (NTLM hash of the krbtgt account) known only to the KDC and cannot be decrypted by the client. 
3. When the user wishes to access resources of the domain, the client sends the KDC a Ticket Granting Service Request (TGS-REQ) that consists of the current user and a timestamp encrypted with the session key, the name of the resource, and the encrypted TGT.
4. The ticket granting service of the KDC checks that the requested resource exists in the domain, then decrypts the TGT using the secret key. The session key is then extracted from the TGT and used to decrypt the username and timestamp of the request. The KDC checks that:
	1. The TGT has a valid timestamp
	2. The username from the TGS-REQ matches that in the TGT
	3. The client IP address matches that in the TGT
   The ticket granting service responds to the client with a Ticket Granting Server Reply (TGS-REP) containing
	1. The name of the service for which access has been granted. (encrypted with original session key)
	2. A session key to be used between the client and the service. (encrypted with original session key)
	3. A service ticket containing the username and group memberships along with the newly created session key (encrypted using the password hash of the service account registered with the service in question)
5. Client sends the application server an Application Request (AP-REQ), which includes the username and a timestamp encrypted with the session key associated with the service ticket along with the service ticket itself
6. The application server decrypts the service ticket using the service account password hash and extracts the username and the session key. It then uses the latter to decrypt the username from the AP-REQ. If the AP-REQ username matches the one decrypted from the service ticket, the request is accepted. Before access is granted, the service inspects the supplied group memberships in the service ticket and assigns appropriate permissions to the user, after which the user may access the requested service.

## Cached AD Credentials

Since Microsoft’s implementation of Kerberos makes use of single sign-on, password hashes must be stored somewhere in order to renew a TGT request.

In modern versions of Windows, these hashes are stored in the Local Security Authority Subsystem Service (LSASS) memory space. We need SYSTEM (or local admin) permissions to access this.

### Mimikatz

Mimikatz can be used to extract domain hashes from LSASS. 

Since Mimikatz is a well-known detection signature, we may use the methods discussed in the Antivirus Evasion Module, or use Task Manager to dump the LSASS process memory then move the dumped data to a helper machine, and then load the data into Mimikatz.

Example: Using Mimikatz to obtain domain credentials
1. RDP into target Windows machine as a local admin
2. Start Mimikatz:
   `.\mimikatz.exe`
3. Engage the SeDebugPrivilege privilege, which will allow us to interact with a process owned by another account:
   `privilege::debug`
4. Dump hashes of all logged on users, cached in LSASS:
   `sekurlsa::logonpasswords`
	- Either NTLM (for AD instances at a functional level of Windows 2003) or both NTLM and SHA-1 (for AD instances running Windows 2008 or above) may be available. On older OS where WDigest is enabled, cleartext passwords may be available.
5. Crack hashes as described in Password Attacks module

Example: Using Mimikatz to abuse TGT and service tickets
1. RDP into target Windows machine as local admin and start Mimikats
2. Engage the SeDebugPrivilege privilege, which will allow us to interact with a process owned by another account:
   `privilege::debug`
3. Show tickets stored in memory (includes all local users tickets):
   `sekurlsa::tickets`

### Public Key Infrastructure (PKI)

Microsoft provides the AD role Active Directory Certificate Services (AD CS)1091 to implement a PKI, which exchanges digital certificates between authenticated users and trusted resources.

If a server is installed as a Certification Authority (CA),1092 it can issue and revoke digital certificates (and much more).

We can use Mimikatz to export certificates with private key (even non-exportable private keys):
- patch the CryptoAPI function with `crypto::capi`
- patch the KeyIso service with `crypto::cng`



