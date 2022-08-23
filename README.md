# Bellabeat: Case Study
*This document is created as part of the capstone project of the Google Data Analytics Professional Certificate.*

## Introduction and scenario
  You are a junior data analyst working on the marketing analyst team at Bellabeat, a high-tech manufacturer of health-focused products for women. Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market. Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart device fitness data could help unlock new growth opportunities for the company. You have been asked to focus on one of Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices. The insights you discover will then help guide marketing strategy for the company. You will present your analysis to the Bellabeat executive team along with your high-level recommendations for Bellabeat’s marketing strategy.

#### About the company
Urška Sršen and Sando Mur founded Bellabeat, a high-tech company that manufactures health-focused smart products. Sršen used her background as an artist to develop beautifully designed technology that informs and inspires women around the world. Collecting data on activity, sleep, stress, and reproductive health has allowed Bellabeat to empower women with knowledge about their own health and habits. Since it was founded in 2013, Bellabeat has grown rapidly and quickly positioned itself as a tech-driven wellness company for women.

This project follows six steps of the data analysis process: **ask, prepare, process, analyze, share, and act.**

## Phase 1: Ask
Key tasks: Analyze smart device usage data to gain insight and help guide Bellabeat marketing strategy.

Primary stakeholders: Urška Sršen and Sando Mur, executive team members

Secondary stakeholders: Bellabeat marketing analytics team

## Phase 2: Prepare
* The data used in this analysis is FitBit Fitness Tracker Data from 30 users made available uploaded by a data scientist called Mobius stored on [Kaggle](https://www.kaggle.com/datasets/arashnic/fitbit).
* The data contains personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring.
* The dataset consists of 18 CSV files organized in long format.

### ROCCC Analysis
* Reliability - We cannot be 100% sure that it is correct as it was collected from only 30 participants whose gender is unknown.
* Originality - This is a thrid-party data collected via Amazon Mechanical Turk.
* Comprehensive - The dataset contains physical activity data, heart rate, and sleep monitoring in format of day, hour, and minute.
* Current - The dataset was generated from the 12th of April, 2016 to the 12th of May 2016. The data is not current, users' habit may change now.
* Cited - Unknown

### Limitations of this dataset:
* Only data from 30 users available. Eventhough the central limit theorem rule of n≥30 applies, a larger sample size is preferred in this analysis.

## Phase 3: Process
In this step, we need to make sure that our data is clean, error free and in the right format. 

### Tools
R will be used to process the data as it can efficiently handle huge dataset and Tableau will be used later on to create visualization.
```
# First, install and load the packages that I will be using

library(tidyverse)
library(skimr)
library(janitor)
library(lubridate)
library(dplyr)
library(ggplot2)
```
### Importing the dataset
I will use three main tables: daily activity, sleep day, and weight log.
```
daily_activity <- read.csv("dailyActivity_merged.csv")
day_sleep <- read.csv("sleepDay_merged.csv")
weight_log <- read.csv("weightLogInfo_merged.csv")
```
### Data cleaning
After reviewing the dataset, I found that the daily activity table has almost all of the information needed for the analysis.
```
# In this step, I am going to use the str() to see the structure of the database.

str(daily_activity)
str(day_sleep)
str(weight_log)
```
Then, use `n_distinct()` to check for unique ID in each table, daily activity table has 33 unique IDs, 24 from sleep, and 8 from weight log table. The dataset is inconsistent as we expect 30 unique IDs and some users did not record all their data. The weight log table shows the highest consistency with 22 input missing. This would affect the result of the analysis.

To keep the consistency throughout the table, I use `clean_names()` to change the column name to lowercase and add underscore sign to seperate words
```
daily_activity <- daily_activity %>% clean_names()
day_sleep <- day_sleep %>% clean_names()
weight_log <- weight_log %>% clean_names()
```
Next, I am going to check for blank cells in each dataset.
```
sum(is.na(daily_activity))
sum(is.na(day_sleep))
sum(is.na(weight_log))
```
I found that daily_activity table and day_sleep is complete with no blank cells but the weight_log table has 65 blank cells in fat column and also found that 41 rows from 67 rows of data were filled manually.

From the str() step, the date format of daily_activity table is set to character so I am going to change to a date format "month/date/year".
```
# change the date format to "month/date/year"

daily_activity <- daily_activity %>% 
  rename(date = activity_date) %>% mutate(date = as_date(date, format = "%m/%d/%Y"))
weight_log <- weight_log %>% 
  mutate(date = as_date(date, format = "%m/%d/%Y"))
daily_calories <- daily_calories %>% 
  rename(date = activity_day) %>% mutate(date = as_date(date, format = "%m/%d/%Y"))
```

After that, I will check for duplicated rows.
```
sum(duplicated(daily_activity))
sum(duplicated(day_sleep))
sum(duplicated(wegith_log))
```
I found that there 3 duplicated records in day_sleep table, so I removed them by running this code.
```
day_sleep <- day_sleep %>% distinct()
```

## Phase 4: Analyze
After the data cleaning phase, the dataset is now clean and ready for the analysis. I will be using R and begin with the daily_activity table.

First, I will start with checking for the credibility of the usage of the FitBit.
```
no_usage <- daily_steps %>%
  group_by(id) %>%
  filter(step_total == 0) %>%
  count(step_total == 0)
  
no_usage$id <- as.character(no_usage$id)
ggplot(no_usage) + geom_col(aes(id, n, fill=n)) + theme(axis.text.x = element_text(angle = 90)) + labs(fill="Count", title = "Number of days with no usage", subtitle="Tracked by individual ID", x="ID", y="Number of days without usage")
```
![Number of days without usage](https://user-images.githubusercontent.com/109403200/186077217-140532bb-c174-4bb5-9a6e-65dad24ebcdf.png)

Next, I will sum up the number of days and nights that had been tracked by this Fitbit.
```
total_per_id <- daily_log %>% 
  group_by(id) %>% 
  summarise(total_days_tracked = sum(total_steps>0), total_nights_tracked = sum(total_minutes_asleep>0))

total_per_id$id <- as.character(total_per_id$id)

ggplot(total_per_id, aes(id,total_nights_tracked, fill=total_nights_tracked)) + geom_col() + labs(title="Number of Tracked Nights", x="ID", y="Number of Nights Tracked") + theme(axis.text.x = element_text(angle=90)) + scale_fill_gradient(low = "plum1", high="royalblue3")
ggplot(total_per_id, aes(id, total_days_tracked, fill=total_days_tracked)) + geom_col() + labs(title="Number of Tracked Days", x="ID", y="Number of Days Tracked") + theme(axis.text.x = element_text(angle = 90)) + scale_fill_gradient(low = "pink", high = "salmon")
```
![image](https://user-images.githubusercontent.com/109403200/186108638-4b7d78b2-6d13-4395-aac6-4f85dae7c9c7.png)
![image](https://user-images.githubusercontent.com/109403200/186108531-8f0834d3-eb5b-4d90-b5a6-213a520c149a.png)


### Activity level
According to the [MedicineNet website](https://www.medicinenet.com/how_many_steps_a_day_is_considered_active/article.htm), activity level can be categorized into 5 types by number of steps taken per day. 

* Sedentary: Less than 5,000 steps daily
* Low Active: About 5,000 to 7,499 steps daily
* Somewhat Active: About 7,500 to 9,999 steps daily
* Active: About 10,000 to 12,499 steps daily
* Highly Active: More than 12,500 steps daily

```
activity_level <- daily_activity %>%
  group_by(id) %>%
  summarise(daily_steps = mean(total_steps)) %>%
  mutate(activity_level = case_when(
    daily_steps <= 4999 ~ "Sedentary",
    daily_steps >= 5000 & daily_steps <= 7499 ~ "Low Active",
    daily_steps >= 7500 & daily_steps <= 9999 ~ "Somewhat Active",
    daily_steps >= 10000 & daily_steps <= 12499 ~ "Active",
    daily_steps >= 12500 ~ "Very Active"
  ))
  
ggplot(activity_level) + geom_bar(aes(y=activity_level, fill=activity_level)) + labs(fill="Activity Level", title="Number of users in each activity level", y="Activity Level", x="Number of users") + theme(legend.position = "none")
```
![Number of users in each activity level](https://user-images.githubusercontent.com/109403200/186077582-a1220694-7241-4e5a-a1b5-bd637ed191ed.png)

Then, I am going to compare the relationship between total daily steps and calories burned.
```
ggplot(daily_activity, aes(total_steps, calories)) + geom_point(color="salmon") + geom_smooth() + labs(title="Calories Burned VS Total Daily Steps",x="Total Steps", y="Calories")
```
![image](https://user-images.githubusercontent.com/109403200/185782215-804660b5-ee2a-441f-bbc7-164d7c1ee99d.png)

From the plot above, there is a positive correlation between steps taken and calories burned, which means that the more steps you take, the more calories are burned.

Next, I will mutate 2 columns into the daily_activity table and name it 'daily_activity2', which are dow(day of week) and total_active_minutes.
```
daily_activity2 %>% daily_activity %>%
  mutate(dow = wday(ymd(daily_activity$date), label = 'TRUE')) %>%
  mutate(total_active_minutes = very_active_minutes + fairly_active_minutes + lightly_active_minutes)
```

Lastly, I will find the number of average steps that participants took on each day of the week.
```
avg_day <- daily_activity2 %>% 
  group_by(dow) %>% 
  summarise(avg_steps_per_day = mean(total_steps))
  
ggplot(avg_day, aes(dow, avg_steps_per_day)) + geom_col(fill="salmon") + labs(title="Average Total Steps in Each Day of Week", x="Day of Week", y="Avg. Total Steps")
```
![Average Total Steps of Each Day of Week](https://user-images.githubusercontent.com/109403200/186068077-3c125e20-2a57-4862-ae4b-7a692c04d3a3.png)

### Sleep log
First, I want to see whether there is a relationship between total minutes asleep and calories burned in a day or not.
```
# Joining daily_activity table and day_sleep table
daily_log <- full_join(daily_activity2, day_sleep, by = c('id', 'date'))
daily_log[is.na(daily_log)] <- 0
str(daily_log)

ggplot(daily_log, aes(total_minutes_asleep, calories, color=total_minutes_asleep)) + geom_point() + geom_smooth(method="lm") + labs(title="Calories Burned VS Total Minutes Asleep", x="Total Minutes Asleep", y="Calories")
```
![image](https://user-images.githubusercontent.com/109403200/185781169-8c65ef97-5915-4fea-ad84-fad13402f60b.png)

Next, I will find the average sleeping hours for each individual user ID.
```
filter_sleep <- day_sleep %>% 
  group_by(id) %>% 
  summarise(avg_min_asleep = mean(total_minutes_asleep)) %>% 
  mutate(avg_hr_asleep = avg_min_asleep/60)

filter_sleep$id <- as.character(filter_sleep$id)

p <- ggplot(filter_sleep, aes(id, avg_hr_asleep, fill=avg_hr_asleep)) + geom_col() + scale_fill_gradient(low = "plum1", high = "royalblue3")
p + theme(axis.text.x = element_text(angle=90)) + labs(title="Average Hour Asleep", x="ID", y="Avg. Hour Asleep", subtitle = "Tracked by individual ID")
```
![image](https://user-images.githubusercontent.com/109403200/186121104-90fe3515-63c3-42d6-b923-88830cd83bff.png)


### Weight log
Start with creating a new data frame to check how many rows were filled manually.
```
weight_log2 <- c("Manual_True", "Manual_False")
weight_log_count <- c(nrow(subset(weight_log, is_manual_report == "True")), nrow(subset(weight_log, is_manual_report == "False")))

weight_log_report <- data.frame(weight_log2, weight_log_count)
ggplot(weight_log_report, aes(weight_log2, weight_log_count)) + geom_col(fill="salmon", width = 0.5) + labs(title="Type of data log", x="Manual or Not", y="Count", subtitle = "Manual_True : data were filled manually and vice versa")
```
![image](https://user-images.githubusercontent.com/109403200/185792785-079a6c12-7555-4934-b694-1332b4dce91e.png)

As shown in above bar chart, 41 records were manually filled in, and 26 of them were filled automatically.

## Phase 5: Share
[Bellabeat Tableau Dashboard](https://public.tableau.com/views/BellabeatCapstoneProject_16611802148200/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link)
![bellabeat tableau](https://user-images.githubusercontent.com/109403200/185954583-a9fc9daf-47cd-4e73-b7c4-06f65f9af18c.png)

## Phase 6: Act
As I stated in the Process phase that a larger and detailed sample size would make this analysis more completed since now we do not know gender or age of the participants. However, after reviewing the analysis, these are the conclusions that I came up with.

* Most users are sedentary and somewhat active, which took up 51.5% of total users. 
* 50% of the users did not wear the Fitbit to sleep for more than 10 days, but almost everyone wore the Fitbit almost everday.
* Users took most steps on Saturday and the number dropped on Sunday. Plus, the number of steps peaked during 5PM to 7 PM.

## Recommendation
* At the moment, the Fitbit bracelet does not have a screen, instead it was designed to be a fashion bracelet. However, in the future design I personally think that a small screen to show some progress during the day would be nice. For example, a user can easily check the Fitbit for number of steps he/she has taken so far that day.
* Another recommended feature that I would like to add is a reminder to stand or move a bit when users have been sitting for too long. This would help increasing active minutes per day.
* Set goal or badge for completing each activity challenge, for instance, hit 8 hours of sleep, exercise for 30 minutes, and take 6000 steps.
* Feature to connect the Fitbit to a compatible smart scales to track weight more efficiently.
