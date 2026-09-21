# API Testing

API testing consists of examining an application's APIs to understand their attack surface and identify potential security weaknesses.

This section documents the concepts and techniques I studied while working through API security labs.

## Topics

- [API Reconnaissance](api-reconnaissance.md)
- [API Endpoints](endpoints.md)
- [Hidden Parameters and Mass Assignment](hidden-parameters-and-mass-assignment.md)
- [Server-Side Parameter Pollution](server-side-parameter-pollution.md)
- [Practical Labs](labs.md)

## Main Testing Workflow

```text
Discover API surface
        ↓
Identify endpoints
        ↓
Understand methods and formats
        ↓
Find hidden functionality
        ↓
Test parameters and object properties
        ↓
Investigate server-side request construction
        ↓
Confirm security impact
```
