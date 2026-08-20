## Path Traversal — Absolute Path Bypass

Like in the previous lab, intercept the image-loading request using Burp Suite.

![Product image](https://github.com/user-attachments/assets/ede98a51-9ef9-4a33-b852-89c6b4028a2e)

The original request is:

```http
GET /image?filename=51.jpg HTTP/2
```

Modify the `filename` parameter to reference `/etc/passwd` using an **absolute path**:

```http
GET /image?filename=51.jpg HTTP/2
→
GET /image?filename=/etc/passwd HTTP/2
```

![Absolute path traversal result](https://github.com/user-attachments/assets/6276ec19-8e61-4ba6-bff5-4e74a8a6b59f)

The server successfully returns the contents of `/etc/passwd`.

This confirms that although the application may prevent conventional traversal sequences such as `../`, it still accepts absolute filesystem paths.

In this case, the traversal protection can therefore be bypassed by directly supplying:

```text
/etc/passwd
```

instead of:

```text
../../../etc/passwd
```
Successfully retrieved the contents of ```/etc/passwd```.

This confirms that the application blocks traversal sequences but still accepts absolute paths, allowing the restriction to be bypassed.
