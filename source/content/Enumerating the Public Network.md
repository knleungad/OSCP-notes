### set up a tracker work environment
- a directory for each target machine we have access to at the moment
- a creds.txt file for storing identified credentials and users

### Perform nmap scan
`sudo nmap -sC -sV -oN mailsrv1/nmap 192.168.50.242`
- `-sC`: enable nmap's default scripts
- `-sV`: enable service and version detection
- `-oN`: create output file

Search online for potential vulnerabilities and exploits against identified service versions

### Enumerate web server

Browse to the web page

Use gobuster/dirb
`gobuster dir -u http://192.168.50.242 -w /usr/share/wordlists/dirb/common.txt -o mailsrv1/gobuster -x txt,pdf,config`

Feroxbuster:
`feroxbuster --url {URL}`

Identify technology used 
- check source code
- use `whatweb`:
  `whatweb http://192.168.50.244`
- For Wordpress sites, use `wpscan --url website.com/path/to/wordpress --api-token osgkdV0NKb4kDiCfPCqMKRNd93HNoMJz8cLE7pvEdL8`
	- Enumerate users: `wpscan --url http://loly.lc/wordpress --enumerate u`

Identify vulnerabilities in technology used
- searchsploit:
  `searchsploit [keyword]`

### Exploit vulnerability

Gain access to machine


