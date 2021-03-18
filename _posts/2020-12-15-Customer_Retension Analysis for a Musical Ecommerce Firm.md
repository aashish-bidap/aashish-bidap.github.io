---
layout: single
title: "Customer Retention ANalysis for a Musical E-Commerce Firm (Quantum Analytica - Experential Learning)"
#date: 2020-08-15
#tags: [data wrangling, data science, messy data]
classes: wide
excerpt: "E Commerce, Transactional Data Analysis, Association Analysis,Dashboard Visualizations"
mathjax: "true"
---
## Description
### As part of academic program (Experiential Learning), provided a solution to addressing the business objective of Improving the Customer Retension for a Musical Ecommerce Firm. <br>

## Solution
- Developed an interactive dashboard to analyse repurchasing patterns of customers by analysing in house orders data collected over past 10 years.<br>
- Identified the duration between the repeat purchases for each product.<br>
- Analysed trends in repurchases, change in repeat purchase rate over years & business from new customers vs returning customers.
- Furthermore Implement Apriori analysis to find the associations in the frequently purchased products.<br>
- Based on the analysis the dashboard also provided a comprehensive list of target customers that will be used by the marketing team to initiate targeted email campaigns.<br>
- <a href='https://acb-dashboard.herokuapp.com/'>Dashboard Implementation </a>

## Environment
- Python (Numpy,Pandas,Plotly,Streamlit)
- Heroku

## Data Visualizations
![EDA1](/images/acb/1.png)
![EDA2](/images/acb/2.png)
![EDA3](/images/acb/3.png)
![EDA4](/images/acb/4.png)
![EDA5](/images/acb/5.png)
![EDA6](/images/acb/6.png)

## Results:
On studying the associations between the categories in the dataset we see that,
- There is a strong relation between BB trumpet, Mouthpieces, and Cases & Gig Bags with a confidence of 52% and a lift value of 1.49.
- Correlations between mutes and mouthpieces, repair services and mouthpieces, cornets and mouthpieces, valve oil and mouthpieces with a confidence greater than 30%.
- Valve Oils are purchased after a median of 269 days, Mouthpieces after 141 days, and cases and gig bags after 279 days.

Based on the observations made, client can make recommendations to their customers with target email campaigns.
