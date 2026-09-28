# Path Traversal Labs

- PortSwigger — File path traversal, simple case

### PortSwigger — File path traversal, traversal sequences blocked with absolute path bypass

- **Difficulty:** Practitioner
- **Vulnerability:** Path Traversal — Absolute Path Bypass
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application loaded product images using a user-controlled `filename` parameter.

Directory traversal sequences such as:

```text
../
```

were blocked, preventing the usual technique of escaping the image directory with a relative path.

Instead of attempting to move upward through the filesystem, I supplied the target file directly using an absolute path:

```text
/etc/passwd
```

The resulting request used:

```text
filename=/etc/passwd
```

and the application returned the contents of the file.

The bypass worked because blocking traversal sequences did not prevent the application from accepting an absolute filesystem path.

Conceptually:

```text
Blocked approach:

../../../etc/passwd
↑
contains traversal sequences

Bypass:

/etc/passwd
↑
starts directly from the filesystem root
```

This lab demonstrated that filtering only `../` is not sufficient protection against path traversal. The application must ensure that the final resolved file path remains inside the intended directory, regardless of whether the supplied path is relative or absolute.

