# Fall-2021-DS-Challenge

## Question 2

### (a) How many orders were shipped by Speedy Express in total?

54 orders were shipped by Speedy Express in total. I first joined the Orders and Shippers table by Shipper ID.
Then, I filtered the resulting table with orders shipped by "Speedy Express" and counted the entries. 

```sql
SELECT COUNT(*) AS NumShipped
FROM Orders o 
JOIN Shippers s
ON o.ShipperID = s.ShipperID
WHERE s.ShipperName = "Speedy Express"
```

### (b) What is the last name of the employee with the most orders?

The last name of the employee with the most orders is Peacock. I joined the Orders and Employees table by Employee ID.
Then, I grouped the resulting table by employee's last name then returned the top row after ordering the number of orders
in decreasing order. 

```sql
SELECT e.LastName, COUNT(*) AS NumOrders
FROM Orders o 
JOIN Employees e
ON o.EmployeeID = e.EmployeeID
GROUP BY e.LastName
Order By NumOrders DESC
LIMIT 1
```

### (c) What product was ordered the most by customers in Germany?

Boston Crab Meat (Product ID 40) was ordered the most by customers in Germany.
For this problem, I decided to first make a temporary table to gather information that is 
scattered across different tables. I joined Orders, OrderDetails, Products, and Customers
then grouped the temporary table by ProductID so I could find the number of sales 
for each product using SUM. Then I filtered the result by orders placed in Germany and 
chose the product with most orders from the temp table using MAX.

```sql
WITH temp as (SELECT od.ProductID, p.ProductName,
	      SUM(Quantity) AS total
    FROM OrderDetails od
    JOIN Orders o 
    	ON od.OrderID = o.OrderID
    JOIN Products p 
    	ON od.ProductID = p.ProductID
    JOIN Customers c
    	ON c.CustomerID = o.CustomerID
    WHERE Country = "Germany"
    GROUP BY od.ProductID
   )
 SELECT * FROM temp
 WHERE total = (SELECT MAX(total) FROM temp)
```
