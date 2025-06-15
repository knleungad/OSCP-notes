get password hash of zip:
`zip2john sitebackup3.zip`

look up format of hash in hashcat help page

crack the hash:
`hashcat -m 13600 sitebackup3.hash /usr/share/wordlists/rockyou.txt --force`

unzip with password:
`7z x sitebackup3.zip -pcodeblue`