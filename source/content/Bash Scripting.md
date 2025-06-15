# Intro to Bash Scripting

E.g. hello-world.sh
```sh
#!/bin/bash -x
# Hello World Bash Script
echo "Hello World!"
```
- `#!/bin/bash`: "shebang" + absolute path to the interpreter
- `-x`: print additional debug output
- `# ...`: a comment
Note: a script must be set to executable (`chmod +x script.sh`) to run
Run script: `./hello-world.sh`

# Variables

**Declare a variable:**
- `myvar=Good` (Note: no space before or after `=`)
Reference a variable:
- `$myvar`
Note: bash is case-sensitive

**Single quotes vs Double quotes:**
- `myvar='Good Hacker'` or `myvar="Good Hacker"`: both OK
- single quotes: all characters are viewed literally
- double quotes: all characters are viewed literally, except `$`, `` ` ``,  `\`
E.g. 
```sh
greeting1='$myvar' 
greeting2="$myvar"
# greeting1 would have value $myvar while greeting2 would have value Good
```

**Command Substitution:**
- `user=$(whoami)` or `` user2=`whoami `` (discouraged)
- Note: command substitution happens in a subshell and changes to variables in the subshell do not alter variables in the master process

### Arguments
e.g. `ls -l /var/log`: `-l` and `/var/log` are arguments to the `ls` command

e.g. `arg.sh`
```sh
#!/bin/bash
echo "The first two arguments are $1 and $2"
```
`$1`: first argument, `$2`: second argument

`$0`: name of the script itself

Other special bash variables:
- `$?`: exit status of the last process
- `$RANDOM`: a random number
- for more, refer to lab guide

### Reading User Input
`read` command: capture interactive user input while the script is running

e.g. `input.sh`
```sh
#!/bin/bash
echo "Hello, please answer Y/N"
read answer
echo "Your answer was $answer"
```

Options: 
- `-p`: specify a prompt
- `-s`: make user input silent
e.g. `input2.sh`
```sh
#!/bin/bash
# Prompt the user for credentials

read -p 'Username: ' username
read -sp 'Password: ' password

echo "Your credentials are as follows: " $userame " and " $password
```

# if, else, elif

### if
Syntax: (Note the required spaces)
```sh
if [ <some test> ]
then
	<perform an action>
	<perform another action>
fi
```

E.g. `if.sh`
```sh
#!/bin/bash

read -p "What is your age: " age

if [ $age -lt 16 ]
then
	echo "You are too young."
fi
```
(`-lt`: less than)

The square brackets `[  ]` refer to test commands (refer to lab guide for a list of test commands). The following code does the same thing:
```sh
#!/bin/bash

read -p "What is your age: " age

if test $age -lt 16
then
	echo "You are too young."
fi
```

### else
Syntax:
```sh
if [ <some test> ]
then
	<perform action>
else
	<perform another action>
fi
```

e.g. `else.sh`
```sh
#!/bin/bash

read -p "What is your age: " age

if [ $age -lt 16 ]
then
	echo "You are too young."
else
	echo "Welcome!"
fi
```

### elif
Syntax:
```sh
if [ <some test> ]
then
	<perform action>
elif [ <some test> ]
then
	<perform different action>
else
	<perform another different action>
fi
```

# Boolean Logical Operations

### And (`&&`)
Executes the following command only if the previous command succeeds (returns True or 0)
e.g.
```sh
user2=kali
grep $user2 /etc/passwd && echo "$user2 found!"
```

### Or (`||`)
Executes the following command only if the previous command fails (returns False or non-0)
e.g.
```sh
user2=bob
grep $user2 /etc/passwd && echo "$user2 found!" || echo "$user2 not found!"
```

### Testing multiple conditions
e.g. `and.sh`
```sh
#!/bin/bash
if [ $USER == 'kali' ] && [ $HOSTNAME == 'kali' ]
then
	echo "Multiple statements are true!"
else
	echo "Not much to see here."
fi
```
e.g. `or.sh`
```sh
#!/bin/bash
if [ $USER == 'kali' ] || [ $HOSTNAME == 'pwn' ]
then
	echo "One or more condition is true."
else
	echo "You are out of luck!"
fi
```

# Loops

### For Loops
Syntax:
```sh
for var-name in <list>
do
	<action to perform>
done
```

e.g. 
```sh
for ip in $(seq 1 10); do echo 10.11.1.$ip; done
```
- `$(seq 1 10)`: print a sequence of numbers 1 to 10 (inclusive)
Output:
![[Pasted image 20231227135746.png|500]]
This does the same thing:
```sh
for ip in {1..10}; do echo 10.11.1.$ip; done
```
- `{1..10}` is called a sequence expression

### While Loops
Syntax:
```sh
while [ <some test> ]
do
	<perform an action>
done
```
e.g. `while.sh`
```sh
#!/bin/bash

counter=1
while [ $counter -lt 10 ]
do
	echo "10.11.1.$counter"
	((counter++))
done
```
- `((counter++))`: double-parentheses construct performs arithmetic expansion and evaluation at the same time
Output:
![[Pasted image 20231227140930.png|255]]

# Functions
Syntax:
```sh
function function_name
	{
commands...
}
```
or
```sh
function_name () {
commands...
}
```

e.g. `func.sh`
```sh
#!/bin/bash

print_me () {
	echo "You have been printed!"
}

print_me
```

**Accepting arguments**
e.g. `funcarg.sh`
```sh
#!/bin/bash

pass_arg() {
	echo "Today's random number is: $1"
}

pass_arg $RANDOM
```

Note: The parentheses `()` only serve as decoration and are never actually used.

**Returning values**
Value returned by function can be accessed using the `$?` global variable.
e.g. `funcrvalue.sh`
```sh
#!/bin/bash

return_me() {
 echo "I'm returning a random value."
 return $RANDOM
}

return_me

echo "The previous function returned a value of $?"
```

If no return value is specified, the exit status of the function (`0` for success) will be returned to `$?`.

**Local and global variables**
To declare a local variable, use `local varname=value`
Local variables can be declared inside functions. Changes to the value of the local variable within a function does not affect the global variable. Changing a global variable within a function affects its global value.