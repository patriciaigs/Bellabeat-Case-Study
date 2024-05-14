# Bellabeat: Case Study

Patrícia Santos

2024-05-14

- <a href="#background-information"
  id="toc-background-information">Background Information</a>
- <a href="#data-preparation" id="toc-data-preparation">Data
  Preparation</a>
- <a href="#data-exploration" id="toc-data-exploration">Data
  Exploration</a>
- <a href="#recommendations" id="toc-recommendations">Recommendations</a>

## Background Information

**Bellabeat** is a high-tech company that manufactures health-focused smart products. Collecting data on activity, sleep, stress, and reproductive health has allowed Bellabeat to empower women with knowledge about their own health and habits.

### Stakeholders 

- Urška Sršen - Bellabeat cofounder and Chief Creative Officer
- Sando Mur - Bellabeat cofounder and key member of Bellabeat executive team
- Bellabeat Marketing Analytics team

### Business Task

Identify trends in the usage of smart devices to discover potential opportunities to improve the Bellabeat marketing strategy. 

### About the Data

The data used in this case study can be found [here](https://www.kaggle.com/datasets/arashnic/fitbit?source=post_page-----c18835475563--------------------------------)

These datasets were made available through Mobius in Kaggle. Licensed under the CCO Public Domain. It’s an open source data. According to the information available on Zenodo.org, these datasets where generated by respondents to a distributed survey via Amazon Mechanical Turk between 03.12.2016 and 05.12.2016. Contains personal tracker data from thirty eligible Fitbit users, including minute-level output for physical activity, heart rate, and sleep monitoring. 

### Data Organization and Credibility

Available for download are 18 CSV files. Each document has different quantitative data tracked by Fitbit. The data is stored in long format with each ID having data in multiple rows, since data is tracked by day and time. 

By sorting and filtering with Pivot Tables in Excel I verified that the data does not pass the integrity and credibility test. Zenodo.org informs us that we have 30 eligible Fitbit users consented to the submission of data, when in reality we can find 33 users in the dataset. Also stated was a timeframe between 03.12.2016 and 05.12.2016 but we only have 31 days represented. Another problem is that the dataset is not current. 

The sample size presents a bias. If we had a larger sample size it would have been more representative of the population. It does not have any demographic information such as gender or age and it makes it impossible to understand if it is a true representation. 

This will impact any recommendations provided after the analysis.

## Data Preparation

After checking each dataset information, for this analysis, I will be using the following datasets:
- daily_activity_merged
- hourlysteps_merged
- sleep_day_merged

Weight and Heart Rate were not considered due to the small sample presented (8 and 7 users). 

The cleaning and analysis of the data was done in R due to accessibility and ability to provide visualizations so I can share my results.

##### 1. Installing packages

```{r message = FALSE}
install.packages("tidyverse", repos = "http://cran.us.r-project.org")
library(tidyverse)
install.packages("here", repos = "http://cran.us.r-project.org")
library(here)
install.packages("skimr", repos = "http://cran.us.r-project.org")
library(skimr)
install.packages("janitor", repos = "http://cran.us.r-project.org")
library(janitor)
install.packages("lubridate", repos = "http://cran.us.r-project.org")
library(lubridate)
install.packages("ggplot2", repos = "http://cran.us.r-project.org")
library(ggplot2)
install.packages("dplyr", repos = "http://cran.us.r-project.org")
library(dplyr)
install.packages("tidyr", repos = "http://cran.us.r-project.org")
library(tidyr)
```

##### 2. Importing the Datasets

```{r message = FALSE}
daily_activity <- read_csv("Downloads/archive/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
hourly_steps <- read_csv("Downloads/archive/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/hourlySteps_merged.csv")
daily_sleep <- read_csv("Downloads/archive/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")
```

##### 3. Sorting the Data 

```{r}
n_distinct(daily_activity$Id)
n_distinct(hourly_steps$Id)
n_distinct(daily_sleep$Id)
```

[1] 33

[1] 33

[1] 24

##### 4. Checking for Duplicates

```{r}
sum(duplicated(daily_activity))
sum(duplicated(hourly_steps)) 
sum(duplicated(daily_sleep))
```
[1] 0

[1] 0

[1] 3

```{r}
# cleaning 3 duplicates found in the daily_sleep dataset, remaining with 410 unique values after cleaning. 
daily_sleep <- daily_sleep %>%
  distinct() %>%
  drop_na()
```
##### 5. Formatting Date and Time Columns 

The column with date and time was cleaned on the daily_activity and daily_sleep. We will be turning it into a data type to ensure consistency and make analysis easier later on. 

```{r}
daily_activity$ActivityDate <-as.Date(daily_activity$ActivityDate, format = "%m/%d/%Y")
daily_sleep$SleepDay <-as.Date(daily_sleep$SleepDay, format = "%m/%d/%Y")
```

In the dataset hourly_steps the ActivityHour column string was converted to date time. 

```{r}
hourly_steps<- hourly_steps %>% 
    rename(date_time = ActivityHour) %>% 
     mutate(date_time = as.POSIXct(date_time,format ="%m/%d/%Y %I:%M:%S %p" , tz=Sys.timezone()))
```

##### Formatting Column Names

To make analysis easier later on the column Activity_Hour (daily_activity) and SleepDay (daily_sleep) will both be renamed to date. 

```{r}
daily_activity = rename(daily_activity, date = ActivityDate)
daily_sleep = rename(daily_sleep, date = SleepDay)
```

##### View the Datasets 

```{r}
head(daily_activity)
head(hourly_steps)
head(daily_sleep)
```
![daily_activity](https://github.com/patriciaigs/images/blob/main/daily_activity.png?raw=true)

![daily_sleep](https://github.com/patriciaigs/images/blob/main/daily_sleep.png?raw=true)

![hourly_steps](https://github.com/patriciaigs/images/blob/main/hourly_steps.png?raw=true)

## Data Exploration

##### Summary Statistics

First let's take a look at some summary statistics of the datasets. 

```{r}
# activity
daily_activity %>%  
  select(TotalSteps,
              TotalDistance,
         	Calories) %>%
    summary()

# explore number of active minutes per category
daily_activity %>%
  select(VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes, SedentaryMinutes) %>%
  summary()

# daily_sleep
daily_sleep %>%
  select(TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed) %>%
  summary()
```

![summary_one](https://github.com/patriciaigs/images/blob/main/summary%201.png?raw=true)

![summary_two](https://github.com/patriciaigs/images/blob/main/summary%202.png?raw=true)

By analyzing this data we can already discover some important information:

  •	Observing the number of active minutes per category we can see that the sedentary column has the highest average: 991 minutes, or 16 hours.
  
  •	Average total steps per day are 7638. 
  
  •	On average, participants sleep 1 time for 7 hours. 

##### Defining Types of Users 

For this definition, since we don’t have demographic variables, I will classify our types of users by activity considering their daily number of steps. 
The categorization will be: 

  •	Sedentary - Less than 5000 steps a day.
  
  •	Lightly active - Between 5000 and 7499 steps a day.
  
  •	Fairly active - Between 7500 and 9999 steps a day.
  
  •	Very active - More than 10000 steps a day.
  
According to the article that can be found [here](https://www.10000steps.org.au/articles/healthy-lifestyles/counting-steps/).

```{r}
# finding the daily average number of steps by user 

daily_average <- daily_activity %>%
group_by(Id) %>%
summarise (mean_daily_steps = mean(TotalSteps))
```

```{r}
head(daily_average)
```




