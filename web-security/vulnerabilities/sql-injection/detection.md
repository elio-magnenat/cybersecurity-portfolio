# Detecting SQL Injection
SQL injection can be detected manually by testing each user-controlled entry point and comparing the application's responses.
Common tests include:

- Submitting a single quote `'` and looking for errors or unusual behavior.
- Testing SQL expressions that should produce equivalent or different results.
- Comparing boolean conditions such as:

```text
OR 1=1
OR 1=2
```

- Using payloads that trigger a time delay and comparing response times.
- Using out-of-band techniques to detect interactions when the result is not directly visible.

The important idea is to compare responses systematically rather than relying on a single payload.

## SQL injection in different parts of a query
SQL injection is commonly found in the `WHERE` clause of a `SELECT` statement, but user-controlled input can appear in many other parts of a SQL query.
Possible locations include:

- `WHERE` conditions in `SELECT` or `UPDATE` queries.
- Values modified by an `UPDATE` statement.
- Values inserted by an `INSERT` statement.
- Table or column names in a `SELECT` query.
- The `ORDER BY` clause used to sort results.

The important point is that SQL injection can occur anywhere user-controlled input is used to construct SQL syntax, not only inside a `WHERE` clause.

## SQL injection in different input formats

SQL injection is not limited to URL parameters or HTML form fields.

Any user-controlled value that is later included in a SQL query may be vulnerable, including values supplied through formats such as JSON or XML.

Different input formats can also introduce additional encoding layers.

For example, XML character references can represent normal characters:

```xml
<storeId>999 &#x53;ELECT ...</storeId>
```

The XML parser decodes:

```text
&#x53;
```

into:

```text
S
```

so the application may eventually pass `SELECT` to the SQL interpreter.

This can sometimes bypass weak filters or WAF rules that inspect the raw request for SQL keywords before the input is decoded.

When testing SQL injection, it is therefore important to consider both the location of user-controlled input and any decoding or transformation that occurs before the value reaches the database.
