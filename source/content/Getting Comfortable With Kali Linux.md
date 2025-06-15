## Finding your way around Kali
### Directories
- `/bin/`: ls, cd, cat
- `/sbin/`: system programs like fdisc, makefs, sysctl
- `/etc/`: config files
- `/tmp/`: temp files, typically cleared on boot
- `/usr/bin/`: app locations (e.g. apt, ncat, nmap)
- `/usr/share`: application support and data files
### Basic Linux Commands
- `man [command]`
	- `man -k [keyword]`: keyword search (can use regex)
	- `apropos`: same as `man -k`
- `ls`
	- `ls -a1`: `-a` display all files (including hidden ones), `-1` each file on one line
- `cd`
	- `pwd`: prints current directory
	- `cd ~`: return to home directory
- `mkdir`
	- `mkdir -p test/{recon,exploit,report}`: make multiple directories at once (using "brace expansion")
### Finding files
- `find`
	- more complicated syntax, but more powerful
	- `sudo find / -name sbd*`
	- can search by file name, type, owner, time, permissions, etc.
- `locate`
	- searches locate.db 
		- updates regularly
		- to update manually: `sudo updatedb`
- `which`
	- searches within $PATH directories
	- returns full path
## Managing Kali Linux Services
- default Kali has network services like ssh, http, mysql
- by default, prevents network services from starting, unless explicitly enabled
### SSH
- tcp based
- default listens on port 22
- `sudo systemctl start ssh`: start SSH service
- `sudo ss -antlp | grep sshd`: check if SSH is running and listening on TCP port 22
- `sudo systemctl enable ssh`: allow SSH to start by default at runtime
### HTTP
- sample usage:
	- hosting site
	- platform for downloading files to victim machine
- TCP based
- default listens on port 80
- `sudo systemctl start apache2`
- `sudo ss -antlp | grep apache`: check if HTTP is running and listening on TCP port 80
- `sudo systemctl enable apache2`: allow HTTP to start by default at runtime
### Others
- `systemctl list-unit-files`: see table of all available services, and if they're enabled or disabled
## apt: Searching, Installing and Removing Tools
- apt: package manager on Debian-based systems

- `sudo apt update`: update apt database
- `sudo apt upgrade`: upgrade apt packages
	- `sudo apt upgrade [package name]`: upgrade single package

- `apt-cache search [package name]`: search for application in Kali repository (includes searching in package description)
- `apt show [package name]`: show description of package

- `sudo apt install [package name]`: add package to system

- `apt remove --purge [package name]`: completely remove package (`--purge`: including user config files)

- `sudo dpkg -i [path to package file]`: install package
	- works offline
	- will NOT install dependencies

