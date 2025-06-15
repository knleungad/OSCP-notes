## Backgrounding processes (`bg`)
- Method 1
	- `[command] &`: run command in background
		- E.g. `ping -c 400 localhost > ping_results.txt &`
- Method 2 (don't use for time-sensitive jobs, e.g. `ping`)
	- `Ctrl + z` after command has started: suspend job
	- `bg`: resume job in the background
## Jobs Control: `jobs` and `fg`
- `jobs`: list jobs running in current terminal session
- `fg %[job number]`: return job to foreground
## Process control: `ps` and `kill`
- `ps`: list system-wide processes and details
	- `ps -ef`
		- `-e`: select all processes
		- `-f`: display full format listing
	- `ps -fC [command name]`
		- `-C`: search by command name
- `kill [pid]`: terminate process