---
title: "Reproducible Research: Peer Assessment 1"
---
Author: Peter M�ller, 2015-12-13

## Overview

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

### Loading and preprocessing the data


```r
# loading the data
activity = read.csv("activity.csv", colClasses = c("numeric", "character", "numeric"))
# convert to date
activity$date <- as.Date(activity$date, "%Y-%m-%d")
```

### What is mean total number of steps taken per day?


```r
# total number of steps for each day
dailySteps = aggregate(steps ~ date, data = activity, FUN=sum, na.rm = TRUE)
# draw histogramm
hist(dailySteps$steps, breaks=20, main = "daily steps", xlab = "total steps / day", col = "green")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

The mean is **10766.19** and median is **10765**.

### What is the average daily activity pattern?


```r
# calculate average of each 5 minute interval
dailyPattern = aggregate(steps ~ interval, data = activity, FUN=mean, na.rm = TRUE)
# plot the time series
plot(dailyPattern$interval, 
     dailyPattern$steps, 
     type="l", 
     col="green", 
     lwd = 2,
     xlab="5-minute interval", 
     ylab="average number of steps", 
     main="Time series plot of the average daily activity pattern")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

5-minute interval containing the maximum steps: **835** (206 steps)

### Imputing missing values

Total records: **17568**

NA values: **2304**


```r
# replace NAs with average of the interval in a new dataset
activity1 = activity
for (i in 1:nrow(activity1)) {
    if (is.na(activity1$steps[i])) {
        activity1$steps[i] <- subset(dailyPattern, interval == activity1$interval[i])$steps
    }
}
```

NA values after replacing NA: **0**


```r
# total number of steps for each day
dailySteps = aggregate(steps ~ date, data = activity1, FUN=sum, na.rm = TRUE)
# draw histogramm
hist(dailySteps$steps, breaks=20, main = "daily steps (NA removed)", xlab = "total steps / day", col = "green")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

The mean is **10766.19** and median is **10766.19**.

After replacing NA values the mean and the median are equal to the previously calculated mean.

### Are there differences in activity patterns between weekdays and weekends?


```r
# create function to determin wekkeday or weekend
type <- function(date) {
    if (weekdays(date) %in% c("Saturday", "Sunday", "Samstag", "Sonntag")) {
        "weekend"
    } else {
        "weekday"
    }
}

# add type to dataset
activity1$daytype = as.factor(sapply(activity1$date, type))

# calculate average of each 5 minute interval
weekday = aggregate(steps ~ interval, subset(activity1, daytype == "weekday"), FUN=mean)
weekday$daytype = as.factor("weekday") 
weekend = aggregate(steps ~ interval, subset(activity1, daytype == "weekend"), FUN=mean)
weekend$daytype = as.factor("weekend")
dailyPattern = rbind(weekday, weekend) 

library(lattice)
xyplot(
        steps ~ interval | daytype,
        dailyPattern,
        type = "l",
        layout = c(1,2),
        col="green", 
        lwd = 2,
        xlab="5-minute interval", 
        ylab="average number of steps", 
        main="Time series plot of the average daily activity pattern \ngrouped by weekend/weekdays"
)
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 




