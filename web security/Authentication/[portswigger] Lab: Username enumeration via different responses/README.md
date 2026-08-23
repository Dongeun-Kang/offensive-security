# Credential Brute Force using Burp Suite Intruder

This is a lab for practicing **credential brute force attacks** using **Burp Suite Intruder**.

<img width="1129" height="715" alt="image" src="https://github.com/user-attachments/assets/7e41bbfe-c4ec-4c67-84c4-a9e9bf695604" />

The objective of this lab is to identify a valid username and password using the provided wordlists.

Navigate to the login page and submit any credentials while intercepting the request using Burp Suite.

<img width="961" height="479" alt="image" src="https://github.com/user-attachments/assets/dfe5cc8c-7922-4631-a34c-f2e6e0e0866e" />

Send the intercepted login request to **Intruder**.

<img width="1210" height="505" alt="image" src="https://github.com/user-attachments/assets/d3789d0d-de48-4c39-9aa0-b0f25e60c389" />

First, add the `username` parameter as the payload position and load the provided username list.

Start the attack and compare the responses.

<img width="1672" height="71" alt="image" src="https://github.com/user-attachments/assets/2692bc09-c1da-42fd-af60-dad25909ed7b" />

We can see that the response for the username `apache` has a different response length compared to the other attempts.

Let's inspect the response.

<img width="327" height="123" alt="image" src="https://github.com/user-attachments/assets/171e28a5-1e11-47a2-9481-6f3b8f79f594" />

Instead of returning:

```text
Invalid username
```

the application returns:

```text
Incorrect password
```

This indicates that `apache` is a valid username because the application has moved on to validating the password.

Now, change the username value to:

```text
apache
```

Then, set the `password` parameter as the new payload position and load the provided password list.

<img width="379" height="105" alt="image" src="https://github.com/user-attachments/assets/bdf76db2-c1fc-4b07-b73c-3c57b2cd0095" />

Start another Intruder attack using the password list.

<img width="1233" height="75" alt="image" src="https://github.com/user-attachments/assets/18aa917e-aa90-4dec-84c9-7a3e7e7cf341" />

Once again, compare the response lengths and look for an unusual result.

<img width="565" height="118" alt="image" src="https://github.com/user-attachments/assets/1923be3c-53af-43b4-9899-939241d4f3ff" />

The unusual response indicates that the correct password has been identified.

Using the discovered username and password, we can successfully log in to the application.

This confirms that the login functionality is vulnerable to **credential brute force attacks**, and the difference in error messages also allows **username enumeration**.
