---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

Reproducible Research: Peer Assessment 1
----------------------------------------



## Loading and preprocessing the data
- Create a folder for ProjectData
- Unzip file if required

```r
path <- "ProjectData"
if (!file.exists(path)) {
        dir.create(path)
}
if (!file.exists(file.path(path, "activity.csv"))) {
        unzip("activity.zip", exdir = path)
}
data <- read.csv(file.path(path, "activity.csv"))
steps.by.day <- aggregate(steps ~ date, data=data, FUN=sum)
```

## What is mean total number of steps taken per day?

```r
hist(steps.by.day$steps, breaks=10, xlab="Number of steps", main="Steps taken per day")
```

![plot of chunk histogram](figure/histogram-1.png) 

## What is the average daily activity pattern?



## Imputing missing values



## Are there differences in activity patterns between weekdays and weekends?
