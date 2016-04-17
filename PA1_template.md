#---
#title: "Reproducible Research: Peer Assessment 1"
#output: 
#  html_document:
#    keep_md: true
#---

#setwd("C:\\Projects\\reproducable_research")

## Loading and preprocessing the data

unzip(zipfile = "activity.zip")
activityData <- read.csv("activity.csv",stringsAsFactors=FALSE)

## What is mean total number of steps taken per day?

library(ggplot2)
stepsCount <- tapply(activityData$steps,activityData$date,FUN=sum,na.rm=TRUE )
qplot(stepsCount,binwidth=1000,xlab="total number of steps taken each day")
mean(stepsCount,na.rm=TRUE)
median(stepsCount,na.rm = TRUE)

## What is the average daily activity pattern?

averageSteps <- aggregate(x=list(steps=activityData$steps),by=list(interval=activityData$interval),FUN=mean,na.rm= TRUE )
ggplot(data=averageSteps, aes(x=interval, y=steps)) +
  geom_line() +
  xlab("5-minute interval") +
  ylab("average number of steps taken")
averageSteps[which.max(averageSteps$steps),]

## Imputing missing values

sum(is.na(activityData$steps))
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
mean(filledStepsCount)
median(filledStepsCount)

## Are there differences in activity patterns between weekdays and weekends?

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


