# Server-Side Request Forgery (SSRF)
Server-Side Request Forgery (SSRF) is a vulnerability that allows an attacker to make the server send requests to locations that were not intended by the application.
Instead of the attacker connecting directly to the target, the vulnerable web server makes the request for them.
This can be dangerous because the server may have access to:
- Internal services that are not accessible from the internet
- Other systems inside the organization's network
- External services
- Sensitive information or credentials returned by these services

## Common vulnerable patterns
### SSRF against the local server
A common SSRF attack happens when an attacker makes the application send a request back to the same server.
This can be done using local addresses such as:
- `127.0.0.1`
- `localhost`

For example, an application may allow the user to provide the URL of a back-end API:
```text
stockApi=http://stock.example.net/product/stock
```
If this value is not properly restricted, an attacker may replace it with:

`stockApi=http://localhost/admin`

The web application then sends the request to its own local administrative endpoint.
This can be dangerous because some internal functionality may trust requests coming from the local machine and apply weaker access controls.
As a result, the attacker may use the vulnerable server to access functionality that cannot normally be reached directly from the internet.
Some applications trust requests coming from the local machine more than requests coming from external users.
This can happen for several reasons:
- Access control may be enforced by another component in front of the application server. A request sent directly from the server to itself may bypass this component.
- Administrative functionality may intentionally allow local access without authentication for maintenance or recovery purposes.
- Sensitive services may run on different local ports that are not exposed to external users.

Because of these trust relationships, an SSRF vulnerability can allow an attacker to reach privileged functionality by making the server send the request from a trusted location.

### SSRF against other back-end systems

An SSRF vulnerability can also be used to make the application server send requests to other internal systems.
These back-end systems may use private IP addresses and may not be directly reachable from the internet.

For example, an internal administrative service may be available at:

`http://192.168.0.68/admin`

If the application accepts a user-controlled URL, an attacker may be able to make the server request this internal address.

This is dangerous because internal systems sometimes have weaker security controls. They may trust requests coming from the internal network or may not require authentication at all.
In this situation, the vulnerable web server acts as a bridge between the attacker and systems that should normally remain inaccessible from the outside.

## Impact 
The impact of an SSRF vulnerability depends on which systems and resources the vulnerable server can access. 
A successful SSRF attack may allow an attacker to: 
- Access internal services that are not exposed to the internet
- Reach administrative interfaces protected only by network restrictions
- Discover hosts, ports, and services inside the internal network
- Read sensitive information returned by internal APIs
- Access cloud metadata services and potentially obtain temporary credentials
- Perform privileged actions on internal applications
- Make requests to external systems using the vulnerable server's network identity

In some cases, SSRF can be used as a first step toward further attacks. For example, an attacker may first use SSRF to discover an internal service and then interact with that service to access sensitive data or administrative functionality. The severity therefore depends heavily on the network privileges of the vulnerable server and on the security controls applied to the internal services it can reach.

## Prevention
Preventing SSRF requires controlling where the server is allowed to send requests rather than simply blocking a few dangerous addresses.
### Restrict allowed destinations

When possible, applications should use an allowlist of trusted hosts, domains, protocols, and ports.

For example, if a feature only needs to communicate with a specific API, the application should only allow requests to that API instead of accepting arbitrary URLs.

### Validate user-controlled URLs

Applications should carefully parse and validate URLs before making server-side requests.

Validation should prevent access to:
- Local addresses such as `127.0.0.1` and `localhost`
- Private IP ranges such as `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`
- Link-local addresses such as `169.254.0.0/16`
- Internal hostnames and sensitive infrastructure services
- Unexpected protocols and ports

Validation should be performed using a reliable URL parser rather than simple string matching, because URLs can be represented in different ways that may bypass weak filters.

### Control redirects and DNS resolution
Applications should also verify the final destination after DNS resolution and HTTP redirects.
Otherwise, an apparently safe URL could resolve or redirect to an internal IP address and bypass the original validation.
### Apply network-level restrictions
The application server should only be able to communicate with systems that it actually needs.
Firewalls, network segmentation, and outbound filtering can reduce the impact of SSRF by preventing the vulnerable server from reaching sensitive internal services.
### Protect internal services
Internal applications should not assume that requests are trusted simply because they originate from the internal network.
Sensitive services should still require proper authentication and authorization.
### Protect cloud metadata services
Cloud environments should use available protections for instance metadata services and restrict unnecessary access to them.
This reduces the risk of SSRF being used to retrieve credentials or other sensitive cloud metadata.

## Detection and testing
SSRF vulnerabilities are usually found in features where the server makes a request based on user-controlled input.
Common places to investigate include:
- Stock or availability checks
- URL preview or link expansion features
- Webhooks and callback URLs
- File import from a remote URL
- Image or document fetching
- API integrations
- Proxy functionality

A first step is to identify parameters that contain complete URLs, hostnames, IP addresses, or paths used by the server to contact another system.
For example:
`stockApi=http://example.com/api/stock`
The parameter can then be modified to determine whether the application accepts a different destination.
### Testing local access
A common test is to replace the original destination with a local address such as:

`http://localhost/`

or:

`http://127.0.0.1/`

If the server returns content from a local service, this may indicate that the application is vulnerable to SSRF.
### Testing internal systems
If the server can access the internal network, different private IP addresses or ports may return different responses.
Indicators can include differences in:

- HTTP status codes
- Response bodies
- Response lengths
- Error messages
- Response times

These differences may reveal reachable internal hosts or services.
Tools such as Burp Suite Repeater are useful for manually modifying and comparing requests, while Burp Suite Intruder can help test multiple hosts or values systematically.
### Out-of-band detection
Some SSRF vulnerabilities do not return the response from the requested destination to the attacker.
In these cases, an external interaction service can be used to determine whether the server performed the request.
This type of vulnerability is commonly referred to as blind SSRF.

## Classification
- **CWE:** CWE-918 — Server-Side Request Forgery (SSRF)
- **OWASP Top 10 2025:** A01:2025 — Broken Access Control
- **OWASP Top 10 2021:** A10:2021 — Server-Side Request Forgery (SSRF)
- **CAPEC:** CAPEC-664 — Server Side Request Forgery

SSRF is classified as CWE-918 by MITRE.
In the OWASP Top 10:2021, SSRF had its own category: `A10:2021 — Server-Side Request Forgery`.
In the OWASP Top 10:2025, SSRF was incorporated into `A01:2025 — Broken Access Control` rather than remaining a separate category.

## Labs completed
### PortSwigger — Basic SSRF against the local server
- **Difficulty:** Apprentice
- **Vulnerability:** SSRF against the local server
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application used a `stockApi` parameter to make server-side requests to an internal system.
By modifying this parameter, it was possible to make the server request `http://localhost/admin`. The admin interface was not directly accessible from the browser, but it became reachable through the vulnerable server.
After accessing the internal admin interface, I was able to trigger an administrative action through the same SSRF vulnerability.
This lab demonstrates how SSRF can be used to access functionality that is only available from the local machine. It also shows why applications must not trust user-controlled URLs and why internal services should not rely only on the source of the request for access control.

### PortSwigger — Basic SSRF against another back-end system

- **Difficulty:** Apprentice
- **Vulnerability:** Server-Side Request Forgery (SSRF)
- **Result:** Solved
- **Tools:** Burp Suite Intruder / Repeater

The application's stock check feature allowed the `stockApi` parameter to control the destination of a server-side HTTP request.
Using Burp Suite Intruder, I scanned the internal `192.168.0.x:8080` range and identified a host exposing an administrative interface by comparing the responses.
I then used Burp Repeater to request the internal administrative endpoint and delete the target user.
This lab demonstrates how SSRF can be used to discover and interact with services on an internal network that are not directly accessible from the Internet.


## References
- [PortSwigger Web Security Academy — Server-Side Request Forgery (SSRF)](https://portswigger.net/web-security/ssrf)
- [PortSwigger — Testing for SSRF with Burp Suite](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/ssrf/testing-for-ssrf)
- [OWASP — Server-Side Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP Top 10:2025 — A01 Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/)
- [OWASP Top 10:2021 — A10 Server-Side Request Forgery (SSRF)](https://owasp.org/Top10/2021/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [MITRE CWE-918 — Server-Side Request Forgery (SSRF)](https://cwe.mitre.org/data/definitions/918.html)
