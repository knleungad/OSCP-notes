### Main content
- `grep [string]`
	- search text file for lines containing string
	- `-r`: recursive search
	- `-i`: ignore case
	- `-v`: search for lines NOT containing string
	- `-o <regex>`: only return string defined in the regular expression
- `sed`
	- text editing on stream of text (displays on stdout)
	- E.g. 
		```
		> echo "I need to try hard" | sed 's/hard/harder/'
		| I need to try harder
		```
- `cut`
	- extract field using specified delimiter
	- E.g.
		```
		> echo "a,b,c,d" | cut -f 2 -d ","
		| b
		```
- `awk`
	- programming language for data extraction and reporting
	- can use like cut, but can do multiple extractions at once
	- E.g.
		```
		> echo "a::b::c" | awk -f "::" '{print $1, $3}'
		| a c
		```
### Example
- `wc -l [filepath]`
	- display total number of lines in file
- `[command] | sort -u`
	- sort and remove duplicates
- `cat access.log | cut -d " " -f 1 | sort | uniq -c | sort -urn`
	- sort by number of occurences
	- `uniq -c`: prefix output line with number of occurences
