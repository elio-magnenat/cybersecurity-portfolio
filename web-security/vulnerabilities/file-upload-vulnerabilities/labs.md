# File Upload Vulnerabilities Labs

## PortSwigger — Remote code execution via web shell upload
- **Difficulty:** Apprentice
- **Vulnerability:** Unrestricted file upload leading to remote code execution
- **Result:** Solved
- **Tool used:** Burp Suite Proxy / Repeater

## PortSwigger — Web shell upload via Content-Type restriction bypass
- **Difficulty:** Apprentice
- **Vulnerability:** Flawed file type validation
- **Result:** Solved
- **Tools:** Burp Suite Proxy / Repeater

The application tried to restrict avatar uploads to image files by checking the MIME type provided in the request.
I first attempted to upload a PHP web shell, but the application rejected it because its `Content-Type` was not an allowed image type.
Using Burp Repeater, I modified the file-specific `Content-Type` value to `image/jpeg` while keeping the uploaded file as `exploit.php`. The server trusted this user-controlled value and accepted the file.
I then requested the uploaded PHP file from `/files/avatars/exploit.php`. The server executed the script and returned the contents of `/home/carlos/secret`.
This lab demonstrates why file upload validation should not rely only on the `Content-Type` value provided by the client, as it can be modified and used to bypass weak MIME type restrictions.
