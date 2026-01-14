Use excel_task;

##To understand the structure and sample data of mechanics.
SELECT * FROM mechanics_data LIMIT 10;
 
 #How much revenue did the company generate after discounts?
SELECT 
    SUM((item_selling_price * quantity) - total_discount) AS total_revenue
FROM sales_data;


##What is the actual quantity sold after returns?
select (sum(quantity-return_quantity)) As net_quantity 
from sales_data ;

 
##Which products generate the highest profit?
SELECT 
    s.sku_id,
    ROUND(
        SUM(
            (sl.item_selling_price - s.average_purchase_price) 
            * (sl.quantity - sl.return_quantity)
        ), 
        1
    ) AS total_profit
FROM sales_data sl
JOIN stock_data s 
    ON s.sku_id = sl.sku_id
GROUP BY s.sku_id
ORDER BY total_profit DESC;


##Which products receive the highest discounts?
select sku_id,
round(sum(item_selling_price*quantity), 1) As gross_sales,
round(SUM(toTal_discount), 1) as total_discount 
from sales_data 
group by sku_id 
order by total_discount desc ; 


##Which products have high return rates indicating possible quality issues?
Select sku_id , 
round(sum(return_quantity)*100 / sum(quantity) ,2 ) As return_rate_Quantity
from sales_data 
group by sku_id
order by return_rate_Quantity desc ;



##Which mechanics generate the highest revenue?
select 
m.mechanic_id , m.name_en,
round(
sum((item_selling_price*quantity)-total_discount ) ,0 ) As revenue 
from sales_data sl 
join mechanics_data m on sl.mechanic_id=m.mechanic_id
group by m.mechanic_id , m.name_en
order by revenue desc; 


##Are products being sold below or above the suggested price?
SELECT 
    sku_id, 
    suggested_selling_price,
    avg_actual_price, 
    price_diff, 
    CASE 
        WHEN price_diff < -10 THEN 'under_priced'
        WHEN price_diff BETWEEN -10 AND 10 THEN 'ok'
        ELSE 'overpriced'
    END AS price_status
FROM (
    SELECT 
        st.sku_id,
        st.suggested_selling_price,
        AVG(sl.item_selling_price) AS avg_actual_price, 
        AVG(sl.item_selling_price) - st.suggested_selling_price AS price_diff
    FROM stock_data st 
    JOIN sales_data sl 
        ON st.sku_id = sl.sku_id
    GROUP BY st.sku_id, st.suggested_selling_price
) t;




##How do mechanics rank based on revenue performance?
SELECT
    m.name_en,
    m.mechanic_id,
    ROUND(SUM((sl.item_selling_price * sl.quantity) - sl.total_discount), 0) AS revenue,
    DENSE_RANK() OVER (
        PARTITION BY m.name_en
        ORDER BY SUM((sl.item_selling_price * sl.quantity) - sl.total_discount) DESC
    ) AS revenue_rank_within_branch
FROM sales_data sl
JOIN mechanics_data m 
    ON sl.mechanic_id = m.mechanic_id
GROUP BY  m.name_en, m.mechanic_id
ORDER BY  revenue_rank_within_branch;


##Which mechanics offer higher discounts that may impact profitability?		
select m.mechanic_id , m.name_en,
round(
avg(s.total_discount / (s.item_selling_price * s.quantity))*100,1) as avg_discoiunt
from sales_data s
join mechanics_data m on m.mechanic_id = s.mechanic_id
GROUP BY m.mechanic_id, m.name_en
order by avg_discoiunt desc ;


##How does each mechanic’s sales performance change over time?
SELECT 
    m.mechanic_id,
    m.name_en,
    DATE_FORMAT(s.order_date, '%Y-%m') AS month,
    round(
    SUM(s.item_selling_price * s.quantity - s.total_discount),1) AS monthly_sales
FROM sales_data s
JOIN mechanics_data m ON s.mechanic_id = m.mechanic_id
GROUP BY m.mechanic_id, m.name_en, month
ORDER BY m.mechanic_id, month;


##Simplifies reporting and allows easy reuse of revenue calculations.
Create  view Mechanics_revenue as 
select 
m.mechanic_id , m.name_en,
round(
sum((item_selling_price*quantity)-total_discount ) ,0 ) As revenue 
from sales_data sl 
join mechanics_data m on sl.mechanic_id=m.mechanic_id
group by m.mechanic_id , m.name_en
order by revenue desc; 

Select * from Mechanics_revenue ;




