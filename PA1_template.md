# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
*From the assignment page:*
This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.


```r
## make sure any libraries we need are installed, then load them into workspace
pkgs = c("downloader", "ggplot2", "dplyr") ## use "downloader" package from CRAN
if(length(new.pkgs <- setdiff(pkgs, rownames(installed.packages())))) 
  install.packages(new.pkgs, repos="http://cran.rstudio.com/")
suppressMessages(library(downloader))
suppressMessages(library(ggplot2))
suppressMessages(library(dplyr))

## setup file handles, and get the file and unzip if not in working directory
fileURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
zipfile <- "repdata_data_activity.zip"
if(!file.exists(zipfile)) {
  download(fileURL, dest=zipfile, mode="wb")
  unzip(zipfile, exdir = "./")
}

activity <- read.csv("activity.csv")
```

We are only given the date that particular intervals occured on. We will have need for what day of the week it might be, so will add a "day" variable to our observations. Additionally the intervals for each day as given are integers of the form:

**(0,5,10,15,20,25,30,35,40,45,50,50,100,105,...,2350,2355)**

We will convert the integers into a "time" variable of form hh:mm, which will designate the hour and minute start of an interval for that day.


```r
activity$date <- as.Date(activity$date, format = "%Y-%m-%d")

# to convert integer interval to hh:mm, we use technique found on stackoverflow:
## http://stackoverflow.com/questions/25272457/convert-an-integer-column-to-time-hhmm
activity$time <-as.POSIXct(sprintf("%04d", activity$interval), format="%H%M") # make 4 digit with leading zeros
```

## What is mean total number of steps taken per day?

Per instruction, ignoring "NA". First we find the total steps per day.


```r
steps <- activity %>% 
  group_by(date) %>%
  summarise(total = sum(steps, na.rm = TRUE))
```

Now a simple histogram

```r
hist(steps$total, freq = TRUE, breaks = 12, main = "Histogram of Steps Taken per Day", 
     xlab = "Number of Steps per Day", col = "green")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

and the Mean for all the days:

```r
mean(steps$total, na.rm=TRUE)
```

```
## [1] 9354.23
```
and the Median for all of the days:

```r
median(steps$total, na.rm=TRUE)
```

```
## [1] 10395
```

## What is the average daily activity pattern?


```r
intervals <- activity %>% 
  group_by(time) %>%
  summarise(avg = mean(steps, na.rm = TRUE))

plot(intervals$time, intervals$avg, type = "l",
    xlab = "5-minute intervals", ylab = "Average Number of Steps Taken")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

We note that maximal aveverage activity occured:

```r
max.avg <- intervals[which.max(intervals$avg),]
```

Which shows that the maximum average was 206.1698113 which occured in the interval 08:35.

## Imputing missing values

First we determine how many "NA" values occur in the dataframe. We can do this easily with a simple **sapply**

```r
totMissing <- sapply(activity, function(y) sum(is.na(y)))
totMissing
```

```
##    steps     date interval     time 
##     2304        0        0        0
```

From which we see that "steps" is the only column with missing values, of which there are `r totMissing[1]' of them.

Our approach for filling in step data for a particular missing interval will be to fill it in with the average of all non NA for that particular interval. For example assume we are missing the 09:30 to 09:35 interval for a particular day. We will fill in that missing datum with the average of all the non-missing 09:30 to 09:35 intervals from all the other days. Obviously not a robust method, but within assignment guidance.


```r
## now process dataframe supply missing data with weekday/interval averages
noNA <- activity %>%
  left_join(intervals, by = "time") %>%
  mutate(steps = ifelse(is.na(steps), avg, steps))
```

We repeat the histogram, but now use our dataset that contains imputed variables:

```r
steps <- noNA %>% 
  group_by(date) %>%
  summarise(total = sum(steps))

hist(steps$total, freq = TRUE, breaks = 12, main = "Histogram of Steps Taken per Day (no NA)", 
     xlab = "Number of Steps per Day", col = "green")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

and the Mean for all the days with no NA:

```r
mean(steps$total, na.rm=TRUE)
```

```
## [1] 10766.19
```
and the Median for all the days with no NA:

```r
median(steps$total, na.rm=TRUE)
```

```
## [1] 10766.19
```


## Are there differences in activity patterns between weekdays and weekends?