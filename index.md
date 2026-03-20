---
layout: default
title: Predicting How Long the Lights Stay Off
---

# Power Outage: Predicting How Long the Lights Stay Off

**Vivian Zhu**

---

## Introduction

Power outages affect millions of Americans every year, disrupting daily life, 
businesses, and critical infrastructure. But not all outages have the same impact,
some last minutes and only a few blocks feel it. Others can span on for days,
and impact thousands of people. What factors determines how long an outage will last?

This project analyzes a dataset containing data about major power outage data from January 2000 to July 2016.
This data is from Purdue University's research data. The data contains information about where the
power outage occurred, when it occurred, what was its duration, its impact, and geographical, climate,
economical data on the surrounding area. 

I will perform various data cleaning and analysis on the data to gain an understanding of the data.
Then I will analyze the cause of missingness of data in a column. Then, I will explore the research question, do
power outages occurr at the same frequencies during the daytime and night time. Finally we will create a baseline model
and a final model for the central question:

> **Can we predict the duration of a power outage based on information 
> available at the time it begins?**


The dataset contains **1,534 rows** and **56 columns**, each representing a single major power 
outage event. For our project we will focus on thh following columns:

| Column | Description |
|--------|-------------|
| `YEAR` | Year the outage occurred |
| `MONTH` | Month the outage occurred |
| `U.S._STATE` | State where the outage occurred |
| `POSTAL.CODE` | Postal code where the outage occurred |
| `NERC.REGION` | North American Electric Reliability Corporation region involved in the outage event |
| `CLIMATE.REGION` | U.S. climate region specified by National Centers for Environmental Information |
| `CLIMATE.CATEGORY` | Climate categories (Warm, Cold ,Normal)based on a threshold of ± 0.5 °C for the Oceanic Niño Index (ONI) |
| `ANOMALY.LEVEL` | Oceanic El Niño/La Niña index indicating climate anomaly severity |
| `OUTAGE.START.DATE` | Date the outage began |
| `OUTAGE.START.TIME` | Time of day the outage began |
| `OUTAGE.RESTORATION.DATE` | Date power was fully restored |
| `OUTAGE.RESTORATION.TIME` | Time power was fully restored |
| `CAUSE.CATEGORY` | General cause of the outage |
| `HURRICANE.NAMES` | Name of the hurricane if the outage was due to hurricane |
| `OUTAGE.DURATION` | Duration of the outage in minutes |
| `CUSTOMERS.AFFECTED` | Number of customers affected by the outage |
| `TOTAL.PRICE` | Average monthly electricity price in the state (cents/kWh) |

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

We performed the following cleaning steps on the raw dataset:

1. **Combined date and time columns** — `OUTAGE.START.DATE` and `OUTAGE.START.TIME` 
were merged into a single `OUTAGE.START` timestamp column, and similarly for 
`OUTAGE.RESTORATION.DATE` and `OUTAGE.RESTORATION.TIME`. This makes it easier 
to compute durations and extract time features.

2. **Dropped rows with missing timestamps or duration** — Rows where 
`OUTAGE.START`, `OUTAGE.RESTORATION`, or `OUTAGE.DURATION` were null were 
removed. These represent outages where the start or end time was never recorded, 
making them impossible to use for duration prediction.

3. **Created `IS.HURRICANE` column** — Instead of keeping the raw 
`HURRICANE.NAMES` column (which was mostly null), we created a binary column 
indicating whether the outage was hurricane-related, then dropped the original column.

4. **Imputed missing `TOTAL.PRICE` values** — All null values in `TOTAL.PRICE` 
corresponded to July 2016. Since no July 2016 prices existed, we imputed using 
random sampling from July 2014 and July 2015 values, preserving the seasonal 
distribution of electricity prices from recent years.

5. **Filled missing `CLIMATE.REGION` with 'Hawaii'** — All null values in 
`CLIMATE.REGION` belonged to Hawaii, which is not part of the standard U.S. 
climate regions. We assigned these rows their own `'Hawaii'` region rather than 
dropping them.

6. **Extracted time features** — Added `START.HOURS` (hour of day, 0-23) and 
`DAY.OF.WEEK` (0=Monday, 6=Sunday) from the `OUTAGE.START` timestamp for use 
as model features.

After cleaning, the dataset contains **1,476 rows** and **19 columns**. Here are the first few rows:

| |   YEAR |   MONTH | U.S._STATE   | POSTAL.CODE   | NERC.REGION   | CLIMATE.REGION     | CLIMATE.CATEGORY   |   ANOMALY.LEVEL | OUTAGE.START.DATE   | OUTAGE.START.TIME   | OUTAGE.RESTORATION.DATE   | OUTAGE.RESTORATION.TIME   | CAUSE.CATEGORY     |   OUTAGE.DURATION |   CUSTOMERS.AFFECTED |   TOTAL.PRICE | OUTAGE.START        | OUTAGE.RESTORATION   | IS.HURRICANE   |   START.HOURS |   DAY.OF.WEEK |
|-------:|--------:|:-------------|:--------------|:--------------|:-------------------|:-------------------|----------------:|:--------------------|:--------------------|:--------------------------|:--------------------------|:-------------------|------------------:|---------------------:|--------------:|:--------------------|:---------------------|:---------------|--------------:|--------------:|
|   2011 |       7 | Minnesota    | MN            | MRO           | East North Central | normal             |            -0.3 | 2011-07-01 00:00:00 | 17:00:00            | 2011-07-03 00:00:00       | 20:00:00                  | severe weather     |              3060 |                70000 |          9.28 | 2011-07-01 17:00:00 | 2011-07-03 20:00:00  | False          |            17 |             4 |
|   2014 |       5 | Minnesota    | MN            | MRO           | East North Central | normal             |            -0.1 | 2014-05-11 00:00:00 | 18:38:00            | 2014-05-11 00:00:00       | 18:39:00                  | intentional attack |                 1 |                  nan |          9.28 | 2014-05-11 18:38:00 | 2014-05-11 18:39:00  | False          |            18 |             6 |
|   2010 |      10 | Minnesota    | MN            | MRO           | East North Central | cold               |            -1.5 | 2010-10-26 00:00:00 | 20:00:00            | 2010-10-28 00:00:00       | 22:00:00                  | severe weather     |              3000 |                70000 |          8.15 | 2010-10-26 20:00:00 | 2010-10-28 22:00:00  | False          |            20 |             1 |
|   2012 |       6 | Minnesota    | MN            | MRO           | East North Central | normal             |            -0.1 | 2012-06-19 00:00:00 | 04:30:00            | 2012-06-20 00:00:00       | 23:00:00                  | severe weather     |              2550 |                68200 |          9.19 | 2012-06-19 04:30:00 | 2012-06-20 23:00:00  | False          |             4 |             1 |
|   2015 |       7 | Minnesota    | MN            | MRO           | East North Central | warm               |             1.2 | 2015-07-18 00:00:00 | 02:00:00            | 2015-07-19 00:00:00       | 07:00:00                  | severe weather     |              1740 |               250000 |         10.43 | 2015-07-18 02:00:00 | 2015-07-19 07:00:00  | False          |             2 |             5 ||

---

### Univariate Analysis

<iframe src="plots/outage_histogram.html" width="800" height="500" frameborder="0"></iframe>

The distribution of outage durations is heavily right-skewed, with most outages 
lasting under 2,500 minutes (~1.7 days) but a long tail of extreme events 
stretching beyond 50,000 minutes. This skew motivated our use of a log 
transformation on the target variable during modeling.

<iframe src="plots/outage_average_by_cause.html" width="800" height="500" frameborder="0"></iframe>

Fuel supply emergencies and severe weather cause the longest average outages by 
far, while intentional attacks and public appeals tend to resolve much faster. 
This suggests that cause category will be an important predictor of outage duration.

---

### Bivariate Analysis

<iframe src="plots/avg_duration_by_state.html" width="800" height="500" frameborder="0"></iframe>

Average outage duration varies substantially by state, with several northeastern 
and midwestern states experiencing the longest outages on average. This geographic 
pattern suggests that regional infrastructure and climate conditions play a role 
in restoration time.

The boxplot below shows outage duration by climate category on a log scale:

<img src="plots/outage_by_climate.png" alt="Outage Duration by Climate Category" width="800">

Warm climate periods tend to produce slightly longer and more variable outages 
compared to cold or normal periods, though all three categories show considerable 
spread. The log scale reveals that the bulk of outages across all climate 
categories cluster between 100 and 10,000 minutes.

---

### Interesting Aggregates

We grouped outages by `CLIMATE.REGION` to examine regional patterns in outage 
severity and scale:

| (embed agg_by_cause.to_html() table here) |

The Northeast and East North Central regions stand out with the highest average 
outage durations and customer impact, likely reflecting their aging grid 
infrastructure and exposure to severe winter storms. The Southwest, despite 
having frequent outages, tends to resolve them faster on average.