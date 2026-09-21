# SQL Injection
SQL injection (SQLi) is a web vulnerability that allows an attacker to interfere with the SQL queries an application sends to its database.
If user-controlled input is included inside a database query without being handled safely, an attacker may be able to change the meaning of the query.
This can allow an attacker to access data that should normally be hidden, including information belonging to other users or sensitive application data.
Depending on the vulnerability, an attacker may also be able to modify or delete data stored in the database.
In more serious cases, SQL injection can affect the underlying server or other back-end systems, and may sometimes be used to cause denial-of-service conditions.

## Topics

- [Detection and testing](detection.md)
- [UNION attacks](union-attacks.md)
- [Database enumeration](database-enumeration.md)
- [Blind SQL injection](blind-sqli.md)
- [Completed labs](labs.md)

## Common vulnerable patterns

### Retrieving hidden data
SQL injection can allow an attacker to modify a `WHERE` clause and retrieve data that the application normally hides.
For example, an application may use a query like:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

The condition `released = 1` ensures that only released products are displayed.
If the application directly includes user-controlled input in the query, an attacker may inject SQL syntax such as:

```text
Gifts'--
```

This can produce a query like:

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

In SQL, `--` starts a comment, so the rest of the query is ignored. This removes the `released = 1` condition and can expose products that should remain hidden.
Another example is:

```text
Gifts' OR 1=1--
```

This can produce:

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

Because `1=1` is always true, the query may return all products.
Care must be taken when testing conditions such as `OR 1=1`, because the same input could also be reused in other SQL queries such as `UPDATE` or `DELETE`, which could modify or remove data.

### Subverting application logic
SQL injection can also be used to change the logic of an application.
For example, a login form may use a query like:

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```

The login succeeds only if the query returns a matching user.
If the username is vulnerable to SQL injection, an attacker may use:

```text
administrator'--
```

with an empty password.
This can produce a query like:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

Because `--` comments out the rest of the query, the password check is removed.
The application may then authenticate the attacker as the `administrator` user without requiring the correct password.

### Second-order SQL injection

Second-order SQL injection, also called stored SQL injection, occurs when malicious input is stored safely at first but is later reused inside a SQL query in an unsafe way.

In a first-order SQL injection, the application immediately inserts user-controlled input from the current HTTP request into a vulnerable SQL query.

In a second-order SQL injection, the process happens in two stages:

1. The application receives user-controlled input and stores it, usually in a database.
2. Later, the application retrieves that stored value and uses it to build another SQL query unsafely.

The initial storage operation may be secure and use parameterized queries, so no vulnerability is triggered at that moment.

The problem appears later when developers assume that data already stored in the database is trusted and concatenate it directly into a new SQL query.

For example:

```text
User input
    ↓
Safely stored in database
    ↓
Retrieved later
    ↓
Inserted unsafely into another SQL query
    ↓
SQL injection
```

The important lesson is that data should not be considered safe simply because it came from the application's own database.

Stored values that originally came from users must still be handled safely whenever they are used in SQL queries.

## Impact
SQL injection can have a serious impact because it allows an attacker to interfere directly with database queries.
Depending on the vulnerability, an attacker may be able to:

- Access sensitive data that should normally be hidden.
- Bypass authentication.
- Read information belonging to other users.
- Modify or delete data stored in the database.
- Change the behavior of the application.
- In severe cases, interact with other back-end systems or the underlying server.

The exact impact depends on the database permissions and on how the application uses the vulnerable query.

## Prevention

The main defense against SQL injection is to avoid constructing SQL queries by concatenating untrusted input.

Applications should use parameterized queries, also known as prepared statements.

A vulnerable query may be built like this:

```java
String query =
    "SELECT * FROM products WHERE category = '" + input + "'";
```

Because the user-controlled value is inserted directly into the SQL string, it can change the structure of the query.

A safer approach is:

```java
PreparedStatement statement =
    connection.prepareStatement(
        "SELECT * FROM products WHERE category = ?"
    );

statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```

The SQL structure remains fixed, while the user-controlled input is supplied separately as data.

Parameterized queries should be used whenever untrusted input represents a value, including:

- `WHERE` conditions.
- Values used in `INSERT` statements.
- Values used in `UPDATE` statements.

They cannot normally be used for structural elements such as:

- Table names.
- Column names.
- `ORDER BY` expressions.

When user input affects these parts of a query, the application should use a different design, such as strict whitelisting of permitted values.

The SQL query template itself should remain a hard-coded constant and should never contain dynamically concatenated data.

Developers should not assume that some values are safe simply because they came from an internal source or were previously stored in the database.

Input validation and least-privileged database accounts are useful additional defenses, but they should not replace parameterized queries.

## Detection and testing
SQL injection can be tested by identifying inputs that may be included in database queries.
Interesting inputs can include:
- URL parameters.
- Form fields.
- Login credentials.
- Search fields.
- Filters and category parameters.

A tester can modify these values and observe whether SQL syntax changes the application's response or behavior.
Examples of useful indicators include:

- Unexpected database errors.
- Different application responses after changing quotes or conditions.
- Hidden data becoming visible.
- Authentication being bypassed.
- A condition such as `OR 1=1` changing the number of returned results.

Testing should be done carefully because injected input may sometimes affect queries such as `UPDATE` or `DELETE`, which could modify data.

## Classification
- **CWE:** CWE-89 — Improper Neutralization of Special Elements used in an SQL Command
- **OWASP Top 10 2025:** A05 — Injection
- **Category:** Server-side vulnerability
- **Main target:** Database queries
- **Possible impact:** Data disclosure, authentication bypass, data modification or deletion

## Labs

- [Completed labs](labs.md)

## References
- [PortSwigger Web Security Academy — SQL injection](https://portswigger.net/web-security/sql-injection)
- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [OWASP Cheat Sheet Series — SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [OWASP Web Security Testing Guide — Testing for SQL Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_SQL_Injection)
- [MITRE — CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html)