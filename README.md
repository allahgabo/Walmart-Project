PROJECT WALMART

Create an account for Kaggle
Create a token in Kaggle , it will download as json file 
We need to create a environment env

We need to install Kaggle  pip install Kaggle
kaggle datasets download -d --

Business 

-- select all 
SELECT * FROM public.walmart;

-- DROP TABLE walmart;


-- distinct in payment method 
SELECT DISTINCT payment_method FROM walmart;

-- group by payment method
SELECT DISTINCT payment_method , COUNT(*)  FROM walmart GROUP BY payment_method;

SELECT DISTINCT(branch)  FROM walmart;

SELECT COUNT(DISTINCT branch)  FROM walmart;


-- Max 
SELECT MAX(quantity) FROM walmart;

-- MIN 
SELECT MIN(quantity) FROM walmart;

-- Business problems 
-- Find different payment method and number of the transaction number of qty sold

SELECT DISTINCT payment_method ,COUNT(*),SUM(quantity) AS NO_QTY FROM walmart GROUP BY payment_method;



-- identify the highest -rated category in each branch , displaying the branch , category avg rating

SELECT * FROM walmart;
SELECT branch, category,AVG(rating) AS avg_rating FROM walmart GROUP BY branch,category ORDER BY branch,avg_rating DESC;


SELECT branch, category,AVG(rating) AS avg_rating , RANK() OVER(PARTITION BY branch ORDER BY AVG(rating) DESC) AS rank FROM walmart GROUP BY branch,category;


SELECT * FROM (
	SELECT branch, category,AVG(rating) AS avg_rating , RANK() OVER(PARTITION BY branch ORDER BY AVG(rating) DESC) AS rank
	FROM walmart GROUP BY branch,category
) as ranks
WHERE rank = 1;


-- identify the busiest day for each branch based on the number of transaction
-- SELECT date ,TO_DATE(date ,'DD-MM-YY') AS formated_date ,branch  FROM walmart;

-- SELECT branch  ,TO_CHAR(TO_DATE(date ,'DD-MM-YY'),'Day') AS format_day ,	count(*) as no_transaction FROM walmart GROUP BY 1,2 ORDER BY 1,3 DESC;


SELECT * FROM (
	SELECT 
	branch  ,TO_CHAR(TO_DATE(date ,'DD-MM-YY'),'Day') AS day_name ,	
	COUNT(*) as no_transaction ,
	RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS rank
	FROM walmart GROUP BY 1,2
)  AS rank
WHERE rank = 1



-- calculate the total quantity of items sold per payment method 
-- list payment method and total quantity 

SELECT payment_method,SUM(quantity) as no_qty FROM walmart GROUP BY payment_method;



-- determine the avg , min,and max rating of categroy for each city 
-- list the city , agv rating min rating and max rating

SELECT  city, category,MIN(rating) as min_rating,MAX(rating) as max_rating, AVG(rating) as avg_rating FROM walmart GROUP BY 1,2;



-- calculate toatl profit for each category by considering toatl profit as (unit price * quantity * profil margin)
-- lsit category and total profilt order by from highest ot lowers profit


SELECT category , SUM(total_price) AS total_revenue ,SUM(total_price * profit_margin) as profit FROM walmart GROUP BY 1 ORDER BY 1,2 DESC;



-- determine the most common payment method for each branch display the branch and preferred payment method

SELECT branch, payment_method,COUNT(*) AS total_transaction,
	RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS rank	
	FROM walmart GROUP BY branch,payment_method;

WITH ciy as 
(
	SELECT branch, payment_method,COUNT(*) AS total_transaction,
	RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS rank	
	FROM walmart GROUP BY branch,payment_method

)SELECT *  FROM ciy 
WHERE rank = 1;





-- categorize city into 3 group morning ,afternoon ,evening 
-- find out whci of the shift and number of invocies
--  convert time::time to time 

SELECT 
	 * ,
	CASE 
		WHEN EXTRACT (HOUR FROM (time::time)) < 12  THEN 'Morning'
		WHEN EXTRACT (HOUR FROM (time::time)) BETWEEN 12 AND 17 THEN 'Afternoon'
		ELSE 'Evening'
		
	END day_time 
FROM walmart ;


SELECT 
    branch,
    CASE 
        WHEN EXTRACT(HOUR FROM time::time) < 12 THEN 'Morning'
        WHEN EXTRACT(HOUR FROM time::time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS day_time,
    COUNT(*) AS total_count
FROM walmart
GROUP BY branch, day_time ORDER BY 1,3;



-- identify 5 branch with the highest descres ratio in 
-- revenue comapre to  last year (currect year 2023 and last 2022)

--  to calculate ratio =last year - current year / last revune * 100


--  2022
WITH reveue_2022 AS 
(
	SELECT branch,SUM(total_price) as revenue FROM walmart
	WHERE EXTRACT (YEAR FROM TO_DATE(date ,'DD/MM/YY')) =2022
	GROUP BY 1
),

revnue_2023 as 
(
SELECT branch,SUM(total_price) as revenue FROM walmart
	WHERE EXTRACT (YEAR FROM TO_DATE(date ,'DD/MM/YY')) =2023
	GROUP BY 1
	
)
SELECT ly.branch,
		
		cy.revenue as last_year_revenue,
		cy.revenue as current_year_revenue,
		ROUND((ly.revenue - cy.revenue )::numeric / ly.revenue::numeric * 100,2) AS revenue_decrse 
FROM reveue_2022 as ly
JOIN revnue_2023 as cy 
ON  ly.branch = cy.branch

WHERE ly.revenue > cy.revenue 
ORDER BY 4 DESC 
LIMIT 5
