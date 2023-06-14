Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

SELECT country, city, 
	   CAST(SUM(totaltransactionrevenue) AS decimal(10,2)) AS totalRevenue 
	   FROM all_sessions
       GROUP BY country, city
       ORDER BY totalRevenue DESC


Answer:

1- United States, Unknown city  US$6,092.55
2- United States, San Francisco US$1,564.32
3- United States, Sunnyvale     US$992.23
4- United States, Atlanta       US$854.44
5- United States, Palo Alto     US$608.00




**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:

SELECT CAST(AVG(sv.units_sold) as decimal(10,2)) AS AvgOrder, al.country, al.city
	FROM sales_by_visitor AS sv
	JOIN all_sessions AS al
	ON sv.fullvisitorid = al.fullvisitorid
	-- WHERE sv.units_sold <> 0
	GROUP BY al.country, al.city
	ORDER BY AvgOrder DESC
						


Answer:

1- United States, Charlotte     43.11
2- United States, San Burno     30.26
3- United States, Unknow City   16.24





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

SELECT al.product_category AS Category, COUNT(*) AS TotalOrder, al.country, al.city
	FROM all_sessions AS al
	JOIN sales_by_visitor AS sv
	ON sv.fullvisitorid = al.fullvisitorid
	WHERE sv.units_sold <> 0 OR al.product_category <> 'NA'
	GROUP BY al.country, al.city, Category
	ORDER BY COUNT(*) DESC


Answer:
YES, there a pattern for some category like:

1- There is a high demand on apperal in USA
2- Youtube is popular in USA, UK and German
3- Men's T=Shirts are very popular in USA and Inida


**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

WITH totalproductOrder AS (
SELECT *,
RANK() OVER (PARTITION BY country , city ORDER BY Total_sold DESC)
FROM
(SELECT al.v2productname AS productname, SUM(sv.units_sold) AS Total_sold, al.country as country, al.city as city
	FROM all_sessions AS al
	JOIN sales_by_visitor AS sv
	ON sv.fullvisitorid = al.fullvisitorid
	WHERE sv.units_sold <> 0
	GROUP BY al.v2productname,al.country, al.city
	ORDER BY SUM(sv.units_sold) DESC) AS new_table
)	
SELECT *
FROM totalproductorder
WHERE RANK = 1


Answer:

- Android Twill Cap is the most popular product in USA
- Google Device Stand is the most popular product in Hong Kong
- Youtube Leatherette Notebook Combo is the most popular product in Canada





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







