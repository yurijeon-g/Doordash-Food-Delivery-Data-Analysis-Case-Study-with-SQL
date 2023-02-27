# Food-Delivery-Data-Analysis-Case-Study-with-SQL
Using a dataset of Doordash found on Kaggle, I exercised my time analysis skills to manipulate and uncover insights about food delivery intervals as well as insights on driver income, hourly trends in high demand for food delivery as well as identified high revenue grossing restaurants and analyzed contributing factors to high grossing restaurants. 


## Introduction
I will be analyzing the "fooddelivery" dataset, which contains information about doordash food delivery orders placed by consumers in a certain regions. The dataset includes information about the order, such as the restaurant and driver involved, as well as timestamps for when the order was placed and when it was delivered.

Throughout this project, I will be exploring the data to uncover insights and trends that can help improve the food delivery service for both consumers and drivers. By analyzing the dataset, I hope to answer questions such as: What are the most popular restaurants in the region? How long does it typically take for an order to be delivered? Are there any factors that influence whether an order is refunded?

## About the Data: 
Below is a glossary of definitions for the variables included in the data set that may need clarification.

Customer placed order datetime: Time that customer placed the order; the format is dd: hh: mm: ss

Placed order with restaurant datetime: Time that restaurant received order; the format is dd: hh: mm: ss

Driver at restaurant datetime: Time that driver arrives at restaurant; the format is dd: hh: mm: ss

Delidered to consumer datetime: Time that driver delivered to customer; the format is dd: hh: mm: ss

Driver ID: Unique identifier of driver
Restaurant ID: Unique identifier of restaurant
Consumer ID: Unique identifier of customer
Delivery Region: City where restaurant is located

Is ASAP: Equals TRUE for on-demand orders; FALSE for scheduled deliveries (e.g., a customer places an order at 10am for 12noon)

Order total: Amount customer spent (including delivery fee); units are in dollars

Amount discount: Amount of discounts redeemed (e.g., for referrals); units are in dollars

Amount of tip: Amount of tip given; units are in dollars

Refunded amount: Amount refunded to customer; units are in dollars

Times: Time is in UTC 



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
The query shows this inconsistency

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

**In the case where the delivery of the food is recorded as an earlier date than when the customer ordered **

 ``` sql
UPDATE fooddelivery
SET
    order_with_restaurant_timestamp = to_timestamp(
        CASE WHEN customer_order_timestamp > order_with_restaurant_timestamp
             THEN to_char(order_with_restaurant_timestamp + INTERVAL '1 month', 'YYYY-02-DD HH24:MI:SS')
             ELSE to_char(order_with_restaurant_timestamp, 'YYYY-02-DD HH24:MI:SS')
        END,
        'YYYY-MM-DD HH24:MI:SS'
    ),
    driver_at_restaurant_timestamp = to_timestamp(
        CASE WHEN customer_order_timestamp > driver_at_restaurant_timestamp
             THEN to_char(driver_at_restaurant_timestamp + INTERVAL '1 month', 'YYYY-02-DD HH24:MI:SS')
             ELSE to_char(driver_at_restaurant_timestamp, 'YYYY-02-DD HH24:MI:SS')
        END,
        'YYYY-MM-DD HH24:MI:SS'
    ),
    delivered_to_consumer_timestamp = to_timestamp(
        CASE WHEN customer_order_timestamp > delivered_to_consumer_timestamp
             THEN to_char(delivered_to_consumer_timestamp + INTERVAL '1 month', 'YYYY-02-DD HH24:MI:SS')
             ELSE to_char(delivered_to_consumer_timestamp, 'YYYY-02-DD HH24:MI:SS')
        END,
        'YYYY-MM-DD HH24:MI:SS'
    )
WHERE customer_order_timestamp > delivered_to_consumer_timestamp;


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

| restaurant_id | total | delivery_region | avgdelivery | ranking |
| --- | --- | --- | --- | --- |
| 303 | 51.24 | Palo Alto | 1:11:00 | 225 |
| 392 | 50.91 | San Jose | 1:22:50 | 219 |
| 323 | 49.45 | San Jose | 1:18:57 | 215 |
| 383 | 49.36 | San Jose | 04:38.5 | 178 |
| 151 | 48.37 | San Jose | 0:47:24 | 35 |
| 57 | 46.75 | Palo Alto | 41:01.5 | 56 |
| 130 | 45.15 | San Jose | 0:59:25 | 138 |
| 378 | 45.15 | San Jose | 0:24:16 | 1 |
| 400 | 42.92 | San Jose | 1:08:15 | 187 |
| 408 | 37.12 | Mountain View | 20:39.5 | 217 |
| 335 | 36.94 | Mountain View | 0:38:38 | 227 |
| 177 | 34.76 | San Jose | 1:40:50 | 241 |
| 250 | 31.83 | Palo Alto | 0:36:36 | 112 |
| 368 | 31.5 | Mountain View | 1:22:22 | 218 |
| 225 | 28.73 | Palo Alto | 0:39:48 | 63 |
| 116 | 26.06 | Palo Alto | 0:45:14 | 312 |
| 226 | 25.52 | Palo Alto | 1:04:25 | 185 |
| 338 | 23.35 | Mountain View | 0:41:36 | 12 |
| 363 | 23.35 | San Jose | 0:44:10 | 19 |
| 260 | 18.94 | Palo Alto | 0:59:05 | 29 |
| 25 | -3 | Palo Alto | 1:41:04 | 181 |
| 210 | -11.1 | Palo Alto | 366 days 01:07:02 | 314 |


  ```


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
| 9 | 38846.96 | Palo Alto | 03:19.0 | 274 |
| 8 | 37716.77 | Palo Alto | 14:09.7 | 272 |
| 20 | 31697.43 | Palo Alto | 31:14.9 | 273 |
| 63 | 26671.6 | Palo Alto | 16:19.4 | 224 |
| 107 | 25549.49 | Palo Alto | 2 days 06:46:44.498982 | 285 |
| 10 | 21776.36 | Palo Alto | 1 day 23:42:04.88189 | 283 |
| 68 | 19101.26 | Palo Alto | 01:44.7 | 160 |
| 12 | 15319.42 | Palo Alto | 1 day 14:27:42.023305 | 278 |
| 3 | 14308.12 | Palo Alto | 1 day 10:44:26.330827 | 276 |
| 5 | 12787.61 | Palo Alto | 3 days 20:11:44.26943 | 293 |
| 28 | 11622.84 | Palo Alto | 1 day 13:02:49.269388 | 277 |
| 27 | 10864.67 | Palo Alto | 3 days 02:46:35.171548 | 288 |
| 98 | 10603.74 | Mountain View | 1 day 20:46:09.895522 | 279 |
| 83 | 10545.24 | Palo Alto | 1 day 17:56:40.502304 | 280 |
| 19 | 10025.15 | Palo Alto | 28:55.4 | 255 |
| 205 | 9711.67 | Palo Alto | 1 day 20:09:32.741463 | 281 |
| 100 | 9647.06 | Palo Alto | 2 days 05:06:20.047337 | 284 |
| 47 | 9460.4 | Palo Alto | 39:33.5 | 239 |
| 6 | 9294.8 | Palo Alto | 1 day 22:13:46.623116 | 282 |
| 62 | 9178.59 | Palo Alto | 3 days 18:34:29.55102 | 299 |
| 194 | 9004.15 | San Jose | 49:45.8 | 52 |  
  
     
## Analysis of Orders by Time of Day

**Highest Median Order Values by Hour**

 ``` sql
SELECT date_part('hour',customer_order_timestamp), percentile_cont(.5) WITHIN GROUP
(ORDER BY order_total ) FROM fooddelivery 
GROUP BY 1 
ORDER BY 2 DESC


  ```
 **Distinguishing by Immediate vs Scheduled Orders** 


 ``` sql
SELECT date_part('hour',customer_order_timestamp), percentile_cont(.5) WITHIN GROUP
(ORDER BY order_total ) FROM fooddelivery 
WHERE is_asap= 'TRUE'
GROUP BY 1 
ORDER BY 2 DESC

  ```
| date_part | percentile_cont |
| --- | --- |
| 0 | 127.4 |
| 1 | 74.35 |
| 9 | 41.72 |
| 8 | 40.19 |
| 10 | 39.44 |
| 11 | 36.075 |
| 12 | 34.7 |
| 2 | 33.14 |
| 7 | 32.315 |
| 13 | 31.5 |
| 3 | 30.615 |

 ``` sql
SELECT date_part('hour',customer_order_timestamp), percentile_cont(.5) WITHIN GROUP
(ORDER BY order_total ) FROM fooddelivery 
WHERE is_asap= 'FALSE'
GROUP BY 1 
ORDER BY 2 DESC

  ```
| date_part | percentile_cont |
| --- | --- |
| 14 | 348.52 |
| 16 | 293.095 |
| 12 | 121.935 |
| 13 | 103.765 |
| 0 | 81.96 |
| 20 | 70.23 |
| 17 | 68.53 |
| 1 | 68.48 |
| 11 | 54.44 |
| 2 | 54.29 |
  
     
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

  ```
  
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
| 16 | 7.78 |
| 0 | 5.06 |
| 20 | 4.855 |
| 22 | 4 |
| 1 | 3.81 |
| 23 | 3.645 |


  ### Hourly Tip per Driver
**Which driver earns the most amount of tips per hour on the average?**

 ``` sql
With main as (SELECT driver_id, ABS(SUM(hours_worked)) as totalhours, SUM(amount_of_tip) as totaltip FROM (select driver_id, EXTRACT(epoch FROM driver_delivery_time)/3600 AS hours_worked, amount_of_tip From driverdelivery) sub GROUP BY driver_id
ORDER BY 2 DESC)

SELECT driver_id, (totaltip/totalhours) tiphourly, totalhours FROM main WHERE totalhours >10 ORDER BY 2 DESC 


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


  ### XX

 ``` sql
SELECT EXTRACT(hour FROM customer_order_timestamp) as Hour, count(*) as 
"no. of orders"
FROM fooddelivery
GROUP BY 1
ORDER BY 2 DESC

  ```
  
  ### XX

 ``` sql
SELECT EXTRACT(hour FROM customer_order_timestamp) as Hour, count(*) as 
"no. of orders"
FROM fooddelivery
GROUP BY 1
ORDER BY 2 DESC

  ```
