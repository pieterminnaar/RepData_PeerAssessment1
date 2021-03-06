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
library(lattice)
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
Replace missing values with for steps with the mean

```r
complete.data <- inner_join(data, interval.means)
```

```
## Joining by: "interval"
```

```r
complete.data.derived <- mutate(complete.data, derived.steps = ifelse ((is.na(steps)), mean, steps))
derived.steps.by.day <- aggregate(derived.steps ~ date, data=complete.data.derived, FUN=sum)
hist(derived.steps.by.day$derived.steps, breaks=10, xlab="Number of steps", main="Steps taken per day")
```

![plot of chunk derive_values_for_na_from_mean](figure/derive_values_for_na_from_mean-1.png) 

## Are there differences in activity patterns between weekdays and weekends?

```r
valid.steps.weekdays <- mutate(valid.steps, 
                               day_type = ifelse(weekdays(as.Date(date)) == 'Saturday' | 
                                                 weekdays(as.Date(date)) == 'Sunday', 
                                                 'Weekend', 'Weekday'))
valid.steps.weekdays$day_type <- as.factor(valid.steps.weekdays$day_type)
grouped.steps.day_types <- group_by(valid.steps.weekdays, interval, day_type)
interval.day_type.means <- summarise(grouped.steps.day_types, mean(steps))
names(interval.day_type.means) <- c('interval', 'day_type', 'mean_steps')
attach(interval.day_type.means)
xyplot(mean_steps ~ interval | day_type, 
       panel = panel.lines,
       xlab = "Interval", ylab = "Number of steps", 
       main = "Steps per interval by type of day")
```

![plot of chunk compare_weekdays](figure/compare_weekdays-1.png) 
