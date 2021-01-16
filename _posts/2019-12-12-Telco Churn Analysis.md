---
title: "Customer Churn Prediction & Analysis for Telecom Data."
#date: 2018-01-28
#tags: [data wrangling, data science, messy data]
header:
  #image: "/images/churn/churn1.png"
excerpt: "R programming:Exploratory Data Analysis,Descriptive Analysis,Predictive Analysis(Classification):Stepwise Logit Regression-Decision Tree-Random Forest Modeling,Data Visualization:ggplot2"
mathjax: "true"
---
***Customer Churn Prediction & Analysis for Telecom Data.***
<br><br>
***Dataset Source*** : This dataset is based on IBM based sample dataset obtained from Kaggle for Customer Churn.
<br><br>
***Dimensions for the DataSet [rows vs columns]*** : 7043 * 21.
<br><br>
***The Data set includes information about*** :
<br><br>
Customers who left within the last month – the column is called Churn. Services that each customer has signed up for – phone, multiple lines, internet, online security, online backup, device protection, tech support, and streaming TV and movies.<br><br>
Customer account information – how long they’ve been a customer, contract, payment method, paperless billing, monthly charges, and total charges.<br><br>
Demographic info about customers – gender, age range, and if they have partners and dependents.
<br><br>
***Analysis***  :
<br>As the target variable in our analysis was a binary attribute , thus the problem we were dealing with is a Classification problem.
<br>Tasks performed:
<br><t>1.Data Cleaning
<br>2.EDA
<br>3.Feature Engineering and Optimization
<br>4.Data Modeling.
<br><br>
1.We implemented Logistic Regression and found an accuracy of 79.01%.
<br>
2.Further we tried implementing the Decision Tree classifier but did not find any improvement in our accuracy.
<br>
3.Lastly we performed random forest classifier and we observed an accuracy of 80.07% [best model]<br>
***Importing packages***
```R
library(ggplot2)
library(dbplyr)
library(dplyr)
library(gridExtra)
library(corrplot)
library(dummies)
library(party)
library(MASS)
library(pROC)
library(caret)
library(rpart)
library(randomForest)
library(rpart.plot)
```
***Data***
```R
#importing data
my_data <- read.csv("../input//telco-customer-churn//WA_Fn-UseC_-Telco-Customer-Churn.csv")
head(my_data)
```
***Data Overview***
```R
my_missing_NA_value_function <- function(dataset){

  Total_NA <- sum(is.na(dataset))
  Column_sums <- colSums(is.na(dataset))
  cat("Total NA in the dataset - \n\n",Total_NA)
    cat("\n--------------##-----------------")
  cat('\n\n Total NA by column in the dataset-\n\n',Column_sums)
        cat("\n--------------##-----------------")
  Column_names <- colnames(dataset)[apply(dataset,2,anyNA)]
  cat('\n\n Names of NA columns in the dataset-\n\n',Column_names)
}

my_data_overview <- function(dataset){
  data <- dim(dataset)
  cat("\nTotal Number of [rows vs columns] in the dataset- \n",data)
        cat("\n--------------##-----------------")
  Column_datatypes <- sapply(dataset,class)
  cat('\n\n Datatypes of all the columns in the dataset-\n',Column_datatypes)
        cat("\n--------------##-----------------")
  Column_Names <- colnames(dataset)
  cat('\n\n Names of all the columns in the dataset-\n',Column_Names)    
}
```
```R
my_missing_NA_value_function(my_data)
```
Output:

    Total NA in the dataset -

     11
    --------------##-----------------

     Total NA by column in the dataset-

     0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 11 0
    --------------##-----------------

     Names of NA columns in the dataset-

     TotalCharges


```R
my_data_overview(my_data)
```

    Total Number of [rows vs columns] in the dataset-
     7043 21
    --------------##-----------------

     Datatypes of all the columns in the dataset-
     factor factor integer factor factor integer factor factor factor factor factor factor factor factor factor factor factor factor numeric numeric factor
    --------------##-----------------

     Names of all the columns in the dataset-
     customerID gender SeniorCitizen Partner Dependents tenure PhoneService MultipleLines InternetService OnlineSecurity OnlineBackup DeviceProtection TechSupport StreamingTV StreamingMovies Contract PaperlessBilling PaymentMethod MonthlyCharges TotalCharges Churn

***Data Manipulation***

```R
#count of missing values
sum(is.na(my_data$TotalCharges))

#missing value deletion
my_data <- na.omit(my_data)
```
Output:

11

```R
#senior citizen is in integer form
my_data$SeniorCitizen <- as.factor(my_data$SeniorCitizen)

#creating new column tenure_bin and converting the tenure in years.
my_data <- mutate(my_data,tenure_bin=tenure)
my_data$tenure_bin[my_data$tenure_bin >= 0 & my_data$tenure_bin <= 12]   <- "0 - 1 years"
my_data$tenure_bin[my_data$tenure_bin >= 13 & my_data$tenure_bin <= 24]  <- "1 - 2 years"
my_data$tenure_bin[my_data$tenure_bin >= 25 & my_data$tenure_bin <= 36]  <- "2 - 3 years"
my_data$tenure_bin[my_data$tenure_bin >= 37 & my_data$tenure_bin <= 48]  <- "3 - 4 years"
my_data$tenure_bin[my_data$tenure_bin >= 49 & my_data$tenure_bin <= 60]  <- "4 - 5 years"
my_data$tenure_bin[my_data$tenure_bin >= 61 & my_data$tenure_bin <= 72]  <- "5 - 6 years"

#converting the newly created column into factor tenure_bin.
my_data$tenure_bin = as.factor(my_data$tenure_bin)
```

```R
#In the column MultipleLines modifying the No phone service -> No
my_data$MultipleLines[which(my_data$MultipleLines=="No phone service")]<-"No"
```

***Exploratory Data Analysis***

```R
#Graphical Representation of the Numerical Variables histograms
plot_A <- ggplot(my_data,aes(x = tenure_bin,fill=tenure_bin))+ geom_bar(width=0.4)
plot_B <- ggplot(my_data,aes(x = MonthlyCharges))+ geom_bar(fill="Steelblue",width=0.4)
plot_C <- ggplot(my_data,aes(x = TotalCharges))+ geom_histogram(fill="steelblue",binwidth =500)

grid.arrange(plot_A,plot_B,plot_C)
```

Output:
![png](/images/churn/Telco%20Churn%20Analysis_14_1.png)

```R
plot4 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = PaymentMethod,y=..prop..,group=2),fill="Steelblue",stat='count',width=0.4)
plot5 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = gender,y=..prop..,group=2),fill="Steelblue",stat='count',width=0.4)
plot6 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = SeniorCitizen,y=..prop..,group=2),fill="Steelblue",stat='count',width=0.4)
plot7 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = Partner,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot8 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = Dependents,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot9 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = PhoneService,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot10 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = MultipleLines,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot11 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = InternetService,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot12 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = OnlineSecurity,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot13 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = OnlineBackup,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot14 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = DeviceProtection,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot15 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = TechSupport,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot16 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = StreamingTV,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot17 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = StreamingMovies,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot18 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = Contract,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot19 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = PaperlessBilling,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
plot20 <- ggplot(data=my_data) + geom_bar(mapping = aes(x = Churn,y=..prop..,group=2),fill="Steelblue",width=0.4,stat='count')
```


```R
grid.arrange(plot5,plot19,plot20,ncol=3)
```

Output:
![png](/images/churn/Telco%20Churn%20Analysis_16_0.png)

<b>1.  The data includes almost equal proportion of males and females.<br>
<b>2.  Almost 58% customers are on paperless billing.<br>
<b>3.  26% of the customers have churned from the platform.


```R
grid.arrange(plot11,plot12,plot13,plot16,plot14,plot15,nrow=2, ncol=3)
```

Output:
![png](/images/churn/Telco%20Churn%20Analysis_18_0.png)


<b>1. Almost 40% of the customers have subscribed for the Fibre optic internet service.<br>
<b>2. Almost 50% of the customers have no online security and almost 45% customers have no online backup.<br>
<b>3. Almost 50% customers have no techsupport access and 40% have no streamingtv as a service.<br>
<b>4. 45% of the customers have no service of device protection.


```R
grid.arrange(plot4,plot6,plot7,plot8,plot9,plot10,nrow=2,ncol=3)
```

Output:
![png](/images/churn/Telco%20Churn%20Analysis_20_0.png)


<b>1. Maximum number of customers have subscribered for electronic check for their payments.<br>
<b>2. Very less i.e approx 20% of the customers are senior citizens.<br>
<b>3. Equal number of customers with and without partners.<br>
<b>4. 65% of the customers have no dependents.<br>
<b>5. Almost 87% of the customers are with the phoneservice.

**Numerical Summaries of  Variables**
```R
#Gender
cat("Gender")
type_counts1 <- table(my_data$gender)
type_counts1 / sum(type_counts1) * 100

#SeniorCitizen
cat("SeniorCitizen")
type_counts2 <- table(my_data$SeniorCitizen)
type_counts2 / sum(type_counts2) * 100

#Partner
cat("Partner")
type_counts3 <- table(my_data$Partner)
type_counts3 / sum(type_counts3) * 100

#Dependents
cat("SeniorCitizen")
type_counts4 <- table(my_data$Dependents)
type_counts4 / sum(type_counts4) * 100

#PhoneService
cat("Dependents")
type_counts5 <- table(my_data$PhoneService)
type_counts5 / sum(type_counts5) * 100

#MultipleLines
cat("MultipleLines")
type_counts6 <- table(my_data$MultipleLines)
type_counts6 / sum(type_counts6) * 100

#InternetService
cat("InternetService")
type_counts7 <- table(my_data$InternetService)
type_counts7 / sum(type_counts7) * 100

#OnlineSecurity
cat("OnlineSecurity")
type_counts8 <- table(my_data$OnlineSecurity)
type_counts8 / sum(type_counts8) * 100

#OnlineBackup
cat("OnlineBackup")
type_counts9 <- table(my_data$OnlineBackup)
type_counts9 / sum(type_counts9) * 100

#DeviceProtection
cat("DeviceProtection")
type_counts10 <- table(my_data$DeviceProtection)
type_counts10 / sum(type_counts10) * 100

#TechSupport
cat("TechSupport")
type_counts11 <- table(my_data$TechSupport)
type_counts11 / sum(type_counts11) * 100

#StreamingTV
cat("StreamingTV")
type_counts12 <- table(my_data$StreamingTV)
type_counts12 / sum(type_counts12) * 100

#Streaming Movies
cat("Streaming Movies")
type_counts13 <- table(my_data$StreamingMovies)
type_counts13 / sum(type_counts13) * 100

#Contract
cat("Contract")
type_counts14 <- table(my_data$Contract)
type_counts14 / sum(type_counts14) * 100

#PaperlessBilling
cat("PaperlessBilling")
type_counts15 <- table(my_data$PaperlessBilling)
type_counts15 / sum(type_counts15) * 100

#PaymentMethod
cat("PaymentMethod")
type_counts16 <- table(my_data$PaymentMethod)
type_counts16 / sum(type_counts16) * 100

#Churn
cat("Churn")
type_counts17 <- table(my_data$Churn)
type_counts17 / sum(type_counts17) * 100

```
Output:<br>
    Gender
      Female     Male
    49.53072 50.46928

    SeniorCitizen
           0        1
    83.75995 16.24005

    Partner
          No      Yes
    51.74915 48.25085

    SeniorCitizen
          No      Yes
    70.15074 29.84926

    Dependents
          No      Yes
     9.67008 90.32992

    MultipleLines
                  No No phone service              Yes
            57.80717          0.00000         42.19283

    InternetService
            DSL Fiber optic          No
       34.35722    44.02730    21.61547


    OnlineSecurity
                     No No internet service                 Yes
               49.72981            21.61547            28.65472


    OnlineBackup
                     No No internet service                 Yes
               43.89932            21.61547            34.48521

    DeviceProtection
                     No No internet service                 Yes
               43.99886            21.61547            34.38567

    TechSupport
                     No No internet service                 Yes
               49.37429            21.61547            29.01024

    StreamingTV
                     No No internet service                 Yes
               39.94596            21.61547            38.43857

    STreaming Movies
                     No No internet service                 Yes
               39.54778            21.61547            38.83675

    Contract
    Month-to-month       One year       Two year
          55.10523       20.93288       23.96189

    PaperlessBilling
         No     Yes
    40.7281 59.2719

    PaymentMethod
    Bank transfer (automatic)   Credit card (automatic)          Electronic check
                     21.92833                  21.62969                  33.63197
                 Mailed check
                     22.81001

    Churn
         No     Yes
    73.4215 26.5785



```R
#Dependent variable Churn vs Tenure_bin
ggplot(my_data, aes(x = tenure_bin , y = MonthlyCharges)) + geom_point(aes(colour=factor(Churn)))

#Dependent variable Churn vs
ggplot(my_data, aes(x = MonthlyCharges , y = TotalCharges)) + geom_point(aes(colour=factor(Churn)))
```
Output:
![png](/images/churn/Telco%20Churn%20Analysis_23_0.png)

![png](/images/churn/Telco%20Churn%20Analysis_23_1.png)


<b>1. Maximum Customers churned from the platform are the one having a tenure of 0-1 years.<br>
<b>2. Maximum Churned customers have a Monthly charge more than $65.

**Relationship between the variables (dependent variable Churn vs Categorical Variables)**
```R
#gender vs churn
plot_relation_1 =ggplot(my_data, aes(x = gender,fill=Churn)) + geom_bar(width = 0.4)
#SeniorCitizen vs Churn
plot_relation_2 =ggplot(my_data, aes(x = SeniorCitizen,fill=Churn)) + geom_bar(width = 0.4)
#partner vs Churn
plot_relation_3 =ggplot(my_data, aes(x = Partner,fill=Churn)) + geom_bar(width = 0.4)
#Dependents vs Churn
plot_relation_4 =ggplot(my_data, aes(x = Dependents,fill=Churn)) + geom_bar(width = 0.4)
#phoneservices vs Churn
plot_relation_5 =ggplot(my_data, aes(x = PhoneService,fill=Churn)) + geom_bar(width = 0.4)
#MultipleLines vs Churn
plot_relation_6 =ggplot(my_data, aes(x = MultipleLines,fill=Churn)) + geom_bar(width = 0.4)
#InternetServices vs Churn
plot_relation_7 =ggplot(my_data, aes(x = InternetService,fill=Churn)) + geom_bar(width = 0.4)
#OnlineSecurity vs Churn
plot_relation_8 =ggplot(my_data, aes(x = OnlineSecurity,fill=Churn)) + geom_bar(width = 0.4)
#Onlinebackup vs Churn
plot_relation_9 =ggplot(my_data, aes(x = OnlineBackup,fill=Churn)) + geom_bar(width = 0.4)
#DeviceProtection vs Churn
plot_relation_10 =ggplot(my_data, aes(x = DeviceProtection,fill=Churn)) + geom_bar(width = 0.4)
#TechSupport vs Churn
plot_relation_11 =ggplot(my_data, aes(x = TechSupport,fill=Churn)) + geom_bar(width = 0.4)
#StreamingTV vs Churn
plot_relation_12 =ggplot(my_data, aes(x = StreamingTV,fill=Churn)) + geom_bar(width = 0.4)
#StreamingMovies vs Churn
plot_relation_13 =ggplot(my_data, aes(x = StreamingMovies,fill=Churn)) + geom_bar(width = 0.4)
#Contract vs Churn
plot_relation_14 =ggplot(my_data, aes(x = Contract,fill=Churn)) + geom_bar(width = 0.4)
#PaperlessBilling vs Churn
plot_relation_15 =ggplot(my_data, aes(x = PaperlessBilling,fill=Churn)) + geom_bar(width = 0.4)
#PaymentMethod vs Churn
plot_relation_16 =ggplot(my_data, aes(x = PaymentMethod,fill=Churn)) + geom_bar(width = 0.4)
```


```R
grid.arrange(plot_relation_1,plot_relation_2,plot_relation_3,plot_relation_4,plot_relation_5,plot_relation_6,plot_relation_15,plot_relation_16)
```

Output:
![png](/images/churn/Telco%20Churn%20Analysis_26_0.png)


<b>* Churn rate is equally divided among the male and female customers.<br>
<b>* Churn rate is more among the customers with no dependents.<br>
<b>* Churn rate is more with customers having phone service.<br>
<b>* Churn rate is more with customers having paperless billing.<br>
<b>* Churn rate is more with customers having electronic check as the payment mode.


```R
grid.arrange(plot_relation_9,plot_relation_10,plot_relation_11,plot_relation_12,plot_relation_7,plot_relation_8)
```

Output:
![png](/images/churn/Telco%20Churn%20Analysis_28_0.png)


<b>* Churn rate is more with customers having month to month contract.<br>
<b>* Churn rate is more with customers having no online security and techsupport.<br>
<b>* Churn rate is almost equal among the subscribers with or without the streamingtv.

```R
#creating dummy variables
my_data_cat <- my_data[,-c(1,3,6,19,20)]
dummy<- data.frame(sapply(my_data_cat,function(x) data.frame(model.matrix(~x-1,data = my_data))[,-1]))
```


```R
my_new_data_set <- cbind(my_data[,c(1,3,6,19,20)],dummy)
```

```R
#now that we dont need the customerID ll be removing it off from the dataset.
my_new_data_set$customerID <- NULL
#removing the tenure from the dataset as we have already transformed the tenure values in tenure_bin.
my_new_data_set$tenure <- NULL
```


```R
#data splliting into test and train into 80 20 ratio.


num = sample(2,nrow(my_new_data_set),replace = T,prob = c(0.7,0.3))

train_data <-  my_new_data_set[num==1,]
test_data  <-  my_new_data_set[num==2,]

```
```R
str(my_new_data_set)

#my_new_data_set$SeniorCitizen = as.numeric(my_new_data_set$SeniorCitizen)
```

    'data.frame':	7032 obs. of  35 variables:
     $ SeniorCitizen                         : Factor w/ 2 levels "0","1": 1 1 1 1 1 1 1 1 1 1 ...
     $ MonthlyCharges                        : num  29.9 57 53.9 42.3 70.7 ...
     $ TotalCharges                          : num  29.9 1889.5 108.2 1840.8 151.7 ...
     $ gender                                : num  0 1 1 1 0 0 1 0 0 1 ...
     $ Partner                               : num  1 0 0 0 0 0 0 0 1 0 ...
     $ Dependents                            : num  0 0 0 0 0 0 1 0 0 1 ...
     $ PhoneService                          : num  0 1 1 0 1 1 1 0 1 1 ...
     $ MultipleLines.xNo.phone.service       : num  0 0 0 0 0 0 0 0 0 0 ...
     $ MultipleLines.xYes                    : num  0 0 0 0 0 1 1 0 1 0 ...
     $ InternetService.xFiber.optic          : num  0 0 0 0 1 1 1 0 1 0 ...
     $ InternetService.xNo                   : num  0 0 0 0 0 0 0 0 0 0 ...
     $ OnlineSecurity.xNo.internet.service   : num  0 0 0 0 0 0 0 0 0 0 ...
     $ OnlineSecurity.xYes                   : num  0 1 1 1 0 0 0 1 0 1 ...
     $ OnlineBackup.xNo.internet.service     : num  0 0 0 0 0 0 0 0 0 0 ...
     $ OnlineBackup.xYes                     : num  1 0 1 0 0 0 1 0 0 1 ...
     $ DeviceProtection.xNo.internet.service : num  0 0 0 0 0 0 0 0 0 0 ...
     $ DeviceProtection.xYes                 : num  0 1 0 1 0 1 0 0 1 0 ...
     $ TechSupport.xNo.internet.service      : num  0 0 0 0 0 0 0 0 0 0 ...
     $ TechSupport.xYes                      : num  0 0 0 1 0 0 0 0 1 0 ...
     $ StreamingTV.xNo.internet.service      : num  0 0 0 0 0 0 0 0 0 0 ...
     $ StreamingTV.xYes                      : num  0 0 0 0 0 1 1 0 1 0 ...
     $ StreamingMovies.xNo.internet.service  : num  0 0 0 0 0 0 0 0 0 0 ...
     $ StreamingMovies.xYes                  : num  0 0 0 0 0 1 0 0 1 0 ...
     $ Contract.xOne.year                    : num  0 1 0 1 0 0 0 0 0 1 ...
     $ Contract.xTwo.year                    : num  0 0 0 0 0 0 0 0 0 0 ...
     $ PaperlessBilling                      : num  1 0 1 0 1 1 1 0 1 0 ...
     $ PaymentMethod.xCredit.card..automatic.: num  0 0 0 0 0 0 1 0 0 0 ...
     $ PaymentMethod.xElectronic.check       : num  1 0 0 0 1 1 0 0 1 0 ...
     $ PaymentMethod.xMailed.check           : num  0 1 1 0 0 0 0 1 0 0 ...
     $ Churn                                 : num  0 0 1 0 1 1 0 0 1 0 ...
     $ tenure_bin.x1...2.years               : num  0 0 0 0 0 0 1 0 0 0 ...
     $ tenure_bin.x2...3.years               : num  0 1 0 0 0 0 0 0 1 0 ...
     $ tenure_bin.x3...4.years               : num  0 0 0 1 0 0 0 0 0 0 ...
     $ tenure_bin.x4...5.years               : num  0 0 0 0 0 0 0 0 0 0 ...
     $ tenure_bin.x5...6.years               : num  0 0 0 0 0 0 0 0 0 1 ...


MODEL PROCESSING

****1 .STEPWISE LOGISTIC REGRESSION****

```R
log_model <- glm(Churn ~ .,family ="binomial", data=train_data)
```

```R
summary(log_model)
```

    Call:
    glm(formula = Churn ~ ., family = "binomial", data = train_data)

    Deviance Residuals:
        Min       1Q   Median       3Q      Max  
    -2.0790  -0.6604  -0.2975   0.6457   3.0386  

    Coefficients: (7 not defined because of singularities)
                                             Estimate Std. Error z value Pr(>|z|)
    (Intercept)                             8.303e-01  9.733e-01   0.853 0.393660
    SeniorCitizen1                          3.017e-01  1.022e-01   2.951 0.003165
    MonthlyCharges                         -4.163e-02  3.826e-02  -1.088 0.276599
    TotalCharges                           -1.282e-04  7.214e-05  -1.777 0.075648
    gender                                 -3.022e-02  7.814e-02  -0.387 0.698921
    Partner                                -4.163e-02  9.310e-02  -0.447 0.654774
    Dependents                             -1.305e-01  1.086e-01  -1.202 0.229557
    PhoneService                            3.275e-01  7.807e-01   0.419 0.674859
    MultipleLines.xNo.phone.service                NA         NA      NA       NA
    MultipleLines.xYes                      5.306e-01  2.141e-01   2.478 0.013199
    InternetService.xFiber.optic            2.068e+00  9.632e-01   2.147 0.031814
    InternetService.xNo                    -1.884e+00  9.715e-01  -1.939 0.052442
    OnlineSecurity.xNo.internet.service            NA         NA      NA       NA
    OnlineSecurity.xYes                    -1.026e-01  2.152e-01  -0.477 0.633470
    OnlineBackup.xNo.internet.service              NA         NA      NA       NA
    OnlineBackup.xYes                       1.800e-02  2.096e-01   0.086 0.931547
    DeviceProtection.xNo.internet.service          NA         NA      NA       NA
    DeviceProtection.xYes                   2.229e-01  2.127e-01   1.048 0.294785
    TechSupport.xNo.internet.service               NA         NA      NA       NA
    TechSupport.xYes                       -8.832e-02  2.173e-01  -0.406 0.684398
    StreamingTV.xNo.internet.service               NA         NA      NA       NA
    StreamingTV.xYes                        6.551e-01  3.938e-01   1.664 0.096192
    StreamingMovies.xNo.internet.service           NA         NA      NA       NA
    StreamingMovies.xYes                    7.940e-01  3.939e-01   2.016 0.043846
    Contract.xOne.year                     -6.819e-01  1.270e-01  -5.370 7.87e-08
    Contract.xTwo.year                     -1.642e+00  2.136e-01  -7.686 1.51e-14
    PaperlessBilling                        2.208e-01  8.870e-02   2.490 0.012787
    PaymentMethod.xCredit.card..automatic. -4.747e-02  1.337e-01  -0.355 0.722580
    PaymentMethod.xElectronic.check         3.325e-01  1.120e-01   2.970 0.002977
    PaymentMethod.xMailed.check            -3.326e-02  1.364e-01  -0.244 0.807383
    tenure_bin.x1...2.years                -9.058e-01  1.271e-01  -7.128 1.02e-12
    tenure_bin.x2...3.years                -1.112e+00  1.761e-01  -6.312 2.76e-10
    tenure_bin.x3...4.years                -7.749e-01  2.339e-01  -3.313 0.000923
    tenure_bin.x4...5.years                -9.613e-01  3.043e-01  -3.159 0.001585
    tenure_bin.x5...6.years                -1.131e+00  4.021e-01  -2.814 0.004898

    (Intercept)                               
    SeniorCitizen1                         **
    MonthlyCharges                            
    TotalCharges                           .  
    gender                                    
    Partner                                   
    Dependents                                
    PhoneService                              
    MultipleLines.xNo.phone.service           
    MultipleLines.xYes                     *  
    InternetService.xFiber.optic           *  
    InternetService.xNo                    .  
    OnlineSecurity.xNo.internet.service       
    OnlineSecurity.xYes                       
    OnlineBackup.xNo.internet.service         
    OnlineBackup.xYes                         
    DeviceProtection.xNo.internet.service     
    DeviceProtection.xYes                     
    TechSupport.xNo.internet.service          
    TechSupport.xYes                          
    StreamingTV.xNo.internet.service          
    StreamingTV.xYes                       .  
    StreamingMovies.xNo.internet.service      
    StreamingMovies.xYes                   *  
    Contract.xOne.year                     ***
    Contract.xTwo.year                     ***
    PaperlessBilling                       *  
    PaymentMethod.xCredit.card..automatic.    
    PaymentMethod.xElectronic.check        **
    PaymentMethod.xMailed.check               
    tenure_bin.x1...2.years                ***
    tenure_bin.x2...3.years                ***
    tenure_bin.x3...4.years                ***
    tenure_bin.x4...5.years                **
    tenure_bin.x5...6.years                **
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

    (Dispersion parameter for binomial family taken to be 1)

        Null deviance: 5626.1  on 4868  degrees of freedom
    Residual deviance: 4057.9  on 4841  degrees of freedom
    AIC: 4113.9

    Number of Fisher Scoring iterations: 6

```R
#stepAIC
step_2 <- stepAIC(log_model,direction = "both")
step_2$anova
```
    Start:  AIC=4113.91
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + gender +
        Partner + Dependents + PhoneService + MultipleLines.xNo.phone.service +
        MultipleLines.xYes + InternetService.xFiber.optic + InternetService.xNo +
        OnlineSecurity.xNo.internet.service + OnlineSecurity.xYes +
        OnlineBackup.xNo.internet.service + OnlineBackup.xYes + DeviceProtection.xNo.internet.service +
        DeviceProtection.xYes + TechSupport.xNo.internet.service +
        TechSupport.xYes + StreamingTV.xNo.internet.service + StreamingTV.xYes +
        StreamingMovies.xNo.internet.service + StreamingMovies.xYes +
        Contract.xOne.year + Contract.xTwo.year + PaperlessBilling +
        PaymentMethod.xCredit.card..automatic. + PaymentMethod.xElectronic.check +
        PaymentMethod.xMailed.check + tenure_bin.x1...2.years + tenure_bin.x2...3.years +
        tenure_bin.x3...4.years + tenure_bin.x4...5.years + tenure_bin.x5...6.years


    Step:  AIC=4113.91
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + gender +
        Partner + Dependents + PhoneService + MultipleLines.xNo.phone.service +
        MultipleLines.xYes + InternetService.xFiber.optic + InternetService.xNo +
        OnlineSecurity.xNo.internet.service + OnlineSecurity.xYes +
        OnlineBackup.xNo.internet.service + OnlineBackup.xYes + DeviceProtection.xNo.internet.service +
        DeviceProtection.xYes + TechSupport.xNo.internet.service +
        TechSupport.xYes + StreamingTV.xNo.internet.service + StreamingTV.xYes +
        StreamingMovies.xYes + Contract.xOne.year + Contract.xTwo.year +
        PaperlessBilling + PaymentMethod.xCredit.card..automatic. +
        PaymentMethod.xElectronic.check + PaymentMethod.xMailed.check +
        tenure_bin.x1...2.years + tenure_bin.x2...3.years + tenure_bin.x3...4.years +
        tenure_bin.x4...5.years + tenure_bin.x5...6.years


    Step:  AIC=4113.91
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + gender +
        Partner + Dependents + PhoneService + MultipleLines.xNo.phone.service +
        MultipleLines.xYes + InternetService.xFiber.optic + InternetService.xNo +
        OnlineSecurity.xNo.internet.service + OnlineSecurity.xYes +
        OnlineBackup.xNo.internet.service + OnlineBackup.xYes + DeviceProtection.xNo.internet.service +
        DeviceProtection.xYes + TechSupport.xNo.internet.service +
        TechSupport.xYes + StreamingTV.xYes + StreamingMovies.xYes +
        Contract.xOne.year + Contract.xTwo.year + PaperlessBilling +
        PaymentMethod.xCredit.card..automatic. + PaymentMethod.xElectronic.check +
        PaymentMethod.xMailed.check + tenure_bin.x1...2.years + tenure_bin.x2...3.years +
        tenure_bin.x3...4.years + tenure_bin.x4...5.years + tenure_bin.x5...6.years


    Step:  AIC=4113.91
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + gender +
        Partner + Dependents + PhoneService + MultipleLines.xNo.phone.service +
        MultipleLines.xYes + InternetService.xFiber.optic + InternetService.xNo +
        OnlineSecurity.xNo.internet.service + OnlineSecurity.xYes +
        OnlineBackup.xNo.internet.service + OnlineBackup.xYes + DeviceProtection.xNo.internet.service +
        DeviceProtection.xYes + TechSupport.xYes + StreamingTV.xYes +
        StreamingMovies.xYes + Contract.xOne.year + Contract.xTwo.year +
        PaperlessBilling + PaymentMethod.xCredit.card..automatic. +
        PaymentMethod.xElectronic.check + PaymentMethod.xMailed.check +
        tenure_bin.x1...2.years + tenure_bin.x2...3.years + tenure_bin.x3...4.years +
        tenure_bin.x4...5.years + tenure_bin.x5...6.years


    Step:  AIC=4113.91
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + gender +
        Partner + Dependents + PhoneService + MultipleLines.xNo.phone.service +
        MultipleLines.xYes + InternetService.xFiber.optic + InternetService.xNo +
        OnlineSecurity.xNo.internet.service + OnlineSecurity.xYes +
        OnlineBackup.xNo.internet.service + OnlineBackup.xYes + DeviceProtection.xYes +
        TechSupport.xYes + StreamingTV.xYes + StreamingMovies.xYes +
        Contract.xOne.year + Contract.xTwo.year + PaperlessBilling +
        PaymentMethod.xCredit.card..automatic. + PaymentMethod.xElectronic.check +
        PaymentMethod.xMailed.check + tenure_bin.x1...2.years + tenure_bin.x2...3.years +
        tenure_bin.x3...4.years + tenure_bin.x4...5.years + tenure_bin.x5...6.years


    Step:  AIC=4113.91
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + gender +
        Partner + Dependents + PhoneService + MultipleLines.xNo.phone.service +
        MultipleLines.xYes + InternetService.xFiber.optic + InternetService.xNo +
        OnlineSecurity.xNo.internet.service + OnlineSecurity.xYes +
        OnlineBackup.xYes + DeviceProtection.xYes + TechSupport.xYes +
        StreamingTV.xYes + StreamingMovies.xYes + Contract.xOne.year +
        Contract.xTwo.year + PaperlessBilling + PaymentMethod.xCredit.card..automatic. +
        PaymentMethod.xElectronic.check + PaymentMethod.xMailed.check +
        tenure_bin.x1...2.years + tenure_bin.x2...3.years + tenure_bin.x3...4.years +
        tenure_bin.x4...5.years + tenure_bin.x5...6.years


    Step:  AIC=4113.91
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + gender +
        Partner + Dependents + PhoneService + MultipleLines.xNo.phone.service +
        MultipleLines.xYes + InternetService.xFiber.optic + InternetService.xNo +
        OnlineSecurity.xYes + OnlineBackup.xYes + DeviceProtection.xYes +
        TechSupport.xYes + StreamingTV.xYes + StreamingMovies.xYes +
        Contract.xOne.year + Contract.xTwo.year + PaperlessBilling +
        PaymentMethod.xCredit.card..automatic. + PaymentMethod.xElectronic.check +
        PaymentMethod.xMailed.check + tenure_bin.x1...2.years + tenure_bin.x2...3.years +
        tenure_bin.x3...4.years + tenure_bin.x4...5.years + tenure_bin.x5...6.years


    Step:  AIC=4113.91
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + gender +
        Partner + Dependents + PhoneService + MultipleLines.xYes +
        InternetService.xFiber.optic + InternetService.xNo + OnlineSecurity.xYes +
        OnlineBackup.xYes + DeviceProtection.xYes + TechSupport.xYes +
        StreamingTV.xYes + StreamingMovies.xYes + Contract.xOne.year +
        Contract.xTwo.year + PaperlessBilling + PaymentMethod.xCredit.card..automatic. +
        PaymentMethod.xElectronic.check + PaymentMethod.xMailed.check +
        tenure_bin.x1...2.years + tenure_bin.x2...3.years + tenure_bin.x3...4.years +
        tenure_bin.x4...5.years + tenure_bin.x5...6.years

                                             Df Deviance    AIC
    - OnlineBackup.xYes                       1   4057.9 4111.9
    - PaymentMethod.xMailed.check             1   4058.0 4112.0
    - PaymentMethod.xCredit.card..automatic.  1   4058.0 4112.0
    - gender                                  1   4058.1 4112.1
    - TechSupport.xYes                        1   4058.1 4112.1
    - PhoneService                            1   4058.1 4112.1
    - Partner                                 1   4058.1 4112.1
    - OnlineSecurity.xYes                     1   4058.1 4112.1
    - DeviceProtection.xYes                   1   4059.0 4113.0
    - MonthlyCharges                          1   4059.1 4113.1
    - Dependents                              1   4059.4 4113.4
    <none>                                        4057.9 4113.9
    - StreamingTV.xYes                        1   4060.7 4114.7
    - TotalCharges                            1   4061.0 4115.0
    - InternetService.xNo                     1   4061.7 4115.7
    - StreamingMovies.xYes                    1   4062.0 4116.0
    - InternetService.xFiber.optic            1   4062.5 4116.5
    - MultipleLines.xYes                      1   4064.1 4118.1
    - PaperlessBilling                        1   4064.1 4118.1
    - tenure_bin.x5...6.years                 1   4066.2 4120.2
    - SeniorCitizen                           1   4066.6 4120.6
    - PaymentMethod.xElectronic.check         1   4066.8 4120.8
    - tenure_bin.x4...5.years                 1   4068.3 4122.3
    - tenure_bin.x3...4.years                 1   4069.3 4123.3
    - Contract.xOne.year                      1   4088.1 4142.1
    - tenure_bin.x2...3.years                 1   4099.9 4153.9
    - tenure_bin.x1...2.years                 1   4110.3 4164.3
    - Contract.xTwo.year                      1   4130.1 4184.1

    Step:  AIC=4111.92
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + gender +
        Partner + Dependents + PhoneService + MultipleLines.xYes +
        InternetService.xFiber.optic + InternetService.xNo + OnlineSecurity.xYes +
        DeviceProtection.xYes + TechSupport.xYes + StreamingTV.xYes +
        StreamingMovies.xYes + Contract.xOne.year + Contract.xTwo.year +
        PaperlessBilling + PaymentMethod.xCredit.card..automatic. +
        PaymentMethod.xElectronic.check + PaymentMethod.xMailed.check +
        tenure_bin.x1...2.years + tenure_bin.x2...3.years + tenure_bin.x3...4.years +
        tenure_bin.x4...5.years + tenure_bin.x5...6.years

                                             Df Deviance    AIC
    - PaymentMethod.xMailed.check             1   4058.0 4110.0
    - PaymentMethod.xCredit.card..automatic.  1   4058.0 4110.0
    - gender                                  1   4058.1 4110.1
    - Partner                                 1   4058.1 4110.1
    - PhoneService                            1   4058.4 4110.4
    - TechSupport.xYes                        1   4058.5 4110.5
    - OnlineSecurity.xYes                     1   4058.7 4110.7
    - Dependents                              1   4059.4 4111.4
    <none>                                        4057.9 4111.9
    - DeviceProtection.xYes                   1   4060.6 4112.6
    - TotalCharges                            1   4061.0 4113.0
    + OnlineBackup.xYes                       1   4057.9 4113.9
    - MonthlyCharges                          1   4063.1 4115.1
    - PaperlessBilling                        1   4064.1 4116.1
    - tenure_bin.x5...6.years                 1   4066.2 4118.2
    - SeniorCitizen                           1   4066.6 4118.6
    - PaymentMethod.xElectronic.check         1   4066.8 4118.8
    - StreamingTV.xYes                        1   4068.3 4120.3
    - tenure_bin.x4...5.years                 1   4068.3 4120.3
    - tenure_bin.x3...4.years                 1   4069.3 4121.3
    - InternetService.xNo                     1   4072.5 4124.5
    - StreamingMovies.xYes                    1   4073.9 4125.9
    - MultipleLines.xYes                      1   4074.0 4126.0
    - InternetService.xFiber.optic            1   4078.9 4130.9
    - Contract.xOne.year                      1   4088.1 4140.1
    - tenure_bin.x2...3.years                 1   4099.9 4151.9
    - tenure_bin.x1...2.years                 1   4110.3 4162.3
    - Contract.xTwo.year                      1   4130.1 4182.1

    Step:  AIC=4109.98
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + gender +
        Partner + Dependents + PhoneService + MultipleLines.xYes +
        InternetService.xFiber.optic + InternetService.xNo + OnlineSecurity.xYes +
        DeviceProtection.xYes + TechSupport.xYes + StreamingTV.xYes +
        StreamingMovies.xYes + Contract.xOne.year + Contract.xTwo.year +
        PaperlessBilling + PaymentMethod.xCredit.card..automatic. +
        PaymentMethod.xElectronic.check + tenure_bin.x1...2.years +
        tenure_bin.x2...3.years + tenure_bin.x3...4.years + tenure_bin.x4...5.years +
        tenure_bin.x5...6.years

                                             Df Deviance    AIC
    - PaymentMethod.xCredit.card..automatic.  1   4058.1 4108.1
    - gender                                  1   4058.1 4108.1
    - Partner                                 1   4058.2 4108.2
    - PhoneService                            1   4058.5 4108.5
    - TechSupport.xYes                        1   4058.6 4108.6
    - OnlineSecurity.xYes                     1   4058.8 4108.8
    - Dependents                              1   4059.4 4109.4
    <none>                                        4058.0 4110.0
    - DeviceProtection.xYes                   1   4060.7 4110.7
    - TotalCharges                            1   4061.1 4111.1
    + PaymentMethod.xMailed.check             1   4057.9 4111.9
    + OnlineBackup.xYes                       1   4058.0 4112.0
    - MonthlyCharges                          1   4063.1 4113.1
    - PaperlessBilling                        1   4064.2 4114.2
    - tenure_bin.x5...6.years                 1   4066.2 4116.2
    - SeniorCitizen                           1   4066.7 4116.7
    - tenure_bin.x4...5.years                 1   4068.3 4118.3
    - StreamingTV.xYes                        1   4068.3 4118.3
    - tenure_bin.x3...4.years                 1   4069.3 4119.3
    - PaymentMethod.xElectronic.check         1   4072.2 4122.2
    - InternetService.xNo                     1   4072.6 4122.6
    - StreamingMovies.xYes                    1   4073.9 4123.9
    - MultipleLines.xYes                      1   4074.1 4124.1
    - InternetService.xFiber.optic            1   4079.0 4129.0
    - Contract.xOne.year                      1   4088.1 4138.1
    - tenure_bin.x2...3.years                 1   4100.1 4150.1
    - tenure_bin.x1...2.years                 1   4110.5 4160.5
    - Contract.xTwo.year                      1   4130.1 4180.1

    Step:  AIC=4108.05
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + gender +
        Partner + Dependents + PhoneService + MultipleLines.xYes +
        InternetService.xFiber.optic + InternetService.xNo + OnlineSecurity.xYes +
        DeviceProtection.xYes + TechSupport.xYes + StreamingTV.xYes +
        StreamingMovies.xYes + Contract.xOne.year + Contract.xTwo.year +
        PaperlessBilling + PaymentMethod.xElectronic.check + tenure_bin.x1...2.years +
        tenure_bin.x2...3.years + tenure_bin.x3...4.years + tenure_bin.x4...5.years +
        tenure_bin.x5...6.years

                                             Df Deviance    AIC
    - gender                                  1   4058.2 4106.2
    - Partner                                 1   4058.2 4106.2
    - PhoneService                            1   4058.6 4106.6
    - TechSupport.xYes                        1   4058.7 4106.7
    - OnlineSecurity.xYes                     1   4058.8 4106.8
    - Dependents                              1   4059.5 4107.5
    <none>                                        4058.1 4108.1
    - DeviceProtection.xYes                   1   4060.7 4108.7
    - TotalCharges                            1   4061.2 4109.2
    + PaymentMethod.xCredit.card..automatic.  1   4058.0 4110.0
    + PaymentMethod.xMailed.check             1   4058.0 4110.0
    + OnlineBackup.xYes                       1   4058.0 4110.0
    - MonthlyCharges                          1   4063.2 4111.2
    - PaperlessBilling                        1   4064.3 4112.3
    - tenure_bin.x5...6.years                 1   4066.4 4114.4
    - SeniorCitizen                           1   4066.7 4114.7
    - StreamingTV.xYes                        1   4068.4 4116.4
    - tenure_bin.x4...5.years                 1   4068.5 4116.5
    - tenure_bin.x3...4.years                 1   4069.5 4117.5
    - InternetService.xNo                     1   4072.6 4120.6
    - StreamingMovies.xYes                    1   4074.0 4122.0
    - MultipleLines.xYes                      1   4074.2 4122.2
    - PaymentMethod.xElectronic.check         1   4076.2 4124.2
    - InternetService.xFiber.optic            1   4079.1 4127.1
    - Contract.xOne.year                      1   4088.4 4136.4
    - tenure_bin.x2...3.years                 1   4100.6 4148.6
    - tenure_bin.x1...2.years                 1   4110.9 4158.9
    - Contract.xTwo.year                      1   4130.4 4178.4

    Step:  AIC=4106.21
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + Partner +
        Dependents + PhoneService + MultipleLines.xYes + InternetService.xFiber.optic +
        InternetService.xNo + OnlineSecurity.xYes + DeviceProtection.xYes +
        TechSupport.xYes + StreamingTV.xYes + StreamingMovies.xYes +
        Contract.xOne.year + Contract.xTwo.year + PaperlessBilling +
        PaymentMethod.xElectronic.check + tenure_bin.x1...2.years +
        tenure_bin.x2...3.years + tenure_bin.x3...4.years + tenure_bin.x4...5.years +
        tenure_bin.x5...6.years

                                             Df Deviance    AIC
    - Partner                                 1   4058.4 4104.4
    - PhoneService                            1   4058.7 4104.7
    - TechSupport.xYes                        1   4058.8 4104.8
    - OnlineSecurity.xYes                     1   4059.0 4105.0
    - Dependents                              1   4059.7 4105.7
    <none>                                        4058.2 4106.2
    - DeviceProtection.xYes                   1   4060.9 4106.9
    - TotalCharges                            1   4061.3 4107.3
    + gender                                  1   4058.1 4108.1
    + PaymentMethod.xCredit.card..automatic.  1   4058.1 4108.1
    + OnlineBackup.xYes                       1   4058.2 4108.2
    + PaymentMethod.xMailed.check             1   4058.2 4108.2
    - MonthlyCharges                          1   4063.3 4109.3
    - PaperlessBilling                        1   4064.4 4110.4
    - tenure_bin.x5...6.years                 1   4066.6 4112.6
    - SeniorCitizen                           1   4066.9 4112.9
    - StreamingTV.xYes                        1   4068.6 4114.6
    - tenure_bin.x4...5.years                 1   4068.7 4114.7
    - tenure_bin.x3...4.years                 1   4069.7 4115.7
    - InternetService.xNo                     1   4072.7 4118.7
    - StreamingMovies.xYes                    1   4074.2 4120.2
    - MultipleLines.xYes                      1   4074.3 4120.3
    - PaymentMethod.xElectronic.check         1   4076.3 4122.3
    - InternetService.xFiber.optic            1   4079.2 4125.2
    - Contract.xOne.year                      1   4088.6 4134.6
    - tenure_bin.x2...3.years                 1   4100.8 4146.8
    - tenure_bin.x1...2.years                 1   4111.2 4157.2
    - Contract.xTwo.year                      1   4130.5 4176.5

    Step:  AIC=4104.39
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + Dependents +
        PhoneService + MultipleLines.xYes + InternetService.xFiber.optic +
        InternetService.xNo + OnlineSecurity.xYes + DeviceProtection.xYes +
        TechSupport.xYes + StreamingTV.xYes + StreamingMovies.xYes +
        Contract.xOne.year + Contract.xTwo.year + PaperlessBilling +
        PaymentMethod.xElectronic.check + tenure_bin.x1...2.years +
        tenure_bin.x2...3.years + tenure_bin.x3...4.years + tenure_bin.x4...5.years +
        tenure_bin.x5...6.years

                                             Df Deviance    AIC
    - PhoneService                            1   4058.9 4102.9
    - TechSupport.xYes                        1   4059.0 4103.0
    - OnlineSecurity.xYes                     1   4059.2 4103.2
    <none>                                        4058.4 4104.4
    - Dependents                              1   4060.8 4104.8
    - DeviceProtection.xYes                   1   4061.1 4105.1
    - TotalCharges                            1   4061.6 4105.6
    + Partner                                 1   4058.2 4106.2
    + gender                                  1   4058.2 4106.2
    + PaymentMethod.xCredit.card..automatic.  1   4058.3 4106.3
    + OnlineBackup.xYes                       1   4058.4 4106.4
    + PaymentMethod.xMailed.check             1   4058.4 4106.4
    - MonthlyCharges                          1   4063.5 4107.5
    - PaperlessBilling                        1   4064.6 4108.6
    - SeniorCitizen                           1   4066.9 4110.9
    - tenure_bin.x5...6.years                 1   4066.9 4110.9
    - StreamingTV.xYes                        1   4068.7 4112.7
    - tenure_bin.x4...5.years                 1   4069.1 4113.1
    - tenure_bin.x3...4.years                 1   4070.0 4114.0
    - InternetService.xNo                     1   4072.9 4116.9
    - StreamingMovies.xYes                    1   4074.3 4118.3
    - MultipleLines.xYes                      1   4074.4 4118.4
    - PaymentMethod.xElectronic.check         1   4076.4 4120.4
    - InternetService.xFiber.optic            1   4079.3 4123.3
    - Contract.xOne.year                      1   4088.9 4132.9
    - tenure_bin.x2...3.years                 1   4101.5 4145.5
    - tenure_bin.x1...2.years                 1   4111.9 4155.9
    - Contract.xTwo.year                      1   4130.8 4174.8

    Step:  AIC=4102.92
    Churn ~ SeniorCitizen + MonthlyCharges + TotalCharges + Dependents +
        MultipleLines.xYes + InternetService.xFiber.optic + InternetService.xNo +
        OnlineSecurity.xYes + DeviceProtection.xYes + TechSupport.xYes +
        StreamingTV.xYes + StreamingMovies.xYes + Contract.xOne.year +
        Contract.xTwo.year + PaperlessBilling + PaymentMethod.xElectronic.check +
        tenure_bin.x1...2.years + tenure_bin.x2...3.years + tenure_bin.x3...4.years +
        tenure_bin.x4...5.years + tenure_bin.x5...6.years

                                             Df Deviance    AIC
    <none>                                        4058.9 4102.9
    - TechSupport.xYes                        1   4061.0 4103.0
    - DeviceProtection.xYes                   1   4061.2 4103.2
    - Dependents                              1   4061.3 4103.3
    - OnlineSecurity.xYes                     1   4061.5 4103.5
    - TotalCharges                            1   4062.3 4104.3
    + PhoneService                            1   4058.4 4104.4
    + OnlineBackup.xYes                       1   4058.6 4104.6
    + Partner                                 1   4058.7 4104.7
    + gender                                  1   4058.8 4104.8
    + PaymentMethod.xCredit.card..automatic.  1   4058.8 4104.8
    + PaymentMethod.xMailed.check             1   4058.9 4104.9
    - PaperlessBilling                        1   4065.0 4107.0
    - SeniorCitizen                           1   4067.3 4109.3
    - tenure_bin.x5...6.years                 1   4067.6 4109.6
    - tenure_bin.x4...5.years                 1   4069.7 4111.7
    - tenure_bin.x3...4.years                 1   4070.7 4112.7
    - MonthlyCharges                          1   4073.2 4115.2
    - PaymentMethod.xElectronic.check         1   4076.8 4118.8
    - MultipleLines.xYes                      1   4077.4 4119.4
    - StreamingTV.xYes                        1   4078.1 4120.1
    - Contract.xOne.year                      1   4089.3 4131.3
    - StreamingMovies.xYes                    1   4090.8 4132.8
    - tenure_bin.x2...3.years                 1   4102.4 4144.4
    - InternetService.xNo                     1   4110.3 4152.3
    - InternetService.xFiber.optic            1   4111.0 4153.0
    - tenure_bin.x1...2.years                 1   4112.5 4154.5
    - Contract.xTwo.year                      1   4131.0 4173.0

```R
log_model <- glm(Churn ~ SeniorCitizen + TotalCharges + PhoneService + MultipleLines.xYes +
    InternetService.xFiber.optic + InternetService.xNo + OnlineSecurity.xYes +
    OnlineBackup.xYes + TechSupport.xYes + StreamingTV.xYes +
    StreamingMovies.xYes + Contract.xOne.year + Contract.xTwo.year +
    PaperlessBilling + PaymentMethod.xElectronic.check + tenure_bin.x1...2.years +
    tenure_bin.x2...3.years + tenure_bin.x3...4.years + tenure_bin.x4...5.years +
    tenure_bin.x5...6.years,family ="binomial", data=train_data)
```


```R
summary(log_model)
```
    Call:
    glm(formula = Churn ~ SeniorCitizen + TotalCharges + PhoneService +
        MultipleLines.xYes + InternetService.xFiber.optic + InternetService.xNo +
        OnlineSecurity.xYes + OnlineBackup.xYes + TechSupport.xYes +
        StreamingTV.xYes + StreamingMovies.xYes + Contract.xOne.year +
        Contract.xTwo.year + PaperlessBilling + PaymentMethod.xElectronic.check +
        tenure_bin.x1...2.years + tenure_bin.x2...3.years + tenure_bin.x3...4.years +
        tenure_bin.x4...5.years + tenure_bin.x5...6.years, family = "binomial",
        data = train_data)

    Deviance Residuals:
        Min       1Q   Median       3Q      Max  
    -2.0817  -0.6552  -0.2976   0.6475   3.0417  

    Coefficients:
                                      Estimate Std. Error z value Pr(>|z|)    
    (Intercept)                     -2.889e-01  1.548e-01  -1.867 0.061968 .  
    SeniorCitizen1                   3.198e-01  1.003e-01   3.187 0.001438 **
    TotalCharges                    -1.293e-04  7.057e-05  -1.832 0.066922 .  
    PhoneService                    -5.073e-01  1.553e-01  -3.266 0.001091 **
    MultipleLines.xYes               3.232e-01  9.695e-02   3.334 0.000857 ***
    InternetService.xFiber.optic     1.034e+00  1.177e-01   8.786  < 2e-16 ***
    InternetService.xNo             -8.551e-01  1.654e-01  -5.170 2.34e-07 ***
    OnlineSecurity.xYes             -3.152e-01  1.019e-01  -3.092 0.001986 **
    OnlineBackup.xYes               -1.898e-01  9.335e-02  -2.033 0.042034 *  
    TechSupport.xYes                -2.954e-01  1.024e-01  -2.885 0.003913 **
    StreamingTV.xYes                 2.395e-01  9.792e-02   2.445 0.014470 *  
    StreamingMovies.xYes             3.836e-01  9.820e-02   3.907 9.36e-05 ***
    Contract.xOne.year              -6.916e-01  1.260e-01  -5.488 4.06e-08 ***
    Contract.xTwo.year              -1.653e+00  2.123e-01  -7.788 6.80e-15 ***
    PaperlessBilling                 2.244e-01  8.853e-02   2.534 0.011261 *  
    PaymentMethod.xElectronic.check  3.604e-01  8.372e-02   4.305 1.67e-05 ***
    tenure_bin.x1...2.years         -9.155e-01  1.257e-01  -7.282 3.30e-13 ***
    tenure_bin.x2...3.years         -1.129e+00  1.739e-01  -6.489 8.64e-11 ***
    tenure_bin.x3...4.years         -7.881e-01  2.312e-01  -3.408 0.000654 ***
    tenure_bin.x4...5.years         -9.723e-01  2.993e-01  -3.248 0.001161 **
    tenure_bin.x5...6.years         -1.152e+00  3.970e-01  -2.901 0.003719 **
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

    (Dispersion parameter for binomial family taken to be 1)

        Null deviance: 5626.1  on 4868  degrees of freedom
    Residual deviance: 4062.0  on 4848  degrees of freedom
    AIC: 4104

    Number of Fisher Scoring iterations: 6


```R
predict_log_model <- predict(log_model,newdata=test_data)
```

```R
y_pred_num <- ifelse(predict_log_model > 0.5, 1, 0)
```

```R
y_predicted <- factor(y_pred_num, levels=c(0, 1))
```

```R
y_observed <- test_data$Churn
```

**Evaluate prediction model by calculating accuracy (proportion of y_predicted that match y_observed)**
```R

Accuracy_LM <- mean(y_observed == y_predicted)
roc_lm <- roc(test_data$Churn,predict_log_model,plot=TRUE)
```

    Setting levels: control = 0, case = 1

    Setting direction: controls < cases


Output:
![png](/images/churn/Telco%20Churn%20Analysis_47_1.png)

<b>With the above Logistic regression kmodel we could see an accuracy of 78% in the model.

```R
table(test_data$Churn)
```
       0    1
    1582  581

```R
LM_confusionMatrix <- confusionMatrix(y_predicted,as.factor(y_observed))
```

```R
LM_confusionMatrix
```
    Confusion Matrix and Statistics

              Reference
    Prediction    0    1
             0 1503  375
             1   79  206

                   Accuracy : 0.7901          
                     95% CI : (0.7723, 0.8071)
        No Information Rate : 0.7314          
        P-Value [Acc > NIR] : 1.561e-10       

                      Kappa : 0.3632          

     Mcnemar's Test P-Value : < 2.2e-16       

                Sensitivity : 0.9501          
                Specificity : 0.3546          
             Pos Pred Value : 0.8003          
             Neg Pred Value : 0.7228          
                 Prevalence : 0.7314          
             Detection Rate : 0.6949          
       Detection Prevalence : 0.8682          
          Balanced Accuracy : 0.6523          

           'Positive' Class : 0               

```R
fourfoldplot(LM_confusionMatrix$table,main = "Confusion Matrix Logistic Regression")
```

Output:
![png](/images/churn/Telco%20Churn%20Analysis_52_0.png)


****2.DECISION TREE****

```R
train_data$Churn <- as.factor(train_data$Churn)

test_data$Churn <- as.factor(test_data$Churn)

options(repr.plot.width = 10, repr.plot.height = 8)

decision_model <- rpart(Churn ~ ., data=train_data,method="class")

rpart.plot(decision_model,type = 4,extra=101)

```
Output:
![png](/images/churn/Telco%20Churn%20Analysis_54_0.png)

```R
predict_decision_model <- predict(decision_model,newdata=test_data,type='class')

table(predict_decision_model,test_data$Churn)

DT_confusionMatrix <- confusionMatrix(predict_decision_model,as.factor(test_data$Churn))
```
    predict_decision_model    0    1
                         0 1474  340
                         1  108  241

```R
pred <- predict(decision_model,test_data,type = 'prob')

roc_dm <- roc(test_data$Churn,pred[,2])

Accuracy_DT <- mean(y_observed == predict_decision_model)
```

    Setting levels: control = 0, case = 1

    Setting direction: controls < cases

```R
fourfoldplot(DT_confusionMatrix$table,main="Confusion Matrix Decision Tree Model")
```

Output:
![png](/images/churn/Telco%20Churn%20Analysis_57_0.png)

```R
DT_confusionMatrix
```
Output:
    Confusion Matrix and Statistics

              Reference
    Prediction    0    1
             0 1474  340
             1  108  241

                   Accuracy : 0.7929          
                     95% CI : (0.7752, 0.8098)
        No Information Rate : 0.7314          
        P-Value [Acc > NIR] : 2.051e-11       

                      Kappa : 0.3966          

     Mcnemar's Test P-Value : < 2.2e-16       

                Sensitivity : 0.9317          
                Specificity : 0.4148          
             Pos Pred Value : 0.8126          
             Neg Pred Value : 0.6905          
                 Prevalence : 0.7314          
             Detection Rate : 0.6815          
       Detection Prevalence : 0.8387          
          Balanced Accuracy : 0.6733          

           'Positive' Class : 0               

***3. Random Forest Modeling***

```R
forest_model <- randomForest(Churn ~ .,data = train_data,proximity=TRUE)
```

```R
predict_forest_model <- predict(forest_model,newdata=test_data,type ="prob" )

predict_forest_model_cm<- predict(forest_model,newdata=test_data)

table(predict_forest_model_cm,test_data$Churn)
```
    predict_forest_model_cm    0    1
                          0 1439  288
                          1  143  293

```R
RF_confusionMatrix <- confusionMatrix(predict_forest_model_cm,as.factor(test_data$Churn))

roc_rfm <- roc(test_data$Churn,predict_forest_model[,2])

Accuracy_RF <-  mean(y_observed == predict_forest_model_cm)
```

    Setting levels: control = 0, case = 1

    Setting direction: controls < cases

```R
fourfoldplot(RF_confusionMatrix$table,main = "Confusion Matrix for Random Forest Model")
```
Output:
![png](/images/churn/Telco%20Churn%20Analysis_63_0.png)

```R
RF_confusionMatrix
```
Output:
    Confusion Matrix and Statistics

              Reference
    Prediction    0    1
             0 1439  288
             1  143  293

                   Accuracy : 0.8007          
                     95% CI : (0.7833, 0.8174)
        No Information Rate : 0.7314          
        P-Value [Acc > NIR] : 3.789e-14       

                      Kappa : 0.4494          

     Mcnemar's Test P-Value : 4.027e-12       

                Sensitivity : 0.9096          
                Specificity : 0.5043          
             Pos Pred Value : 0.8332          
             Neg Pred Value : 0.6720          
                 Prevalence : 0.7314          
             Detection Rate : 0.6653          
       Detection Prevalence : 0.7984          
          Balanced Accuracy : 0.7070          

           'Positive' Class : 0               

***Accuracy Comparison for the three Models***

```R
cat("Accuracy for Logistic Model" ,Accuracy_LM)
cat("\nAccuracy for Decision Tree Model",Accuracy_DT)
cat("\nAccuracy for Random Forest Model",Accuracy_RF)
```

    Accuracy for Logistic Model 0.7901063
    Accuracy for Decision Tree Model 0.7928803
    Accuracy for Random Forest Model 0.8007397

**ROC analysis for the three Models**

```R
#First curve:
plot(roc_rfm, col = "red", lty = 2, main = "ROC analysis for Random Forest Model-Logistic Model-Decision Tree Model")
# to add to the same graph: add=TRUE
plot(roc_lm, col = "green", lty = 3, add = TRUE)
plot(roc_dm, col = "black", lty = 10, add = TRUE)
legend(0.6,0.45, c('Random Forest','Logistic','Decision'),lty=c(1,1),lwd=c(2,2),col=c('red','green','black'))
```

Output:
![png](/images/churn/Telco%20Churn%20Analysis_68_0.png)

***BUSINESS COST ASSUMPTION :***

```R
#If no predictive model is implemented by the company
Total_Cost_No_model <- length(test_data$Churn[test_data$Churn == 1]) * 500

#Logistic regression
LM_acuracy <- data.frame(LM_confusionMatrix$table)
TP_LM <- LM_acuracy$Freq[LM_acuracy$Prediction == 0 & LM_acuracy$Reference == 0]
TN_LM <- LM_acuracy$Freq[LM_acuracy$Prediction == 1 & LM_acuracy$Reference == 1]
FN_LM <- LM_acuracy$Freq[LM_acuracy$Prediction == 0 & LM_acuracy$Reference == 1]
FP_LM <- LM_acuracy$Freq[LM_acuracy$Prediction == 1 & LM_acuracy$Reference == 0]

Total_Cost_LM <- FN_LM * 500 + TN_LM * 100 + FP_LM * 100

#Decision tree
DT_acuracy <- data.frame(DT_confusionMatrix$table)
TP_DT <- DT_acuracy$Freq[DT_acuracy$Prediction == 0 & DT_acuracy$Reference == 0]
TN_DT <- DT_acuracy$Freq[DT_acuracy$Prediction == 1 & DT_acuracy$Reference == 1]
FN_DT <- DT_acuracy$Freq[DT_acuracy$Prediction == 0 & DT_acuracy$Reference == 1]
FP_DT <- DT_acuracy$Freq[DT_acuracy$Prediction == 1 & DT_acuracy$Reference == 0]

Total_Cost_DT <- FN_DT * 500 + TN_DT * 100 + FP_DT * 100

#Random Forest
RF_acuracy <- data.frame(RF_confusionMatrix$table)
TP_RF <- RF_acuracy$Freq[RF_acuracy$Prediction == 0 & RF_acuracy$Reference == 0]
TN_RF <- RF_acuracy$Freq[RF_acuracy$Prediction == 1 & RF_acuracy$Reference == 1]
FN_RF <- RF_acuracy$Freq[RF_acuracy$Prediction == 0 & RF_acuracy$Reference == 1]
FP_RF <- RF_acuracy$Freq[RF_acuracy$Prediction == 1 & RF_acuracy$Reference == 0]

Total_Cost_RF <- FN_RF * 500 + TN_RF * 100 + FP_RF * 100

```

```R
Total_Cost_No_model
```
->290500



```R
x <- c("Cost_logitModel","Cost_DecisionTree","Cost_RandomForest","Cost_No_model")
y <- c(Total_Cost_LM,Total_Cost_DT,Total_Cost_RF,Total_Cost_No_model)

model_cost <- data.frame("Model" = x,"Cost" = y)

ggplot(model_cost) +
geom_bar(aes(x=Model,y=Cost),stat ="identity",fill="steelblue") +
#coord_flip()+
xlab("Model Name")+
ylab("Model Cost")+
labs(title = "Business Cost Assumptions for all Models")

```

Output:
![png](/images/churn/Telco%20Churn%20Analysis_72_0.png)


***Findings***

<b>1.We successfully implemented three classification models in order to predict the potential churn customers. Below ROC plot compares the results of the three models , the ROC curve for Logistic model and the Random Forest model can be considered to be “good” and further comparing it with the aforementioned business cost assumptions Random Forest Model can be considered to be a best fit model.<br>
<b>2.Based upon the descriptive analysis , a greater number of Churn customers are observed in case of customers with Tenure in between 0-1 year and with increase in the Monthly Charges there is substantial increase in the Churn Rate.<br>
<b>3.Customer on Month to Month contract are more susceptible to Churn.<br>
<b>4.22% of the total customers were observed with No Internet Service and thus customers are missing the exclusive services of OnlineBackup, StreamingTV, Online Security and Device Protection.
