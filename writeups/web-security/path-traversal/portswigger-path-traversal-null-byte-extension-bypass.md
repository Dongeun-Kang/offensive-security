# Lab: File path traversal, validation of file extension with null byte bypass

Like in the previous lab, intercept the image-loading request using Burp Suite.

<img width="407" height="401" alt="image" src="https://github.com/user-attachments/assets/59f04caa-f842-47ff-94c6-b083fe7ac62d" />

The original request is:

```http
GET /image?filename=7.jpg HTTP/2
```

In this case, the application validates whether the supplied filename ends with the expected `.jpg` extension.

Therefore, we need to keep `.jpg` at the end of the parameter while still attempting to access `/etc/passwd`.

Modify the request:

```http
GET /image?filename=7.jpg HTTP/2
>
GET /image?filename=../../../etc/passwd%00.jpg HTTP/2
```

Here, `%00` represents a **null byte**.

The application sees the value as ending with `.jpg`, allowing it to pass the extension validation. However, in a vulnerable backend or filesystem API, the null byte can terminate the filename before `.jpg` is processed.

As a result, the effective path becomes:

```text
../../../etc/passwd
```

<img width="405" height="499" alt="image" src="https://github.com/user-attachments/assets/fcffb01f-7b7a-442a-b951-4189eca3c804" />

The server successfully returns the contents of `/etc/passwd`.

This confirms that the file-extension validation can be bypassed using a null byte while preserving `.jpg` at the end of the supplied value.

Successfully retrieved `/etc/passwd` using a null-byte bypass.
