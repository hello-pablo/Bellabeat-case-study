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
2. **Importing datasets**: 
    ```r
    daily_activity <- read.csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/dailyActivity_merged.csv")
   daily_sleep <- read_csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/sleepDay_merged.csv")
    weight <- read_csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/weightLogInfo_merged.csv")
    hourly_intensity <- read_csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/hourlyIntensities_merged.csv")
    hourly_calories <- read_csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/hourlyCalories_merged.csv")
    hourly_steps <- read_csv("~/DATA/Case Study 2/Fitabase 4.12.16-5.12.16/hourlySteps_merged.csv") 
 
 - Removing null values and inconsistencies.
      - Formatting dates and times using `lubridate`.

3. **Exploratory Data Analysis**:
    - Analyzed user activity and sleep patterns.
    - Generated visualizations to highlight trends.

4. **Insights**:
    - Most users were more active on weekdays.
    - Sleep duration was highest on weekends.

## Visualizations
Below are key plots generated during the analysis:

1. Average activity level by day of the week:
![Activity Plot](figures/activity_by_day.png)

2. Sleep duration distribution:
![Sleep Plot](figures/sleep_duration.png)

## Conclusion
Bellabeat could focus on promoting weekend relaxation features and fitness challenges during weekdays to align with user behavior.

## Files
- `analysis/`: R scripts for data cleaning and analysis.
- `figures/`: Plots and visualizations.
- `data/`: (Contains raw and cleaned datasets).

## Author
[Your Name](https://github.com/tuo-username)
