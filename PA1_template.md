# Reproducible Research: Peer Assessment 1

[github.com/TribeTect, July 2015]

## 1. Loading and preprocessing the data

```r
activity <- read.csv(unz("activity.zip", "activity.csv"))
activity[["date"]] <- as.Date(activity[["date"]], "%Y-%m-%d")
```

## 2. What is mean total number of steps taken per day?

```r
activity2 <- activity[complete.cases(activity),] # ignoring NA observations for this part of the assignment
daily_steps <- tapply(X = activity2$steps,INDEX = activity2$date, FUN = "sum")
cat("Mean daily steps (ignoring NAs): ", mean(daily_steps))
```

```
## Mean daily steps (ignoring NAs):  10766.19
```

(showing first  daily totals)

```r
cat("2.1 Total number of steps per day: \n", head(daily_steps, 10), " ... \n", tail(daily_steps, 10), " (first and last 10 records)")
```

```
## 2.1 Total number of steps per day: 
##  126 11352 12116 13294 15420 11015 12811 9900 10304 17382  ... 
##  4472 12787 20427 21194 14478 11834 11162 13646 10183 7047  (first and last 10 records)
```

### 2.2 Histogram of steps per day:

```r
hist(daily_steps, xlab = "Daily Steps", col = "red")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

```r
mean_daily_steps <- mean(daily_steps)
median_daily_steps <- median(daily_steps)
cat("Mean Daily Steps: ", mean_daily_steps, "\n")
```

```
## Mean Daily Steps:  10766.19
```

```r
cat("Median Daily Steps: ", median_daily_steps, "\n")
```

```
## Median Daily Steps:  10765
```
## 3. What is the average daily activity pattern?

### 3.1 Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
### 3.2 Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
interval_steps <- tapply(X = activity2$steps, INDEX = activity2$interval, FUN = "mean")

max_mean_steps <- as.data.frame(interval_steps[interval_steps == max(interval_steps)])
max_mean_x = row.names(max_mean_steps) 
max_mean_y = max_mean_steps[[1]]

plot(x = row.names(interval_steps), y = interval_steps, xlab = "Interval", ylab = "Mean steps", type = "l")
points(x = max_mean_x, y = max_mean_y, pch = 19, col = "red")
abline(v = max_mean_x, col = "red")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

```r
cat("The interval with highest mean: ", max_mean_x)
```

```
## The interval with highest mean:  835
```

```r
cat("\n Highest mean: ", max_mean_y)
```

```
## 
##  Highest mean:  206.1698
```

## 4. Imputing missing values
### 4.1 Report count of observations with NAs

```r
cat("4.1 Number of cases with NA values: ", sum(is.na(activity)))
```

```
## 4.1 Number of cases with NA values:  2304
```

### 4.2 Devise a strategy to replace missing values 
#### Missing values will be replaced with corresponding interval means, rounded off 

### 4.3 Create a new dataset with NA values replaced


```r
activity_sub <- activity #clone the dataset for replacing NAs

for(i in 1:nrow(activity_sub)) {
  if(is.na(activity_sub[i,1])) {
    interval_i <- as.character(activity_sub[i,3]) #get the interval
    mean_step_i <- interval_steps[interval_i] #get mean steps for interval
    activity_sub[i,1] <- round(as.numeric(mean_step_i)) #replace NA value 
      
  }
}
```
### 4.4 Histogram of daily steps, with new dataset, mean and median

```r
#re calculate daily steps
daily_steps_sub <- tapply(X = activity2$steps,INDEX = activity2$date, FUN = "sum")
hist(daily_steps_sub, xlab = "Total Daily Steps", col = "blue")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png) 

```r
mean_daily_steps_sub <- mean(daily_steps_sub)
median_daily_steps_sub <- median(daily_steps_sub)
cat("New mean: ", mean_daily_steps_sub, " Previous mean: ", mean_daily_steps, "\n" )
```

```
## New mean:  10766.19  Previous mean:  10766.19
```

```r
cat("New median: ", median_daily_steps_sub, " Previous mean: ", median_daily_steps, "\n")
```

```
## New median:  10765  Previous mean:  10765
```
#### QUESTION: Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

### ANSWER: Same mean and median shows no impact of imputing missing data on those measures

---

## 5. Are there differences in activity patterns between weekdays and weekends?

```r
require(dplyr, quietly = TRUE)
```

```
## 
## Attaching package: 'dplyr'
## 
## The following object is masked from 'package:stats':
## 
##     filter
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
activity_sub <- mutate(activity_sub, day_type = "Weekday")

for(i in 1:nrow(activity_sub)){

    weekend_i <- (weekdays(activity_sub[i,2]) == "Saturday") | (weekdays(activity_sub[i,2]) == "Sunday")
    
    if(weekend_i) {activity_sub[i,4] = "Weekend"}
    
}

table(activity_sub$day_type)
```

```
## 
## Weekday Weekend 
##   12960    4608
```

```r
require(lattice, quietly = TRUE)
xyplot(log(steps) ~ interval | day_type, data = activity_sub, layout = c(1,2), type = "l")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 
