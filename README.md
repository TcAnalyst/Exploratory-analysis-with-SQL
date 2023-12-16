# Analyzing Adidas sales with SQL

### Introduction

Adidas sales dataset is a collection of data that includes information on the sales of Adidas products. This project is aimed at analyzing the adidas sales data to draw insights, extracting observations, and make decisions that could lead to the growth of the business.


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

#### Data source: [Download here](https://www.kaggle.com/datasets/heemalichaudhari/adidas-sales-dataset/data)

### Tool used: MS SQL SERVER


### Data importation and cleaning

Data importation was done with SQL server import and export wizard.

1. Let’s have an overview of the data set:

```sql
select * from Addidas;
```
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
0 rows returned, which implies that there are no duplicate rows.

### Data exploration and analysis
#### sales analysis;

1. what are the products sold and how many are they?
```sql
select distinct product
from addidas;

select count(distinct product) ProductCount
from addidas;
```
Men’s Apparel, Women’s Apparel, Women’s Street Footwear, Men’s Athletic Footwear, Women’s Athletic Footwear and Men’s Street Footwear are the products sold. They are 6 products in number.

2. who are the retailers ?
```sql
select distinct retailer
from addidas;
```

the query above returns the retailers involved in the selling of these products.

3. what is the best and least selling product?
```sql
select product,sum(units_sold) UnitsSold
from addidas
group by product
order by UnitsSold desc;
```
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
In 2021, $717,821,450 was realised from sales , a 75% increase from the previous year.



#### profit analysis;

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
The query returns each retailer and their most profitable sales method. For Amazon, the online purchase gives the most profit. Footlocker and West gear make their most profits from In-store purchase while Khol’s, Sports direct, Walmart generate their most profits from Outlet purchase.

5. which cities generate most and least profit?
```sql
select top 5 city,sum(operating_profit) profit
from addidas
group by city
order by profit desc;
```
```sql
select top 5  city,sum(operating_profit) profit
from addidas
group by city
order by profit;
```
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
Highest profit of $34.5m was made in the month of August with July not being far of with a profit of $34m. December and September also recorded good profit of $31.5 and $31m respectively. March generated the least profit of $20.4m.

#### FINDINGS ;

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

3. The men’s category generated more sales than the female category of the products. The query below shows that.
```sql
select 
   substring(product,1,charindex(' ', product + ' ') - 1) category,
   sum(Total_Sales) sales
from Addidas
   group by substring(product,1,charindex(' ', product + ' ') - 1)
   order by 2 desc;
```

#### RECOMMENDATIONS

1. Make available other method of purchase in all cities to attract customers who would prefer other methods.
2. Consider the economic conditions of each city especially the less profitable cities like Omaha. Economic downturns or low consumer confidence can lead to reduced selective spending on non-essential items like sportswear.
3. The demographics of the population in each city can be investigated. Age groups, income levels, and lifestyle preferences can influence sportswear purchasing behavior.
4. Lesser production of sport wears from December through the first quarter of a new year is recommended because this period is when the United States is the coldest.

#### LIMITATION ;

Demographics like Age groups, income levels, method of payment etc. would have aided in uncovering more insights.
