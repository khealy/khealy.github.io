# Bike Share Case Study
Kristen Healy
2024-04-16

- [Executive Summary](#executive-summary)
  - [Further Research](#further-research)
- [Background](#background)
- [Business Problem](#business-problem)
- [Key Questions](#key-questions)
- [Data Description](#data-description)
  - [What’s Included](#whats-included)
  - [Data Limitations](#data-limitations)
- [Data Import/Exploration](#data-importexploration)
- [Data Transformation](#data-transformation)
- [Data Cleaning](#data-cleaning)
- [Data Analysis](#data-analysis)
  - [Duration](#duration)
  - [Ride Counts](#ride-counts)
  - [Bike Types](#bike-types)

## Executive Summary

The focus of this case study is to answer the question: **How do annual
members and casual riders use their rental bikes from Divvy
differently?**, in order to determine marketing strategies to convert
casual riders into members.

To answer that question, I cleaned, transformed, and analyzed all of
Divvy’s historical trip data from 2023. I noted the following
differences between the two customer types:

- Casual riders take the rental bikes out for almost twice as long per
  ride as do members
- Members take more rides from Tuesday through Thursday, while casual
  riders take more rides on weekends
- While casual riders rent electric bikes slightly more often than
  classic bikes, they keep the classic bikes out for about three times
  longer. Contrast that to members, who rent the two bike types
  approximately the same amount, with a much smaller discrepancy between
  the rental durations.

**Since casual riders seem to like the electric bikes as much or more
than the classic bikes (based on the number of rentals) but don’t keep
them out as long, one promising tactic to get them to convert into
members may be to focus on the significant price break they’d get on
renting the electric bikes compared to what they are paying currently.**

### Further Research

If the company is interested in going further with this research, we
should pursue getting access to customer-level data. This would allow us
to determine whether there are any significant demographic differences
between the two groups, differences in rental frequencies, etc., that
might lead us to additional campaign opportunities.

## Background

Divvy is a Chicago-based bike share company with rental locations
throughout Chicago. They have 3 different ways to rent their bikes: a
single ride pass, a day pass, and an annual membership. The company’s
[historical trip
data](https://divvy-tripdata.s3.amazonaws.com/index.html) is available
to the public.

Although this is real, anonymized data, the rest of the background for
this analysis is based on some fictionalized assumptions:

- The company’s financial analysts have determined that the customers
  who purchase annual memberships are more profitable for the company
  than the single-ride and day-pass customers, who are grouped together
  in the data as *casual customers*.
- The director of marketing believes that annual memberships are the key
  to the company’s growth, and that there is a good opportunity to
  convert the company’s existing casual customers into members.

## Business Problem

How can the company convert its current casual customers into annual
members?

## Key Questions

To solve this problem, the team needs to answer three questions:

- **How do annual members and casual riders use their rental bikes
  differently?**
- Why would casual riders buy an annual memberships?
- How can the company use digital media to influence casual riders to
  become members?

This study will focus on answering the first question.

## Data Description

The data for this study was collected automatically from the company’s
stations when the bikes were checked in and out. It has been anonymized
to remove personally identifiable user information. I used the latest
full year of data–2023–for this study.

### What’s Included

The data includes:

- Ride IDs
- Bike type
- Trip start/end times
- Start/end station names, IDs, longitudes and latitudes
- Customer type (member or casual)

### Data Limitations

This data is missing a fair amount of location information, as noted
below.

There also is no customer-level data in this data set; it includes only
customer group data. Therefore, there’s no way to determine, for
example:

- How many times per week, month, year, etc. either casual riders or
  members rent a bike on average
- How many rides, on average, a customer takes before becoming a member
- The average length of membership
- Any demographic differences between members and casual riders

## Data Import/Exploration

The data was packaged by month into a zipped .csv file. After
downloading the 12 files–January 2023 through December 2023–from Divvy,
I did some preliminary exploration of each monthly files in Microsoft
Excel, including:

- Sorting the data by start and end times to ensure no times were
  missing.
- Reviewing the column names to ensure they were the same in all files.

Next, I imported the data into RStudio for further cleaning and
exploration.

``` r
#load necessary libraries
library(readr)
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ purrr     1.0.2
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.0     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

Then I combined the monthly data frames into a single data frame for
easier analysis.

When looking at the Excel files, I saw that quite a bit of location
information appeared to be missing. Once the data was imported into
RStudio, I checked to see how prevalent the problem was.

``` r
count(AllData_2023, start_station_id == "")
```

    ##   start_station_id == ""       n
    ## 1                  FALSE 4844029
    ## 2                   TRUE  875848

``` r
count(AllData_2023, start_station_name == "")
```

    ##   start_station_name == ""       n
    ## 1                    FALSE 4844161
    ## 2                     TRUE  875716

``` r
count(AllData_2023, end_station_id == "")
```

    ##   end_station_id == ""       n
    ## 1                FALSE 4790534
    ## 2                 TRUE  929343

``` r
count(AllData_2023, end_station_name == "")
```

    ##   end_station_name == ""       n
    ## 1                  FALSE 4790675
    ## 2                   TRUE  929202

Since the problem seems to be fairly widespread, any analysis by station
location doesn’t seem feasible.

## Data Transformation

The start and end times in the data set were initially of type *char*,
so I changed them into date-times so trip durations could be calculated.

``` r
#add columns and transform data types to the formats I need for analysis
#rename started_at and ended_at to start_time and end_time
#change start_time and end_time to DateTime data type
#calculate the trip duration and populate that to a trip_duration column
AllData_2023 <- AllData_2023 %>%
  rename(start_time = started_at) %>%
  mutate(start_time = as.POSIXct(start_time, format = "%Y-%m-%d %H:%M:%S")) %>%
  rename(end_time = ended_at) %>%
  mutate(end_time = as.POSIXct(end_time, format = "%Y-%m-%d %H:%M:%S")) %>%
  mutate(trip_duration = end_time - start_time)
```

Then I broke the dates into the components needed to aggregate the data
for further analysis.

``` r
#add columns and transform data types to the formats I need for analysis
#rename started_at and ended_at to start_time and end_time
#change start_time and end_time to DateTime data type
#calculate the trip duration and populate that to a trip_duration column
#create separate date columns (date, month, day, year, day of week, hour) for easier aggregation
AllData_2023 <- AllData_2023 %>%
  mutate(date = as.Date(start_time)) %>% #The default format is yyyy-mm-dd
  mutate(month = format(as.Date(date), "%m")) %>%
  mutate(day = format(as.Date(date), "%d")) %>%
  mutate(year = format(as.Date(date), "%Y")) %>%
  mutate(day_of_week = format(as.Date(date), "%A")) %>%
  mutate(hour = format(as.POSIXct(start_time), format = "%H"))
glimpse(AllData_2023)
```

    ## Rows: 5,719,877
    ## Columns: 20
    ## $ ride_id            <chr> "F96D5A74A3E41399", "13CB7EB698CEDB88", "BD88A2E670…
    ## $ rideable_type      <chr> "electric_bike", "classic_bike", "electric_bike", "…
    ## $ start_time         <dttm> 2023-01-21 20:05:42, 2023-01-10 15:37:36, 2023-01-…
    ## $ end_time           <dttm> 2023-01-21 20:16:33, 2023-01-10 15:46:05, 2023-01-…
    ## $ start_station_name <chr> "Lincoln Ave & Fullerton Ave", "Kimbark Ave & 53rd …
    ## $ start_station_id   <chr> "TA1309000058", "TA1309000037", "RP-005", "TA130900…
    ## $ end_station_name   <chr> "Hampden Ct & Diversey Ave", "Greenwood Ave & 47th …
    ## $ end_station_id     <chr> "202480.0", "TA1308000002", "599", "TA1308000002", …
    ## $ start_lat          <dbl> 41.92407, 41.79957, 42.00857, 41.79957, 41.79957, 4…
    ## $ start_lng          <dbl> -87.64628, -87.59475, -87.69048, -87.59475, -87.594…
    ## $ end_lat            <dbl> 41.93000, 41.80983, 42.03974, 41.80983, 41.80983, 4…
    ## $ end_lng            <dbl> -87.64000, -87.59938, -87.69941, -87.59938, -87.599…
    ## $ member_casual      <chr> "member", "member", "casual", "member", "member", "…
    ## $ trip_duration      <drtn> 651 secs, 509 secs, 794 secs, 526 secs, 919 secs, …
    ## $ date               <date> 2023-01-22, 2023-01-10, 2023-01-02, 2023-01-22, 20…
    ## $ month              <chr> "01", "01", "01", "01", "01", "01", "01", "01", "01…
    ## $ day                <chr> "22", "10", "02", "22", "12", "31", "16", "25", "26…
    ## $ year               <chr> "2023", "2023", "2023", "2023", "2023", "2023", "20…
    ## $ day_of_week        <chr> "Sunday", "Tuesday", "Monday", "Sunday", "Thursday"…
    ## $ hour               <chr> "20", "15", "07", "10", "13", "07", "21", "10", "20…

I noticed that the days of week were out of order, so I fixed that.

``` r
#fix day of week order
AllData_2023$day_of_week <- ordered(AllData_2023$day_of_week, levels=c("Sunday", "Monday",
                                                                       "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))
```

## Data Cleaning

[Divvy says](https://divvybikes.com/system-data) that its data has been
cleaned to remove trips taken by staff for service and inspection, and
also trips less than 60 seconds long since they assume those were “false
starts.” However, that turned out to be wrong.

``` r
count(AllData_2023, trip_duration < 60)
```

    ##   trip_duration < 60       n
    ## 1              FALSE 5570262
    ## 2               TRUE  149615

So, as my first data cleaning step, I removed those “false starts”.

``` r
  AllData_2023_v2 <- AllData_2023[!(AllData_2023$trip_duration<60),]
  count(AllData_2023_v2, trip_duration < 60)
```

    ##   trip_duration < 60       n
    ## 1              FALSE 5570262

Next I took at look at the long duration rides. Although Divvy includes
3 hours of ride time in its day pass program, [it charges by the minute
after that](https://divvybikes.com/pricing). Similarly, members get 45
minutes free per ride, but are charged by the minute after that. Since
costs for the rental would be well north of \$200 for casual riders
beyond 24 hours, I assume that any observations with durations beyond
that were the result of a stolen bike or a data recording error. Here
are those numbers:

``` r
count(AllData_2023_v2, trip_duration > 86400)
```

    ##   trip_duration > 86400       n
    ## 1                 FALSE 5563818
    ## 2                  TRUE    6444

I removed these long-duration rides from the data set.

``` r
  AllData_2023_v2 <- AllData_2023_v2[!(AllData_2023_v2$trip_duration>86400),]
  count(AllData_2023_v2, trip_duration >86400)
```

    ##   trip_duration > 86400       n
    ## 1                 FALSE 5563818

With that done, I took a look at the bike types (rideable_type).

``` r
#group by rideable_type and get a count for each type
AllData_2023_v2 %>%        
  group_by(rideable_type) %>%   
  tally()                  
```

    ## # A tibble: 3 × 2
    ##   rideable_type       n
    ##   <chr>           <int>
    ## 1 classic_bike  2649756
    ## 2 docked_bike     76114
    ## 3 electric_bike 2837948

The “docked_bikes” were a surprise to me; I had been led to believe that
there were only two bike types: classic and electric. After some quick
Googling to see what others had said about this aspect of the Divvy
data, I found that “docked_bike” was the old name for “classic_bike”. So
I converted “docked_bike” data to “classic_bike”.

``` r
AllData_2023_v2 <- AllData_2023_v2 %>%
  mutate(rideable_type = recode(rideable_type,"docked_bike" = "classic_bike"))
```

As my last data cleaning/validation step, I checked to make sure there
were no unexpected customer types (member_casual).

``` r
#group by member_casual and get a count for each type
AllData_2023_v2 %>%        
  group_by(member_casual) %>%   
  tally()                  
```

    ## # A tibble: 2 × 2
    ##   member_casual       n
    ##   <chr>           <int>
    ## 1 casual        2000473
    ## 2 member        3563345

The results were as I expected, so I was ready to move on to analysis.

## Data Analysis

After cleaning, the data set included 5563818 total observations.

### Duration

I started my analysis by doing some basic statistical comparisons of
duration differences between the casual riders and the members.

It’s clear from these numbers that, in general, the casual riders keep
the bikes out for quite a bit longer on average than the members.
However, because the median values show less of a difference, it may be
the case that our data still might include quite a few instances of
recording errors that are skewing the averages higher.

``` r
# trip duration mean comparison
aggregate(AllData_2023_v2$trip_duration ~ AllData_2023_v2$member_casual, FUN = mean)
```

    ##   AllData_2023_v2$member_casual AllData_2023_v2$trip_duration
    ## 1                        casual                1273.4485 secs
    ## 2                        member                 741.9872 secs

``` r
# trip duration median comparison
aggregate(AllData_2023_v2$trip_duration ~ AllData_2023_v2$member_casual, FUN = median)
```

    ##   AllData_2023_v2$member_casual AllData_2023_v2$trip_duration
    ## 1                        casual                      730 secs
    ## 2                        member                      525 secs

``` r
# trip duration max comparison
aggregate(AllData_2023_v2$trip_duration ~ AllData_2023_v2$member_casual, FUN = max)
```

    ##   AllData_2023_v2$member_casual AllData_2023_v2$trip_duration
    ## 1                        casual                    86343 secs
    ## 2                        member                    86392 secs

``` r
# trip duration min comparison
aggregate(AllData_2023_v2$trip_duration ~ AllData_2023_v2$member_casual, FUN = min)
```

    ##   AllData_2023_v2$member_casual AllData_2023_v2$trip_duration
    ## 1                        casual                       60 secs
    ## 2                        member                       60 secs

#### Duration Differences by Month

The customer groups show a similar pattern over the course of a year,
with ride duration peaking in July for both groups. However, the members
show less of a seasonal change in their ride durations.

``` r
# Group and summarize data, then create a bar chart showing duration by customer type (by month)
AllData_2023_v2 %>%
  mutate(ridemonth = month(start_time, label = TRUE)) %>%
  group_by(member_casual, ridemonth) %>%
  summarise(number_of_rides = n()
            ,average_duration = mean(trip_duration)/60) %>%
  arrange(member_casual, ridemonth) %>%
  ggplot(aes(x = ridemonth, y = average_duration, fill= member_casual)) +
  geom_col(position = "dodge")+
  labs(title = "Ride Duration by Customer Type (Month)", x=NULL, y="Average Duration (minutes)", fill = "Customer Type")
```

![](figure-gfm/duration%20by%20month-1.png)<!-- -->

#### Duration Differences by Day of Week

Unsurprisingly, both customer groups take longer rides on the weekends
than during the week. Again, there’s not as much of a change for the
members as for the casual riders, but because the daily weekday numbers
are very similar for the members, we could conclude that members may be
using the bikes for their commutes to work.

``` r
# Group and summarize data, then create a bar chart showing duration by customer type (by day of week)
AllData_2023_v2 %>%
  mutate(weekday = wday(start_time, label = TRUE)) %>%
  group_by(member_casual, weekday) %>%
  summarise(number_of_rides = n()
            ,average_duration = mean(trip_duration)/60) %>%
  arrange(member_casual, weekday) %>%
  ggplot(aes(x = weekday, y = average_duration, fill= member_casual)) +
  geom_col(position = "dodge")+
  labs(title = "Ride Duration by Customer Type (Day of Week)", x=NULL, y="Average Duration (minutes)", fill = "Customer Type")
```

![](figure-gfm/duration%20day%20of%20week-1.png)<!-- -->

#### Duration Differences by Hour of Day

The graphs here show that casual riders tend to ride quite a bit longer
on average between 9 a.m. and 2 p.m. Member durations don’t show as much
variation.

``` r
# Group and summarize data, then create a bar chart showing duration by customer type (by day of week)
AllData_2023_v2 %>%
  group_by(member_casual, hour) %>%
  summarise(number_of_rides = n()
            ,average_duration = mean(trip_duration)/60) %>%
  arrange(member_casual, hour) %>%
  ggplot(aes(x = hour, y = average_duration, fill= member_casual)) +
  geom_col(position = "dodge")+
  labs(title = "Ride Duration by Customer Type (Hour of Day)", x=NULL, y="Average Duration (minutes)", fill = "Customer Type")
```

![](figure-gfm/number%20of%20rides%20by%20hour%20of%20day-1.png)<!-- -->

### Ride Counts

I also compared the number of rides taken by member compared to casual
riders. Here, the members come out on top, with quite a few more rides
taken on average by members than casual riders.

#### Ride Count Differences by Month

The ride counts by month show a similar seasonal pattern to the ride
durations, although the peak here is in July for casual riders and
August for members.

``` r
options(scipen = 999)
# Group and summarize data, then create a bar chart showing number of rides by customer type (by month)
AllData_2023_v2 %>%
  mutate(ridemonth = month(start_time, label = TRUE)) %>%
  group_by(member_casual, ridemonth) %>%
  summarise(number_of_rides = n()
            ,average_duration = mean(trip_duration)) %>%
  arrange(member_casual, ridemonth) %>%
  ggplot(aes(x = ridemonth, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge") +
  labs(title = "Ride Count by Customer Type (Month)", x=NULL, y="Number of Rides", fill = "Customer Type")
```

![](figure-gfm/ride%20counts%20by%20month-1.png)<!-- -->

#### Ride Count Differences by Day of Week

There’s an interesting disparity with the day of week ride patterns: the
members show peak numbers Tuesday through Thursday, while Saturday is
the most popular day for casual riders.

``` r
options(scipen = 999)
# analyze ridership data by customer type and day of week
AllData_2023_v2 %>%
  mutate(weekday = wday(start_time, label = TRUE)) %>% #creates weekday field using wday()
  group_by(member_casual, weekday) %>% #groups by user type and weekday
  summarise(number_of_rides = n() #calculates the number of rides and average duration
            ,average_duration = mean(trip_duration)) %>% # calculates the average duration
  arrange(member_casual, weekday) %>% # sorts
  ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge") +
  labs(title = "Ride Count by Customer Type (Day of Week)", x=NULL, y="Number of Rides", fill = "Customer Type")
```

![](figure-gfm/ride%20counts%20by%20day%20of%20week-1.png)<!-- -->

#### Ride Count Differences by Hour of Day

Both groups of riders show a clear peak in the number of rides between 4
and 6 p.m., perhaps coinciding with an end-of-day commute home from
work.

``` r
options(scipen = 999)
# analyze ridership data by customer type and day of week
AllData_2023_v2 %>%
  group_by(member_casual, hour) %>% 
  summarise(number_of_rides = n() 
            ,average_duration = mean(trip_duration)) %>% 
  arrange(member_casual, hour) %>%
  ggplot(aes(x = hour, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge") +
  labs(title = "Ride Count by Customer Type (Hour of Day)", x=NULL, y="Number of Rides", fill = "Customer Type")
```

![](figure-gfm/ride%20counts%20by%20hour%20of%20day-1.png)<!-- -->

### Bike Types

Neither customer group shows a strong preference for renting classic
bikes versus electric bikes. And while there isn’t a dramatic difference
between ride duration between the two bike types for members, the casual
riders spend much more time per ride on the classic bikes. One likely
explanation for this is the [price
differential](https://divvybikes.com/pricing) between the classic bikes
at electric bikes; for casual riders, the difference amounts to:

- \$18.10 for 3 hours on the classic bike, plus \$.18 per minute after
  that
- \$.44 per minute on the electric bike, which is \$79.20 for the first
  hour

``` r
AllData_2023_v2 %>%
  group_by(member_casual, rideable_type) %>% 
  summarise(number_of_rides = n(),average_duration = mean(trip_duration)) %>% 
  arrange(member_casual, rideable_type) 
```

    ## # A tibble: 4 × 4
    ## # Groups:   member_casual [2]
    ##   member_casual rideable_type number_of_rides average_duration
    ##   <chr>         <chr>                   <int> <drtn>          
    ## 1 casual        classic_bike           936852 1712.3140 secs  
    ## 2 casual        electric_bike         1063621  886.8898 secs  
    ## 3 member        classic_bike          1789018  790.9688 secs  
    ## 4 member        electric_bike         1774327  692.6000 secs

``` r
options(scipen = 999)
# Plot rides by bike type
AllData_2023_v2 %>% 
  ggplot(aes(x = rideable_type)) + 
  geom_bar() + 
  geom_bar(aes(fill = member_casual)) + 
  labs(title = "Number of Rides by Customer and Bike Type", x=NULL, y="Number of Rides", fill = "Customer Type")
```

![](figure-gfm/customer%20type%20and%20bike%20type-1.png)<!-- -->

``` r
AllData_2023_v2 %>%
  group_by(member_casual, rideable_type) %>%
  summarise(number_of_rides = n()
            ,average_duration = mean(trip_duration)/60) %>%
  arrange(member_casual, rideable_type) %>%
  ggplot(aes(x = rideable_type, y = average_duration, fill = member_casual)) +
  geom_col(position = "dodge") +
  labs(title = "Ride Duration Customer and Bike Types", x=NULL, y="Ride Duration (minutes)", fill = "Customer Type")
```

![](figure-gfm/customer%20type%20and%20bike%20type%20(duration)-1.png)<!-- -->
