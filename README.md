# **Analyzing Adidas sales with SQL**

#### Table of content
- [Introduction](#introduction)
- [Data Description](#data-description)
- [Data source](#data-source)
- [Data importation and cleaning](#data-importation-and-cleaning)
- [Data exploration and analysis](#data-exploration-and-analysis) 
- [FINDINGS](#findings)
- [RECOMMENDATIONS](#recommendations)
- [LIMITATION](#limitation) 

### Introduction
---

Adidas sales dataset is a collection of data that includes information on the sales of Adidas products. This project is aimed at analyzing the adidas sales data to draw insights, extracting observations, and make decisions that could lead to the growth of the business.
![adidas 1](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/a8e6d2de-de44-4db2-a4a8-a164028d2073)


### Data Description

- Retailer: The entity or organization selling Adidas products
- Retailer ID: A unique identifier for each retailer
- Invoice Date: The date when the sales transaction occurred
- Region: The geographical region where the retailer operates
- State: The state within the region where the retailer is located
- City: The city where the retailer is situated
- Product: The Adidas product being sold
- Price per Unit: The cost of one unit of the Adidas product
- Units Sold: The number of units of the Adidas product sold in a particular transaction
- Total Sales: The total revenue generated from the sale of Adidas products in a transaction
- Operating Profit: The profit earned by the retailer from the sale after deducting operating costs
- Operating Margin: The percentage of operating profit in relation to total sales
- Sales Method: The method or channel through which the sales transaction occurred

### Data source
[Download here](https://www.kaggle.com/datasets/heemalichaudhari/adidas-sales-dataset/data)

### Tool used: MS SQL SERVER


### Data importation and cleaning

Data importation was done with SQL server import and export wizard.

1. Let’s have an overview of the data set:

```sql
select * from Addidas;
```
![adidas overview](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/e13fd4c5-d531-4e21-b878-c94fb3534a94)

this data set has 9648 rows and 14 columns.


2. To change the “invoice date” column to date type:
```sql
alter table addidas
alter column invoice_date date;
```
Checking for duplicate rows:
```sql
select *
from addidas
group by retailer, retailer_id,invoice_date, region,state,city,product,price_per_unit,
  units_sold,total_sales,operating_profit,operating_margin,sales_method
having count(*) > 1;
```
![adidas checking duplicate rows](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/c6d17a8d-5fa6-4464-b6c3-15ebdbbbe841)

0 rows returned, which implies that there are no duplicate rows.

### Data exploration and analysis
#### *sales analysis;*

1. what are the products sold and how many are they?
```sql
select distinct product
from addidas;

select count(distinct product) ProductCount
from addidas;
```
![adidas product sold and count](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/83e52512-1f48-4c11-bd40-0a85c454cd1f)

Men’s Apparel, Women’s Apparel, Women’s Street Footwear, Men’s Athletic Footwear, Women’s Athletic Footwear and Men’s Street Footwear are the products sold. They are 6 products in number.

2. who are the retailers ?
```sql
select distinct retailer
from addidas;
```
![adidas retailers](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/2b20a99b-71c4-48c1-a88e-0121fb7da367)

the query above returns the retailers involved in the selling of these products.

3. what is the best and least selling product?
```sql
select product,sum(units_sold) UnitsSold
from addidas
group by product
order by UnitsSold desc;
```
![adidas best selling product](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/dd5f53ef-060d-4b7b-844d-1c43b351153d)

The best selling product is the Men’s Street Footwear with 593320 units sales, Men’s Athletic Footwear comes second with 435526 units sales. The least selling product is the Men’s Apparel with 306683 units sales.

4. Is the best selling product the same for all retailers?
```sql
with HighestSales as 
 (select retailer, product, sum(total_sales) sales
from addidas
group by retailer, product),
RankedRetailer as 
 (select retailer, product,sales,row_number() over (partition BY retailer order BY sales desc) as RN
 from HighestSales)
select retailer,product,sales
 from RankedRetailer
 where RN=1;
```
![adidas best selling product 2](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/e4ab85e5-31f5-468b-9cce-d6e04caa719f)

The men’s street footwear was the best selling product for all retailers except for Walmart where the women’s Apparel is the best selling product.

5. what was the total sales for each year?
```sql
with YearlySales as 
 (select year(invoice_date) as year, sum(Total_Sales) TotalSales
 from addidas
 group by invoice_date)
select year,sum(TotalSales) YearlySales
 from YearlySales
 group by year
 order by 2 desc;
```
![adidas total sales per year](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/c1de43db-041f-45c5-bbd2-aa3e2716eb65)

In 2021, $717,821,450 was realised from sales , a 75% increase from the previous year.


#### *profit analysis;*

1. What was the total profit generated in general?
```sql
select sum(operating_profit)
from addidas;
```
the total profit made in both years was $332,134,761.

2. What is the most and least profitable product
```sql
with ProductProfit as (select product,sum(operating_profit) profit
from addidas
group by product
)
select product, profit, profit / SUM(profit) OVER () * 100 AS profitPercentage
from ProductProfit
group by product, profit
order by 2 desc;
```
![adidas most profitable product](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/397ac951-7efb-4d8c-bef1-041e60b20e63)

Just as in the best selling category, the Men’s street footwear is the most profitable product with $82.8m profit realized in both years, which accounts for almost 25% of total profit. Quite on like in the best selling category where the women’s athletic footwear is the second highest selling product, it is the least profitable product with about $39m (11.7%) profit realized.


3. what is the profit spread across different sales method?
```sql
with methodprofit as (select sales_method,sum(operating_profit) profit
from addidas
group by sales_method)
select sales_method, profit, profit/sum(profit) over()*100 ProfitPercentage
from methodprofit
group by sales_method, profit
order by 2 desc;
```
![adidas profit across sales method](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/7aa8e7c6-f747-4c1f-a3f5-a83e2e555c03)

The in-store purchase method accounted for the highest profit , outlet, the second most profitable while the online purchase accounts for the least, with $127.6m, $108m and $96.6m profits respectively.

4. Is the in-store purchase the most profitable method for all retailers?
```sql
with GeneralProfit as 
 (select retailer, Sales_Method, sum(operating_profit) Profit
from addidas
group by retailer, Sales_Method),
RankedRetailer as 
 (select retailer, Sales_Method,Profit,ROW_NUMBER() OVER (PARTITION BY retailer ORDER BY Profit DESC) AS RN
 from GeneralProfit)
select retailer,Sales_Method,profit
 from RankedRetailer
 where RN=1;
```
![adidas instore purchase profit](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/0317d60d-d508-4be5-9fb6-a14881ede69f)

The query returns each retailer and their most profitable sales method. For Amazon, the online purchase gives the most profit. Footlocker and West gear make their most profits from In-store purchase while Khol’s, Sports direct, Walmart generate their most profits from Outlet purchase.

5. which cities generate most and least profit?
```sql
select top 5 city,sum(operating_profit) profit
from addidas
group by city
order by profit desc;
```
![adidas city profits](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/1a991b2f-ac45-4a40-838b-4665cf7e20be)

```sql
select top 5  city,sum(operating_profit) profit
from addidas
group by city
order by profit;
```
![adidas city least profits](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/ed7fdeac-92d6-40ab-ac47-90b57aa9691f)

The city of Charleston generated most profit with $15.6m profit, New york city generated the second highest profit with $13.9m. Miami, Portland and San Francisco made complete the top 5, accounting for $12.2m, $10.8m, $10.3m respectively. Omaha generated the least profit of $2.4m.

6. what are the best performing months in terms of profit generated?
```sql
with monthlyProfit as
 (select format(invoice_date,'MMMM') AS Month, SUM(operating_profit) profit
 from Addidas
 group by invoice_date)
select month, sum(profit) MonthlyProfit
 from MonthlyProfit
 group by month
 order by 2 desc;
```
![adidas best performing months](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/e30e1073-f3fb-40cc-927f-c945676d4fc7)

Highest profit of $34.5m was made in the month of August with July not being far of with a profit of $34m. December and September also recorded good profit of $31.5 and $31m respectively. March generated the least profit of $20.4m.

#### FINDINGS

1. most cities with low profit, don't have all the purchasing method available to customers. The query below returns the sales method for each of the cities in the bottom list of profits generated.
```sql
SELECT city, sales_method
FROM 
(
    SELECT City,  Sales_Method
    FROM addidas
    GROUP BY City, Sales_Method
) as tt
where city='omaha' or city='Des Moines' or city='Minneapolis' or city='Fargo' or city='Baltimore'
ORDER BY city;
```
![adidas findings 1](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/63cfe0e0-8837-4542-998f-79eede3a8d88)

2. Most sales and profit were made in the 3rd quarter of the year and the lowest, in the first quarter. This could be as a result of the first quarter being the coldest quarter of the year, customers will mainly go for thicker wears during this period which in turn reduces sales and profit. The query below proves that.

-- calculating profit by quarter
```sql
select quarter, sum(operating_profit) Quarterlyprofit
from Addidas
group by quarter
order by 2 desc;
```
-- calculating sales by quarter
```sql
select quarter, sum(total_sales) QuarterlySales
from Addidas
group by quarter
order by 2 desc;
```
![adidas findings 2](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/cfa06c9d-0e92-4d5f-991c-8becd4876535)


3. The men’s category generated more sales than the female category of the products. The query below shows that.
```sql
select 
   substring(product,1,charindex(' ', product + ' ') - 1) category,
   sum(Total_Sales) sales
from Addidas
   group by substring(product,1,charindex(' ', product + ' ') - 1)
   order by 2 desc;
```
![adidas findings 3](https://github.com/TcAnalyst/ProjectsPortfolio/assets/142181097/0f4d8711-de3f-4de1-aac6-0ef54dee11de)


#### RECOMMENDATIONS

1. Make available other method of purchase in all cities to attract customers who would prefer other methods.
2. Consider the economic conditions of each city especially the less profitable cities like Omaha. Economic downturns or low consumer confidence can lead to reduced selective spending on non-essential items like sportswear.
3. The demographics of the population in each city can be investigated. Age groups, income levels, and lifestyle preferences can influence sportswear purchasing behavior.
4. Lesser production of sport wears from December through the first quarter of a new year is recommended because this period is when the United States is the coldest.

#### LIMITATION

Demographics like Age groups, income levels, method of payment etc. would have aided in uncovering more insights.


#### THANK YOU! 🥰  
