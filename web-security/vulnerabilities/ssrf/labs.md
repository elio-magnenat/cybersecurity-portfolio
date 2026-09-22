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

