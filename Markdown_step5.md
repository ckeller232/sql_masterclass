# Step 5 - Table Summaries with Joins
## Question 1 - Which top 3 mentors have the most Bitcoin quantity as of the 29th of August?
*
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

