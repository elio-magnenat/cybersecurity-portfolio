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


### PortSwigger — Finding and exploiting an unused API endpoint

- **Difficulty:** Practitioner
- **Vulnerability:** Unused API Endpoint / Improper Access Control
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application exposed an API endpoint used to retrieve product prices.

While inspecting product requests, I identified an endpoint similar to:

```http
GET /api/products/1/price
```

I tested the endpoint with the `OPTIONS` method and discovered that it also supported `PATCH`, even though this functionality was not exposed through the normal application interface.

Attempting to use `PATCH` before authentication returned an `Unauthorized` response, indicating that the endpoint required an authenticated session.

After logging in, I repeated the request using:

```http
PATCH /api/products/1/price
```

The initial requests failed, but the API error messages revealed how the request needed to be constructed.

The server first indicated that the request required:

```http
Content-Type: application/json
```

After sending an empty JSON object:

```json
{}
```

the response revealed that a `price` parameter was required.

I then supplied the expected integer value:

```json
{
  "price": 0
}
```

The request succeeded and changed the product price to `$0.00`.

I then added the product to the basket and completed the purchase.

This lab demonstrated that API functionality may exist even when it is not used by the visible application interface.

It also showed how testing alternative HTTP methods and carefully analyzing error responses can reveal the exact structure required to interact with hidden or unintended API functionality.


### PortSwigger — Exploiting a mass assignment vulnerability

- **Difficulty:** Practitioner
- **Vulnerability:** Mass Assignment
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a mass assignment vulnerability in the checkout API.

After adding the target product to the basket and attempting to place the order, I inspected the API requests related to:

```http
/api/checkout
```

I compared the `GET` and `POST` requests and noticed an important difference.

The `GET` response contained an additional field:

```json
{
  "chosen_discount": {
    "percentage": 0
  }
}
```

This field was not included in the normal `POST /api/checkout` request.

This suggested that `chosen_discount` was part of the internal checkout object and could potentially be accepted as an undocumented input parameter.

I added the field manually to the checkout request:

```json
{
  "chosen_discount": {
    "percentage": 0
  },
  "chosen_products": [
    {
      "product_id": "1",
      "quantity": 1
    }
  ]
}
```

The request was accepted without error.

To confirm that the server was actually processing the hidden parameter, I replaced the percentage with an invalid value.

This produced a validation error, confirming that `chosen_discount` was recognized and processed by the application.

I then changed the discount percentage to:

```json
{
  "chosen_discount": {
    "percentage": 100
  }
}
```

The server accepted the value and applied a 100% discount to the order, allowing the purchase to be completed without enough account credit.

This lab demonstrated how comparing API responses with API requests can reveal hidden object properties.

It also showed that a hidden field is not automatically a vulnerability. The issue becomes exploitable when the server accepts and applies that field without properly restricting which internal properties the user is allowed to modify.
