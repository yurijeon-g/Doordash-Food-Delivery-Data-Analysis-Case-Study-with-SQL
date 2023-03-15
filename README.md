# Food-Delivery-Data-Analysis-Case-Study-with-SQL
Using a dataset of Doordash (source: Kaggle), I had the opportunity to exercise my time analysis skills to manipulate and uncover insights about food delivery intervals as well as insights on driver income, hourly trends in high demand for food delivery as well as identified high revenue grossing restaurants and analyzed contributing factors to high grossing restaurants. 


## Introduction
DoorDash is a food delivery service that connects customers with local restaurants. To maintain its edge in an increasingly competitive market, DoorDash has invested heavily in data analytics to gain insights into customer preferences and improve its operations. In this portfolio project, I will analyze a dataset provided by DoorDash, which contains information on thousands of food deliveries. By examining this data,I will produce insights into the factors that drive orders and identify areas where DoorDash can improve its operations to better serve its customers. The dataset includes information such as the driver, restaurant, and customer IDs, the order total, amount of discount and tip, delivery time, and more. Using data analysis techniques, I will delve into this data to gain insights that will inform my understanding of DoorDash's operations.

I hope to answer questions such as: What are the most popular restaurants in the region? How long does it typically take for an order to be delivered? Are there any factors that influence whether an order is refunded? Are there any particular relationships between these features?

## Insights from the Analysis

The data analysis project above provides information about the restaurant and region breakdown. The results are divided into three sections; delivery region breakdown, identifying low and high performing restaurants, and the relationship between delivery time and restaurant ranking.

The first section shows the revenue and order volume of each delivery region. From the revenue breakdown, we can conclude that Palo Alto generates the most revenue, followed by Mountain View and San Jose. On the other hand, when we look at the order volume, Palo Alto still dominates with the highest number of orders, followed by Mountain View and San Jose.

The second section, "Identifying Low & High Performing Restaurants," provides insight into the best and worst-performing restaurants based on their revenue. The analysis includes a partitioned bottom 5% of the total revenue into evenly distributed bins. This result considers potential refunds and subtracts them from the order total if they occur. The analysis reveals that the lowest revenue-generating restaurants are in Palo Alto. This information can be helpful in the decision-making process to improve the business of underperforming restaurants.

Lastly, the third section analyzes the relationship between delivery time and restaurant ranking. The data suggests that there is no correlation between a low ranking and average delivery time. The results reveal that restaurant 303 has the lowest ranking, despite having the second-fastest delivery time. This information could indicate that other factors besides delivery time, such as customer service or menu selection, may contribute to restaurant ranking.

Overall, this data analysis project provides helpful insights to evaluate the restaurant and region breakdown, allowing stakeholders to make data-driven decisions to improve business.


## About the Data: 
Glossary of the dataset is provided [here](https://github.com/yurijeon-g/Food-Delivery-Data-Analysis-Case-Study-with-SQL/blob/main/Data%20Glossary)


## Data Manipulation

### Converting data types to timezone datatypes

**All the features recording time were varchar datatypes thus, needed to firstly be converted to datetime datatypes.**

 ``` sql
 ALTER TABLE fooddelivery ADD COLUMN customer_order_timestamp TIMESTAMP;
ALTER TABLE fooddelivery ADD COLUMN order_with_restaurant_timestamp TIMESTAMP;
ALTER TABLE fooddelivery ADD COLUMN driver_at_restaurant_timestamp TIMESTAMP;
ALTER TABLE fooddelivery ADD COLUMN delivered_to_consumer_timestamp TIMESTAMP;

UPDATE fooddelivery 
SET 
    customer_order_timestamp = to_timestamp(Customer_placed_order_datetime, 'DD HH24:MI:SS'), 
    order_with_restaurant_timestamp = to_timestamp(Placed_order_with_restaurant_datetime, 'DD HH24:MI:SS'), 
    driver_at_restaurant_timestamp = to_timestamp(Driver_at_restaurant_datetime, 'DD HH24:MI:SS'), 
    delivered_to_consumer_timestamp = to_timestamp(Delivered_to_consumer_datetime, 'DD HH24:MI:SS');

ALTER TABLE fooddelivery DROP COLUMN Customer_placed_order_datetime;
ALTER TABLE fooddelivery DROP COLUMN Placed_order_with_restaurant_datetime;
ALTER TABLE fooddelivery DROP COLUMN Driver_at_restaurant_datetime;
ALTER TABLE fooddelivery DROP COLUMN Delivered_to_consumer_datetime;


  ```
  
### Correcting month portion of date time features
As seen below, there are negative values for total time elapsed due to the lack of month in this data. The default was put as 1. Thus what would have been a new month by the time the food had been delivered is shown as a date earlier than when the customer ordered the food. 

This query shows this inconsistency

 ``` sql
SELECT total_time_elapsed customer_order_timestamp, order_with_restaurant_timestamp,	driver_at_restaurant_timestamp,delivered_to_consumer_timestamp 
FROM fooddelivery 
WHERE customer_order_timestamp > delivered_to_consumer_timestamp




  ```

| customer_order_timestamp | order_with_restaurant_timestamp | driver_at_restaurant_timestamp | delivered_to_consumer_timestamp |
| --- | --- | --- | --- |
| -29 days +02:35:39.000000000 | 0001-01-02 18:22:29 BC | 0001-01-02 19:34:56 BC |  |
| -31 days +02:00:15.000000000 | 0001-01-01 00:03:54 BC | 0001-01-01 00:24:35 BC | 0001-01-01 00:47:03 BC |
| -31 days +00:34:10.000000000 | 0001-01-31 23:58:17 BC | 0001-01-01 00:10:13 BC | 0001-01-01 00:31:26 BC |
| -31 days +00:43:42.000000000 | 0001-01-31 23:37:15 BC | 0001-01-31 23:47:26 BC | 0001-01-01 00:09:00 BC |
| -31 days +00:54:34.000000000 | 0001-01-31 23:45:35 BC | 0001-01-01 00:18:05 BC |  |
| -30 days +01:00:22.000000000 | 0001-01-01 17:45:48 BC | 0001-01-01 18:16:04 BC | 0001-01-01 18:55:53 BC |
| -31 days +02:24:11.000000000 | 0001-01-01 00:14:09 BC | 0001-01-01 00:46:01 BC | 0001-01-01 01:00:53 BC |

  ### Editing the month values to 02

**In the case where the delivery of the food is recorded as an earlier date than when the customer ordered**

 ``` sql
UPDATE fooddelivery
SET delivered_to_consumer_timestamp = delivered_to_consumer_timestamp + INTERVAL '1 month'
WHERE customer_order_timestamp > delivered_to_consumer_timestamp

UPDATE fooddelivery
SET order_with_restaurant_timestamp
 = order_with_restaurant_timestamp + INTERVAL '1 month'
WHERE customer_order_timestamp > order_with_restaurant_timestamp
;

UPDATE fooddelivery
SET driver_at_restaurant_timestamp = driver_at_restaurant_timestamp
 + INTERVAL '1 month'
WHERE customer_order_timestamp > driver_at_restaurant_timestamp
;


  ```
  

  ### Creating an interval type column for total time taken/ driver time taken

**Creating interval of total time**

 ``` sql
 ALTER TABLE fooddelivery
ADD COLUMN total_time INTERVAL;

UPDATE fooddelivery
SET total_time = delivered_to_consumer_timestamp - customer_order_timestamp;


  ```
  
**Interval created for driver time taken**

 ``` sql
ALTER TABLE fooddelivery ADD COLUMN driver_delivery_time INTERVAL;
UPDATE fooddelivery SET driver_delivery_time = delivered_to_consumer_timestamp - Driver_at_restaurant_timestamp;


  ```
  
  
  
  ### Finding Null Counts


 ``` sql
 SELECT 
  COUNT(CASE WHEN id IS NULL THEN 1 END) AS id_null_count,
  COUNT(CASE WHEN driver_id IS NULL THEN 1 END) AS driver_id_null_count,
  COUNT(CASE WHEN restaurant_id IS NULL THEN 1 END) AS restaurant_id_null_count,
  COUNT(CASE WHEN consumer_id IS NULL THEN 1 END) AS consumer_id_null_count,
  COUNT(CASE WHEN is_new IS NULL THEN 1 END) AS is_new_null_count,
  COUNT(CASE WHEN delivery_region IS NULL THEN 1 END) AS delivery_region_null_count,
  COUNT(CASE WHEN is_asap IS NULL THEN 1 END) AS is_asap_null_count,
  COUNT(CASE WHEN order_total IS NULL THEN 1 END) AS order_total_null_count,
  COUNT(CASE WHEN amount_of_discount IS NULL THEN 1 END) AS amount_of_discount_null_count,
  COUNT(CASE WHEN amount_of_tip IS NULL THEN 1 END) AS amount_of_tip_null_count,
  COUNT(CASE WHEN refunded_amount IS NULL THEN 1 END) AS refunded_amount_null_count,
  COUNT(CASE WHEN total_time_elapsed IS NULL THEN 1 END) AS total_time_elapsed_null_count,
  COUNT(CASE WHEN customer_order_timestamp IS NULL THEN 1 END) AS customer_order_timestamp_null_count,
  COUNT(CASE WHEN order_with_restaurant_timestamp IS NULL THEN 1 END) AS order_with_restaurant_timestamp_null_count,
  COUNT(CASE WHEN driver_at_restaurant_timestamp IS NULL THEN 1 END) AS driver_at_restaurant_timestamp_null_count,
  COUNT(CASE WHEN driver_delivery_time IS NULL THEN 1 END) AS driver_delivery_time_null_count,
  COUNT(CASE WHEN total_time IS NULL THEN 1 END) AS total_time_timestamp_null_count,
  COUNT(CASE WHEN delivered_to_consumer_timestamp IS NULL THEN 1 END) AS delivered_to_consumer_timestamp_null_count
FROM fooddelivery;


  ```
| order_with_restaurant_timestamp_null_count | driver_at_restaurant_timestamp_null_count | driver_delivery_time_null_count |
| --- | --- | --- |
| 40 | 4531 | 4531 |
  
## Restaurant and Region Analysis

### Delivery Region Breakdown

**Breakdown of Delivery Regions By Revenue**

 ``` sql
SELECT delivery_region, sum(order_total) as total FROM fooddelivery 
GROUP BY 1

  ```
  
| Palo Alto | 600034.2 |
| --- | --- |
| Mountain View | 195792.4 |
| San Jose | 125045.3 |
| None | 5833.46 |
  

**Breakdown of Order Volume by Delivery Region**

 ``` sql
SELECT delivery_region, count(*) as total FROM fooddelivery 
GROUP BY 1
ORDER BY 2 DESC


  ```
| delivery_region | total |
| --- | --- |
| Palo Alto | 11433 |
| Mountain View | 3760 |
| San Jose | 2859 |
| None | 26 |
  
  
  ### Identifying Low & High Performing Restaurants

Partitioned bottom 5% of the total revenue into evenly distributed bins. We have also taken into account potential refunds thus will subtract them from order total if they occur.

**Lowest 5% of Restaurants in Revenue**

 ``` sql
WITH percentile AS (
  SELECT restaurant_id, delivery_region, sum(order_total-refunded_amount) as total, ntile(20) OVER (order by sum(order_total-refunded_amount) ) as ntile
  FROM fooddelivery
  GROUP BY restaurant_id,delivery_region
)
SELECT restaurant_id, total,delivery_region
FROM percentile
WHERE ntile = 1
ORDER BY total;


  ```
| restaurant_id | total | delivery_region |
| --- | --- | --- |
| 210 | -11.1 | Palo Alto |
| 25 | -3 | Palo Alto |
| 260 | 18.94 | Palo Alto |
| 338 | 23.35 | Mountain View |
| 363 | 23.35 | San Jose |
| 226 | 25.52 | Palo Alto |
| 116 | 26.06 | Palo Alto |
| 225 | 28.73 | Palo Alto |
| 368 | 31.5 | Mountain View |
| 250 | 31.83 | Palo Alto |
| 177 | 34.76 | San Jose |
| 335 | 36.94 | Mountain View |
| 408 | 37.12 | Mountain View |
| 400 | 42.92 | San Jose |
| 378 | 45.15 | San Jose |
| 130 | 45.15 | San Jose |
| 57 | 46.75 | Palo Alto |
| 151 | 48.37 | San Jose |
| 383 | 49.36 | San Jose |
| 323 | 49.45 | San Jose |
| 392 | 50.91 | San Jose |
| 303 | 51.24 | Palo Alto |
  


**Highest 5% of Restaurants in Revenue**
 ``` sql
 WITH percentile AS (
  SELECT restaurant_id, delivery_region, sum(order_total-refunded_amount) as total, ntile(20) OVER (order by sum(order_total-refunded_amount) ) as ntile
  FROM fooddelivery
  GROUP BY restaurant_id,delivery_region
)
SELECT restaurant_id, total,delivery_region
FROM percentile
WHERE ntile = 20
ORDER BY total DESC;


  ```
| restaurant_id | total | delivery_region |
| --- | --- | --- |
| 9 | 38846.96 | Palo Alto |
| 8 | 37716.77 | Palo Alto |
| 20 | 31697.43 | Palo Alto |
| 63 | 26671.6 | Palo Alto |
| 107 | 25549.49 | Palo Alto |
| 10 | 21776.36 | Palo Alto |
| 68 | 19101.26 | Palo Alto |
| 12 | 15319.42 | Palo Alto |
| 3 | 14308.12 | Palo Alto |
| 5 | 12787.61 | Palo Alto |
| 28 | 11622.84 | Palo Alto |
| 27 | 10864.67 | Palo Alto |
| 98 | 10603.74 | Mountain View |
| 83 | 10545.24 | Palo Alto |
| 19 | 10025.15 | Palo Alto |
| 205 | 9711.67 | Palo Alto |
| 100 | 9647.06 | Palo Alto |
| 47 | 9460.4 | Palo Alto |
| 6 | 9294.8 | Palo Alto |
| 62 | 9178.59 | Palo Alto |
| 194 | 9004.15 | San Jose |  
  
     
  ### Relationship between delivery time and restaurant ranking
Why are they unpopular is it the delivery? Where they ranking in terms of average delivery time?

**Low Ranking Restaurants against Avg Delivery Time**

 ``` sql
 WITH p AS (
  SELECT restaurant_id, delivery_region, sum(order_total-refunded_amount) as total, ntile(20) OVER (order by sum(order_total-refunded_amount) ) as ntile, avg(total_time) as avgdelivery
  FROM fooddelivery
  GROUP BY restaurant_id,delivery_region
), r AS (SELECT restaurant_id, dense_rank() OVER (ORDER BY AVG(total_time)) as ranking FROM fooddelivery GROUP BY restaurant_id )
SELECT p.restaurant_id, p.total,p.delivery_region, p.avgdelivery, r.ranking
FROM p
LEFT JOIN r ON p.restaurant_id= r.restaurant_id
WHERE ntile = 1
ORDER BY total DESC;

  ```
| restaurant_id | total | delivery_region | avgdelivery | ranking |
| --- | --- | --- | --- | --- |
| 303 | 51.24 | Palo Alto | 1:11:00 | 255 |
| 392 | 50.91 | San Jose | 1:22:50 | 246 |
| 323 | 49.45 | San Jose | 1:18:57 | 241 |
| 383 | 49.36 | San Jose | 04:38.5 | 187 |
| 151 | 48.37 | San Jose | 0:47:24 | 36 |
| 57 | 46.75 | Palo Alto | 41:01.5 | 58 |
| 378 | 45.15 | San Jose | 0:24:16 | 1 |
| 130 | 45.15 | San Jose | 0:59:25 | 143 |
| 400 | 42.92 | San Jose | 1:08:15 | 201 |
| 408 | 37.12 | Mountain View | 20:39.5 | 243 |
| 335 | 36.94 | Mountain View | 0:38:38 | 257 |
| 177 | 34.76 | San Jose | 1:40:50 | 277 |
| 250 | 31.83 | Palo Alto | 0:36:36 | 117 |
| 368 | 31.5 | Mountain View | 1:22:22 | 245 |
| 225 | 28.73 | Palo Alto | 0:39:48 | 65 |
| 116 | 26.06 | Palo Alto | 0:45:14 | 105 |
| 226 | 25.52 | Palo Alto | 1:04:25 | 197 |
| 363 | 23.35 | San Jose | 0:44:10 | 19 |
| 338 | 23.35 | Mountain View | 0:41:36 | 12 |
| 260 | 18.94 | Palo Alto | 0:59:05 | 29 |
| 25 | -3 | Palo Alto | 1:41:04 | 190 |
| 210 | -11.1 | Palo Alto | 1:07:02 | 91 |

**High Ranking Restaurants against Avg Delivery Time**
You can see that the higher revenue generating restaurants tend to have a much higher ranking in terms of average delivery time

 ``` sql
WITH p AS (
  SELECT restaurant_id, delivery_region, sum(order_total-refunded_amount) as total, ntile(20) OVER (order by sum(order_total-refunded_amount) ) as ntile, avg(total_time) as avgdelivery
  FROM fooddelivery
  GROUP BY restaurant_id,delivery_region
), r AS (SELECT restaurant_id, dense_rank() OVER (ORDER BY AVG(total_time)) as ranking FROM fooddelivery GROUP BY restaurant_id )
SELECT p.restaurant_id, p.total,p.delivery_region, p.avgdelivery, r.ranking
FROM p
LEFT JOIN r ON p.restaurant_id= r.restaurant_id
WHERE ntile = 20
ORDER BY total DESC;

  ```
| restaurant_id | total | delivery_region | avgdelivery | ranking |
| --- | --- | --- | --- | --- |
| 9 | 38846.96 | Palo Alto | 44:07.8 | 279 |
| 8 | 37716.77 | Palo Alto | 25:46.5 | 252 |
| 20 | 31697.43 | Palo Alto | 15:09.5 | 230 |
| 63 | 26671.6 | Palo Alto | 16:19.4 | 253 |
| 107 | 25549.49 | Palo Alto | 06:32.3 | 195 |
| 10 | 21776.36 | Palo Alto | 35:28.0 | 271 |
| 68 | 19101.26 | Palo Alto | 01:44.7 | 167 |
| 12 | 15319.42 | Palo Alto | 14:28.8 | 228 |
| 3 | 14308.12 | Palo Alto | 43:05.1 | 281 |
| 5 | 12787.61 | Palo Alto | 10:11.0 | 214 |
| 28 | 11622.84 | Palo Alto | 11:38.2 | 221 |
| 27 | 10864.67 | Palo Alto | 16:12.6 | 232 |
| 98 | 10603.74 | Mountain View | 04:04.5 | 179 |
| 83 | 10545.24 | Palo Alto | 27:55.2 | 258 |
| 19 | 10025.15 | Palo Alto | 28:55.4 | 296 |
| 205 | 9711.67 | Palo Alto | 18:37.1 | 240 |
| 100 | 9647.06 | Palo Alto | 07:45.3 | 198 |
| 47 | 9460.4 | Palo Alto | 39:33.5 | 275 |
| 6 | 9294.8 | Palo Alto | 05:20.1 | 288 |
| 62 | 9178.59 | Palo Alto | 0:56:32 | 168 |
| 194 | 9004.15 | San Jose | 49:45.8 | 54 |
  
     
## Analysis of Orders by Time of Day

**Highest Median Order Values by Hour**

 ``` sql
SELECT date_part('hour',customer_order_timestamp), percentile_cont(.5) WITHIN GROUP
(ORDER BY order_total ) FROM fooddelivery 
GROUP BY 1 
ORDER BY 2 DESC


  ```
 **Distinguishing by Immediate vs Scheduled Orders** 

**Immediate Orders**
 ``` sql
SELECT date_part('hour',customer_order_timestamp), percentile_cont(.5) WITHIN GROUP
(ORDER BY order_total ) FROM fooddelivery 
WHERE is_asap= 'TRUE'
GROUP BY 1 
ORDER BY 2 DESC

  ```
| date_part | percentile_cont |
| --- | --- |
| 16 | 178.66 |
| 15 | 76.14 |
| 17 | 74.35 |
| 1 | 41.695 |
| 0 | 40.69 |
| 2 | 38.03 |
| 23 | 36.94 |
| 3 | 34.66 |
| 4 | 32.64 |
| 18 | 31.45 |
| 19 | 29.93 |
| 20 | 28.35 |
| 21 | 27.1 |
| 22 | 22.815 |
| 5 | 21.715 |

**Scheduled Orders**
 ``` sql
SELECT date_part('hour',customer_order_timestamp), percentile_cont(.5) WITHIN GROUP
(ORDER BY order_total ) FROM fooddelivery 
WHERE is_asap= 'FALSE'
GROUP BY 1 
ORDER BY 2 DESC

  ```
| date_part | percentile_cont |
| --- | --- |
| 8 | 435.2 |
| 5 | 348.52 |
| 10 | 327.12 |
| 7 | 242.91 |
| 6 | 202.31 |
| 4 | 86.21 |
| 12 | 70.23 |
| 16 | 69.405 |
| 15 | 66.195 |
| 17 | 61.98 |
| 20 | 61.08 |
| 3 | 57.75 |
| 11 | 51.81 |
| 0 | 51.675 |
| 22 | 50.26 |
| 18 | 50.015 |
| 14 | 47.765 |
| 23 | 46.645 |
| 21 | 44.99 |
| 1 | 44.55 |
| 13 | 42.52 |
| 19 | 41.83 |
| 2 | 41.17 |
| 9 | 19.63 |
  
     
### Scheduled vs Immediate Delivery out of Total Orders
Comparing scheduled vs immediate delivery order count share and avg order value

 ``` sql
SELECT 
    (SELECT COUNT(*) FROM fooddelivery WHERE Is_ASAP = 'FALSE') * 100.0 / COUNT(*) AS scheduled_delivery,
	(SELECT AVG(order_total) FROM fooddelivery WHERE Is_ASAP = 'FALSE' ) as scheduled_delivery_avg_order,
    (SELECT COUNT(*) FROM fooddelivery WHERE Is_ASAP = 'TRUE') * 100.0 / COUNT(*) AS immediate_delivery,
	(SELECT AVG(order_total) FROM fooddelivery WHERE Is_ASAP = 'TRUE' ) as immediate_delivery_avg_order,
    COUNT(*) AS total_orders
FROM fooddelivery;

  ```
| scheduled_delivery | scheduled_delivery_avg_order | immediate_delivery | immediate_delivery_avg_order | total_orders |
| --- | --- | --- | --- | --- |
| 20.15157 | 80.16231 | 79.84843 | 43.96772 | 18078 |
  
   
### Order Frequency per Hour

 ``` sql
SELECT EXTRACT(hour FROM customer_order_timestamp) as Hour, count(*) as 
"no. of orders"
FROM fooddelivery
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
  ```
| hour | no. of orders |
| --- | --- |
| 1 | 3373 |
| 0 | 2880 |
| 2 | 2843 |
| 3 | 1764 |
| 18 | 1418 |
| 19 | 1198 |
| 23 | 1146 |
| 20 | 889 |
| 17 | 820 |
| 4 | 656 |
| 22 | 361 |
| 21 | 328 |
| 16 | 249 |
| 15 | 73 |
| 5 | 26 |
| 14 | 16 |
| 6 | 12 |
| 7 | 9 |


We see that people tend to order more frequently late at night as a midnight meal.

### Highest Average Order Total by Hour

 ``` sql
SELECT EXTRACT(hour FROM customer_order_timestamp) as Hour, avg((order_total)) as 
"avg order value"
FROM fooddelivery
GROUP BY 1
ORDER BY 2 DESC
  ```
| hour | avg order value |
| --- | --- |
| 6 | 435.645 |
| 7 | 391.5211 |
| 10 | 327.12 |
| 8 | 314.9933 |
| 5 | 280.6058 |
| 16 | 101.957 |
| 17 | 93.89315 |
| 15 | 88.50466 |
| 14 | 79.08 |
| 12 | 70.23 |
| 22 | 66.0928 |
| 23 | 57.04955 |
| 11 | 51.81 |

We see that strangely 6am ~ 8am and 10am have significantly higher average order values. We will look into the orders from these times to understand why this is the case.

**Investigate the strange 6am high average order total**
 ``` sql
SELECT * FROM (SELECT *, EXTRACT(hour FROM customer_order_timestamp) as Hour
FROM fooddelivery) b WHERE Hour in (6,7,8,10)
ORDER BY order_total DESC
  ```
  
Results will show that they are all scheduled orders. Thus, we will find the highest avg order total without scheduled orders. 

**Adjusted avg order total per hour**
 ``` sql
SELECT EXTRACT(hour FROM customer_order_timestamp) as Hour, avg((order_total)) as 
"avg order value"
FROM fooddelivery
WHERE IS_ASAP = 'TRUE'
GROUP BY 1
ORDER BY 2 DESC

  ```
| hour | avg order value |
| --- | --- |
| 16 | 178.66 |
| 15 | 76.14 |
| 17 | 74.35 |
| 1 | 48.67216 |
| 0 | 48.17345 |
| 23 | 46.41321 |
| 2 | 43.92599 |
| 18 | 43.70335 |
| 3 | 40.634 |
| 19 | 38.4439 |
| 4 | 36.69604 |
| 20 | 34.89708 |
| 21 | 32.59519 |
| 5 | 26.48125 |
| 22 | 24.09333 |

Now we see that 4pm is the time period where average order total is much higher. However another interesting insight from this table is that morning orders are all scheduled orders.

## Analysis of Drivers

### Calcuating median tips for drivers by the hour
**What hour of the day do drivers recieve the most tips?**

 ``` sql

SELECT date_part('hour',customer_order_timestamp), percentile_cont(.5) WITHIN GROUP
(ORDER BY amount_of_tip)  FROM fooddelivery 

GROUP BY 1 
ORDER BY 2 DESC
  ```

| date_part | percentile_cont |
| --- | --- |
| 8 | 7.91 |
| 7 | 7.65 |
| 12 | 4.855 |
| 16 | 4.68 |
| 15 | 4.19 |
| 11 | 3.655 |
| 14 | 3.465 |
| 17 | 3.375 |
| 22 | 3.19 |
| 23 | 2.99 |
| 9 | 2.94 |
| 1 | 2.8 |
| 0 | 2.79 |
| 2 | 2.5 |
| 13 | 2.47 |
| 18 | 2.34 |
| 3 | 2.16 |
| 21 | 2.095 |
| 19 | 1.99 |


  ### Hourly Tip per Driver
**Which driver earns the most amount of tips per hour on the average?**

 ``` sql
With main as (SELECT driver_id, ABS(SUM(hours_worked)) as totalhours, SUM(amount_of_tip) as totaltip FROM (select driver_id, EXTRACT(epoch FROM driver_delivery_time)/3600 AS hours_worked, amount_of_tip From driverdelivery) sub GROUP BY driver_id
ORDER BY 2 DESC)

SELECT driver_id, (totaltip/totalhours) tiphourly, totalhours FROM main 
WHERE totalhours >10 
ORDER BY 2 DESC 
LIMIT 10




  ```
| driver_id | tiphourly | totalhours |
| --- | --- | --- |
| 329 | 13.02963 | 18.58917 |
| 87 | 12.36148 | 15.95278 |
| 139 | 12.29624 | 16.24806 |
| 97 | 12.26799 | 25.61056 |
| 116 | 12.12354 | 17.99639 |
| 157 | 12.04275 | 12.39833 |
| 49 | 11.92401 | 19.82806 |
| 24 | 11.81643 | 11.73028 |
| 337 | 11.66848 | 18.02806 |
| 112 | 11.61889 | 19.56556 |



  ### Fastest median driver delivery time in hour
**Finding out what hours of the day do drivers deliver the food the fastest**


 ``` sql

SELECT date_part('hour', customer_order_timestamp) as order_hour, 
percentile_cont(.5) WITHIN GROUP
(ORDER BY driver_delivery_time ) 
as median_time FROM fooddelivery 
GROUP BY 1
order by 2 
LIMIT 10

  ```
| order_hour | median_time |
| --- | --- |
| 14 | 0:18:43 |
| 13 | 19:51.5 |
| 9 | 0:20:09 |
| 17 | 0:20:28 |
| 8 | 20:28.5 |
| 20 | 21:07.5 |
| 23 | 0:21:38 |
| 21 | 21:46.5 |
| 22 | 21:47.5 |
| 3 | 0:21:55 |


 Output shows that drivers are on median, the fastest in the afternoon at around 1 or 2pm

##Exploring refunds

**What hours of the days experience the most refund requests proportionally to total orders in that hour?
 ``` sql
SELECT 
  date_part('hour', delivered_to_consumer_timestamp) AS hour,
  COUNT(CASE WHEN refunded_amount > 0 THEN 1 END)::FLOAT / COUNT(*) AS refunded_orders_ratio
FROM 
  fooddelivery
GROUP BY 
  hour
ORDER BY 
  refunded_orders_ratio DESC;


  ```

| hour | refunded_orders_ratio |
| --- | --- |
| 6 | 0.5 |
| 18 | 0.046154 |
| 19 | 0.036496 |
| 22 | 0.033113 |
| 20 | 0.029249 |
| 1 | 0.028081 |
| 5 | 0.027322 |
| 0 | 0.0273 |
| 4 | 0.025268 |
| 21 | 0.02451 |
| 2 | 0.022104 |
| 3 | 0.021959 |
| 23 | 0.008547 |


6 am was found to only have 12 orders in the query just now that showed order count by hour. Thus, the value of 0.5 is not very insightful considering the low amount of orders. However, an interesting insight is that orders from  dinner time hours of 6-8pm seem to see quite a high refund rate.

### Slowest Drivers against Drivers with Highest Refund Rate
**Investigating whether speed of delivery is a contributing factor by comparing driver's delivery speed against driver's refund request rate**

 ``` sql
With driver_refunds as (SELECT driver_id, sum(refunded_amount), count(refunded_amount) FROM fooddelivery WHERE is_asap ='TRUE' AND refunded_amount > 1
GROUP BY driver_id
ORDER BY count DESC LIMIT 20), driveravgtime as (SELECT driver_id, avg(driver_delivery_time) FROM fooddelivery GROUP BY driver_id ORDER BY 2 LIMIT 20) 

SELECT count (*) FROM driver_refunds INNER JOIN driveravgtime ON driver_refunds.driver_id= driveravgtime.driver_id

```

Only 2 drivers are in the slowest average time and highest refund count list. There does not seem to be a significant overlap between these 2 characteristics

### Correlation between total time taken and refund count

 ``` sql
With i as (SELECT time_ntile, count(refunded_amount) as ref
FROM(SELECT total_time, NTILE(100) OVER (ORDER BY total_time) as time_ntile, refunded_amount, is_asap
FROM fooddelivery) as x
WHERE refunded_amount > 0 AND is_asap= 'TRUE'
GROUP BY time_ntile
ORDER by 1)

SELECT corr(time_ntile, ref)
FROM i
```

Correlation comes out to 0.22 thus signifying that there is not actually a large correlation between time taken for a delivery and refund requests.
