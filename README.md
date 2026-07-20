# Olist_ecom_analysis
Business Impact: Built an business analyst project where I have analyzed 100K+ e-commerce orders across 9 relational tables. Generated insights into revenue trends, customer behavior, seller performance, and logistics efficiency through 20+ SQL analyses and 4 interactive Power BI dashboards, enabling data driven recommendations to improve operational performance and customer satisfaction.

SQL queries:

1.KPIs

    select 
    
      (select sum(payment_value) as total_revenue 
      from olist_order_payments), 
      
      (select	count(distinct order_id) as total_orders 
      from olist_order_payments), 
      
      (select	round(avg(payment_value),2) as avg_order_value 
      from olist_order_payments), 
      
      (select count(distinct customer_id) as total_customer 
      from olist_customers);

<img width="470" height="68" alt="image" src="https://github.com/user-attachments/assets/880a06e2-966d-42cf-a15f-8bb0cb96ec54" />


2. Month on month growth percentage:

        with monthly_revenue as (select 
        	
          extract(year from oo.order_purchase_timestamp) as order_year, 
        	
          extract(month from oo.order_purchase_timestamp) as order_month, 
        	
          count(oo.order_id) as monthly_orders,
        	
          sum(oop.payment_value) as curr_month_revenue,
        	
          lag(sum(oop.payment_value)) over(order by extract(year from oo.order_purchase_timestamp), extract(month from oo.order_purchase_timestamp)) as prev_month_revenue
        
        from olist_orders oo 
        	
          join olist_order_payments oop on oo.order_id = oop.order_id 
        
        group by order_year, order_month)
        
        select
            
          order_year,
          
          order_month,
          
          monthly_orders,
        	
          curr_month_revenue,
        	
          prev_month_revenue,
        	
          round((curr_month_revenue - prev_month_revenue) * 100.0 / prev_month_revenue, 2) as mom_growth_perc
        
        from monthly_revenue;

<img width="756" height="665" alt="image" src="https://github.com/user-attachments/assets/b7c3ecc6-796d-47b2-af1e-acc0672c947f" />


3. Distribution of order status

        select  
        
          distinct order_status as status,  
        
          count(*) as total_orders,  
        
          round(100.0 * count(*) / sum(count(*)) over(),2) as percentage 
        
        from olist_orders group by status order by total_orders desc;

<img width="380" height="242" alt="image" src="https://github.com/user-attachments/assets/f68c7989-0f97-4e3f-8b11-9241cf2f51f8" />

4. Top 10 cities with most revenue

        select  
        
          oc.customer_city as city, 
        
          sum(oop.payment_value) as revenue 
        
        from olist_order_payments oop  
        
          join olist_orders oo on oo.order_id = oop.order_id 
        
          join olist_customers oc on oc.customer_id = oo.customer_id 
        
        group by city 
        
        order by revenue desc 
        
        limit 10; 

<img width="280" height="292" alt="image" src="https://github.com/user-attachments/assets/2e82c793-e6ba-43c2-960a-9d8df1dc5036" />


5. One time customer vs repeat customers:

        with order_count_table as (
        select  
        	oc.customer_unique_id, count(oo.order_id) as order_count 
        
        from olist_customers oc join olist_orders oo on oc.customer_id = oo.customer_id 
        group by oc.customer_unique_id) 
        
        select 
        	case when order_count > 1 then 'repeat_customer' else 'one_time_customer' end as customer_type, 
        	count(*) as customers
        	
        from order_count_table 
        group by customer_type;

<img width="291" height="91" alt="image" src="https://github.com/user-attachments/assets/39f25f14-c92f-4cfc-8749-6c6968036ec2" />


6. Customer with higher purchase value:

        select 
        	oc.customer_unique_id, 
        	count(distinct oop.order_id) as total_orders, 
        	sum(oop.payment_value) as total_purchase, 
        	dense_rank() over(order by sum(oop.payment_value) desc) as cust_rank 
        from olist_customers oc 
        	join olist_orders oo on oc.customer_id = oo.customer_id 
        	join olist_order_payments oop on oo.order_id = oop.order_id 
        
        group by oc.customer_unique_id 
        order by total_purchase desc
        limit 10;

<img width="564" height="289" alt="image" src="https://github.com/user-attachments/assets/f0a7679f-f59e-4d6f-9cd9-1a34521ec8d5" />


7. Product analysis:

        select  
        
        	pcnt.product_category_name_english as product_category, 
        	
        	sum(ooi.price) as total_revenue, 
        	
        	round(sum(ooi.price) * 100.0 / sum(sum(ooi.price)) over(),2) as percentage, 
        	
        	count(distinct ooi.order_id) as total_orders, 
        	
        	round(avg (ooi.price),2) as revenue_per_order, 
        	
        	sum(ooi.freight_value) as freight_value, 
        	
        	round(sum(ooi.freight_value) / count(distinct ooi.order_id),2) as avg_fright_value 
        
        from product_category_name_translation pcnt 
        
        join olist_products op on pcnt.product_category_name = op.product_category_name 
        
        join olist_order_items ooi on ooi.product_id = op.product_id 
         
        group by product_category 
        
        order by total_revenue desc

        limit 25;

<img width="909" height="664" alt="image" src="https://github.com/user-attachments/assets/fe63774e-6037-4a08-8dcc-383b3d8e7a99" />


8. Top 10 product which sells the most:

        select 
        
          pcnt.product_category_name_english as category, 
        
          count(ooi.order_id) as order_count 
        
        from olist_order_items ooi 
        
          join olist_products op on ooi.product_id = op.product_id 
        
          join product_category_name_translation pcnt on op.product_category_name = pcnt.product_category_name 
        
        group by category 
        
        order by order_count desc
        
        limit 10; 

<img width="293" height="291" alt="image" src="https://github.com/user-attachments/assets/367f23bd-9819-4af8-b427-4c7d8ddabfdf" />


9. Category which generates most revenue with orderr count and ratings:

        select  
        
          pcnt.product_category_name_english as category,
          
          sum(ooi.price) as total_revenue,
          
          count(distinct oo.order_id) as order_count, 
          
          round(avg(oor.review_score),1) as avg_review 
        
        from product_category_name_translation pcnt 
        
          join olist_products op on pcnt.product_category_name = op.product_category_name 
          
          join olist_order_items ooi on ooi.product_id = op.product_id 
          
          join olist_order_reviews oor on oor.order_id = ooi.order_id 
        
          join  olist_orders oo on oo.order_id = oor.order_id 
        
        group by category 
        
        order by total_revenue desc
    
        limit 10;

<img width="562" height="288" alt="image" src="https://github.com/user-attachments/assets/2d773c9c-7bee-4b9a-b136-c641c6070683" />


10. Sellers with high revenue:

        select  
        
        	os.seller_id, 
        	
        	sum(price) as seller_revenue, 
        	
        	count(ooi.order_id) as total_orders,
        	
        	round(sum(price) * 100.0 / sum(sum(price)) over(),2) as revenue_share 
        
        from olist_sellers os 
        
        join olist_order_items ooi on os.seller_id = ooi.seller_id 
        
        group by os.seller_id	 
        
        order by seller_revenue desc
        
        limit 10; 

<img width="616" height="289" alt="image" src="https://github.com/user-attachments/assets/e5407fe0-b62a-4346-9b82-2c69d13c4784" />


11. Seller with best ratings with 100+ order count:

        select 
        
        	ooi.seller_id, 
        	
        	round(avg(oor.review_score),1) as avg_review, 
        	
        	sum(ooi.price) as revenue, 
        	
        	count(ooi.order_id) as orders 
        
        from olist_order_items ooi 
        
        	join olist_order_reviews oor on ooi.order_id = oor.order_id 
        
        group by ooi.seller_id 
        
        having count(ooi.order_id) > '100' 
        
        order by avg_review desc 

        limit 10;

    <img width="527" height="292" alt="image" src="https://github.com/user-attachments/assets/b2f9310b-cfcf-461e-b12c-d8f034f44e7c" />


12. Seller with high cancellation rate:

        select  
        
        	ooi.seller_id, 
        	
        	count(distinct ooi.order_id) as total_order, 
        	
        	count(distinct case when oo.order_status = 'canceled' then ooi.order_id end) as canceled_order, 
        	
        	round(count(distinct case when oo.order_status = 'canceled' then ooi.order_id end)*100.00 / count(distinct ooi.order_id),2) as cancelation_rate 
        
        from olist_order_items ooi 
        
        	join olist_orders oo on oo.order_id = ooi.order_id 
        
        group by ooi.seller_id	 
        
        having count(distinct ooi.order_id) >= 20 
        
        order by cancelation_rate desc
        
        limit 10; 

<img width="623" height="290" alt="image" src="https://github.com/user-attachments/assets/f6135771-b885-4439-b360-d153a49b16d0" />

13. Top 10 cities with most sellers:


        select  
        
        	seller_city, 
        
        	count(*) as seller_count 
        
        from olist_sellers 
        
        group by seller_city 
        
        order by seller_count desc 
        
        limit 10;

<img width="292" height="293" alt="image" src="https://github.com/user-attachments/assets/7e8b070e-f391-4b8a-b384-3307da43cf1a" />


14. Delivery break up:

        select  
        
        	distinct order_id,  
        
        	order_approved_at::date - order_purchase_timestamp::date as seller_approval_time,  
        	
        	order_delivered_carrier_date::date - order_approved_at::date as handover_to_logistic,  
        	
        	order_delivered_customer_date::date - order_delivered_carrier_date::date as shipping_time, 
        	
        	order_delivered_customer_date::date - order_purchase_timestamp::date as total_delivery_time 
        
        from olist_orders  
        
        where order_status = 'delivered' and order_delivered_customer_date is not null  
        
        order by total_delivery_time desc; 

<img width="834" height="641" alt="image" src="https://github.com/user-attachments/assets/45f30a0f-e441-442c-a7d9-d7dfaeca1a2b" />


15. On time vs late delivery:
    
        select 
        	round(count(case when order_estimated_delivery_date::date >= order_delivered_customer_date::date then 1 end) * 100.0 /  
        	count(order_id),2) as on_time_delivery,
        	
          round(count(case when order_estimated_delivery_date::date < order_delivered_customer_date::date then 1 end) * 100.0 /  
        	count(order_id),2) as late_delivery
        
        from olist_orders 
        
        where order_status = 'delivered';

<img width="266" height="68" alt="image" src="https://github.com/user-attachments/assets/9893aedc-48c0-40c4-bbf2-faa5940a4747" />

16. Orders delivered in each year and month:

        select 
        
        	extract(month from order_delivered_customer_date) as months, 
        
        	extract(year from order_delivered_customer_date) as years, 
        
        	count(*) as orders_delivered 
        
        from olist_orders 
        
        where order_status = 'delivered' 
        
        group by years, months;


<img width="307" height="665" alt="image" src="https://github.com/user-attachments/assets/7bc043de-b905-467b-ae33-c72ecc62a5ce" />


 17. Top 10 states with most deliveries:

    select  
    
    	oc.customer_state as states, 
    
    	count(*) as order_delivered 
    
    from olist_customers oc 
    
    	join olist_orders oo on oc.customer_id = oo.customer_id 
    
    where oo.order_status = 'delivered' 
    
    group by states 
    
    order by order_delivered desc
    
    limit 10;

<img width="251" height="293" alt="image" src="https://github.com/user-attachments/assets/f56a59d1-a68f-46f4-b8ef-824b78ab540a" />


18. Delivery count in periods: 

        with delivery_cte as (select   
        
        	distinct order_id,   
        
        	order_delivered_customer_date::date - order_purchase_timestamp::date as total_delivery_time  
        
        from olist_orders   
        
        where order_status = 'delivered' and order_delivered_customer_date is not null   
        
        order by total_delivery_time desc) 
        
        select 
        
        	case when total_delivery_time <= '05' then '0-5 days' 
        
        		 when total_delivery_time <= '10' and total_delivery_time > '5' then '06-10 days' 
        
        		 when total_delivery_time <= '15' and total_delivery_time > '10' then '11-15 days' 
        
        		 when total_delivery_time <= '20' and total_delivery_time > '15' then '16-20 days' 
        
        		 else '20+ days' end as delivery_period, 
        
        		 count(*) as delivery_count 
        
        from delivery_cte	 
        
        group by delivery_period
        
        order by delivery_period; 

<img width="294" height="166" alt="image" src="https://github.com/user-attachments/assets/f3efdab8-57fe-40d2-bf57-43f3ac68bb18" />











