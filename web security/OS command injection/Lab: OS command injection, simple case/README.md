<img width="319" height="404" alt="image" src="https://github.com/user-attachments/assets/df70032e-6ee8-47f8-a3cc-1223f9084efb" />
[^1]Access to one of products[^1]
<img width="812" height="40" alt="image" src="https://github.com/user-attachments/assets/92483c80-014b-44d7-ab05-498a3e42569f" />
Try to modify the URL parameter.
/product?productId=1 > /product?productId=1;whoami
Gain a response saying "Invalid product ID". That parameter isn't probably submitted as part of a OS Command.
<img width="587" height="111" alt="image" src="https://github.com/user-attachments/assets/703553cf-86fd-4c84-9bf9-b8eb88d57102" />
There is a button to check the stock.
Use Burp Suite to intercept a HTTP Request.
<img width="808" height="713" alt="image" src="https://github.com/user-attachments/assets/a65c6222-0fd5-4933-8383-4d2d874ebb53" />
Modify the parameters.
productId=1&storeId=1 > productId=1&storeId=1;whoami
<img width="575" height="129" alt="image" src="https://github.com/user-attachments/assets/c3764756-90e4-420c-b7c2-08579e230877" />
Gain the name of the current user "peter-FOQOdA"
