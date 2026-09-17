# Password-Based Authentication

In a password-based authentication system, a user proves their identity by providing a username and a secret password associated with the account.

The security of this mechanism depends on keeping the password secret. If an attacker can obtain or correctly guess another user's credentials, they may be able to authenticate as that user.

Common attack paths include credential guessing, brute-force attacks, leaked or reused passwords, and weaknesses in the protections designed to limit repeated login attempts.

## Brute-force attacks

A brute-force attack happens when an attacker repeatedly tries different usernames, passwords, or both until valid credentials are found.

These attacks are usually automated with wordlists and dedicated tools, which makes it possible to test many login attempts quickly.

Brute force does not always use random values. Attackers may use common passwords, leaked credentials, personal information, or predictable username formats to make their guesses more effective.

Applications that only use password-based authentication and do not have enough protection against repeated login attempts are especially vulnerable.

### Brute-forcing usernames

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

### Brute-forcing passwords

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

### Flawed brute-force protection

Brute-force protections commonly rely on account lockouts or IP-based rate limiting after repeated failed login attempts.

These controls can be ineffective when their logic can be reset or bypassed.

For example, some applications reset the failed-attempt counter for an IP address after a successful login. If an attacker has access to their own valid account, they may be able to insert a legitimate login after every few failed attempts and prevent the protection threshold from ever being reached.

This demonstrates that brute-force defenses must not only exist, but must also be designed so that attackers cannot reset or manipulate their state during an attack.

### Account locking

Account locking is commonly used to slow down brute-force attacks by temporarily locking an account after a certain number of failed login attempts.

However, the lockout behavior itself can introduce weaknesses.

If the application responds differently when an account becomes locked, this can reveal that the username is valid and therefore support username enumeration.

Account locking also mainly protects against repeated attempts against one specific account. It is less effective when an attacker distributes a small number of password guesses across many usernames.

For example, if an account is locked after three failed attempts, an attacker can try three highly probable passwords against a large list of usernames without exceeding the lockout threshold for any single account.

Account locking also does not effectively prevent credential stuffing attacks. In credential stuffing, the attacker tests previously leaked `username:password` pairs against the application. Because each username may only be attempted once, account lockout thresholds may never be reached.

This means that account lockout should be combined with other protections such as rate limiting, anomaly detection, multi-factor authentication, and monitoring of suspicious login patterns.

### User rate limiting

User rate limiting is another defense against brute-force attacks. Instead of locking a specific account, the application limits or blocks login requests coming from the same client IP after too many attempts within a short period of time.

The block may be removed automatically after a delay, manually by an administrator, or after the user completes an additional challenge such as a CAPTCHA.

Compared with account locking, IP-based rate limiting is generally less likely to reveal whether a username exists and is less vulnerable to account-targeted denial-of-service attacks.

However, its effectiveness depends on how reliably the application identifies the client. If the application trusts attacker-controlled headers such as `X-Forwarded-For`, an attacker may be able to change their apparent IP address and bypass the rate limit.

Rate limiting can also be weakened if a single HTTP request allows multiple password guesses, because the application may count requests rather than individual authentication attempts.

Rate limiting should therefore be combined with other protections such as multi-factor authentication, anomaly detection, secure proxy configuration, and monitoring of suspicious authentication activity.

## HTTP Basic Authentication

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

## Username enumeration

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

[Back to Authentication Vulnerabilities](README.md)
