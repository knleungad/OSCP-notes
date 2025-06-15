## Theory
Classes:
- Stored XSS
	- Exploit payload is stored in a database or otherwise cached by a server
	- Can attack all site users
- Reflected XSS
	- Payload is included in a crafted link or request
	- Only attacks user visiting the link or submitting the request

Either class can manifest as:
- Client-side
- Server-side
- DOM-based (occur when browser parses page content, instead of when JavaScript is executed)

Possible impacts:
- Session hijacking
- Forced redirection to malicious pages
- Execution of local applications as that user
- Trojanised web applications

## Identifying XSS Vulnerabilities

Common special characters: `< > ' " { } ;`
(check if application removes or encodes these characters)

Encoding:
- HTML encoding: used to display characters that normally have special meanings
- URL encoding: used to convert non-ASCII and reserved characters for URL

## Cookie stealing

Interesting cookie flags:
- `Secure`: browser may only send the cookie over encrypted connections (e.g. HTTPS)
- `HttpOnly`: browser deny JavaScript access to the cookie (if flag is not set, we can use XSS to steal cookie)

## Steps to exploit:
1. Write JS payload.
2. Use [jscompress.com](jscompress.com) to minify payload into a oneliner.
3. Encode the minified payload so any bad characters won't interfere with sending the payload.
```javascript
function encode_to_javascript(string) { var input = string var output = ''; for(pos = 0; pos < input.length; pos++) { output += input.charCodeAt(pos); if(pos != (input.length - 1)) { output += ","; } } return output; } let encoded = encode_to_javascript('insert_minified_javascript') console.log(encoded)
```
4. Final payload:
```javascript
<script>eval(String.fromCharCode('insert_encoded_payload'))</script>
```

