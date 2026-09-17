# API Testing Labs

## API documentation

### PortSwigger — Exploiting an API endpoint using documentation

- **Difficulty:** Apprentice
- **Vulnerability:** Exposed API Documentation / Unprotected API Endpoint
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application exposed API documentation that revealed available endpoints and the HTTP methods they supported.

After logging in with the provided account, I modified the account's email address and inspected the resulting request in Burp Proxy.

The application sent a request to:

```http
PATCH /api/user/wiener
```

I sent this request to Burp Repeater and progressively removed parts of the path to investigate the API's base endpoints.

Removing the username produced:

```text
/api/user
```

and removing `/user` led to:

```text
/api
```

Requesting `/api` exposed interactive API documentation.

The documentation revealed a `DELETE` endpoint that accepted a username as a parameter. I used this documented functionality with the `carlos` username to delete the target user and solve the lab.

This lab demonstrated why API documentation is valuable during reconnaissance. Even when documentation is not directly linked by the application's interface, investigating parent paths of known API endpoints can reveal it.

It also showed that exposed documentation can disclose sensitive functionality that may not otherwise be visible through the normal application interface.
