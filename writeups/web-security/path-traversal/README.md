
# Path Traversal

Path traversal, also known as directory traversal, is a vulnerability that allows an attacker to access arbitrary files on the server running an application. This can expose sensitive information such as application code and data, back-end system credentials, and operating system files.

In some cases, the vulnerability may also allow an attacker to write or modify arbitrary files on the server. This can lead to changes in application data or behavior and, in severe cases, complete compromise of the server.

## Reconnaissance & Identification

*

## Testing & Payload Injection

*

## Remediation & Verification

*

## Writeups

- [Lab: File path traversal, simple case](portswigger-path-traversal-simple-case.md)
- [Lab: File path traversal, traversal sequences blocked with absolute path bypass](portswigger-path-traversal-absolute-path-bypass.md)
- [Lab: File path traversal, traversal sequences stripped non-recursively](portswigger-path-traversal-stripped-non-recursively.md)
- [Lab: File path traversal, traversal sequences stripped with superfluous URL-decode](portswigger-path-traversal-superfluous-url-decode.md)
- [Lab: File path traversal, validation of file extension with null byte bypass](portswigger-path-traversal-null-byte-extension-bypass.md)
- [Lab: File path traversal, validation of start of path](portswigger-path-traversal-start-of-path-validation.md)
