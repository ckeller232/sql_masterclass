# Step 5 - Table Summaries with Joins
## Question 1 - Which top 3 mentors have the most Bitcoin quantity as of the 29th of August?
select 

t.member_id,

m.first_name,

sum(case when txn_type = 'BUY' THEN quantity else 0 end) - sum(case when txn_type = 'SELL' then quantity else 0 end) as total_qty

from "learning"."public"."transactions" as t

left join "learning"."public"."members" as m
	on t.member_id = m.member_id

where t.ticker ilike ('%btc%')
	
group by t.member_id,m.first_name

order by test desc

limit 3

## Question 2 - What is total value of all Ethereum portfolios for each region at the end date of our analysis? Order the output by descending portfolio value?

with quantity as (

select 

region,

t.ticker,

sum(case when t.txn_type = 'BUY' then t.quantity else 0 end) - sum(case when t.txn_type = 'SELL' then t.quantity else 0 end) as total_qty

from "learning"."public"."transactions" as t

inner join "learning"."public"."members" as m

	on t.member_id = m.member_id

inner join "learning"."public"."prices" as p

	on t.ticker = p.ticker
	
	and t.txn_date = p.market_date
	
where t.ticker ilike ('%eth%')

group by m.region,t.ticker

	)
	
select 

	q.region,
	
	q.ticker,
	
	round(q.total_qty::numeric,2) as total_qty,
	
	round((p.price * q.total_qty)::numeric,2) as total_value

from quantity as q

left join "learning"."public"."prices" as p

	on q.ticker = p.ticker
	
	and market_date = '2021-08-29'
	
order by total_value desc











	






