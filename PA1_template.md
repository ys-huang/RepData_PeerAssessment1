# Reproducible Research: Peer Assessment 1


# Loading and preprocessing the data

```r
data <- read.csv("D:/My Program files/Rspace/RepData_PeerAssessment1/activity.csv")
```


# What is mean total number of steps taken per day?
1. Make a histogram of the total number of steps taken each day

```r
step.day <- aggregate(steps ~ date, data, sum, na.rm = TRUE)
library("ggplot2")
```

```
## Warning: package 'ggplot2' was built under R version 3.1.3
```

```r
ggplot(step.day, aes(x = steps)) + geom_histogram()
```

![](PA1_template_files/figure-html/stepsPerDay-1.png) 
2. Calculate and report the **mean** and **median** total number of steps taken per day

```r
mean(step.day$steps)
```

```
## [1] 10766.19
```

```r
median(step.day$steps)
```

```
## [1] 10765
```


# What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
step.int <- aggregate(steps ~ interval, data, mean, na.rm = TRUE)
ggplot(step.int, aes(x = interval, y = steps )) + geom_line()
```

![](PA1_template_files/figure-html/timeSeriesPlot-1.png) 
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
step.int$interval[which.max(step.int$steps)]
```

```
## [1] 835
```

# Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(data))
```

```
## [1] 2304
```
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in
--Using mean for the day to fill in missing vlues

```r
nas <- is.na(data$steps)
stepMean.day <- aggregate(steps ~ date, data, FUN=mean, na.rm=TRUE)
names(stepMean.day) <- c("date", "steps_mean")

data.f <- data
for (i in 1:nrow(data.f)){
        if (nas[i] == TRUE){
                date <- data.f$date[i]
                date <- as.character(date)
                if (length(grep(date,stepMean.day[,1])) >0){
                         #look for the mean of that date
                        data.f$steps[i] <- stepMean.day[grep(date,stepMean.day[,1]),2]
                }
                else{
                        data.f$steps[i] <- 0 
                }                    
        }
}
```
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
step.day2 <- aggregate(steps ~ date, data.f, sum, na.rm = TRUE)
ggplot(step.day2, aes(x = steps)) + geom_histogram()
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

```r
mean(step.day2$steps)
```

```
## [1] 9354.23
```

```r
median(step.day2$steps)
```

```
## [1] 10395
```
The mean and median are both getting lower after using the mean of the day to fill the missing data. Because in some of the date, there were missing data in the whole day, the missing data thus were filled using 0, and then shift the mean and median.

# Are there differences in activity patterns between weekdays and weekends?
1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” 1indicating whether a given date is a weekday or weekend day.

```r
Sys.setlocale("LC_TIME", "English")
```

```
## [1] "English_United States.1252"
```

```r
data.wk <- data.f
daytype <- function(date) {
    if (weekdays(as.Date(date)) %in% c("Saturday", "Sunday")) {
        "weekend"
    } else {
        "weekday"
    }
}
data.wk$daytype <- as.factor(sapply(data.wk$date, daytype))
```
2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
step.int.wk = aggregate(steps ~ interval + daytype, data.wk, mean)
p<-ggplot(step.int.wk, aes(x = interval, y = steps)) 
p+ geom_line() + facet_wrap( ~ daytype, ncol =1)
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 
