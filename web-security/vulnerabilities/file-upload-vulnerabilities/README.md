# File Upload Vulnerabilities

File upload vulnerabilities occur when an application allows users to upload files to the server without sufficiently validating properties such as:

- The filename
- The file type
- The file contents
- The file size

Weak or incorrectly enforced restrictions can allow an attacker to upload files that the application was never intended to accept.

For example, a feature designed to accept profile images might accidentally allow the upload of a server-side script.

If that file is stored in a location where the web server can execute it, the vulnerability may lead to remote code execution.

Depending on how the application processes uploaded files, exploitation may happen in different ways.

In some cases, simply uploading a malicious file is enough to cause an impact.

In other cases, the attacker must make a second HTTP request to the uploaded file in order to trigger processing or execution.

Conceptually:

```text
User uploads file
        ↓
Application validates it incorrectly
        ↓
Dangerous file stored on server
        ↓
File processed or requested
        ↓
Potential security impact
```

## How File Upload Vulnerabilities Arise

Most real applications do not allow completely unrestricted file uploads. Instead, vulnerabilities usually appear because the validation logic is incomplete, inconsistent, or checks properties that the attacker can influence.

### Incomplete Blacklists

An application may try to block dangerous file types by maintaining a blacklist of forbidden extensions.

This approach is fragile because the application may:

- Forget less common but still dangerous file types.
- Interpret a filename differently from the component that later stores or executes the file.
- Apply extension checks in a way that can be bypassed by parsing differences.

The general problem is that blocking a list of known-bad values requires the developer to anticipate every dangerous representation.

### Trusting Attacker-Controlled Properties

Some applications decide whether a file is safe by checking metadata or request properties supplied by the client.

If the server trusts values that the user can modify, the validation may not reflect the file that is actually being uploaded.

Conceptually:

```text
Client sends file + metadata
        ↓
Application trusts metadata
        ↓
Attacker changes that metadata
        ↓
Dangerous file may pass validation
```

This is why file upload checks should not rely only on values declared by the client.

### Inconsistent Validation

Validation may also be applied differently across the hosts, directories, or components that make up a website.

For example, one part of the application may reject a file while another part handles the same file differently.

This creates discrepancies that can become exploitable when:

```text
Upload component
        ↓
validates file one way

Storage / serving component
        ↓
interprets file another way
```

### Key Principle

File upload vulnerabilities often arise not because no defenses exist, but because the defenses make incorrect assumptions.

The important question is whether every stage agrees on what the uploaded file actually is and whether the same security rules are enforced consistently from upload to storage and later access.

## How Web Servers Handle Requests for Static Files

To understand why uploaded files can become dangerous, it helps to understand how a web server decides what to do when a file is requested.

Historically, request paths often mapped directly to files and directories on the server's filesystem.

For example:

```text
GET /images/logo.png
```

could correspond directly to a file such as:

```text
/images/logo.png
```

on the server.

Modern web applications are often dynamic, so a request path does not necessarily correspond directly to a real file. However, web servers still serve static resources such as images, stylesheets, and other files.

When a static file is requested, the server may inspect the file extension and use its configuration to determine the file type and how it should be handled.

Conceptually:

```text
HTTP request
    ↓
Requested path
    ↓
Server identifies file extension
    ↓
Extension mapped to a file type
    ↓
Server decides how to handle the file
```

### Non-Executable Files

If the requested file type is not executable, the server can simply return its contents to the client.

For example:

```text
image.png
    ↓
Server reads file
    ↓
File contents returned in HTTP response
```

Static HTML files can also be served this way.

### Executable Files

If the requested file type is executable and the server is configured to execute that type, the server runs the file instead of simply returning its source.

For example:

```text
script.php
    ↓
Server recognizes PHP
    ↓
PHP code executes
    ↓
Generated output returned to client
```

Before execution, request information such as headers and parameters may be made available to the script.

This behavior is especially important for file upload vulnerabilities because an uploaded server-side script can become dangerous if it is stored in a location where the server will execute it.

### Executable Extension Without an Execution Handler

A server may recognize that a file type is normally executable but not be configured to execute it.

In this situation, the server may return an error.

In some configurations, however, it may serve the file contents as plain text instead.

This can expose source code or other sensitive information.

### Key Principle

The security impact of an uploaded file depends not only on whether the upload succeeds, but also on how the web server handles that file when it is later requested.

```text
Uploaded file
    ↓
Stored on server
    ↓
Requested later
    ↓
Server configuration determines:
    ├── return contents
    ├── execute as code
    └── reject / expose as text
```

## Common vulnerable patterns
### Unrestricted upload of executable files
One of the most dangerous file upload vulnerabilities happens when an application allows users to upload server-side scripts such as PHP files.
If the server stores the uploaded file in a location where it can be executed, an attacker may upload a web shell.
A web shell is a malicious script that allows commands or actions to be executed on the server through HTTP requests.
For example, a PHP file could read a file from the server:

`<?php echo file_get_contents('/path/to/target/file'); ?>`

When the uploaded PHP file is requested, the server executes the script and returns the content of the target file.
A more powerful web shell can execute commands provided through a URL parameter:

`<?php echo system($_GET['command']); ?>`

For example:

`GET /uploads/exploit.php?command=id`

In this case, PHP reads the `command` parameter and passes its value to the operating system. The command is executed with the permissions of the account running the web application.
This can turn a file upload vulnerability into remote code execution and may give an attacker significant control over the server.
### Flawed file type validation
Some applications try to validate uploaded files by checking the `Content-Type` value sent with the file.
For example, a website that only accepts images may allow MIME types such as:
- `image/jpeg`
- `image/png`

The problem is that this value is provided by the client and can be modified before the request reaches the server.
If the application trusts the `Content-Type` header without checking the actual content of the file, an attacker may be able to upload a file that should normally be rejected.
For this reason, file type validation should not rely only on the MIME type declared in the HTTP request.

## Impact

The impact of a file upload vulnerability depends mainly on two factors:

- Which properties of the uploaded file are not validated correctly, such as its type, filename, contents, or size.
- What the server allows the uploaded file to do after it has been stored.

### Executable File Uploads

The most serious case occurs when the application does not properly validate the file type and the server is configured to execute uploaded server-side scripts.

For example, if files such as:

```text
.php
.jsp
```

can be uploaded into an executable location, an attacker may be able to deploy a web shell.

A web shell can provide a way to execute commands or interact with the server through HTTP requests, potentially leading to remote code execution and significant control over the compromised system.

Conceptually:

```text
Dangerous file uploaded
        ↓
Stored in executable location
        ↓
Requested through HTTP
        ↓
Server executes the file as code
        ↓
Potential remote code execution
```

### Overwriting Existing Files

If the application does not safely validate or generate filenames, an attacker may be able to upload a file using the same name as an existing file.

Depending on how the server handles uploads, this could overwrite application data or other important files.

If file upload functionality can also be combined with a path traversal weakness, the attacker may be able to influence where the file is written and reach locations outside the intended upload directory.

### Denial of Service Through File Size

If the application does not enforce reasonable file size limits, an attacker may be able to upload very large files or many files until the server's available storage is exhausted.

This can result in a denial-of-service condition by preventing the application or other services from writing additional data.

### Key Principle

The impact of an upload vulnerability is not determined only by whether an unexpected file can be uploaded.

It also depends on what happens after the upload:

```text
What can be uploaded?
        +
Where is it stored?
        +
How does the server process it?
        ↓
Overall impact
```

## Prevention
File upload functionality should use several security controls together rather than relying on a single check.
Applications should:

- Use an allowlist of file extensions that are actually required
- Validate the real file type and not trust only the `Content-Type` value provided by the client
- Check the content of uploaded files when possible
- Generate safe filenames instead of directly using filenames provided by users
- Limit the maximum file size
- Only allow authorized users to upload files
- Store uploaded files outside the web root when possible
- Prevent uploaded files from being executed by the web server
- Apply the principle of least privilege to filesystem permissions

Using several independent protections reduces the risk that bypassing one validation mechanism results in a complete file upload vulnerability.

## Detection and testing
File upload vulnerabilities can be tested by first identifying functionality that allows users to upload files, such as:
- Profile pictures
- Documents
- Attachments
- Images
- Import features

A normal file can first be uploaded to understand how the application handles and stores it.
The upload request can then be inspected using Burp Suite to identify information such as:

- The uploaded filename
- The file extension
- The file-specific `Content-Type`
- The upload endpoint
- The location where the uploaded file can later be accessed

Different file types can then be tested to determine which restrictions are enforced by the application.
If a file is rejected based only on its MIME type, the request can be inspected to determine whether the application trusts the user-controlled `Content-Type` value.
It is also important to check what happens after the file is uploaded. If an uploaded server-side script can be requested and executed by the web server, the vulnerability may lead to remote code execution.
Tools such as Burp Suite Proxy and Repeater are useful for inspecting and modifying file upload requests during authorized security testing.

## Classification
- **CWE:** CWE-434 — Unrestricted Upload of File with Dangerous Type
- **OWASP Top 10 2025:** A06:2025 — Insecure Design
- **OWASP Top 10 2021:** A04:2021 — Insecure Design

CWE-434 describes situations where an application allows dangerous file types to be uploaded and processed within its environment.
File upload weaknesses such as CWE-434 are associated with the Insecure Design category in the OWASP Top 10.

## Labs

- [Completed labs](labs.md)

## References
- [PortSwigger Web Security Academy — File upload vulnerabilities](https://portswigger.net/web-security/file-upload)
- [OWASP — File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [MITRE — CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)
- [OWASP Top 10:2025 — A06 Insecure Design](https://owasp.org/Top10/2025/A06_2025-Insecure_Design/)
- [OWASP Top 10:2021 — A04 Insecure Design](https://owasp.org/Top10/2021/A04_2021-Insecure_Design/)
