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
