What issues will you address by cleaning the data?

- Make sure all data have been imported
- Data type format
- Dealing with NULL/missing data
- Drop unwanted columns and focus on relative data only
- TRIM space on string values
- Assign Primary and forigen keys
- Make sure the data make sense (ex quantity can't be negative)




Queries:
Below, provide the SQL queries you used to clean your data.

-- ##### Import the data from CSV file ##### --

DROP TABLE IF EXISTS analytics; 
CREATE TABLE analytics (
            visitnumber int,
            visitid int,
            visitstarttime int,
            date date,
            fullvisitorid Varchar,
            userid int,
            channelgrouping varchar,
            socialengagementtype Varchar,
            units_sold int,
            pageviews int,
            timeonsite Varchar,
            bounces int,
            revenue bigint,
            unitprice int
);
DROP TABLE IF EXISTS all_sessions;
CREATE TABLE all_sessions( 
            fullVisitorId Varchar,
             channelgrouping Varchar,
              time Varchar,
             country Varchar,
             city Varchar,
             totaltransactionrevenue Varchar,
             transactions int,
             timeonsite int,
             pageviews int,
             sessionqualitydim int,
             date date,
             visitid int,
             type Varchar,
             productrefundamt Varchar,
             productquantity Varchar,
             productprice int,
             productrevenue Varchar,
             productSKU Varchar,
             v2productname Varchar,
             v2productcategory Varchar,
             productvariant Varchar,
             currencycode Varchar,
             itemquantity int,
             itemrevenue int,
             transactionrevenue int,
             transactionid Varchar,
             pagetitle Varchar,
             searchkeyword varchar,
             pagepathlevel1 Varchar,
            ecommerceaction_type int,
             ecommerceaction_step int,
             ecommerceaction_option Varchar
);

DROP TABLE IF EXISTS products;
CREATE TABLE products(
            SKU Varchar,
            name Varchar,
            orderedQuantity int,
            stockLevel int,
            restockingLeadTime int,
            sentimentScore float,
            sentimentMagnitude float
);

DROP TABLE IF EXISTS sales_by_sku;
CREATE TABLE sales_by_sku(
            productSKU Varchar,
            total_ordered int
);


DROP TABLE IF EXISTS sales_report;
CREATE TABLE sales_report(
            productSKU Varchar,
            total_ordered int,
            name Varchar,
            stockLevel int,
            restockingLeadTime int,
            sentimentScore float,
            sentimentMagnitude float,
            ratio float
);


-- ##### All_sessions Table ##### --

SELECT * FROM all_sessions limit 10 -- Overview of the data
SELECT COUNT(*) FROM all_sessions -- Number of rows 15,134


UPDATE all_sessions  -- make sure no space at the beginning or end
SET fullvisitorid = TRIM(fullvisitorid),
	channelgrouping = TRIM(channelgrouping), 
	time = TRIM(time),
	country = TRIM(country),
	city = TRIM(city),
	totaltransactionrevenue = TRIM(totaltransactionrevenue),
	type = TRIM(type),
	productquantity = TRIM(productquantity),
	productrevenue = TRIM(productrevenue),
	productsku = TRIM(productsku),
	v2productname = TRIM(v2productname),
	v2productcategory = TRIM(v2productcategory),
	productvariant = TRIM(productvariant),
	currencycode = TRIM(currencycode),
	transactionid = TRIM(transactionid),
	pagetitle = TRIM(pagetitle),
	pagepathlevel1 = TRIM(pagepathlevel1),
	ecommerceaction_option = TRIM(ecommerceaction_option) 
	
 


ALTER TABLE all_sessions  
ALTER time TYPE float
USING "time"::double precision -- Change time column data type to float

SELECT COUNT(*) FROM all_sessions 
WHERE productsku IS NULL -- make sure no NULL value in productsku column

SELECT COUNT(*) FROM all_sessions 
WHERE v2productname IS NULL -- make sure no NULL value in v2productname column



SELECT COUNT(*) FROM all_sessions 
WHERE totaltransactionrevenue IS NULL -- check NULL values in totaltransactionrevenue column, 15,053 rows

UPDATE all_sessions  
SET totaltransactionrevenue = 0
WHERE totaltransactionrevenue IS NULL -- Set NULL value to 0

ALTER TABLE all_sessions  
ALTER totaltransactionrevenue TYPE float
USING totaltransactionrevenue::double precision -- change column type to float

UPDATE all_sessions  
SET totaltransactionrevenue = totaltransactionrevenue / 1000000 
WHERE totaltransactionrevenue <> 0 -- divid totaltransactionrevenue by 1,000,000

SELECT * FROM all_sessions
WHERE totaltransactionrevenue <> 0 -- 81 rows


SELECT COUNT(*) FROM all_sessions 
WHERE transactions IS NULL -- check NULL values in transactions column, 15,053 rows

SELECT * FROM all_sessions 
WHERE transactions IS NOT NULL -- number of row with transcation made 81

UPDATE all_sessions  
SET transactions = 0
WHERE transactions IS NULL -- Set NULL value to 0



SELECT COUNT(*) FROM all_sessions 
WHERE productquantity IS NULL -- check NULL values in productquantity column, 15,081 rows

UPDATE all_sessions  
SET productquantity = 0
WHERE productquantity IS NULL -- Set NULL value to 0

ALTER TABLE all_sessions  
ALTER productquantity TYPE int
USING productquantity::integer -- change column type to integer



SELECT COUNT(*) FROM all_sessions 
WHERE productrevenue IS NULL -- check NULL values in orderedquantity column, 15,130 rows

UPDATE all_sessions  
SET productrevenue = 0
WHERE productrevenue IS NULL -- Set NULL value to 0

ALTER TABLE all_sessions  
ALTER productrevenue TYPE float
USING productrevenue::double precision -- change column type to float

UPDATE all_sessions  
SET productrevenue = productrevenue / 1000000 
WHERE productrevenue <> 0 -- divid productrevenue by 1,000,000



SELECT COUNT(*) FROM all_sessions 
WHERE ecommerceaction_option IS NULL -- check NULL values in ecommerceaction_option column, 15,103 rows

SELECT * FROM all_sessions 
WHERE ecommerceaction_option IS NOT NULL -- 1 for Billing and Shipping, 2 for Payment, 3 for Review

SELECT DISTINCT ecommerceaction_step FROM all_sessions -- We have only 3 step (1,2,3)

-- Fill out ecommerceaction_option column based on the data in ecommerceaction_step column
update all_sessions
	SET ecommerceaction_option = 
		CASE
			WHEN ecommerceaction_option IS NOT NULL THEN ecommerceaction_option
			WHEN ecommerceaction_step = 1 THEN 'Billing and Shipping'
			WHEN ecommerceaction_step = 2 THEN 'Payment'
			WHEN ecommerceaction_step = 3 THEN 'Review'
			END 
			



ALTER TABLE all_sessions  
ALTER productprice TYPE float -- Change column type to float

UPDATE all_sessions  
SET productprice = (productprice / 1000000) -- divide productprice by 1,000,000



SELECT COUNT(*) FROM all_sessions 
WHERE transactionrevenue IS NOT NULL -- Only 4 rows contain data

ALTER TABLE all_sessions  
ALTER transactionrevenue TYPE float -- Change column type to float

UPDATE all_sessions  
SET transactionrevenue = 0
WHERE transactionrevenue IS NULL

UPDATE all_sessions  
SET transactionrevenue = transactionrevenue / 1000000
WHERE transactionrevenue <> 0



SELECT DISTINCT country FROM all_sessions
ORDER BY country -- make sure country names are consistence

SELECT COUNT(*) FROM all_sessions 
WHERE country IS NULL OR country like '%not%' -- check NULL values in country column, 24 rows

SELECT * FROM all_sessions
WHERE country like '%not%' -- 24 rows, when country is not set, city is not set as well, and no all revenu data is 0, will delete those rows

DELETE FROM all_sessions
WHERE country IS NULL OR country like '%not%'




SELECT DISTINCT city FROM all_sessions
ORDER BY city -- make sure city names are consistence

SELECT COUNT(*) FROM all_sessions 
WHERE city IS NULL OR city like '%not%' -- check NULL or invlaid values in city column, 8,656 rows

	
SELECT country, city, count(*) FROM all_sessions
group by country, city
order by country -- Missing cities per country

UPDATE all_sessions  
SET city = 'NA'
WHERE city IS NULL OR city like '%not%' -- unified the value

SELECT COUNT(*) FROM all_sessions 
WHERE city = 'NA' -- 8,632 rows



SELECT COUNT(*) FROM all_sessions 
WHERE productrefundamt IS NULL -- check NULL values in productrefundamt column, 15,134 rows (all rows), will drop the column

ALTER TABLE all_sessions DROP productrefundamt -- Drop productrefundamt column


SELECT COUNT(*) FROM all_sessions 
WHERE sessionqualitydim IS NULL -- check NULL values in sessionqualitydim column, 13,906 rows

SELECT * FROM all_sessions 
WHERE sessionqualitydim IS NOT NULL -- 1228 rows, I think this column has no benefit, we can drop it

ALTER TABLE all_sessions DROP sessionqualitydim -- Drop sessionqualitydim column


SELECT COUNT(*) FROM all_sessions 
WHERE transactionid IS NULL -- check NULL values in transactionid column, 15,125 rows. Will drop the column

ALTER TABLE all_sessions DROP transactionid -- Drop transactionid column



SELECT COUNT(*) FROM all_sessions 
WHERE searchkeyword IS NULL -- check NULL values in searchkeyword column, 15,134 rows (all rows), will drop it

ALTER TABLE all_sessions DROP searchkeyword -- Drop searchkeyword column


SELECT DISTINCT currencycode from all_sessions -- Only one currency(USD), there no benefit from this column we can drop it

ALTER TABLE all_sessions DROP currencycode -- Drop currencycode column


SELECT COUNT(*) FROM all_sessions 
WHERE itemquantity IS NULL -- 15,134 all rows are empty, will drop it

SELECT COUNT(*) FROM all_sessions 
WHERE itemrevenue IS NULL -- 15,134 all rows are empty, will drop it

ALTER TABLE all_sessions DROP itemrevenue, DROP itemquantity -- Drop itemrevenue & itemquantity


SELECT productvariant, COUNT(*) FROM all_sessions
GROUP BY productvariant -- 15,094 has missing data, will drop it

ALTER TABLE all_sessions DROP productvariant -- Drop productvariant column


SELECT pagepathlevel1, COUNT(*) FROM all_sessions 
GROUP BY pagepathlevel1 -- This column has no beneift, will drop it

ALTER TABLE all_sessions DROP pagepathlevel1 -- Drop pagepathlevel1 column




-- ##### Analytics Table ##### --

SELECT * FROM analytics limit 10 -- Overview of the data
SELECT COUNT(*) FROM analytics -- Number of rows 4,301,122



UPDATE analytics  
SET fullvisitorid = TRIM(fullvisitorid),
	channelgrouping = TRIM(channelgrouping),
	socialengagementtype = TRIM(socialengagementtype),
	timeonsite = TRIM(timeonsite) -- make sure no space at the beginning or end



ALTER TABLE analytics  
ALTER visitstarttime TYPE TIMESTAMPTZ
USING timezone('UTC', TO_TIMESTAMP(visitstarttime)) -- Change visitstarttime column data type to timestamp
	


SELECT COUNT(*) FROM analytics
WHERE userid is NULL -- No data in userid column, better to drop it

ALTER TABLE analytics DROP userid -- Drop userid column



ALTER TABLE analytics
ALTER unitprice TYPE float -- Change data type to float

SELECT * from analytics
where unitprice = 0 -- 188,314

UPDATE analytics
SET unitprice = unitprice / 1000000
WHERE unitprice <>0 



SELECT COUNT(*) FROM analytics
WHERE units_sold is NULL -- Number of rows 4,205,975

SELECT * FROM analytics
WHERE units_sold is NOT NULL -- 95,147 rows

UPDATE analytics
SET units_sold = 0
WHERE units_sold IS NULL



SELECT COUNT(*) FROM analytics
WHERE bounces is NULL -- Number of rows 3,826,283

SELECT bounces, count(*) FROM analytics
GROUP BY bounces -- This column seems not important we can drop it

ALTER TABLE analytics DROP bounces -- Drop bounces column



SELECT socialengagementtype, count(*) FROM analytics
GROUP BY socialengagementtype -- Contains only one type, not important we can drop it

ALTER TABLE analytics DROP socialengagementtype -- Drop bounces column



SELECT COUNT(*) FROM analytics
WHERE revenue is NULL -- Number of rows 4,285,767

ALTER TABLE analytics
ALTER revenue TYPE float

UPDATE analytics
SET revenue = 0
WHERE revenue IS NULL

UPDATE analytics
SET revenue = revenue / 1000000
WHERE revenue <> 0 



SELECT DISTINCT channelgrouping from analytics -- 8 groups



-- ##### Products Table ##### --


SELECT * FROM products limit 10 -- Overview of the data
SELECT COUNT(*) FROM products -- 1,092 products



UPDATE products  -- make sure no space at the beginning or end
SET name = TRIM(name),
	sku = TRIM(sku)



SELECT * FROM products -- Make sure no NULL values in SKU (Primary Key) column
WHERE sku IS NULL

SELECT sku, count(*) from products
GROUP BY sku
HAVING COUNT(*) > 1 -- Make sure no duplication in sku (Primary Key) column



SELECT COUNT(*) FROM products -- Make sure no NULL values in name column
WHERE name IS NULL

SELECT name, count(*) from products
GROUP BY name
HAVING COUNT(*) > 1 -- We have products with the same name, that's OKay.



ALTER TABLE products ADD PRIMARY KEY(sku) -- SET sku column as PRIMARY KEY



SELECT COUNT(*) FROM products -- No NULL values in orderedquantity column
WHERE orderedquantity IS NULL

SELECT * FROM products 
WHERE orderedquantity = 0 -- 190 products that have no order placed



SELECT COUNT(*) FROM products -- No NULL values in stocklevel column
WHERE stocklevel IS NULL



SELECT COUNT(*) FROM products -- No NULL values in restockingleadtime column
WHERE restockingleadtime IS NULL



SELECT COUNT(*) FROM products -- No NULL values in restockingleadtime column
WHERE restockingleadtime IS NULL


	
SELECT COUNT(*) FROM products 
WHERE sentimentscore IS NULL -- check NULL values in sentimentscore column, 1 row


SELECT COUNT(*) FROM products 
WHERE sentimentmagnitude IS NULL -- check NULL values in sentimentmagnitude column, 1 row


SELECT * FROM products
WHERE sentimentscore IS NULL 
	  OR sentimentmagnitude IS NULL -- Show the records that contain NULL value, make sense becuase no order made.


UPDATE products
SET sentimentscore = 0, sentimentmagnitude = 0
WHERE sentimentscore IS NULL OR sentimentmagnitude IS NULL -- Replace NULL values with 0



SELECT MIN(orderedquantity) AS minOrderedquantity, 
	   MIN(stocklevel) AS minStocklevel,
	   MIN(restockingleadtime) AS minRestockingleadtime
FROM products  -- make sure no negative values in orderedquantity, stocklevel, restockingleadtime columns



SELECT MIN(sentimentscore) AS minsentimentscore, MAX(sentimentscore) AS maxsentimentscore,
	   MIN(sentimentmagnitude) AS minsentimentmagnitude, MAX(sentimentmagnitude) AS maxsentimentmagnitude
FROM products -- understand the range for sentimentscore and sentimentmagnitude   
 
