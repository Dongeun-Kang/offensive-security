# Lab: Username enumeration via subtly different responses

This is a lab for practicing username enumeration via subtly different responses.

<img width="1115" height="789" alt="image" src="https://github.com/user-attachments/assets/31f1d325-dbb9-4f23-a0a4-8fa076c93ad0" />

The objective of this lab is to identify a valid username and password by detecting subtle differences in the application's responses.

The initial process is the same as the previous lab, so I'll skip those steps.

Previous lab:
https://github.com/Dongeun-Kang/offensive-security/tree/main/web%20security/Authentication/%5Bportswigger%5D%20Lab%3A%20Username%20enumeration%20via%20different%20responses

This time, the responses are only subtly different, so simply comparing the response length may not be enough.

We need to extract the error message from each response and compare them directly.

<img width="193" height="41" alt="image" src="https://github.com/user-attachments/assets/bf3cb84f-80c7-40cc-9871-7978b7252118" />

In Burp Suite Intruder, locate the `Grep - Extract` function and configure it to extract the error message from the response.

<img width="1020" height="901" alt="image" src="https://github.com/user-attachments/assets/c7e7e379-e11b-4db2-ad7d-805579a61c24" />

Start the attack and compare the extracted error messages.

<img width="1850" height="69" alt="image" src="https://github.com/user-attachments/assets/64da18fb-f940-467f-8af3-8e9ea0a87f38" />

One response contains a slightly different error message from the others.

This indicates that the corresponding username is valid.

Now, set the discovered username as a fixed value and repeat the same brute-force process against the `password` parameter using the provided password list.

<img width="755" height="22" alt="image" src="https://github.com/user-attachments/assets/6959b1fd-8c5e-4078-b1e0-75ebd10451f5" />

The different response allows us to identify the correct password.

Using the discovered username and password, we can successfully log in to the application.
