---
layout: single
title: "Reddit Stock Analysis using AWS Cloud"
#date: 2020-08-15
#tags: [data wrangling, data science, messy data]
classes: wide
excerpt: "Event Driven Data Pipeline, Reddit Data Analysis, NLP, Chatbot, AWS, Dashboard Visualizations"
mathjax: "true"
---
## Description
### Finding out about stocks before they make the news using Reddit Data , Nasdaq Data &amp; Cloud Analytics Services from Amazon Web Services.
Lately, a group of online traders collaborated on Reddit & took down the hedge fund by bidding up the stock price of GameStop.This unprecedented move by the online traders took the market by storm with continuous ups and downs in the stock value. Therefore, we decided to come up with a project that can analyze the discussions on Reddit for “/r/WallStreetBets” and gives the users an idea about what are some of the stocks that are being discussed, sentiment of their discussion and further allowing the user to view the additional details about the required stock over an interactive dashboard.

## Environment
- AWS Cloud Envionment , Python

## Tasks performed:
- Reddit Data Collection using open REST API
- Event Driven Data Pipeline
- Conversational User Interface
- Natural Language Processing
- Data Analysis
- Interactive Dashboard Visualization

- Try the Application here : <a href="http://redstocks.com.s3-website-us-east-1.amazonaws.com/"> Project Demo </a> 
  - Amazon Lex : Chatbot user interface to easy access.<br>
  - AWS Lambda : Lambda Triggers to collect data from Reddit upon specific Lex Intent.<br>
  - Amazon S3 : Storage of the Output files [reddit scrapped data & nasdaq stock data] <br>
  - AWS Glue & AWS Athena : Athena Table for the Nasdaq Stocks Data.<br>
  - Amazon Comprehend : Text Analytics [Detect Entities & Detect Sentiment]<br>
  - Amazon QuickSight : Interactive dashboard allowing users to visualize the stock data attributes.<br>

## Architecture 
![Architecture](/images/architecture_reddit.png)

## Dashboard  
![Dashboard](/images/dashboard_itc6460.png)

## WebPage View
![WebPage](/images/webpage_itc6460.png)

- Benefits <br>
  - Chatbot easy interaction, no need to read the reddit discussions.<br>
  - We analyse the redditors posts and do all the heavy lifting.<br>
  - Provides an interactive dashboard where the user can visualize the stock attributes.<br>

- Assumptions <br>
  - Redditors on the channel “/WallStreetBets” discuss only about stocks and option trading.<br>
  - As fast-growing stocks are listed on Nasdaq, we have considered Nasdaq stocks data.<br>

