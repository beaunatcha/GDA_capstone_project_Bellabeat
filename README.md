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

#### ROCCC Analysis
* Reliability - We cannot be 100% sure that it is correct as it was collected from only 30 participants whose gender is unknown.
* Originality - This is a thrid-party data collected via Amazon Mechanical Turk.
* Comprehensive - The dataset contains physical activity data, heart rate, and sleep monitoring in format of day, hour, and minute.
* Current - The dataset was generated from the 12th of April, 2016 to the 12th of May 2016. The data is not current, users' habit may change now.
* Cited - Unknown

#### Limitations of this dataset:
* Only data from 30 users available. Eventhough the central limit theorem rule of n≥30 applies, a larger sample size is preferred in this analysis.

## Phase 3: Process
In this step, we need to make sure that our data is clean, error free and in the right format. 

#### Tools
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
#### Importing the dataset
I will use three main tables: daily activity, sleep day, and weight log.
```
daily_activity <- read.csv("dailyActivity_merged.csv")
daily_steps <- read.csv("dailySteps_merged.csv") !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
day_sleep <- read.csv("sleepDay_merged.csv")
weight_log <- read.csv("weightLogInfo_merged.csv")
```
#### Data cleaning
After reviewing the dataset, I found that the daily activity table has almost all of the information needed for the analysis.
```
# In this step, I am going to use the str() to see the structure of the database.

str(daily_activity)
str(daily_steps) !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
str(day_sleep)
str(weight_log)
```
Then, use `n_distinct()` to check for unique ID in each table, daily activity table has 33 unique IDs, 24 from sleep, and 8 from weight log table. The dataset is inconsistent as we expect 30 unique IDs and some users did not record all their data. The weight log table shows the highest consistency with 22 input missing. This would affect the result of the analysis.

To keep the consistency throughout the table, I use `clean_names()` to change the column name to lowercase and add underscore sign to seperate words
```
daily_activity <- daily_activity %>% clean_names()
day_sleeps <- day_sleep %>% clean_names()
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

First, I will start with checking for the credibiity of the usage of the FitBit.
```
no_usage <- daily_steps %>%
  group_by(id) %>%
  filter(step_total == 0) %>%
  count(step_total == 0)
  
view(no_usage)
```

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
  
 ggplot(activity_level) + geom_bar(aes(y=activity_level, fill=activity_level))
```
![Number of users in each activily level](http://127.0.0.1:51306/graphics/plot_zoom_png?width=1200&height=633)


percentage_user_type <- activity_level %>% 
  group_by(activity_level) %>% 
  summarise(total_no = n()) %>% 
  mutate(totals = sum(total_no)) %>% 
  group_by(activity_level) %>% 
  summarise(total_percent = total_no/totals) %>% 
  mutate(percent = scales::percent(total_percent))

