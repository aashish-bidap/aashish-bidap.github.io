---
layout: single
permalink: /about/
title: "US Airline Market Analysis"
#date: 2020-08-15
#tags: [data wrangling, data science, messy data]
classes: wide
#excerpt: "R programming:Exploratory Data Analysis,Descriptive Analysis,Predictive Analysis(Classification):Stepwise Logit Regression-Decision Tree-Random Forest Modeling,Data Visualization:ggplot2"
mathjax: "true"
---

## Description
-	Objective of the project is to analyse flight data for US for a period of 20 years(~128 million rows) to find patterns to gauge different choices of travel.
-   Identifying trends and patterns over the period of time.<br>
-	Using Spark Dataframe and Spark SQL for handling large data efficiently and Amazon S3 as the data lake.<br>

## Technology Used
-   Big Data Analysis using Spark
-   Amazon Web Services

## Environment
Python(PySpark), Spark SQL, Databricks, Amazon S3,Apache Parquet

Analysis: <a href="https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/8891199222519419/1251010126247737/720247590727539/latest.html"> Notebook </a>

## Architecture 
<br>
![Architecture](/images/airline_analysis/Architecture.png)

<br>

**Analysis 1 : Exploratory Data Analysis of Data:** <br>
![analysis1](/images/airline_analysis/analysis-1.png)

**Analysis 2 : Impact of Global Recession:** <br>
![analysis2](/images/airline_analysis/analysis-2.png)

**Analysis 3 : Fraud Data Analysis:** <br>
![analysis3](/images/airline_analysis/analysis-3.png)

## Performance Optmization
-	Repartitioning of the dataframe with optimized number of partitions & Speeding up Shuffle.partitions.<br>
-	Pulling data sets into a cluster-wide in-memory cache.<br>
-	Using Apache Parquet,columnar storage format which support flexible compression options and also provides an efficient encoding system.<br>
