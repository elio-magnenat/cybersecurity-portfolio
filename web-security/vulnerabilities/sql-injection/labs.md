# SQL Injection Labs

## PortSwigger — SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
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

## PortSwigger — SQL injection vulnerability allowing login bypass
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

## PortSwigger — SQL injection UNION attack, determining the number of columns returned by the query
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

## PortSwigger — SQL injection UNION attack, finding a column containing text
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

## PortSwigger — SQL injection UNION attack, retrieving data from other tables

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

## PortSwigger — SQL injection UNION attack, retrieving multiple values in a single column
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

## PortSwigger — SQL injection attack, querying the database type and version on MySQL and Microsoft
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

## PortSwigger — SQL injection attack, listing the database contents on non-Oracle databases
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

## PortSwigger — Blind SQL injection with conditional responses
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

## PortSwigger — Blind SQL injection with conditional errors

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

## PortSwigger — Visible error-based SQL injection

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

## PortSwigger — Blind SQL injection with time delays and information retrieval

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

## PortSwigger — SQL injection with filter bypass via XML encoding

- **Difficulty:** Practitioner
- **Vulnerability:** SQL Injection — WAF Bypass via XML Encoding
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a SQL injection vulnerability in the stock check feature.

The `productId` and `storeId` values were sent to the server inside an XML request. I first confirmed that the `storeId` value was evaluated by the back-end and then attempted a `UNION SELECT` attack.

A direct SQL injection payload was blocked by the application's web application firewall (WAF), so I encoded the payload using XML hexadecimal character entities.

For example:

```xml
<storeId>2 &#x55;&#x4E;&#x49;&#x4F;&#x4E; &#x53;&#x45;&#x4C;&#x45;&#x43;&#x54; ...</storeId>
```

The WAF inspected the encoded XML representation, but the XML parser decoded the entities before the value reached the SQL interpreter.

This allowed the SQL payload to execute while bypassing the filter.

After confirming that the original query returned only one column, I used a `UNION SELECT` payload to retrieve data from the `users` table.

Because only one column was available, usernames and passwords had to be returned through a single output value.

The attack successfully exposed the administrator credentials, which I used to log in and solve the lab.

This lab demonstrated that SQL injection can appear in structured input formats such as XML, and that decoding performed after security filtering can sometimes allow encoded payloads to bypass weak WAF rules.
