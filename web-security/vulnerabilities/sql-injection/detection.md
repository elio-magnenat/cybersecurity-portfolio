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
