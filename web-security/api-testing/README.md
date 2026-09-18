# API Testing

API testing consists of examining an application's APIs to understand their attack surface and identify potential security weaknesses.

An application may expose API functionality that is not fully visible or used by the website's front-end. Testing these APIs therefore requires understanding which endpoints exist, what data they accept, and how they can be interacted with.

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


## Identifying API Endpoints

API endpoints are not always fully documented or directly visible through the application's interface.

Even when documentation exists, it is useful to inspect how the application actually communicates with its back end. Documentation may be incomplete, outdated, or may not include every internal endpoint used by the application.

A useful starting point is to browse the application normally while observing the HTTP requests it generates.

Interesting indicators include URL patterns such as:

```text
/api/
/v1/
/v2/
/users/
/account/
```

These paths can reveal how the API is structured and may expose resources that can be tested individually.

### Looking beyond visible requests

Not every API endpoint is triggered during normal browsing.

Client-side JavaScript files can contain references to endpoints that are called only under specific conditions or that are not directly accessible through the visible user interface.

Useful things to look for inside JavaScript include:

```text
/api/users
/api/admin
/api/profile
/api/orders
```

Other interesting information may include:

- HTTP methods used by the application
- Parameter names
- Internal API paths
- Versioned endpoints
- Administrative functionality
- Hidden or rarely used features

This means that API reconnaissance should not rely only on the requests generated by clicking through the application.

Reviewing the application's client-side code and request patterns can reveal additional attack surface that would otherwise remain hidden.
