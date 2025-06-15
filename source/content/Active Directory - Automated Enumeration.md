---
tags: SharpHound, BloodHound
---
## Collecting Data with SharpHound

> It’s often best to combine automatic and manual enumeration techniques when assessing Active Directory. Even though we could theoretically gather the same information with a manual approach, graphical relationships often reveal otherwise unnoticed attack paths.

[SharpHound](https://bloodhound.readthedocs.io/en/latest/data-collection/sharphound.html) is available as source code (compile it yourself), executable or PS script.

Import SharpHound PS script:
(PS) `Import-Module .\Sharphound.ps1`

Before running SharpHound, must run `Invoke-BloodHound` first:
(PS) `Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\Users\stephanie\Desktop\ -OutputPrefix "corp audit"`
- By default, SharpHound will gather the data in JSON files and automatically zip them for us.

Data from the scan is stored in the output zip file. Note that the bin cache file is not needed for our analysis and can be deleted.

> SharpHound also supports looping, which means that the collector will run cyclical queries of our choosing over a period of time

## Analysing Data using BloodHound

BloodHound can be run in Kali or Windows. The below information is for running in Kali.

1. Before using BloodHound (in Kali), start `neo4j` service:
   `sudo neo4j start`
2. Open the web interface of neo4j at `http://localhost:7474`
3. Authenticate using default credentials (`neo4j` as both username and password)
4. Change the password when prompted (but remember it for future access) (I changed it to my basic password)
5. Start BloodHound:
   `bloodhound`
6. When prompted, log in with the credentials of neo4j
7. Upload the scan data zip file to BloodHound
8. Database info
	1. Note that the database may need some time to update if there is a lot of data. Refresh to update the view
9. Analysis
	1. Especially useful ones: 
		1. **Domain Information -> Find all Domain Admins**
			1. **Settings -> Node Label Display -> Always Display** to always display
		2. **Shortest Paths -> Find Shortest Paths to Domain Admins**
			1. Right click and select **Help** for additional info
		3. **Shortest Paths -> Shortest Paths to Domain Admins from Owned Principals**
			1. Can mark any objects as "owned" in BloodHound to use this feature:
				1. Search for the object
				2. Right click the object shown in the middle
				3. Click *Mark User as Owned*

### Useful BloodHound custom queries
RawQuery function at the bottom of the GUI

Display all computers identified:
`MATCH (m:Computer) RETURN m`
- click on each item for more info (e.g. OS)
- use nslookup on the compromised machine to obtain IP address of each computer:
  `nslookup INTERNALSRV1.BEYOND.COM`

Display all user accounts on the domain:
`MATCH (m:User) RETURN m`

Display all active sessions:
`MATCH p = (c:Computer)-[:HasSession]->(m:User) RETURN p`

### Useful Bloodhound pre-built queries

Mark owned users as *Owned* by right-clicking on the nodes and selecting *Mark User as Owned*

Display all domain admins:
Analysis -> Find all domain admins

Find potential vectors to elevate our privileges or gain access to other systems:
- Find Workstations where Domain Users can RDP
- Find Servers where Domain Users can RDP
- Find Computers where Domain Users are Local Admin
- Shortest Path to Domain Admins from Owned Principals
- Shortest Paths to High Value Targets

Identify all Kerberoastable users in domain:
List all Kerberoastable Accounts

For an owned account:
- Outbound object control (list stuff this account can do)

### High value items to look for
- [[GenericAll]] (see abuse info tab in bloodhound for exploit)
- CanPSRemote
- non-default items (the last number of the object ID > 1000)

Usually to exploit BloodHound relationships, we use an impacket tool
- E.g. https://www.hackingarticles.in/abusing-ad-dacl-writeowner/





