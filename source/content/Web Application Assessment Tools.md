---
tags:
  - Nmap
  - Wappalyzer
  - Gobuster
  - Burp
---
## Fingerprinting Web Servers with Nmap
### 1. Get web server version
Use Nmap service scan (`-sV`) to grab web server (`-p80`) banner:
`sudo nmap -p80 -sV 192.168.50.20`

### 2. Perform initial fingerprinting
`sudo nmap -p80 --script=http-enum 192.168.50.20`

## Technology Stack Identification with [Wappalyzer](https://www.wappalyzer.com/)

On the Wappalyzer website, perform a technology lookup on the target site to learn about
- the OS
- the UI framework
- the web server
- the JavaScript libraries used
- and more
Check versions for known vulnerabilities.
## Directory Brute Force with [Gobuster](https://www.kali.org/tools/gobuster/)

`gobuster dir -u 192.168.50.20 -w /usr/share/wordlists/dirb/common.txt -t 5`
- `dir`: enumerate files and directories
- `-u [ip]`: specify target ip
- `-w [filepath]`: specify wordlist
- `-t [int]`: specify number of running threads (default is 10)

`dirb [url] [wordlist] -o [save to file] -X [extension]`

Other useful wordlists:
- `/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt` (feroxbuster default wordlist)
- `/usr/share/wordlists/dirb/big.txt`

### Fuzz for virtual hosts / subdomains

`ffuf -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ -u http://board.htb/ -H 'Host: FUZZ.board.htb' -fs 15949`

## Security Testing with Burp Suite

Configure Firefox in Kali to use Burp Suite proxy
1. In Firefox, navigate to about:preferences#general
2. *Network Settings* -> *Settings*
3. Select *Manual proxy configuration*
4. Enter proxy ip and port (127.0.0.1 8080 by default) and enable for all protocol options
Note: If "detectportal.firefox.com" keeps appearing in the proxy history, enter about:config in the address bar, search for "network.captive-portal-service.enabled" and set it to "false".

