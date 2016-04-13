Loading and preprocessing the data
----------------------------------

Before beggining, the working directory must be set where the file
`activity.csv` is located. This can be done with the setwd() command.
Loading packages *dplyr*, *lubridate* and *ggplot2*. Reading the table
and transforming date variable into POSIXct.

    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(lubridate)
    library(ggplot2)

    activity <- read.csv("activity.csv")
    activity$date <- ymd(activity$date)

What is mean total number of steps taken per day?
-------------------------------------------------

Using dplyr *group\_by* and *summarize*, creating a dataframe containing
the total number of steps per day. Then creating histogram with the
qplot function.

    activity_day <- (activity %>% group_by(date))

    totalsteps <- summarize(activity_day, sum_steps=sum(steps, na.rm=TRUE))

    qplot(sum_steps, data=totalsteps, xlab="Total number of steps per day")

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-2-1.png)<!-- -->

What is mean total number of steps taken per day?
-------------------------------------------------

The mean and median number of steps per day are calculated with the same
dataframe.

    meansteps <- mean(totalsteps$sum_steps, na.rm=TRUE)
    mediansteps <- median(totalsteps$sum_steps, na.rm=TRUE)

The mean is **9354.2295082**. The median is **10395**

What is the average daily activity pattern?
-------------------------------------------

This time, we are grouping the dataframe by interval with *group\_by*.

    activity_interval <- (activity %>% group_by(interval))

    meaninterval <- summarize(activity_interval, mean=mean(steps, na.rm=TRUE))

    qplot(interval, mean, data=meaninterval, geom="line")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-4-1.png)<!-- -->

    maxinterval <- filter(meaninterval, mean==max(mean))$interval

The 5-minute interval that, on average, contains the maximum number of
steps is interval **835**.

Imputing missing values
-----------------------

Calculate and report the total number of missing values in the dataset
(i.e. the total number of rows with NAs)

    NA_number <- sum(is.na(activity$steps))

The number of missing value is **2304**.

Filling the missing values for each interval with the mean for that 5-minute interval.
--------------------------------------------------------------------------------------

We are merging the dataframe activity and meaninterval. For each row, we
now have the original steps value (which may be NA) and the mean value
for that interval (mean variable). For each step value, creating the
fill variable containing the mean value in case of NA value or
containing 0 if the value is not NA. Finally, adding this value to the
steps value.

    activity_filled <- merge(activity, meaninterval, by="interval")

    ## column "fill" containing value for filling
    activity_filled$fill <- is.na(activity_filled$steps)*activity_filled$mean
    activity_filled$steps <- rowSums(activity_filled[,c("steps","fill")], na.rm=TRUE)

    ## Create a new dataset that is equal to the original dataset but with the missing data filled in.

    activity_filled <- select(activity_filled, 1:3)

    activity_filled_day <- (activity_filled %>% group_by(date))

    totalsteps_filled <- summarize(activity_filled_day, sum_steps=sum(steps))

    ## Histogram
    qplot(sum_steps, data=totalsteps_filled, xlab="Total number of steps per day")

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)<!-- -->

mean and median value of the filled dataframe
---------------------------------------------

    meansteps_f <- mean(totalsteps_filled$sum_steps)
    mediansteps_f <- median(totalsteps_filled$sum_steps)

The mean is
**1.076618910<sup>{4}**\\ The\\ median\\ is\\ **1.076618910</sup>{4}**
Mean and median steps are higher than before.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Create a new factor variable in the dataset with two levels: weekday and
weekend indicating whether a given date is a weekday or weekend day.

    weekday <- function(x) {
            day <- weekdays(x)
            if (day %in% c("segunda-feira", "terÃ§a-feira", "quarta-feira",
                           "quinta-feira", "sexta-feira"))
                    "weekday"
            else "weekend"
    }

    activity_filled$day <- sapply(activity_filled$date, weekday)
    activity_filled$day <- as.factor(activity_filled$day)

Make a panel plot containing a time series plot (i.e. type = "l") of the
5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis). We are
grouping the dataframe by interval and day type (weekday and weekend).
Then making a plot with ggplot and facet\_wrap.

    activity_day <- (activity_filled %>% group_by(interval, day) %>% summarize(mean_steps = mean(steps)))

    g <- ggplot(activity_day, aes(x= interval, y=mean_steps))
    g + geom_line(aes(color=day)) + facet_wrap("day", nrow=2) + 
            labs(x = "5-minute interval", y= "Number of steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-10-1.png)<!-- -->

We can notice that movement is less frequent during weekend. During,
weekday, there is a plike between 8:00 a.m and 9:00 a.m, corresponding
with the subject getting ready and going to work. During weekend, much
less movement at that same time.
