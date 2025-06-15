---
tags: watch, tcpdump
---
## Inspecting User Trails

dotfiles:
- files with a period prepended before the filename
- system does not display these files when inspecting by basic listing commands
- applications frequently store user-specific configuration files and subdirectories as dotfiles

`.bashrc` script: executed when a new shell instance is started from an existing login session

Look for interesting environmental variable entries (e.g. password):
`env`

Inspect **.bashrc** script to confirm if the interesting environmental variable is permanent:
`cat .bashrc` (see if the variable is defined there)

Verify if we are running as a privileged user (user credentials required):
`sudo -l`

Elevate to root (requires that we are running as a privileged user, user credentials required):
`sudo -i`

## Inspecting Service Footprints

### System daemons

System daemons: 
- Spawned at boot time to perform specific operations without user interaction
- Used for SSH, web servers, databases, etc.
- Often used to execute ad-hoc tasks (sometimes neglect security best practices!)

Enumerate all running processes (including those running inside higher-privilege user context like root):
`ps`

Run `ps` command every minute (using `watch`) and filter results containing "pass":
`watch -n 1 "ps -aux | grep pass"`
- Might find passwords here

### Network traffic

`tcpdump`: 
- command for packet capture
- requires administrative access
- some IT personnel may have been given exclusive sudo access to this tool

Capture traffic in and out of the loopback interface, and filter results containing "pass":
`sudo tcpdump -i lo -A | grep "pass"`


