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
