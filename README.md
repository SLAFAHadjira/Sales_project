# Sales analysis performance 
End-to-End Data Pipeline
## 📋 Project Overview 
This project focuses exclusively on sales data and analyzes performance metrics to identify trends, top-performing products, revenue contribution, and profitability. The goal is to support data-driven business decisions through SQL analysis and interactive dashboards.

 ## Business Objectives :
- Analyze Overall Performance : Evaluate comprehensive sales and profit performance across all business operations.

- Year-over-Year Analysis : Compare sales and profit differences between the current year and previous year to measure growth trends.
- Calculate Total Quantity Sold  and analyze Product Distribution to examine how products are distributed by quantity sold.
- Create Product Segmentation to categorize based on quantity sold to get the most top product sold 
- Identify Top Performers

## Data set description:
The data contains 3 tables : 
1. Sales : Transaction data
                  -sales_id , Date , unit_price , quantity 
2. Products : all details about products
                 - product_id , product_name , coast , supplier 
3. Inventory : information about store and locations
   ### Refer to this image for a detailed description of all columns, rows, and data models.
   image link 


## 🛠️ Tech Stack
### Database : postgreSQL (Data Cleaning, Transformation, and Aggregation) , Exploratory 

### Visualization: Tableau (Interactive Dashboarding)

### Documentation: GitHub (Version Control) & drow io 

## Phase 1 : 
use SQL 
* Data integration
* Data Modeling 
* Data cleaning


- Create Database
  ``` 
  CREATE DATABASE sales_project ;
  ```
  ###- create silver layer
  ```
  create schema silver ;
  ```

   ```
      CREATE TABLE silver.sales (
	Sales_Id	integer ,
	Store_Id	integer ,
	Product_Id	integer ,
	Date		date ,
	Unit_Price	decimal,	
	Quantity	integer 
  )

  copy  silver.sales (
	Sales_Id	,
	Store_Id	,
	Product_Id	,
	Date		,
	Unit_Price	,	
	Quantity
  )
  FROM 'C:\Users\dell\Documents\Sales project for portfolio\Sales.csv'
  DELIMITER ';'
  CSV HEADER ;

   
  CREATE  TABLE silver.product (
   Product_Id		integer,
		Product_Name	varchar,
		Supplier		varchar,
		Product_Cost	decimal 	 
  )

  COPY silver.product (
	  Product_Id ,
	  Product_Name,	
	  Supplier	,
	  Product_Cost )
 
  FROM 'C:\Users\dell\Documents\Sales project for portfolio\Products.csv'
  DELIMITER ';'
  CSV HEADER ;


  CREATE TABLE silver.inventory (
			Product_Id				integer,
			Store_Id			 	integer,	
			Store_Name			 	varchar,
			Address					varchar,
			neighborhood 			varchar ,	
			Quantity_Available		integer

  )

  COPY silver.inventory (
		Product_Id	,
		Store_Id	,
		Store_Name	,
		Address,
		neighborhood,	
		Quantity_Available)
  
  FROM 'C:\Users\dell\Documents\Sales project for portfolio\Inventory.csv'
  DELIMITER ';'
  CSV HEADER ;
  ```

  - check data quality 
    1. Product table 
    
    ```
          /** Exploring table product 
      **/

    -- Explory dimension 'Distinct'
	      SELECT DISTINCT (product_id) 
     	FROM silver.product;
	
	SELECT DISTINCT( product_name)
	FROM silver.product ;
    -- Count different between product_id and product_name 
	SELECT COUNT( DISTINCT (product_id)) - COUNT( DISTINCT( product_name))
	FROM silver.product ;

    --check if can have same product name ,same price and supplier 
	SELECT * 
	FROM (
			SELECT
					DISTINCT( product_id) , product_name,product_cost,
					ROW_NUMBER () OVER (partition by product_name) AS rank_product_name ,
					supplier,
					ROW_NUMBER () OVER (partition by product_name,supplier) AS Supp_rank 
			FROM silver.product 
			ORDER BY product_name) h 
	WHERE rank_product_name >1	and Supp_rank != 1 
	order by product_name  ;

    -- All products have same suppliers but different cost 
	SELECT  * FROM silver.product
	WHERE product_name = 'Eggplant Oriental';

	SELECT  * FROM silver.product
	WHERE product_name = 'Milk - Buttermilk';


	SELECT  * FROM silver.product
	where product_name = 'Oil - Sunflower';

	SELECT  * FROM silver.product
	WHERE product_name = 'Sambuca Cream';

	SELECT  * FROM silver.product
	WHERE product_name = 'Wine - Montecillo Rioja Crianza';

    --this one have same things same supplier same cost 
	   SELECT  * FROM silver.product
	   WHERE product_name = 'Muffin - Bran Ind Wrpd';

    --2 diffrent supplier and 2 different cost 
	    SELECT  * FROM silver.product
	    WHERE product_name = 'Squash - Pattypan, Yellow';

    --CHECK NULL's 
    SELECT *
    FROM silver.product 
    WHERE product_id IS NULL
    OR product_name IS NULL
    OR supplier IS NULL
    OR product_cost IS NULL ;

     ```
 2. Inventory table
    --Exploring table inventory

```
select * from silver.inventory ;

select count(*) from silver.inventory ;

SELECT DISTINCT store_id
FROM silver.inventory ;

SELECT DISTINCT store_name 
FROM silver.inventory ;

SELECT DISTINCT address 
FROM silver.inventory ;


select * from silver.inventory 
WHERE  store_id IS NULL 
OR product_id IS NULL 
OR store_name IS NULL
OR address IS NULL
OR neighborhood IS NULL
OR quantity_available IS NULL
;

```
3. Sales table
```
select * from INFORMATION_SCHEMA.COLUMNS 
where TABLE_NAME = 'sales';

SELECT 
	count(sales_id)-count(DISTINCT sales_id) 
	
FROM silver.sales 	;

select 
* 
FROM (
	SELECT 
	sales_id,
	row_number () over (partition  by sales_id) as rank 
	FROM silver.sales )
where rank >1 ;


select 
	*
FROM silver.sales 
WHERE sales_id = 38329;

SELECT 
count(sale_row_id)
FROM (
SELECT  
	  ROW_NUMBER() OVER (ORDER BY date, product_id) AS sale_row_id,
	*

FROM silver.sales );

select *
from (
       select 
product_id,
count(product_name) over (partition by product_id) as rank
from silver.product
)
where rank >1

select * from silver.product 
where product_name = 'Muffin - Banana Nut Individual'


select 
	product_name,
	COUNT(product_id)
from silver.product 
group by product_name
order by COUNT(product_id) desc ;
```

Create gold layer 
```
--Create a gold layer schema to make the table analysis-ready
CREATE SCHEMA Gold ;

CREATE TABLE Gold.Store_name (
		store_id integer ,
		store_name varchar 
);

create table gold.sales (
	New_Sales_Id	integer ,
	Store_Id	integer ,
	Store_name  varchar,
	Product_Id	integer ,
	Product_name varchar,
	Supplier		varchar,
	Product_Cost	decimal ,
	Date		date ,
	Unit_Price	decimal,	
	Quantity	integer 
);


--Insert data into new table 

INSERT INTO Gold.Store_name (
		store_id ,
		store_name 
)
SELECT 
	DISTINCT(store_id) as store_id ,
	(store_name)as store_name 
FROM silver.inventory 
WHERE store_name IS NOT NULL ;

--Insert sales table 'our fact table '
INSERT INTO gold.sales (
	New_Sales_Id	 ,
	Store_Id	 ,
	Store_name  ,
	Product_Id	 ,
	Product_name ,
	Supplier		,
	Product_Cost ,
	Date		 ,
	Unit_Price	,	
	Quantity	 
)

SELECT
	Row_number () over (order by s.date,s.sales_id)as New_Sales_Id	,
	s.Store_Id,
	st.Store_name,
	s.Product_Id,
	p.Product_name ,
	p.Supplier		,
	p.Product_Cost	 ,
	s.Date		 ,
	s.Unit_Price	,	
	s.Quantity	
	
FROM silver.sales  s
LEFT JOIN gold.store_name st 
ON s.store_id = st.store_id
LEFT JOIN silver.product p 
ON s.product_id = p.product_id ;
```

