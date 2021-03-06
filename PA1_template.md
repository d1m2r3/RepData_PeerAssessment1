# PA1 Reproducible Research Coursera
d1m2r3  
Sunday, November 16, 2014  

This is a submission for Peer Assessment 1 (PA1), Reproducible Research course,
offered by  Coursera.

This R Markdown file is submitted into a GitHub repository for this assigment.
This repository is  forked from the official GitHub repository of this course.
It already contains the assigment description, so the details are not 
repeated here.

###Preliminary: Loading and preprocessing the data

*Show any code that is needed to
1. Load the data (i.e. read.csv() )
2. Process/transform the data (if necessary) into a format suitable for your analysis*


The zip-file "repdata_data_activity.zip" contains the data for this project.   
Read the file "activity.csv" out of this zip-file using *unz()* function.  
Convert it into a  data frame using *read.csv()*. 



```r
f_pa1 <- "./activity.zip"

df1 <- read.csv(unz(f_pa1, "activity.csv"))

#Examine its column names and dimensions
names(df1); dim(df1)
```

```
## [1] "steps"    "date"     "interval"
```

```
## [1] 17568     3
```


Now, we are ready to work with the data frame.


### Part 1: What is mean total number of steps taken per day?

*For this part of the assignment, you can ignore the missing values in the dataset.  
1. Make a histogram of the total number of steps taken each day  
2. Calculate and report the mean and median total number of steps taken per day*  



#### 1.a. Histogram

We need only first two columns of *df1*; and, we can ignore rows with "NA".  


```r
#Extract the 1st two columns
q1_df1 <-  df1[,1:2]

#Extract the complete cases out of this
q1_df2 <- q1_df1[complete.cases(q1_df1),]

#Examine the data frame for plotting the histogram
names(q1_df2); dim(q1_df2)
```

```
## [1] "steps" "date"
```

```
## [1] 15264     2
```

Use the *aggregate()*  function to sum the number of steps taken by day. Since 
there are 288 intervals of 5-minute duration over a day, we should get this many rows. 
The  *hist()* function produces the desired histogram.


```r
q1_a <- aggregate(x = q1_df2$steps,
                  by = list(q1_df2$date),
                  FUN = sum)

#Examine this data frame

names(q1_a); dim(q1_a); head(q1_a)
```

```
## [1] "Group.1" "x"
```

```
## [1] 53  2
```

```
##      Group.1     x
## 1 2012-10-02   126
## 2 2012-10-03 11352
## 3 2012-10-04 12116
## 4 2012-10-05 13294
## 5 2012-10-06 15420
## 6 2012-10-07 11015
```

```r
#Give meaningful names to this data frame

names(q1_a) <- c("date", "totalSteps")

hist(q1_a$totalSteps, 
     xlim = c(0,25000),
     ylim = c(0,20),
     nclass = 15,
     main = "Histogram of number of steps in  a day",
     xlab = "Number of steps per day ",
     ylab = "Count of days")
```

![](./PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

```r
#Also, write to file
plot1 <- "./figure/histo_1.png"


png(plot1, width = 480, height = 480, units = "px")

hist(q1_a$totalSteps, 
     xlim = c(0,25000),
     ylim = c(0,20),
     nclass = 15,
     main = "Histogram of number of steps in  a day",
     xlab = "Number of steps per day ",
     ylab = "Count of days")

dev.off()
```

```
## pdf 
##   2
```

####1.b. Mean and median number of steps taken per day 




The mean and median number of steps taken per day are simply calculated on the column *q1_a$totalSteps*:
mean = 10766.2, and median = 10765. 

###Part 2. What is the average daily activity pattern?

*1. Make a time series plot (i.e. type = "l" ) of the 5-minute interval (x-axis) and the average number of
steps taken, averaged across all days (y-axis)  
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number
of steps?*  

#### Part 2.1. Time series plot

Aggregate the data by interval, with function as "mean".   
The resultant data frame, *q2_a*, is used for this part.  
Give meaningful names to the columns of this aggregation, and do a  simple plot.


```r
#Make a data frame for this part
#Re-use the cleaned-up data frame from part 1
df2 <- df1[complete.cases(df1),]

q2_a <- aggregate(x = df2$steps,
                  by = list(df2$interval),
                  FUN = mean)

#As there are 288 intervals of 5-min per day, you should get 288 rows.
dim(q2_a);
```

```
## [1] 288   2
```

```r
#Give meaningful names to the columns of the aggregated data frame
names(q2_a) <- c("intervalStart", "AvgSteps")

#Try a simple plot

plot(q2_a$intervalStart, q2_a$AvgSteps,
     main = "Average number of steps over 5-min interval",
     xlab = "Interval Start", 
     ylab = "Average Number of steps in 5-min interval",
     type = "l")
```

![](./PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

```r
#Also, write to file
plot1 <- "./figure/histo_2.png"


png(plot1, width = 480, height = 480, units = "px")

plot(q2_a$intervalStart, q2_a$AvgSteps,
     main = "Average number of steps over 5-min interval",
     xlab = "Interval Start", 
     ylab = "Average Number of steps in 5-min interval",
     type = "l")


dev.off()
```

```
## pdf 
##   2
```

####2.b. Time interval with maximum number of steps

This is a straight-forward calculation on the *AvgSteps* column of data frame *q2_a*. 

```r
#max(q2_a$AvgSteps)
rMax <- which.max(q2_a$AvgSteps)

#Obtain the starting minute index of that row
maxStartTime <- q2_a$intervalStart[rMax]

maxHr <- maxStartTime %/% 100
maxMin <- maxStartTime %% 100
```

The maximum number of steps are recorded in the 5 minute interval starting 8:35 hours.



### Part 3: Imputing missing values

*Note that there are a number of days/intervals where there are missing values (coded as NA ). The
presence of missing days may introduce bias into some calculations or summaries of the data.   
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with
NA s)  
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be
sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute
interval, etc.  
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.  
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and
median total number of steps taken per day. Do these values differ from the estimates from the first
part of the assignment? What is the impact of imputing missing data on the estimates of the total daily
number of steps?  *  


#### 3.1: Number of rows with missing values (NA's)

```r
NA_rows <-  nrow(df1) - nrow(df2)
```

It is simply the number of rows in original data frame minus the number of rows in the cleaned-up data frame, which happens to be 2304.

#### 3.2: Filling missing values

The strategy used here is very simple. We already have the average number of steps, in a given 5-min interval,
over all days. So, if the number of steps entry is NA for a given time interval, just replace it with
the average over other days (where data is available).



#### 3.3: Clean-up the data frame

We just run over every entry in the "steps" column of the **original** data frame (*df1*).
If the value is present, there is nothing to do. If the value is a NA,
just replace it with its interval average (calculated in part 2, above); values are in 
the data frame *q2_a*. Treat the 288 rows as indexed by the time interval. The first
5-min slot (00:00 to 00:05) is at index 1, the second one is at (00:05 to 00:10), and the last one
is at slot (23:55 to 24:00). There is a straight-forward conversion of interval starting time
to index (of 5-min slot), using integer division.


I did think of a parallel implementation (as in subsetting operation). However, the current method is very 
easy (although linear), and works when the NA's are randomly scatttered in the original data frame.
The result is a cleaned-up data frame, *df3*.



```r
#df3 is the name of the cleaned-up data frame

df3 <- df1

for (k in 1:nrow(df3)) {
        if (is.na(df3$steps[k])) {
                # Get the interval of the NA entry
                i1 <- df3$interval[k]
                # convert it into an index
                h1 <- i1 %/% 100  #Index of hour
                m1 <- i1 %% 100 #Index of minute
                # There are twelve 5 minute intervals per hour
                inx <- h1*12 + m1%/%5 +1
                #Add 1 to start counting the index from 1
                #Otherwise, the 5-minute slot at 0:00 will have index 0
                #This is out of range in R array indexing
                df3$steps[k] <- q2_a$AvgSteps[inx]
        
                
        }
        
}
```


#### Part 3.4: Histogram on cleaned-up data

Logic is the same as in part 1; code works with just change of variable names.



```r
q3_a <- aggregate(x = df3$steps,
                  by = list(df3$date),
                  FUN = sum)


#Give meaningful column names for q3-a
names(q3_a) <- c("date", "totalSteps")

#make a histogram, as in part 1

hist(q3_a$totalSteps, 
     xlim = c(0,25000),
     ylim = c(0,30),
     nclass = 15,
     main = "CleanedUp Data: Total steps in  a day",
     xlab = "Number of steps per day ",
     ylab = "Count of days")
```

![](./PA1_template_files/figure-html/cleanHistogram-1.png) 

```r
#Also, write to a file

plot1 <- "./figure/hist_3.png"


png(plot1, width = 480, height = 480, units = "px")

hist(q3_a$totalSteps, 
     xlim = c(0,25000),
     ylim = c(0,30),
     nclass = 15,
     main = "CleanedUp Data: Total steps in  a day",
     xlab = "Number of steps per day ",
     ylab = "Count of days")



dev.off()
```

```
## pdf 
##   2
```

```r
#Get the mean, mediansteps per day
mn3 <- format(mean(q3_a$totalSteps), digits = 6)
md3 <- format(median(q3_a$totalSteps), digits = 6)
```

On the cleaned-up data, the new mean value of number of steps per day is 10766.2, and median is 10766.2. 
The previous values (on full rows only, from part 1), were 10766.2 and 10765 respectively. There is hardly any change.



### Part 4: Are there differences in activity patterns between weekdays and weekends?

*For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing
values for this part.    
1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating
whether a given date is a weekday or weekend day.  
2. Make a panel plot containing a time series plot (i.e. type = "l" ) of the 5-minute interval (x-axis) and
the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The
plot should look something like the following, which was creating using simulated data:    (snip)    
Your plot will look different from the one above because you will be using the activity monitor data. Note
that the above plot was made using the lattice system but you can make the same version of the plot using
any plotting system you choose.*

The key step is classify the days as weekdays (Monday through Friday), and weekends (Sat-Sun). 
Re-use the *df3* data frame from part 3, above.


The idea is to build a logical vector, with same number of rows as *df3*, which 
says if the day falls on a weekday or not.

1. Run through the $date column
2. Treat each entry as a date, using *as.Date()* function.
3. If it falls on a weekday, set indicator to TRUE; (it is FALSE otherwise).

Bind this vector to the *df3*, call it *df4*, and give an appropriate name to the new column.



```r
wkDays <- c("Mon", "Tue", "Wed", "Thu", "Fri")
wkEnd <- c("Sat", "Sun")

w1 <- (weekdays(as.Date(df3$date),TRUE) %in% wkDays)

df4 <- cbind(df3,w1)

names(df4)[4] <- "isWeekday"
```

Once we get this indicator, it is straight-forward to subset *df4* for weekdays and weekends, into, say,
*df4_a* and *df4_b*.  Use *aggregate* on these two data frames, and give meaningful names to the columns.


```r
#Subset for weekdays
df4_a <- subset(df4, df4$isWeekday) 

#Aggregate over days, for each time interval
q4_a <-  aggregate(x = df4_a$steps,
                          by = list(df4_a$interval),
                          FUN = mean)

#Set meaningful column names.
names(q4_a) <- c("intervalStart", "AvgSteps")


#Repeat for weekends, which is NOT(weekday)

df4_b <- subset(df4, !df4$isWeekday)



q4_b <- aggregate(x = df4_b$steps,
                  by = list(df4_b$interval),
                  FUN = mean)

names(q4_b) <- c("intervalStart", "AvgSteps")
```


Now, we are ready to plot. Use the *par* setting of 2-rows (mfpar()), 
and do a simple plot number steps by interval over day, for weekdays and weekends.


```r
# we want 2 rows, one column

par(mfrow=c(2,1))

#Top: weekday plot
plot(q4_a$intervalStart, q4_a$AvgSteps,
     main = "Weekday: Average number of steps over 5-min interval",
     xlab = "Interval", 
     ylab = "Avg steps in 5-min",
     type = "l")

#Bottom: weekend plot

plot(q4_b$intervalStart, q4_b$AvgSteps,
     main = "Weekend: Average number of steps over 5-min interval",
     xlab = "Interval", 
     ylab = "Avg steps in 5-min",
     type = "l")
```

![](./PA1_template_files/figure-html/unnamed-chunk-8-1.png) 

```r
#Do the same into a file as well

plot1 <- "./figure/line_4.png"


png(plot1, width = 480, height = 480, units = "px")


par(mfrow=c(2,1))

#Top: weekday plot
plot(q4_a$intervalStart, q4_a$AvgSteps,
     main = "Weekday: Average number of steps over 5-min interval",
     xlab = "Interval", 
     ylab = "Avg steps in 5-min",
     type = "l")

#Bottom: weekend plot

plot(q4_b$intervalStart, q4_b$AvgSteps,
     main = "Weekend: Average number of steps over 5-min interval",
     xlab = "Interval", 
     ylab = "Avg steps in 5-min",
     type = "l")

dev.off()
```

```
## pdf 
##   2
```


And, **yes**,  there is a difference in how much the person walks on weekdays versus weekends.
The person is much more active, through the day,  over weekends. 

