# Fall-2021-DS-Challenge

## Question 2

### (a) How many orders were shipped by Speedy Express in total?

54 orders were shipped by Speedy Express in total. I first joined the Orders and Shippers table by Shipper ID.
Then, I filtered the resulting table with orders shipped by "Speedy Express" and count the entries. 

```sql
SELECT COUNT(*) AS NumShipped
	FROM Orders
    JOIN Shippers 
    ON Orders.ShipperID = Shippers.ShipperID
    WHERE Shippers.ShipperName = "Speedy Express"
```
