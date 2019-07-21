---
title: "Reproducible Research Project 1"
author: "tbabola"
date: "7/21/2019"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## R Markdown

Load data

```{r}
data <- read.csv("./Data/activity.csv", header = TRUE)
data$date <- as.Date(data$date,"%Y-%m-%d")
```

### Mean total number of steps taken per day

Calculate the total number of steps taken per day.
```{r}
sumSteps <- aggregate(data$steps, by=list(d = data$date),FUN=sum)
```

Plot the number of steps per day as a histogram.
```{r stepsHist, echo=FALSE}
hist(sumSteps$x,xlab="Number of steps", ylab="Number of days",main="Total number of steps per day")
```

Calculate the mean and median total number of steps taken per day.
```{r}
mean(sumSteps$x,na.rm=TRUE)
median(sumSteps$x,na.rm=TRUE)
```


### Average daily activity pattern

Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis).

```{r}
avgIntervals <- aggregate(data$steps,by=list(interval = data$interval),FUN=mean, na.rm=TRUE)
```
```{r avgInts, echo=FALSE}
plot(avgIntervals$interval,avgIntervals$x,type="l")
```

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```{r}
avgIntervals[avgIntervals$x==max(avgIntervals$x),]$interval
```


### Imputing missing values
Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```{r}
sum(is.na(data$steps))
```

Missing values filled with mean for that 5-minute interval over entire month of data.

```{r}
missingIntervals <- data[is.na(data$steps),'interval']

dataNAfill <- data
dataNAfill[is.na(data$steps),'steps'] = unlist(lapply(missingIntervals,function(x) avgIntervals[avgIntervals$interval == x,'x']))

sumStepsNA <- aggregate(dataNAfill$steps, by=list(d = dataNAfill$date),FUN=sum)
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.


```{r stepsHist2, echo=FALSE}
hist(sumStepsNA$x,xlab="Number of steps", ylab="Number of days",main="Total number of steps per day")
```

Calculate the mean and median total number of steps taken per day with NA values filled in.
```{r}
mean(sumStepsNA$x)
median(sumStepsNA$x)
```

###Differences in weekdays versus weekends

Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
```{r}
dataNAfill$daytype <- unlist(lapply(weekdays(dataNAfill$date),function(x) ifelse(x == "Saturday" | x == "Sunday","weekend","weekday")))
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```{r}
weekdayMean <- aggregate(dataNAfill$steps, by=list(interval = dataNAfill$interval,dt= dataNAfill$daytype),FUN=mean)
```

```{r weekdayPlot, echo=FALSE}
library(ggplot2)
p <- ggplot(weekdayMean,aes(x=interval,y=x, col=dt)) + geom_line()
p + facet_grid(dt~.) + xlab("Interval") + ylab("Mean number of steps") + guides(fill=FLASE)
```
