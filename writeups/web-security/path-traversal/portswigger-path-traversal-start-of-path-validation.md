# Lab: File path traversal, validation of start of path

Like in the previous lab, intercept the image-loading request using Burp Suite.

<img width="406" height="421" alt="image" src="https://github.com/user-attachments/assets/b4b095ff-9740-434e-be18-647625e47b8b" />

The original request is:

```http
GET /image?filename=/var/www/images/37.jpg HTTP/2
```

In this case, the application validates that the supplied file path starts with the expected directory:

```text
/var/www/images/
```

Therefore, instead of removing the expected path, keep it at the beginning of the parameter and append path traversal sequences.

Modify the request:

```http
GET /image?filename=/var/www/images/37.jpg HTTP/2
>
GET /image?filename=/var/www/images/../../../etc/passwd HTTP/2
```

<img width="407" height="461" alt="image" src="https://github.com/user-attachments/assets/ee74015e-de04-4f4e-8f2b-6120bc7a9180" />

The server successfully returns the contents of `/etc/passwd`.

This confirms that the application only validates whether the supplied path begins with the expected directory.

By keeping `/var/www/images/` at the beginning of the input and adding `../../../etc/passwd`, the initial validation is satisfied while the traversal sequences cause the filesystem path to resolve outside the intended image directory.

Successfully retrieved the contents of `/etc/passwd` by bypassing the start-of-path validation.
