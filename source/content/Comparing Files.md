## comm
- compares two text files
- `comm [file1] [file2]`
- output: 3 space-offset columns
	1. lines unique to file1
	2. lines unique to file2
	3. lines appearing in both files
- `comm -[column numbers] [file1] [file2]`: suppress columns
	- e.g. `comm -12 [file1] [file2]` results in only 3rd column displaying
### diff
- detect differences between files
- `diff -[format] [file1] [file2]`
- output formats:
	- `-c`: context format
		- `-` indicates line appears in file1 but not file2
		- `+` indicates line appears in file2 but not in file1
	- `-u`: unified format
		- like `-c`, but does not show lines appearing in both files
### vim diff
- opens vim with highlights to indicate diffs
- `vimdiff [file1] [file2]`
- useful shortcuts
	- `Ctrl + w` then `->`: switch between windows  
	- `] + c`: jump to next change in diff
	- `[ + c`: jump to previous change in diff
	- `d + o`: get change in other window and put it in current one
	- `d + p`: get change in current window and put it in other one
	- same shortcuts as vi