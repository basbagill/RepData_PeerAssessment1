# PA1_template.Rmd
John Basbagill  
February 28, 2017  

Course 5 Week 2 Course Project

1 Code for reading in the dataset and/or processing the data


```r
## Create data directory if it doesn't exist 
if(!file.exists("./data")) {
  dir.create("./data")
  }

## Read data
activity <- read.csv("activity.csv")
```

2 Histogram of the total number of steps taken each day


```r
## Sum total number of steps taken each day
steps.total <- aggregate(steps ~ date, activity, FUN = sum, na.rm = TRUE)

## Create histogram of total number of steps taken each day 
hist(steps.total$steps, main="Histogram of Steps Taken Each Day", xlab="Steps Taken Each Day", 
		ylab="Frequency")
```

![](PA1_template_files/figure-html/histogram of total steps-1.png)<!-- -->

3 Mean and median number of steps taken each day


```r
## Get mean and median total steps taken each day
mean(steps.total$steps)
```

```
## [1] 10766.19
```

```r
median(steps.total$steps)
```

```
## [1] 10765
```

4 Time series plot of the average number of steps taken


```r
## Get steps taken for each 5-minute interval averaged across all days
steps.interval.mean <- aggregate(steps ~ interval, activity, FUN = mean, na.rm = TRUE)

## Plot time-series of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
plot(steps.interval.mean$interval, steps.interval.mean$steps, type="l", xlab="5-minute daily interval", ylab="Average steps across all days", 
		main="Time Series of Average Steps Taken by 5-Minute Interval")
```

![](PA1_template_files/figure-html/time series of 5-min intervals vs. average steps-1.png)<!-- -->

5 The 5-minute interval that, on average, contains the maximum number of steps


```r
## Get 5-minute interval containing maximum number of steps averaged over all days
interval.max <- steps.interval.mean[which.max(steps.interval.mean$steps), ]
interval.max$interval
```

```
## [1] 835
```

6 Code to describe and show a strategy for imputing missing data

STRATEGY The strategy is to first determine the number of rows containing NAs. Then a new dataset will be created that will replace NAs with imputed data. The imputed data will be the mean for that 5-minute interval averaged over all days. A for loop will be used to replace each NA in the dataset. In the loop, each row will be checked for an NA value. If NA, the NA will be replaced with the average 5-minute interval.


```r
## install.packages("plyr", repos = "http://cran.us.r-project.org")
library(plyr)
```

Calculate the total number of rows with NAs

```r
## Calculate total number of rows with NAs
steps.na <- count(activity$steps == "NA")
steps.na[2,2]
```

```
## [1] 2304
```

Impute the missing data.

```r
## Create new dataset equal to original dataset
activity.new = activity

## Impute missing data. Replace each row of NA values of new dataset with mean for that 5-minute interval averaged over all days
for (i in 1:nrow(activity.new)) {  ## Loop through each row in new dataset
	if(is.na(activity.new$steps[i])) {  ## Check if row has NA value
		activity.new$steps[i] = steps.interval.mean$steps[((i-1)%%nrow(steps.interval.mean))+1] 
				## Replace NA value with value from that 5-minute interval averaged over all days (found in         ## steps.interval.mean). Note '%%' computes remainder, since number of rows in 
		    ## steps.interval.mean is equal to number of 5-minute intervals. This is less than number 
		    ## of rows in activity.new, so current row is divided by number of rows in 
		    ## steps.interval.mean to index appropriate row in steps.interval.mean.
	}
}
```

7 Histogram of the total number of steps taken each day after missing values are imputed


```r
## Create histogram of total number of steps taken each day with imputed data.
steps.new.total <- aggregate(steps ~ date, activity.new, FUN = sum)
hist(steps.new.total$steps, main="Histogram of Steps Taken Each Day - Includes Imputed Data", 
		xlab="Steps Taken Each Day", ylab="Frequency")
```

![](PA1_template_files/figure-html/histogram of total steps with imputed data-1.png)<!-- -->

The mean and median total number of steps taken per day are calculated as follows.

```r
## Get mean and median total steps taken each day
mean(steps.new.total$steps)
```

```
## [1] 10766.19
```

```r
median(steps.new.total$steps)
```

```
## [1] 10766.19
```

Do these values differ from the estimates from the first part of the assignment? The values hardly changed from the estimates from the first part of the assignment. The mean steps did not change, whereas the median steps increased by 1.

What is the impact of imputing missing data on the estimates of the total daily number of steps?
Imputing missing data increases the estimates of the total daily number of steps.

8 Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends


```r
## Reformat dates so zeros are appended to days with single digits (e.g., "10/5/12" becomes "10/05/12")
activity.new$newday = as.Date(activity.new$date, "%m/%d/%y")

## Create factor variable in the dataset with two levels – “weekday” and “weekend” 
activity.new$day.of.week = weekdays(activity.new$newday)
for (i in 1: nrow(activity.new)) { ## Loop through the data replacing each day with "weekday" or                                      ## "weekend"
		if (activity.new$day.of.week[i] == "Saturday" | activity.new$day.of.week[i] == "Sunday") {
			activity.new$day.of.week[i] = "weekend" 
			} else {activity.new$day.of.week[i] = "weekday"
			} 
}

## Get steps taken for each 5-minute interval averaged across all weekdays or weekends
steps.byday.byinterval.mean <- aggregate(steps ~ interval + day.of.week, activity.new, FUN = mean)

## Create panel plot containing time series plot of 5-minute interval and average number of steps taken, averaged across all weekday days or weekend days

## install.packages("lattice", repos = "http://cran.us.r-project.org")
library(lattice)
xyplot(steps~interval|day.of.week, type = "l", layout = c(1,2), main = "Difference in Activity 
       Patterns between Weekdays and Weekends", data = steps.byday.byinterval.mean)
```

![](PA1_template_files/figure-html/panel plot of average steps across weekdays and weekends-1.png)<!-- -->
