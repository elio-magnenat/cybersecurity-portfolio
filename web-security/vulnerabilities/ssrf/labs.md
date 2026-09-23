# Server-Side Request Forgery (SSRF) Labs

## PortSwigger — Basic SSRF against the local server
- **Difficulty:** Apprentice
- **Vulnerability:** SSRF against the local server
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application used a `stockApi` parameter to make server-side requests to an internal system.
By modifying this parameter, it was possible to make the server request `http://localhost/admin`. The admin interface was not directly accessible from the browser, but it became reachable through the vulnerable server.
After accessing the internal admin interface, I was able to trigger an administrative action through the same SSRF vulnerability.
This lab demonstrates how SSRF can be used to access functionality that is only available from the local machine. It also shows why applications must not trust user-controlled URLs and why internal services should not rely only on the source of the request for access control.

## PortSwigger — Basic SSRF against another back-end system

- **Difficulty:** Apprentice
- **Vulnerability:** Server-Side Request Forgery (SSRF)
- **Result:** Solved
- **Tools:** Burp Suite Intruder / Repeater

The application's stock check feature allowed the `stockApi` parameter to control the destination of a server-side HTTP request.
Using Burp Suite Intruder, I scanned the internal `192.168.0.x:8080` range and identified a host exposing an administrative interface by comparing the responses.
I then used Burp Repeater to request the internal administrative endpoint and delete the target user.
This lab demonstrates how SSRF can be used to discover and interact with services on an internal network that are not directly accessible from the Internet.

### PortSwigger — SSRF with blacklist-based input filter

- **Difficulty:** Practitioner
- **Vulnerability:** Server-Side Request Forgery (SSRF) — Blacklist Bypass
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a stock check feature that made server-side requests using a user-controlled `stockApi` parameter.

The objective was to access the local administrative interface at:

```text
http://localhost/admin
```

and delete the `carlos` user.

A direct request to the loopback address was blocked by the application's anti-SSRF filter:

```text
http://127.0.0.1/
```

I bypassed this first restriction by using an alternative representation of the same loopback address:

```text
http://127.1/
```

The application accepted this representation even though it still resolved to the local server.

Accessing the administrative path directly was also blocked:

```text
http://127.1/admin
```

The application therefore appeared to blacklist both local-address patterns and the `admin` path.

I bypassed the second restriction by double URL-encoding the first character of `admin`:

```text
http://127.1/%2561dmin
```

The encoded value was decoded during request processing:

```text
%2561
   ↓
%61
   ↓
a
```

This allowed the server-side request to ultimately reach:

```text
http://127.0.0.1/admin
```

without the blacklist detecting the blocked string in its original form.

After accessing the internal admin interface, I used the same SSRF mechanism to delete the `carlos` user and solve the lab.

This lab demonstrated why blacklist-based SSRF defenses are unreliable. Different representations of the same IP address and multiple decoding stages can allow an attacker to reach a destination that simple string-based filters attempt to block.

### PortSwigger — SSRF with filter bypass via open redirection vulnerability

- **Difficulty:** Practitioner
- **Vulnerability:** Server-Side Request Forgery (SSRF) — Open Redirect Filter Bypass
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a stock check feature that performed server-side requests using a user-controlled `stockApi` parameter.

The objective was to reach an internal administration interface located at:

```text
http://192.168.0.12:8080/admin
```

and delete the `carlos` user.

Directly changing the `stockApi` parameter to another host was blocked because the stock checker was restricted to URLs belonging to the local application.

While examining the application's navigation, I found that the "next product" functionality used a user-controlled `path` parameter in an HTTP redirect.

For example:

```text
/product/nextProduct?path=http://example.com
```

caused the application to redirect to the supplied destination.

This created an open redirect that could be combined with the SSRF vulnerability.

Instead of supplying the internal address directly to the stock checker, I supplied a local application URL containing the internal destination inside the redirect parameter:

```text
/product/nextProduct?path=http://192.168.0.12:8080/admin
```

The SSRF filter accepted the initial URL because it pointed to the permitted application.

The request flow was therefore:

```text
Stock checker
      ↓
Allowed local URL
      ↓
Open redirect
      ↓
http://192.168.0.12:8080/admin
```

Because the HTTP client followed the redirect, the server eventually made a request to the otherwise blocked internal administration interface.

After reaching the admin interface, I modified the redirect target to request the administrative action used to delete the `carlos` user and solved the lab.

This lab demonstrated that validating only the initial destination of an SSRF request is insufficient when redirects are automatically followed.

A trusted URL can become an intermediate step:

```text
Allowed URL
    ↓
Redirect
    ↓
Restricted internal destination
```

SSRF protections therefore need to consider the final destination reached after redirects, not only the first URL supplied by the user.

