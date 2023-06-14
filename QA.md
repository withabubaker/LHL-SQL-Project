What are your risk areas? Identify and describe them.

- Unable to extract the data, the file could be corrupted.
- Use the correct data type and format.
- Alot of missing data, which could impact the value fo the data and make it useless
- The accuracy of the data, wrong information.
- Diffult to find a correlation between the tables, hard to indetify primary and foreign key.
- Unable to understand the meaning of the data, due to unclear coloumn name and clear description.


QA Process:
Describe your QA process and include the SQL queries used to execute it.

--## Extracting Process
1- Extract the data from 5 CVS files using the proper data type, 
   and make sure the data has been extracted completed by comparing the number of rows in the CSV files and database (Refer Code 1).

--## Transformation Process  
2- Analyze the NULL values, if column is numeric, then replace it with 0, 
   if the column is string then replace it with NA. This will make easy when manipulating the data (Ex. Refer Code 2).

3- Convert the data type as needed (Ex. Refer Code 3).

4- DROP the column If all the data is NULL, or the column has no benefit (Ex. Refer Code 4).

5- TRIM string column, remove all spaces from the text except the ones between the words (Ex. Refer Code 5).

6- Check the accuracy and validity of the data, some numeric column should'nt have negative value like units_sold column (Ex. Refer Code 6).

7- Set SKU column in product column as PRIMARY KEY (Refer Code 7).

8- Create new table named sales_visitors, continas fullvisitorid and total units sold by each visitor, 
   then SET fullvisitorid as primary key (Refer Code 8).
   
9- Create new table named visitor_locations, continas fullvisitorid, country and city for each visitor, 
   then SET fullvisitorid as foreign key (Refer Code 9).




--- #### Code 1 ####---

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


--- #### Code 2 ####---

SELECT COUNT(*) FROM analytics
WHERE units_sold is NULL -- Number of rows 4,205,975

UPDATE analytics
SET units_sold = 0
WHERE units_sold IS NULL


--- #### Code 3 ####---

ALTER TABLE analytics  
ALTER visitstarttime TYPE TIMESTAMPTZ
USING timezone('UTC', TO_TIMESTAMP(visitstarttime)) -- Change visitstarttime column data type to timestamp


--- #### Code 4 ####---

SELECT COUNT(*) FROM analytics
WHERE userid is NULL -- No data in userid column, better to drop it

ALTER TABLE analytics DROP userid -- Drop userid column


--- #### Code 5 ####---

UPDATE analytics  
SET fullvisitorid = TRIM(fullvisitorid),
	channelgrouping = TRIM(channelgrouping),
	socialengagementtype = TRIM(socialengagementtype),
	timeonsite = TRIM(timeonsite) -- make sure no space at the beginning or end
	
	
--- #### Code 6 ####---	

SELECT MIN(units_sold) FROM analytics -- some values in negative, that's wrong

SELECT * from analytics 
WHERE units_sold < 0 -- one record only in negative (-89), we will update it to positive

UPDATE analytics
SET units_sold = 89
WHERE units_sold = -89


--- #### Code 7 ####---	

SELECT * FROM products -- Make sure no NULL values in SKU (Primary Key) column
WHERE sku IS NULL

SELECT sku, count(*) from products
GROUP BY sku
HAVING COUNT(*) > 1 -- Make sure no duplication in sku (Primary Key) column

ALTER TABLE products ADD PRIMARY KEY(sku) -- SET sku column as PRIMARY KEY



--- #### Code 8 ####---	

CREATE TABLE IF NOT EXISTS sales_by_visitor (
	fullvisitorid Varchar,
	units_sold int
) -- Create new table for units_sold per visitor

INSERT INTO sales_by_visitor(fullvisitorid, units_sold)
	SELECT fullvisitorid, SUM(units_sold)
	FROM analytics
	GROUP BY fullvisitorid -- Insert data into new table


SELECT fullvisitorid, COUNT(*) FROM sales_by_visitor
GROUP BY fullvisitorid
HAVING COUNT(*) > 1 -- Make sure no duplicate value

ALTER TABLE sales_by_visitor ADD PRIMARY KEY(fullvisitorid)


--- #### Code 9 ####---	

CREATE TABLE IF NOT EXISTS visitors_location (
	fullvisitorid Varchar,
	country Varchar,
	city Varchar
) -- Create new table for visitor location

INSERT INTO visitors_location(fullvisitorid, country, city)
	SELECT fullvisitorid, country, city
	FROM all_sessions
	GROUP BY fullvisitorid, country, city -- Insert data into visitors_location table
					

SELECT fullvisitorid, country, city,
COUNT(fullvisitorid) OVER (PARTITION BY fullvisitorid) AS Total
FROM visitors_location
ORDER BY total DESC -- Some visitors appear in two different countires, I think thats Okay

SELECT fullvisitorid
	FROM visitors_location
	WHERE fullvisitorid IN (SELECT fullvisitorid FROM sales_by_visitor) -- There are 10,328 visitorid not listed in sales_by_visitor
	
INSERT INTO sales_by_visitor
	SELECT distinct fullvisitorid
	FROM visitors_location
	WHERE fullvisitorid NOT IN (SELECT fullvisitorid FROM sales_by_visitor) -- Insert missing visitorid into sales_by_visitor table
	
UPDATE sales_by_visitor
SET units_sold = 0
WHERE units_sold is NULL -- Set NULL value to 0	


ALTER TABLE visitors_location ADD FOREIGN KEY(fullvisitorid) 
	REFERENCES sales_by_visitor(fullvisitorid) -- Assign fullvisitorid foreign key in visitors_location table

ALTER TABLE all_sessions ADD FOREIGN KEY(fullvisitorid) 
	REFERENCES sales_by_visitor(fullvisitorid) -- Assign fullvisitorid foreign key in visitors_location table