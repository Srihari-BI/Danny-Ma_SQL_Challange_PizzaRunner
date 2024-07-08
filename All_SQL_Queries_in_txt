#Pizza Metrices
#1
select count(order_id) from customer_orders
#2
SELECT count(distinct(customer_id)) from customer_orders
#3
select runner_id, count(order_id) from runner_orders where cancellation is not null group by 1
#4
select pizza_id, count(ro.order_id) from runner_orders ro join customer_orders co on ro.order_id = co.order_id
where cancellation is not null group by 1
#5
SELECT customer_id,
       SUM(CASE WHEN pizza_id = 1 THEN 1 ELSE 0 END) AS meatlovers_ordered,
       SUM(CASE WHEN pizza_id = 2 THEN 1 ELSE 0 END) AS vegetarian_ordered
FROM customer_orders
GROUP BY customer_id;
#6
select 
max(nop) as max_no_of_pizzas_in_order
from (select 
order_id,
count(pizza_id) as nop
from customer_orders
group by 1)sub
#7
SELECT customer_id,
       SUM(CASE WHEN exclusions IS NOT NULL OR extras IS NOT NULL THEN 1 ELSE 0 END) AS changed_pizzas,
       SUM(CASE WHEN exclusions IS NULL AND extras IS NULL THEN 1 ELSE 0 END) AS unchanged_pizzas
FROM customer_orders
GROUP BY customer_id;
#8
SELECT COUNT(*) AS pizzas_with_exclusions_and_extras
FROM customer_orders
WHERE exclusions IS NOT NULL AND extras IS NOT NULL;
#9
SELECT EXTRACT(HOUR FROM order_time) AS hour_of_day,
       COUNT(*) AS total_pizzas_ordered
FROM customer_orders
GROUP BY EXTRACT(HOUR FROM order_time)
ORDER BY hour_of_day;
#10
SELECT DAYOFWEEK(order_time) AS day_of_week,
       COUNT(*) AS total_orders
FROM customer_orders
GROUP BY DAYOFWEEK(order_time)
ORDER BY day_of_week;
#---------------------------------------------------------------------------------------------
#Runner and Customer Experience

#1
SELECT DATE_FORMAT(registration_date, '%Y-%U') AS week_start,
       COUNT(*) AS new_runners_signed_up
FROM runners
GROUP BY week_start
ORDER BY week_start;
#2
SELECT runner_id,
       AVG(TIME_TO_SEC(duration) / 60) AS avg_time_to_pickup_minutes
FROM runner_orders
WHERE pickup_time IS NOT NULL
GROUP BY runner_id;
#3
#Based on the data, it appears that there is a correlation between the number of pizzas per order and the preparation time. Orders with more pizzas tend to have longer preparation times. For instance, an order with 4 pizzas, associated with customer ID 103, had the longest preparation time of 29 minutes. In contrast, orders with fewer pizzas, such as 1 or 2 pizzas, generally have shorter preparation times ranging from 10 to 21 minutes.

#This trend suggests that as the complexity or size of the order increases (in terms of the number of pizzas), the time required for preparation also increases. This could be due to factors such as additional cooking time, handling more ingredients, or managing multiple cooking processes simultaneously.

#Understanding these patterns can help in optimizing operations, predicting preparation times more accurately, and possibly adjusting staffing or resource allocation based on the size of incoming orders.
#4
SELECT co.customer_id,
       AVG(CAST(REPLACE(ro.distance, 'km', '') AS DECIMAL(10,2))) AS avg_distance_km
FROM runner_orders ro
INNER JOIN customer_orders co ON ro.order_id = co.order_id
WHERE ro.distance IS NOT NULL
GROUP BY co.customer_id;
#5
SELECT MAX(TIME_TO_SEC(duration)) - MIN(TIME_TO_SEC(duration)) AS delivery_time_difference_seconds
FROM runner_orders
WHERE pickup_time IS NOT NULL;
#6
SELECT runner_id,
       AVG(CAST(REPLACE(distance, 'km', '') AS DECIMAL(10,2)) / (TIME_TO_SEC(duration) / 3600)) AS avg_speed_kmh
FROM runner_orders
WHERE pickup_time IS NOT NULL
GROUP BY runner_id;
#7
SELECT ro.runner_id,
       COUNT(*) AS total_orders,
       SUM(CASE WHEN ro.cancellation IS NULL THEN 1 ELSE 0 END) AS successful_orders,
       (SUM(CASE WHEN ro.cancellation IS NULL THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS success_percentage
FROM runner_orders ro
GROUP BY ro.runner_id;

#-----------------------------------------------------------------------------------------
#Ingredient Optimization
#1
SELECT pn.pizza_name, 
       GROUP_CONCAT(pt.topping_name ORDER BY pt.topping_id SEPARATOR ', ') AS standard_ingredients
FROM pizza_names pn
JOIN pizza_recipes pr ON pn.pizza_id = pr.pizza_id
JOIN pizza_toppings pt ON pr.pizza_id = pt.topping_id
GROUP BY pn.pizza_name;

#2
SELECT extras AS most_common_extra, COUNT(*) AS frequency
FROM customer_orders
WHERE extras IS NOT NULL
GROUP BY extras
ORDER BY frequency DESC
LIMIT 1;

#3
SELECT exclusions AS most_common_exclusion, COUNT(*) AS frequency
FROM customer_orders
WHERE exclusions IS NOT NULL
GROUP BY exclusions
ORDER BY frequency DESC
LIMIT 1;

#4
SELECT 
   CONCAT(
       pn.pizza_name,
       CASE WHEN co.exclusions IS NOT NULL THEN CONCAT(' - Exclude ', REPLACE(co.exclusions, ',', ', '))
            ELSE ''
       END,
       CASE WHEN co.extras IS NOT NULL THEN CONCAT(' - Extra ', REPLACE(co.extras, ',', ', '))
            ELSE ''
       END
   ) AS order_item
FROM customer_orders co
JOIN pizza_names pn ON co.pizza_id = pn.pizza_id;

#5
SELECT 
   pn.pizza_name,
   GROUP_CONCAT(
       CASE WHEN pt.topping_name IS NOT NULL THEN 
                CONCAT(
                    CASE WHEN pt.topping_id IN (1, 2, 3, 4, 5, 6, 8, 10) THEN '2x' ELSE '' END,
                    pt.topping_name
                )
       END
       ORDER BY pt.topping_name SEPARATOR ', '
   ) AS ingredient_list
FROM customer_orders co
JOIN pizza_names pn ON co.pizza_id = pn.pizza_id
LEFT JOIN pizza_recipes pr ON co.pizza_id = pr.pizza_id
LEFT JOIN pizza_toppings pt ON FIND_IN_SET(pt.topping_id, pr.toppings)
GROUP BY pn.pizza_name;

#6
SELECT pt.topping_name, COUNT(*) AS total_quantity
FROM customer_orders co
JOIN pizza_recipes pr ON co.pizza_id = pr.pizza_id
JOIN pizza_toppings pt ON FIND_IN_SET(pt.topping_id, pr.toppings)
GROUP BY pt.topping_name
ORDER BY total_quantity DESC;

#------------------------------------------------------------------------------------------------------
#Pricing and Ratings
#1
SELECT SUM(
    CASE WHEN pn.pizza_name = 'Meatlovers' THEN 12
         WHEN pn.pizza_name = 'Vegetarian' THEN 10
         ELSE 0
    END) AS total_revenue
FROM customer_orders co
JOIN pizza_names pn ON co.pizza_id = pn.pizza_id;

#2
SELECT SUM(
    CASE WHEN pn.pizza_name = 'Meatlovers' THEN 12 + IF(co.extras IS NOT NULL, 1, 0)
         WHEN pn.pizza_name = 'Vegetarian' THEN 10 + IF(co.extras IS NOT NULL, 1, 0)
         ELSE 0
    END) AS total_revenue
FROM customer_orders co
JOIN pizza_names pn ON co.pizza_id = pn.pizza_id;

#3
CREATE TABLE ratings (
    rating_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT,
    order_id INT,
    runner_id INT,
    rating INT,
    CONSTRAINT fk_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    CONSTRAINT fk_order FOREIGN KEY (order_id) REFERENCES customer_orders(order_id),
    CONSTRAINT fk_runner FOREIGN KEY (runner_id) REFERENCES runners(runner_id)
);

-- Insert sample ratings data (assuming customer_id, order_id, and runner_id are already present in their respective tables)
INSERT INTO ratings (customer_id, order_id, runner_id, rating)
VALUES
    (101, 1, 1, 4),
    (102, 2, 1, 5),
    (103, 3, 2, 3),
    (104, 4, 3, 5),
    (105, 5, 2, 4);

SELECT co.customer_id,
       co.order_id,
       ro.runner_id,
       r.rating,
       co.order_time,
       ro.pickup_time,
       TIMEDIFF(ro.pickup_time, co.order_time) AS time_between_order_and_pickup,
       ro.duration AS delivery_duration,
       ROUND(REPLACE(ro.distance, 'km', '') / TIME_TO_SEC(ro.duration) * 60, 2) AS average_speed_kmh,
       COUNT(*) AS total_pizzas
FROM customer_orders co
JOIN runner_orders ro ON co.order_id = ro.order_id
LEFT JOIN ratings r ON co.order_id = r.order_id  
GROUP BY co.customer_id, co.order_id, ro.runner_id, r.rating, co.order_time, ro.pickup_time, ro.duration, ro.distance;

#4
SELECT
    SUM(
        CASE WHEN pn.pizza_name = 'Meatlovers' THEN 12
             WHEN pn.pizza_name = 'Vegetarian' THEN 10
             ELSE 0
        END
        + IF(co.extras IS NOT NULL, 1, 0)  
        - (REPLACE(ro.distance, 'km', '') * 0.30)  
    ) AS net_profit
FROM customer_orders co
JOIN pizza_names pn ON co.pizza_id = pn.pizza_id
JOIN runner_orders ro ON co.order_id = ro.order_id;

#--------------------------------------------------------------------------------------------
#Bonus

-- Step 1: Expand Range of Pizzas

-- Insert into pizza_names
INSERT INTO pizza_names (pizza_id, pizza_name)
VALUES (3, 'Supreme');

-- Insert into pizza_recipes
INSERT INTO pizza_recipes (pizza_id, toppings)
VALUES (3, '1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12');

-- Step 2: Insert Order for "Supreme" Pizza

-- Assuming customer_id 105 exists and "Supreme" pizza has pizza_id 3
INSERT INTO customer_orders (order_id, customer_id, pizza_id, exclusions, extras, order_time)
VALUES (11, 105, 3, '', '', '2020-01-15 12:00:00');

-- Step 3: Calculate Pizza Runner's Net Profit

-- Calculate net profit after paying runners ($0.30 per km)
SELECT
    SUM(
        CASE WHEN pn.pizza_name = 'Meatlovers' THEN 12
             WHEN pn.pizza_name = 'Vegetarian' THEN 10
             ELSE 0
        END
        + IF(co.extras IS NOT NULL, 1, 0)  
        - (REPLACE(ro.distance, 'km', '') * 0.30)  
    ) AS net_profit
FROM customer_orders co
JOIN pizza_names pn ON co.pizza_id = pn.pizza_id
JOIN runner_orders ro ON co.order_id = ro.order_id;


