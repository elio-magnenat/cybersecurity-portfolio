# Preventing Authentication Attacks

Authentication should be designed so that attackers cannot easily guess credentials, discover valid accounts, or bypass required authentication steps.

## Protect user credentials

All authentication traffic should use HTTPS, and attempted HTTP requests should be redirected to HTTPS.

Applications should also avoid exposing usernames or email addresses unnecessarily through public profiles, responses, or other application functionality.

## Do not rely on users for security

Strong authentication should be enforced by the application rather than depending entirely on users making secure choices.

Traditional complexity rules can still lead to predictable passwords such as `Password1!`. Password-strength estimation can provide better feedback and help reject weak passwords more effectively.

## Prevent username enumeration

Authentication responses should be as consistent as possible regardless of whether a username exists.

Applications should use:

- Generic and identical error messages
- The same HTTP status codes
- Similar response lengths
- Similar response times

This reduces the amount of information available to an attacker attempting to identify valid accounts.

## Implement robust brute-force protection

Applications should rate-limit repeated authentication attempts and make sure that attackers cannot bypass these controls by manipulating client-controlled headers such as `X-Forwarded-For`.

Additional defenses may include:

- Temporary delays
- Account or client lockouts where appropriate
- CAPTCHA challenges after repeated failures
- Anomaly detection
- Monitoring and alerting on suspicious authentication activity

Rate limiting should not be treated as a complete solution, but it should make automated attacks significantly more difficult.

## Audit verification logic

Authentication and verification logic should be reviewed carefully for bypassable checks.

A security check that can be skipped, reset, or manipulated may provide little meaningful protection.

Every step in a multi-stage authentication flow should be securely bound to the same server-side identity and authentication state.

## Protect supplementary authentication functionality

Password reset, password change, account recovery, persistent login, and similar features should receive the same level of security review as the main login page.

Attackers may be able to create their own accounts and explore these workflows in detail, making them an important part of the authentication attack surface.

## Implement proper multi-factor authentication

Multi-factor authentication should use independent authentication factors.

Verifying the same underlying factor multiple times does not provide true MFA.

SMS-based 2FA provides an additional factor but can be exposed to attacks such as SIM swapping. Dedicated authenticator applications, hardware tokens, or security keys generally provide stronger protection.

The 2FA logic itself must also be enforced on the server side so that protected resources cannot be accessed before all required factors have been successfully verified.

## Practical checklist

The application should:

- Enforce HTTPS for all authentication traffic and redirect HTTP requests to HTTPS
- Enforce strong password policies
- Encourage long and unique passwords instead of only requiring predictable complexity rules
- Use password-strength estimation instead of relying only on predictable complexity rules
- Rate-limit repeated login attempts
- Ensure rate limiting cannot be bypassed using attacker-controlled client IP headers
- Add temporary delays or lockouts after too many failed attempts
- Use multi-factor authentication for sensitive accounts or actions
- Make sure all authentication steps are completed before creating a fully authenticated session
- Return similar error messages for valid and invalid usernames
- Avoid exposing usernames or email addresses unnecessarily
- Protect password reset and account recovery mechanisms
- Apply the same security requirements to password reset, password change, account recovery, and other supplementary authentication functionality
- Thoroughly audit authentication and verification logic for bypassable checks
- Store passwords securely using strong password hashing
- Monitor and log suspicious authentication attempts

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

[Back to Authentication Vulnerabilities](README.md)
