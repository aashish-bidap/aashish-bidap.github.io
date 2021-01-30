---
layout: single
title: "Customer Churn Prediction & Analysis for Telecom Data."
#date: 2020-08-15
#tags: [data wrangling, data science, messy data]
classes: wide
excerpt: "Churn Predition,R programming,Data Analysis,Predictive Analysis(Binary Classification),Data Visualization"
mathjax: "true"
---
## Description
The objective of the project is to build a model to predict the probability of the customer churning from the platfrom for a telecom data.<br>
Dataset Source : This dataset is based on IBM based sample dataset obtained from Kaggle for Customer Churn.<br>
Dimensions for the DataSet [rows vs columns]: 7043 * 21.<br>
The Data set includes information about:<br>
- Customers who left within the last month – the column is called Churn.<br>
- Services that each customer has signed up for – phone, multiple lines, internet, online security, online backup, device protection, tech support, and streaming TV and movies.<br>
- Customer account information – how long they’ve been a customer, contract, payment method, paperless billing, monthly charges, and total charges.<br>
- Demographic info about customers – gender, age range, and if they have partners and dependents.<br>

## Environment
- R Studio, R (ggplot2,dplyr,randomForest,corrplot)

## Tasks performed:
- Data Cleaning
- EDA
- Feature Engineering and Optimization
- Data Modeling.
- We implemented Logistic Regresion and found an accuracy of 79.01%.
- Further we tried implementing the Decision Tree classifier but did not find any improvement in our accuracy.
- Lastly we performed random forest claasifier and we observed an accuracy of 80.07%

## Findings
Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_16_0.png)

<b>1.  The data includes almost equal proportion of males and females.<br>
<b>2.  Almost 58% customers are on paperless billing.<br>
<b>3.  26% of the customers have churned from the platform.


Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_18_0.png)

<b>1. Almost 40% of the customers have subscribed for the Fibre optic internet service.<br>
<b>2. Almost 50% of the customers have no online security and almost 45% customers have no online backup.<br>
<b>3. Almost 50% customers have no techsupport access and 40% have no streamingtv as a service.<br>
<b>4. 45% of the customers have no service of device protection.


Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_20_0.png)

<b>1. Maximum number of customers have subscribered for electronic check for their payments.<br>
<b>2. Very less i.e approx 20% of the customers are senior citizens.<br>
<b>3. Equal number of customers with and without partners.<br>
<b>4. 65% of the customers have no dependents.<br>
<b>5. Almost 87% of the customers are with the phoneservice.


Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_23_0.png)
<br>
![png](/images/churn/Telco%20Churn%20Analysis_23_1.png)

<b>1. Maximum Customers churned from the platform are the one having a tenure of 0-1 years.<br>
<b>2. Maximum Churned customers have a Monthly charge more than $65.

Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_26_0.png)

<b>* Churn rate is equally divided among the male and female customers.<br>
<b>* Churn rate is more among the customers with no dependents.<br>
<b>* Churn rate is more with customers having phone service.<br>
<b>* Churn rate is more with customers having paperless billing.<br>
<b>* Churn rate is more with customers having electronic check as the payment mode.


Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_28_0.png)
<br>
<b>* Churn rate is more with customers having month to month contract.<br>
<b>* Churn rate is more with customers having no online security and techsupport.<br>
<b>* Churn rate is almost equal among the subscribers with or without the streamingtv.<br>


Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_47_1.png)
<br>
<b>With the above Logistic regression kmodel we could see an accuracy of 78% in the model.

Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_52_0.png)


****2.DECISION TREE****

Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_57_0.png)

***3. Random Forest Modeling***

Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_63_0.png)

***Accuracy Comparison for the three Models***

    Accuracy for Logistic Model 0.7901063
    Accuracy for Decision Tree Model 0.7928803
    Accuracy for Random Forest Model 0.8007397

**ROC analysis for the three Models**

Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_68_0.png)

***BUSINESS COST ASSUMPTION :***

Output:<br>
![png](/images/churn/Telco%20Churn%20Analysis_72_0.png)

***Results***

- We successfully implemented three classification models in order to predict the potential churn customers. Considering the compares the results aforementioned business cost assumptions Random Forest Model can be considered to be a best fit model of the three implemented models with the best accuracy.<br>
- Based upon the descriptive analysis , a greater number of Churn customers are observed in case of customers with Tenure in between 0-1 year and with increase in the Monthly Charges there is substantial increase in the Churn Rate.<br>
- Customer on Month to Month contract are more susceptible to Churn.<br>
- 22% of the total customers were observed with No Internet Service and thus customers are missing the exclusive services of OnlineBackup, StreamingTV, Online Security and Device Protection.
