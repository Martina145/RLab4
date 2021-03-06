Prediction of flight delay
========================================================

# 1

The data sets `weather` and `flights` are loaded along with the packages `nycflights`, `dplyr`, `caret` and `RLab4` according to:

```r
library(nycflights13)
data(flights)
data(weather)
require(dplyr)
```

```
## Loading required package: dplyr
## 
## Attaching package: 'dplyr'
## 
## The following objects are masked from 'package:stats':
## 
##     filter, lag
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
library(RLab4)

flights<- flights %>% select(year, month, day, hour, origin, dep_delay, arr_delay, distance) %>% na.omit()
weather<-weather %>% select(-dewp, -humid, -pressure) %>% na.omit()
wfComb<-flights %>% inner_join(weather, by=c("origin", "year", "month", "day", "hour")) %>% mutate("prec_wind"=precip*wind_speed, "dir_wind"=wind_dir*wind_speed) 
```


# 2
The datasets are then combined, from `flights` the variable `dep_delay` is saved along with necessary merge variables. From `weather`, all variables except `dewp`, `humid` and `pressure` are used. Two interaction effects are 
included - `prec_wind`, which create effects for preciption and wind speed, and `dir_wind`, creating interation between wind speed and wind direction.


```r
flights<- flights %>% select(year, month, day, hour, origin, dep_delay, arr_delay, distance) %>% na.omit()
weather<-weather %>% select(-dewp, -humid, -pressure) %>% na.omit()
```

```
## Error in eval(expr, envir, enclos): object 'dewp' not found
```

```r
wfComb<-flights %>% inner_join(weather, by=c("origin", "year", "month", "day", "hour")) %>% mutate("prec_wind"=precip*wind_speed, "dir_wind"=wind_dir*wind_speed) 
```


# 3
Data partitions are created with

```r
trainTestIndex <- createDataPartition(wfComb$dep_delay, p = .85, list = FALSE,times = 1)

Valid  <- wfComb[-trainTestIndex,]
Left <- wfComb[trainTestIndex,]

trainIndex <- createDataPartition(Left$dep_delay, p = 80/85, list = FALSE,times = 1)

Test <- Left[-trainIndex,]
Train <- Left[ trainIndex,] 
```

# 4
Three test models are trained, using `lambda=0.01, 0.001, 0.0001`. Below the code for the best model, with `lambda=0.0001`,is presented. Training to find the best value for lambda confirmed 0.0001 to be beneficial.


```r
model7 <- ridgereg(dep_delay ~ distance + temp + wind_dir + wind_speed
                   + precip + visib + prec_wind + dir_wind,
                   data=Train,0.0001)
```

```
## Error in eval(expr, envir, enclos): could not find function "ridgereg"
```


The parameters of the applied model is used to present the RMSE for the validation set:

```r
v3<-model7$predict(Valid)
```

```
## Error in eval(expr, envir, enclos): object 'model7' not found
```

```r
v3Error<-v3-Valid$dep_delay
```

```
## Error in eval(expr, envir, enclos): object 'v3' not found
```

```r
RMSE3<-sqrt(mean(v3Error^2))
```

```
## Error in mean(v3Error^2): object 'v3Error' not found
```

```r
RMSE3
```

```
## Error in eval(expr, envir, enclos): object 'RMSE3' not found
```

# 5
Finally, the RMSE for the Test data is computed.

```r
vTest<-model7$predict(Test)
```

```
## Error in eval(expr, envir, enclos): object 'model7' not found
```

```r
vTestError<-vTest-Test$dep_delay
```

```
## Error in eval(expr, envir, enclos): object 'vTest' not found
```

```r
RMSETest<-sqrt(mean(vTestError^2))
```

```
## Error in mean(vTestError^2): object 'vTestError' not found
```

```r
RMSETest
```

```
## Error in eval(expr, envir, enclos): object 'RMSETest' not found
```
which is not far from the RMSE obtained with the training and validation sets, concluding the model seems reliable enough to generalize.

