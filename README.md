[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/r-tQZu0l)
# BBT3104-Lab1of6-DatabaseTransactions


| **Key**                                                               | Value                                                                                                                                                                              |
|---------------|---------------------------------------------------------|
| **Group Name**   
GROUP C3                                                            | ? |
| **Semester Duration**                                                 | 19<sup>th</sup> August - 25<sup>th</sup> November 2024                                                                                                                       |

## Flowchart //to  understand the transaction
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
## Pseudocode // to understand the transaction
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
procedure create_new_order(customer_id, product_ids):
    begin transaction;

    -- Calculate the latest order number
    set latest_order_number = (select max(orderNumber) from orders);
    set new_order_number = latest_order_number + 1;

    -- Create a new order
    insert into orders (orderNumber, customerNumber, orderDate, requiredDate, shippedDate, status, comments)
    values (new_order_number, customer_id, current_date, current_date + 30, null, 'Processing', '');

    -- Insert the first product
    insert into orderdetails (orderNumber, productCode, quantityOrdered, priceEach, orderLineNumber)
    values (new_order_number, product_ids[1], 1, (select price from products where productCode = product_ids[1]), 1);

    -- Update the product's quantity in stock
    update products
    set quantityInStock = quantityInStock - 1
    where productCode = product_ids[1];

    -- Optional: Create a savepoint before adding the second product
    savepoint before_second_product;

    -- Insert the second product (potentially rolled back)
    insert into orderdetails (orderNumber, productCode, quantityOrdered, priceEach, orderLineNumber)
    values (new_order_number, product_ids[2], 1, (select price from products where productCode = product_ids[2]), 2);

    -- Update the product's quantity in stock (potentially rolled back)
    update products
    set quantityInStock = quantityInStock - 1
    where productCode = product_ids[2];

    -- Roll back to the savepoint if necessary
    if <condition to roll back> then
        rollback to savepoint before_second_product;
    end if;

    -- Insert the third product
    insert into orderdetails (orderNumber, productCode, quantityOrdered, priceEach, orderLineNumber)
    values (new_order_number, product_ids[3], 1, (select price from products where productCode = product_ids[3]), 3);

    -- Update the product's quantity in stock
    update products
    set quantityInStock = quantityInStock - 1
    where productCode = product_ids[3];

    -- Insert the payment (assuming a simple payment model)
    insert into payments (customerNumber, checkNumber, paymentDate, amount, invoiceNumber)
    values (customer_id, 'CHECK001', current_date, (select sum(priceEach * quantityOrdered) from orderdetails where orderNumber = new_order_number), new_order_number);

    -- Commit the transaction
    commit;

    -- Select order details for the created order
    select * from orderdetails where orderNumber = new_order_number;
end procedure;

//suppose the sales deprtment accpets paymentsin installments
SELECT 
    o.orderNumber,
    o.customerNumber,
    SUM(od.quantityOrdered * od.priceEach) AS total_order_amount,
    SUM(op.payment_amount) AS total_payments,
    SUM(od.quantityOrdered * od.priceEach) - SUM(op.payment_amount) AS balance_due
FROM
    orders o
JOIN
    orderdetails od ON o.orderNumber = od.orderNumber
LEFT JOIN
    order_payments op ON o.orderNumber = op.order_id
GROUP BY
    o.orderNumber,
    o.customerNumber
HAVING
    SUM(od.quantityOrdered * od.priceEach) - SUM(op.payment_amount) > 0;
