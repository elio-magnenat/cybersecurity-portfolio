# Path Traversal

Path traversal, also known as directory traversal, enables an attacker to read arbitrary files on the server.

These files can include:
- Application code and data
- Credentials for back-end systems
- Sensitive operating system files

## Application code and data
This category includes source code, configuration files, files uploaded by users, and application logs.
This data can be very sensitive because source code may reveal other vulnerabilities, configuration files may contain secrets, user files may contain private information, and logs may reveal tokens, IP addresses, or internal errors.

## Credentials for back-end systems
This category is even more critical because it can include database credentials, API keys, JWT secrets, SMTP passwords, or SSH private keys.
Some of these secrets may be stored in a `.env` file. This file is often used to keep configuration values outside the source code, such as database passwords, API keys, or application secrets. If an attacker can read it, they may gain access to other systems used by the application.

## Sensitive operating system files
This category includes files used by the operating system to manage users, authentication, services, and system configuration.

On Linux, `/etc/passwd` contains information about local user accounts, such as usernames, user IDs, home directories, and login shells. It does not normally contain password hashes.

Password hashes are usually stored in `/etc/shadow`, which is much more sensitive and should only be readable by privileged processes.
On Windows, the closest equivalent is the SAM database. It is a logical database, but it is stored in a physical registry file, usually located at: `C:\Windows\System32\config\SAM`
On macOS, local account information is usually stored in several files rather than in one direct equivalent of `/etc/shadow`. These records can be found under directories such as: `/private/var/db/dslocal/nodes/Default/users/`
A path traversal vulnerability could theoretically target these files, but only if the web application process has permission to read them. Path traversal bypasses application path restrictions, not operating system permissions.

## How path traversal works
A path traversal vulnerability usually occurs when an application uses user input to build a file path on the server.
For example, an application may receive the name of an image or document from a URL parameter and use it to locate the requested file.
If the application does not properly validate this input, an attacker may manipulate the path and make the server access a file outside the intended directory.
The server then reads the file with the permissions of the web application process.

## Common vulnerable patterns
Path traversal vulnerabilities often appear in features that read or manage files on the server.
Common examples include:
- Displaying images or user avatars
- Downloading documents or invoices
- Previewing uploaded files
- Loading templates or language files
- Reading logs or generated reports
- Importing or exporting files
- Uploading, deleting, or renaming files

The common pattern is that the application uses user-controlled input, such as a file name or path, to access a file on the server without safely restricting the final location.

## Limitations
A path traversal vulnerability does not automatically give access to every file on the server.
The attacker can only read files that are accessible to the operating system account running the web application.
For example, if the application does not have permission to read `/etc/shadow` or the Windows SAM file, the attack will not be able to access them.
Path traversal bypasses the application's intended directory restrictions, but it does not normally bypass operating system permissions.

## Prevention
The safest solution is to avoid using user-controlled input directly as a file path.
Whenever possible, the application should use an internal identifier instead of accepting a file name or path from the user. The server can then map this identifier to a known file.
If user input must be used, the application should:
- Use an allowlist of accepted values
- Resolve and normalize the final path
- Verify that the resolved path remains inside the intended directory
- Reject absolute paths and unexpected path separators
- Avoid relying only on blocking patterns such as `../`
- Store public and sensitive files in separate directories
- Run the application with the minimum required permissions
- Avoid exposing internal file paths in error messages

The principle of least privilege is also important. The operating system account running the web application should only have access to the files required by the application.

This does not fix the vulnerability itself, but it limits the damage if a path traversal vulnerability is exploited.

## Detection and testing
I first browse the application and look for features that display, download, upload, or manage files. I pay attention to parameters that may contain a file name or a path, such as `file`, `filename`, `path`, `document`, or `template`.
I do not only inspect the visible URL. File-related values may also be sent in URL paths, POST bodies, JSON data, form fields, cookies, or request headers.
When I identify a suspicious request, I intercept it with Burp Suite or another interception proxy. I first keep a valid request as a baseline and send it to Burp Repeater.
I then modify one value at a time and compare the responses. I look at:
- The HTTP status code
- The response body
- The response length
- The content type
- Error messages
- Response time
- Differences from the original request

On an authorized testing environment, I may test whether the application accepts traversal sequences and tries to access a file outside the intended directory. For example, `/etc/passwd` is commonly used in Linux training environments, while `C:\Windows\win.ini` can be used on Windows.
If the request fails, I try to understand why. The application may block a specific pattern, normalize the path, restrict access to an allowed directory, or lack operating system permissions to read the requested file.
A successful test should use the minimum amount of sensitive data required to prove the vulnerability.

## References

- [PortSwigger Web Security Academy — Path traversal](https://portswigger.net/web-security/file-path-traversal)
- [PortSwigger Web Security Academy — Path traversal learning path](https://portswigger.net/web-security/learning-paths/path-traversal)
- [PortSwigger — File path traversal vulnerability](https://portswigger.net/kb/issues/00100300_file-path-traversal)
- [OWASP — Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [MITRE — CWE-22: Path Traversal](https://cwe.mitre.org/data/definitions/22.html)
- [Linux manual — passwd(5)](https://man7.org/linux/man-pages/man5/passwd.5.html)
- [Linux manual — shadow(5)](https://man7.org/linux/man-pages/man5/shadow.5.html)
