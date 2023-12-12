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
