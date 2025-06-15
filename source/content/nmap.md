`sudo nmap -sS [ip] -vv -p- -Pn`
- or `-sC`
`sudo nmap -p [port1,port2,...] [ip] -Pn -A -sCV -O`

`sudo nmap -Pn -n [ip] -sU --top-ports=100 --reason`