# Server-Side Parameter Pollution

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

## Possible impact

Server-side parameter pollution may allow an attacker to:

- Add unexpected parameters
- Override existing parameters
- Change internal API behavior
- Access data that should not be available
- Reach functionality that is not normally exposed

The exact impact depends on how the internal API handles the manipulated request.

## Possible input locations

Any user-controlled value that is reused inside a server-side request may potentially be interesting to test.

Examples include:

- Query parameters
- Form fields
- HTTP headers
- URL path parameters

The important point is not where the input originally appears, but whether the server later inserts it into another request without correctly separating user data from the structure of that request.

## Key concept

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


## Testing Server-Side Parameter Pollution in Query Strings

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

### Truncating the Internal Query String

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

### Injecting Additional Parameters

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

### Injecting Valid Parameters

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

### Overriding Existing Parameters

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

## Testing Strategy

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


## Testing Server-Side Parameter Pollution in REST Paths

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

## Why this works

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

## What to look for

When testing this behavior, compare the application's response for clues such as:

- Different user data being returned
- Different status codes
- Access to another internal resource
- Authorization errors
- Changes in response length or content

The objective is to determine whether user-controlled input can modify the path of a server-side request and cause the internal API to access a different resource.


## Testing Server-Side Parameter Pollution in Structured Data

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

## Why this is dangerous

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

## What to look for

Useful indicators include:

- Different validation errors
- Changes in the returned JSON
- Additional properties being accepted
- Changes to the user's account or permissions
- Different behavior after injecting structured syntax

The important goal is to determine whether user-controlled data is treated purely as a value or whether it can escape its intended position and modify the structure of the server-side request.

## Structured Data Already Supplied as JSON

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

## Structured Data Injection in Responses

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


## Testing Server-Side Parameter Pollution with Automated Tools

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

## Automated Detection

Automated scanners may identify inputs whose behavior differs from normal values.

They may flag cases where:

- Encoded characters are decoded
- Input structure changes
- Additional parameters appear to be created
- Path normalization occurs
- Structured data is modified
- Error messages change unexpectedly

These findings should be treated as indicators rather than confirmed vulnerabilities.

## Manual Verification

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


## Preventing Server-Side Parameter Pollution

Server-side parameter pollution can be reduced by strictly controlling how user input is included in server-side requests.

A key defense is to use an allowlist that defines which characters or values are permitted.

Any user-controlled data that does not match the expected format should either be rejected or safely encoded before being inserted into another request.

## Main Defenses

Applications should:

- Define the expected format of every input
- Allow only valid characters or values
- Encode user-controlled data before inserting it into query strings, paths, JSON, XML, or other structured formats
- Avoid building server-side requests through unsafe string concatenation
- Validate that the final request structure matches what the application expects

For example, if an input is supposed to contain only a username, values containing unexpected structural characters such as:

```text
&
#
/
"
\
```

should not be allowed to alter the structure of the internal request.

The general principle is:

```text
User input
        ↓
Validation
        ↓
Safe encoding
        ↓
Server-side request
```

User-controlled input should remain data and should never be able to become part of the request structure itself.
