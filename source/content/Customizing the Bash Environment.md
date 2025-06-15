## Bash `history` Customization
3 environmental variables: HISTCONTROL, 
- change value of variable using `export VARIABLE=value`
### `HISTCONTROL`
- whether or not to ignore duplicate commands, commands that include spaces from history, or both
	- default: both are removed
- `export HISTCONTROL=ignoredups`: only ignore duplicate commands (more useful)
### `HISTIGNORE`
- set commands to be filtered from the history
- `export HISTIGNORE="&:ls:[bf]g:exit:history"`
### `HISTTIMEFORMAT`
- controls date and/or time stamps in the output of the `history` command
- e.g. `export HISTTIMEFORMAT='%F %T'`
	- `man strftime` for list of formats

## Alias
A string we can define to replace a command name
- e.g. `alias lsa='ls -la'`
- list all defined aliases: `alias`

### Persistent bash customization
system-wide bash settings: `/etc/bash.bashrc`
user bash settings (to override system-wide bash settings): `.bashrc`
`.bashrc` is a shell script that runs every time the user logs in
