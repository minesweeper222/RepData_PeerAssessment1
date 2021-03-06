# Reproducible Research: Peer Assessment 1

Loading and preprocessing the data


```r
unzip(zipfile = "activity.zip")
activityData <- read.csv("activity.csv",stringsAsFactors=FALSE)
```

What is mean total number of steps taken per day?


```r
library(ggplot2)
stepsCount <- tapply(activityData$steps,activityData$date,FUN=sum,na.rm=TRUE )
qplot(stepsCount,binwidth=1000,xlab="total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
mean(stepsCount,na.rm=TRUE)
```

```
## [1] 9354.23
```

```r
median(stepsCount,na.rm = TRUE)
```

```
## [1] 10395
```

What is the average daily activity pattern?


```r
averageSteps <- aggregate(x=list(steps=activityData$steps),by=list(interval=activityData$interval),FUN=mean,na.rm= TRUE )
ggplot(data=averageSteps, aes(x=interval, y=steps)) +
  geom_line() +
  xlab("5-minute interval") +
  ylab("average number of steps taken")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
averageSteps[which.max(averageSteps$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```

Imputing missing values


```r
sum(is.na(activityData$steps))
```

```
## [1] 2304
```

```r
f_naFill <- function(steps, interval) {
  filled <- NA
  if (!is.na(steps))
    filled <- c(steps)
  else
    filled <- (averageSteps[averageSteps$interval==interval, "steps"])
  return(filled)
}
filledActivity <- activityData
filledActivity$steps <- mapply(f_naFill, filledActivity$steps, filledActivity$interval)

filledStepsCount <- tapply(filledActivity$steps, filledActivity$date, FUN=sum)
qplot(filledStepsCount, binwidth=1000, xlab="total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
mean(filledStepsCount)
```

```
## [1] 10766.19
```

```r
median(filledStepsCount)
```

```
## [1] 10766.19
```

Are there differences in activity patterns between weekdays and weekends?


```r
weekday_or_weekend <- function(date) {
  day <- weekdays(date)
  if (day %in% c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday"))
    return("weekday")
  else if (day %in% c("Saturday", "Sunday"))
    return("weekend")
  else
    stop("invalid date")
}
filledActivity$date <- as.Date(filledActivity$date)
filledActivity$day <- sapply(filledActivity$date, FUN=weekday_or_weekend)
averages <- aggregate(steps ~ interval + day, data=filledActivity, mean)
ggplot(averages, aes(interval, steps)) + geom_line() + facet_grid(day ~ .) +
  xlab("5-minute interval") + ylab("Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

