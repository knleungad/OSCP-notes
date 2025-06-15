## Environment Variables
- `$PATH`
- `$USER`
- `$PWD`: present working directory
- `$HOME`
- assigning environment variables
	- `export [variable name]=[content]`: variable is accessible to any sub-process spawned from current bash instance
		- to use: `$[variable name]`
	- `[variable name]=[content]`: variable only accessible in current shell
- default environmental variables
	- `env`: view
## Bash History Tricks
- `history`: display record of entered commands (preceded by line number)
- `![line number]`: re-execute command at line number
- `!!`: re-execute previous command
- location of history file: `~/.bash_history`
- environment vars:
	- `$HISTSIZE`: num of commands stored in memory for current session
	- `$HISTFILESIZE`: num of commands kept in history file
- Ctrl+R: search and execute a command in history
## Piping and Redirection
- Streams:
	- STDIN (fd: 0)
	- STDOUT (fd: 1)
	- STDERR (fd: 2)
- redirect to file
	- `>`: overwrite file
	- `>>`: append to existing file
- redirect from file
	- `<`: use content of file as input to command
- redirect stderr
	- `[command] 2> [file path]`: redirect stderr to file
- pipe
	- `|`

