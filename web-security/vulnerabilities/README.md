# Web Vulnerabilities

This directory groups my notes and practical exercises by vulnerability family.

Each topic focuses on how the vulnerability works, where it commonly appears, how it can be detected, its potential impact, and how it can be prevented. Practical PortSwigger lab write-ups are kept in a separate `labs.md` file when available.

## Topics

| Topic | Notes |
| --- | --- |
| [Authentication Vulnerabilities](authentication-vulnerabilities/README.md) | Password authentication, MFA, recovery mechanisms, testing, and prevention |
| [Broken Access Control](broken-access-control/README.md) | Vertical and horizontal privilege escalation, IDOR, and authorization failures |
| [File Upload Vulnerabilities](file-upload-vulnerabilities/README.md) | Unsafe uploads, validation weaknesses, and executable file handling |
| [OS Command Injection](os-command-injection/README.md) | User input reaching operating-system commands and shell execution |
| [Path Traversal](path-traversal/README.md) | Manipulating file paths to access unintended files and directories |
| [SQL Injection](sql-injection/README.md) | Query manipulation, UNION attacks, database enumeration, and blind SQL injection |
| [Server-Side Request Forgery (SSRF)](ssrf/README.md) | Server-side requests to local, internal, or unintended destinations |

## Structure

A typical topic folder follows this pattern:

```text
vulnerability-name/
├── README.md        # Overview, theory, impact, prevention, testing
├── labs.md          # Practical lab write-ups
└── topic-files.md   # Additional notes when the topic is large
```

Not every topic needs every file. Smaller topics remain intentionally compact.
