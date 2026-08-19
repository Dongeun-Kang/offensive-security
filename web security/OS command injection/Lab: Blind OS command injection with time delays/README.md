# Blind OS Command Injection

## 1. Submit Feedback Request

The application provides a **Submit Feedback** page.

<img width="918" height="776" alt="image" src="https://github.com/user-attachments/assets/2090b68d-0287-4d63-9c44-7084224df38b" />

I submitted a normal feedback form and intercepted the request.

<img width="805" height="754" alt="image" src="https://github.com/user-attachments/assets/721f2141-bfec-4cd4-909b-36cf2150f923" />

The original request body was:

```http
csrf=cyp5c8AjlGhMikHv7NRAAlzNfqlXSNSm&name=1&email=test%40test.com&subject=1&message=1
```

## 2. Identifying the Vulnerable Parameter

I tested the parameters individually and found that only the `email` parameter showed behavior consistent with OS command execution.

This suggests that the `email` value is likely passed to an **OS command or shell command sink**, while the other parameters are handled differently.

I modified the request as follows:

```http
csrf=cyp5c8AjlGhMikHv7NRAAlzNfqlXSNSm&name=1&email=test||ping -c 10 127.0.0.1;&subject=1&message=1
```

## 3. Why `||` Is Used

`||` is the shell **OR operator**.

```bash
command1 || command2
```

The second command runs only if the first command fails.

In this case:

```bash
test || ping -c 10 127.0.0.1
```

If the original command using the `email` value fails, the injected `ping` command is executed.

A single `|` is different because it is a **pipe operator**, which passes the output of one command into another.

Therefore, `||` is used to control command execution flow, not to satisfy the email format.

## 4. Confirming Blind OS Command Injection

<img width="409" height="386" alt="image" src="https://github.com/user-attachments/assets/58fb305f-013d-44b3-8171-2629e4db0902" />

The server response was delayed by approximately 10 seconds.

The command output was not directly displayed in the response, but the delay confirmed that the injected command was executed.

This confirms a:

> **Time-Based Blind OS Command Injection**
