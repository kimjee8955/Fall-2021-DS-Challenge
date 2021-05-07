# Fall-2021-DS-Challenge

## Question 1
(The answers below are a summary of my insights. Please take a look at the jupyter notebook I have uploaded for the full code 
that generated the answers.)

### (a) Think about what could be going wrong with our calculation. Think about a better way to evaluate this data. 

The high AOV of \$3,145.13 comes from naively calculating the mean of the order_amount of a highly right-skewed data without examining outliers such as bulk orders and stores that sell more expensive brands.

### (b) What metric would you report for this dataset?

1. I would report the median because it is less sensitive to outliers and our dataset is skewed.
2. If I want to report the mean, I can take advantage of the Central Limit Theorem and find the mean of the sampling distribution of the sample means.

### (c) What is its value?

1. The AOV is `$284` if we find the median of the entire dataset. The AOV is `$272` if we find the median after getting rid of outliers using IQR. However, it must be noted that this way of removing outliers is quite naive and we would need more domain knowledge to be more selective about the process. 
```python
#median
df.order_amount.median()

#remove outliers
df_clean = df[(df.order_amount < Q2 + IQR * 1.5) & (df.order_amount > Q2 - IQR * 1.5)]
df_clean['order_amount'].median()
```
2. The AOV is `$283.29` if we find the mean of the sampling distribution of the sample means after getting rid of outliers. The code snippet below demonstrates how I did the sampling (I used a sample size of 500 and did 10000 iterations). We can see that the sampling distribution is approximately normal after performing this method.
```python
np.random.seed(1)
size= 500
sample_means = np.repeat(0,10000)

for i in range(10000):
    sample = df_clean['order_amount'].sample(n=size)
    sample_means[i] = sample.mean()

sample_means.mean()
```
<img width="395" alt="Screen Shot 2021-05-06 at 11 05 18 PM" src="https://user-images.githubusercontent.com/54642556/117404858-88ada500-aebf-11eb-8aa0-892c03e14d43.png">



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
I joined Orders, OrderDetails, Products, and Customers then grouped the resulting 
table by ProductID so I could find the number of total sales for each product using SUM.
Then I filtered that result by orders placed in Germany and chose the product with 
most orders.

```sql
SELECT od.ProductID, p.ProductName,SUM(Quantity) AS total
FROM OrderDetails od
JOIN Orders o 
ON od.OrderID = o.OrderID
JOIN Products p 
ON od.ProductID = p.ProductID
JOIN Customers c
ON c.CustomerID = o.CustomerID
WHERE Country = "Germany" 
GROUP BY od.ProductID
ORDER BY total DESC
LIMIT 1;
```
