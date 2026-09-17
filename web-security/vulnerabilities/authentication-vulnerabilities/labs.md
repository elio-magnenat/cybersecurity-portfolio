# Authentication Labs

## Password-based authentication

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

## Multi-factor authentication

### PortSwigger — 2FA simple bypass
- **Difficulty:** Apprentice
- **Vulnerability:** Two-factor authentication bypass
- **Result:** Solved

The application asked for a 2FA verification code after the password step, but it did not correctly enforce this second authentication step before allowing access to the account page.
After logging in with valid credentials, it was possible to directly access the protected account page without entering the verification code.
This lab demonstrates that multi-factor authentication must be enforced on the server side for every protected resource. A user should only be considered fully authenticated after all required authentication steps have been completed.

### PortSwigger — 2FA broken logic

- **Difficulty:** Practitioner
- **Vulnerability:** Broken Two-Factor Authentication Logic
- **Result:** Solved
- **Tools used:** Burp Suite Repeater / Python

The application used a client-controlled `verify` value to determine which account the second authentication step belonged to.

I first authenticated with my own account and inspected the 2FA flow. I identified that the `verify` parameter was used to select the account associated with the verification code.

I then changed this value to the victim's username and triggered the generation of a temporary 2FA code for that account.

After returning to the login flow with my own valid credentials, I submitted requests to `/login2` while keeping the victim's username in the `verify` value and brute-forcing the `mfa-code` parameter.

A successful code produced an HTTP `302` response, which allowed access to the victim's account.

This lab demonstrated that multi-step authentication must securely bind every step to the same server-side user identity. Relying on client-controlled values such as cookies or request parameters can allow attackers to complete the second authentication step for another account without knowing that user's password.

## Other authentication mechanisms

### PortSwigger — Brute-forcing a stay-logged-in cookie

- **Difficulty:** Practitioner
- **Vulnerability:** Predictable Persistent Authentication Token
- **Result:** Solved
- **Tools used:** Burp Suite / Python

The application used a persistent `stay-logged-in` cookie to keep users authenticated after closing their browser.

By inspecting my own cookie, I identified that it was Base64-encoded and followed the structure:

```text
base64(username:md5(password))
```

Because the cookie was derived from predictable values and used an unsalted MD5 hash of the password, it was possible to generate valid candidate cookies offline.

I then targeted the victim account by generating cookies using:

```text
base64(carlos:md5(candidate-password))
```

and testing each candidate against the victim's account page.

A successful request returned the authenticated account page, confirming the correct password-derived cookie.

This lab demonstrated that persistent authentication tokens must not be generated from predictable user data or password hashes. Remember-me tokens should instead use cryptographically secure random values with proper expiration and revocation controls.

### PortSwigger — Offline password cracking

- **Difficulty:** Practitioner
- **Vulnerability:** Stored XSS / Weak Persistent Authentication Token
- **Result:** Solved
- **Tools used:** Burp Suite / Browser DevTools

The application stored a password-derived value inside the persistent `stay-logged-in` cookie.

By inspecting my own cookie, I confirmed that the token was Base64-encoded and followed the structure:

```text
username:md5(password)
```

The comment functionality was also vulnerable to stored XSS.

I used this XSS vulnerability to make the victim's browser expose its cookies when the victim viewed the affected blog post. This allowed me to obtain Carlos's `stay-logged-in` cookie.

After Base64-decoding the cookie, I recovered a value containing the victim username and an MD5 hash of the password.

Because the password hash was unsalted and based on a weak password, it could be cracked offline using known password hashes or password-cracking tools.

Once the password was recovered, I authenticated as the victim and deleted the account to solve the lab.

This lab demonstrated how multiple weaknesses can be chained together. A stored XSS vulnerability can expose persistent authentication cookies, while password-derived tokens and unsalted hashes can turn cookie theft into full credential recovery.

### PortSwigger — Password reset broken logic

- **Difficulty:** Apprentice
- **Vulnerability:** Broken Password Reset Logic
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application used a password reset token in the reset URL, but failed to validate the token when the new password was finally submitted.

I first requested a password reset for my own account and inspected the reset flow in Burp Suite.

The final password change request contained both a reset token and a client-controlled `username` parameter.

By removing the reset token and resending the request, I confirmed that the application still accepted the password change.

I then changed the `username` parameter to the victim account and submitted a new password.

Because the server trusted the supplied username without requiring a valid reset token, the victim's password was changed successfully.

I then logged in as the victim and accessed the account page to solve the lab.

This lab demonstrated that reset tokens must be validated at the final password-change step and securely bound to the intended user. A reset flow is vulnerable if the server relies on client-controlled account identifiers after the token has been omitted or invalidated.

### PortSwigger — Password reset poisoning via middleware

- **Difficulty:** Practitioner
- **Vulnerability:** Password Reset Poisoning / Host Header Injection
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application generated password reset links dynamically and trusted the `X-Forwarded-Host` header when constructing the URL.

I first inspected the password reset flow and confirmed that a unique reset token was sent to the user by email.

Using Burp Repeater, I added an attacker-controlled `X-Forwarded-Host` value to the `POST /forgot-password` request.

The application then generated a password reset link using the supplied host instead of the legitimate application domain.

I targeted the victim account and caused the reset email to contain a link pointing to my exploit server.

When the victim followed this poisoned link, the browser sent the reset token to my server as part of the URL.

I recovered the victim's token from the exploit server access logs and reused it with a legitimate reset URL.

This allowed me to set a new password for the victim account and log in successfully.

This lab demonstrated that password reset URLs must be generated from trusted server-side configuration. Headers such as `X-Forwarded-Host` must never be trusted blindly when constructing security-sensitive links.

### PortSwigger — Password brute-force via password change

- **Difficulty:** Practitioner
- **Vulnerability:** Password Brute Force / Password Change Logic Flaw
- **Result:** Solved
- **Tool used:** Burp Suite Intruder

The password change functionality exposed a logic flaw that made it possible to brute-force another user's current password.

The request contained a client-controlled `username` parameter together with the current password and the two new password fields.

I observed that the application returned different error messages depending on whether the supplied current password was correct.

When the current password was incorrect and the two new passwords were different, the application returned:

```text
Current password is incorrect
```

When the current password was correct but the two new passwords did not match, it instead returned:

```text
New passwords do not match
```

I changed the username to the victim account, placed a payload position on the `current-password` parameter, and used Burp Intruder to test the provided password list.

I added a grep match rule for `New passwords do not match`, which allowed me to identify the single request where the current password was valid.

I then used the recovered password to log in to the victim account and solve the lab.

This lab demonstrated that password-change functionality can become a password-verification oracle when error messages reveal whether the supplied current password is valid. Sensitive account-management endpoints must verify the authenticated user securely and avoid exposing distinguishable authentication states.

[Back to Authentication Vulnerabilities](README.md)
