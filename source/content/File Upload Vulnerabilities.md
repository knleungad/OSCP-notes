## Overview
Three categories:
- Can upload files which are executable by the web application
	- E.g. Can upload a php script to a web server where php is enabled
- Requires us to combine the file upload mechanism with another vulnerability
	- E.g. Directory Traversal: Use a path in the file upload request to overwrite files like **authorized_keys**
	- E.g. XXE (XML External Identity): Upload an avatar of SVG file type with an embedded XXE attack
	- E.g. XXS
- Relies on user interaction
	- E.g. Upload a .docx file with malicious macros, wait for a person to access our uploaded file

> Note: When testing file upload, test what happens when the same file is uploaded twice. Error messages may be used to bruteforce directory contents or gauge the programming language or web technologies in use.

## Using executable files
Example: Upload and access a webshell (then create reverse shell)

Bypassing file extension blacklists:
- `php`: `phps`, `php7`, `phtml`
- changing characters in the file extension to upper case (e.g. `pHP`)

Note: To bypass filtering of file extension, may also upload webshell with inconspicuous file extension (e.g. txt) and then use a vulnerability to rename it.

E.g. Powershell reverse shell (see pdf)

## Using non-executable files

If directory traversal can be used when uploading a file (e.g. upload with filename `../../../../test.txt`), we may blindly overwrite files to lead to system access. 
Note: Do with caution in production systems!!!

### Overwrite SSH key of root
Always verify whether we can leverage root or admin privileges in a file upload vulnerability (from developers running the web application as root).

- (Linux) Overwrite `authorized_keys` file in the home directory for *root* -> SSH into system as *root*
	1. Generate key pair with command 
	   `ssh-keygen`
	   `cat fileup.pub > authorized_keys`
	2. Upload `authorized_keys` to relative path `../../../../../../../root/.ssh/authorized_keys`
	3. `ssh -p 2222 -i fileup root@mountaindesserts.com`

Note: root user often does not have SSH access permissions. Can try overwriting other users' by displaying contents of `/etc/passwd` if possible. Write the generated file `xxx.pub` into `/home/{username}/.ssh/authorized_keys`




