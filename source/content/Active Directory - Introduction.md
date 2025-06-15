[Active Directory Domain Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview):
- a service that allows system administrators to update and manage operating systems, applications, users, and data access on a large scale
- has a standard configuration, but sys admins often customize it to fit the needs of the organization
- contains critical information about the environment, storing information about users, groups, and computers, each referred to as *objects*
	- computer objects: actual servers and workstations that are domain-joined
	- user objects: accounts that can be used to log in to the domain-joined computers
- All AD objects have attributes, which will vary depending on the type of object
- Permissions set on each object dictate the privileges that object has within the domain
- An instance of AD has a domain name (e.g. "corp.com"). Within this domain, administrators can add various types of objects that are associated with the organization such as computers, users, and group objects.
- *Organizational Units (OUs)*: containers used to store objects within a domain
- AD relies on several components and communication services
	- One or more *Domain Controllers (DCs)* act as the hub and core of the domain, storing all OUs, objects, and their attributes
- Objects can be assigned to AD groups so that administrators can manage those object as a single unit
- Members of *Domain Admins* are among the most privileged objects in the domain (essentially have complete control of the domain)
- *Domain tree*: a group of more than one domain
- *Domain forest*: a group of more than one domain tree
- Members of the *Enterprise Admins* group are granted full control over all the domains in the forest and have Administrator privilege on all DCs

## Enumeration - Defining our Goals
We should perform enumeration each time we gain access to a new account, even if it seems to have the same level of access as the previous ones, since individual users may often be granted increased permissions based on their role in the organisation