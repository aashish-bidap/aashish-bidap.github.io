---
layout: single
title: "Cannabis Market Analysis for Colorado & Massachusetts"
#date: 2020-08-15
#tags: [data wrangling, data science, messy data]
classes: wide
excerpt: "Data Analysis, Predictive Modeling, Data Visualization, NLP, Multi Label Classification, Tableau, Python"
mathjax: "true"
---
## Description
### Analyzed the Cannabis Market of Colorado and Massachusetts with the Dispensary, Strain & publicly available Sales datasets.
The global cannabis market size was valued at USD 10.60 billion in 2018 and is projected to reach USD 97.35 billion by the end of 2026.
Considering this growing industry,the cannatech industry is developing software solutions targeted at improving the day-to-day operations of cannabis brands, manufacturers, distributors, and retailers. Data Analytics can help cannabis businesses accurately determine which products to develop and segments of the market in which to position them.<br> 

Our data analysis focusses towards the emerging cannabis market of Colorado & Massachusetts and delivers an end to end data driven dashboarding solution which can be used by the dispensary executives for market research.

## Data Insights that were analyzed
- Identifying majorly sold Product Categories
- Price variations across the categories
- Dispensaries characteristics based on Geography
- Top performing City/County in CO & MA
- Sales comparison/trends across the States
- Top Strain variants based on Cannabis type

## Environment
- Python, Tableau

## Tasks performed:
- Data Collection
- Data Cleaning
- Natural Language Processing
- Transfer Learning (Word2Vec)
- Multi Label ML Classifier

## Data
![Data](/images/cannabis/data.png)

## NLP Architecture 
<b> Handling Null Product Categories in Products Data by implementing a Multi Label Classifier that predicts the 15 Product Categories(Target) using the Product names(Independent Variable).<b><br>

Implemented the below mentioned ML models:<br>
- Linear Discriminant Analysis
- Support Vector Machine
- Random Forest Classifier 
- Gradient Boosting Classfier

Based on the model's performance validation, Random Forest outperformed the other models with a testing accuracy of 74.5%.The model predictions where further used to impute the missing values under the product categories attribte in the products data.

![NLP Architecture](/images/cannabis/nlp.png)

## Dashboard  
<a href="https://public.tableau.com/app/profile/ashishbidap/viz/CannabisMarketAnalysis-COMA-Dashboard/Dashboard1"> Tableau Dashboard </a>

- Benefits <br>
  - One stop data dashboarding solution for researching the Cannabis Markets of Colorado and Massachusetts.<br>
  - Interactive dashboard allowing the user to make data driven decisions to visualize the results across dispensary geographies, Sales across states/counties, Product Price Variations & Strain Trends.<br>
