---
tags: john, hashcat
---
To decrypt an encrypted password we must determine the key used to encrypt it. To determine the plaintext of a hashed password, we must run various plaintext passwords through the hashing algorithm and compare the returned hash to the target hash.

Pros:
- conserves network bandwidth
- does not lock accounts
- is not affected by traditional defensive technologies

Hashing a string: 
`echo -n "secret" | sha256sum`
- `-n` to strip newline, which would affect the output hash value

Tools:
- John the Ripper (JtR)
	- CPU-based which also supports GPU
	- No additional drivers required
- Hashcat
	- GPU-based which also supports CPU
	- Requires OpenCL or CUDA for the GPU cracking process

For most algos, GPU is much faster than CPU, but some slow hashing algos (like bcrypt) work better on CPU.

## Calculating cracking time

cracking time = (keyspace) / (hash rate)

Note that cracking time increases exponentially with password length, but polynomially with password complexity (charset).

### Keyspace
Keyspace: Number of unique variations that can be generated for an x-character password with this character set

**Keyspace = (size of character set) ^ (length of password)**

Find size of character set:
`echo -n "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789" | wc -c`

Calculate keyspace (e.g. size of character set = 62 and length of password = 5):
`python3 -c "print(62**5)"`

### Hash rate

Find hash rate:
`hashcat -b`

Unit: MH/s (1,000,000 hashes per second)

## Mutating wordlists

Need to prepare wordlist by removing all passwords which do not satisfy the password policy or by modifying the wordlist to include appropriate passwords.

### Rule-based attack

Basic steps:
1. Create a file containing rule functions
2. (Optional) Use debug mode to check if rule function file is correct:
   `hashcat -r demo.rule --stdout demo.txt`
	- `-r [rule function file]`: specify rule function file
	- `--stdout`: debug mode
	- `demo.txt`: replace with the file of passwords that you want to mutate
3. Run Hashcat:
   `hashcat -m 0 crackme.txt /usr/share/wordlists/rockyou.txt -r demo3.rule --force`
	- `-m [hash type code]`: specify hash type
	- `crackme.txt`: replace with the target hash file
	- `/usr/share/wordlists/rockyou.txt`: replace with wordlist
	- `-r [rule function file]`: specify rule function file
	- `--force`: ignore related warnings

> Note that rule-based attacks increase the number of attempted passwords tremendously

[Hashcat wiki list of rule functions](https://hashcat.net/wiki/doku.php?id=rule_based_attack#writing_rules)
Commonly used rule functions:
- `$X`: Append character 'X' to end
- `^X`: Prepend character 'X' to front
- `c`: Capitalize first letter and lower the rest

Rule function file:
- Rules on same line separated by a space: rules are used consecutively on each password
- Rules on separate lines: rules are used separately, resulting in a mutated password for every rule for every password
```
kali@kali:~/passwordattacks$ cat demo1.rule 
$1 c 

kali@kali:~/passwordattacks$ hashcat -r demo1.rule --stdout demo.txt 
Password1 
Iloveyou1 
Princess1 
Rockyou1 
Abc1231 

kali@kali:~/passwordattacks$ cat demo2.rule 
$1 
c 

kali@kali:~/passwordattacks$ hashcat -r demo2.rule --stdout demo.txt 
password1 
Password 
iloveyou1 
Iloveyou 
princess1 
Princess 
...
```

Typical human behaviour:
- When an upper case letter is required, most users capitalize the first letter. 
	- E.g. `computer` -> `Computer`
- When special characters are required, most users add the special character at the end of the password and rely on characters on the left side of the keyboard since these digits are easy to reach and type.
	- E.g. `computer` -> `computer!`
- When numbers are required, most users add '1', '2' or '123' at the end of the password.
	- E.g. `computer` -> `computer123`

When we don't have any information about the target's password policy, we may use predefined rules at `/usr/share/hashcat/rules`.

#### John the Ripper

To use rule functions in JtR
1. Save the rule file (e.g. `ssh.rule`) using naming syntax like this:
   ```
   [List.Rules:sshRules] 
   c $1 $3 $7 $! 
   c $1 $3 $7 $@ 
   c $1 $3 $7 $#
   ```
2. Append contents of our rule file into `/etc/john/john.conf`:
   `sudo sh -c 'cat /home/kali/passwordattacks/ssh.rule >> /etc/john/john.conf'`


## Cracking Methodology

Steps for cracking a hash:
1. Extract hashes
	- E.g. Dump the database containing the hashed user passwords
2.  Format hashes
	- Identify the hash type (with `hash-identifier` or `hashid`)
	- Check if the representation format of the hashes is correct for our cracking tool. If not, may use helper tool to change it.
3. Calculate cracking time
	- If the calculated cracking time exceeds our expected lifetime, we might reconsider this approach!
	- Consider the duration of the current penetration test
4. Prepare wordlist
	- In nearly all cases, will need to mutate wordlist 
	- Should investigate potential password policies and research other password vectors, including online password leak sites
	- Else, run multiple wordlists with (or without) preexisting rules for a broad coverage of possible passwords
5. Attack the hash
	- Take special care in copying and pasting our hashes (no extra spaces or newlines!)
	- Make sure we know for sure the hash type!

## Password Manager
Password managers create and store passwords for different services, protecting them with a master password. This master password grants access to all passwords held by the password manager.

Scenario: We have access to a system desktop
(Hashcat)
1. Identify any password manager programs
	1. (Windows) GUI method: click Windows -> search "Apps" -> *Add or remove programs*
2. Research database file used by the password manager
	1. E.g. KeePass uses *.kbdx* file
3. (PowerShell) Locate files in specified locations:
   `Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue`
4. Transfer database file to Kali
5. Transform the hash into a format our cracking tool can use
	1. E.g. To transform *.kbdx* to Hashcat format, can use `ssh2john` or `keepass2john`
	2. Further modify the resulting hash if needed. 
	   E.g. Remove `Database:` (it acts as a username for the target hash, which is helpful for cracking database hashes)
6. Determine the hash type:
   `hashcat --help | grep -i "KeePass"` or look it up on [Hashcat wiki](https://hashcat.net/wiki/doku.php?id=example_hashes)
7. Calculate cracking time
8. Prepare wordlist
9. Start cracking:
   `hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force`

## SSH Private Key Passphrase

After connecting to an SSH service using a compromised SSH private key (e.g. obtained via directory traversal), we would be prompted for a passphrase. We need to crack the passphrase.

1. Obtain SSH private key (e.g. `id_rsa`)
2. Transform SSH private key to hash format accepted by cracking tool:
   `ssh2john id_rsa > ssh.hash`
3. Remove filename before first colon, and remove the colon too.
4. Determine hash type.
5. Determine Hashcat mode
   `hashcat -h | grep -i "ssh"`
6. Prepare rule file and wordlist
7. Start cracking:
   `hashcat -m 22921 ssh.hash ssh.passwords -r ssh.rule -- force`, or
   `john --wordlist=ssh.passwords --rules=sshRules ssh.hash`
8. Use cracked passphrase to SSH
   `ssh -i id_rsa -p 2222 dave@192.168.50.201` then enter passphrase when prompted

## /etc/passwd

If the password is shadowed, you must have /etc/passwd and /etc/shadow.
1. Save the whole line of the user in /etc/shadow
2. Save the whole line of the user in /etc/passwd
3. `unshadow passwd.txt shadow.txt > unshadowed.txt`
4. `john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt`

If the password is not shadowed (its encrypted form is already in /etc/passwd), only need /etc/passwd
1. Save the whole line of the user in /etc/passwd
2. `john --wordlist=/usr/share/wordlists/rockyou.txt webadmin.hash.2`

## Identify a hash
Tool: `hash-identifier`
Command: `hash-identifier`

## Other tools
https://hashes.com/en/decrypt/hash