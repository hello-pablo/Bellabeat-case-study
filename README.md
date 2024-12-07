# Bellabeat Case Study
Google Data Analytics capstone project

## Introduction
This project is part of the Google Data Analytics Certificate and focuses on analyzing smart device usage data for Bellabeat, a high-tech company specializing in health-focused products for women.

## Objectives
- Understand user behavior through smart device data.
- Provide actionable insights for Bellabeat's marketing strategy.

## Tools
- **Programming language**: R (RStudio Cloud)
- **Libraries**: `tidyverse`, `lubridate`, `ggplot2`, `dplyr`

## Data
The data was provided as part of the case study and includes user activity, sleep, and weight metrics collected from smart devices. [Fitbit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit) . 
## Limitation 
- Sample size: 30 people is not a large enough sample to be representative of all FitBit users
- Outdated: The dataset contains data from one month period in 2016 only. For a deeper and more accurate analysis of trends, we would need data from the current year, preferably collected for an entire year to look at if trends vary during different times of year.
- Limited: The dataset does not contain any demographic information about the users, including gender, age, or location, which would be beneficial for marketing purposes to target specific customers

## Analysis Process
1. **Load Packages**:

    ```r
   install.packages("tidyverse")
    install.packages("here")
    install.packages("janitor")
    install.packages("lubridate")
    install.packages("skimr")
    install.packages("ggplot2")
    ```
2. **Importing datasets**: 
    ```r
    daily_activity <- read.csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/dailyActivity_merged.csv")
   daily_sleep <- read_csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/sleepDay_merged.csv")
    weight <- read_csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/weightLogInfo_merged.csv")
    hourly_intensity <- read_csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/hourlyIntensities_merged.csv")
    hourly_calories <- read_csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/hourlyCalories_merged.csv")
    hourly_steps <- read_csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/hourlySteps_merged.csv") 
 
- i looked at the different datasets to see the columns and the first 6 rows of each.

   ```r
  head(daily_activity)
  head(daily_sleep)
  head(weight)
  head(hourly_intensity)
  head(hourly_calories)
  head(hourly_steps)
   ```
- the three datasets containing time-related information could be merged into a single one using “Id” and “ActivityHour” columns

  ```r
  hourly_activity <- merge(hourly_intesity, hourly_calories, by=c("Id","ActivityHour"))
  hourly_activity <- merge(hourly_activity, hourly_steps, by=c("Id","ActivityHour"))
  ```
3. **Checking the number of unique participant**:

    ```r
   n_distinct(daily_activity$Id)
   n_distinct(daily_sleep$Id)
   n_distinct(weight$Id)
   n_distinct(hourly_activity$Id)
    ```
 - The “weight” dataset only had 8 unique participants, making the sample size too small, and thus it will be excluded from further analysis.  

4. **View and clean up the data sets**:

   ```r
    clean_names(daily_activity)
    clean_names(daily_sleep)
    clean_names(hourly_activity)

   #check for duplicates, daily_sleep has 3 duplicate

    sum(duplicated(daily_activity))
    sum(duplicated(daily_sleep))
    sum(duplicated(hourly_activity))

    #Cleaning the 3 duplicates from daily_sleep
    
    daily_sleep <- unique(daily_sleep)
   ```
- Converting date & time format

  ```r
    daily_activity$ActivityDate <- mdy(daily_activity$ActivityDate)
    colnames(daily_activity)[2] = "Date"
    daily_sleep$SleepDay <- mdy_hms(daily_sleep$SleepDay)
    colnames(daily_sleep)[2] = "Date"

    #ActivityHour column could be divided in Date and Hour

    hourly_activity$ActivityHour <- mdy_hms(hourly_activity$ActivityHour) 
    hourly_activity <- hourly_activity %>% separate(ActivityHour, c("Date", "Hour"), sep = " ")

    #add a column for the day of the week in daily_activity

     daily_activity$day_of_week <- wday(daily_activity$ActivityDate, label = TRUE, abbr = FALSE)
    
    #add a column for the total sum of activities in daily_activity

     daily_activity$Total_Active_Hours = (daily_activity$SedentaryMinutes + daily_activity$LightlyActiveMinutes + daily_activity$FairlyActiveMinutes + daily_activity$VeryActiveMinutes)/60
    daily_activity$Total_Active_Hours <- round(daily_activity$Total_Active_Hours, 2)
    
    #add two columns to the daily_sleep dataset to convert minutes to hours

    daily_sleep$Total_hour_in_Bed = round((daily_sleep$TotalTimeInBed)/60 , 2)
    daily_sleep$Total_hour_Asleep = round((daily_sleep$TotalMinutesAsleep)/60 , 2)


 ## Data Exploration:

 - observe the distribution of daily steps: 

```r
    summary(daily_activity$TotalSteps)
    ggplot(daily_activity, aes(x = TotalSteps)) + geom_boxplot() 
```
  ![steps distribution](https://github.com/user-attachments/assets/000d8496-238c-41b4-b5ab-cdd5f0b4d4aa)

Most of the daily total steps appear to be around 4000-11000.

- **Sleep Data**:
  
 Calculate the average hours of sleep for each partecipant
```r
mean_sleep <- daily_sleep %>% 
 group_by(Id) %>% 
  summarise(mean_sleep = mean(Total_hour_Asleep),
            num_observations = n()
            ) %>% 
  select(Id, mean_sleep, num_observations) %>% 
  arrange(mean_sleep) %>% 
  mutate_if(is.numeric, round, 2) %>%
  as.data.frame()
```
```
head(mean_sleep)
          Id mean_sleep num_observations
1 2320127002       1.02                1
2 7007744171       1.15                2
3 4558609924       2.13                5
4 3977333714       4.89               28
5 1644430081       4.90                4
6 8053475328       4.95                3
```
- What percent of the time did participants actually spend sleeping while laying in bed?
```r
sleep_rateo <- daily_sleep %>% 
  group_by(Id) %>% 
  summarise(percent_sleep = (TotalMinutesAsleep/TotalTimeInBed)*100, na.rm = TRUE) %>% 
  select(Id,percent_sleep) %>% 
  summarise(avg_percent_sleep = mean(percent_sleep)) %>% 
  arrange(avg_percent_sleep) %>% 
  mutate_if(is.numeric, round, 2)  
```
```
head(sleep_rateo)
# A tibble: 6 × 2
          Id avg_percent_sleep
       <dbl>             <dbl>
1 3977333714              63.4
2 1844505072              67.8
3 1644430081              88.2
4 2320127002              88.4
5 4558609924              90.7
6 2347167796              91.0
```
Most participants slept at least 90% of the time they spent in bed

- **Activity Level for partecipant**:

```r
activity_id <- daily_activity %>% 
  group_by(Id) %>% 
  summarize(sum_very_active = sum(VeryActiveMinutes),
            sum_fairly_active = sum(FairlyActiveMinutes),
            sum_lightly_active = sum(LightlyActiveMinutes),
            sum_sedentary = sum(SedentaryMinutes)) %>% 
  select(Id, sum_very_active, sum_fairly_active, sum_lightly_active, sum_sedentary) %>% 
  as.data.frame()
```
```
          Id sum_very_active sum_fairly_active sum_lightly_active sum_sedentary
1 1503960366            1200               594               6818         26293
2 1624580081             269               180               4758         38990
3 1644430081             287               641               5354         34856
4 1844505072               4                40               3579         37405
5 1927972279              41                24               1196         40840
6 2022484408            1125               600               7981         34490
```

- **Steps**:

- on average, during which hour of the day were the most steps taken?

```r
hourly_activity %>% 
  group_by(Hour) %>% 
  summarize(mean_steps = mean(StepTotal)) %>% 
  select(Hour, mean_steps) %>% 
  arrange(desc(mean_steps)) %>% 
  head(1)
```
```
 A tibble: 1 × 2
  Hour     mean_steps
  <chr>         <dbl>
1 18:00:00       599.
```
on average 600 steps are taken at 6pm.
I insert the results into a dataframe.

```r
mean_steps <- hourly_activity %>% 
  group_by(Hour) %>% 
  summarize(mean_steps = mean(StepTotal)) %>% 
  select(Hour, mean_steps) %>% 
  as.data.frame()
```
- I analyze the mean and standard deviation of each participant's steps.

```r
steps_by_Id <- hourly_activity %>% 
  group_by(Id) %>% 
  summarize(mean_steps_id = mean(StepTotal), sd_steps_id = sd(StepTotal)) %>% 
  select(Id, mean_steps_id, sd_steps_id) %>% 
  mutate_if(is.numeric, round, 2) %>% 
  as.data.frame()
```
```
         Id mean_steps_id sd_steps_id
1 1503960366        522.38      836.48
2 1624580081        241.51      760.46
3 1644430081        307.81      589.61
4 1844505072        109.36      232.06
5 1927972279         38.59      164.17
6 2022484408        477.87      861.30
```
A high standard deviation value could mean that the user alternates very active days with less active days
conversely a low value could indicate a more regular behavior.


## Visualization

- Relationship between steps taken and calories:
```r
    ggplot(data = daily_activity, aes(x = TotalSteps, y = Calories)) + 
  geom_point() + 
  geom_smooth() +
  labs(title = "Total Steps Vs. Calories", x = "Steps", Y = "Calories" )
```
![Total Steps Vs  Calories](https://github.com/user-attachments/assets/9032da16-c143-4177-8e17-c7b232fb21e2) 

There appears to be a positive correlation between steps and calories,
although some variability in the data suggests there may be other factors that influence calorie burn.

- Average amount of time participants slept each night during the course of the study:
 ```r
options(scipen = 999)
ggplot(data = mean_sleep, aes(x = Id, y = mean_sleep)) + 
  geom_col(aes(reorder(Id, + mean_sleep), y = mean_sleep)) +
  labs(title = "Avarage hours of sleep", x = "Partecipant", y = "Hours") + 
  theme(axis.text.x = element_text(angle = 90)) + 
  geom_hline(yintercept = mean(mean_sleep$mean_sleep), color = "red")
```
 ![Avarage hours of sleep](https://github.com/user-attachments/assets/62778b59-6851-4202-a7fd-5437466d37bd)

 The graph shows the average sleep of each participant individually, as well as how their sleep compares to the overall average across all participants.

- On which day did participants sleep the longest?

```r 
daily_sleep <- daily_sleep %>%
  mutate(day_of_week = weekdays(Date))
```
```
#I aggregate the data by day of the week
avg_sleep_by_day <- daily_sleep %>% 
  group_by(day_of_week) %>% 
  summarise(avg_minutes_asleep = mean(TotalMinutesAsleep, na.rm = TRUE))
```
```
#order the days of the week
avg_sleep_by_day <- avg_sleep_by_day %>%
  mutate(day_of_week = factor(day_of_week, 
                              levels = c("Monday", "Tuesday", "Wednesday", 
                                         "Thursday", "Friday", "Saturday", "Sunday")))

ggplot(avg_sleep_by_day, aes(x = day_of_week, y = avg_minutes_asleep)) +
  geom_col() + labs(title = "Avarage time asleep per day", x = "day", y = "avg time asleep")
```
![Avg time asleep per day](https://github.com/user-attachments/assets/32c69223-93f3-4e10-be1c-25b2e342d2d7)

On average, participants slept more on Sunday.

- Average steps in relation to time:

```r
ggplot(mean_steps, aes(x = Hour, y = mean_steps)) + 
  geom_col() + theme(axis.text.x = element_text(angle = 90)) + 
  labs(title = "Avarage Steps taken per Hour of Day", 
       x = "Hour", y = "Avarage Steps")
```
![Avarage Steps per Hour](https://github.com/user-attachments/assets/ed674e6f-09b4-4a11-b96c-79ca8d73687f)

Peaks are noted in the time slots 12AM-2PM and 5PM-7PM

- Calories by day of the week:

```r
ggplot(daily_activity, aes(x = day_of_week, y = Calories)) +
  geom_col() +
    labs(title = "Calories vs Day of Week", x = "Day" , y = "Calories")
```
![Calories vs Day of Week](https://github.com/user-attachments/assets/37d34d89-1b8a-4f27-8496-924806c5c8da)

```r
#I merge two datasets to look for a correlation

combined_data <- merge(activity_id, steps_by_Id, by = "Id")
```
```r
#now I can create a graph

ggplot(combined_data, aes(x = mean_steps_id, y = sum_very_active)) +
  geom_point() + labs(title = "Avg steps taken in a day compared to very active minutes",
                      x = "avarage steps", y = "very active minutes")
```
![Avarage steps vs very active](https://github.com/user-attachments/assets/3bf5919f-44a7-4da1-b1f9-778ce906b9ca)

A positive correlation is observed, the intensity minutes increase as the number of steps increases.

- Exporting the three data set:
```r
write.csv(daily_activity, "daily_activity.csv", row.names = FALSE)
write.csv(hourly_activity, "hourly_activity.csv", row.names = FALSE)
write.csv(daily_sleep, "daily_sleep.csv", row.names = FALSE)
```

The analyses performed and the data visualization show the following trends: 
- there is a positive correlation between the number of steps and the calories burned by the users. the more active the user is, the more significant the number of burned calories is.
- examining the relationship between the steps taken and the time of day. We can observe that the hours after work from 5 to 7 PM, as well as the break hours starting from 12 PM, are the most active.
- On average, participants slept the most on Sundays, which was also the day they took the least amount of steps.
- The days on which participants burned the most calories on average are Tuesday and Wednesday.
- Users who take more steps per day are more likely to accumulate "very active minutes."


## Conclusion
It is important to note that the data collected comes from a limited sample and user demographic information was not considered part of the analysis.
- Only a few participants provided weight-related data, suggesting an opportunity for further research to explore this aspect in more detail.
- On average, participants slept less than the CDC-recommended 7 hours per night, highlighting an opportunity to promote Bellabeat's sleep tracking feature. By leveraging its ability to record users' average wake-up times, Bellabeat could provide personalized bedtime recommendations via notifications to help users achieve better rest. Additionally, marketing the device alongside a meditation app or habit tracker could further support users in improving their sleep patterns.
- Data indicates that device usage peaks around 6 PM, suggesting that most users follow typical work hours and tend to accumulate the majority of their steps after work. Targeted advertising aimed at working adults could highlight how the device seamlessly tracks steps throughout their busy schedules. Additionally, sending reminder notifications around 12 PM and 8 PM could motivate users to stay active during breaks, such as lunchtime and after dinner, helping them incorporate more movement into their routines.
- Bellabeat could focus on promoting weekend relaxation features and fitness challenges during weekdays to align with user behavior.
- To encourage user activity, Bellabeat could introduce a gamified system where users earn points for achieving their daily, weekly, or monthly goals. These points could be redeemed for rewards such as discounts on sports equipment, providing a fun and tangible incentive to stay active and engaged with the platform.

## Files


- `analysis/`: R scripts for data cleaning and analysis.
- `figures/`: Plots and visualizations.
- `data/`: (Contains raw and cleaned datasets).


## Author
[Stefano Ciriello](https://github.com/hello-pablo)
