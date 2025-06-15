---
tags: curl
---
### `wget [url]`
- download files using HTTP and FTP protocols
- `-O [filename]`: save file using a different name on the local machine
- FTP download all files recursively: 
	`wget -r -nH --cut-dirs=5 -nc ftp://user:pass@server//absolute/path/to/directory`
### `curl`
- transfer data to or from a server using a host of protocols
	- can download or upload files, or other stuff
- basic usage: `curl [url]`: download data from that url
	- `-o [filename]`: save file using a different name on the local machine
- Proxy (e.g. to Burp): `--proxy [ip]:[port]`
- Header: `--user-agent "[header content]"`
### `axel`
- download accelerator which transfers a file from FTP or HTTP server through multiple connections
	- especially useful for large downloads to speed up downloading process
- `-n [number]`: specify number of connections to use
- `-a`: more precise progress indicator
- `-o [filename]`: save file using a different name on the local machine

## PowerShell
`iwr -uri http://192.168.118.2/winPEASx64.exe -Outfile winPEAS.exe`