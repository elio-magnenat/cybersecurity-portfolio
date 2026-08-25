# SQL Injection
SQL injection (SQLi) is a web vulnerability that allows an attacker to interfere with the SQL queries an application sends to its database.
If user-controlled input is included inside a database query without being handled safely, an attacker may be able to change the meaning of the query.
This can allow an attacker to access data that should normally be hidden, including information belonging to other users or sensitive application data.
Depending on the vulnerability, an attacker may also be able to modify or delete data stored in the database.
In more serious cases, SQL injection can affect the underlying server or other back-end systems, and may sometimes be used to cause denial-of-service conditions.

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
The main protection against SQL injection is to avoid building SQL queries by directly concatenating user-controlled input.
Applications should use parameterized queries, also called prepared statements.
Instead of inserting user input directly into SQL syntax, the application sends the SQL structure and the user-controlled values separately.
Input validation can also help by restricting values to the expected format, but it should not be the only protection.
The database account used by the application should also have only the permissions it really needs.

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

## Labs completed
### PortSwigger — SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
- **Difficulty:** Apprentice
- **Vulnerability:** SQL Injection
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a SQL injection vulnerability in the product category filter.
When a category was selected, the application used a query similar to:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

The condition `released = 1` was used to hide unreleased products.
I intercepted the category request with Burp Suite and modified the `category` parameter with:

```text
' OR 1=1--
```

The condition `OR 1=1` is always true, and `--` comments out the rest of the SQL query.
This removed the restriction on released products and caused the application to display hidden unreleased products, confirming the SQL injection vulnerability.

### PortSwigger — SQL injection vulnerability allowing login bypass
- **Difficulty:** Apprentice
- **Vulnerability:** SQL Injection
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a SQL injection vulnerability in the login function.
The objective was to bypass authentication and log in as the `administrator` user without knowing the password.
I intercepted the login request with Burp Suite and modified the `username` parameter with:

```text
administrator'--
```

The `'` character closed the username value in the SQL query, and `--` commented out the rest of the query, including the password check.
This allowed the application to authenticate me as the `administrator` user without providing the correct password.

### PortSwigger — SQL injection UNION attack, determining the number of columns returned by the query
- **Difficulty:** Practitioner
- **Vulnerability:** SQL Injection — UNION Attack
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a SQL injection vulnerability in the product category filter.
Because the query results were returned in the response, a `UNION` attack could be used. The objective was to determine how many columns were returned by the original query.
I intercepted the category request with Burp Suite and tested `UNION SELECT` payloads with an increasing number of `NULL` values:

```text
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

I continued adding `NULL` values until the error disappeared and the injected row was accepted.
This allowed me to determine the number of columns returned by the original query, which is required before constructing more advanced `UNION` attacks.

### PortSwigger — SQL injection UNION attack, finding a column containing text
- **Difficulty:** Practitioner
- **Vulnerability:** SQL Injection — UNION Attack
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a SQL injection vulnerability in the product category filter.
After determining that the original query returned three columns, the objective was to identify which column was compatible with string data.
I first confirmed the number of columns with:

```text
' UNION SELECT NULL,NULL,NULL--
```

I then replaced each `NULL` value one at a time with the random string provided by the lab:

```text
' UNION SELECT 'abcdef',NULL,NULL--
' UNION SELECT NULL,'abcdef',NULL--
' UNION SELECT NULL,NULL,'abcdef'--
```

When the request succeeded and the injected string appeared in the response, I identified a column that could contain text data.
This is an important step before using a `UNION` attack to retrieve textual information from other database tables.

### PortSwigger — SQL injection UNION attack, retrieving data from other tables

- **Difficulty:** Practitioner
- **Vulnerability:** SQL Injection — UNION Attack
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a SQL injection vulnerability in the product category filter.
The objective was to retrieve usernames and passwords from a separate `users` table and use the recovered credentials to log in as the `administrator` user.
I first confirmed that the original query returned two columns that could both contain text:

```text
' UNION SELECT 'abc','def'--
```

I then retrieved data from the `users` table using:

```text
' UNION SELECT username,password FROM users--
```

The application's response displayed the usernames and passwords returned by the injected query.
I used the recovered administrator credentials to successfully log in as the `administrator` user.

### PortSwigger — SQL injection UNION attack, retrieving multiple values in a single column
- **Difficulty:** Practitioner
- **Vulnerability:** SQL Injection — UNION Attack
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a SQL injection vulnerability in the product category filter.
The objective was to retrieve usernames and passwords from the `users` table, but only one returned column was compatible with text data.

I first confirmed the usable text column with:

```text
' UNION SELECT NULL,'abc'--
```

I then concatenated the `username` and `password` values into the same column:

```text
' UNION SELECT NULL,username||'~'||password FROM users--
```

The `||` operator concatenated both values and `~` was used as a separator.
The response displayed the usernames and passwords, allowing me to recover the administrator credentials and log in successfully.

### PortSwigger — SQL injection attack, querying the database type and version on MySQL and Microsoft
- **Difficulty:** Practitioner
- **Vulnerability:** SQL Injection — Database Enumeration
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a SQL injection vulnerability in the product category filter.
The objective was to retrieve and display the database version using a `UNION` attack.
I first confirmed that the original query returned two text-compatible columns:

```text
' UNION SELECT 'abc','def'#
```

I then used the MySQL/Microsoft version variable:

```text
' UNION SELECT @@version,NULL#
```

The database version was returned in the application's response, confirming the underlying database technology and version.
This lab also showed that SQL comment syntax can depend on the database system. On MySQL, `#` can be used as a comment marker, while `--` must be followed by whitespace.

### PortSwigger — SQL injection attack, listing the database contents on non-Oracle databases
- **Difficulty:** Practitioner
- **Vulnerability:** SQL Injection — Database Enumeration
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a SQL injection vulnerability in the product category filter.

The objective was to discover the table containing user credentials, identify its columns, retrieve the usernames and passwords, and log in as the `administrator` user.
I first confirmed that the original query returned two text-compatible columns:

```text
' UNION SELECT 'abc','def'--
```

I then listed the database tables using:

```text
' UNION SELECT table_name,NULL FROM information_schema.tables--
```

After identifying the table containing user credentials, I listed its columns with:

```text
' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users_abcdef'--
```

Once the username and password columns were identified, I retrieved their contents:

```text
' UNION SELECT username_abcdef,password_abcdef FROM users_abcdef--
```

The response exposed the administrator credentials, which I used to successfully log in.
This lab demonstrated how SQL injection can be used to enumerate the database structure before retrieving sensitive data.

### PortSwigger — Blind SQL injection with conditional responses
- **Difficulty:** Practitioner
- **Vulnerability:** Blind SQL Injection
- **Result:** Solved
- **Tools used:** Burp Suite Repeater / Intruder

The application contained a blind SQL injection vulnerability in the `TrackingId` cookie.
The SQL query results were not returned directly, but the application displayed a `Welcome back` message when the injected condition was true.
I first confirmed the behavior using boolean conditions:

```text
TrackingId=xyz' AND '1'='1
TrackingId=xyz' AND '1'='2
```

I then confirmed the existence of the `users` table and the `administrator` account.
After determining the administrator password length, I extracted the password character by character using conditions based on `SUBSTRING()`:

```text
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
```

I used Burp Intruder to test possible characters and identified the correct value by looking for the `Welcome back` response.
I repeated the process for each password position, reconstructed the full administrator password, and successfully logged in.

### PortSwigger — Blind SQL injection with conditional errors

- **Difficulty:** Practitioner
- **Vulnerability:** Blind SQL Injection — Conditional Errors
- **Result:** Solved
- **Tools used:** Burp Suite Repeater / Intruder

The application contained a blind SQL injection vulnerability in the `TrackingId` cookie.

The query results were not visible and normal true/false conditions did not change the page, but SQL errors caused a different HTTP response.

I first confirmed the Oracle database behavior and built valid injected subqueries using the `dual` table.

I then used conditional errors with a `CASE` expression:

```text
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

A true condition triggered a database error and returned HTTP `500`, while a false condition returned a normal HTTP `200` response.

I used this behavior to confirm the `administrator` user, determine the password length, and extract the password character by character with:

```text
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

I used Burp Intruder to test possible characters and identified the correct value by looking for HTTP `500` responses.

After reconstructing the full password, I successfully logged in as the `administrator` user.

### PortSwigger — Visible error-based SQL injection

- **Difficulty:** Practitioner
- **Vulnerability:** Error-Based SQL Injection
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a SQL injection vulnerability in the `TrackingId` cookie.

The SQL query results were not returned directly, but verbose database errors exposed useful information about the query structure and later leaked data from the database.

I first confirmed that the cookie value was inserted inside a single-quoted SQL string by appending a quote and observing the resulting syntax error.

I then built a valid boolean condition using `CAST()`:

```text
TrackingId=' AND 1=CAST((SELECT 1) AS int)--
```

After confirming the syntax, I queried the `users` table. Because the original payload was too long, I removed the original cookie value to stay within the character limit.

I first retrieved one username using:

```text
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

The database attempted to convert the returned username to an integer and leaked it inside the error message, revealing that the first user was `administrator`.

I then retrieved the corresponding password with:

```text
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

The verbose conversion error exposed the administrator password directly.

I used the recovered credentials to successfully log in as the `administrator` user.

### PortSwigger — Blind SQL injection with time delays and information retrieval

- **Difficulty:** Practitioner
- **Vulnerability:** Blind SQL Injection — Time-Based
- **Result:** Solved
- **Tools used:** Burp Suite Repeater / Python script

The application contained a blind SQL injection vulnerability in the `TrackingId` cookie.

The query results were not visible, normal true/false conditions did not change the page, and database errors were handled silently. However, conditional time delays could still be used as a side channel.

I first confirmed the technique manually with Burp Suite by triggering a PostgreSQL delay only when a condition was true:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(2)+ELSE+pg_sleep(0)+END--
```

A true condition caused a delayed response, while a false condition returned immediately.

I had already used the same general blind SQL injection approach in previous labs: determine the password length, then recover the password one character at a time by observing a true/false signal.

The official approach uses Burp Intruder to test every possible character for each password position. Since I had already understood and practiced this mechanism, repeating hundreds of sequential time-based requests would have been unnecessarily slow.

I therefore used AI assistance to help me create a Python script that automated the same technique more efficiently.

Instead of testing every possible character individually, the script used binary search with comparisons based on the ASCII value of each character:

```sql
ASCII(SUBSTRING(password, position, 1)) > midpoint
```

If the condition was true, the database introduced a delay. The script then discarded half of the remaining possible values and repeated the process until the character was identified.

This reduced the number of tests from up to 36 requests per character to approximately 6 requests per character.

The script reconstructed the administrator password position by position, and I used the recovered credentials to successfully log in as the `administrator` user.

## References
- [PortSwigger Web Security Academy — SQL injection](https://portswigger.net/web-security/sql-injection)
- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [OWASP Cheat Sheet Series — SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [OWASP Web Security Testing Guide — Testing for SQL Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_SQL_Injection)
- [MITRE — CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html)
