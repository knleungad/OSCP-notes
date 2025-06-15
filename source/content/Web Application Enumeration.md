---
tags:
  - Gobuster
  - Burp
  - DevTools
---
Discovery of the technology stack of the target:
- host OS
- web server software
- database software
- frontend/backend programming language

## Debugging page content

### URL address extension

URL address extension can reveal the programming language the application is written in (e.g. PHP uses `.php`; Java might use `.jsp`, `.do` or `.html`)
However, the usage of routes allows the mapping of a section of code to a URI, so file extensions become irrelevant.

### Browser debugger tool

May display
- JavaScript frameworks
- hidden input fields
- comments
- client-side controls 

If JS code is minified, prettify it by pressing the "{}" button

Use Inspector tool to find hidden form fields in the HTML by inspecting input elements.

## Inspecting HTTP Response Headers and Sitemaps

*Network* tab in Firefox devtools, or a proxy like BurpSuite

Important headers:
- `Server`: name of web server software (also version by default)
- Non-standard headers' names and values can reveal additional info about the technology stack used (e.g. `X-Powered-By`, `x-amz-cf-id`: Amazon CloudFront, `X-Aspnet-Version`: ASP.NET)

[Sitemaps](https://www.sitemaps.org/)
- `/robots.txt`: specifies which URLs are excluded from being crawled
- `/sitemap.xml`: specifies inclusive directives

## Enumerating and Abusing APIs

### Using Gobuster to bruteforce API endpoints

API paths are often followed by a version number, e.g. `/api_name/v1`

Bruteforcing using Gobuster's pattern feature
1. Prepare a file with patterns
```
{GOBUSTER}/v1 {GOBUSTER}/v2
```
2. `gobuster dir -u http://192.168.50.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern`

Note: if `405 METHOD NOT ALLOWED` is returned when accessing an API endpoint, try other methods such as POST, PUT and PATCH
Example: POST request
```
curl -d '{"password":"fake","username":"admin"}' -H 'Content-Type: application/json' http://192.168.50.16:5002/users/v1/login
```
- `-d`: specify data (in JSON format in this example)
- `-H`: specify extra header to use
Example: PUT request
``` 
curl -X 'PUT' \ 'http://192.168.50.16:5002/users/v1/admin/password' \ -H 'Content-Type: application/json' \ -H 'Authorization: OAuth eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzE3OTQsImlhdCI6MTY0OTI3MTQ5NCwic 3ViIjoib2Zmc2VjIn0.OeZH1rEcrZ5F0QqLb8IHbJI7f9KaRAkrywoaRUAsgA4' \ -d '{"password": "pwned"}'
```
- `-X`: specify method

Curl upload file API example:
`curl -i -L -X POST -H "Content-Type: multipart/form-data" -F file=@/home/kali/Desktop/testfile.txt -F filename="test.txt" http://192.168.55.249:33414/file-upload --proxy 127.0.0.1:8080`

### Using Burp Suite to bruteforce API endpoints

> To use Burp Suite's repeater, add `–proxy 127.0.0.1:8080` to the curl command.

Go to *Target* -> *Site Map* to view an entire map of the tested paths.




`
