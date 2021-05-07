# Fall-2021-DS-Challenge

## Question 1
(The answers have code snippets to show the code that generated my outputs and plots. For the full code, 
please check out the jupyter notebook that I have also uploaded in this repository.)

### (a) Think about what could be going wrong with our calculation. Think about a better way to evaluate this data. 

The high AOV of \$3,145.13 comes from naively calculating the mean of the order_amount of a highly right-skewed data without examining outliers such as bulk orders and stores that sell more expensive brands. The code snippets and outputs below describe my data exploration process.

From the summary statistics, we can see that AOV is calculated by using the mean of order_amount. We also see that we have outliers because the third quartile is 390 while the maximum is 704000. Therefore, I decided to further investigate the overall distribution of order amounts.
```python
df.order_amount.describe()
```
![Screen Shot 2021-05-07 at 4 11 12 PM](https://user-images.githubusercontent.com/54642556/117516780-dae9d700-af4e-11eb-9f65-61d3601d5e0c.png)

From both the histogram and the boxplot below, we can see that we have many outliers and the data is heavily right-skewed. Let's further explore why we are getting such large outliers because they are causing AOV to be high.

![Screen Shot 2021-05-07 at 4 11 48 PM](https://user-images.githubusercontent.com/54642556/117516797-f05f0100-af4e-11eb-97f0-de24d1ac0956.png)
![Screen Shot 2021-05-07 at 4 12 01 PM](https://user-images.githubusercontent.com/54642556/117516803-f7860f00-af4e-11eb-8447-e11e66fe3ecb.png)

First I made a table of outliers by filtering the dataset by order_amounts that are larger than Q3+1.5*IQR. From the first table, we can see that the maximum order amount of $704000 is coming from user #607 making bulk orders of 2000. This result made me wonder if there were other users making bulk orders, so I made another table summarizing total items a user has purchased at a shop. Fortunately, we see from the second table that no other user is making bulk orders from the same shop. 
```python
Q1 = df.order_amount.quantile(q=0.25)
Q2 = df.order_amount.quantile(q=0.5)
Q3 = df.order_amount.quantile(q=0.75)
IQR = Q3 - Q1
outliers = df[df.order_amount>=Q3+1.5*IQR]
outliers.sort_values(by='order_amount',ascending=False).head(20)
```
![Screen Shot 2021-05-07 at 4 17 22 PM](https://user-images.githubusercontent.com/54642556/117517072-b6422f00-af4f-11eb-8b72-cb8ec6eca74d.png)

```python
bulk=outliers.groupby(['user_id','shop_id','total_items']).size().reset_index(name='order_count')
bulk.sort_values(by='total_items',ascending=False).head()
```
![Screen Shot 2021-05-07 at 4 24 12 PM](https://user-images.githubusercontent.com/54642556/117517360-aa0aa180-af50-11eb-8319-f5a4eee5476a.png)

From there, I decided to investigate where other outliers are coming from by calculating the price per shoes for each shop. The table below shows that shop ID 78 sells a shoe model that is \$25,725. It is likely that the shop sells a high-end shoe model. In conclusion, the two main sources of outliers that are driving up the AOV are users placing bulk orders and stores selling expensive shoes. 

```python
outliers['price'] = outliers.order_amount/outliers.total_items
prices=outliers.groupby(['shop_id','price']).size().reset_index(name='order_count')
prices.sort_values(by='price',ascending=False).head()
```
![Screen Shot 2021-05-07 at 4 27 38 PM](https://user-images.githubusercontent.com/54642556/117517539-256c5300-af51-11eb-8995-0633cc5bed25.png)





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
