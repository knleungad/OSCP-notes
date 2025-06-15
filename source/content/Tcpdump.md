> A text-based network sniffer
> 
> Captures traffic from the network and reads existing capture files
> 
> Platforms: Unix and Linux

### Read existing capture file
- `sudo tcpdump -r file.pcap`
	- `-r`: read packet capture file

### Filter traffic
**Analyse traffic**
- `sudo tcpdump -n -r file.pcap | awk -f" " '{print $3}' | sort | uniq -c | head`
	- `-n`: skip DNS name lookup
	- `awk -f" " '{print $3}'`: print destination IP address and port contained in the 3rd field
	- `sort`: sort the output
	- `uniq -c`: count number of times the field appears in the capture
	- `head`: only display first 10 lines of output
	- e.g. 
	  ![[Pasted image 20231224223808.png|825]]
		- `172.16.40.10.81`: low port number (81), probably server
		- `208.68.234.99.32768`: high port number (32768), probably client
**Apply filters**
- `sudo tcpdump -n src host 172.16.40.10 -r file.pcap`
	- `src host 172.16.40.10`: output only traffic originating from the given host 
- `sudo tcpdump -n dest host 172.16.40.10 -r file.pcap`
	- `dest host 172.16.40.10`: output only traffic destined for the given host
- `sudo tcpdump -n port 81 -r file.pcap`
	- `port 81`: filter by given port number (displays both source and destination traffic)
**Inspect packets**
- `sudo tcpdump -nX -r file.pcap`
	- `-X`: print packet data in both hex and ASCII format

### Advanced Header Filtering
**Filter out and display only data packets**
- TCP header format: ![[Pasted image 20231224225018.png]]
- Need to choose only the packets with ACK and PSH bits turned on (4th and 5th bit of the 14th byte)
1. Convert the binary to decimal representation: `echo $((2#00011000))` (the output is `24`)
2. Apply filter: `sudo tcpdump -A -n 'tcp[13] = 24' -r file.pcap`
	- `-A`: print packets in ASCII
	- `'tcp[13] = 24'`: filter out and display only packets with the 14th byte set to 24 (i.e. ACK and PSH bits turned on)
		- Note: array index used for counting the bytes starts from 0
