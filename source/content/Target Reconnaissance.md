---
tags: exiftool
---
Goals:
- identify potential users to target
- gather info about OS and installed application software

## Metadata analysis
Analysing the metadata of publicly accessible documents associated with the target organisation

Pros:
- hands-off
- Can obtain variety of information about a document including author, creation date, the name and version of the software used to create the document, operating system of the client, and much more
Cons:
- findings may be outdated if we are inspecting older documents
- different branches of the organisation may use slightly different software

Techniques:
- Google dork: `site:example.com filetype:pdf`
- Use *gobuster* with the `-x` parameter to search specific file extensions
	- Note that this is noisy and will generate log entries

### Exiftool

`exiftool -a -u brochure.pdf`
- `-a`: display duplicated tags
- `u`: display unknown tags

Notice:
- Time of creation/modification
- Author
- Software

## Client fingerprinting
to obtain operating system and browser information from a target in a non-routable internal network

Resources:
- The Harvester: https://github.com/laramies/theHarvester
	- OSINT Tool
- HTML Applications: https://learn.microsoft.com/en-us/previous-versions//ms536496(v=vs.85)?redirectedfrom=MSDN
	- Attached to email to execute code in the context of Internet Explorer and to some extent Microsoft Edge
- Canarytokens: https://canarytokens.org/nest/generate
	- generates a link with an embedded token that we’ll send to the target. When the target opens the link in a browser, we will get information about their browser, IP address, and operating system







