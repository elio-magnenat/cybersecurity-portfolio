# Authentication Vulnerabilities
Authentication is the process used by an application to verify a user's identity.
Authentication vulnerabilities happen when this process is weak, incorrectly implemented, or can be bypassed. They may allow an attacker to access another user's account, sensitive data, or protected functionality.
Authentication vulnerabilities generally arise either because the mechanism is too weak against attacks such as brute force, or because implementation and logic flaws allow the authentication process to be bypassed entirely. This is often referred to as broken authentication.
These vulnerabilities can affect different authentication mechanisms, such as passwords, password reset flows, multi-factor authentication, and session-related processes.
This section covers common authentication weaknesses, their impact, how to detect them, and how they can be prevented.

### Authentication factors

Authentication mechanisms generally rely on one or more types of factors:

- **Something you know:** for example, a password, PIN, or answer to a security question.
- **Something you have:** for example, a mobile phone, hardware token, or security key.
- **Something you are:** for example, fingerprints, facial recognition, or other biometric characteristics.

Using multiple independent factors can provide stronger protection than relying on a password alone.

## Authentication vs authorization
Authentication verifies that a user is who they claim to be.
Authorization checks what an authenticated user is allowed to access or do.
For example, authentication verifies that the person logging in as `Carlos123` is really the owner of that account. Once the user is authenticated, authorization determines whether they can access another user's data or perform sensitive actions such as deleting an account.

## Common vulnerable patterns

### Password-based authentication

In a password-based authentication system, a user proves their identity by providing a username and a secret password associated with the account.

The security of this mechanism depends on keeping the password secret. If an attacker can obtain or correctly guess another user's credentials, they may be able to authenticate as that user.

Common attack paths include credential guessing, brute-force attacks, leaked or reused passwords, and weaknesses in the protections designed to limit repeated login attempts.

### Brute-force attacks
A brute-force attack happens when an attacker repeatedly tries different usernames, passwords, or both until valid credentials are found.
These attacks are usually automated with wordlists and dedicated tools, which makes it possible to test many login attempts quickly.
Brute force does not always use random values. Attackers may use common passwords, leaked credentials, personal information, or predictable username formats to make their guesses more effective.
Applications that only use password-based authentication and do not have enough protection against repeated login attempts are especially vulnerable.

#### Brute-forcing usernames
Usernames can be easy to guess when they follow a predictable format.
For example, company accounts often use patterns such as:
```text
firstname.lastname@company.com
```
Privileged accounts may also use common names such as:
- admin
- administrator
- support

During testing, I check whether usernames or email addresses are exposed publicly.
Possible sources include:
- Public user profiles
- Author names
- Reviews or comments
- Error messages
- API responses
- HTTP response bodies
- Contact or support pages

Even when a profile is partially hidden, the displayed name may be the same as the login username.
Discovering valid usernames can make password brute-force attacks more effective because the attacker no longer needs to guess both the username and the password.

#### Brute-forcing passwords
Passwords can also be guessed through repeated login attempts.
The difficulty depends mainly on the strength of the password. A long and unpredictable password is much harder to guess than a short or common one.
Many websites use password policies that require:
- A minimum number of characters
- Lowercase and uppercase letters
- At least one number
- At least one special character

These rules can make passwords harder to guess, but they are not always enough. Users may still create predictable passwords such as `Password123!`.
Weak password policies, common passwords, reused passwords, and insufficient protection against repeated login attempts can make brute-force attacks more effective.
Even when a password policy requires uppercase letters, numbers, and special characters, users may still create predictable passwords.

Instead of choosing a truly random password, they often modify an easy password to match the rules, for example:
```text
Mypassword1!
Myp4$$w0rd
```
When users are required to change their password regularly, they may also make only small changes, such as replacing one character or increasing a number.
These predictable habits make brute-force attacks more effective because attackers can test likely variations instead of trying every possible combination.

#### Flawed brute-force protection

Brute-force protections commonly rely on account lockouts or IP-based rate limiting after repeated failed login attempts.

These controls can be ineffective when their logic can be reset or bypassed.

For example, some applications reset the failed-attempt counter for an IP address after a successful login. If an attacker has access to their own valid account, they may be able to insert a legitimate login after every few failed attempts and prevent the protection threshold from ever being reached.

This demonstrates that brute-force defenses must not only exist, but must also be designed so that attackers cannot reset or manipulate their state during an attack.

#### Account locking

Account locking is commonly used to slow down brute-force attacks by temporarily locking an account after a certain number of failed login attempts.

However, the lockout behavior itself can introduce weaknesses.

If the application responds differently when an account becomes locked, this can reveal that the username is valid and therefore support username enumeration.

Account locking also mainly protects against repeated attempts against one specific account. It is less effective when an attacker distributes a small number of password guesses across many usernames.

For example, if an account is locked after three failed attempts, an attacker can try three highly probable passwords against a large list of usernames without exceeding the lockout threshold for any single account.

Account locking also does not effectively prevent credential stuffing attacks. In credential stuffing, the attacker tests previously leaked `username:password` pairs against the application. Because each username may only be attempted once, account lockout thresholds may never be reached.

This means that account lockout should be combined with other protections such as rate limiting, anomaly detection, multi-factor authentication, and monitoring of suspicious login patterns.

#### User rate limiting

User rate limiting is another defense against brute-force attacks. Instead of locking a specific account, the application limits or blocks login requests coming from the same client IP after too many attempts within a short period of time.

The block may be removed automatically after a delay, manually by an administrator, or after the user completes an additional challenge such as a CAPTCHA.

Compared with account locking, IP-based rate limiting is generally less likely to reveal whether a username exists and is less vulnerable to account-targeted denial-of-service attacks.

However, its effectiveness depends on how reliably the application identifies the client. If the application trusts attacker-controlled headers such as `X-Forwarded-For`, an attacker may be able to change their apparent IP address and bypass the rate limit.

Rate limiting can also be weakened if a single HTTP request allows multiple password guesses, because the application may count requests rather than individual authentication attempts.

Rate limiting should therefore be combined with other protections such as multi-factor authentication, anomaly detection, secure proxy configuration, and monitoring of suspicious authentication activity.

#### HTTP Basic Authentication

HTTP Basic Authentication is a simple authentication mechanism where the client sends a username and password in the `Authorization` header.

The credentials are concatenated using the format:

```text
username:password
```

and then Base64-encoded:

```http
Authorization: Basic base64(username:password)
```

The browser typically stores these credentials and automatically includes the header in subsequent requests.

Base64 encoding does not provide encryption. The credentials can be decoded easily, so the security of HTTP Basic Authentication depends heavily on secure transport.

Because the same static credentials are repeatedly sent with requests, weak implementations may expose them to interception, brute-force attacks, or credential reuse.

HTTP Basic Authentication may also lack built-in protections against attacks such as brute force and CSRF, depending on how the application is implemented.

Even if the protected area appears low-value, recovered credentials may still be reused in other applications or more sensitive parts of the same environment.

### Username enumeration
Username enumeration happens when an application responds differently depending on whether a username exists.
This often appears on login pages. For example, the application may return one message for an unknown username and another message for a valid username with an incorrect password.
It can also happen on registration or password reset forms when the application reveals that an account already exists.
This helps an attacker create a list of valid usernames before attempting to guess passwords, which makes brute-force attacks faster and more effective.
Possible differences may include:
- Different error messages
- Different HTTP status codes
- Different response lengths
- Different response times
- Different redirects
- Different account lockout behavior

Timing differences can sometimes be amplified by submitting an unusually long password. If the application performs expensive password processing only when the username is valid, this can make the response-time difference easier to detect.

### Two-factor authentication bypass
Two-factor authentication can sometimes be bypassed when the application does not correctly verify that the second authentication step has been completed.
For example, a user may first enter a valid username and password, then be redirected to a page asking for a verification code.
If the application already considers the user authenticated after the password step, it may be possible to directly access pages that should only be available after completing 2FA.
This happens when the server protects the verification page, but does not verify the 2FA state again before allowing access to protected resources.
The application should only create a fully authenticated session after all required authentication steps have been successfully completed.

## Impact
Authentication vulnerabilities can allow attackers to access accounts they do not own.
Possible impacts include:
- Account takeover
- Access to private user data
- Unauthorized actions performed as another user
- Access to privileged or administrator accounts
- Bypassing multi-factor authentication
- Changing passwords or account settings
- Accessing sensitive business information
- Financial, legal, or reputational damage

The impact depends on the privileges of the compromised account and the type of application.
If an administrator account is compromised, the attacker may gain access to sensitive functions and data across the entire application.
Even a low-privileged account can increase the attack surface by exposing internal pages and functionality that are not accessible to unauthenticated users. These authenticated areas may contain additional vulnerabilities or provide stepping stones toward more severe attacks.
Authentication vulnerabilities mainly affect confidentiality and integrity, but they can also affect availability if the compromised account can delete data, disable users, or change critical settings.

## Prevention
Authentication should be designed so that attackers cannot easily guess credentials, discover valid accounts, or bypass required authentication steps.
The application should:
- Enforce strong password policies
- Encourage long and unique passwords instead of only requiring predictable complexity rules
- Rate-limit repeated login attempts
- Add temporary delays or lockouts after too many failed attempts
- Use multi-factor authentication for sensitive accounts or actions
- Make sure all authentication steps are completed before creating a fully authenticated session
- Return similar error messages for valid and invalid usernames
- Avoid exposing usernames or email addresses unnecessarily
- Protect password reset and account recovery mechanisms
- Store passwords securely using strong password hashing
- Monitor and log suspicious authentication attempts

For example, a login page should not reveal whether the username exists by returning different error messages such as `Invalid username` and `Incorrect password`.
Multi-factor authentication must also be enforced on the server side. A user should not be able to access protected pages until every required authentication step has been successfully completed.

## Detection and testing
I first identify all authentication-related features in the application, such as:
- Login pages
- Registration forms
- Password reset functions
- Multi-factor authentication
- Account recovery
- Session creation and logout

I then observe how the application reacts to valid and invalid authentication attempts.
Using Burp Suite, I intercept authentication requests and compare differences such as:
- Error messages
- HTTP status codes
- Response length
- Redirects
- Response time
- Cookies or session tokens

For username enumeration, I test whether the application responds differently when the username exists but the password is incorrect.
For brute-force protection, I check whether repeated failed login attempts are limited, delayed, blocked, or monitored.
I also verify whether authentication state is correctly enforced. For example, after completing only the password step of a multi-factor login, I test whether protected pages can still be accessed before the second factor is completed.
I pay particular attention to:
- User-controlled authentication parameters
- Predictable usernames
- Weak or inconsistent error messages
- Missing rate limiting
- Weak account lockout mechanisms
- Password reset flows
- Multi-factor authentication bypasses
- Session behavior before and after authentication

A successful test occurs when the application reveals useful authentication information, allows too many automated attempts, or grants access without completing all required authentication steps.

## Classification
- **Common name:** Authentication Vulnerabilities
- **OWASP Top 10:2025:** A07 — Authentication Failures
- **General CWE family:** CWE-287 — Improper Authentication
- **OWASP WSTG category:** Authentication Testing
More specific CWE identifiers may apply depending on the exact vulnerability:
- **CWE-307:** Improper Restriction of Excessive Authentication Attempts
- **CWE-204:** Observable Response Discrepancy
- **CWE-308:** Use of Single-factor Authentication
- **CWE-613:** Insufficient Session Expiration

## Labs completed
### PortSwigger — Username enumeration via different responses
- **Difficulty:** Apprentice
- **Vulnerability:** Username enumeration / brute-force
- **Result:** Solved
- **Tool used:** Burp Suite Intruder

The login page returned different responses depending on whether the username was valid.
By using Burp Intruder with a list of candidate usernames, I identified a valid account because the response was different from the others. Invalid usernames returned `Invalid username`, while the valid username returned `Incorrect password`.
I then kept the valid username and used a password wordlist to test possible passwords. One request returned a different HTTP status code (`302` instead of `200`), which indicated a successful login.
This lab demonstrates how small differences in error messages, response length, or HTTP status codes can reveal valid usernames and make password brute-force attacks much more efficient.
It also shows why authentication responses should be as consistent as possible and why login endpoints need protection against repeated automated attempts.

### PortSwigger — Username enumeration via subtly different responses

- **Difficulty:** Practitioner
- **Vulnerability:** Username Enumeration / Brute Force
- **Result:** Solved
- **Tool used:** Burp Suite Intruder

The login page returned almost identical error messages for valid and invalid usernames.

I used Burp Intruder with the provided username wordlist and compared the returned error messages.

Most invalid usernames returned:

```text
Invalid username or password.
```

One response was subtly different:

```text
Invalid username or password
```

The only difference was the missing final period, which revealed that this username was valid.

After identifying the valid username, I replaced the username payload with that account and brute-forced the password using the provided password wordlist.

One request returned HTTP `302` instead of the normal `200` responses, indicating a successful login.

I then used the recovered credentials to access the account page and solve the lab.

This lab demonstrated that username enumeration can rely on extremely small response differences, including punctuation, whitespace, or other subtle variations that may be difficult to notice manually.

### PortSwigger — Username enumeration via response timing

- **Difficulty:** Practitioner
- **Vulnerability:** Username Enumeration / Timing Attack
- **Result:** Solved
- **Tool used:** Burp Suite Intruder

The login mechanism was vulnerable to username enumeration through differences in response time.

The application also implemented IP-based brute-force protection. I identified that the `X-Forwarded-For` header was trusted, allowing me to spoof a different client IP for each request and bypass the protection.

I used a Pitchfork attack in Burp Intruder with two payload positions:
- An incrementing value in the `X-Forwarded-For` header
- A list of candidate usernames

I submitted an unusually long password to amplify timing differences. Invalid usernames produced similar response times, while one username consistently caused a noticeably slower response, indicating that the application was performing additional password processing for that account.

After confirming the valid username, I launched a second Pitchfork attack using:
- Incrementing spoofed IP addresses
- The candidate password wordlist

One request returned HTTP `302`, indicating a successful login.

I then used the identified credentials to access the account page and solve the lab.

This lab demonstrated how timing side channels can reveal valid usernames even when error messages and status codes remain consistent. It also showed how trusting client-controlled headers such as `X-Forwarded-For` can weaken IP-based brute-force protections.

### PortSwigger — Broken brute-force protection, IP block

- **Difficulty:** Practitioner
- **Vulnerability:** Broken Brute-force Protection / Logic Flaw
- **Result:** Solved
- **Tool used:** Burp Suite Intruder

The login mechanism temporarily blocked the client IP after several consecutive failed login attempts.

However, the failed-attempt counter was reset whenever a successful login occurred. This made it possible to bypass the protection by alternating legitimate logins to my own account with brute-force attempts against the victim account.

I generated an alternating payload list where each request to `carlos` was preceded by a valid login using my own credentials.

The requests followed this pattern:

```text
username=wiener&password=peter
username=carlos&password=<candidate-password>
username=wiener&password=peter
username=carlos&password=<next-candidate-password>
```

I then sent the sequence through Burp Intruder with requests processed in order.

After the attack completed, I filtered the results to keep only successful responses and sorted the payload column. This made it easy to identify the single successful request that targeted `carlos` rather than my own account.

I used the corresponding password to log in to Carlos's account and solve the lab.

This lab demonstrated that brute-force protections can be bypassed when their internal state can be reset by unrelated successful authentication events.

### PortSwigger — Username enumeration via account lock

- **Difficulty:** Practitioner
- **Vulnerability:** Username Enumeration / Account Lockout Logic Flaw
- **Result:** Solved
- **Tools used:** Burp Suite Intruder / Python

The login mechanism locked valid accounts after repeated failed authentication attempts.

I first tested each candidate username multiple times with an invalid password. Most usernames always returned the same response, while one username eventually triggered the message:

```text
You have made too many incorrect login attempts.
```

This revealed that the username was valid because only an existing account had a failed-login counter and could enter the locked state.

After identifying the valid username, I brute-forced the candidate password list. Normal incorrect passwords returned an error message, while the correct password produced a different response and ultimately an HTTP `302` redirect to the account page.

Because the lockout was temporary, I waited for it to reset before confirming the recovered credentials and accessing the account.

This lab demonstrated that account lockout mechanisms can unintentionally become a username-enumeration oracle. It also showed that lockout controls can still be bypassed or worked around when their behavior leaks useful authentication state.

### PortSwigger — 2FA simple bypass
- **Difficulty:** Apprentice
- **Vulnerability:** Two-factor authentication bypass
- **Result:** Solved

The application asked for a 2FA verification code after the password step, but it did not correctly enforce this second authentication step before allowing access to the account page.
After logging in with valid credentials, it was possible to directly access the protected account page without entering the verification code.
This lab demonstrates that multi-factor authentication must be enforced on the server side for every protected resource. A user should only be considered fully authenticated after all required authentication steps have been completed.

## References
- [PortSwigger Web Security Academy — Authentication vulnerabilities](https://portswigger.net/web-security/authentication)
- [PortSwigger Web Security Academy — Password-based authentication](https://portswigger.net/web-security/authentication/password-based)
- [PortSwigger Web Security Academy — Multi-factor authentication](https://portswigger.net/web-security/authentication/multi-factor)
- [OWASP Top 10:2025 — A07: Authentication Failures](https://owasp.org/Top10/2025/A07_2025-Authentication_Failures/)
- [OWASP Web Security Testing Guide — Authentication Testing](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/README)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Multifactor Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
- [MITRE — CWE-287: Improper Authentication](https://cwe.mitre.org/data/definitions/287.html)
- [MITRE — CWE-307: Improper Restriction of Excessive Authentication Attempts](https://cwe.mitre.org/data/definitions/307.html)
