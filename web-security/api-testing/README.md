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


## Discovering Hidden Endpoints

Once some API endpoints have been identified, their structure can be used to search for additional functionality that is not directly exposed by the application.

For example, suppose the following endpoint is known:

```http
PUT /api/user/update
```

The final part of the path may represent an action. Other likely actions can then be tested in the same position, such as:

```text
/api/user/delete
/api/user/add
/api/user/create
/api/user/reset
```

This technique consists of replacing part of a known path with candidate values and observing how the server responds.

### Choosing useful candidate values

Generic API wordlists can help discover common endpoint names, but application-specific terms are often even more useful.

Good candidates may come from:

- Existing endpoint names
- Page names and visible features
- JavaScript files
- Parameter names
- Business terminology used by the application
- Common API actions such as `add`, `delete`, `update`, `create`, or `reset`

For example, an application related to orders might use endpoints such as:

```text
/api/orders/create
/api/orders/cancel
/api/orders/refund
```

while an account management API might expose:

```text
/api/user/reset-password
/api/user/disable
/api/user/permissions
```

### What to look for

Responses should be compared carefully when testing candidate paths.

Useful indicators include:

- Different HTTP status codes
- Different response lengths
- Redirects
- Error messages
- Authentication errors
- Method-related errors

For example, a response such as:

```http
HTTP/2 405 Method Not Allowed
Allow: POST
```

may indicate that the tested endpoint exists, but the current HTTP method is not accepted.

This can reveal hidden functionality even when the request itself fails.

The goal is therefore not only to find requests that return `200 OK`, but to identify any response that behaves differently from requests to non-existent endpoints.


## Finding Hidden Parameters

APIs may support parameters that are not documented or exposed through the normal application interface.

These hidden parameters can sometimes influence application behavior and may reveal functionality that was not intended to be directly accessible.

For example, an API request may normally contain:

```json
{
  "username": "wiener"
}
```

but the endpoint may also accept additional parameters such as:

```json
{
  "username": "wiener",
  "role": "admin"
}
```

even if `role` is not documented.

### Discovering hidden parameters

Hidden parameters can be identified by testing likely parameter names and observing how the API responds.

Useful candidate names may come from:

- Existing parameters
- JavaScript files
- API responses
- Error messages
- Business terminology used by the application
- Common field names such as `role`, `admin`, `status`, `discount`, or `permissions`

The goal is to determine whether adding or modifying an undocumented parameter changes the server's behavior.

Interesting indicators include:

- Different response bodies
- Different HTTP status codes
- New fields appearing in responses
- Unexpected changes to application state
- Validation errors that reveal expected parameter names

Automated wordlists can help test large numbers of possible parameter names, but application-specific names discovered during reconnaissance are often more valuable.

## Mass Assignment Vulnerabilities

Mass assignment, also known as auto-binding, occurs when an application automatically maps request parameters to fields of an internal object.

This can become a security issue when the application accepts fields that the user was never intended to control.

For example, an internal user object might contain:

```text
username
email
role
isAdmin
```

The application may only intend to let the user update their email:

```json
{
  "email": "new@example.com"
}
```

However, if the framework automatically binds every supplied field to the internal object, an attacker may try:

```json
{
  "email": "new@example.com",
  "isAdmin": true
}
```

If the server accepts the additional field, the attacker may be able to modify sensitive internal properties.

### Why mass assignment happens

Modern frameworks often provide automatic object binding to simplify development.

Instead of manually assigning each permitted field, the application may accept an entire request object and map matching parameter names directly to properties of an internal model.

Conceptually:

```text
Request data
    ↓
Automatic binding
    ↓
Internal application object
```

This becomes dangerous when sensitive properties are included in the internal object but are not explicitly blocked from user input.

### Security impact

Depending on the exposed fields, mass assignment may allow an attacker to:

- Change account privileges
- Modify roles or permissions
- Alter account status
- Change ownership information
- Modify internal flags
- Manipulate values such as discounts or balances

The impact depends on which internal properties can be controlled.

The key testing idea is to look for fields that appear in API responses or internal object structures but are not normally present in client requests, then check whether the server accepts them when they are added manually.


### Identifying Hidden Parameters from API Responses

Mass assignment can often be investigated by comparing the fields accepted by an update request with the fields returned by the API for the same object.

For example, an update request may only contain:

```json
{
  "username": "wiener",
  "email": "wiener@example.com"
}
```

while a request retrieving the same user may return:

```json
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "isAdmin": false
}
```

The additional fields in the response may indicate properties that exist on the internal object even though they are not normally exposed as editable parameters.

Interesting fields may include:

- `id`
- `role`
- `isAdmin`
- `status`
- `permissions`
- Ownership-related fields
- Internal flags

These fields are not automatically vulnerable, but they are good candidates for further testing.

A useful approach is therefore:

```text
Compare update request fields
        ↓
Compare returned object fields
        ↓
Identify additional properties
        ↓
Test whether they are accepted in update requests
```

### Testing Mass Assignment

Once a potentially sensitive hidden field has been identified, it can be added manually to an update request.

For example:

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": false
}
```

If the request is accepted, the next step is to determine whether the server is actually processing the additional field.

One useful technique is to send an invalid value:

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": "foo"
}
```

If the response changes or validation fails specifically because of `isAdmin`, this can indicate that the server recognizes and processes the parameter.

The field can then be tested with a valid but security-sensitive value:

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": true
}
```

If the application binds this value directly to the internal user object without restricting which properties may be modified, the user's privileges may change.

The final step is to verify the effect in the application itself, for example by checking whether previously unavailable administrative functionality becomes accessible.

The important distinction is:

```text
Hidden field discovered
≠
Vulnerability confirmed

Hidden field accepted and applied to a sensitive internal property
=
Mass assignment vulnerability
```


## Preventing API Vulnerabilities

API security should be considered from the design stage rather than added after the API has already been deployed.

A secure API should expose only the functionality and information that clients actually need.

### Protect API documentation

API documentation can reveal a large part of the application's attack surface, including endpoints, parameters, supported methods, and request formats.

Documentation that is not intended to be public should therefore require appropriate access controls.

Documentation should also remain accurate and up to date so that developers and authorized security testers have a reliable view of the API.

### Restrict HTTP methods

Each endpoint should accept only the HTTP methods that are actually required.

For example, an endpoint intended only to retrieve information should not unexpectedly support methods such as:

```text
POST
PATCH
DELETE
```

Using an explicit allowlist of permitted methods reduces the risk of exposing unused or unintended functionality.

### Validate content types

Endpoints should verify that requests use the expected content type.

For example, if an endpoint is designed to accept JSON:

```http
Content-Type: application/json
```

it should not automatically accept XML or other formats unless they are intentionally supported and protected.

Different parsers and processing paths can introduce different security risks.

### Avoid overly informative errors

API errors should provide enough information for legitimate clients to understand that a request failed without unnecessarily revealing internal implementation details.

Responses should avoid exposing information such as:

- Internal object structures
- Sensitive parameter names
- Framework or database details
- Internal paths
- Debug information

Detailed diagnostic information should be kept in server-side logs rather than exposed directly to clients.

### Protect every API version

Security controls should be applied consistently across all active API versions.

For example:

```text
/api/v1/
/api/v2/
/api/v3/
```

Older versions should not remain accessible with weaker authentication, validation, or authorization controls simply because a newer version exists.

Unused API versions should be removed or disabled when they are no longer required.

### Prevent mass assignment

Applications should explicitly control which object properties users are allowed to modify.

For example, if users should only be able to update:

```text
username
email
```

the application should allow only those fields rather than automatically binding every supplied parameter to the internal user object.

Sensitive properties such as:

```text
isAdmin
role
permissions
accountStatus
```

should not be writable through normal user-controlled requests.

The safest approach is to maintain an explicit allowlist of properties that may be updated and prevent sensitive internal fields from being modified through client input.


## Server-Side Parameter Pollution

Server-side parameter pollution occurs when an application includes user-controlled input inside a request to another server-side API without safely encoding or validating it.

The attacker does not necessarily communicate with the internal API directly.

Instead, the flow may look like this:

```text
User
  ↓
Public application
  ↓
Internal API
```

The vulnerability appears when user input is inserted into the internal request in a way that allows the user to modify its structure.

For example, suppose the application receives:

```http
GET /account?username=wiener
```

and internally constructs a request such as:

```http
GET /internal/users?username=wiener&details=basic
```

If the value of `username` is inserted without adequate encoding, specially crafted input may introduce additional parameters into the internal request.

Conceptually:

```text
Expected input:
username=wiener

Internal request:
username=wiener&details=basic
```

An attacker may try to inject another parameter:

```text
username=wiener&details=full
```

Depending on how the internal API parses duplicated or additional parameters, this may modify how the request is processed.

### Possible impact

Server-side parameter pollution may allow an attacker to:

- Add unexpected parameters
- Override existing parameters
- Change internal API behavior
- Access data that should not be available
- Reach functionality that is not normally exposed

The exact impact depends on how the internal API handles the manipulated request.

### Possible input locations

Any user-controlled value that is reused inside a server-side request may potentially be interesting to test.

Examples include:

- Query parameters
- Form fields
- HTTP headers
- URL path parameters

The important point is not where the input originally appears, but whether the server later inserts it into another request without correctly separating user data from the structure of that request.

### Key concept

The vulnerability can be summarized as:

```text
User-controlled input
        ↓
Inserted into internal request
        ↓
Input changes request structure
        ↓
Internal API receives unintended parameters
```

Server-side parameter pollution is different from mass assignment.

Mass assignment involves supplying additional object properties that the application automatically binds to an internal object.

Server-side parameter pollution instead involves manipulating the structure or parameters of a request that the application sends to another server-side component.


### Testing Server-Side Parameter Pollution in Query Strings

When user-controlled input is reused to build a request to an internal API, query-string syntax can sometimes be injected into that internal request.

Useful characters to test include:

```text
#
&
=
```

Because these characters have a structural meaning inside URLs, they can help determine whether user input is being safely encoded before it reaches the internal API.

Consider a public request such as:

```http
GET /userSearch?name=peter&back=/home
```

The application may internally generate:

```http
GET /users/search?name=peter&publicProfile=true
```

The objective is to determine whether manipulating the public `name` parameter can change the structure of this internal request.

#### Truncating the Internal Query String

The `#` character normally introduces a URL fragment.

If it reaches the internal request unencoded, it may cause everything after it to be treated as a fragment rather than part of the query string.

Because a raw `#` would normally be handled by the browser and not sent to the server, it must first be URL-encoded:

```text
%23
```

For example:

```http
GET /userSearch?name=peter%23foo&back=/home
```

may cause the application to construct:

```http
GET /users/search?name=peter#foo&publicProfile=true
```

If the internal HTTP client interprets the `#` as a fragment delimiter, the effective request may become:

```http
GET /users/search?name=peter
```

This would remove:

```text
publicProfile=true
```

from the query sent to the internal API.

Differences in the response can help determine whether truncation occurred.

For example:

```text
Normal request
name=peter
        ↓
name=peter&publicProfile=true
```

versus:

```text
Injected input
name=peter%23foo
        ↓
name=peter#foo&publicProfile=true
        ↓
query potentially truncated after peter
```

If truncation removes a security-related parameter, it may expose information or functionality that should normally remain restricted.

#### Injecting Additional Parameters

The `&` character separates parameters in a query string.

If an encoded ampersand is decoded before the internal request is sent, it may be possible to introduce another parameter.

The encoded value is:

```text
%26
```

For example:

```http
GET /userSearch?name=peter%26foo=xyz&back=/home
```

may result in the internal request:

```http
GET /users/search?name=peter&foo=xyz&publicProfile=true
```

The additional parameter does not need to be valid during initial testing.

A deliberately unknown parameter such as:

```text
foo=xyz
```

can help determine whether the structure of the internal query has been modified.

An unchanged response does not necessarily mean the injection failed. The internal API may simply ignore unknown parameters.

#### Injecting Valid Parameters

Once parameter injection appears possible, known or suspected API parameters can be tested.

For example, if an `email` parameter has been discovered elsewhere:

```http
GET /userSearch?name=peter%26email=foo&back=/home
```

may become:

```http
GET /users/search?name=peter&email=foo&publicProfile=true
```

The response can then be compared with the original request to determine whether the injected parameter affects the internal API.

Parameters discovered during API reconnaissance, error analysis, or hidden-parameter testing are particularly useful candidates.

#### Overriding Existing Parameters

A stronger test is to inject another parameter with the same name as one that already exists.

For example:

```http
GET /userSearch?name=peter%26name=carlos&back=/home
```

may produce:

```http
GET /users/search?name=peter&name=carlos&publicProfile=true
```

The internal API now receives two `name` parameters.

How duplicate parameters are interpreted depends on the server-side technology.

Possible behaviors include:

```text
First value wins
name=peter

Last value wins
name=carlos

Values are combined
name=peter,carlos
```

For example, the behavior described in the training material differs between technologies:

- PHP commonly uses the last value.
- ASP.NET may combine duplicate values.
- Node.js / Express may use the first value.

Because duplicate-parameter handling varies, the response must always be observed rather than assuming how the backend will behave.

If the injected value overrides the original one, sensitive parameters may potentially be manipulated.

For example:

```text
name=peter
```

could potentially become:

```text
name=administrator
```

if the internal API accepts the injected duplicate parameter.

### Testing Strategy

A practical testing sequence is:

```text
Known user-controlled parameter
        ↓
Test encoded query syntax
        ↓
Try truncation with %23
        ↓
Try parameter injection with %26
        ↓
Test a known valid parameter
        ↓
Inject a duplicate parameter
        ↓
Compare the application's responses
```

The important goal is to determine whether user-controlled input can modify the structure of the server-side request, rather than simply changing the value of the original parameter.


### Testing Server-Side Parameter Pollution in REST Paths

REST APIs often place resource identifiers directly inside the URL path rather than in the query string.

For example:

```text
/api/users/123
```

can be interpreted as:

```text
/api
/users
/123
```

where:

- `/api` is the API root
- `/users` is the resource
- `/123` identifies a specific user

A vulnerability may appear when user-controlled input is inserted directly into an internal API path without adequate validation or encoding.

For example, a public request may look like:

```http
GET /edit_profile.php?name=peter
```

The application may then construct an internal request such as:

```http
GET /api/private/users/peter
```

If the `name` value is inserted directly into the internal path, an attacker may attempt to modify the path structure.

One possible test is to inject URL-encoded path traversal sequences.

For example:

```http
GET /edit_profile.php?name=peter%2f..%2fadmin
```

may cause the application to construct:

```http
GET /api/private/users/peter/../admin
```

If the internal HTTP client or API normalizes the path, this may become:

```http
GET /api/private/users/admin
```

This means the attacker may be able to access a different internal resource than the application originally intended.

### Why this works

The issue appears when user input is treated as part of the URL path structure instead of being safely handled as a simple value.

Conceptually:

```text
Expected input:
peter
        ↓
/api/private/users/peter
```

but with crafted input:

```text
peter/../admin
        ↓
/api/private/users/peter/../admin
        ↓
path normalization
        ↓
/api/private/users/admin
```

The important question is whether the server-side client or internal API normalizes the injected path.

### What to look for

When testing this behavior, compare the application's response for clues such as:

- Different user data being returned
- Different status codes
- Access to another internal resource
- Authorization errors
- Changes in response length or content

The objective is to determine whether user-controlled input can modify the path of a server-side request and cause the internal API to access a different resource.


### Testing Server-Side Parameter Pollution in Structured Data

Server-side parameter pollution can also affect structured data formats such as JSON or XML.

The vulnerability appears when user-controlled input is inserted into structured data that the application later sends to an internal API.

For example, a public request may contain:

```http
POST /myaccount

name=peter
```

The application may then construct an internal API request such as:

```http
PATCH /users/7312/update
Content-Type: application/json

{
  "name": "peter"
}
```

If the `name` value is inserted directly into the JSON structure without adequate validation or encoding, an attacker may attempt to inject additional JSON properties.

For example:

```text
peter","access_level":"administrator
```

could potentially transform the internal request into:

```json
{
  "name": "peter",
  "access_level": "administrator"
}
```

The attacker has now changed the structure of the internal JSON object instead of only controlling the value of `name`.

### Why this is dangerous

The application may intend to expose only a limited field such as:

```text
name
```

while the internal API may support additional sensitive properties such as:

```text
access_level
role
permissions
isAdmin
```

If an attacker can inject one of these properties into the server-side request, they may be able to modify data or privileges that the public application was never intended to expose.

Conceptually:

```text
Expected input:
peter
        ↓
{"name":"peter"}
```

but with crafted input:

```text
peter","access_level":"administrator
        ↓
{"name":"peter","access_level":"administrator"}
```

### What to look for

Useful indicators include:

- Different validation errors
- Changes in the returned JSON
- Additional properties being accepted
- Changes to the user's account or permissions
- Different behavior after injecting structured syntax

The important goal is to determine whether user-controlled data is treated purely as a value or whether it can escape its intended position and modify the structure of the server-side request.

### Structured Data Already Supplied as JSON

The same issue can occur even when the client already sends JSON.

For example, the browser may send:

```http
POST /myaccount
Content-Type: application/json

{
  "name": "peter"
}
```

The application may decode this JSON, extract the `name` value, and then build another JSON request for an internal API:

```http
PATCH /users/7312/update
Content-Type: application/json

{
  "name": "peter"
}
```

If the extracted value is inserted into the new JSON structure without being encoded safely, structured data can still be injected.

For example, a crafted client request may contain:

```json
{
  "name": "peter\",\"access_level\":\"administrator"
}
```

After decoding and unsafe reconstruction, the internal API request could become:

```json
{
  "name": "peter",
  "access_level": "administrator"
}
```

This shows that using JSON on the client side does not automatically prevent structured data injection.

The important question is how the application handles the value after decoding it and before embedding it into another structured request.

### Structured Data Injection in Responses

Structured format injection can also affect responses.

For example, user-controlled data may be stored safely in a database but later inserted into a JSON response from a back-end API without adequate encoding.

Conceptually:

```text
User-controlled value
        ↓
Stored safely
        ↓
Read from database
        ↓
Inserted unsafely into JSON response
        ↓
Response structure modified
```

This means that data should not be considered safe simply because it was previously stored in the application's database.

The same principle applies to other structured formats such as XML.

The key idea is that whenever user-controlled data is embedded into structured data, it must remain data and must not be allowed to alter the surrounding structure.


### Testing Server-Side Parameter Pollution with Automated Tools

Automated security testing tools can help identify behavior that may indicate server-side parameter pollution.

A useful signal is an unexpected transformation of user-controlled input.

For example:

```text
User input
    ↓
Application modifies or decodes the value
    ↓
Modified value is reused in another server-side request
```

This behavior does not automatically mean that a vulnerability exists.

An application may legitimately transform input before processing it.

However, transformations involving characters such as:

```text
&
#
/
..
"
\
```

can be worth investigating when the value is later inserted into:

- Query strings
- URL paths
- JSON
- XML
- Other structured server-side requests

### Automated Detection

Automated scanners may identify inputs whose behavior differs from normal values.

They may flag cases where:

- Encoded characters are decoded
- Input structure changes
- Additional parameters appear to be created
- Path normalization occurs
- Structured data is modified
- Error messages change unexpectedly

These findings should be treated as indicators rather than confirmed vulnerabilities.

### Manual Verification

Suspicious results should be verified manually.

A useful workflow is:

```text
Automated tool detects unusual behavior
        ↓
Identify the affected input
        ↓
Reproduce the request manually
        ↓
Change one element at a time
        ↓
Compare responses
        ↓
Determine whether the internal request can actually be modified
```

The objective is to distinguish normal input processing from a real server-side injection vulnerability.

Automation is therefore useful for discovering potential entry points, while manual testing is required to understand and confirm their security impact.
