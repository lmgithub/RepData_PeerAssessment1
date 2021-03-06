# Reproducible Research: Peer Assessment 1

Peer Assessment 1 project for [Reproducible Research](https://www.coursera.org/course/repdata) course on [Coursera](https://www.coursera.org/).

## Data description
This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.  
The variables included in the dataset are:
* steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)
* date: The date on which the measurement was taken in YYYY-MM-DD format
* interval: Identifier for the 5-minute interval in which measurement was taken


## Requirements
Requirements for Peer Assessment 1 are described [here](https://github.com/rdpeng/RepData_PeerAssessment1/blob/master/README.md).

## Loading and preprocessing the data

1. Load the data.

```r
dt <- read.csv("activity.csv")
```

2. Process the data into a format suitable for analysis.

Check variables

```r
names(dt)
```

```
## [1] "steps"    "date"     "interval"
```

Check number of records

```r
nrow(dt)
```

```
## [1] 17568
```

Make clean data set (without NA values) and convert steps to numeric

```r
dtClean <- dt[!is.na(dt$steps), ]
dtClean$steps <- as.numeric(as.character(dtClean$steps))
```

Check number of records of clean data set

```r
nrow(dtClean)
```

```
## [1] 15264
```

## What is mean total number of steps taken per day?

1. Histogram of the total number of steps taken each day for clean data without missed values.

```r
## Calculate total number of steps taken each day
stepsTotalPerDay <- tapply(dtClean$steps, as.factor(dtClean$date), sum)
## Build histogram
hist(stepsTotalPerDay, 
     xlab="Total number of steps taken each day", 
     main="Histogram of the total number of steps taken each day", 
     sub=" for clean data without missed values", 
     col="blue")
```

![](PA1_template_files/figure-html/StepsTotalPerDayHist-1.png) 

2. The mean and median total number of steps taken per day for clean data without missed values.

```r
stepsMeanPerDay <- mean(stepsTotalPerDay, na.rm=TRUE)
stepsMedianPerDay <- median(stepsTotalPerDay, na.rm=TRUE)
```
Mean: 10766.19  
Median: 10765

## What is the average daily activity pattern?

1. Time series plot of the 5-minute interval and the average number of steps taken, averaged across all days.

```r
## Compute average number of steps per intervals
stepsMeanPerInterval <- aggregate(dtClean$steps, list(dtClean$interval), mean)
## Set column names
names(stepsMeanPerInterval) <- c("interval", "mean_steps")
## Build a plot
plot(stepsMeanPerInterval$interval, stepsMeanPerInterval$mean_steps, 
     type="l", 
     xlab="Interval", ylab="Average number of steps", 
     main="Average number of steps per 5-minute intervals", 
     col="blue")
```

![](PA1_template_files/figure-html/StepsMeanPerInterval-1.png) 

2. The 5-minute interval, on average across all the days in the dataset,  which contains the maximum number of steps.


```r
stepsMeanPerInterval$interval[which.max(stepsMeanPerInterval$mean_steps)]
```

```
## [1] 835
```

## Imputing missing values

1. Total number of missing values in the dataset.

```r
sum(is.na(dt$steps))
```

```
## [1] 2304
```

2. Create new data set based on original and fill in the missing values with average steps for that 5-minute interval valies.

```r
## Copy data set
dtFilled <- dt
## merge new data set with average steps for that 5-minute interval
dtFilled <- merge(dtFilled, stepsMeanPerInterval, by = "interval", sort=FALSE)
## Fill out missing values with average steps for that 5-minute interval
dtFilled$steps[is.na(dtFilled$steps)] <- dtFilled$mean_steps[is.na(dtFilled$steps)] 
## Remove mean_steps column
dtFilled <- subset(dtFilled, select=-c(mean_steps))
```

3. Histogram of the total number of steps taken each day for data with filled in missed values.

```r
## Calculate total number of steps taken each day
stepsTotalPerDayFilled <- tapply(dtFilled$steps, as.factor(dtFilled$date), sum)
## Build histogram
hist(stepsTotalPerDayFilled, 
     xlab="Total number of steps taken each day", 
     main="Histogram of the total number of steps taken each day", 
     sub="for data with filled in missed values", 
     col="red")
```

![](PA1_template_files/figure-html/StepsTotalPerDayHistFilled-1.png) 

4. The mean and median total number of steps taken per day for data with filled in missed values.

```r
stepsMeanPerDayFilled <- mean(stepsTotalPerDayFilled, na.rm=TRUE)
stepsMedianPerDayFilled <- median(stepsTotalPerDayFilled, na.rm=TRUE)
```
Mean: 10766.19  
Median: 10766.19

Mean number of steps is not changed because of we filled in the missing values with mean values.
Median number of steps is changed.

The impact of imputing missing data on the estimates of the total daily number of steps is to make the data more close to reality because when we don't count intervals with missed values we lose steps which probably was made in those intervals. 


## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable "weekday" in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
## I use POSIXlt$wday instead of weekday() function from assessment requirements because of weekday() is language sensetive and POSIXlt$wday returns number, which is more useful in this case.
dtFilled$weekday <- ifelse(as.POSIXlt(dtFilled$date, format="%Y-%m-%d")$wday %in% c(0,6), "weekend", "weekday")
```

2. Time series plot of the 5-minute interval and the number of steps taken, averaged across all weekday days or weekend days.


```r
## Compute average number of steps per intervals
stepsMeanIW <- aggregate(dtFilled$steps, list(dtFilled$weekday, dtFilled$interval), mean)
## Set column names
names(stepsMeanIW) <- c("weekday", "interval", "mean_steps")
## Build a plot
library(ggplot2)
qplot(interval, mean_steps, data=stepsMeanIW, 
      geom="path", color="red", 
      xlab="Interval", 
      ylab="Average number of steps", 
      main="") + facet_wrap(~weekday, ncol=1) + theme(legend.position="none")
```

![](PA1_template_files/figure-html/stepsMeanPerIntervalAndWeekday-1.png) 

