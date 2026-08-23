# Database Enumeration

SQL injection attacks often require information about the underlying database before useful data can be retrieved.
Important information includes:
- The database type and version.
- The tables present in the database.
- The columns contained in those tables.

This information can help adapt SQL injection payloads to the database system and identify interesting data to retrieve.

## Querying the database type and version

Different database systems expose their version information using different SQL queries.

Common examples include:

| Database | Query |
| --- | --- |
| Microsoft SQL Server | `SELECT @@version` |
| MySQL | `SELECT @@version` |
| Oracle | `SELECT * FROM v$version` |
| PostgreSQL | `SELECT version()` |

When a SQL injection point supports `UNION` queries, these expressions can sometimes be injected to identify the database system and its version.

For example:

```text
' UNION SELECT @@version--
```

If the injected query succeeds and the result is displayed in the response, the returned version string can reveal both the database type and the exact version in use.

Knowing the database type is useful because SQL syntax, system tables, functions, and comment styles can differ between database systems.
