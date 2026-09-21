# Broken Access Control Labs

## PortSwigger — Unprotected admin functionality
- **Difficulty:** Apprentice
- **Vulnerability:** Unprotected administrative functionality
- **Result:** Solved

The application exposed an administrative panel without checking whether the user had administrator permissions.
The panel location was disclosed in the `robots.txt` file. After accessing the admin endpoint directly, it was possible to use administrative functionality as a normal unauthenticated user.
This lab demonstrates that hiding a sensitive URL is not an access control mechanism. The server must verify the user's permissions before displaying the page and before performing every sensitive action.

## PortSwigger — Unprotected admin functionality with unpredictable URL
- **Difficulty:** Apprentice
- **Vulnerability:** Unprotected administrative functionality / security by obscurity
- **Result:** Solved

The application used an unpredictable URL for the admin panel, but the location was exposed in client-side JavaScript.
By reviewing the page source, it was possible to discover the hidden endpoint and access the administrative functionality without being an administrator.
This lab demonstrates that using a hard-to-guess URL does not replace proper access control. Sensitive endpoints must always be protected by server-side permission checks.

## PortSwigger — User role controlled by request parameter
- **Difficulty:** Apprentice
- **Vulnerability:** Parameter-based access control
- **Result:** Solved

The application stored the user's administrative role in a cookie controlled by the browser.
By modifying the cookie value, it was possible to change the user's role and access the admin panel without having real administrator permissions.
This lab demonstrates that access control decisions must not rely on data controlled by the user. Roles and permissions should be verified using trusted server-side information.

## PortSwigger — User ID controlled by request parameter with unpredictable IDs
- **Difficulty:** Apprentice
- **Vulnerability:** Horizontal privilege escalation / IDOR
- **Result:** Solved

The application used GUIDs to identify users, which made the identifiers difficult to guess.
However, another user's GUID was exposed elsewhere in the application through public content. After finding this identifier, it was possible to modify the account request and access data belonging to that user.
This lab demonstrates that unpredictable identifiers do not replace authorization checks. The server must verify that the authenticated user is allowed to access the requested account or resource.

## PortSwigger — User ID controlled by request parameter with password disclosure
- **Difficulty:** Apprentice
- **Vulnerability:** Horizontal to vertical privilege escalation
- **Result:** Solved

The application allowed a user to access another account by modifying the user identifier in the request.
The administrator's account page exposed the current password inside a masked form field. Even if the password was not visible in the interface, it was still present in the HTTP response and could be read using Burp Suite.
After obtaining the administrator's password, it was possible to log in as the administrator and access privileged functionality.
This lab demonstrates how a horizontal access control vulnerability can become a vertical privilege escalation when the targeted account has higher privileges. It also shows that sensitive values must never be sent to the browser unless they are strictly required.
