# Reproducible Data Project 1
Rick Flake  
2017-07-16 07:49:35  



```r
# include this code chunk as-is to set options
knitr::opts_chunk$set(comment=NA, prompt=TRUE)
library(Rcmdr)
```

```
## Loading required package: splines
```

```
## Loading required package: RcmdrMisc
```

```
## Loading required package: car
```

```
## Loading required package: sandwich
```

```
## The Commander GUI is launched only in interactive sessions
```

```r
library(car)
library(RcmdrMisc)
library(mice)
library(data.table)
library(chron)
library(ggplot2)
```



```r
> # include this code chunk as-is to enable 3D graphs
> library(rgl)
> knitr::knit_hooks$set(webgl = hook_webgl)
```
1) Code for reading in the dataset and/or processing the data
*Download data zip file from https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip
*Extract activity data file to local drive
*Run the following code to load data



```r
> activity <- 
+   read.table("C:/Users/Rick/Documents/Coursera/Reproducible Research/activity.csv",
+    header=TRUE, sep=",", na.strings="NA", dec=".", strip.white=TRUE)
```
2) Histogram of the total number of steps taken each day


```r
> activity <- data.table(activity)
> setkey(activity,date)
> dactivity <- as.data.frame(activity[, sum(steps, na.rm = TRUE),by = date])
> names(dactivity)[c(2)] <- c("steps")
> with(dactivity, Hist(steps, scale="frequency", breaks="Sturges", col="darkgray"))
```

![](Reproducible_Research_Project1_files/figure-html/unnamed-chunk-4-1.png)<!-- -->
3) Mean and median number of steps taken each day


```r
> meanday <- with(dactivity,mean(steps))
> medianday <- with(dactivity,median(steps))
> mactivity <- as.data.frame(activity[, mean(steps, na.rm = TRUE),by = date])
> mdactivity <- as.data.frame(activity[, median(steps, na.rm = TRUE),by = date])
> names(mactivity)[c(2)] <- c("meansteps")
> names(mdactivity)[c(2)] <- c("mediansteps")
> stactivity <-merge(mactivity,mdactivity,all.x = TRUE)
```
Mean of days is 9354.2295082 and Median of days is 10395.


4) Time series plot of the average number of steps taken


```r
> with(activity, plotMeans(steps, date, error.bars="none", connect=TRUE))
```

![](Reproducible_Research_Project1_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

5) The 5-minute interval that, on average, contains the maximum number of steps


```r
> setkey(activity,interval)
> iactivity <- as.data.frame(activity[, mean(steps, na.rm = TRUE),by = interval])
> names(iactivity)[c(2)] <- c("steps")
> subset(iactivity,steps == max(steps))
```

```
    interval    steps
104      835 206.1698
```

6) Code to describe and show a strategy for imputing missing data


```r
> # calculating imputed data. It uses linear reqression against the existing values
> library(mice)
> imactivity <- mice(data = activity, m = 5, method = "pmm", maxit = 5, seed = 500)
```

```

 iter imp variable
  1   1  steps
  1   2  steps
  1   3  steps
  1   4  steps
  1   5  steps
  2   1  steps
  2   2  steps
  2   3  steps
  2   4  steps
  2   5  steps
  3   1  steps
  3   2  steps
  3   3  steps
  3   4  steps
  3   5  steps
  4   1  steps
  4   2  steps
  4   3  steps
  4   4  steps
  4   5  steps
  5   1  steps
  5   2  steps
  5   3  steps
  5   4  steps
  5   5  steps
```

```r
> #completing data using the 2nd of 5 result sets
> completeactivity <- complete(imactivity,2)
```

7) Histogram of the total number of steps taken each day after missing values are imputed


```r
> completeactivity <- data.table(completeactivity)
> setkey(completeactivity,date)
> dcompactivity <- as.data.frame(completeactivity[, sum(steps, na.rm = TRUE),by = date])
> names(dcompactivity)[c(2)] <- c("steps")
> with(dcompactivity, Hist(steps, scale="frequency", breaks="Sturges", col="darkgray"))
```

![](Reproducible_Research_Project1_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

8) Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends. 



```r
> library(chron)
> completeactivity1 <- completeactivity[,daynum:=wday(date)]
> completeactivity1[,weekend:=is.weekend(date)]
>  Ciactivity <- as.data.frame(completeactivity1[, mean(steps, na.rm = TRUE),keyby = .(interval,weekend)])
> names(Ciactivity)[c(3)] <- c("meansteps")
> Ciactivity <- data.table(Ciactivity)
> Ciactivity$weekend <- replace(Ciactivity$weekend,Ciactivity$weekend =='TRUE','Weekend')
> Ciactivity$weekend <- replace(Ciactivity$weekend,Ciactivity$weekend =='FALSE','Weekday')
> library(lattice)
> xyplot(data=Ciactivity, meansteps~interval|weekend, type ='l',layout = c(1,2))
```

![](Reproducible_Research_Project1_files/figure-html/unnamed-chunk-10-1.png)<!-- -->


