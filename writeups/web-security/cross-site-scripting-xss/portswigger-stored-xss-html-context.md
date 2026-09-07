# Lab: Stored XSS into HTML context with nothing encoded

This is a lab for practicing **Stored Cross-Site Scripting (Stored XSS)** exploitation.

<img width="1152" height="660" alt="image" src="https://github.com/user-attachments/assets/7fa667e8-01f8-4698-88c8-afeef09b82bd" />

The objective of this lab is to exploit a stored XSS vulnerability and successfully call the JavaScript `alert()` function.

Once we access the given URL, we are presented with a blog page containing multiple posts.

<img width="922" height="826" alt="image" src="https://github.com/user-attachments/assets/e0931ffc-71e6-4e8d-8665-eded5e885dac" />

Let's have a look at one of the posts.

<img width="925" height="768" alt="image" src="https://github.com/user-attachments/assets/1035608f-9906-49b5-94de-b7dd53b0c61e" />

Looking at the post page, we can identify an input field that may be useful for delivering an XSS payload.

<img width="924" height="763" alt="image" src="https://github.com/user-attachments/assets/c33a0596-9d95-4e17-ae53-cd1353acb3d7" />

Let's submit the following payload:

```html
<script>alert("Hi");</script>
```

<img width="991" height="218" alt="image" src="https://github.com/user-attachments/assets/6807b5ce-5fc4-4dc8-a95f-43e3273dedfb" />

The alert does not appear immediately.

This is expected because this is a **Stored XSS** vulnerability. Unlike reflected XSS, the payload is stored by the application and executed when the affected page is loaded.

Let's go back to the post and check whether the payload was stored successfully.

<img width="530" height="184" alt="image" src="https://github.com/user-attachments/assets/ea1ccc8e-1b5a-47f9-9a3b-4d63032c7856" />

The JavaScript `alert()` function is successfully executed when the post is loaded.

This confirms that the input is vulnerable to **Stored Cross-Site Scripting (Stored XSS)**.
