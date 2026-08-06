# Broken Access Control
Access control determines what a user is allowed to access or do in an application.
It depends on three main elements:
- **Authentication** verifies the user's identity.
- **Session management** links later HTTP requests to the same user.
- **Access control** checks whether this user has permission to perform the requested action.

Broken access control happens when these permissions are not correctly enforced. This can allow users to access data or features that should not be available to them.
## Vertical privilege escalation
Vertical privilege escalation happens when a user gains access to features reserved for a more privileged role.
For example, a normal user may be able to access an administrator page and perform actions such as deleting user accounts.

## Common vulnerable patterns
### Unprotected functionality
Some sensitive features are hidden from normal users but are not protected by a server-side access control check.
For example, an administrator page may not appear in a normal user's interface, but the user may still be able to access it directly by entering its URL.
Hiding a URL is not a security control. The server must verify the user's permissions every time a protected resource or action is requested.
### Security by obscurity
Some applications try to protect sensitive functionality by using an unpredictable URL.
For example, an admin panel may use a path such as:
`/administrator-panel-yb556`
This makes the endpoint harder to guess, but it does not provide real access control.
The hidden URL may still be exposed in JavaScript files, `robots.txt`, error messages, API responses, or other public resources.
Even if the URL remains secret, the server must still verify that the user has permission to access the functionality.

### Parameter-based access control
Some applications store the user's role or permissions in a value that the user can modify.
This value may be stored in:
- A hidden form field
- A cookie
- A query string parameter
For example:

```text
?admin=true
?role=1
```
If the server trusts this value without checking the user's real permissions, a normal user may modify it and gain access to administrative functionality.
Access control decisions should never rely only on user-controlled data. The server must verify the user's permissions using trusted server-side information.
## Impact


## Prevention

## Detection and testing

## Classification


## Labs completed

### PortSwigger — Unprotected admin functionality
- **Difficulty:** Apprentice
- **Vulnerability:** Unprotected administrative functionality
- **Result:** Solved

The application exposed an administrative panel without checking whether the user had administrator permissions.
The panel location was disclosed in the `robots.txt` file. After accessing the admin endpoint directly, it was possible to use administrative functionality as a normal unauthenticated user.
This lab demonstrates that hiding a sensitive URL is not an access control mechanism. The server must verify the user's permissions before displaying the page and before performing every sensitive action.

### PortSwigger — Unprotected admin functionality with unpredictable URL
- **Difficulty:** Apprentice
- **Vulnerability:** Unprotected administrative functionality / security by obscurity
- **Result:** Solved

The application used an unpredictable URL for the admin panel, but the location was exposed in client-side JavaScript.
By reviewing the page source, it was possible to discover the hidden endpoint and access the administrative functionality without being an administrator.
This lab demonstrates that using a hard-to-guess URL does not replace proper access control. Sensitive endpoints must always be protected by server-side permission checks.

### PortSwigger — User role controlled by request parameter

- **Difficulty:** Apprentice
- **Vulnerability:** Parameter-based access control
- **Result:** Solved

The application stored the user's administrative role in a cookie controlled by the browser.
By modifying the cookie value, it was possible to change the user's role and access the admin panel without having real administrator permissions.
This lab demonstrates that access control decisions must not rely on data controlled by the user. Roles and permissions should be verified using trusted server-side information.

## References

