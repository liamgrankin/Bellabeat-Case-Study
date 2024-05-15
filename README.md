# Bellabeat-Case-Study

## Summary

## Ask
### What is the question I am trying to solve?
A stakeholder is asking me to analyze smart device usage data to gain insight into how consumers use non-Bellabeat smart
devices. Then, they want me to select one Bellabeat product to apply these insights for my presentation. 

My task is to gain insight on how customers are using the non-Bellabeat smart devices. Using the smart device data, I can draw conclusions using the following questions as guidelines:
1. What are some trends in smart device usage?
2. How could these trends apply to Bellabeat customers?
3. How could these trends help influence Bellabeat marketing strategy?

## Prepare
To answer the business task at hand, I will need a data set to observe. Kaggle provides a public dataset on smart device usage that explores the 30 users' daily habits. Essentially, 30 users consented to submitting their personal tracking data including minute-level output for physical activity, heart rate, and sleep monitoring. It includes
information about daily activity, steps, and heart rate that can be used to explore users’ habits.

### How is the data stored?

The fitness data is broken up into two months (March 2016 and April 2016). Each of these months are further broken down into individual CSV files containing the health and fitness information from the users over the course of the month.

After observing the files, it appears the "daily_merged" file is a table with a wide format that contains several of the other tables information. We can observe the daily_activity table here:
```
head(daily_activity)
colnames(daily_activity)
nrow(daily_activity)
```
After exploring the csvs in Excel, it appears that the dailyActivity_merged file is a wide table containing information from several of the other CSVs. Let's take a look at the Daily Activity, Sleep, and Weight tables.
```
weight_1 <- read_csv("C:/Users/liamg/OneDrive/Documents/capstone/weightLogInfo_merged.csv")
weight_2 <- read_csv("C:/Users/liamg/OneDrive/Documents/capstone/weightLogInfo_merged_2.csv")
daily_1 <- read_csv("C:/Users/liamg/OneDrive/Documents/capstone/dailyActivity_merged.csv")
daily_2 <- read_csv("C:/Users/liamg/OneDrive/Documents/capstone/dailyActivity_merged_2.csv")
sleep_1 <- read_csv("C:/Users/liamg/OneDrive/Documents/capstone/minuteSleep_merged_1.csv")
sleep_2 <- read_csv("C:/Users/liamg/OneDrive/Documents/capstone/minuteSleep_merged_2.csv")

-- Each file is divided into two months. The two months have identical columns so we are able easily merge the tables

weight <- bind_rows(weight_1,weight_2)
daily <- bind_rows(daily_1,daily_2)
sleep <- bind_rows(sleep_1,sleep_2)

n_distinct(weight$Id)
n_distinct(daily$Id)
n_distinct(sleep$Id)
```
We can see that the Weight table only has 13 total participants compared almost double that in the Sleep and Daily Activity tables. This may make it hard to draw meaningful conclusions due to the small sample size.

![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/977e03aa-4ecb-43fb-afe9-12f315ce33c6)

#### The Daily Activity table
```
colnames(daily)
```
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/10bd7bb1-d7aa-4787-b73e-ee0649db2ec7)

This table contains useful information on intensities of workouts throughout the day and calories, etc.

```
colnames(sleep_day)
```
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
with cte as (
	select 
	id
	,active_group
	,row_number() over (partition by id order by count(*) desc)
	,count(*) from (
		select id
		,case 
			when sedentaryminutes > 1229 then 'Extra Sedentary'
			when sedentaryminutes between 991 and 1229 then 'Avg Sedentary'
			when sedentaryminutes < 991 then 'Not Sedentary'
		end as active_group
	from daily
) as temp
group by id,active_group
	)

select
	cte.active_group,
	daily.*
from cte 
inner join daily
	on daily.id = cte.id
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
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/9499aacd-fade-40f6-b33b-eef84e79864e)

We can see that there is definitely more time spent in bed than asleep. Let's explore this further. First let's set up the data to be broken up by week.

```
daily_sleep <- sleep_day %>%
  rename(date = SleepDay) %>%
  mutate(date = as_date(date, format ="%m/%d/%Y %I:%M:%S %p", tz = Sys.timezone()))

weekday_sleep <- daily_sleep %>%
  mutate(weekday = weekdays(date))

weekday_sleep$weekday <-ordered(weekday_sleep$weekday, levels=c("Monday", "Tuesday", "Wednesday", "Thursday","Friday", "Saturday", "Sunday"))

weekday_sleep <-weekday_sleep %>%
  group_by(weekday) %>%
  summarize (mins_asleep = mean(TotalMinutesAsleep),mins_inbed = mean(TotalTimeInBed))

ggplot(weekday_sleep, aes(x = weekday, y = mins_asleep,mins_inbed, fill = mins_inbed)) + geom_bar(stat = "identity")

weekday_sleep$Difference = weekday_sleep$mins_inbed - weekday_sleep$mins_asleep
average_difference <- mean(weekday_sleep$Difference)
```

``` 
ggplot(weekday_sleep, aes(x = weekday, y = mins_inbed, fill = "Minutes in Bed")) +
  geom_bar(stat = "identity") +
  geom_bar(aes(y = mins_asleep, fill = "Minutes Asleep"), stat = "identity") +
  scale_fill_manual(values = c("Minutes in Bed" = "steelblue", "Minutes Asleep" = "lightblue")) +
  labs(title = "Weekly Sleep Analysis",
       subtitle = paste("Average difference:", round(average_difference, 2), "minutes"),
       x = "Day of the Week",
       y = "Total Minutes",
       fill = "Legend") +
  theme_minimal()
```
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/7fb49263-80a6-478c-b5a6-39777925a90b)

Here we can look at the relationship between total steps + distance per day and calories burned

```
ggplot(daily, aes(x = totalsteps, y = calories, color = totaldistance)) +
  geom_point() +  
  scale_color_gradient(low = "skyblue", high = "navyblue") +
  labs(title = "Calories burned by Total Steps",x = "Total Steps", y = "Calories Burned", color = "Total Distance") +
  theme_minimal()
```
![image](https://github.com/liamgrankin/Bellabeat-Case-Study/assets/54017776/f25c9953-701d-458f-907c-8b3347bffa30)

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
