Reproducible Research first assign
================
YanirFigovich
4 בדצמבר 2018

loading the data
----------------

loading the data to R and libraries

``` r
data<- read.csv("activity.csv")
library(dplyr)
```

    ## Warning: package 'dplyr' was built under R version 3.4.4

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(lubridate)
```

    ## Warning: package 'lubridate' was built under R version 3.4.4

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

``` r
library(tidyr)
```

    ## Warning: package 'tidyr' was built under R version 3.4.4

``` r
library(ggplot2)
```

    ## Warning: package 'ggplot2' was built under R version 3.4.4

``` r
library(gridExtra)
```

    ## Warning: package 'gridExtra' was built under R version 3.4.4

    ## 
    ## Attaching package: 'gridExtra'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

ploting number of steps
-----------------------

preparing and summarising the data in order to make the plot

``` r
data$date<-as_date(data$date)
agg_data<- group_by(data,date)
agg_data<- summarise(agg_data,steps_per_day=sum(steps))
plot1<- ggplot(agg_data)+geom_histogram(aes(steps_per_day),fill="white",color="blue")
plot1<-plot1+ labs(y="number of days")
plot(plot1)
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

    ## Warning: Removed 8 rows containing non-finite values (stat_bin).

![](assign_1_1_files/figure-markdown_github/make%20plot-1.png) \#\#find mean and median steps per day

``` r
mean_steps_day<-mean(agg_data$steps_per_day,na.rm = T)
mean_steps_day
```

    ## [1] 10766.19

``` r
med_steps_day<- median(agg_data$steps_per_day,na.rm = T)
med_steps_day
```

    ## [1] 10765

plot average steps for each interval
------------------------------------

``` r
agg_data<- group_by(data,interval)
agg_data<- summarise(agg_data,avg_steps_per_interval=mean(steps,na.rm = T))
```

    ## Warning: package 'bindrcpp' was built under R version 3.4.4

``` r
plot2<-ggplot(agg_data)+geom_line(aes(interval,avg_steps_per_interval),lwd=1.1,color="blue")
plot(plot2)
```

![](assign_1_1_files/figure-markdown_github/plot%20average%20steps-1.png) \#\#find interval with max steps on average

``` r
filter(agg_data,avg_steps_per_interval==max(avg_steps_per_interval))
```

    ## # A tibble: 1 x 2
    ##   interval avg_steps_per_interval
    ##      <int>                  <dbl>
    ## 1      835                   206.

imputing NA
-----------

first we would like to know where and how much NA we have in the data. then we will impute the mean where ever we find na

``` r
#finding the NA
sapply(data,function(x){sum(is.na(x))})
```

    ##    steps     date interval 
    ##     2304        0        0

``` r
##imputing avg steps where steps==NA
#calculating the avg steps for each interval
agg_data<- group_by(data,interval)
agg_data<- summarise(agg_data,avg_steps_per_interval=mean(steps,na.rm = T))
#mreging the base data with the avg steps
merge_data<- left_join(data,agg_data,by="interval")
#computing a new column to select the avg if steps=NA
new_data<- mutate(merge_data,comp_steps=ifelse(is.na(steps),avg_steps_per_interval,steps))
#see some of the new data
head(new_data)
```

    ##   steps       date interval avg_steps_per_interval comp_steps
    ## 1    NA 2012-10-01        0              1.7169811  1.7169811
    ## 2    NA 2012-10-01        5              0.3396226  0.3396226
    ## 3    NA 2012-10-01       10              0.1320755  0.1320755
    ## 4    NA 2012-10-01       15              0.1509434  0.1509434
    ## 5    NA 2012-10-01       20              0.0754717  0.0754717
    ## 6    NA 2012-10-01       25              2.0943396  2.0943396

plot average steps for each interval after adding avg steps insted of NA
------------------------------------------------------------------------

``` r
agg_by_day<- group_by(new_data,date)
agg_by_day<-summarise(agg_by_day,comp_steps_day=sum(comp_steps))
plot3<- ggplot(agg_by_day)+geom_histogram(aes(comp_steps_day),fill="white",color="blue")
plot3<-plot3+labs(y="number of days")
plot(plot3)
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](assign_1_1_files/figure-markdown_github/plot%20steps%20per%20day-1.png) \#\#plot average steps per interval comparing weekdays VS weekends we will first add anew column to our data with numeric representive of the day of the week. then we will separate for 2 tables: one for week days and the other for weekends. then we will plot a histogram for each table

``` r
##adding the numeric weekday
days_data<- mutate(data,week_day=wday(date))
## a table for weekdays
weekdays_data<- filter(days_data,week_day<6)
##aggragating by interval mean
agg_weekdays<- group_by(weekdays_data,interval)
agg_weekdays<- summarise(agg_weekdays,avg_steps_per_interval=mean(steps,na.rm = T))
## a table for weekends
weekdEnds_data<- filter(days_data,week_day>5)
##aggragating by interval mean
agg_weekEnds<- group_by(weekdEnds_data,interval)
agg_weekEnds<- summarise(agg_weekEnds,avg_steps_per_interval=mean(steps,na.rm = T))
plot4<-ggplot(agg_weekdays)+geom_line(aes(interval,avg_steps_per_interval),lwd=1.1,color="blue")+labs(title = "weekday")
plot4<-plot4 + scale_y_continuous(limits=c(0, 220))
plot5<-ggplot(agg_weekEnds)+geom_line(aes(interval,avg_steps_per_interval),lwd=1.1,color="blue")+labs(title = "week ends")
plot5<-plot5 + scale_y_continuous(limits=c(0, 220))
pp<-grid.arrange(plot4,plot5,nrow=2)
```

![](assign_1_1_files/figure-markdown_github/plot%20compering%20weekendsan%20weekdays-1.png)