# Bellabeat-Case-Study

## Summary

## Ask
I was asked to gain insight on how customers are using the non-Bellabeat smart devices. Using this information, I can draw conclusions for how to apply these insights.

## Prepare

After cleaning the data, I used SQL to classify the users into groups based on their average sedentary time spent per day. 
```
with cte as (select id,active_group,row_number() over (partition by id order by count(*) desc), count(*) from (select
	id
	,case when sedentaryminutes > 1200 then 'Extra Sedentary'
	when sedentaryminutes between 900 and 1200 then 'Avg Sedentary'
	when sedentaryminutes < 900 then 'Not Sedentary'
	end active_group
from daily
) as temp
group by id,active_group
)

select cte.active_group,daily.* from cte 
inner join daily on daily.id = cte.id
where row_number = 1
```
The results were distributed fairly evenly among the users:
```
select 
	active_group
	,count(*) 
from cte
group by active_group
```
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/5a70a22b-5ead-4595-9c76-5d382d6cbfcb)

