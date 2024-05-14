# Bellabeat-Case-Study

## Summary

## Ask
My task is to gain insight on how customers are using the non-Bellabeat smart devices. Using the smart device data, I can draw conclusions using the following questions as guidelines:


## Prepare
First I wanted to take a look at the data to see what kind of information we are dealing with. 
```
head(daily_activity)
colnames(daily_activity)

```
The Daily Activity table
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/6ab62f44-2b6d-44aa-ac60-d90b615e6537)
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/10bd7bb1-d7aa-4787-b73e-ee0649db2ec7)

This table contains useful information on intensities of workouts throughout the day and calories, etc.

```
head(daily_slee
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

```
mydf <- read_csv("C:/Users/liamg/OneDrive/Documents/capstone/data_done2.csv")

mydf$activityhour = as.POSIXct(mydf$activitydate, format = "%m/%d/%Y %I:%M:%S %p",tz = Sys.timezone())

intensity <- read_csv("C:/Users/liamg/OneDrive/Documents/capstone/hourlyIntensities_merged.csv")

intensity$ActivityHour=as.POSIXct(intensity$ActivityHour, format = "%m/%d/%Y %I:%M:%S %p",tz = Sys.timezone())

intensity <- intensity %>% 
  rename(id = 'Id')

df2 <- mydf %>% 
  inner_join(intensity)

df2 <- df2 %>% 
  mutate(ActivityHour = ymd_hms(ActivityHour),
         hours = hour(ActivityHour), mins = minute(ActivityHour),
         secs = second(ActivityHour))

df2 <- df2 %>% 
  group_by(hours,active_group) %>% 
  summarize(mean = mean(TotalIntensity))

viz <- ggplot(df2) + geom_line(aes(hours,mean, group = active_group, color = active_group, size = ".5")) + labs(x = "Hour of the day", y = "Intensity of workout", title = "Intensity Of Workout per Hour")
print(viz)
```
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/dd76beb5-34bc-4ef4-b29c-f8c5e904d6ce)
