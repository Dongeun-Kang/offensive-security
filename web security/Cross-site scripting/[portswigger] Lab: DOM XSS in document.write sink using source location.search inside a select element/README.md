# Lab: DOM XSS in document.write sink using source location.search inside a select element

This is a lab for practicing **DOM-based Cross-Site Scripting (DOM XSS)** exploitation.

<img width="1144" height="754" alt="image" src="https://github.com/user-attachments/assets/a8d41c65-d209-4676-be3d-ec8278f50aa2" />

The objective of this lab is to exploit a DOM XSS vulnerability and successfully call the JavaScript `alert()` function.

Let's have a look at one of the posts.

<img width="386" height="429" alt="image" src="https://github.com/user-attachments/assets/494e93ef-b44d-43ae-98f0-cb429ac70201" />

Looking at the page, we can identify an interesting input that may be worth investigating.

<img width="626" height="90" alt="image" src="https://github.com/user-attachments/assets/ce114885-7c50-4ce2-9dbb-2a62438404be" />

Let's interact with it and inspect the page source to see how the value is handled.

<img width="938" height="372" alt="image" src="https://github.com/user-attachments/assets/5b2194c5-4a82-40c5-af24-f83fbe796b6b" />

From the code, we can identify that the parameter name is `storeId`.

Since the supplied value is inserted into a `<select>` element, we can try to break out of the existing HTML context and inject a new element.

Let's use the following payload:

```html
"></select><img src=1 onerror=alert("Hi")>
```

The payload first closes the existing `<select>` element using:

```html
"></select>
```

Then, it injects an `<img>` element:

```html
<img src=1 onerror=alert("Hi")>
```

The browser attempts to load the image from `src=1`. Since the image does not exist, the request fails and the `onerror` event handler is triggered, executing the `alert()` function.

I initially tried to modify the request using Burp Suite, but the payload did not execute as expected. I am not yet sure why, so I will investigate the reason and update this write-up once I understand the behaviour.

Instead, we can directly modify the URL.

```text
/product?productId=1
```

Change it to:

```text
/product?productId=1&storeId="></select><img src=1 onerror=alert("Hi")>
```

<img width="566" height="205" alt="image" src="https://github.com/user-attachments/assets/b07cfa2d-a044-4ea8-a246-e3fa033bf660" />

The JavaScript `alert()` function is successfully executed.

This confirms that the `storeId` parameter is vulnerable to **DOM-based Cross-Site Scripting (DOM XSS)**.
