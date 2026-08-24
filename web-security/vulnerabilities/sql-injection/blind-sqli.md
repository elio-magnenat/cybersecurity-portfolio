# Blind SQL Injection
Blind SQL injection occurs when an application is vulnerable to SQL injection, but the results of the injected query are not directly visible in the HTTP response.
The application may also hide database error messages.
Because of this, techniques such as `UNION` attacks are usually not useful, since they depend on displaying the results of the injected query.
Blind SQL injection can still be exploited, but the attacker has to infer information indirectly by observing changes in the application's behavior, response content, response time, or external interactions.

## Conditional responses
Blind SQL injection can sometimes be exploited by making the application respond differently depending on whether an injected condition is true or false.
For example, an application may use a tracking cookie inside a SQL query:

```sql
SELECT TrackingId FROM TrackedUsers
WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```

If a valid tracking ID is found, the application may display a message such as:

```text
Welcome back
```

This difference can be used as a true/false signal.

For example:

```text
xyz' AND '1'='1
```

The condition is true, so the application behaves as if the tracking ID matched.

But:

```text
xyz' AND '1'='2
```

is false, so the response changes.
This makes it possible to ask the database yes/no questions and infer information one piece at a time.

### Extracting data character by character
A condition can also test individual characters from a value.
For example:

```text
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'),1,1) = 's
```

This means:
- Retrieve the administrator password.
- Take its first character.
- Test whether this character is `s`.

If the application's "true" response appears, the first character is `s`.

Characters can also be compared using conditions such as:

```text
> 'm'
```

This allows the possible character range to be reduced progressively instead of testing every character one by one.
The same process can then be repeated for the second, third, and following characters until the full value is recovered.
The exact substring function depends on the database system. Some databases use `SUBSTRING`, while others use `SUBSTR`.

## Error-based SQL injection

Error-based SQL injection uses database errors as an observable signal to extract or infer information.

There are two main approaches:

- Triggering an error only when a specific condition is true. The presence or absence of the error can then be used as a boolean signal.
- Causing the database error itself to include sensitive data returned by an injected query.

The first approach is similar to conditional-response blind SQL injection, except that the signal is an error instead of a change in page content.

The second approach can sometimes turn an otherwise blind SQL injection vulnerability into one where extracted data becomes directly visible in the application's error response.

### Conditional errors

When normal application responses do not change depending on whether a SQL condition is true or false, database errors can be used as an alternative signal.

A `CASE` expression can execute different expressions depending on a condition.

For example:

```text
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
```

Because `1=2` is false, the expression returns `'a'` and no error occurs.

But:

```text
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

causes a divide-by-zero error because the condition is true.

This creates a boolean signal:

```text
True condition  -> database error
False condition -> normal response
```

The same technique can be used to infer sensitive data one character at a time:

```text
xyz' AND (SELECT CASE WHEN (Username='Administrator' AND SUBSTRING(Password,1,1)>'m') THEN 1/0 ELSE 'a' END FROM Users)='a
```

If the request causes an error, the tested condition is true. Otherwise, it is false.

The exact technique used to trigger an error depends on the database system.
