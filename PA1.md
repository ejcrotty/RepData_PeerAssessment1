# Reproducible Research Peer Assessment 01
Ed Crotty  
Friday, Jan 06, 2017  
        

```
## Warning: package 'knitr' was built under R version 3.3.2
```

```
## Warning: package 'lattice' was built under R version 3.3.2
```

## Loading and preprocessing the data

Load the data, and ensure that dates are correctly loaded as dates. By default, they will be loaded as strings.


```r
activity <- read.csv("./activity.csv", stringsAsFactors = FALSE)
activity$date <- as.Date(activity$date)
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

```r
names(activity)
```

```
## [1] "steps"    "date"     "interval"
```

## What is mean total number of steps taken per day?

### 1. Create a histogram of the total number of steps taken each day.


```r
steps_per_day <- aggregate(steps ~ date, data = activity, FUN = sum)
barplot(steps_per_day$steps, names.arg = steps_per_day$date)
```

![](PA1_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

### 2. Calculate and report the mean and median total number of steps taken per day


```r
mean(steps_per_day$steps)
```

```
## [1] 10766.19
```

```r
median(steps_per_day$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

Create a data set for steps by interval.


```r
steps_by_interval <- aggregate(steps ~ interval, data = activity, FUN = mean)
```

### 3. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
plot(steps_by_interval$interval, steps_by_interval$steps, type = "l", 
     xlab = "Interval", ylab = "Avg. Steps by Interval")
```

![](PA1_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

### 4. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
steps_by_interval$interval[which.max(steps_by_interval$steps)]
```

```
## [1] 835
```

## Imputing missing values

*Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.*


### 5. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
        

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```



### 6. Devise a strategy for filling in all of the missing values in the dataset. 

* Create an extra column that contains the average steps per 5 min interval
* If the number of steps is `NA` replace the value with the value in the steps_by_interval columns


```r
#Create a new dataset that is equal to the original dataset but with the missing data filled in.
activity2 <-activity
# add the average per interval using merge
# averages already existed above!!
activity2 <- merge(activity, steps_by_interval, by = "interval")
```








### 7. Create a new dataset that is equal to the original dataset but with the missing data filled in.
and confirm that we no longer have `NA` values.

```r
# Now paste the average over the NAs
activity2[is.na(activity2$steps.x),]$steps.x <- activity2[is.na(activity2$steps.x),]$steps.y
sum(is.na(activity2$steps.x))
```

```
## [1] 0
```



### 8. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 


```r
steps_per_day2 <- aggregate(steps.x ~ date, data = activity2, FUN = sum)
barplot(steps_per_day2$steps.x, names.arg = steps_per_day2$date)
```

![](PA1_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

```r
mean(steps_per_day2$steps.x)
```

```
## [1] 10766.19
```

```r
median(steps_per_day2$steps.x)
```

```
## [1] 10766.19
```

*Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?*

The mean is the same and now the mean and median are equal when using an the average for each interval, instead of `NA`.

## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

## 9. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
activity2$day_of_week <- weekdays(activity2$date, abbreviate = TRUE)
days <- c("Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun")
day_types <- c(rep("weekday", 5), rep("weekend", 2))
days_of_week <- data.frame(days, day_types, stringsAsFactors = FALSE)
colnames(days_of_week) <- c("day_of_week", "day_type")
activity2 <- merge(activity2, days_of_week, by = "day_of_week")
activity2$day_type <- as.factor(activity2$day_type)
```

## 10. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
steps_by_interval2 <- aggregate(steps.x ~ interval + day_type, data = activity2, mean)
xyplot(steps.x ~ interval | day_type, data = steps_by_interval2,
type = "l", layout = c(1, 2),
xlab = "Interval", ylab = "Avg. Steps by Interval")
```

![](PA1_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

We see that there are a lot of steps earlier on weekdays ( folks going to work?) than on weekends. 
