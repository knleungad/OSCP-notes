Flat network topology is poor security practice

Segmented network type (more secure):
- broken into subnets
- each subnet contains a group of devices that have a specific purpose
- devices on that subnet are only granted access to other subnets and hosts when absolutely necessary
- compromising a single host does not give access to every other device in the network

### Network technologies

Firewall: 
- can be implemented at endpoint software level
	- Linux: iptables
	- Windows: Windows Defender Firewall
- can also be implemented as a features within a piece of physical network infrastructure
- can also be a standalone hardware firewall, filtering all traffic in network
- tend to allow or block traffic in line with a set of rules based on IP addresses and port numbers,

Deep-packet inspection:
- monitors contents of incoming and outgoing traffic and terminates it based on a set of rules
- more fine-grained control than firewall

### Strategies for traversing network boundaries

Port redirection (various types of port forwarding):
- Modifying flow of data so that packet sent to one socket will be taken and passed to another socket

Port tunneling:
- Encapsulating one type of data stream with another
- E.g. transporting HTTP traffic within SSH connection (from external perspective, only SSH traffic will be visible)

