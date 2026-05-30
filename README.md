# Maven-Market

## 1. Project Title / Headline
An end-to-end retail intelligence project for Maven Market, a global grocery chain. This project leverages relational database modeling and advanced SQL to optimize inventory, improve customer segmentation, and track regional profitability.

## 2. Short Description / Purpose
The Maven Market Analysis project transforms raw transactional data into actionable business insights. By integrating sales, customer, and return data, this dashboard helps stakeholders identify high-value customer segments, calculate product-level return rates, and evaluate store performance across international regions.

## 3. Tech Stack
This project leverages the following technologies:
- **Power BI Desktop** - Interactive dashboarding and time-intelligence reporting
- **DAX (Data Analysis Expressions)** - Development of business logic for measures such as Total Revenue, Total Profit, Quantities Sold, Return Rate and Quantities returned	
- **Data Modeling** - Implementation of a complex Star Schema with bridges between Fact and Dimension tables.
- **Power Query** - Data transformation and ETL processes
- **PostgreSQL**  - Data cleansing, complex multi-table joins, window functions for ranking, and performance metrics calculation.
- **File Types** – `.pbix`, `.sql`,`.csv`

## 4. Data Source
**Source**: Maven Analytics (Maven Market)
**Location**: South America (USA, Canada, Mexico)
**Key Fields**: 
- transactions and returns: Fact tables capturing operational data.
- customers: Demographic data (Income, Education, Occupation).
- products: Hierarchical data (Brands, Pricing, Fat content).
- stores and regions: Geographic and structural store details.
- calender: A specialized time-table used for solving business questions

## 5. Features / Highlights
### • Business Problem
Maven Market needed to bridge the gap between transactional records and strategic decision-making. Key pain points included identifying "lost" customers (1997 vs 1998) and understanding the impact of returns on net profitability.

### • Goal of the Dashboard
to provide a "Single Source of Truth" for retail operations and management. It aims to:
- Monitor Profitability: Move beyond just "Revenue" to understand "Net Profit" by accounting for product costs.
- Optimize Inventory & Logistics: Use return rate analysis to identify poor-quality products and manage stock levels efficiently.
- Segment Customer Behavior: Translate demographic data (income, occupation, education) into actionable marketing insights.
- Evaluate Regional Efficiency: Benchmark store performance by district and region to identify under-performing areas and successful retail models.
  
### • Key Analytics and Metrics
**Business KPIs**
- Total Revenue: Total Revenue generated
- Net Profit: Total profit earned after covering product cost
- Quantity Sold - Total quantites sold for a perticular product
- Total Returns: Products which where returned 
- Return Rate - The rate of product returning  

**Customer Insights**
- Total Unique Customers: Count of distinct customer IDs.
- Revenue per Customer: Average spend per customer across the dataset.
- Customer Segmentation: Analysis based on Yearly Income (e.g., Entry vs. Executive) and Member Card status (Bronze, Normal, Golden).
- Churn/Retention: Tracking customer activity between years (e.g., the 1997 vs. 1998 cohort analysis you performed in SQL).

**Product & Brand Performance**
- Hero Products/Brands: Top 5 brands generating the highest profit.
- Low-Fat & Recyclable Performance: Analysis of product attributes to see if health/eco-conscious products drive higher revenue.
- Return-Heavy Products: Products with a return rate above the company average (identifying inventory/quality issues).

### Business Impact & Insights
- Customer Retention: Identified a cohort of 1997 shoppers who did not return in 1998, enabling targeted re-engagement marketing.
- Inventory Quality Control: Pinpointed items with disproportionate return rates, providing data for supply-chain review.
- Pricing Strategy: Enabled a clear view of brand-level profitability, allowing management to prioritize high-margin products.
- Regional Growth: Provided a granular view of performance across the USA, Canada, and Mexico, supporting better-informed regional expansion.

## 6. SQL Logic & Insights

1. Write a query to retrieve the full names of all customers who live in Mexico.
```sql
select 
	concat(first_name,' ',last_name) as full_name,customer_country
from customers
where customer_country = 'Mexico'
```

2. How many stores does Maven Market have in each country?
```sql
select
	store_country,count(*) as no_of_stores
from stores 
group by store_country
```

3. List all product names that are both low fat and recyclable.
```sql
select
	product_name,low_fat,recyclable
from products
where low_fat = 1 and recyclable = 1
```

4. Show all unique sales_region names available in the Regions table
```sql
select 
	distinct sales_region
from regions
```

5. Retrieve all columns from the Transactions table for a specific customer (customer_id = 3449)
```sql
select 
	*
from transactions
where customer_id = 3449
```

6. calculate the total revenue generated by each product_brand
```sql
select
	product_brand,
	sum(quantity * product_retail_price) as total
from transactions t
join products p on t.product_id = p.product_id 
group by product_brand
order by total desc
```

7. find the top 5 customers (name and total quantity purchased) who have bought the most items across all transactions.
```sql
select 
	c.customer_id,first_name,last_name,
	sum(quantity) as total_items_purchased
from customers c
join transactions t on c.customer_id = t.customer_id
group by c.customer_id,first_name,last_name
order by total_items_purchased desc
limit 5; 
```

8. Join Transactions, Stores, and Regions to find the total quantity of items sold in the 'North West' sales region.
```sql
select
	sales_region,
	sum(quantity) as total_quantity
from transactions t 
join stores s on t.store_id = s.store_id
join regions r on s.region_id = r.region_id
where sales_region = 'North West'
group by sales_region;
```

9. Count the total number of returns for each product_id, but only include products that have been returned more than 5 times.
```sql
SELECT
	r.product_id, p.product_name,
	count(*) as times_returned
FROM returns r
join products p on r.product_id = p.product_id
group by r.product_id, p.product_name
having count(*) > 5
order by times_returned desc
```

10. Calculate the Total Profit (Total Revenue - Total Cost) for every month in 1998.
```sql
select
	extract (month from transaction_date) as month_number,
	to_char(date,'Month') as month_name,
	ROUND(SUM(quantity * product_retail_price), 2) as total_revenue,
    ROUND(SUM(quantity * product_cost), 2) as total_cost,
    ROUND(SUM(quantity * (product_retail_price - product_cost)), 2) AS total_profit
from products p
join transactions t on p.product_id = t.product_id
join calender c on t.transaction_date = c.date
where extract (year from transaction_date) = 1998
group by month_number,month_name
order by month_number
```

11. For every product, calculate its Return Rate. Formula: (Total Quantity Returned / Total Quantity Sold) * 100.
```sql
SELECT
	r.product_id,
	sum(t.quantity) as total_sales,
	sum(r.quantity) as total_returns,
	round(((sum(r.quantity) / sum(t.quantity))*100),2) as return_rate
FROM returns r
join transactions t on r.product_id = t.product_id
group by r.product_id
order by return_rate desc
```

12. . Count how many customers fall into each category of income level order by income level low to high.
```sql
SELECT 
	yearly_income,
	count(*) as no_of_customers
FROM customers
group by yearly_income
order by 
	case yearly_income
		when '$10K - $30K' then 1
		when '$30K - $50K' then 2
		when '$50K - $70K' then 3
		when '$70K - $90K' then 4
		when '$90K - $110K' then 5
		when '$110K - $130K' then 6
		when '$130K - $150K' then 7
		when '$150K +' then 8 
	end
```

13. Rank the sales_district within each sales_region based on their total sales quantity using a Window Function
```sql
select
	sales_region,
	sales_district,
	sum(quantity) as total_quantity,
	row_number() over(partition by sales_region order by sum(quantity) desc) as rn
from transactions t
join stores s on t.store_id = s.store_id
join regions r on s.region_id = r.region_id
group by sales_region,sales_district
```

14. Find customers who made a purchase in 1997 but have zero transactions recorded in 1998
```sql
SELECT 
    c.customer_id, 
    c.first_name, 
    c.last_name,
    c.customer_city,
    c.customer_country
FROM customers c
WHERE 
    c.customer_id IN (
        SELECT customer_id 
        FROM Transactions 
        WHERE transaction_date BETWEEN '1997-01-01' AND '1997-12-31'
    )
    AND c.customer_id NOT IN (
        SELECT customer_id 
        FROM Transactions 
        WHERE transaction_date BETWEEN '1998-01-01' AND '1998-12-31'
    )
ORDER BY c.customer_id;
```

## 7. Screenshots / Demos
![https://github.com/l2Aquel/Healthcare-Data-Analysis/blob/main/Dashboard_preview.png](Dashboard_preview.png)
![https://github.com/l2Aquel/Healthcare-Data-Analysis/blob/main/Data_Model.png](Data_Model.png)
![https://github.com/l2Aquel/Healthcare-Data-Analysis/blob/main/PostgreSQL.png](PostgreSQL.png)
    

