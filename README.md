# Bellabeat-Case-Study

## Summary

## Ask
My task is to gain insight on how customers are using the non-Bellabeat smart devices. Using the smart device data, I can draw conclusions using the following questions as guidelines:


## Prepare
First I wanted to take a look at the data to see what kind of information we are dealing with. 
```
head(daily_activity)
colnames(daily_activity)
nrow(daily_activity)
```
The Daily Activity table
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/6ab62f44-2b6d-44aa-ac60-d90b615e6537)
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/10bd7bb1-d7aa-4787-b73e-ee0649db2ec7)

This table contains useful information on intensities of workouts throughout the day and calories, etc.

```
head(sleep_day)
colnames(sleep_day)

```
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/f312173f-2f52-4ecb-898f-f907f0797b4a)
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/9f05e0f1-570e-4ef3-800d-9d4a1c38a1a1)


After cleaning the data, I used SQL to classify the users into groups based on their average sedentary time spent per day. 

Here I look at a quick summary of the statistics 

```
daily_activity %>%  
  select(TotalSteps,
         TotalDistance,
         SedentaryMinutes) %>%
  summary()
```
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/8d68b98d-a789-414f-9e7a-384e32a00fa2)


I can see that the first quartile is 729, the median is 1057, and the third quartile is around 1229. 

#### Let's take a look at the sleep data
```
sleep_day %>%  
  select(TotalSleepRecords,
         TotalMinutesAsleep,
         TotalTimeInBed) %>%
  summary()
```
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/2f864dd7-09f8-4edf-bfb8-399cfb5b8586)

We can already see from the Total Minutes Asleep and Total Time in Bed summaries that there is a clear discrepency. This is something we should definitely explore down the line. 
I then brought the data into SQL so I could easily group the data into different categories. First, let's use our information on the sedentary minutes to see divide the users based on their time spent sendentary per day.
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
As expected, the results were distributed fairly evenly among the users:
```
select 
	active_group
	,count(*) 
from cte
group by active_group
```
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/5a70a22b-5ead-4595-9c76-5d382d6cbfcb)

## Analyze
```
ggplot(data=sleep_day, aes(x=TotalMinutesAsleep, y= TotalTimeInBed)) + geom_point() + geom_smooth()
```
We can see that there is definitely more time spent in bed than asleep. Let's explore this further.


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
