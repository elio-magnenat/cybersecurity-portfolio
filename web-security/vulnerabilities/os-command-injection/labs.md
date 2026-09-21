# OS Command Injection Labs

## PortSwigger — OS command injection, simple case

- **Difficulty:** Apprentice
- **Vulnerability:** OS Command Injection
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application contained a stock checker that used user-controlled product and store IDs inside an operating system command.
The objective was to execute the `whoami` command and identify the operating system user running the application.
I intercepted the stock check request with Burp Suite and modified the `storeId` parameter:

```text
storeId=1|whoami
```

The `|` character is a shell pipe operator. It allowed the injected `whoami` command to be executed by the server.
The response contained the result of `whoami`, confirming the OS command injection vulnerability.
I initially tried using `&`, but the request used `application/x-www-form-urlencoded`, where `&` is also used to separate HTTP parameters. This helped me understand that characters can be interpreted differently depending on whether they are processed by HTTP or by the operating system shell.
