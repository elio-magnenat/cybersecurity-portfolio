# OS Command Injection
OS command injection, also called shell injection, is a vulnerability that allows an attacker to execute operating system commands on the server running a web application.
This can be very dangerous because the attacker may be able to access or modify application data and interact directly with the server's operating system.
In some cases, the attacker may also use the compromised server to access other systems inside the organization's network.
## Common vulnerable patterns

### User input passed to shell commands
OS command injection can happen when an application includes user-controlled input directly inside a system command.
For example, a shopping application may receive a product ID and a store ID:

`/stockStatus?productID=381&storeID=29`

The server may use these values to build a command such as:

`stockreport.pl 381 29`

If the application does not properly validate the input, an attacker may be able to add shell characters that change the command executed by the server.
For example, an input such as:

`& echo test &`

could result in a command similar to:

`stockreport.pl & echo test & 29`

In this example, the shell interprets the separators and executes the injected `echo` command separately.
If `test` appears in the server's response, this can indicate that the injected command was successfully executed.
The main problem is that user-controlled data is passed directly to a system shell without being safely handled.

## Impact

OS command injection can allow an attacker to execute commands on the operating system with the privileges of the vulnerable application.
Depending on these privileges, an attacker may be able to:

- Read sensitive files and application data.
- Modify or delete files on the server.
- Gather information about the operating system and network.
- Execute additional programs or commands.
- Use the compromised server to access other systems inside the network.

In severe cases, this can lead to a complete compromise of the application and the server.
## Prevention

The safest way to prevent OS command injection is to avoid executing operating system commands when possible and use safer programming language functions or libraries instead.
If executing a system command is necessary:

- Do not directly include user-controlled input inside shell commands.
- Validate input using a strict allowlist of expected values.
- Restrict input to the expected format, such as numbers when only numeric IDs are required.
- Avoid passing commands through a shell when it is not necessary.
- Run the application with the minimum privileges required.

Input validation alone should not be used as the only protection.
## Detection and testing
OS command injection can be tested by identifying inputs that may be used by the server to execute operating system commands.
Possible targets include parameters related to:

- File operations.
- Network utilities.
- System administration functions.
- Product or stock checking functions.
- Server-side processing tools.

A tester can modify these inputs and observe whether shell operators affect the application's behavior.
For example, commands such as `whoami` can help confirm whether command execution is possible.
The response may directly contain the output of the injected command. Some command injection vulnerabilities are blind, meaning that the command executes but its output is not directly returned in the HTTP response.
### Useful commands after confirming command execution

After confirming an OS command injection vulnerability, simple system commands can help identify the environment in which the application is running.

| Purpose | Linux | Windows |
| --- | --- | --- |
| Current user | `whoami` | `whoami` |
| Operating system | `uname -a` | `ver` |
| Network configuration | `ifconfig` | `ipconfig /all` |
| Network connections | `netstat -an` | `netstat -an` |
| Running processes | `ps -ef` | `tasklist` |

These commands can reveal information about the operating system, the account running the application, the network configuration, and the processes running on the server.

## Classification

- **CWE:** CWE-78 — Improper Neutralization of Special Elements used in an OS Command
- **OWASP Top 10 2025:** A05 — Injection
- **Category:** Server-side vulnerability
- **Impact type:** Arbitrary operating system command execution

## Labs completed
### PortSwigger — OS command injection, simple case

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

## References

- [PortSwigger Web Security Academy — OS command injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP — Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [OWASP Web Security Testing Guide — Testing for Command Injection](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/12-Testing_for_Command_Injection)
- [OWASP Cheat Sheet Series — OS Command Injection Defense](https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html)
- [MITRE — CWE-78: OS Command Injection](https://cwe.mitre.org/data/definitions/78.html)
