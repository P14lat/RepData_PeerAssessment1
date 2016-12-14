---
title: "reproducible research - Assessment 1"
output: html_document
---

##Loading the data
1. Load the .csv file
```{r init}
unzip(zipfile = "activity.zip")
initialload <- read.csv("activity.csv")
```
install.packages("Hmisc")
install.packages("ggplot2")
library(ggplot2)
library(Hmisc)


##Mean total number of steps taken per day
```{r daily_steps}
daily_steps <- tapply(initialload$steps, initialload$date, sum, na.rm=T)
```

1. Histogram - total number of steps taken per day
```{r plot1}
library(ggplot2)
library(Hmisc)
library(scales)

qplot(daily_steps, xlab='Total steps per day', ylab='with 1000', binwidth=1000)
```

2. Mean and median of steps taken per day
```{r mean_median}
mean_steps <- mean(daily_steps)
median_steps <- median(daily_steps)
```
Mean: `r mean_steps`

Median: `r median_steps`

##Daily activity pattern
```{r activity_pattern}
avg_per_time <- aggregate(x=list(mean_steps=initialload$steps),
                          by=list(interval=initialload$interval), 
                          FUN=mean, 
                          na.rm=TRUE
                          )
```

1. Plot - time series
```{r plot2}
ggplot(data=avg_per_time, aes(x=interval, y=mean_steps)) +
    geom_line() +
    xlab("interval is 5 minutes") +
    ylab("average steps taken") 
```

2. the 5 minute interval, which contains the max number of steps across the dataset.

```{r maxsteps}
max_steps <- which.max(avg_per_time$mean_steps)
max_steps_t <- gsub("([0-9]{1,2})([0-9]{2})", "\\1:\\2",avg_per_time[max_steps,'interval'])
```
Max steps at: `r max_steps_t ` o'clock.

## Input of missing values
1. report the total number of missing values.
```{r missing_values}
missing_values <- length(which(is.na(initialload$steps)))

```

The number of missing values is `r missing_values `.

2. Strategy to fill in missing values?
-> floor should be fine - alternativ "If in doubt, leave it out"

3. New Dataset which is equal to the orginal, NA need to be filled.

```{r clean}
library(ggplot2)
library(scales)
library(Hmisc)
initl <- initialload
initl$steps <- impute(initialload$steps, fun=mean)
```

4. Histogram - total number of steps taken each day
```{r}
imutedsteps <- tapply(initl$steps, initl$date, sum)
qplot(imutedsteps, xlab='imuted - steps per day', ylab='using with 1000', binwidth=1000)
```

Calculate and report median and mean of total steps taken per day.

```{r}
reinitmean <- mean(imutedsteps)
reinitmedian <- median(imutedsteps)
```

Mean(floor):`r reinitmean`


Median(floor):`r reinitmedian`

##Activity differences between Weekends and weekdays

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```{r}
initl$dateType <- ifelse(as.POSIXlt(initl$date)$wday %in% c(0,5), 
                         'weekend', 
                         'weekday')
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```{r}
avginitl <- aggregate(steps ~ interval + dateType, data=initl, mean)
ggplot(initl, aes(interval, steps)) + 
    geom_line() + 
    facet_grid(dateType ~ .) +
    xlab("Interval is 5 Minutes") + 
    ylab("AVG steps")
```