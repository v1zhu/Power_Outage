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