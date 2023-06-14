Question 1: Which countries their visitors spent highest time on the site, is there a correlation with the revenues?

SQL Queries:

WITH sitetime AS ( 
SELECT SUM(DISTINCT a.timeonsite) AS totalTimeOnSite, al.country AS Country
FROM analytics AS a
JOIN all_sessions AS al
ON a.fullvisitorid = al.fullvisitorid
GROUP BY al.country
)
SELECT st.country, st.totalTimeOnSite,
	   CAST(SUM(s.totaltransactionrevenue) AS decimal(10,2)) AS totalRevenue 
	   FROM all_sessions AS s
	   JOIN sitetime AS st
	   ON st.country = s.country
       GROUP BY st.country, st.totalTimeOnSite
       ORDER BY totalRevenue DESC -- Q1
	   
	   

Answer: 

There is no correlation between time spent on site and the revenues, for example

Candain spent ove 7 hours and 20 mins on the site but their revenues is around US$150,
On the other hand, Australian spent almost half of Canadian time, and their revenue is US$358, almost 240% more. 


Question 2: Which month of the year makes more revenues ?

SQL Queries:

SELECT EXTRACT (MONTH FROM date) AS revenuDate, CAST(SUM(totaltransactionrevenue) AS decimal(10,2)) AS totalRevenues
FROM all_sessions
GROUP BY EXTRACT (MONTH FROM date)
ORDER BY totalRevenues DESC; --Q2

Answer:

1- The revenues in March at the highest level US$2,986
2- Then Decemebr with US$2,556, probably becuase of boxing day, black friday and holiday season.
3- August, September and October have the lowest revenues. 
   I think most of customers try to save their money to spend it in holiday season, where they can get better offer

   

Question 3: What is the top-selling product in each month?

SQL Queries:

WITH totalproductOrder AS (
SELECT *,
RANK() OVER (PARTITION BY soldDate ORDER BY Total_sold DESC)
FROM
(SELECT al.v2productname AS productname, SUM(sv.units_sold) AS Total_sold, EXTRACT (MONTH FROM al.date) AS soldDate
	FROM all_sessions AS al
	JOIN sales_by_visitor AS sv
	ON sv.fullvisitorid = al.fullvisitorid
	WHERE sv.units_sold <> 0
	GROUP BY al.v2productname, EXTRACT (MONTH FROM al.date)
	ORDER BY SUM(sv.units_sold) DESC) AS new_table
)	
SELECT *
FROM totalproductorder
WHERE RANK = 1

Answer: 

1- Google Men's short sleeve, #1 selling in January & Google Zip Hoodie is #1 selling in February.
I think becuase there will be a big discount on summer and fall clothes in winter.

2- In summer (June and July) we can see heather cap and bottel infuser are more popular, which makes sense.





Question 4: 

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:
