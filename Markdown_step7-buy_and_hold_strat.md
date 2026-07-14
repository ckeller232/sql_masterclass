# Buy and Hold Strategy - assumes initial purchase and sell at a date in the future


### create temp table to answer questions
drop table if exists leah_hold_strategy;
create temp table leah_hold_strategy as (
select *

from "learning"."public"."transactions"

where txn_date = '2017-01-01'
	and member_id = 'c20ad4' -- member id for leah
	and quantity = 50
	)
	
## Question - what is Leah's end value and profitability if she purchases 50 shares of BTC and ETH on 2017-01-01 and sells it on 2021-08-29.
with initial_summary as (
select 
	l.*,
	p.price,
	round((l.quantity * p.price)::numeric,2) as value,
	round((l.quantity * p.price * (l.percentage_fee/100))::numeric,2) as fees

from leah_hold_strategy as l

left join "learning"."public"."prices" as p
	on l.ticker = p.ticker
	and l.txn_date = p.market_date
	),
	
final_value as (
select
	l.*,
	p.price,
	round((l.quantity * p.price)::numeric,2) as value,
	round((l.quantity * p.price * (l.percentage_fee/100))::numeric,2) as fees
from leah_hold_strategy as l

left join "learning"."public"."prices" as p
	on l.ticker = p.ticker
	and p.market_date = '2021-08-29'

),

initial_summary_cum as (
select 
	1 as col,
	sum(value) as initial_value,
	sum(fees) as initial_fees
from initial_summary
	),
final_value_cum as (
select 
	1 as col,
	sum(value) as final_value
	--sum(fees) as final_fees  ## fees only collected at time of purchase so this is not needed for calculation
	
from final_value
	)

select 
	isc.initial_value,
	isc.initial_fees,
	f.final_value,
	f.final_value/isc.initial_value as profit_multiplier

from final_value_cum as f

right join initial_summary_cum as isc
	on f.col = isc.col

























































