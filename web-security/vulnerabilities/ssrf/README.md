# Server-Side Request Forgery (SSRF)

## What is SSRF?

Server-Side Request Forgery (SSRF) is a vulnerability that allows an attacker to make a server-side application send requests to locations that were not intended by the application.

Instead of the attacker connecting directly to the target system, the vulnerable application performs the request on their behalf.

Conceptually:

```text
Attacker
    ↓
Vulnerable application
    ↓
Unintended destination
```

The destination may be:

- An internal service that is not directly accessible from the Internet
- Another system within the organization's infrastructure
- An arbitrary external server

For example, an application may normally make a server-side request to retrieve information from a legitimate service.

If the destination of that request can be influenced by user input, an attacker may be able to redirect the request toward another location.

```text
Normal behavior:

User
  ↓
Application
  ↓
Expected service
```

```text
SSRF:

Attacker-controlled input
        ↓
Application
        ↓
Unexpected internal or external service
```

This can expose systems or information that the attacker could not access directly.

In some cases, SSRF can also cause sensitive information, such as authorization credentials, to be sent to another system.

Server-Side Request Forgery (SSRF) is a vulnerability that allows an attacker to make the server send requests to locations that were not intended by the application.
Instead of the attacker connecting directly to the target, the vulnerable web server makes the request for them.
This can be dangerous because the server may have access to:
- Internal services that are not accessible from the internet
- Other systems inside the organization's network
- External services
- Sensitive information or credentials returned by these services

## Common SSRF Attacks

SSRF attacks often exploit trust relationships that exist between systems.

The vulnerable application may be trusted more than an external user because it is running:

- On the same server as sensitive functionality
- Inside the organization's internal network
- In an environment where back-end services accept requests from trusted internal sources

An attacker can abuse this trust by making the vulnerable application send requests on their behalf.

Conceptually:

```text
Attacker
    ↓
Vulnerable application
    ↓
Trusted internal access
    ↓
Sensitive functionality
```

Two common scenarios are:

- SSRF against the application server itself
- SSRF against other back-end systems

In both cases, the attacker attempts to use the vulnerable server's position and network access to reach functionality that would normally be unavailable from the Internet.

### SSRF Attacks Against the Server

A common SSRF scenario occurs when an attacker makes the vulnerable application send an HTTP request back to the same server that hosts the application.

This usually involves loopback addresses such as:

```text
127.0.0.1
localhost
```

These addresses refer to the local machine itself.

For example, an application may allow the client to supply the URL of a back-end service used to retrieve stock information:

```text
stockApi=http://stock.example.net/product/stock
```

If the application accepts this value without sufficient restrictions, an attacker may replace it with a local URL:

```text
stockApi=http://localhost/admin
```

The vulnerable application then makes the request itself:

```text
Attacker
    ↓
Vulnerable application
    ↓
http://localhost/admin
```

The response from the local administrative interface may then be returned to the attacker.

### Why Local Requests May Be Trusted

This becomes especially dangerous when the application treats requests originating from the local machine differently from normal external requests.

Possible reasons include:

- Access control may be enforced by a separate component in front of the application server. A request made directly from the server to itself may bypass this component.
- Administrative functionality may allow unauthenticated local access for recovery purposes.
- Sensitive administrative interfaces may listen on a different port that is not directly reachable from external users.

This creates a trust relationship:

```text
External request
    ↓
Normal access controls

Local request
    ↓
Trusted differently
    ↓
Sensitive functionality
```

An SSRF vulnerability can therefore allow an attacker to make the application access functionality that would normally be protected from external users.

The important point is that the attacker is not bypassing the access controls directly. Instead, they make the trusted server send the request from a location that the application already considers privileged.

### SSRF Attacks Against Other Back-End Systems

An SSRF vulnerability can also allow an attacker to make the application server send requests to other internal systems.

These back-end systems may use private IP addresses and may not be directly reachable from the Internet.

For example:

```text
192.168.0.68
```

may identify an internal server that external users cannot normally access.

Conceptually:

```text
Attacker
    ↓
Public application
    ↓
Internal network
    ↓
Back-end system
```

This can be dangerous because internal systems are often protected mainly by the network topology.

Since they are not expected to receive requests directly from the Internet, they may have weaker security controls or expose sensitive functionality without authentication.

For example, an internal administrative interface may exist at:

```text
http://192.168.0.68/admin
```

If the application contains an SSRF vulnerability, an attacker may modify a user-controlled URL such as:

```text
stockApi=http://192.168.0.68/admin
```

The application server then sends the request to the internal system on the attacker's behalf.

```text
Attacker
    ↓
Vulnerable application
    ↓
http://192.168.0.68/admin
```

This effectively turns the vulnerable application into a bridge between the attacker and systems that are normally isolated from external users.

The key security issue is that internal network access should not be treated as equivalent to authorization. A service should not expose sensitive functionality simply because the request originates from another internal system.

## Impact of SSRF Attacks

A successful SSRF attack can allow an attacker to make the vulnerable application access resources or perform actions that should not normally be available to them.

The impact may affect:

- The vulnerable application itself
- Internal services
- Other back-end systems that the application can reach

Depending on the exposed internal functionality, SSRF may result in:

- Unauthorized access to internal data
- Unauthorized actions on internal systems
- Access to back-end services that are not exposed publicly
- In some cases, arbitrary command execution

The security impact therefore depends heavily on what systems the vulnerable application is able to communicate with.

### Requests to External Systems

SSRF is not limited to internal infrastructure.

If an attacker can force the application to connect to an external third-party system, the vulnerable server may be used to send malicious requests to other targets.

Conceptually:

```text
Attacker
    ↓
Vulnerable application
    ↓
External target
```

From the external target's point of view, these requests may appear to originate from the organization hosting the vulnerable application rather than from the attacker directly.

This means SSRF can potentially be used not only to access internal systems, but also to make the vulnerable server participate in attacks against other systems.

## Circumventing Common SSRF Defenses

Applications that make server-side requests often include protections intended to prevent users from accessing sensitive destinations.

However, these protections can sometimes be bypassed when they rely on incomplete URL validation, simple string matching, or assumptions about how URLs and IP addresses are represented.

A common example is a blacklist that blocks known dangerous values such as:

```text
127.0.0.1
localhost
/admin
```

The problem is that the same destination can sometimes be represented in several different ways.

### SSRF with Blacklist-Based Input Filters

A blacklist-based defense attempts to reject specific hostnames, IP addresses, paths, or strings considered dangerous.

For example:

```text
Blocked:
127.0.0.1
localhost
/admin
```

If the filter only compares strings, an attacker may be able to express the same destination differently.

Possible techniques include:

- Alternative IP address representations
- URL encoding
- Case variations
- Domains that resolve to an internal address
- Redirects through an attacker-controlled server

#### Alternative IP Representations

The IPv4 loopback address:

```text
127.0.0.1
```

can sometimes be written in other formats while still resolving to the same host.

Examples include:

```text
127.0.0.1
127.1
2130706433
017700000001
```

Depending on the URL parser and networking library, these values may all refer to the same loopback address.

This can bypass a filter that only checks whether the literal string `127.0.0.1` appears in the input.

#### Encoding and String Variations

Filters based on simple string matching may also be bypassed by changing how blocked values are represented.

Examples include:

- URL-encoding characters
- Changing character case where the parser treats values case-insensitively
- Encoding parts of a path

The important issue is that the application may validate one representation while the underlying URL parser interprets another.

#### DNS-Based Bypasses

Another possibility is to use a domain name that resolves to an internal or loopback IP address.

Conceptually:

```text
Attacker-controlled domain
        ↓ DNS
127.0.0.1
```

A filter that checks only the hostname string may consider the domain safe even though the resolved IP address points to a restricted destination.

#### Redirect-Based Bypasses

An attacker-controlled URL may also redirect the server to another destination.

For example:

```text
Vulnerable application
        ↓
https://attacker-controlled.example
        ↓ HTTP redirect
http://127.0.0.1/admin
```

If the application validates only the initial URL but automatically follows redirects, the final destination may bypass the original SSRF protection.

Different redirect status codes or protocol changes may also affect how the application processes the destination.

### Key Principle

The general problem with blacklist-based SSRF protection is that the same resource may have multiple valid representations.

```text
Different input
      ↓
Same destination
```

A filter that blocks only known strings may therefore miss alternative ways of reaching the same internal system.

### SSRF with Whitelist-Based Input Filters

Some applications attempt to prevent SSRF by allowing requests only when the supplied URL appears to contain an approved hostname.

For example, an application may try to allow only:

```text
expected-host
```

This approach can become vulnerable when the validation logic and the component that actually performs the HTTP request interpret the URL differently.

The general problem is:

```text
User-controlled URL
        ↓
Security filter parses it
        ↓
HTTP client parses it
```

If both components disagree about which part of the URL represents the destination, an attacker may be able to make the filter accept one host while the HTTP client connects to another.

#### Credentials Before the Hostname

URLs can contain user information before the hostname using the `@` character.

For example:

```text
https://expected-host:fakepassword@evil-host
```

This URL can be broken down as:

```text
https://
expected-host:fakepassword
@
evil-host
```

The important point is that:

```text
expected-host:fakepassword
```

is interpreted as user information, while the actual hostname is:

```text
evil-host
```

A weak filter that only checks whether the URL begins with `expected-host` may therefore accept the URL even though the HTTP request is sent to `evil-host`.

#### URL Fragments

The `#` character introduces a URL fragment.

For example:

```text
https://evil-host#expected-host
```

The hostname is:

```text
evil-host
```

while:

```text
expected-host
```

belongs to the fragment.

Fragments are normally not part of the HTTP request sent to the destination server.

A weak filter that simply searches the entire input for `expected-host` may therefore accept the URL even though the real destination is `evil-host`.

#### Abusing the DNS Naming Hierarchy

A required hostname can also be included as part of a larger domain controlled by an attacker.

For example:

```text
https://expected-host.evil-host
```

The actual hostname is:

```text
expected-host.evil-host
```

This is a subdomain of:

```text
evil-host
```

not of `expected-host`.

If an attacker controls `evil-host`, they can control where this hostname resolves.

A weak filter that only checks whether the string `expected-host` appears somewhere in the URL may incorrectly consider this destination trusted.

#### URL Encoding

URL encoding can sometimes create differences between what the validation code examines and what the HTTP client eventually interprets.

For example, characters that have a structural meaning in a URL may be encoded:

```text
@
→
%40
```

If one component validates the encoded representation but another component decodes it before parsing the URL, they may interpret the destination differently.

The same problem can occur with double encoding.

For example:

```text
%2540
    ↓ first decoding
%40
    ↓ second decoding
@
```

This is particularly useful when different processing layers decode the input a different number of times.

#### Combining Techniques

Whitelist bypasses often involve combining several URL features.

For example, an attacker may combine:

- User information with `@`
- URL fragments with `#`
- Attacker-controlled subdomains
- URL encoding
- Double encoding

The objective is always the same:

```text
Filter interprets URL as allowed
             ↓
HTTP client interprets destination differently
             ↓
Request reaches unintended host
```

### Key Principle

Whitelist-based SSRF defenses can fail when they validate URLs using simple string operations instead of consistently parsing and validating the actual destination.

The security decision should be based on the final parsed and resolved destination, not merely on whether an expected hostname appears somewhere in the user-controlled URL.

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

## Labs

- [Completed labs](labs.md)

## References
- [PortSwigger Web Security Academy — Server-Side Request Forgery (SSRF)](https://portswigger.net/web-security/ssrf)
- [PortSwigger — Testing for SSRF with Burp Suite](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/ssrf/testing-for-ssrf)
- [OWASP — Server-Side Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP Top 10:2025 — A01 Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/)
- [OWASP Top 10:2021 — A10 Server-Side Request Forgery (SSRF)](https://owasp.org/Top10/2021/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [MITRE CWE-918 — Server-Side Request Forgery (SSRF)](https://cwe.mitre.org/data/definitions/918.html)
