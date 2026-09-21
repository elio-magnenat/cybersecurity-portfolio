# API Reconnaissance

## API Recon

API reconnaissance is the process of gathering as much information as possible about an API before testing it for vulnerabilities.

The first step is to identify the API endpoints exposed by the application.

An API endpoint is a location where the API receives a request related to a specific resource.

For example:

```http
GET /api/books HTTP/1.1
Host: example.com
```

In this request, the endpoint is:

```text
/api/books
```

It may return a list of books.

A more specific endpoint could be:

```text
/api/books/mystery
```

which may return books belonging to the `mystery` category.

Identifying endpoints helps reveal the API's available functionality and therefore its potential attack surface.

### Understanding how to interact with an endpoint

Finding an endpoint is only the first step. To test it effectively, I also need to understand how the API expects requests to be constructed.

For each endpoint, useful information includes:

- Required parameters
- Optional parameters
- Supported HTTP methods
- Accepted request and response formats
- Authentication mechanisms
- Rate limits

Understanding these elements makes it possible to construct valid requests, modify them, and later test how the API behaves when unexpected or malicious input is supplied.

## API Documentation

API documentation explains how an API is designed to be used.

It can be useful during reconnaissance because it may reveal endpoints, parameters, supported HTTP methods, expected data formats, authentication requirements, and other information about the API's attack surface.

API documentation can be available in two main forms:

- **Human-readable documentation:** intended for developers and usually includes explanations, examples, and usage scenarios.
- **Machine-readable documentation:** written in structured formats such as JSON or XML and designed to be processed automatically by software.

When an API is intended for external developers, its documentation may be publicly available.

If documentation is available, it should be reviewed early during API reconnaissance because it can provide a direct overview of the API's exposed functionality.

### Discovering API documentation

API documentation is not always directly linked or publicly advertised.

However, it may still be accessible through applications that use the API.

Possible documentation endpoints include:

```text
/api
/swagger/index.html
/openapi.json
```

Swagger and OpenAPI documentation can be particularly useful because they may expose detailed information about available endpoints and request structures.

If I discover an API resource endpoint, I should also investigate its parent paths.

For example, if I find:

```text
/api/swagger/v1/users/123
```

I should also check:

```text
/api/swagger/v1
/api/swagger
/api
```

One of these parent paths may expose documentation or additional API functionality.

API documentation can also be discovered by:

- Browsing the application manually through Burp's browser.
- Crawling the application or API.
- Inspecting requests and responses for references to documentation.
- Testing common API documentation paths.
- Using Burp Intruder with a wordlist of common documentation endpoints.

Finding documentation can significantly simplify further API testing because it may reveal parts of the API that are not directly exposed through the application's normal user interface.
