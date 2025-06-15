- Absolute path: starts with `/` (root directory)
- Relative path: Note that the number of `../` sequences is only relevant until we reach the root file system.

Tips:
- Examine parameters closely when they use files as values (e.g. if we find `https://example.com/cms/login.php?language=en.html`, we should try to navigate to the file directly `https://example.com/cms/en.html` to confirm that `en.html` is a file on the server)
- Check extensions (e.g. the URL in the above example indicates that the web app uses php)
- Use another tool (not just web browser) for testing, since browser may parse elements
- Use URL encoding to bypass WAF (e.g. `../` becomes `%2e%2e/`)

## Linux
Files to check:
- `/etc/passwd`: Used to check for directory traversal, Contains users and home directories of users
- `[home directory]/.ssh/id_rsa`: SSH keys of user
	- Can then SSH into target system as the user `ssh -i dt_key -p 2222 offsec@mountaindesserts.com`
	- Note: Web server is usually run in the context of a dedicated user, so may have limited access to files
	- Note: See [[Password Cracking Fundamentals]] on how to crash the passphrase for the key

## Windows
Files to check:
- Use `C:\Windows\System32\drivers\etc\hosts` to check for directory traversal
- Research log paths specific to the server that is running
	- For IIS, check:
		- logs are located in `C:\inetpub\logs\LogFiles\W3SVC1\`
		- sensitive info like passwords or usernames: `C:\inetpub\wwwroot\web.config`
- Try both `../` and `..\`

