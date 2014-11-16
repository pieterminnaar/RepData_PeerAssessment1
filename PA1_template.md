---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

Uses dplyr library


```r
library(dplyr)
library(knitr)
```


```r
options(scipen = 1, digits = 2)
```

## Loading and preprocessing the data
- Create a folder for ProjectData
- Unzip file if required

```r
path <- "ProjectData"
if (!file.exists(path)) {
        dir.create(path)
}
if (!file.exists(file.path(path, "activity.csv"))) {
        unzip("activity.zip", exdir=path)
}
data <- read.csv(file.path(path, "activity.csv"))
# Uses dplyr library
steps.by.day <- aggregate(steps ~ date, data=data, FUN=sum)
```

## What is mean total number of steps taken per day?

```r
hist(steps.by.day$steps, breaks=10, xlab="Number of steps", main="Steps taken per day")
```

![plot of chunk histogram](figure/histogram-1.png) 
 
Test all days for complete interval values. A day with partly filled 
with interval values would give skew results for mean or median.

```r
incomplete.days <- summarise(group_by(filter(data, is.na(steps)), date), count = n())
complete.dates <- data.frame(setdiff(data$date, incomplete.days$date))
names(complete.dates) <- c("date")
complete.days <- group_by(inner_join(data, complete.dates), date)
```

```
## Joining by: "date"
```

```r
steps.sum.by.day <- summarise(complete.days, total = sum(steps))
mean.steps.per.day <- mean(steps.sum.by.day$total)
median.steps.per.day <- median(steps.sum.by.day$total)
```

####Mean steps per day: 10766.19.

####Median steps per day: 10765.


## What is the average daily activity pattern?

```r
valid.steps <- filter(data, !is.na(steps))
grouped.steps <- group_by(valid.steps, interval)
interval.means <- summarise(grouped.steps, mean(steps))
names(interval.means) <- c('interval', 'mean')
plot(interval.means$interval, interval.means$mean, 
     main = 'Average steps per interval',
     xlab = '5 minute interval', 
     ylab = 'Number of steps')
lines(interval.means$interval, interval.means$mean, type='l')
```

![plot of chunk plot_for_steps](figure/plot_for_steps-1.png) 

```r
max_steps <- max(interval.means$mean)
```

####Interval period with highest average: 835

## Imputing missing values



## Are there differences in activity patterns between weekdays and weekends?
