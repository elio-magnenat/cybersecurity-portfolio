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

## Finding columns with a useful data type
After determining the number of columns returned by the original query, it is necessary to find which columns can contain the type of data we want to retrieve.
Interesting information such as usernames, passwords, or other text values is usually stored as strings.
To test which columns can contain string data, a string value can be placed in each column one at a time.
For example, if the query returns four columns:

```text
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

If placing `'a'` in a column causes an error, that column is probably not compatible with string data.
If the request succeeds and the injected value appears in the response, that column can be used to retrieve text data.

## Retrieving data from other tables
Once the number of columns and their compatible data types are known, a `UNION` attack can be used to retrieve data from another table.
For example, if the original query returns two text-compatible columns and the database contains a `users` table with `username` and `password` columns:

```text
' UNION SELECT username, password FROM users--
```

The results from the `users` table can then be appended to the original query results and displayed by the application.
To do this, the attacker needs to know the names of the relevant tables and columns.
Modern databases expose metadata about their own structure, which can sometimes be queried through SQL injection to discover table and column names.

## Retrieving multiple values in a single column
Sometimes a `UNION` attack only provides one column that can contain text.
In this case, multiple values can be combined into the same column using string concatenation.
For example, on Oracle:

```text
' UNION SELECT username || '~' || password FROM users--
```

The `||` operator concatenates strings, and the `~` character is used as a separator.
The result can look like:

```text
administrator~s3cure
wiener~peter
carlos~montoya
```

This makes it possible to retrieve multiple fields, such as usernames and passwords, through a single text-compatible column.
Concatenation syntax depends on the database system.
