---
tags: metasploit
---
## Setup and Work with MSF

### Initial database setup

Start MSF database service (for storing information about target hosts and keeping track of successful exploitation attempts, optional):
`sudo msfdb init`

Enable database service at boot time:
`sudo systemctl enable postgresql`

Launch Metasploit command line interface:
`sudo msfconsole`

### Inside msfconsole database setup

Verify database connectivity:
`db_status`

### Metasploit workspace
for separately storing information for each assessment (have a different workspace for each)

List all previously created workspaces:
`workspace`

Create new workspace:
`workspace -a pen200`
- `-a`: create new workspace
- `pen200`: name of workspace

### Populating the workspace

Execute Nmap inside Metasploit and save findings to database:
`db_nmap -A 192.168.50.202`
- syntax same as nmap

### Database backend commands

List of discovered hosts:
`hosts`

List of discovered services:
`services`
`services -p 8000` (filter for specific port number)

### Modules

View all module categories:
`show -h`

Activate module:
`use [module name]`
- syntax of module name: slash-delimited (module type/os, vendor, app, operation, or protocol/module name)

## Auxiliary Modules
provide functionality such as protocol enumeration, port scanning, fuzzing, sniffing, and more

List all auxiliary modules (very long list):
`show auxiliary`

Search module with filters (by app, type, CVE ID, operation, platform and more):
`search type:auxiliary smb` (search all smb auxiliary modules)

Activate a module:
`use [module name]` or `use [module number from search result]`
- e.g. `use 56`

Get information about currently activated module:
`info`

Display options of a module:
`show options`

Add or remove values from options:
`set [option] [value]`
`unset [option] [value]` (do this before setting a new value)

Set value of option to results in database:
E.g. `services -p 445 --rhosts` (set RHOSTS to all discovered hosts with port 445 open)

Launch module:
`run`

Show all automatically detected vulnerabilities so far:
`vulns`

Show all gathered credentials so far:
`creds`

## Exploit Modules
most commonly contain exploit code for vulnerable applications and services

Mostly the same commands as auxiliary modules

Always review module's information (`info` command) and understand what it does before using

After launching module with `run`, a session is created (e.g. interactive shell)

Send session to background:
Ctrl+Z then `y`

List all active sessions:
`sessions -l`

Interact with a session:
`sessions -i [session ID]`

Kill a session:
`sessions -k [session ID]`

Launch module in the context of a job (runs in the background):
`run -j`