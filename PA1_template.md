---
title: "Project_1"
author: "DS"
date: "8/31/2021"
output: html_document
---



## Introduction
This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The variables included in this dataset are:
- **steps:** Number of steps taking in a 5-minute interval (missing values are coded as *NA*)
- **date:** The date on which the measurement was taken in *YYYY-MM-DD* format
- **interval:** Identifier for the 5-minute interval in which measurement was taken

## Question 1
### First, load libraries, then read the data and preprocess.


```r
library(dplyr)  #used for data manipulation using tidy principles
library(ggplot2)  #used for graphs
library(skimr)  #used to inspect missing values
```


```r
activity <- read.csv("activity.csv")  #assumes file is in working directory

daily_aggregate <- #create aggregate of steps by day
  activity %>%  #using the activity dataset
  filter(!is.na(steps)) %>%  #remove null values
  group_by(date) %>%  #group by the date column in activity
  summarise(daily_steps = sum(steps))  #sum of steps by group

interval_aggregate <- #create aggregate of steps by interval
  activity %>%  #using the activity dataset
  filter(!is.na(steps)) %>%  #remove null values
  group_by(interval) %>%  #group by the interval column in activity
  summarise(daily_steps = sum(steps),  #total steps by interval
            average_steps = mean(steps),  #average steps by interval
            max_steps = max(steps),  #max steps by interval
            min_steps = min(steps))  #min steps by interval
```


## Question 2
### Histogram of the total number of steps taken each day.


```r
hist(daily_aggregate$daily_steps)  #histogram of steps each day
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)

## Question 3
### Mean and median number of steps taken each day.


```r
daily_aggregate %>%  #start with this data set
  summarise(mean_steps = mean(daily_steps),  #calculate mean of steps
            median_steps = median(daily_steps),
            total_steps = sum(daily_steps))  #calculate median of steps
```

```
## # A tibble: 1 x 3
##   mean_steps median_steps total_steps
##        <dbl>        <int>       <int>
## 1     10766.        10765      570608
```

The mean steps taken each day are 10,766.19 steps.
The median steps taken each day are 10,765 steps.
The total steps in the data is 570,608.

## Question 4
### Time series plot of the average number of steps taken.


```r
with(interval_aggregate,  #plot data from this dataset
     plot(x = interval,  #x axis
          y = average_steps,  #y axis
          type = "l",  #line graph
          main = "Average Daily Steps by Interval",  #chart title
          ylab = "Average Steps",  #y axis label
          xlab = "Interval"))  #x axis label
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png)



## Question 5
### Which 5-minute interval, on average across all the days in the dataset, 
### contains the maximum number of steps?


```r
interval_aggregate %>%  #dataset to summarise
  select(interval,  #keep this column
         average_steps) %>%  #keep this column
  filter(average_steps == max(average_steps))  #filter for max average_steps
```

```
## # A tibble: 1 x 2
##   interval average_steps
##      <int>         <dbl>
## 1      835          206.
```

Interval 835 contains the highest average_steps during the day.

## Question 6
### Code to describe and show a strategy for imputing missing data.


```r
#uses skimr package
skim(activity)
```


Table: Data summary

|                         |         |
|:------------------------|:--------|
|Name                     |activity |
|Number of rows           |17568    |
|Number of columns        |3        |
|_______________________  |         |
|Column type frequency:   |         |
|factor                   |1        |
|numeric                  |2        |
|________________________ |         |
|Group variables          |None     |


**Variable type: factor**

|skim_variable | n_missing| complete_rate|ordered | n_unique|top_counts                             |
|:-------------|---------:|-------------:|:-------|--------:|:--------------------------------------|
|date          |         0|             1|FALSE   |       61|201: 288, 201: 288, 201: 288, 201: 288 |


**Variable type: numeric**

|skim_variable | n_missing| complete_rate|    mean|     sd| p0|    p25|    p50|     p75| p100|hist                                     |
|:-------------|---------:|-------------:|-------:|------:|--:|------:|------:|-------:|----:|:----------------------------------------|
|steps         |      2304|          0.87|   37.38| 112.00|  0|   0.00|    0.0|   12.00|  806|??????????????? |
|interval      |         0|          1.00| 1177.50| 692.45|  0| 588.75| 1177.5| 1766.25| 2355|??????????????? |

We can see from the skim of the "activity" dataframe:
- date has no missing values
- steps has 2,304 missing values, 86.9% completion rate
- interval has no missing values
The total number of rows missing data is therefore 2,304.

### Strategy for imputing NA is to use the mean for the respective interval.

```r
#impute using the mean for the interval and create a new dataset identical to 
#the original but with missing values imputed

activity_imputed <-  #create the dataframe with imputed values
  activity %>%  #use this data set
  left_join(.,  #left table is the table in the line above
            interval_aggregate,  #table to left join in
            by = c("interval" = "interval"),  #columns to join on
            suffix = c("", "_2nd"),  #suffixes for duplicate column names
            keep = FALSE) %>% #don't keep both join on columns from by statement
  select(interval,  #keep this column
         steps,  #keep this column
         date,  #keep this column
         average_steps) %>%  #keep this column
  mutate(steps_imputed = if_else(!is.na(steps),  #if steps is not NA
                                        as.numeric(steps),  #then steps
                                        average_steps)) %>%  #else average_steps
  select(steps_imputed,  #keep this column
         date,  #keep this column
         interval) %>%  #keep this column
  rename(steps = steps_imputed)  #rename the column

skim(activity_imputed)
```


Table: Data summary

|                         |                 |
|:------------------------|:----------------|
|Name                     |activity_imputed |
|Number of rows           |17568            |
|Number of columns        |3                |
|_______________________  |                 |
|Column type frequency:   |                 |
|factor                   |1                |
|numeric                  |2                |
|________________________ |                 |
|Group variables          |None             |


**Variable type: factor**

|skim_variable | n_missing| complete_rate|ordered | n_unique|top_counts                             |
|:-------------|---------:|-------------:|:-------|--------:|:--------------------------------------|
|date          |         0|             1|FALSE   |       61|201: 288, 201: 288, 201: 288, 201: 288 |


**Variable type: numeric**

|skim_variable | n_missing| complete_rate|    mean|     sd| p0|    p25|    p50|     p75| p100|hist                                     |
|:-------------|---------:|-------------:|-------:|------:|--:|------:|------:|-------:|----:|:----------------------------------------|
|steps         |         0|             1|   37.38| 105.32|  0|   0.00|    0.0|   27.00|  806|??????????????? |
|interval      |         0|             1| 1177.50| 692.45|  0| 588.75| 1177.5| 1766.25| 2355|??????????????? |

## Question 7
### Histogram of the total number of steps taken each day after missing values 
### are imputed.


```r
daily_aggregate_imputed <- #create aggregate of steps by day
  activity_imputed %>%  #using the activity_imputed dataset
  filter(!is.na(steps)) %>%  #remove null values
  group_by(date) %>%  #group by the date column in activity
  summarise(daily_steps = sum(steps))  #sum of steps by group

hist(daily_aggregate_imputed$daily_steps)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png)

### Report the mean and median total number of steps taken per day with imputed
### values.


```r
daily_aggregate_imputed %>%  #start with this data set
  summarise(mean_steps = mean(daily_steps),  #calculate mean of steps
            median_steps = median(daily_steps),  #calculate median of steps
            total_steps = sum(daily_steps)  #total steps
            )  
```

```
## # A tibble: 1 x 3
##   mean_steps median_steps total_steps
##        <dbl>        <dbl>       <dbl>
## 1     10766.       10766.     656738.
```

The values do not differ much from the earlier part of the assignment before
missing values were imputed.  This is likely due to using a 'mean' method for
the imputation.  The impact on total steps is more obvious, as the total number 
of steps increased due to the missing values being filled in.

## Question 8
### Panel plot comparing the average number of steps taken per 5-minute 
### interval across weekdays and weekends.


```r
weekday_aggregate <-  #create this data frame
  activity_imputed %>%  #starting from this data frame
  mutate(weekend_flag =  #create weekday vs weekend variable
           as.factor(  #create new field as factor
             if_else(weekdays(as.Date(date)) == "Sunday" |
                            weekdays(as.Date(date)) == "Saturday",  #condition
                            "Weekend",  #true result
                            "Weekday")  #false result
             )
        ) %>%
    group_by(interval,  #group by the interval
             weekend_flag) %>%  #and group by the weekend flag
    summarise(daily_steps = sum(steps),  #total steps by group_by
              average_steps = mean(steps)  #average steps by group_by
              )
```

```
## `summarise()` has grouped output by 'interval'. You can override using the `.groups` argument.
```

```r
weekday_aggregate %>%
  ggplot(aes(x = interval,
             y = average_steps)) +
  geom_line() +
  facet_grid(rows = vars(weekend_flag)) +
  labs(title = "Average Steps by Interval",
       subtitle = "Weekend vs Weekday",
       y = "Average Steps",
       x = "Interval")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png)
