# Authentication Vulnerabilities

Authentication is the process used by an application to verify a user's identity.

Authentication vulnerabilities happen when this process is weak, incorrectly implemented, or can be bypassed. They may allow an attacker to access another user's account, sensitive data, or protected functionality.

Authentication vulnerabilities generally arise either because the mechanism is too weak against attacks such as brute force, or because implementation and logic flaws allow the authentication process to be bypassed entirely. This is often referred to as broken authentication.

These vulnerabilities can affect different authentication mechanisms, such as passwords, password reset flows, multi-factor authentication, and session-related processes.

## Topics

- [Password-based authentication](password-based-authentication.md)
- [Multi-factor authentication](multi-factor-authentication.md)
- [Other authentication mechanisms](other-authentication-mechanisms.md)
- [Prevention and testing](prevention.md)
- [Completed labs](labs.md)

## Authentication factors

Authentication mechanisms generally rely on one or more types of factors:

- **Something you know:** for example, a password, PIN, or answer to a security question.
- **Something you have:** for example, a mobile phone, hardware token, or security key.
- **Something you are:** for example, fingerprints, facial recognition, or other biometric characteristics.

Using multiple independent factors can provide stronger protection than relying on a password alone.

## Authentication vs authorization

Authentication verifies that a user is who they claim to be.

Authorization checks what an authenticated user is allowed to access or do.

For example, authentication verifies that the person logging in as `Carlos123` is really the owner of that account. Once the user is authenticated, authorization determines whether they can access another user's data or perform sensitive actions such as deleting an account.

## Main attack surfaces

Authentication weaknesses commonly appear in:

- Password-based login
- Brute-force protections
- Username enumeration behavior
- Multi-factor authentication flows
- Persistent login mechanisms
- Password reset and recovery flows
- Password-change functionality
- Session creation and authentication-state handling

The detailed theory for each area is documented in the linked topic files above.

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
