# Authentication Vulnerabilities
Authentication is the process used by an application to verify a user's identity.
Authentication vulnerabilities happen when this process is weak, incorrectly implemented, or can be bypassed. They may allow an attacker to access another user's account, sensitive data, or protected functionality.
These vulnerabilities can affect different authentication mechanisms, such as passwords, password reset flows, multi-factor authentication, and session-related processes.
This section covers common authentication weaknesses, their impact, how to detect them, and how they can be prevented.

## Authentication vs authorization
Authentication verifies that a user is who they claim to be.
Authorization checks what an authenticated user is allowed to access or do.
For example, authentication verifies that the person logging in as `Carlos123` is really the owner of that account. Once the user is authenticated, authorization determines whether they can access another user's data or perform sensitive actions such as deleting an account.

## Common vulnerable patterns
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

## Impact

## Prevention

## Detection and testing

## Classification

## Labs completed

## References
