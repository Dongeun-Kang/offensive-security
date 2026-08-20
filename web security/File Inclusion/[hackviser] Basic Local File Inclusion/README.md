# Basic Local File Inclusion

This is a basic-level lab for practicing **Local File Inclusion (LFI)** exploitation.

![Lab](https://github.com/user-attachments/assets/abed8e52-821f-4428-8b7d-68a402a28010)

Once we access the given URL, we are presented with a **404 page**.

![404 page](https://github.com/user-attachments/assets/ec8ebc40-e24f-40f5-915d-859bf7b13bf7)

Looking carefully at the URL, we can identify a parameter that may be worth testing.

![Page parameter](https://github.com/user-attachments/assets/1ea756f6-d812-4e1d-8b7f-eeddf97003d7)

The application is using the following parameter:

```text
/index.php?page=404.php
```

If the value of the `page` parameter is passed to a **filesystem path** and insufficient validation is applied, the parameter may be vulnerable to LFI.

Modify the parameter:

```text
/index.php?page=404.php
→
/index.php?page=/etc/passwd
```

![LFI result](https://github.com/user-attachments/assets/e036c149-3008-4e22-8dbe-23c0983abb51)

The contents of `/etc/passwd` are successfully returned.

This confirms that the `page` parameter is vulnerable to **Local File Inclusion (LFI)**, allowing arbitrary local files accessible by the web application to be read.
