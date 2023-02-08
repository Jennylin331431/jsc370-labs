Lab 05 - Data Wrangling
================

# Learning goals

- Use the `merge()` function to join two datasets.
- Deal with missings and impute data.
- Identify relevant observations using `quantile()`.
- Practice your GitHub skills.

# Lab description

For this lab we will be dealing with the meteorological dataset `met`.
In this case, we will use `data.table` to answer some questions
regarding the `met` dataset, while at the same time practice your
Git+GitHub skills for this project.

This markdown document should be rendered using `github_document`
document.

# Part 1: Setup a Git project and the GitHub repository

1.  Go to wherever you are planning to store the data on your computer,
    and create a folder for this project

2.  In that folder, save [this
    template](https://github.com/JSC370/jsc370-2023/blob/main/labs/lab05/lab05-wrangling-gam.Rmd)
    as “README.Rmd”. This will be the markdown file where all the magic
    will happen.

3.  Go to your GitHub account and create a new repository of the same
    name that your local folder has, e.g., “JSC370-labs”.

4.  Initialize the Git project, add the “README.Rmd” file, and make your
    first commit.

5.  Add the repo you just created on GitHub.com to the list of remotes,
    and push your commit to origin while setting the upstream.

Most of the steps can be done using command line:

``` sh
# Step 1
cd ~/Documents
mkdir JSC370-labs
cd JSC370-labs
# Step 2
wget https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab05/lab05-wrangling-gam.Rmd
mv lab05-wrangling-gam.Rmd README.Rmd
# if wget is not available,
curl https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab05/lab05-wrangling-gam.Rmd --output README.Rmd
# Step 3
# Happens on github
# Step 4
git init
git add README.Rmd
git commit -m "First commit"
# Step 5
git remote add origin git@github.com:[username]/JSC370-labs
git push -u origin master
```

You can also complete the steps in R (replace with your paths/username
when needed)

``` r
# Step 1
setwd("~/Documents")
dir.create("JSC370-labs")
setwd("JSC370-labs")
# Step 2
download.file(
  "https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab05/lab05-wrangling-gam.Rmd",
  destfile = "README.Rmd"
  )
# Step 3: Happens on Github
# Step 4
system("git init && git add README.Rmd")
system('git commit -m "First commit"')
# Step 5
system("git remote add origin git@github.com:[username]/JSC370-labs")
system("git push -u origin master")
```

Once you are done setting up the project, you can now start working with
the MET data.

## Setup in R

1.  Load the `data.table` (and the `dtplyr` and `dplyr` packages if you
    plan to work with those).

``` r
library('data.table')
library('dplyr')
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:data.table':
    ## 
    ##     between, first, last

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library('dtplyr')
```

2.  Load the met data from
    <https://github.com/JSC370/jsc370-2023/blob/main/labs/lab03/met_all.gz>
    or (Use
    <https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab03/met_all.gz>
    to download programmatically), and also the station data. For the
    latter, you can use the code we used during lecture to pre-process
    the stations data:

``` r
#read met
met <- fread("https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab03/met_all.gz")
head(met)
```

    ##    USAFID  WBAN year month day hour min  lat      lon elev wind.dir wind.dir.qc
    ## 1: 690150 93121 2019     8   1    0  56 34.3 -116.166  696      220           5
    ## 2: 690150 93121 2019     8   1    1  56 34.3 -116.166  696      230           5
    ## 3: 690150 93121 2019     8   1    2  56 34.3 -116.166  696      230           5
    ## 4: 690150 93121 2019     8   1    3  56 34.3 -116.166  696      210           5
    ## 5: 690150 93121 2019     8   1    4  56 34.3 -116.166  696      120           5
    ## 6: 690150 93121 2019     8   1    5  56 34.3 -116.166  696       NA           9
    ##    wind.type.code wind.sp wind.sp.qc ceiling.ht ceiling.ht.qc ceiling.ht.method
    ## 1:              N     5.7          5      22000             5                 9
    ## 2:              N     8.2          5      22000             5                 9
    ## 3:              N     6.7          5      22000             5                 9
    ## 4:              N     5.1          5      22000             5                 9
    ## 5:              N     2.1          5      22000             5                 9
    ## 6:              C     0.0          5      22000             5                 9
    ##    sky.cond vis.dist vis.dist.qc vis.var vis.var.qc temp temp.qc dew.point
    ## 1:        N    16093           5       N          5 37.2       5      10.6
    ## 2:        N    16093           5       N          5 35.6       5      10.6
    ## 3:        N    16093           5       N          5 34.4       5       7.2
    ## 4:        N    16093           5       N          5 33.3       5       5.0
    ## 5:        N    16093           5       N          5 32.8       5       5.0
    ## 6:        N    16093           5       N          5 31.1       5       5.6
    ##    dew.point.qc atm.press atm.press.qc       rh
    ## 1:            5    1009.9            5 19.88127
    ## 2:            5    1010.3            5 21.76098
    ## 3:            5    1010.6            5 18.48212
    ## 4:            5    1011.6            5 16.88862
    ## 5:            5    1012.7            5 17.38410
    ## 6:            5    1012.7            5 20.01540

``` r
# Download the data
stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
stations[, USAF := as.integer(USAF)]
```

    ## Warning in eval(jsub, SDenv, parent.frame()): NAs introduced by coercion

``` r
# Dealing with NAs and 999999
stations[, USAF   := fifelse(USAF == 999999, NA_integer_, USAF)]
stations[, CTRY   := fifelse(CTRY == "", NA_character_, CTRY)]
stations[, STATE  := fifelse(STATE == "", NA_character_, STATE)]
# Selecting the three relevant columns, and keeping unique records
stations <- unique(stations[, list(USAF, CTRY, STATE)])
# Dropping NAs
stations <- stations[!is.na(USAF)]
# Removing duplicates
stations[, n := 1:.N, by = .(USAF)]
stations <- stations[n == 1,][, n := NULL]
```

3.  Merge the data as we did during the lecture.

``` r
data <- merge(
# Data
 x = met, 
 y = stations, 
# List of variables to match
 by.x = "USAFID",
 by.y = "USAF", 
# Which obs to keep?
 all.x = TRUE, 
 all.y = FALSE
 ) 
```

``` r
met_lz <- lazy_dt(data, immutable = FALSE)
met_lz
```

    ## Source: local data table [2,377,343 x 32]
    ## Call:   `_DT1`
    ## 
    ##   USAFID  WBAN  year month   day  hour   min   lat   lon  elev wind.dir wind.d…¹
    ##    <int> <int> <int> <int> <int> <int> <int> <dbl> <dbl> <int>    <int> <chr>   
    ## 1 690150 93121  2019     8     1     0    56  34.3 -116.   696      220 5       
    ## 2 690150 93121  2019     8     1     1    56  34.3 -116.   696      230 5       
    ## 3 690150 93121  2019     8     1     2    56  34.3 -116.   696      230 5       
    ## 4 690150 93121  2019     8     1     3    56  34.3 -116.   696      210 5       
    ## 5 690150 93121  2019     8     1     4    56  34.3 -116.   696      120 5       
    ## 6 690150 93121  2019     8     1     5    56  34.3 -116.   696       NA 9       
    ## # … with 2,377,337 more rows, 20 more variables: wind.type.code <chr>,
    ## #   wind.sp <dbl>, wind.sp.qc <chr>, ceiling.ht <int>, ceiling.ht.qc <int>,
    ## #   ceiling.ht.method <chr>, sky.cond <chr>, vis.dist <int>, vis.dist.qc <chr>,
    ## #   vis.var <chr>, vis.var.qc <chr>, temp <dbl>, temp.qc <chr>,
    ## #   dew.point <dbl>, dew.point.qc <chr>, atm.press <dbl>, atm.press.qc <int>,
    ## #   rh <dbl>, CTRY <chr>, STATE <chr>, and abbreviated variable name
    ## #   ¹​wind.dir.qc
    ## 
    ## # Use as.data.table()/as.data.frame()/as_tibble() to access results

## Question 1: Representative station for the US

Across all weather stations, what is the median station in terms of
temperature, wind speed, and atmospheric pressure? Look for the three
weather stations that best represent continental US using the
`quantile()` function. Do these three coincide?

``` r
met_avg_lz <- met_lz %>%  group_by(USAFID)%>% summarise(
  across (c(temp, wind.sp, atm.press), function(x) mean(x, na.rm=TRUE))) 
met_avg_lz
```

    ## Source: local data table [1,595 x 4]
    ## Call:   `_DT1`[, .(temp = (function (x) 
    ## mean(x, na.rm = TRUE))(temp), wind.sp = (function (x) 
    ## mean(x, na.rm = TRUE))(wind.sp), atm.press = (function (x) 
    ## mean(x, na.rm = TRUE))(atm.press)), keyby = .(USAFID)]
    ## 
    ##   USAFID  temp wind.sp atm.press
    ##    <int> <dbl>   <dbl>     <dbl>
    ## 1 690150  33.2    3.48     1010.
    ## 2 720110  31.2    2.14      NaN 
    ## 3 720113  23.3    2.47      NaN 
    ## 4 720120  27.0    2.50      NaN 
    ## 5 720137  21.9    1.98      NaN 
    ## 6 720151  27.6    3.00      NaN 
    ## # … with 1,589 more rows
    ## 
    ## # Use as.data.table()/as.data.frame()/as_tibble() to access results

``` r
#median 
met_med_lz <- met_avg_lz %>% summarise(
  across (c(temp, wind.sp, atm.press), function(x) quantile(x, probs = 0.5, na.rm=TRUE)))
met_med_lz
```

    ## Source: local data table [1 x 3]
    ## Call:   `_DT1`[, .(temp = (function (x) 
    ## mean(x, na.rm = TRUE))(temp), wind.sp = (function (x) 
    ## mean(x, na.rm = TRUE))(wind.sp), atm.press = (function (x) 
    ## mean(x, na.rm = TRUE))(atm.press)), keyby = .(USAFID)][, .(temp = (function (x) 
    ## quantile(x, probs = 0.5, na.rm = TRUE))(temp), wind.sp = (function (x) 
    ## quantile(x, probs = 0.5, na.rm = TRUE))(wind.sp), atm.press = (function (x) 
    ## quantile(x, probs = 0.5, na.rm = TRUE))(atm.press))]
    ## 
    ##    temp wind.sp atm.press
    ##   <dbl>   <dbl>     <dbl>
    ## 1  23.7    2.46     1015.
    ## 
    ## # Use as.data.table()/as.data.frame()/as_tibble() to access results

``` r
med_temp <- met_avg_lz %>% mutate(
  temp_diff = abs(temp - (met_med_lz %>% pull(temp)))) %>% arrange((temp_diff)) %>% slice(1) %>% pull(USAFID)


med_wind <- met_avg_lz %>% mutate(
  wind_diff = abs(wind.sp - (met_med_lz %>% pull(wind.sp)))) %>% arrange((wind_diff)) %>% slice(1) %>% pull(USAFID)


med_pres <- met_avg_lz %>% mutate(
  pres_diff = abs(atm.press - (met_med_lz %>% pull(atm.press)))) %>% arrange((pres_diff)) %>% slice(1) %>% pull(USAFID)


med_temp
```

    ## [1] 720458

``` r
med_wind
```

    ## [1] 720929

``` r
med_pres
```

    ## [1] 722238

**Response** The three staions are not identical. For temperature, the
median station is 720458; for wind speed, the median station is 720929,
and for atmosphere pressure, the median station is 722238. They do not
coincide.

Knit the document, commit your changes, and save it on GitHub. Don’t
forget to add `README.md` to the tree, the first time you render it.

## Question 2: Representative station per state

Just like the previous question, you are asked to identify what is the
most representative, the median, station per state. This time, instead
of looking at one variable at a time, look at the euclidean distance. If
multiple stations show in the median, select the one located at the
lowest latitude.

``` r
met_avg_lz <- met_lz %>%  group_by(STATE)%>% summarise(
  across (c(temp, wind.sp, atm.press), function(x) mean(x, na.rm=TRUE))) %>% as.data.frame()
met_avg_lz
```

    ##    STATE     temp  wind.sp atm.press
    ## 1     AL 26.19799 1.566381  1016.148
    ## 2     AR 26.20697 1.836963  1014.551
    ## 3     AZ 28.80596 2.984547  1010.771
    ## 4     CA 22.36199 2.614120  1012.640
    ## 5     CO 19.54725 3.075255  1013.725
    ## 6     CT 22.25797 2.194895  1015.036
    ## 7     DE 24.58116 2.762484  1015.058
    ## 8     FL 27.53747 2.501976  1015.242
    ## 9     GA 26.53857 1.509138  1015.168
    ## 10    IA 21.27773 2.568719  1015.210
    ## 11    ID 20.69554 2.380664  1013.222
    ## 12    IL 22.41005 2.171944  1014.215
    ## 13    IN 21.76562 2.260783  1015.035
    ## 14    KS 24.25538 3.729569  1013.360
    ## 15    KY 23.87157 1.788035  1015.301
    ## 16    LA 27.97441 1.724747  1014.591
    ## 17    MA 21.47484 2.699620  1014.755
    ## 18    MD 24.55218 1.855838  1015.285
    ## 19    ME 18.79902 2.133954  1014.362
    ## 20    MI 20.19981 2.215030  1014.853
    ## 21    MN 19.35621 2.427195  1014.965
    ## 22    MO 23.87039 2.379529  1014.616
    ## 23    MS 26.41324 1.477365  1014.995
    ## 24    MT 18.16680 3.426447  1014.095
    ## 25    NC 24.37801 1.634159  1015.612
    ## 26    ND 18.37173 3.869228       NaN
    ## 27    NE 22.10408 3.138987  1014.254
    ## 28    NH 18.24715 2.192529  1014.635
    ## 29    NJ 23.34497 1.884296  1014.791
    ## 30    NM 24.47771 3.477918  1011.992
    ## 31    NV 26.04296 2.999347  1011.621
    ## 32    NY 20.12499 2.320394  1015.015
    ## 33    OH 21.83450 2.423660  1015.315
    ## 34    OK 27.40891 3.231213  1012.610
    ## 35    OR 18.68974 2.094975  1015.112
    ## 36    PA 21.61523 1.812037  1015.614
    ## 37    RI 22.32827 2.728330  1013.606
    ## 38    SC 25.87440 1.696787  1015.335
    ## 39    SD 20.03650 3.774762  1014.389
    ## 40    TN 24.78164 1.519564  1014.448
    ## 41    TX 29.59743 3.190673  1012.285
    ## 42    UT 25.82056 4.402966  1012.005
    ## 43    VA 23.88116 1.696749  1015.235
    ## 44    VT 18.76708 1.671583  1014.645
    ## 45    WA 19.11089 1.236213       NaN
    ## 46    WI 18.57907 2.038866  1014.940
    ## 47    WV 21.74214 1.655494  1015.725
    ## 48    WY 18.60170 3.833987  1013.939

Knit the doc and save it on GitHub.

## Question 3: In the middle?

For each state, identify what is the station that is closest to the
mid-point of the state. Combining these with the stations you identified
in the previous question, use `leaflet()` to visualize all \~100 points
in the same figure, applying different colors for those identified in
this question.

Knit the doc and save it on GitHub.

## Question 4: Means of means

Using the `quantile()` function, generate a summary table that shows the
number of states included, average temperature, wind-speed, and
atmospheric pressure by the variable “average temperature level,” which
you’ll need to create.

Start by computing the states’ average temperature. Use that measurement
to classify them according to the following criteria:

- low: temp \< 20
- Mid: temp \>= 20 and temp \< 25
- High: temp \>= 25

Once you are done with that, you can compute the following:

- Number of entries (records),
- Number of NA entries,
- Number of stations,
- Number of states included, and
- Mean temperature, wind-speed, and atmospheric pressure.

All by the levels described before.

Knit the document, commit your changes, and push them to GitHub.

## Question 5: Advanced Regression

Let’s practice running regression models with smooth functions on X. We
need the `mgcv` package and `gam()` function to do this.

- using your data with the median values per station, examine the
  association between median temperature (y) and median wind speed (x).
  Create a scatterplot of the two variables using ggplot2. Add both a
  linear regression line and a smooth line.

- fit both a linear model and a spline model (use `gam()` with a cubic
  regression spline on wind speed). Summarize and plot the results from
  the models and interpret which model is the best fit and why.
