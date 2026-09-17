# Other Authentication Mechanisms

Authentication security is not limited to the main login page.

Applications often provide additional account-management functionality such as:

- Password changes
- Password reset flows
- Account recovery mechanisms

These features can introduce vulnerabilities even when the primary login mechanism is secure.

An attacker who can create their own account may be able to study these workflows in detail and identify weaknesses in how identity is verified or how sensitive account changes are authorized.

All authentication-related functionality should therefore be protected with the same level of care as the main login process.

## Keeping users logged in

Many applications offer a "Remember me" or "Keep me logged in" feature that allows users to remain authenticated after closing their browser.

This is commonly implemented using a persistent cookie containing a long-lived authentication token.

Because possession of this cookie may allow the user to bypass the normal login process, the token must be unpredictable and resistant to brute-force attacks.

Weak implementations sometimes generate remember-me cookies from predictable values such as:

- The username
- A timestamp
- The user's password
- A predictable combination of static values

If an attacker can create their own account, they may be able to study their own cookie and determine how the token is generated.

Once the generation algorithm is understood, the attacker may be able to create or brute-force valid cookies for other users.

Encoding the cookie does not automatically make it secure. For example, Base64 provides no protection because it is only a reversible encoding.

Even hashed values may still be vulnerable if:

- The hashing algorithm is known
- No salt is used
- The underlying values are predictable
- Weak passwords are used as part of the token

If a persistent authentication cookie contains a hash derived from the user's password, weak passwords may sometimes be recovered offline.

Attackers can compare the hash against precomputed hashes of common password lists or generate hashes from candidate passwords until a match is found.

This is especially dangerous when:

- The password is common
- The hashing algorithm is known
- No salt is used
- The hash is exposed directly in the cookie

This demonstrates why password-derived authentication tokens are unsafe. Persistent tokens should be random and independent from the user's password.

An attacker may also obtain a remember-me cookie through another vulnerability such as XSS and use it to understand how the token is constructed.

Persistent authentication tokens should therefore be generated using cryptographically secure random values and should be protected by rate limiting, expiration, revocation, and secure cookie attributes.

## Resetting user passwords

Password reset functionality is inherently security-sensitive because the user cannot authenticate using their existing password.

The application must therefore rely on an alternative method to verify that the person requesting the reset is the legitimate account owner.

If this verification mechanism is weak, an attacker may be able to reset another user's password and take over the account without ever knowing the original credentials.

Password reset flows should therefore be treated as authentication mechanisms in their own right and protected with the same level of care as the main login process.

### Sending passwords by email

A secure application should never be able to send users their existing password because passwords should not be stored in a recoverable form.

Some applications instead generate a new password and send it by email.

This approach is still risky because email is not designed to securely transport or store long-lived authentication secrets.

If a generated password is sent by email, its security depends on controls such as:

- Very short expiration
- Requiring the user to change the password immediately
- Preventing the temporary password from remaining valid indefinitely

Persistent passwords sent through email may be exposed if the mailbox, synchronization channel, or intermediate communication path is compromised.

Password reset mechanisms should therefore prefer short-lived reset tokens over sending reusable passwords directly.

### Resetting passwords using a URL

Password reset flows commonly use a unique URL that allows the user to choose a new password.

Weak implementations may identify the account using a predictable parameter:

```text
/reset-password?user=victim-user
```

If the application trusts this value, an attacker may be able to change the username and reset another user's password.

A more secure design uses a high-entropy, unpredictable reset token:

```text
/reset-password?token=<random-token>
```

The server should associate this token with a specific user and validate it before allowing the password reset.

Reset tokens should be:

- Cryptographically unpredictable
- Bound to a single account
- Short-lived
- Invalidated immediately after use

The token must also be validated again when the reset form is submitted.

If the application validates the token only when displaying the reset page but not when processing the final password change request, an attacker may be able to remove or modify the token and reset an arbitrary user's password.

This demonstrates that every step of a password reset flow must enforce the same server-side authorization state.

### Password reset poisoning

Password reset links may also be vulnerable if the application dynamically constructs the reset URL using attacker-controlled request data.

For example, the application may use headers such as `Host` or `X-Forwarded-Host` when generating the password reset link.

If these values are trusted without validation, an attacker may be able to cause the application to generate a reset link that points to an attacker-controlled domain:

```text
https://attacker.example/reset-password?token=<secret-token>
```

The victim may then receive this poisoned link in a legitimate password reset email and click it.

Because the reset token is included in the URL, the victim's browser sends the token to the attacker's server.

The attacker can then reuse the stolen token on the legitimate application to reset the victim's password.

Password reset URLs should therefore be generated from trusted server-side configuration and must not rely on unvalidated client-controlled headers.

## Password-change functionality

Password-change endpoints are also part of the authentication attack surface.

If the application accepts a client-controlled account identifier or returns different messages depending on whether the supplied current password is correct, the endpoint can become a password-verification oracle.

Sensitive account-management actions should always be bound to the authenticated server-side identity and should avoid exposing distinguishable authentication states that can be abused for brute-force attacks.

[Back to Authentication Vulnerabilities](README.md)
