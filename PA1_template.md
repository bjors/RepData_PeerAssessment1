---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

## Loading and preprocessing the data


```r
if (!dir.exists("data")) {
    dir.create("data")
    unzip("activity.zip", exdir="data")
}

library(readr)
df <- read_csv("data/activity.csv")
```

```
## Parsed with column specification:
## cols(
##   steps = col_double(),
##   date = col_date(format = ""),
##   interval = col_double()
## )
```

## What is mean total number of steps taken per day?

**1. Calculate the total number of steps taken per day**

```r
stepsPerDay <- aggregate(steps ~ date, df, sum, na.action = na.omit)
```

**2. Make a histogram of the total number of steps taken each day**

```r
hist(stepsPerDay$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

**3. Calculate and report the mean and median total number of steps taken per day**

```r
mean(stepsPerDay$steps)
```

```
## [1] 10766.19
```

```r
median(stepsPerDay$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

**1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)**

```r
avgStepsPerInterval <- aggregate(steps ~ interval, df, mean, na.action=na.omit)
plot(avgStepsPerInterval$interval, avgStepsPerInterval$steps, type='l')
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

**2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?**

```r
avgStepsPerInterval[which.max(avgStepsPerInterval$steps),]$interval
```

```
## [1] 835
```

## Imputing missing values
**1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)**

```r
nrow(df[!complete.cases(df),])
```

```
## [1] 2304
```

**2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.**  

*Using the mean for that 5-minute interval*

**3. Create a new dataset that is equal to the original dataset but with the missing data filled in.**

```r
library(plyr)
avgStepsPerInterval <- rename(avgStepsPerInterval, c("steps"="mean"))
dfImputed <- join(df, avgStepsPerInterval, by="interval")
naIndices <- which(is.na(dfImputed$steps))
dfImputed[naIndices,]$steps = dfImputed[naIndices,]$mean
```

**4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps? **

```r
stepsPerDayImputed <- aggregate(steps ~ date, dfImputed, sum, na.action = na.omit)
hist(stepsPerDayImputed$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

```r
mean(stepsPerDayImputed$steps)
```

```
## [1] 10766.19
```

```r
median(stepsPerDayImputed$steps)
```

```
## [1] 10766.19
```

## Are there differences in activity patterns between weekdays and weekends?

**1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.**

```r
library(chron)
dfImputed$weekend <- factor(ifelse(is.weekend(dfImputed$date), "weekend", "weekday"))
```

**2. Make a panel plot containing a time series plot (i.e. type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.**

```r
library(lattice)
stepsPerInterval <- aggregate(steps ~ interval + weekend, dfImputed, mean, na.action = na.omit)
xyplot(steps ~ interval | weekend, stepsPerInterval, type = "l", layout = c(1, 2))
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

### Conclusion
* Test subjects wake up earlier on weekdays then moves a bit more in the mornings (perhaps on their way to work?).
* Average nr of steps are slightly higher during weekends

