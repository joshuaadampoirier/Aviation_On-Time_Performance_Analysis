---
title: "Aviation On-Time Performance Analysis"
author: "Joshua Poirier"
date: "Tuesday, January 12, 2016"
output: pdf_document
---

## Abstract

This study is inspired by the web application and study by Ritchie King and Nate Silver at fivethirtyeight.com titled *Which Flight Will Get You There Fastest?*.  The web application can be found [here](http://projects.fivethirtyeight.com/flights/).  The accompanying [documentation](http://fivethirtyeight.com/features/how-we-found-the-fastest-flights/) describes what they mean by "fastest" flights: 

> Airline A says it will fly you from Seattle to Portland, Oregon, in 45 minutes, but actually takes 60 minutes.  Airline B says it will fly the same route in 75 minutes, and actually takes 70 minutes.  Which flight would you rather take?  That seems easy. Airline A!

My criticism of this approach is that it does not consider connecting passengers who should prioritize scheduling over pure speed.  If a passenger is travelling from SEA-PDX-HNL (Seattle - Portland - Honolulu) - timing that connection in Portland is important!  If both airlines schedule a mere 10 minutes between flights the passenger misses the connection with Airline A, but has a total of 15 minutes to make their connection with Airline B.  I therefore build models optimizing the two cases.

* **Case 1:** Direct Flight - The consumer wants the fastest flight as defined in the aforementioned study.  This is defined as the difference between the actual arrival time and the target arrival time.
* **Case 2:** Connecting Flight - The consumer wants an on-time flight in order to avoid missing a tight connection.  This is defined as the difference between the actual arrival time and the scheduled arrival time.

I hypothesize that the 3 best airlines for the two cases are different.  To show this, a multivariate regression model will be built for each case to account for confounding variables including **month**, **day of week**, **origin airport**, **destination airport**, and **time of day** in addition to the **airline** variable.



## Introduction

This study utilizes aviation on-time performance data provided by the United States Department of Transportation [here](http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time).  The study analyzes **1812011** flights and includes hundreds of dummy variables in the culminating multivariate regression model comparing Cases 1 and 2.  **Case 2** will use the *ARR_DELAY* field to compare against scheduled arrivals.  **Case 1** will use the difference between the gate-to-gate flight time (*ACTUAL_ELAPSED_TIME* field) and the **target time**.  I calculate target time using the simplified formula shown by Ritchie King and Nate Silver.  

$target time = 0.117 * distance + 0.517 * (lon origin - lon dest) + 43.2$  

This formula produces an estimated travel time in minutes.  **distance** is the Great Circle Distance (shortest distance between two points on the surface of a sphere) and is provided in the data set.  The coefficient **0.117** indicates that flights travel at 513 mph.  **lonorigin** and **londest** represent the longitudes of the originating and destination airports.  30 seconds of flight time is added for every degree of westbound longitude travelled.  The constant **43.2** indicates the time airlines budget for taxiing and inefficient routing (flying around severe weather).

In the interest of conciseness, the code used to produce these results is omitted from this report; however, it is available in the R Markdown file used to produce this report [here](https://github.com/joshuaadampoirier/Aviation_On-Time_Performance_Analysis).  Let's take a look at the first few rows of the raw data!


|MONTH |DAY_OF_WEEK |CARRIER |ORIGIN |DEST |CRS_DEP_TIME | ARR_DELAY|
|:-----|:-----------|:-------|:------|:----|:------------|---------:|
|Jan   |3           |AA      |JFK    |LAX  |0900         |        13|
|Jan   |4           |AA      |JFK    |LAX  |0900         |         1|
|Jan   |6           |AA      |JFK    |LAX  |0900         |        59|



| ACTUAL_ELAPSED_TIME| DISTANCE|
|-------------------:|--------:|
|                 384|     2475|
|                 389|     2475|
|                 379|     2475|

## Model Building



**Month**, **Day of Week**, **Origin airport**, **Destination airport**, and **Scheduled departure time** are confounders which are included in the linear regression model in addition to **airline/carrier**.  The **APPENDIX** analyzes these confounders on an independent basis to illustrate that they indeed impact on-time flight performance using hypothesis tests and confidence intervals.  

Due to the computational complexity and size of the data set, a single linear regression model could not be computed using the entire data set.  As such, I will apply the Monte Carlo method to build **10** linear regression models for each case based on random **1**% samples of the data set (without replacement).  The null hypothesis states that the means for each airline (for each case) are equal; while, the alternate hypothesis states that the means are not equal.

$H_0: 0 = \mu_1 = \mu_2 = \mu_3 = ... = \mu_12$  
$H_\alpha: 0 \neq \mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_12$

To evaluate the null hypothesis I compute the 95% confidence interval of the mean delay for each case.  If the confidence intervals do not all overlap we reject the null hypothesis.



|Carrier |     Lower1|      Mean1|     Upper1|    Lower2|      Mean2|     Upper2|
|:-------|----------:|----------:|----------:|---------:|----------:|----------:|
|AS      |  2.0072375|  2.6053170|  3.2033966| -5.675359| -4.4435276| -3.2116964|
|B6      | -1.5998535| -1.1962783| -0.7927031|  5.771006|  7.4853378|  9.1996700|
|DL      | -0.7722744| -0.3999910| -0.0277077|  2.173107|  3.3022240|  4.4313408|
|EV      |  0.8755450|  1.3636409|  1.8517367|  9.584549| 10.5266526| 11.4687565|
|F9      | -2.3452211| -1.7599963| -1.1747716|  5.480216|  7.6523299|  9.8244439|
|FL      | -3.3253686| -2.8400895| -2.3548103| -1.006703|  0.8421222|  2.6909473|
|HA      |  0.8187973|  2.0566600|  3.2945226| -4.688502|  0.7729921|  6.2344867|
|MQ      |  2.0132742|  2.4178206|  2.8223670|  5.160540|  6.4661345|  7.7717293|
|OO      |  3.6427207|  4.1228876|  4.6030544|  2.874141|  4.5548483|  6.2355552|
|UA      | -1.9779547| -1.3670423| -0.7561299| -2.668806| -1.6330639| -0.5973217|
|US      |  0.3704770|  0.8467397|  1.3230024| -2.556134| -1.2256202|  0.1048938|
|VX      | -2.0446191| -1.6825393| -1.3204596| -3.083781| -0.6429801|  1.7978208|
|WN      | -3.8364123| -3.4267270| -3.0170416|  7.563363|  8.7703416|  9.9773199|

The above table shows that the confidence intervals (ranging from the value *Lower1* to *Upper1* and *Lower2* to *Upper2* for Case 1: Direct and Case2: Connecting respectively) do not overlap; therefore, we reject the null hypothesis and state that the airline/carrier impacts the on-time flight performance for both cases.

## Conclusions

The top 3 airlines for Case 1: Optimizing for a direct flight, and Case 2: Optimizing for a connecting flight are shown below.


|Airline                     | Mean Delay (min)|
|:---------------------------|----------------:|
|Southwest Airlines Co.      |        -3.426727|
|AirTran Airways Corporation |        -2.840089|
|Frontier Airlines Inc.      |        -1.759996|



|Airline                                                                             | Mean Delay (min)|
|:-----------------------------------------------------------------------------------|----------------:|
|Alaska Airlines Inc.                                                                |        -4.443528|
|United Air Lines Inc.                                                               |        -1.633064|
|US Airways Inc. (Merged with America West 9/05. Reporting for both starting 10/07.) |        -1.225620|

The hypothesis is thereby confirmed as the top 3 airlines for on-time performance for the two cases are not the same.  Southwest Airlines Co. on average arrives at their destination 3.43 minutes early relative to an independently calculated target time (**Case 1: Direct Flight Optimization**).  Alaska Airlines Inc. on average arrives at their destination 4.44 minutes early relative to their own scheduling (**Case 2: Connecting Flight Optimization**) - making them the best option for a connecting flight.

# APPENDIX

In this section I take an independent look at each feature expected to impact the on-time performance of aircraft.

### Month

The *month* of the flight is expected to impact on-time performance due to the seasonality experienced by the United States.  Airports experiencing harsh winter conditions are expected to perform more poorly during the winter months (December through March).  To show this, let us establish a null hypothesis stating population means for each month are equal while the alternative hypothesis states the means are not equal (the indices 1 through 12 represent January through December).

$H_0: 0 = \mu_1 = \mu_2 = \mu_3 = ... = \mu_12$  
$H_\alpha: 0 \neq \mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_12$

For this, I compute the 95% confidence interval for the mean delays for each month (less the overall mean).  If all confidence intervals do not include 0 we reject the null hypothesis.  




```
## Use suppressPackageStartupMessages to eliminate package startup messages.
```

![plot of chunk plotMonthCI](figure/plotMonthCI-1.png) 

|X   |     Lower1|      Mean1|     Upper1|
|:---|----------:|----------:|----------:|
|Apr | -1.0916424| -1.0531923| -1.0147423|
|Feb |  0.9032465|  0.9532444|  1.0032422|
|Jan |  0.8545432|  0.9031737|  0.9518043|
|Mar | -0.6113358| -0.5719228| -0.5325098|



|X   |    Lower2|     Mean2|    Upper2|
|:---|---------:|---------:|---------:|
|Apr | -3.466254| -3.365328| -3.264401|
|Feb |  1.411596|  1.540049|  1.668503|
|Jan |  4.512049|  4.654241|  4.796433|
|Mar | -2.264782| -2.165042| -2.065302|

Since the confidence intervals for each month do not all contain 0 we reject the null hypothesis and state that the mean delays for each month are not equal.  Therefore we include **Month** as a variable in the final model.

### Day of the Week

The *day of week* of the flight is expected to impact on-time performance due to employee scheduling, and different types of passengers for different days of the week.  Airport/airline staff do not work seven days per week so it is fair to suggest there may be fluctuations in airport/airline efficiencies due to staffing levels as well as the experience and efficiency of staff on duty.  Business travellers are expected to be more likely on week days; while, personal travellers (including families) are expected to be more likely on weekends.  The type of passenger (not recorded in this data set) may generate small departure delays due to increased boarding times and ultimately scheduled arrival delays.  For each case I establish a null hypothesis stating that the mean delays for each day of the week are equal to 0; while, the alternative hypothesis states they are not all equal (the indices 1 through 7 represent Monday through Sunday).

$H_0: 0 = \mu_1 = \mu_2 = \mu_3 = ... = \mu_7$  
$H_\alpha: 0 \neq \mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_7$

For this, I compute the 95% confidence interval for the mean delays for each day of the week (less the overall mean).  If all confidence intervals do not include 0 we reject the null hypothesis.



![plot of chunk plotWeekdayCI](figure/plotWeekdayCI-1.png) 

|X   |     Lower1|      Mean1|     Upper1|
|:---|----------:|----------:|----------:|
|Mon | -0.1108216| -0.0544967|  0.0018282|
|Tue | -0.2303638| -0.1732334| -0.1161029|
|Wed |  0.1288225|  0.1857877|  0.2427530|
|Thu |  0.7235494|  0.7832166|  0.8428837|
|Fri | -0.0900403| -0.0350488|  0.0199426|
|Sat | -0.7867062| -0.7235566| -0.6604071|
|Sun | -0.1924484| -0.1336164| -0.0747844|



|X   |     Lower2|      Mean2|     Upper2|
|:---|----------:|----------:|----------:|
|Mon | -0.1562590| -0.0040155|  0.1482279|
|Tue | -2.7301665| -2.5880627| -2.4459590|
|Wed | -0.4255603| -0.2766170| -0.1276736|
|Thu |  2.0393685|  2.1975444|  2.3557204|
|Fri |  1.6335672|  1.7905640|  1.9475608|
|Sat | -1.0665237| -0.8910790| -0.7156344|
|Sun | -0.7267779| -0.5689149| -0.4110519|

As shown in the above plot and tables, not all of the confidence intervals include 0.  Therefore, we reject the null hypothesis and include **Day of Week** as a variable in the final model.

### Origin Airport

Airports are a key factor impacting the on-time performance of flights.  Design, length of taxi, traffic volume are but some of the airport-specific features contributing to on-time flight performance.  The data set being analyzed features 303 origin airports.  As a result we'll visualize it a bit differently.  We still perform a hypothesis testing (the indices represent different airports).

$H_0: 0 = \mu_1 = \mu_2 = \mu_3 = ... = \mu_n$  
$H_\alpha: = \neq \mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_n$

For this, I compute the 95% confidence interval for the mean delays for each day of the week (less the overall mean).  If all confidence intervals do not include 0 we reject the null hypothesis.

![plot of chunk originHet](figure/originHet-1.png) ![plot of chunk originHet](figure/originHet-2.png) 

The above plots show that some of the origin airport categories *pass* the hypothesis test; however, the hypothesis test states that the mean for each origin airport must be 0 in order to pass.  Since many of the airports fail the test, we reject the null hypothesis and include the origin airport as a variable in the model.

### Destination Airport

Airports are a key factor impacting the on-time performance of flights.  Design, length of taxi, traffic volume are but some of the airport-specific features contributing to on-time flight performance.  The data set being analyzed features 303 destination airports.  As a result we'll visualize it a bit differently.  We still perform a hypothesis testing (the indices represent different airports).

$H_0: 0 = \mu_1 = \mu_2 = \mu_3 = ... = \mu_n$  
$H_\alpha: = \neq \mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_n$

For this, I compute the 95% confidence interval for the mean delays for each day of the week (less the overall mean).  If all confidence intervals do not include 0 we reject the null hypothesis.

![plot of chunk destHet](figure/destHet-1.png) ![plot of chunk destHet](figure/destHet-2.png) 

The above plots show that some of the destination airport categories *pass* the hypothesis test; however, the hypothesis test states that the mean for each origin airport must be 0 in order to pass.  Since many of the airports fail the test, we reject the null hypothesis and include the destination airport as a variable in the model.

### Time of Departure

The *time of day* of the flight is expected to impact on-time performance.  A possible mechanism for this is that aircraft typically service multiple flights per day.  Delayed flights in the morning will have a cascading effect on later flights using the same aircraft.  For this analysis we cut the continuous time data into **Morning** (05:00-11:59), **Afternoon** (12:00-17:59), and **Evening** (18:00-23:59 and 00:00-04:59) to match filtering options offered by flight retailers such as [Expedia](https://www.expedia.com/).  For each case (Direct vs. Connecting) I establish a null hypothesis stating that the mean delays for each time of day category are equal to 0; while, the alternative hypothesis states they are not all equal (the indices 1 through 3 indicate morning, afternoon, and evening).

$H_0: 0 = \mu_1 = \mu_2 = \mu_3 = ... = \mu_7$  
$H_\alpha: 0 \neq \mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_7$

For this, I compute the 95% confidence interval for the mean delays for each day of the week (less the overall mean).  If all confidence intervals do not include 0 we reject the null hypothesis.



![plot of chunk plotTimeCI](figure/plotTimeCI-1.png) 

|X         |     Lower1|      Mean1|     Upper1|
|:---------|----------:|----------:|----------:|
|Morning   |  1.2423995|  1.2771603|  1.3119212|
|Afternoon | -0.2085845| -0.1734742| -0.1383638|
|Evening   | -2.2404045| -2.1934504| -2.1464962|



|X         |    Lower2|     Mean2|    Upper2|
|:---------|---------:|---------:|---------:|
|Morning   | -4.637972| -4.551448| -4.464924|
|Afternoon |  2.763639|  2.861699|  2.959758|
|Evening   |  3.548455|  3.682318|  3.816181|

Interestingly we observe opposite relationships between the two cases.  This may reflect shifting scheduled flight times to accommodate varying taxi times, and weather patterns (midday convective currents for example) throughout the day.  Since not all of the 95% confidence intervals include 0, we reject the null hypothesis and include the variable in the final model.


