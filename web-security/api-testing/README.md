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


## Interacting with API Endpoints

Once an API endpoint has been identified, the next step is to interact with it and observe how it behaves when requests are modified.

The goal is not only to reproduce the requests made by the application, but also to understand which variations the API accepts.

Useful things to test include:

- Changing the HTTP method
- Modifying parameters
- Adding or removing parameters
- Changing the request body
- Changing the content type
- Altering headers
- Testing unexpected values

For example, an endpoint normally accessed with:

```http
GET /api/users/123
```

may behave differently when another method is used:

```http
POST /api/users/123
```

or:

```http
DELETE /api/users/123
```

Even when the application interface only uses one method, the back-end endpoint may support additional operations.

### Analyzing API responses

API responses can reveal useful information about how an endpoint expects requests to be structured.

Important things to examine include:

- HTTP status codes
- Response bodies
- Error messages
- Validation messages
- Required parameter names
- Expected data types
- Supported formats

For example, an error such as:

```json
{
  "error": "Missing parameter: username"
}
```

reveals that the endpoint expects a parameter named `username`.

Similarly, an error indicating that a specific content type is required can reveal the format expected by the API.

This means that error responses are not only signs of failed requests. They can also provide clues that help reconstruct valid requests and discover additional API functionality.


## Identifying Supported Content Types

API endpoints often expect request data to be sent in a specific format.

The format is usually indicated by the `Content-Type` header.

Common examples include:

```http
Content-Type: application/json
```

```http
Content-Type: application/xml
```

```http
Content-Type: application/x-www-form-urlencoded
```

An API may process the same logical data differently depending on the selected content type.

For example, an endpoint may normally receive JSON:

```json
{
  "username": "wiener"
}
```

but may also accept equivalent XML data:

```xml
<username>wiener</username>
```

### Why testing different content types matters

Changing the content type can expose different processing paths inside the application.

This may help to:

- Reveal useful error messages
- Discover additional supported formats
- Bypass validation or filtering logic
- Reach vulnerable parsers
- Expose differences in how the server handles the same input

For example, an endpoint may correctly validate JSON input but process XML input using a different parser with weaker protections.

This means that an endpoint should not be considered secure simply because one supported format behaves safely.

### Testing content types

When testing another content type, both the request header and the request body must be changed consistently.

For example:

```http
Content-Type: application/json
```

with:

```json
{
  "username": "wiener"
}
```

could be changed to:

```http
Content-Type: application/xml
```

with:

```xml
<username>wiener</username>
```

If the server accepts both requests, this indicates that the endpoint supports multiple input formats.

The responses should then be compared carefully for differences in status codes, error messages, validation behavior, and processing logic.

Testing multiple supported content types can reveal attack surface that would remain hidden if only the format used by the normal application interface were tested.
