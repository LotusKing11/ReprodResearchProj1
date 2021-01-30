R Markdown
----------

This is an R Markdown document. Markdown is a simple formatting syntax
for authoring HTML, PDF, and MS Word documents. For more details on
using R Markdown see
<a href="http://rmarkdown.rstudio.com" class="uri">http://rmarkdown.rstudio.com</a>.

When you click the **Knit** button a document will be generated that
includes both content as well as the output of any embedded R code
chunks within the document. You can embed an R code chunk like this:

Reproducible Research: Peer Assessment 1
========================================

\#\#1.Loading and preprocessing the data

##### A. Load the required packages

##### B.Load the data

``` r
if(!file.exists('activity.csv')){
  unzip('activity.zip')
}
activityData <- read.csv('activity.csv')
```

##### C. transform interval data

``` r
#activityData$interval <- strptime(gsub("([0-9]{1,2})([0-9]{2})", "\\1:\\2", activityData$interval), format='%H:%M')
```

------------------------------------------------------------------------

2. What is mean total number of steps taken per day?
----------------------------------------------------

##### A. Calculate the total number of steps taken per day

``` r
StepsPerDay <- tapply(activityData$steps, activityData$date, sum, na.rm=TRUE)
```

##### B. Histogram of the total number of steps taken each day

``` r
qplot(StepsPerDay, xlab='Total Steps per Day', ylab='Frequency using binwidth 500', binwidth=500)
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-5-1.png)

##### C. Calculate and report the mean and median of the total number of steps taken per day

``` r
MeanStepsPerDay <- round(mean(StepsPerDay))
MedianStepsPerDay <- round(median(StepsPerDay))
print(paste("The mean is: ", MeanStepsPerDay))
```

    ## [1] "The mean is:  9354"

``` r
print(paste("The median is: ", MedianStepsPerDay))
```

    ## [1] "The median is:  10395"

------------------------------------------------------------------------

\#\#3. What is the average daily activity pattern?

``` r
avgStepsPer5Min <- aggregate(x=list(meanSteps=activityData$steps), by=list(interval=activityData$interval), FUN=mean, na.rm=TRUE)
```

##### A. Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

``` r
ggplot(data=avgStepsPer5Min, aes(x=interval, y=meanSteps)) +
  geom_line() +
  xlab("5-minute intervals") +
  ylab("Average number of steps taken") 
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-8-1.png)
\#\#\#\#\# B. Which 5-minute interval, on average across all the days in
the dataset, contains the maximum number of steps?

``` r
maxstepsinterval <- which.max(avgStepsPer5Min$meanSteps)
timemaxsteps <-  gsub("([0-9]{1,2})([0-9]{2})", "\\1:\\2", avgStepsPer5Min[maxstepsinterval,'interval'])
print(paste("The 5 minute interval with highest number of average steps is at: ", timemaxsteps, "AM"))
```

    ## [1] "The 5 minute interval with highest number of average steps is at:  8:35 AM"

------------------------------------------------------------------------

\#\#4.Imputing Missing values

##### A. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with

``` r
TotalMissingValues <- length(which(is.na(activityData$steps)))
print(paste("The Total Number of Missing Values in the dataset is: ",TotalMissingValues))
```

    ## [1] "The Total Number of Missing Values in the dataset is:  2304"

##### B. Devise a strategy for filling in all of the missing values in the dataset.

``` r
activityDataImputed <- activityData
```

##### C. Create a new dataset that is equal to the original dataset but with the missing data filled in.

``` r
activityDataImputed$steps <- impute(activityData$steps, fun=mean)
```

##### D. Histogram of the total number of steps taken each day

``` r
StepsPerDayImputed <- tapply(activityDataImputed$steps, activityDataImputed$date, sum)
qplot(StepsPerDayImputed, xlab='Total steps per day (Imputed)', ylab='Frequency using binwidth 250', binwidth=250)
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-13-1.png)

##### E. Calculate and report the mean and median total number of steps taken per day.

``` r
MeanStepsPerDayImputed <- mean(StepsPerDayImputed)
MedianStepsPerDayImputed <- median(StepsPerDayImputed)
print(paste("The imputed mean is: ", MeanStepsPerDayImputed))
```

    ## [1] "The imputed mean is:  10766.1886792453"

``` r
print(paste("The imputed median is: ", MedianStepsPerDayImputed))
```

    ## [1] "The imputed median is:  10766.1886792453"

------------------------------------------------------------------------

5. Are there differences in activity patterns between weekdays and weekends?
----------------------------------------------------------------------------

##### A. Create a new factor variable in the dataset with two levels ??? ???weekday??? and ???weekend??? indicating whether a given date is a weekday or weekend day.

``` r
activityDataImputed$dateType <-  ifelse(as.POSIXlt(activityDataImputed$date)$wday %in% c(0,6), 'weekend', 'weekday')
```

##### B. Panel plot containing a time series plot

``` r
averageActivityDataImputed <- aggregate(steps ~ interval + dateType, data=activityDataImputed, mean)
ggplot(averageActivityDataImputed, aes(interval, steps)) + 
  geom_line() + 
  facet_grid(dateType ~ .) +
  xlab("5-minute interval") + 
  ylab("avarage number of steps")+ ggtitle("Average Activity Weekdays vs Weekends")
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-16-1.png)
