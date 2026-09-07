# Lab: Web shell upload via Content-Type restriction bypass

This is a lab for practicing **Web Shell Upload** by bypassing a **Content-Type restriction**.

<img width="1153" height="754" alt="image" src="https://github.com/user-attachments/assets/93788771-085f-4539-9f26-529053a62b8b" />

Let's log in using the provided credentials.

<img width="910" height="617" alt="image" src="https://github.com/user-attachments/assets/d5e64156-0658-4141-801a-c49530356de9" />

Once logged in, we can find an avatar upload functionality where we may be able to upload a PHP payload.

However, the upload form only accepts files with `.jpg` or `.jpeg` extensions.

To bypass this client-side restriction, we can first rename the PHP payload to use a `.jpg` extension.

For example:

```text
exploit.php
```

Change it to:

```text
exploit.jpg
```

Then, select the file for upload.

<img width="286" height="181" alt="image" src="https://github.com/user-attachments/assets/3a158bfc-6009-4f86-8e9e-5a885de871fe" />

Before the request reaches the server, intercept it using Burp Suite.

<img width="760" height="294" alt="image" src="https://github.com/user-attachments/assets/6aa70a7d-c59d-46b9-b707-2955de4478b0" />

Modify the uploaded filename from:

```text
exploit.jpg
```

to:

```text
exploit.php
```

Then, forward the modified request to the server.

Once the file has been uploaded, navigate to:

```text
/files/avatars/exploit.php
```

<img width="374" height="46" alt="image" src="https://github.com/user-attachments/assets/d9a2ef66-9db3-4658-9aeb-516f7c3b2bce" />

The uploaded PHP file is successfully executed by the server.

This confirms that the upload restriction can be bypassed by modifying the filename after the client-side validation, allowing a **web shell or executable PHP file** to be uploaded and executed.
