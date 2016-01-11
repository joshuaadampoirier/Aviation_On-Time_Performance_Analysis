---
title: "Aviation On-Time Performance Analysis"
author: "Joshua Poirier"
date: "Tuesday, January 12, 2016"
output: html_document
---

## Abstract

This study is inspired by the web application and study by Ritchie King and Nate Silver at fivethirtyeight.com titled *Which Flight Will Get You There Fastest?*.  The web application can be found [here](http://projects.fivethirtyeight.com/flights/).  The accompanying [documentation](http://fivethirtyeight.com/features/how-we-found-the-fastest-flights/) describes what they mean by "fastest" flights: 

> Airline A says it will fly you from Seattle to Portland, Oregon, in 45 minutes, but actually takes 60 minutes.  Airline B says it will fly the same route in 75 minutes, and actually takes 70 minutes.  Which flight would you rather take?  That seems easy. Airline A!

I extend this study by considering two cases:  

* **Case 1:** Direct Flight - The consumer wants the fastest flight as defined in the aforementioned study
* **Case 2:** Connecting Flight - The consumer wants an on-time flight in order to avoid missing a tight connection 

I hypothesize that the 3 best and worst airlines for the two cases are different.  To show this, a multivariate regression model will be built for each case to account for confounding variables including **month**, **day of week**, **origin airport**, **destination airport**, and **time of day** in addition to the **airline**.



## Introduction

This study utilizes aviation on-time performance data provided by the United States Department of Transportation [here](http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time).  **Case 2** will use the *ARR_DELAY* field to compare against scheduled arrivals.  **Case 1** will use the difference between the gate-to-gate flight time (*ACTUAL_ELAPSED_TIME* field) and the **target time**.  I calculate target time using the simplified formula shown by Ritchie King and Nate Silver.  

$target time = 0.117 * distance + 0.517 * (lon origin - lon dest) + 43.2$  

This formula produces an estimated travel time in minutes.  **distance** is the Great Circle Distance (shortest distance between two points on the surface of a sphere) and is provided in the data set.  The coefficient **0.117** indicates that flights travel at 513 mph.  **lonorigin** and **londest** represent the longitudes of the originating and destination airports.  30 seconds of flight time is added for every degree of westbound longitude travelled.  The constant **43.2** indicates the time airlines budget for taxiing and inefficient routing (flying around severe weather).

Let's take a look at the first few rows of the data!


```
##     MONTH DAY_OF_WEEK CARRIER ORIGIN DEST CRS_DEP_TIME ARR_DELAY
## 1 January   Wednesday      AA    JFK  LAX      Morning        13
## 2 January    Thursday      AA    JFK  LAX      Morning         1
## 3 January    Saturday      AA    JFK  LAX      Morning        59
##   ACTUAL_ELAPSED_TIME DISTANCE           O_AIRPORT   O_CITY O_STATE
## 1                 384     2475 John F Kennedy Intl New York      NY
## 2                 389     2475 John F Kennedy Intl New York      NY
## 3                 379     2475 John F Kennedy Intl New York      NY
##   O_COUNTRY    O_LAT    O_LONG                  D_AIRPOT      D_CITY
## 1       USA 40.63975 -73.77893 Los Angeles International Los Angeles
## 2       USA 40.63975 -73.77893 Los Angeles International Los Angeles
## 3       USA 40.63975 -73.77893 Los Angeles International Los Angeles
##   D_STATE D_COUNTRY    D_LAT    D_LONG TARGET_TIME TARGET_DELAY
## 1      CA       USA 33.94254 -118.4081    355.8483     28.15173
## 2      CA       USA 33.94254 -118.4081    355.8483     33.15173
## 3      CA       USA 33.94254 -118.4081    355.8483     23.15173
```

## Feature Examination

In this section I take an independent look at the features expected to impact the on-time performance of aircraft.

### Month

### Day of the Week

### Airline

### Origin Airport

### Destination Airport

### Time of Departure



