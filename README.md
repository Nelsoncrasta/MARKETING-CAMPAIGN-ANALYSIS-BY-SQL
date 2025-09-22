# MARKETING-CAMPAIGN-ANALYSIS-BY-SQL

## Project Overview

**Project Title**: MARKETING CAMPAIGN ANALYSIS 
**Level**: Beginner  
**Database**: `campaign_db`

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries.

This project is to demostrate my SQL knowledge, i will be solving some business problems and give some insight at the end of it based on some queries output. 

i will be using MySQL for this project.

## Objectives

1. **Set up a marketing database**: Creating a Database and importing the CSV file as a table in the Database  
3. **Data Cleaning**: Identify and remove any records with missing or null values.  
4. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.  
5. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the camapagin data.  

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `campaign_db`.
- **Data import**: The dataset in marketing.csv was loaded into MySQL as a table named marketing using MySQL Workbench’s Import Wizard. From the Workbench menu,  
  I selected Server → Data Import, chose the CSV file, selected my campaign_db schema, and started the import. MySQL automatically created the table and inferred the column types.
  The table structure includes columns for id,c_date,campaign_name,category,campaign_id,impressions,mark_spent,clicks,leads,orders,revenue.
  For a reference how `marketing` table schema looks like i have written the code for the same.

```sql
CREATE DATABASE campaign_db;

CREATE TABLE marketing
(
    id INT PRIMARY KEY,
    c_date DATE,	
    campaign_name VARCHAR(100),
    category VARCHAR(50),	
    campaign_id INT,
    impressions INT,
    mark_spent FLOAT,
    clicks INT,
    leads INT,	
    orders INT,
    revenue FLOAT
);
```

### 2. Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Campaign names**: Find out how many unique campaigns are in the dataset.
- **Category Count**: Identify all unique categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset if found delete them.

```sql
SELECT COUNT(*) FROM marketing;
SELECT DISTINCT campaign_name FROM marketing;
SELECT DISTINCT category FROM marketing;

SELECT * FROM marketing
WHERE 
		c_date IS NULL OR campaign_name IS NULL OR category IS NULL;
```
### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

1. **WHICH COMPAIGNS DELIVERS THE BEST RETURN ON INVESTMENT**:

```sql
SELECT campaign_name, 
		ROUND((expenditure/Revenue)*100,2) AS ROI
    FROM (
					SELECT campaign_name,SUM(mark_spent) AS expenditure,
                  SUM(revenue) AS Revenue FROM marketing
					        GROUP BY 1) AS t1
ORDER BY 2 DESC;
```

2. **WHAT IS THE AVERAGE COST TO ACQUIRE A LEAD OR AN ORDER**:

```sql
SELECT campaign_name,
        ROUND(SUM(mark_spent)/sum(leads),2) AS expense_perlead,
        RANK () OVER (ORDER BY ROUND(SUM(mark_spent)/sum(leads),2) ) as leads_ranks, 
        ROUND(SUM(mark_spent)/sum(orders),2) AS expense_perorder,
        RANK() OVER (ORDER BY ROUND(SUM(mark_spent)/sum(orders),2) ) AS order_ranks
        FROM marketing
GROUP BY 1
ORDER BY 3,5;
```

3. **WHAT IS THE AVERAGE COST TO ACQUIRE ORDER BY CATEGORY AND CAMPAIGNS**:
   
```sql
SELECT 
		category,
        ROUND(SUM(mark_spent)/sum(leads),2) AS expense_perlead,
        ROUND(SUM(mark_spent)/sum(orders),2) AS expense_perorder
        FROM marketing
GROUP BY 1
ORDER BY 3;
```

4. **CONVERSION RATE FROM LEADS TO ORDER BY CATEGORY AND CAMPAIGNS**:

```sql
SELECT category,
	  (SUM(ORDERS)/SUM(LEADS))*100 AS conversion_rate
	  FROM marketing
GROUP BY 1
ORDER BY 2 DESC;
```

```sql
SELECT campaign_name,
	    (SUM(ORDERS)/SUM(LEADS))*100 AS conversion_rate
	    FROM marketing
GROUP BY 1
ORDER BY 2 DESC;
```

5. **WHICH CAMAPIGN AND CATEGORY HAS THE BEST CLICK THROUGH RATE**:
```sql
SELECT 
	  campaign_name,
    ROUND((SUM(clicks)/SUM(impressions))*100,2) AS CTR FROM marketing
WHERE impressions>0
GROUP BY 1
order by 2 desc;
```
```sql
SELECT 
	  category,
    ROUND((SUM(clicks)/SUM(impressions))*100,2) AS CTR FROM marketing
WHERE impressions>0
GROUP BY 1
order by 2 desc;
```

6. **THE PERCENTAGE OF CONVERSION FROM EACH FIELD**:
```sql
select 
	category,
    ROUND((SUM(clicks)/SUM(impressions))*100,2) AS clicks_per_impression,
    ROUND((SUM(leads)/SUM(clicks))*100,2) AS leads_per_clicks,
    ROUND((SUM(orders)/SUM(leads))*100,2) AS orders_per_leads
    FROM marketing
GROUP BY 1
ORDER BY 4 DESC;
```

7. **TOP SPENDING V/S TOP EARNING CATEGORY AND CAMPAIGNS**:
```sql
SELECT 
	category,
    ROUND(SUM(mark_spent),0) as mark_spent,
    RANK() OVER(ORDER BY SUM(mark_spent) DESC) AS expense_rank,
	  ROUND(SUM(revenue),0) AS revenue,
    RANK() OVER(ORDER BY SUM(revenue)DESC) AS revenue_rank,
    ROUND((SUM(revenue)/SUM(mark_spent))*100,0)AS ROI
    FROM marketing
GROUP BY 1;
```

```sql
SELECT 
	campaign_name,
    ROUND(SUM(mark_spent),0) as mark_spent,
    RANK() OVER(ORDER BY SUM(mark_spent) DESC) AS expense_rank,
	  ROUND(SUM(revenue),0) AS revenue,
    RANK() OVER(ORDER BY SUM(revenue)DESC) AS revenue_rank,
    ROUND((SUM(revenue)/SUM(mark_spent))*100,0)AS ROI
    FROM marketing
GROUP BY 1;
```

8. **WHICH ADVERTISING CHANNEL GIVES BEST PROFIT MARGIN**:
```sql
SELECT category,
		    ROUND(SUM(revenue)-SUM(mark_spent),0) AS profit_margin 
        FROM marketing
GROUP BY 1
ORDER BY 2 DESC;
```
```sql
SELECT campaign_name,
		    ROUND(SUM(revenue)-SUM(mark_spent),0) AS profit_margin 
        FROM marketing
GROUP BY 1
ORDER BY 2 DESC;
```

9. **THE CAMPAIGNS WHICH GENERATED ZERO REVENUE, THEIR DAY AND THE NUMBER OF TIMES THEY GENERATED ZERO REVENUE**:
```sql
SELECT DAYNAME(c_date),COUNT(*) AS number_of_days,
		    SUM(mark_spent) AS expense 
        FROM marketing
WHERE revenue=0 OR orders = 0
GROUP BY 1
ORDER BY 2 DESC;
```

10. **ON WHICH DAY A PARTICULAR CAMPAIGN HAS RECORDED HIGHEST ROI**

```sql
WITH ranking_days AS (
SELECT campaign_name,
		DAYNAME(c_date) AS dayname,
        ROUND((SUM(revenue)/SUM(mark_spent)*100),2) AS ROI,
        DENSE_RANK() OVER (PARTITION BY campaign_name ORDER BY SUM(revenue)/SUM(mark_spent) DESC) AS ranks
        FROM marketing
GROUP BY 1,2)
SELECT * FROM ranking_days 
WHERE ranks=1 
ORDER BY 3 DESC;
```
11. **WHAT IS THE GROWTH OVER DAY ON EACH DAY FOR EACH CAMPAIGNS**:
```sql
SELECT
    campaign_name,
    c_date,
    DAYNAME(c_date) AS dayname,
    revenue,
    LAG(revenue) OVER (
        PARTITION BY campaign_name
        ORDER BY c_date
    ) AS prev_day_revenue,
    -- % change compared to previous day
    ROUND(
        ( (revenue - LAG(revenue) OVER (
                PARTITION BY campaign_name
                ORDER BY c_date
            )
          ) / NULLIF(LAG(revenue) OVER (
                PARTITION BY campaign_name
                ORDER BY c_date
            ), 0)
        ) * 100, 2
    ) AS revenue_growth_pct
FROM marketing
ORDER BY campaign_name, c_date;
```
12. **DAILY PERFORMANCE TRENDS**:
```sql
SELECT 
	  c_date,
    ROUND(SUM(mark_spent),0) AS expenses,
    SUM(revenue) AS revenue,
    SUM(orders) AS total_orders,
    ROUND(SUM(mark_spent)/SUM(orders),0) AS per_order_expense
    FROM marketing
GROUP BY 1;
```

13. **OVERALL CONVERSION RATES FROM LEADS**:
```sql
SELECT 
	(SUM(orders)/SUM(leads))*100 AS total_avg 
    FROM marketing;
```

## FINDINGS

### ROI  
a.YouTube_Blogger delivered the highest ROI at ~377 %, making it the most successful campaign by a wide margin and a conversion rates from lead to order of about 19%  
b.Facebook_Retargeting ranked second with an ROI of about 201 % and it has a highest conversion rates from leads to order(21%)  
c.Instagram_Tier2 performed the worst, recording only ~63 % ROI and also has the lowest conversion rate only 3%  
d.Campaigns with ROI below 100 %—including Facebook_Tier1, Facebook_Tier2, Google_Wide, and Instagram_Tier2—can be considered unsuccessful.  

### COST TO ACQUIRE LEADS AND ORDERS  
a.Best Overall Performer: YouTube_Blogger offers the strongest cost efficiency—moderate lead cost but the lowest cost per order, showing excellent lead-to-order conversion.
b.Lead vs. Order Gap: Instagram_Tier2 generates the cheapest leads but has a much higher cost per order, indicating weak conversion from leads to paying customers.
c.High-Cost Campaigns: Facebook_LAL, Google_Hot, and FacebOOK_Tier2 have very high costs for both leads and orders, suggesting they need optimization or budget cuts

### PERFORMANCE BASED ON CATEGORY  
a.Influencer category achieves the best order conversion rate (~17 %) and highest profit margins.  
b.Social category generates leads easily but converts them to orders at only ~8 %.  
media has lowest clicks per impression but 2nd highest conversion rate from leads to order  

### DAY WISE TRENDS  
a.Best revenue day: Friday generated the highest total revenue and ROI, followed closely by Saturday.  
b.Daily growth rates fluctuated without a clear upward or downward trend.  

**ENGAGEMNET METRICS**
-Top campaign: Facebook_Retargeting at ~3 %.  
-Lowest campaign: Banner_Partner at ~0.03 %.  
-Top category: Influencer (~1 % CTR).  

## CONCLUSION

**The most successful camapign was Youtube_blogger in terms of ROI**  
**Double-down on Influencer campaigns for better profitability and higher conversion rates.**  
**Social category is the worst performing category with huge loses should spent on these campaigns very carefully**  
**Optimize or discontinue under-performing social campaigns with ROI below 100 %.**  
**Friday campaigns show strong revenue potential, suggesting that ad scheduling can influence success.**  
**On a particular day a campaign can perform better than any other day. For example youtube_blogger could generate 1.5x of ROI on saturdays compared to their overall ROI**
**Media category and banner_partner has the lowest clicks per impression but a better conversion rate when they converted into leads so if focused on the initial phase and can get better CTR the category could perform better**  


## Author - Nelson Crasta

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles.



Thank you for your support, and I look forward to connecting with you!
