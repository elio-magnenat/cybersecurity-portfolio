# Hidden Parameters and Mass Assignment

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
