# Lab: Remote code execution via web shell upload

This is a lab for practicing **Remote Code Execution (RCE)** by exploiting a file upload vulnerability.

<img width="1161" height="708" alt="image" src="https://github.com/user-attachments/assets/05fa033c-d775-49d8-8b6b-b1c8205f540e" />

The objective of this lab is to exploit the application's file upload functionality and retrieve the contents of:

```text
/home/carlos/secret
```

Let's log in using the provided credentials.

<img width="951" height="456" alt="image" src="https://github.com/user-attachments/assets/d667dc5c-d8ae-43b8-b819-7d065d234b68" />

Once logged in, we can access the account page.

<img width="968" height="743" alt="image" src="https://github.com/user-attachments/assets/9882e40e-7dfa-474d-8c6c-a7a38f4a7da9" />

There is an **Avatar upload** functionality where we may be able to upload a PHP file.

Since the objective is to retrieve the contents of `/home/carlos/secret`, we can create the following PHP payload:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

Save the payload with a `.php` extension, for example:

```text
exploit.php
```

Then, upload the file through the avatar upload functionality.

<img width="404" height="87" alt="image" src="https://github.com/user-attachments/assets/03e60a57-1a83-4c2c-b182-7e978871178e" />

After uploading the file, the application confirms that the avatar has been successfully uploaded.

Now, we need to locate and access the uploaded PHP file.

Navigate to:

```text
/files/avatars/exploit.php
```

<img width="398" height="54" alt="image" src="https://github.com/user-attachments/assets/34d889d2-1579-4db2-a938-385d63ea0271" />

The PHP file is executed by the server and the contents of `/home/carlos/secret` are successfully returned.

This confirms that the file upload functionality allows executable PHP files to be uploaded and executed, resulting in **Remote Code Execution (RCE)**.
