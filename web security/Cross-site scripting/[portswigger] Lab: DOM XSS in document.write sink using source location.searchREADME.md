# Lab: DOM XSS in document.write sink using source location.search

This is a lab for practicing **DOM-based Cross-Site Scripting (DOM XSS)** exploitation.

<img width="1144" height="766" alt="image" src="https://github.com/user-attachments/assets/062d3bd2-3726-4969-bd5d-597511c28c01" />

The objective of this lab is to exploit a DOM XSS vulnerability and successfully call the JavaScript `alert()` function.

Looking at the page, we can identify an input field that may be worth testing.

<img width="919" height="133" alt="image" src="https://github.com/user-attachments/assets/725bf81e-7d39-4161-bf9f-32ad586d34e9" />

Let's enter an arbitrary value and use the browser's **Inspector** to see how the input is handled in the DOM.

<img width="455" height="58" alt="image" src="https://github.com/user-attachments/assets/403f04a4-045e-418d-8b8f-8357f22a3a33" />

We can see that the supplied value is inserted into an `<img>` element.

This means we may be able to break out of the existing HTML attribute and inject a new element.

Let's use the following payload:

```html
"><svg onload=alert("Hi")>
```

The initial `">` closes the existing attribute and `<img>` element context, while the injected `<svg>` element executes the `onload` event handler.

<img width="917" height="121" alt="image" src="https://github.com/user-attachments/assets/246eda23-e1fb-4af6-a652-30aa30161a12" />

<img width="539" height="181" alt="image" src="https://github.com/user-attachments/assets/6b720fcc-dab4-4267-8b85-de08a546cdeb" />

The JavaScript `alert()` function is successfully executed.

This confirms that the application is vulnerable to **DOM-based Cross-Site Scripting (DOM XSS)**.
