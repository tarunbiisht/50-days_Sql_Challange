# 50-days_Sql_Challange


--Day 1 to Day 10

select * from customers;

select * from orders;
select * from products;
select * from returns;
select * from sellers;

CREATE TABLE customers (
                            customer_id VARCHAR(25) PRIMARY KEY,
                            customer_name VARCHAR(25),
                            state VARCHAR(25)
);

CREATE TABLE sellers (
                        seller_id VARCHAR(25) PRIMARY KEY,
                        seller_name VARCHAR(25)
);
CREATE TABLE products (
                        product_id VARCHAR(25) PRIMARY KEY,
                        product_name VARCHAR(255),
                        Price int,
                        cogs int
);
CREATE TABLE orders (
                        order_id VARCHAR(25) PRIMARY KEY,
                        order_date DATE,
                        customer_id VARCHAR(25),  -- this is a foreign key from customers(customer_id)
                        state VARCHAR(25),
                        category VARCHAR(25),
                        sub_category VARCHAR(25),
                        product_id VARCHAR(25),   -- this is a foreign key from products(product_id)
                        price_per_unit FLOAT,
                        quantity INT,
                        sale FLOAT,
                        seller_id VARCHAR(25),    -- this is a foreign key from sellers(seller_id)
    
CONSTRAINT fk_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
CONSTRAINT fk_products FOREIGN KEY (product_id) REFERENCES products(product_id),    
CONSTRAINT fk_sellers FOREIGN KEY (seller_id) REFERENCES sellers(seller_id)
);
CREATE TABLE returns (
                       return_id VARCHAR(25) PRIMARY KEY,
                       order_id VARCHAR(25),
                        CONSTRAINT fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);





-- Business Problem:
-- Ques1 :what are the total sales made by each customer?
select 
customer_id,
sum(sale) as Total_sales
from orders
group by 1;

--  Ques2 : How many orders were placed in each state?
 select state,
count(order_id) as Total_orders
from orders
group by state;

--  ques 3: how many unique products were sold?
select count(distinct product_id) as Total_unique_product
from orders

-- ques4: How many returns were made for each product category.

select  o.category,
count(r.return_id)as no_of_returns
from orders o
join
returns r on 
o.order_id = r.order_id
group by 1;


-- ques5: How many orders were placed in each month(2022)

select current_date;

select Extract(month from order_date) as month,
count(order_id)as total_orders
from orders
where 
extract(year from order_date) =2022
group by 1;

-- ques 6: Determine the top 3 products whose revenue has decreased to the previous year?

with Last_year_rev as (
	
select product_id,
sum(sale)as Total_sale
from orders
where
extract(year from order_date) =2022
group by
product_id
),
	
Current_year_rev as (
select product_id,
sum(sale)as Total_sale
from orders
where
extract(year from order_date) =2023
group by
product_id
)
select l.product_id,
l.total_sale as Last_year_rev,
c.total_sale as Current_year_rev,
((l.total_sale - c.total_sale)/ l.total_sale) * 100 as Rev_descrease
 from last_year_rev l
join
Current_year_rev c
ON
l.product_id= c.product_id
order by rev_descrease desc
 limit 3;

--ques7-List all orders where the quantity sold is greater than the average quantity sold across all order
select * from orders
WHERE
quantity > (select avg(quantity) from orders);

-- Ques8 - Find out the top 5 customers who made the highest profit?
select  o.customer_id,
sum((o.price_per_unit - p.cogs) * o.quantity) as Total_sale
from orders o
 join 
products p
on o.product_id = p.product_id
group by o.customer_id
	order by Total_sale desc;


-- Ques 9-Find details of the top 5 products with the highest total sale, where the total sale for EACH
          products is greater than the avg sale across all products?
 
		select p.product_id,
			  p.product_name,
        sum(o.sale) as Total_sale from
orders o
join
products p
on
o.product_id= p.product_id
group by p.product_id,
		 p.product_name
HAVING
sum(o.sale) > (select sum(sale)/ count(distinct product_id) from orders)
			  order by total_sale DESC
			  limit 5;

-- Ques 10 Calculate Profit margin percentage for each sale.
select order_id,
	product_name,
((Sum(o.price_per_unit - p.cogs) * o.quantity) / sum(o.sale) * 100) as Profit_percentage
from orders o
join products p 
on o.product_id =p.product_id
group by order_id,
	product_name;




      
      







