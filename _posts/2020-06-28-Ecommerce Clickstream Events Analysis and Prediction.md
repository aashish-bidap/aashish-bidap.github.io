---
layout: single
title: "Ecommerce Clickstream Events Analysis and Prediction"
#date: 2020-06-28
#tags: [Predictive Modeling, Binary Text Classification,Clickstream Data,Data Trends]
classes: wide
excerpt: "Clickstream Data Analysis,E Commerce,Customer Behavior,Predictive Modeling,Binary Classfication,KNN , Python"
mathjax: "true"
---
With a sequence of clickstream events performed by a user on an e commerce website, following are the business outcomes which we addressed.<br>
1.Analysing customer behaviors in the data.
2.Predicting whether the customer would be making some purchase or not.<br>

As part of this project I have analyzed clickstream data for a large-scale ecommerce retailer in Europe which was published as part of the RecSys 2015 challenge. <br>

This data comprises a large collection of user visits to the European retailer’s website and records a sequence of events on users clicks. The data records the clickstream events for a period of six months in 2014 for the retailer website and also includes the buying events for some of the sessions thus describing more about the purchased items.

Dataset : https://2015.recsyschallenge.com/challenge.html

Source:
1. (2015). Ben-Shimon, Tsikinovsky, Friedmann, Shapira, Rokach & Hoerle .RecSys Challenge 2015 and the YOOCHOOSE Dataset . Association of computing Machinery.10.1145/2792838.2798723

## Technologies Used:
- Predictive Modeling
  - Binary Text Classification
   - Logistic Regression
   - Random Forest
   - Light GBM 
   - 2 Dense Layer Neural Net
    
## Environment
Python (scikitlearn, seaborn, pandas, numpy, matplotlib,Tensorflow,Keras) , Jupyter Notebook 

## Result
- While exploring the data I could find certain trends in the data revealing some important customer behaviors in the data.
  - Maximum clicks where observed on special offers.
  - Average dwell time for a positive buying event is 867.10 sec ~ 14.45 mins. 
  - Average dwell time for a negative buying event is 353.32 sec ~ - 8 mins. 
  - Maximum buying events have been observed on Sundays and Mondays.
  - Worst buying events are observed on Tuesdays.
  - Evening time is suitable for a greater number of buying events.
  - A total of 509,696 positive sessions are available in the training set reporting a buying event.
  - Also we found that the of all the buying events,70% of times the item which was clicked first and 72% of times the item which was last clicked were purchased by the customer.

- With LightGBM,gradient boosting framework for machine learning,implementation a considerable improvement in the accracy of the predictions was observed over logistic regression and Random Forest. Model outperformed with an AUC score of 79% over test data in order to predict the positive buying events.

Thus it was observed that by unifying the clickstream data and understanding the customer behaviour patterns it is possible to predict the journey of the customer.
