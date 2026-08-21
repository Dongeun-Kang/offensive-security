# Lab: Reflected XSS into HTML context with nothing encoded

This is a basic-level lab for practicing **Cross-Site Scripting (XSS)** exploitation.

<img width="940" height="325" alt="image" src="https://github.com/user-attachments/assets/560c4568-0a42-40d2-b407-b96991a2cf59" />

Once we access the given URL, we are presented with the web application.

<img width="1169" height="673" alt="image" src="https://github.com/user-attachments/assets/a96872ce-56fc-4b97-82ce-699721a5cd7e" />

The objective of this lab is to exploit an XSS vulnerability and successfully call the JavaScript `alert()` function.

Looking at the page, we can identify an input field that may be worth testing.

<img width="940" height="325" alt="image" src="https://github.com/user-attachments/assets/13b5e73a-ca42-49d2-8183-8842edc5f3da" />

Let's try to submit a simple XSS payload:

```html
<script>alert("Hi");</script>
```

<img width="535" height="187" alt="image" src="https://github.com/user-attachments/assets/f0d0f19a-9311-4795-9baa-ccdf82dfb66e" />

The JavaScript `alert()` function is successfully executed.

This confirms that the input is vulnerable to **Cross-Site Scripting (XSS)**.
