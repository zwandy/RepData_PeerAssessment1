---
title: "Coursera Data Sci Course 5 - Project1"
author: "zwandy"
date: "June 2015"
output: html_document
---

This is the R markdown file created for Coursera Data Science series course #5 -
[Reproducible Research](https://www.coursera.org/course/repdata).

It was initiated by Fork/Clone of the Github repository at 
https://github.com/rdpeng/RepData_PeerAssessment1 - this repository contains the  
required datafile file for the project (activity.zip).

The following headings indicate the project/assignment steps.

---

## Loading and preprocessing the data
Show any code that is needed to

### *1. Load the data (i.e. read.csv())*


```r
##required package
if (!require(dplyr)){ 
    install.packages('dplyr')
  }
  library(dplyr)

##the data
data <- read.csv(unzip('activity.zip'))
```


### *2. Process/transform the data (if necessary) into a format suitable for 
your analysis*


```r
##a look at the data shows us there are 3 columns and that there are many 
##NA values in the 'steps' column
head(data)
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
##number of rows
nrow(data)
```

```
## [1] 17568
```

```r
##for the first few steps, we will ignore NA values
nonNA <- complete.cases(data)
data.nonNA <- data[nonNA, ]
```


---

## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the 
dataset.

### *1. Make a histogram of the total number of steps taken each day*


```r
##non-NA data, total steps by day
days <- group_by(data.nonNA, date)
byday <- summarize(days, totalsteps=sum(steps))

hist(byday$totalsteps, breaks=10, main='Frequency of Total Daily Steps 
     (ignoring NA values)', 
     xlab='Total Daily Steps')
```

<img src="PA1_template_files/figure-html/unnamed-chunk-15-1.png" title="" alt="" width="672" />

The above graph appears to show us that, on more days, daily step count is around 10,000 to 15,000 total steps.


### *2. Calculate and report the mean and median total number of steps taken per 
day*


```r
summary(byday$totalsteps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8840   10800   10800   13300   21200
```

```r
options(scipen=1, digits=0); mean(byday$totalsteps, na.rm=TRUE)
```

```
## [1] 10766
```

```r
median(byday$totalsteps, na.rm=TRUE)
```

```
## [1] 10765
```

The mean total daily steps is 10766. The median total daily steps is 10765.


---

## What is the average daily activity pattern?


### *1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x
-axis) and the average number of steps taken, averaged across all days (y
-axis)*

There are 288 5-min intervals/observations per day ( (60mins/5mins)*24hours = 288 5min).

The data source numbers these 288 daily intervals from 0 to 2355.


```r
##non-NA data, avg steps by interval
intervals <- group_by(data.nonNA, interval)
byinterval <- summarize(intervals, avgsteps=mean(steps))

plot(byinterval$interval, byinterval$avgsteps, type="l", main="Average Steps 
     by 5-min Interval Across All Days (ignoring NA values)", 
     xlab="5-min Interval", ylab="Average Steps")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-17-1.png" title="" alt="" width="672" />

```r
summary(byinterval$avgsteps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0       2      34      37      53     206
```


### *2. Which 5-minute interval, on average across all the days in the dataset,
contains the maximum number of steps?*


```r
maxsteps <- max(byinterval$avgsteps)
maxstepsinterval <- byinterval$interval[byinterval[,"avgsteps"]==maxsteps]
##translate the interval to a clock time of day
timel <- nchar(maxstepsinterval)
time <- ''
if (maxstepsinterval==0) {
  time <- '12:00 AM'
}else if (maxstepsinterval==5){
  time <- '12:05 AM'
}else if (timel==2){
  time <- paste0('12:',maxstepsinterval,' AM')
}else if (timel==3){
  time <- paste0(substr(maxstepsinterval,1,1),':',substr(maxstepsinterval,2,3),
                 ' AM')
}else if ((timel==4) & (maxstepsinterval<1200)){
  time <- paste0(substr(maxstepsinterval,1,2),':',substr(maxstepsinterval,3,4),
                 ' AM')
}else if ((timel==4) & (maxstepsinterval<1300)){
  time <- paste0(substr(maxstepsinterval,1,2),':',substr(maxstepsinterval,3,4),
                 ' PM')
}else {
  hour <- as.integer(substr(maxstepsinterval,1,2))-12
  time <- paste0(hour,':',substr(maxstepsinterval,3,4),' PM')
}
```

On average, the greatest number of steps taken during a 5-min interval on a 
single day is around 206 at the 835 interval (or 
8:35 AM).


---

## Imputing missing values
Note that there are a number of days/intervals where there are missing values 
(coded as NA). The presence of missing days may introduce bias into some 
calculations or summaries of the data.

### *1. Calculate and report the total number of missing values in the dataset 
(i.e. the total number of rows with NAs)*


```r
##number of rows with NA present
nas <- is.na(data)
data.NAs <- data[nas, ]
nrow(data.NAs)
```

```
## [1] 2304
```


### *2. Devise a strategy for filling in all of the missing values in the 
dataset. The strategy does not need to be sophisticated. For example, you could 
use the mean/median for that day, or the mean for that 5-minute interval, etc.*

The strategy that will be used to account for NA step values will be **the mean of the 5-minute interval** for the entire data set (ignoring NA values).


### *3. Create a new dataset that is equal to the original dataset but with the 
missing data filled in.*


```r
##set the 'step' value to the value for the same interval in the list of
##avg step-by-interval dataframe (byinterval)
x <- 1
while (x <= nrow(data.NAs)) {
  datinterval <- data.NAs[x,'interval']
  data.NAs[x,'steps']=byinterval$avgsteps[byinterval[,'interval']==datinterval]
  x <- x+1
}

##combine the non-NA and fixed NA dataframes into one dataframe
data.fixed <- rbind(data.nonNA, data.NAs)
data.fixed <- arrange(data.fixed, date) ##asc order, like the original dataframe
```


### *4. Make a histogram of the total number of steps taken each day and 
Calculate and report the mean and median total number of steps taken per day. Do 
these values differ from the estimates from the first part of the assignment? 
What is the impact of imputing missing data on the estimates of the total daily 
number of steps?*


```r
days.fixed <- group_by(data.fixed, date)
byday.fixed <- summarize(days.fixed, totalsteps=sum(steps))
hist(byday$totalsteps, breaks=10, main='Frequency of Total Daily Steps', 
     xlab='Total Daily Steps (fixed for NAs)')
```

<img src="PA1_template_files/figure-html/unnamed-chunk-21-1.png" title="" alt="" width="672" />

```r
##summary of daily step data before NAs fixed
summary(byday$totalsteps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8840   10800   10800   13300   21200
```

```r
##summary of daily step data after NAs set to dataset means
summary(byday.fixed$totalsteps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    9820   10800   10800   12800   21200
```
By setting the missing values (NAs) to their dataset mean, there is little 
impact on the median or mean daily step count. The first quartile, however, is 
slightly larger with the replaced NAs, and the third quartile is slightly 
smaller, which makes the spread of our data slightly narrower than it was before 
the NAs were replaced with their means.


---

## Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays() function may be of some help here. Use the dataset 
with the filled-in missing values for this part.

### *1. Create a new factor variable in the dataset with two levels -- "weekday" 
and "weekend" indicating whether a given date is a weekday or weekend day.*


```r
data.fixed <- mutate(data.fixed, weekday=weekdays(as.POSIXlt(date)))
data.fixed <- mutate(data.fixed, daytype='NA')

x <- 1
while (x <= nrow(data.fixed)) {
  day <- data.fixed[x, 'weekday']
  if (identical(day,'Saturday') || identical(day,'Sunday')) {
    data.fixed[x, 'daytype']<-'weekend'
  }else{
    data.fixed[x, 'daytype']<-'weekday'
  }
  x <- x+1
}

##the new dataframe
head(data.fixed)
```

```
##   steps       date interval weekday daytype
## 1     2 2012-10-01        0  Monday weekday
## 2     0 2012-10-01        5  Monday weekday
## 3     0 2012-10-01       10  Monday weekday
## 4     0 2012-10-01       15  Monday weekday
## 5     0 2012-10-01       20  Monday weekday
## 6     2 2012-10-01       25  Monday weekday
```


### *2. Make a panel plot containing a time series plot (i.e. type = "l") of the 
5-minute interval (x-axis) and the average number of steps taken, averaged 
across all weekday days or weekend days (y-axis).*


```r
data.we <- filter(data.fixed, daytype=='weekend')
data.wd <- filter(data.fixed, daytype=='weekday')

intervals.we <- group_by(data.we, interval)
byweinterval <- summarize(intervals.we, avgsteps=mean(steps))

intervals.wd <- group_by(data.wd, interval)
bywdinterval <- summarize(intervals.wd, avgsteps=mean(steps))

par(mfrow=c(2,1))
plot(byweinterval$interval, byweinterval$avgsteps, type="l", main="Average Steps 
     by 5-min Interval Across All Weekend Days", xlab="5-min Interval", 
     ylab="Avg. Steps")
plot(bywdinterval$interval, bywdinterval$avgsteps, type="l", main="Average Steps 
     by 5-min Interval Across All Weekday Days", xlab="5-min Interval", 
     ylab="Avg. Steps")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-23-1.png" title="" alt="" width="672" />

The above graphs appear to show us that the highest number of steps taken on 
weekday days is higher than the highest number of steps taken on weekend days; 
however, there are more steps taken (more spikes in the number of steps) 
throughout the day on weekends. The higher number of steps on weekday days are 
only in the mornings.

Hope everyone is (finishing their homework, but also) having an active weekend!



