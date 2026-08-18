# SQL Injection UNION Attacks
A SQL injection UNION attack can be used when the results of a vulnerable SQL query are returned in the application's response.
The `UNION` keyword allows the results of an additional `SELECT` query to be appended to the results of the original query.
For example:

```sql
SELECT a, b FROM table1
UNION
SELECT c, d FROM table2
```

This combines the results of both queries into a single result set.
A `UNION` attack can therefore allow an attacker to retrieve information from other tables in the database.

## Requirements
For a `UNION` query to work:
- Both queries must return the same number of columns.
- Corresponding columns must use compatible data types.

For this reason, before retrieving data with a `UNION` attack, it is normally necessary to determine:
1. How many columns the original query returns.
2. Which returned columns can contain the type of data we want to retrieve, such as text.

## Determining the number of columns
Before performing a `UNION` attack, it is necessary to determine how many columns are returned by the original query.
There are two common methods.
### Using ORDER BY
The `ORDER BY` clause can reference columns by their position:

```text
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

The index is increased until the application produces an error or a different response.
For example, if `ORDER BY 2` works but `ORDER BY 3` causes an error, the original query probably returns two columns.
The application does not always display the database error directly. A generic error, missing results, or any consistent difference in the response can also reveal when the column index is invalid.
### Using UNION SELECT NULL
Another method is to try `UNION SELECT` queries with an increasing number of `NULL` values:

```text
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

A `UNION` query requires both queries to return the same number of columns.
When the number of `NULL` values matches the number of columns in the original query, the payload may succeed.
`NULL` is useful because it is compatible with most common SQL data types, which reduces the chance of a type mismatch.
The result must still be inferred from the application's response, for example through a successful response, additional content, or a different error.

## Database-specific syntax
SQL injection syntax can vary depending on the database system.
For example, Oracle requires every `SELECT` query to include a `FROM` clause. The built-in `DUAL` table can be used for this purpose:

```sql
' UNION SELECT NULL FROM DUAL--
```

Comment syntax can also differ between databases.
For example:
- `--` is commonly used to start a SQL comment.
- On MySQL, `--` must be followed by a space.
- MySQL also supports `#` as a comment character.

Because of these differences, payloads may need to be adapted depending on the database in use.
See the [PortSwigger SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet) for database-specific syntax.
