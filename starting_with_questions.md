Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

SELECT country, city, 
	   SUM(totaltransactionrevenue) AS totalRevenue, COUNT(*) 
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

SELECT AVG(a.units_sold) AS AvgOrder, COUNT(*), s.country, s.city
	FROM analytics AS a
	JOIN all_sessions AS s
	ON a.fullvisitorid = s.fullvisitorid
	WHERE a.units_sold <> 0
	GROUP BY s.country, s.city
	ORDER BY AvgOrder DESC
						


Answer:

1- United States, San Burno     52.60
2- United States, Unknown city  29.09
3- United States, Mountain View 16.16
4- Czechia, Unknown city        15.18
5- United States, San Jose      8.56



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







