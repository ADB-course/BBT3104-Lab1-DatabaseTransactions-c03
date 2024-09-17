[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/r-tQZu0l)
# BBT3104-Lab1of6-DatabaseTransactions


| **Key**                                                               | Value                                                                                                                                                                              |
|---------------|---------------------------------------------------------|
| **Group Name**                                                               | ? |
| **Semester Duration**                                                 | 19<sup>th</sup> August - 25<sup>th</sup> November 2024                                                                                                                       |

## Flowchart
                                     +-------------------+
|  Begin Transaction  |
+-------------------+
           |
           |
           v
+-------------------+
| Calculate Latest   |
|  Order Number      |
+-------------------+
           |
           |
           v
+-------------------+
| Create New Order   |
+-------------------+
           |
           |
           v
+-------------------+
| Insert First Product|
+-------------------+
           |
           |
           v
+-------------------+
| Insert Second Product|
|  (Optional Rollback) |
+-------------------+
           |         |
           |  Yes    | No
           |  (Rollback)|
           v         v
+-------------------+       +-------------------+
| Rollback to Savepoint|       | Proceed to Insert  |
|  (sp_second_product) |       |  Third Product     |
+-------------------+       +-------------------+
           |                       |
           |                       |
           v                       v
+-------------------+       +-------------------+
|  End Transaction  |       | Insert Third Product|
+-------------------+       +-------------------+
                                 |
                                 |
                                 v
+-------------------+
| Insert Payment    |
+-------------------+
           |
           |
           v
+-------------------+
| Commit Transaction|
+-------------------+
           |
           |
           v
+-------------------+
| Select Order Details|
+-------------------+
           |
           |
           v
+-------------------+
|  End              |
+-------------------+
## Pseudocode
Use database `classicmodels`
Begin Transaction
Calculate Latest Order Number
  Retrieve `MAX(orderNumber)` from `orders` table
    Increment by 1
Create New Order
  Insert into `orders` table with calculated `orderNumber`
Insert First Product
 Insert into `orderdetails` table
 Insert into `orderdetails` table (This might be rolled back)
  Update `quantityInStock` in `products` table for the inserted product (This might be rolled back)
  Roll back to the savepoint created before the second product insertion
Savepoint (Optional)** (Create a savepoint before adding the third product)
Insert Third Product
  Insert into `orderdetails` table
 Update `quantityInStock` in `products` table for the inserted product
  Insert into `payments` table for the customer
Commit Transaction
 Commit all changes made within the transaction
 Select order details from `orderdetails` table for the created order
End

## Support for the Sales Departments' Report
