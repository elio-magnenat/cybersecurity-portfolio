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

## Horizontal privilege escalation
Horizontal privilege escalation happens when a user can access data or actions belonging to another user with the same level of permissions.
For example, a user may normally access their own account with a URL such as:
```text
/myaccount?id=123
```
If changing the id value allows access to another user's account, the application does not correctly verify who owns the requested resource.
This is a common example of an IDOR vulnerability.
### Insecure Direct Object Reference (IDOR)
An IDOR vulnerability happens when the application uses a user-controlled identifier to access a resource directly without checking whether the current user is allowed to access it.
The identifier may represent:
- A user account
- A document
- An invoice
- A message
- An order
- A file

The value does not need to be predictable. Applications may use random identifiers or GUIDs, but this is not enough to prevent the vulnerability.
An attacker may discover another user's identifier in messages, reviews, URLs, API responses, or other parts of the application.
The server must always check that the current user is authorized to access the requested resource.

### Horizontal to vertical privilege escalation
A horizontal privilege escalation can sometimes lead to a vertical privilege escalation.
For example, an attacker may first access another user's account by modifying a user identifier. If the targeted account belongs to an administrator, the attacker may then gain access to administrative features.
The compromised account may also expose sensitive information, allow the attacker to change the password, or provide another way to take control of the account.
This shows that a horizontal access control vulnerability can have a much greater impact when a privileged user is targeted.

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
Broken access control can allow attackers to access data or perform actions outside their normal permissions.
Possible impacts include:
- Reading another user's private data
- Modifying or deleting another user's information
- Accessing administrative pages or functions
- Changing account settings or passwords
- Taking control of another user's account
- Performing privileged actions as an administrator
- Accessing sensitive business information
- Causing financial, legal, or reputational damage

The impact depends on the affected functionality and the privileges of the targeted account.
A horizontal privilege escalation may expose another user's data, while a vertical privilege escalation may give access to administrative functions. In some cases, a horizontal vulnerability can become vertical if the attacker compromises a more privileged account.
Broken access control often has a high impact because it directly affects confidentiality, integrity, and sometimes availability.
## Prevention
Access control must always be enforced on the server side.
The application should:
- Check the user's permissions for every protected page, resource, and action
- Deny access by default when no explicit permission is granted
- Avoid trusting user-controlled values such as cookies, hidden fields, or query parameters
- Store roles and permissions in trusted server-side data
- Verify that the current user owns or is allowed to access the requested resource
- Apply access control checks to both the user interface and the underlying API
- Use centralized authorization logic to avoid inconsistent checks
- Recheck permissions before sensitive actions such as deleting users, changing passwords, or accessing administrative functions
- Avoid exposing sensitive endpoints or role information unnecessarily
- Log and monitor failed authorization attempts

Hiding a button, page, or URL is not enough. Even if a feature is not visible in the interface, the server must still verify that the user is authorized to access it.
Unpredictable identifiers such as GUIDs may make resources harder to guess, but they do not replace authorization checks.

## Detection and testing
I first identify the different user roles and the actions that each role should be allowed to perform.
I then test protected pages, resources, and API endpoints using users with different permission levels. I compare the requests and responses to see whether the server correctly enforces access control.
I pay particular attention to:
- Administrative pages and functions
- User identifiers in URLs, forms, or JSON data
- Cookies or parameters containing roles or permissions
- Direct links to documents, invoices, messages, or account pages
- Sensitive actions such as changing passwords, deleting users, or modifying data

Using Burp Suite, I intercept a valid request and send it to Repeater. I then modify one element at a time, such as:
- The user or object identifier
- The requested URL
- The HTTP method
- A role-related cookie or parameter
- The account used to send the request

I also test whether a normal user can:
- Access an administrator endpoint directly
- Access another user's resource
- Perform an action that is hidden in the interface
- Reuse a request created by a more privileged user
- Call the underlying API without using the visible user interface

I compare the HTTP status code, response body, response length, redirects, and error messages.
A successful test occurs when the server returns protected data or performs a sensitive action without verifying that the current user has the required permissions.

## Classification

- **Common name:** Broken Access Control
- **OWASP Top 10:2025:** A01 — Broken Access Control
- **General CWE family:** CWE-284 — Improper Access Control
- **OWASP WSTG category:** Authorization Testing

More specific CWE identifiers may be used depending on the exact vulnerability:

- **CWE-862:** Missing Authorization
- **CWE-863:** Incorrect Authorization
- **CWE-639:** Authorization Bypass Through User-Controlled Key
- **CWE-425:** Direct Request, also known as forced browsing

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

### PortSwigger — User ID controlled by request parameter with unpredictable IDs
- **Difficulty:** Apprentice
- **Vulnerability:** Horizontal privilege escalation / IDOR
- **Result:** Solved

The application used GUIDs to identify users, which made the identifiers difficult to guess.
However, another user's GUID was exposed elsewhere in the application through public content. After finding this identifier, it was possible to modify the account request and access data belonging to that user.
This lab demonstrates that unpredictable identifiers do not replace authorization checks. The server must verify that the authenticated user is allowed to access the requested account or resource.

### PortSwigger — User ID controlled by request parameter with password disclosure
- **Difficulty:** Apprentice
- **Vulnerability:** Horizontal to vertical privilege escalation
- **Result:** Solved

The application allowed a user to access another account by modifying the user identifier in the request.
The administrator's account page exposed the current password inside a masked form field. Even if the password was not visible in the interface, it was still present in the HTTP response and could be read using Burp Suite.
After obtaining the administrator's password, it was possible to log in as the administrator and access privileged functionality.
This lab demonstrates how a horizontal access control vulnerability can become a vertical privilege escalation when the targeted account has higher privileges. It also shows that sensitive values must never be sent to the browser unless they are strictly required.

## References
- [PortSwigger Web Security Academy — Access control vulnerabilities and privilege escalation](https://portswigger.net/web-security/access-control)
- [PortSwigger Web Security Academy — Insecure direct object references (IDOR)](https://portswigger.net/web-security/access-control/idor)
- [OWASP Top 10:2025 — A01: Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/)
- [OWASP Web Security Testing Guide — Authorization Testing](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/README)
- [OWASP WSTG — Testing for Bypassing Authorization Schema](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/05-Authorization_Testing/02-Testing_for_Bypassing_Authorization_Schema)
- [OWASP WSTG — Testing for Insecure Direct Object References](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
- [OWASP Cheat Sheet — Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [MITRE — CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html)
- [MITRE — CWE-862: Missing Authorization](https://cwe.mitre.org/data/definitions/862.html)
- [MITRE — CWE-863: Incorrect Authorization](https://cwe.mitre.org/data/definitions/863.html)
- [MITRE — CWE-639: Authorization Bypass Through User-Controlled Key](https://cwe.mitre.org/data/definitions/639.html)
- [MITRE — CWE-425: Direct Request (Forced Browsing)](https://cwe.mitre.org/data/definitions/425.html)
