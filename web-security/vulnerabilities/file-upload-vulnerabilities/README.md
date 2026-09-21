# File Upload Vulnerabilities
File upload vulnerabilities happen when an application allows users to upload files without properly checking them.
The application may fail to verify things such as:

- The file name
- The file type
- The file content
- The file size

If these restrictions are not correctly enforced, an attacker may be able to upload files that were not intended by the application.
For example, a website that normally accepts images could allow an attacker to upload a server-side script instead.
If the server executes this file, the attacker may be able to run code on the server.
In some cases, simply uploading the file can cause a problem. In other cases, the attacker must access the uploaded file afterward to trigger its execution.

## Common vulnerable patterns
### Weak file validation
Most websites have some protections to prevent users from uploading dangerous files. However, having file upload restrictions does not mean that they are secure. If the validation is incomplete or incorrectly implemented, an attacker may be able to bypass it and upload a file that should normally be rejected. In some cases, this can allow an attacker to upload a server-side script and obtain a web shell, leading to remote code execution.For example, an application may block some dangerous file extensions but forget other extensions that can also be dangerous.
The application may also check properties of the uploaded file that can be modified by the user. An attacker can change these values using tools such as Burp Suite.
Another problem can happen when different parts of the application validate files differently. A file may be rejected in one location but accepted in another.
For this reason, having file upload validation is not enough. The validation must correctly check the uploaded file and be applied consistently.

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
The impact of a file upload vulnerability depends on what files can be uploaded and how the server handles them.
A successful attack may allow an attacker to:

- Upload and execute server-side scripts
- Achieve remote code execution through a web shell
- Read sensitive files stored on the server
- Modify or overwrite files if the web application has sufficient permissions
- Upload malicious content that could affect other users
- Consume server storage by uploading very large files

In the most serious cases, an attacker may gain significant control over the server by executing commands with the permissions of the web application.

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
