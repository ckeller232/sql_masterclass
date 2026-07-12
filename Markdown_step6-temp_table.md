# Create temp_table to use in the 3 questions below...

DROP TABLE IF EXISTS temp_table;
create temp table temp_table as
with summary as (
select 
-- 	t.*,
-- 	m.*,
-- 	p.*
	date_trunc('year',t.txn_date)::date as txn_year,
	m.region,
	m.first_name,
	t.ticker,
	round(sum(case when t.txn_type = 'BUY' then t.quantity else 0 end)::numeric,0) as qty_buy,
	round(sum(case when t.txn_type = 'SELL' then t.quantity else 0 end)::numeric,0) as qty_sell,
	round(sum(case when t.txn_type = 'BUY' then t.quantity else 0 end)::numeric - sum(case when t.txn_type = 'SELL' then t.quantity else 0 end)::numeric,0) as final_shares
	

from "learning"."public"."transactions" as t

inner join "learning"."public"."members" as m
	on t.member_id = m.member_id
	
inner join "learning"."public"."prices" as p
	on t.ticker = p.ticker
	and t.txn_date = p.market_date
	
where t.txn_date < '2021-01-01'

group by txn_year,m.region,m.first_name,t.ticker
),

closing_prices as (
	select 
		*,
		date_trunc('year',market_date)::date as year,
		row_number() over (partition by ticker, date_trunc('year',market_date) order by market_date desc) as row
	from "learning"."public"."prices"
)

select
	s.*,
	c.price,
	s.final_shares * price as final_value
	
from summary as s

left join closing_prices as c
	on s.ticker = c.ticker
	and s.txn_year = c.year
	and c.row = 1


## Question 1
-- what is the portfolio value for each mentor at the end of 2020?
select 
	first_name,
	round(sum(final_value)::numeric,1) as total_value

from temp_table
	where txn_year <= '2020-01-01'
group by first_name

order by first_name

## QUESTION 2
-- what is the total portfolio value for each region at the end of 2019?

select
	region,
	round(sum(final_value)::numeric,1) as total_value
	
from temp_table
where txn_year <= '2019-01-01'
group by region
order by region

## QUESTION 3
-- what percentage of regional portfolio value does each member contribute at the end of 2018?
with region as (
select
	region,
	round(sum(final_value)::numeric,1) as total_value
	
from temp_table
where txn_year <= '2018-01-01'
group by region
order by region
	),

mentor as (
select 
	first_name,
	region,
	round(sum(final_value)::numeric,1) as total_value

from temp_table
	where txn_year <= '2018-01-01'
group by first_name,region

order by first_name
)

select 
	m.first_name,
	r.region,
	m.total_value as mentor_value,
	r.total_value as total_region_value,
	to_char((m.total_value/r.total_value)*100,'999D99')||'%' as percent_contributed

from region as r

left join mentor as m
	on r.region = m.region
	
order by r.region,m.first_name

































































	
	
	
	
	
	
	
	
	
	
	
	
	