# Reproducible Research: Peer Assessment 1




## Loading and preprocessing the data

Data comes from cloned repo:

```
git clone https://github.com/rdpeng/RepData_PeerAssessment1.git
```
 
Reading data into R and aggregate the values into StepsByData:

```r
MyActivityData <- read.csv('activity.csv')
StepsByDay <- aggregate(steps ~ date, MyActivityData, sum)
```

## What is mean total number of steps taken per day?

Make a histogram of the total number of steps taken each day

```r
hist(StepsByDay$steps, main = paste("Total Steps / Day"), col="grey", xlab="No Steps")
```

![<Histogram Steps by Day>](<_images/HistStepsByDay.png>)

Calculate and report the mean and median total number of steps taken per day

```r
MyMean <- mean(StepsByDay$steps)
show(MyMean)
[1] 10766.19
```
* The mean is 10766.19.

```r
MyMedian <- median(StepsByDay$steps)
show(MyMedian)
[1] 10765
```

* The median is 10765.

## What is the average daily activity pattern?

Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
StepsByInterval <- aggregate(steps ~ interval, MyActivityData, mean)
plot(StepsByInterval$interval,StepsByInterval$steps, type="l", xlab="Interval", ylab="No Steps",main="Average Steps / Day by Interval")
```

![<Line Diagram Steps by Day by Interval>](<_images/LinePlotStepsInterval.png>)


Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
MyMaxInterval <- StepsByInterval[which.max(StepsByInterval$steps),1]
show(MyMaxInterval)
[1] 835

TimeMaxSteps <-  gsub("([0-9]{1,2})([0-9]{2})", "\\1:\\2", StepsByInterval[MyMaxInterval,'interval'])

```
* The Maximum is at Interval 835.

## Imputing missing values

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
MyNASum <- sum(is.na(MyActivityData$steps))
> show(MyNASum)
[1] 2304
```

* The total of N/A is 2304.


Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

-> I choose to use the 5min interval values for filling N/A gaps.

```r
MyImputedActivity <- merge(MyActivityData,
                          StepsByInterval, 
                          by="interval", 
                          suffixes=c("",".interval.mean"))

MyNAData <- is.na(MyImputedActivity$steps)
MyImputedActivity$steps[MyNAData] <- MyImputedActivity$steps.interval.mean[MyNAData]
MyImputedActivity <- MyImputedActivity[,1:3]

```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 
```r
hist(MyImputedStepsByDay$steps, main = paste("Total Steps / Day"), col="grey", xlab="No Steps", breaks=30)

```

![<Histogram Steps by Day Imputed>](<_images/HistStepsByDayImputed.png>)


Calculate new mean and median for imputed data.

```r
MyImputedMean <- mean(MyImputedStepsByDay$steps)
> show(MyImputedMean)
[1] 10766.19
```

```r
MyImputedMedian <- median(MyImputedStepsByDay$steps)
> show(MyImputedMedian)
[1] 10766.19
```

Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

Value | Before Imputing | After Imputing|
Mean  |  10766.19 | 10766.19|
Median | 10765 | 10766.19|

## Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
myweekday <- function(date) {
    if (weekdays(as.Date(date)) %in% c("Samstag", "Sonntag")) {
        "weekendday"
    } else {
        "weekday"
    }
}

MyImputedActivityData$myweekday <- as.factor(sapply(MyImputedActivityData$date, myweekday))
```

__Remark:__ Took me quite some time to find out that I had non-english locale on my computer and that I had to use the Swiss/German weekday names.

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
xyplot(myImputedStepsByIntervalWeekdays$steps ~ myImputedStepsByIntervalWeekdays$interval|myImputedStepsByIntervalWeekdays$myweekday, main="Average Steps/Day per Interval",xlab="Interval", ylab="Steps",layout=c(1,2), type="l")
```

![<Line Diagram Steps by Day Weekday/Weekend>](<_images/StepsByDayWeekday-Weekendday.png>)


