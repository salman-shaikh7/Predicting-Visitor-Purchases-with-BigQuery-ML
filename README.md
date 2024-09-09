## About this project

Title : Predicting Visitor Purchases with BigQuery ML

Objective : 
1.  Use BigQuery to find public datasets
2.  Query and explore the ecommerce dataset
3.  Create a training and evaluation dataset to be used for batch prediction
4.  Create a classification (logistic regression) model in BigQuery ML
5.  Evaluate the performance of your machine learning model
6.  Predict and rank the probability that a visitor will make a purchase


**BigQuery ML** can create, train, evaluate, and predict with ML models.

> About dataset

The dataset contains information >about customers, products, orders, logistics, web events and digital marketing campaigns. The contents of this >dataset are synthetic, and are provided to industry practitioners for the purpose of product discovery, testing, and >evaluation.

Information about different feature can be found on this link 

https://support.google.com/analytics/answer/3437719?hl=en

## STEP 1 : Explore ecommerce data

>>what % made a purchase?

* Query

```SQL
WITH visitors AS(
SELECT
COUNT(DISTINCT fullVisitorId) AS total_visitors
FROM `data-to-insights.ecommerce.web_analytics`
),

purchasers AS(
SELECT
COUNT(DISTINCT fullVisitorId) AS total_purchasers
FROM `data-to-insights.ecommerce.web_analytics`
WHERE totals.transactions IS NOT NULL
)

SELECT
  total_visitors,
  total_purchasers,
  total_purchasers / total_visitors AS conversion_rate
FROM visitors, purchasers

```

* Result

![alt text](images/image.png)

>>top 5 selling products
Query
```sql
SELECT
  p.v2ProductName,
  p.v2ProductCategory,
  SUM(p.productQuantity) AS units_sold,
  ROUND(SUM(p.localProductRevenue/1000000),2) AS revenue
FROM `data-to-insights.ecommerce.web_analytics`,
UNNEST(hits) AS h,
UNNEST(h.product) AS p
GROUP BY 1, 2
ORDER BY revenue DESC
LIMIT 5;
```
![alt text](images/image-1.png)

>>How many visitors bought on subsequent visits to the website?
could have bought on first as well
* Query 
```sql

WITH all_visitor_stats AS (
SELECT
  fullvisitorid, # 741,721 unique visitors
  IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit
  FROM `data-to-insights.ecommerce.web_analytics`
  GROUP BY fullvisitorid
)

SELECT
  COUNT(DISTINCT fullvisitorid) AS total_visitors, will_buy_on_return_visit
FROM all_visitor_stats
GROUP BY will_buy_on_return_visit

```



* Result