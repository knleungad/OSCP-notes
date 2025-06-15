# Website Recon
Visiting the target's website and looking for useful information, such as email formats and social media pages.

# Whois Enumeration
Provides information about a domain name, such a name server and registrar
**Forward Lookup**
- Usage: `whois <domain name>`, e.g. `whois megacorpone.com`
- Things to pay attention to:
	- Registrant Name
	- Name Servers (a component of DNS)
**Reverse Lookup**
- Usage: `whois <ip address>`, e.g. `whois 38.100.193.70`
- Things to pay attention to:
	- Who is hosting the IP address

# Google Hacking
Search terms to be used in Google searches
- `site:<domain name>`: limit results to a single domain
	- used to get a rough idea of an organization's web presence
	- e.g. `site:megacorpone.com`
- `filetype:<file type>`: limit results to the specified file type
	- used to find pages supposedly only available to internal network (e.g. php)
	- used to discern what programming languages might be used on a website (e.g. jsp for java server pages, cfm for adobe coldfusion, pl for perl pages)
	- e.g. `site:megacorpone.com filetype:php`
- `-`: exclude item from results
	- used to find interesting (e.g. non-html) pages
	- e.g. `site:megacorpone.com -filetype:html`
- `intitle:<text>`: find pages containing the specified text in the title
	- e.g. `intitle:"index of" "parent directory"` finds pages that contain "index of" in the title and "parent directory" anywhere on the page

Check [Google hacking database](https://www.exploit-db.com/google-hacking-database) for more
Use [dorksearch](https://dorksearch.com/) for pre-built queries and builder.

# [Netcraft](https://www.netcraft.com/)
Various info
- https://searchdns.netcraft.com/: search subdomains
	- **Site report** to view additional information for each subdomain (e.g. Site Technology)

# Open-source code
Gain information about the programming language and frameworks used by the target organization, as well as any sensitive data committed to public repos.
- Online code repos:
	- [GitHub](https://github.com/)
	- [GitHub Gist](https://gist.github.com/)
	- [GitLab](https://about.gitlab.com/)
	- [SourceForge](https://sourceforge.net/)
- GitHub search operators
	- `path:**/*users*`
- Tools for automating search
	- [Gitrob](https://github.com/michenriksen/gitrob): find potentially sensitive files on GitHub
	- [Gitleaks](https://github.com/michenriksen/gitrob): detect hardcoded secrets in git repos
		- `./gitleaks-linux-amd64 -v -r=https://github.com/[repo]`

# [Shodan](https://www.shodan.io/)
A search engine that crawls devices connected to the internet (including routers, IoT devices, etc.)
- `hostname:megacorpone.com`
	- Note services that the servers are running (click on service name in left panel to view version)
	- Click on an IP address for a summary (ports, services, and technologies used) of the host
		- Any published vulnerabilities for any of the identified services or technologies will be shown

# [Security Headers](https://securityheaders.com/) and [SSL/TLS](https://www.ssllabs.com/ssltest/)
- Security Headers: analyze HTTP response headers 
- SSL Server Test: analyzes a server’s SSL/TLS configuration and compares it against current best practices



