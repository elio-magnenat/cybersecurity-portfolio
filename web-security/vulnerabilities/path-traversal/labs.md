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

### PortSwigger — File path traversal, traversal sequences stripped non-recursively

- **Difficulty:** Practitioner
- **Vulnerability:** Path Traversal — Non-Recursive Traversal Sequence Stripping
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application loaded product images using a user-controlled `filename` parameter.

The server attempted to defend against path traversal by removing traversal sequences such as:

```text
../
```

However, the filtering was performed non-recursively.

Instead of using a standard traversal payload, I supplied nested traversal sequences:

```text
....//....//....//etc/passwd
```

Each nested sequence contains a valid `../` pattern inside a larger string.

When the application removed the inner traversal sequence once, the remaining input became:

```text
../../../etc/passwd
```

The application then used this resulting path without performing another sanitization pass.

Conceptually:

```text
Input:
....//....//....//etc/passwd

        ↓ non-recursive stripping

../../../etc/passwd

        ↓ filesystem resolution

/etc/passwd
```

The response returned the contents of `/etc/passwd`, solving the lab.

This lab demonstrated why removing traversal patterns with a single string-replacement pass is not a reliable defense. The sanitization process can itself produce a valid traversal sequence if nested input is used.

### PortSwigger — File path traversal, traversal sequences stripped with superfluous URL-decode

- **Difficulty:** Practitioner
- **Vulnerability:** Path Traversal — Double URL-Encoding Bypass
- **Result:** Solved
- **Tool used:** Burp Suite Repeater

The application loaded product images using a user-controlled `filename` parameter.

It attempted to block path traversal sequences before performing an additional URL-decoding step. This created a mismatch between the representation checked by the filter and the representation later used by the application.

I used the following payload:

```text
..%252f..%252f..%252fetc/passwd
```

The encoded separator is processed in stages:

```text
%252f
   ↓ first decoding
%2f
   ↓ second decoding
/
```

As a result, the supplied value eventually becomes equivalent to:

```text
../../../etc/passwd
```

The traversal sequence therefore appears only after the application's filtering step has already taken place.

Conceptually:

```text
Attacker input
..%252f..%252f..%252fetc/passwd
        ↓
Filter does not see the final ../ sequences
        ↓
Additional URL decoding
        ↓
../../../etc/passwd
        ↓
Filesystem resolves the path
        ↓
/etc/passwd
```

The response returned the contents of `/etc/passwd`, solving the lab.

This lab demonstrated that path validation can fail when filtering and URL decoding occur in the wrong order. Security checks must be applied to the final canonical path rather than to an earlier encoded representation.

