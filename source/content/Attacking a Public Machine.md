Goal: 
- Identify credentials
- local priv esc
- find way to pivot to internal network (if possible from current machine)

### Local enumeration

Use linpeas.sh (for Linux) or winpeas (for Windows)

Transfer the tool to the target machine

Run the tool

Identify:
- OS version
- Network interfaces
	- If it is connected to an internal network, we can use it as a pivot point
- (Linux) commands executable with sudo
	- GTFOBins
- Credentials
	- save any identified credentials in our tracker
- Be creative
	- E.g. if git is used then we can check the commits of the repo for changes in config data and sensitive info

Identify potential priv esc vectors from findings.

After priv esc, enumerate system again