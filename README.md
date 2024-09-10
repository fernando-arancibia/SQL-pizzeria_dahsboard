# SQL Portfolio Project: Pizzeria Dashboard

This project demonstrates how to build a data analyst portfolio using PostgreSQL, focused on creating the database analyzing orders, managing stock levels, and calculating labor costs for a fictional pizzeria. 
The SQL queries include essential operations like handling complex joins, aggregations, and subqueries to produce insights for a business intelligence dashboard.
The project was created in SQL language with PostgreSQL software to create databases and queries, Google Cloud was used to create an instance connection between PostgreSQL and Google Data Studio. 
Data Studio was used to create visualizations and dashboard reports.

# Project overview

## PostgreSQL

The database was created first with schemas and then by adding data with CSV files. Here is the Entity Relational Diagram (EDR) of the structure of the tables of the database.

![image](https://github.com/user-attachments/assets/59062cac-1616-445d-8072-fcc6bc95cd57)

The database consists of 9 tables as follows:
1. orders
2. customer
3. address
4. item
5. recipe
6. ingredient
7. inventory
8. rota
9. shift
10. staff

The data was inserted in every table, is precise to mention that data was obtained from learnBi.academy

This project showcases SQL querying skills by:

Tracking Orders and Sales.
Managing Stock Levels and Inventory Costs.
Analyzing Staff Schedules and Labor Costs.

## Key Features

Custom SQL Queries: Complex queries for aggregating data, including subqueries, joins, and calculations.
Business Intelligence (BI) Ready: Data is formatted for use in BI tools, in this case Google Data Studio to build client dashboards.

## SQL Queries Breakdown

Order quantity: Calculates total orders based orders, item, and address tables.

```sql
Select
o.order_id,
i.item_price,
o.quantity,
i.item_cat,
i.item_name,
o.created_at,
a.delivery_address1,
a.delivery_address2,
a.delivery_city,
a.delivery_zipcode,
o.delivery
FROM orders o
left join item i ON o.item_id = i.item_id
left join address a ON o.add_id = a.add_id;
```
This query corresponds to Dashboard 1 and response to the following insights:

1. Total sales.
2. Total items.
3. Total orders.
4. Average order value.
5. Sales by category.
6. Top selling items.
7. Order by hour.
8. Sales by hour.
9. Orders by address.
10. Orders by delivery/pickup.

Stock Inventory: this query is elaborated with orders, items and ingredient tables an elaborate sub-queries s1 to create some calculations.

```sql
SELECT
	`s1`.`item_name` AS `item_name`,
	`s1`.`ing_name` AS `ing_name`,
	`s1`.`ing_id` AS `ing_id`,
	`s1`.`ing_weight` AS `ing_weight`,
	`s1`.`ing_price` AS `ing_price`,
	`s1`.`order_quantity` AS `order_quantity`,
	`s1`.`recipe_quantity` AS `recipe_quantity`,(
		`s1`.`order_quantity` * `s1`.`recipe_quantity` 
		) AS `ordered_weight`,(
		`s1`.`ing_price` / `s1`.`ing_weight` 
		) AS `unit_cost`,((
			`s1`.`order_quantity` * `s1`.`recipe_quantity` 
		) * ( `s1`.`ing_price` / `s1`.`ing_weight` )) AS `ingredient_cost` 
FROM
	(
	SELECT
		`o`.`item_id` AS `item_id`,
		`i`.`sku` AS `sku`,
		`i`.`item_name` AS `item_name`,
		`r`.`ing_id` AS `ing_id`,
		`ing`.`ing_name` AS `ing_name`,
		`ing`.`ing_weight` AS `ing_weight`,
		`ing`.`ing_price` AS `ing_price`,
		sum( `o`.`quantity` ) AS `order_quantity`,
		`r`.`quantity` AS `recipe_quantity` 
	FROM
		(((
					`orders` `o`
					LEFT JOIN `item` `i` ON ((
							`o`.`item_id` = `i`.`item_id` 
						)))
				LEFT JOIN `recipe` `r` ON ((
						`i`.`sku` = `r`.`recipe_id` 
					)))
			LEFT JOIN `ingredient` `ing` ON ((
					`ing`.`ing_id` = `r`.`ing_id` 
				))) 
	GROUP BY
		`o`.`item_id`,
		`i`.`sku`,
		`i`.`item_name`,
		`r`.`ing_id`,
		`r`.`quantity`,
		`ing`.`ing_weight`,
	`ing`.`ing_price` 
	) `s1`
```

This query stock1(s1) corresponds to Dashboard 2 and responds to:

1. Total quantity by ingredient.
2. Total cost of ingredients.
3. Calculated cost of pizza.

and additional subquery is elaborated for dashboard 2

```SQL
select
s2.ing_name,
s2.ordered_weight,
ing.ing_weight,
inv.quantity,
ing.ing_weight * inv.quantity AS total_inv_weight 
FROM
	( SELECT ing_id, ing_name, sum( ordered_weight ) AS ordered_weight FROM stock1 GROUP BY ing_name, ing_id ) s2
	LEFT JOIN inventory inv ON inv.item_id = s2.ing_id
	LEFT JOIN ingredient ing ON ing.ing_id = s2.ing_id
```
This query stock2(S2) respond to following answers:

1. Stock remaining by ingredient.
2. Re-order ingredient list based on inventory .

Staff: This query is elaborated with rota, staff, and shift tables.

```sql
SELECT
	r.date,
	s.first_name,
	s.last_name,
	s.hourly_rate,
	sh.start_time,
	sh.end_time,
	EXTRACT(
    epoch FROM (
      sh.start_time - sh.end_time
    )
)/3600
FROM
	rota r
	LEFT JOIN staff s ON r.staff_id = s.staff_id
	LEFT JOIN shift sh ON r.shift_id = sh.shift_id
```
The Staff query brings the following information:

1. Staff data. (Names, Date, and Hourly Rate)
2. Shift data.
3.  Calculate staff cost.

## Tools Used
PostgreSQL: For querying and data manipulation.
PGadmin SQL tool: For SQL query development.
BI Tools: Connect SQL data to visualization tools like Power BI or Tableau to build the final dashboard. In this case, Google Data Studio was used using an instance connection with Google Cloud. 
Link to report: https://lookerstudio.google.com/s/qUfBDf9zDPs

## How to Use
Clone the repository and load the provided SQL dump file.
Run the queries to calculate stock levels, staff costs, and other business metrics.
Connect your BI tool to the SQL database to visualize the results.



