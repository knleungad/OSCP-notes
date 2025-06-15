Context: Identified file upload functionality, but unable to upload php file to Apache web server due to file extension restriction (blacklisting)

1. Create `.htaccess` file
	`echo "AddType application/x-httpd-php .dork" > .htaccess`
2. Upload the `.htaccess` file
3. Rename the php file you want to upload to have the extension specified in the `.htaccess` file (in this example `.dork`)
4. Upload the php file

