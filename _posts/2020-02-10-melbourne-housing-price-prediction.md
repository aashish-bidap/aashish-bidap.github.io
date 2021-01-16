**Predicting Housing Prices in Melbourne and Understanding Trends in the Melbourne Housing Data.**

**1. Dependent Variable - Price**

The dataset includes data for the below mentioned paramters for the houses in the Melbourne city.

Suburb, Address,Rooms, Price, Method, Type, SellerG, Date, Distance, Regionname, Propertycount, Bedroom2, Bathroom, Car, Landsize, BuildingArea, YearBuilt, CouncilArea, Lattitude, Longtitude

Analysis :

• Data cleaning with variable research to ensure meaningful and analysable data for modelling.<br>
• Exploratory Data Analysis to analyse trends in the housing data.
<a href="https://public.tableau.com/profile/ashishbidap#!/vizhome/Melbourne_housing/Story1">Exploratory Data Analysis</a><br>
• Implemented Linear regression, Decision tree and Random forest regressor models to predict the target feature price.<br>
• Efficient features selection using stepwise selection and lasso regression.<br>
• Random Forest regressor was the best fit model with efficient R square score.<br>

```R
## Importing packages
```
```R
#installing required packages
install.packages("corrplot")
install.packages("rpart.plot")
install.packages("tree")
install.packages("Metrics")
install.packages("randomForest")
install.packages("dummies")
install.packages("ggcorrplot")
install.packages("VIF")
install.packages("car")
install.packages("corrplot")
install.packages("gvlma")
install.packages("MASS")
install.packages("dplyr")
install.packages("stringr")
install.packages("metrics")
install.packages("glmnet")
install.packages("lava")
```
```R
library(glmnet)
library(lava)
library(stringr)
library(dplyr)
library(tidyr)
library(dummies)
library(ggcorrplot)
library(corrplot)
library(VIF)
library(MASS)
library(car)
library(rpart.plot)
library(tree)
library(Metrics)
library(randomForest)
library(gvlma)
library(dplyr)
library(tidyr)
```
```R
options(scipen=999)  #removing scientific notations.
options(max.print=1000000) #maximum print
```


```R
#data input
mel <- read.csv("../input//images/melbourne-housing-market///images/melbourne_housing_FULL.csv")
```


```R
#Exploring the data
my_missing_NA_value_function <- function(dataset){

  Total_NA <- sum(is.na(dataset))
  Column_sums <- colSums(is.na(dataset))
  cat("Total NA in the dataset in all in the columns- \n\n",Total_NA)
  cat("\n--------------##-----------------")
  Column_names <- colnames(dataset)[apply(dataset,2,anyNA)]
  cat('\n\n Names of NA columns in the dataset-\n\n',Column_names)
  cat('\n\n Total NA by column in the dataset-\n\n',Column_sums)
  cat("\n--------------##-----------------")
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

my_data_overview(mel)
my_missing_NA_value_function(mel)
```
Output:

    Total Number of [rows vs columns] in the dataset-
     34857 21
    --------------##-----------------

     Datatypes of all the columns in the dataset-
     factor factor integer factor integer factor factor factor factor factor integer integer integer integer numeric integer factor numeric numeric factor factor
    --------------##-----------------

     Names of all the columns in the dataset-
     Suburb Address Rooms Type Price Method SellerG Date Distance Postcode Bedroom2 Bathroom Car Landsize BuildingArea YearBuilt CouncilArea Lattitude Longtitude Regionname PropertycountTotal NA in the dataset in all in the columns-

     100964
    --------------##-----------------

     Names of NA columns in the dataset-

     Price Bedroom2 Bathroom Car Landsize BuildingArea YearBuilt Lattitude Longtitude

     Total NA by column in the dataset-

     0 0 0 0 7610 0 0 0 0 0 8217 8226 8728 11810 21115 19306 0 7976 7976 0 0
    --------------##-----------------

```R
########################## Correction of Data types ########################
#checking datatypes of all column in dataset
mel$Address <- as.character(mel$Address)    #changing dataset from factor to character
mel$Rooms <- as.factor(mel$Rooms)   #changing dataset from integer to factor
mel$Distance<- as.integer(mel$Distance)   #changing dataset from factor to integer
mel$Bathroom <- as.factor(mel$Bathroom)    #changing dataset from integer to factor
mel$Car <- as.numeric(mel$Car)            #changing dataset from integer to factor

```
```R
######################### NA's Count in each column ########################
colSums(is.na(mel))

#Remove NA values of Price as its dependent variable
mel1 <- subset(mel,(!is.na(mel[,5])))
```

```R
#Remove BuildingArea Column as it consist more then 60% of NA values
mel2 <- mel1[,c(1:14,16:21)]

```
```R
mel4 <- mel2

#73% of the data for the rooms and Bedrooms is same i.e example if rooms==2 then bedroom2 ==2
temp <- mel4[,c("Rooms","Bedroom2")]
bedroom2 <- temp[which(temp$Rooms == temp$Bedroom2),]

#thus assigning the NA's of Bedrooms with the values of rooms.
my.na <- is.na(mel4$Bedroom2)
mel4$Bedroom2[my.na] <- mel4$Rooms[my.na]

```

```R
########################## Outliers ############################

#Checking the outliers

boxplot(mel4$Price ~ mel4$Regionname, horizontal = TRUE, ylab = "REGION NAME", xlab = "PRICE OF HOUSE", main = "BOXPLOT: PRICE OF HOUSE BY REGION", las = 1)
#par(mar=c(3.1,12,0.95,2.1), mgp = c(11, 1, 0), mfrow = c(2,1))

boxplot(mel4$Landsize ~ mel4$Regionname, horizontal = TRUE, xlab = "Landsize of houses", ylab = "Region Name", main = "BOXPLOT OF LANDSIZE OF HOUSES BY REGION", las = 1)

boxplot(mel4$Distance ~ mel4$Type, horizontal = TRUE, ylab = "Type of House", xlab = "Distance from CBD", main = "Boxplot of distance from CBD vs type of houses", las = 1)

#Checking outliers based on summary
boxplot(mel4$Price,main = "PRICE OF HOUSES.")
mel4$Rooms <- as.integer(mel4$Rooms)
boxplot(mel4$Rooms,main = "NUMBER OF ROOMS.")
boxplot(mel4$Bedroom2,main = "NUMBER OF BEDROOMS.")
boxplot(mel4$Landsize,main="LANDSIZE OF HOUSES.")

#Removing outlier from Rooms column
outliers <- boxplot(mel4$Rooms, plot=FALSE)$out
mel4[which(mel4$Rooms %in% outliers),]
mel4 <- mel4[-which(mel4$Rooms %in% outliers),]
boxplot(mel4$Rooms)

#Removing outlier from Bedroom2 column
outliers <- boxplot(mel4$Bedroom2, plot=FALSE)$out
mel4[which(mel4$Bedroom2 %in% outliers),]
mel4 <- mel4[-which(mel4$Bedroom2 %in% outliers),]
boxplot(mel4$Bedroom2)

#Removing outlier from Landsize column
outliers <- boxplot(mel4$Landsize, plot=FALSE)$out
mel4[which(mel4$Landsize %in% outliers),]
mel4 <- mel4[-which(mel4$Landsize %in% outliers),]
boxplot(mel4$Landsize)

```


![png](/images/melbourne/melbourne-housing-price-prediction_11_0.png)



![png](/images/melbourne/melbourne-housing-price-prediction_11_1.png)



![png](/images/melbourne/melbourne-housing-price-prediction_11_2.png)



![png](/images/melbourne/melbourne-housing-price-prediction_11_3.png)



![png](/images/melbourne/melbourne-housing-price-prediction_11_4.png)



![png](/images/melbourne/melbourne-housing-price-prediction_11_5.png)


![png](/images/melbourne/melbourne-housing-price-prediction_11_7.png)


![png](/images/melbourne/melbourne-housing-price-prediction_11_9.png)


```R
#landsize column fixation
#Making new dataframe of Bedrooms & Landsize
bed.land.df <- mel4[,c("Bedroom2","Landsize")]
unique(bed.land.df$Bedroom2)
colSums(is.na(bed.land.df))
bed.land.df <- na.omit(bed.land.df)
bed.land.df <- bed.land.df[which(bed.land.df$Landsize > 0),]

colSums(is.na(bed.land.df))

bed.land.df_0 <- bed.land.df[which(bed.land.df$Bedroom2 == 0),]
bed.land.df_1 <-  bed.land.df[which(bed.land.df$Bedroom2 == 1),]
bed.land.df_2 <- bed.land.df[which(bed.land.df$Bedroom2 == 2),]
bed.land.df_3 <-  bed.land.df[which(bed.land.df$Bedroom2 == 3),]
bed.land.df_4 <- bed.land.df[which(bed.land.df$Bedroom2 == 4),]
bed.land.df_5 <-  bed.land.df[which(bed.land.df$Bedroom2 == 5),]
bed.land.df_6 <- bed.land.df[which(bed.land.df$Bedroom2 == 6),]
bed.land.df_7 <-  bed.land.df[which(bed.land.df$Bedroom2 == 7),]

#Replacing Na values with 0
mel4$Landsize[which(is.na(mel4$Landsize))] <- 0

#120 logic is used here under the assumption that minimun sq feet required is
#120 mtrs according to the https://www.godownsize.com/minimum-house-square-footage/

mel4$Landsize[which(mel4$Landsize < 120)] <- 0

```


```R
#Replacing 0 values with median values
mel4$Landsize[which(mel4$Bedroom2 == 0 & mel4$Landsize== 0)] <- median(bed.land.df_0$Landsize[which(bed.land.df_0$Landsize > 1)])
mel4$Landsize[which(mel4$Bedroom2 == 1 & mel4$Landsize== 0)] <- median(bed.land.df_1$Landsize[which(bed.land.df_1$Landsize > 1)])
mel4$Landsize[which(mel4$Bedroom2 == 2 & mel4$Landsize== 0)] <- median(bed.land.df_2$Landsize[which(bed.land.df_2$Landsize > 1)])
mel4$Landsize[which(mel4$Bedroom2 == 3 & mel4$Landsize== 0) ] <- median(bed.land.df_3$Landsize[which(bed.land.df_3$Landsize > 1)])
mel4$Landsize[which(mel4$Bedroom2 == 4 & mel4$Landsize== 0) ] <- median(bed.land.df_4$Landsize[which(bed.land.df_4$Landsize > 1)])
mel4$Landsize[which(mel4$Bedroom2 == 5 & mel4$Landsize== 0) ] <- median(bed.land.df_5$Landsize[which(bed.land.df_5$Landsize > 1)])
mel4$Landsize[which(mel4$Bedroom2 == 6 & mel4$Landsize== 0) ] <- median(bed.land.df_6$Landsize[which(bed.land.df_6$Landsize > 1)])
mel4$Landsize[which(mel4$Bedroom2 == 7 & mel4$Landsize== 0) ] <- median(bed.land.df_7$Landsize[which(bed.land.df_7$Landsize > 1)])

#Checking if all the value got atleast 100 those are zero
mel4$Landsize[which(mel4$Landsize < 120)]
summary(mel4)

#Car Column
#Putting median in all the NA values of Car column
mel4$Car <- as.numeric(mel4$Car)
mel4$Car[is.na(mel4$Car)] <- median(mel4$Car[which(!is.na(mel4$Car))])
colSums(is.na(mel4))

#Putting 0 in all the NA values of YearBuilt column
mel4$YearBuilt <- as.numeric(mel4$YearBuilt)
mel4$YearBuilt[which(is.na(mel4$YearBuilt))] <- 0
```

```R
#omitting the missing values from lattitude and longitude
mel4 <- na.omit(mel4)
#------------
str(mel4)
#dropping unused levels from the dataframe.
mel4$Postcode <- droplevels(mel4$Postcode)
mel4$CouncilArea <- droplevels(mel4$CouncilArea)
mel4$Regionname <- droplevels(mel4$Regionname)
mel4$Propertycount <- droplevels(mel4$Propertycount)
str(mel4)

table(mel4$CouncilArea)
#-------------
#converting RegionName into numeric
mel4$Regionname <- as.character(mel4$Regionname)
mel4$Regionname[mel4$Regionname == 'Eastern Metropolitan'] <- 1
mel4$Regionname[mel4$Regionname == 'Eastern Victoria'] <- 2
mel4$Regionname[mel4$Regionname == 'Northern Metropolitan'] <- 3
mel4$Regionname[mel4$Regionname == 'Northern Victoria'] <- 4
mel4$Regionname[mel4$Regionname == 'South-Eastern Metropolitan'] <- 5
mel4$Regionname[mel4$Regionname == 'Southern Metropolitan'] <- 6
mel4$Regionname[mel4$Regionname == 'Western Metropolitan'] <- 7
mel4$Regionname[mel4$Regionname == 'Western Victoria'] <- 8

#converting method into numeric
mel4$Method = as.character(mel4$Method)
mel4$Method[mel4$Method == 'PI'] <- 1
mel4$Method[mel4$Method == 'PN'] <- 2
mel4$Method[mel4$Method == 'S'] <- 3
mel4$Method[mel4$Method == 'SA'] <- 4
mel4$Method[mel4$Method == 'SN'] <- 5
mel4$Method[mel4$Method == 'SP'] <- 6
mel4$Method[mel4$Method == 'SS'] <- 7
mel4$Method[mel4$Method == 'VB'] <- 8
mel4$Method[mel4$Method == 'W'] <- 9

#converting type into numeric
mel4$Type <- as.character(mel4$Type)
mel4$Type[mel4$Type == 'h'] <- 1
mel4$Type[mel4$Type == 't'] <- 2
mel4$Type[mel4$Type == 'u'] <- 3
```

    'data.frame':	20516 obs. of  20 variables:
     $ Suburb       : Factor w/ 351 levels "Abbotsford","Aberfeldie",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ Address      : chr  "85 Turner St" "25 Bloomburg St" "5 Charles St" "40 Federation La" ...
     $ Rooms        : Factor w/ 7 levels "1","2","3","4",..: 2 2 3 3 4 2 3 2 1 2 ...
     $ Type         : Factor w/ 3 levels "h","t","u": 1 1 1 1 1 1 1 1 3 1 ...
     $ Price        : int  1480000 1035000 1465000 850000 1600000 941000 1876000 1636000 300000 1097000 ...
     $ Method       : Factor w/ 9 levels "PI","PN","S",..: 3 3 6 1 8 3 3 3 3 3 ...
     $ SellerG      : Factor w/ 388 levels "@Realty","A",..: 34 34 34 34 246 171 246 246 34 34 ...
     $ Date         : Factor w/ 78 levels "1/07/2017","10/02/2018",..: 61 64 65 65 66 71 71 76 76 76 ...
     $ Distance     : int  82 82 82 82 82 82 82 82 82 82 ...
     $ Postcode     : Factor w/ 212 levels "#N/A","3000",..: 55 55 55 55 55 55 55 55 55 55 ...
     $ Bedroom2     : int  2 2 3 3 3 2 4 2 1 3 ...
     $ Bathroom     : Factor w/ 10 levels "1","2","2.5",..: 2 2 4 4 2 2 4 2 2 2 ...
     $ Car          : num  1 0 0 1 2 0 0 2 1 2 ...
     $ Landsize     : num  202 156 134 534 120 181 245 256 453 220 ...
     $ YearBuilt    : Factor w/ 150 levels "0","1196","1820",..: 1 35 35 1 145 1 44 25 1 35 ...
     $ CouncilArea  : Factor w/ 34 levels "#N/A","Banyule City Council",..: 33 33 33 33 33 33 33 33 33 33 ...
     $ Lattitude    : num  -37.8 -37.8 -37.8 -37.8 -37.8 ...
     $ Longtitude   : num  145 145 145 145 145 ...
     $ Regionname   : Factor w/ 9 levels "#N/A","Eastern Metropolitan",..: 4 4 4 4 4 4 4 4 4 4 ...
     $ Propertycount: Factor w/ 343 levels "#N/A","1008",..: 191 191 191 191 191 191 191 191 191 191 ...

    'data.frame':	20516 obs. of  20 variables:
     $ Suburb       : Factor w/ 351 levels "Abbotsford","Aberfeldie",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ Address      : chr  "85 Turner St" "25 Bloomburg St" "5 Charles St" "40 Federation La" ...
     $ Rooms        : Factor w/ 7 levels "1","2","3","4",..: 2 2 3 3 4 2 3 2 1 2 ...
     $ Type         : Factor w/ 3 levels "h","t","u": 1 1 1 1 1 1 1 1 3 1 ...
     $ Price        : int  1480000 1035000 1465000 850000 1600000 941000 1876000 1636000 300000 1097000 ...
     $ Method       : Factor w/ 9 levels "PI","PN","S",..: 3 3 6 1 8 3 3 3 3 3 ...
     $ SellerG      : Factor w/ 388 levels "@Realty","A",..: 34 34 34 34 246 171 246 246 34 34 ...
     $ Date         : Factor w/ 78 levels "1/07/2017","10/02/2018",..: 61 64 65 65 66 71 71 76 76 76 ...
     $ Distance     : int  82 82 82 82 82 82 82 82 82 82 ...
     $ Postcode     : Factor w/ 205 levels "3000","3002",..: 54 54 54 54 54 54 54 54 54 54 ...
     $ Bedroom2     : int  2 2 3 3 3 2 4 2 1 3 ...
     $ Bathroom     : Factor w/ 10 levels "1","2","2.5",..: 2 2 4 4 2 2 4 2 2 2 ...
     $ Car          : num  1 0 0 1 2 0 0 2 1 2 ...
     $ Landsize     : num  202 156 134 534 120 181 245 256 453 220 ...
     $ YearBuilt    : Factor w/ 150 levels "0","1196","1820",..: 1 35 35 1 145 1 44 25 1 35 ...
     $ CouncilArea  : Factor w/ 33 levels "Banyule City Council",..: 32 32 32 32 32 32 32 32 32 32 ...
     $ Lattitude    : num  -37.8 -37.8 -37.8 -37.8 -37.8 ...
     $ Longtitude   : num  145 145 145 145 145 ...
     $ Regionname   : Factor w/ 8 levels "Eastern Metropolitan",..: 3 3 3 3 3 3 3 3 3 3 ...
     $ Propertycount: Factor w/ 326 levels "1008","10160",..: 180 180 180 180 180 180 180 180 180 180 ...
     - attr(*, "na.action")= 'omit' Named int  9 10 13 14 15 20 25 34 35 40 ...
      ..- attr(*, "names")= chr  "16" "17" "20" "22" ...




              Banyule City Council           Bayside City Council
                              1083                            963
           Boroondara City Council          Brimbank City Council
                              1848                           1074
            Cardinia Shire Council             Casey City Council
                                21                            118
              Darebin City Council         Frankston City Council
                              1770                            182
            Glen Eira City Council Greater Dandenong City Council
                              1193                            161
          Hobsons Bay City Council              Hume City Council
                               630                            844
             Kingston City Council              Knox City Council
                               526                            231
      Macedon Ranges Shire Council        Manningham City Council
                                17                            607
          Maribyrnong City Council         Maroondah City Council
                               963                            277
            /images/melbourne City Council            Melton City Council
                              1017                            196
            Mitchell Shire Council            Monash City Council
                                10                            749
        Moonee Valley City Council        Moorabool Shire Council
                              1219                              4
             Moreland City Council        Nillumbik Shire Council
                              1325                             58
         Port Phillip City Council       Stonnington City Council
                               730                            643
           Whitehorse City Council        Whittlesea City Council
                               350                            585
              Wyndham City Council             Yarra City Council
                               403                            666
        Yarra Ranges Shire Council
                                53



```R
mel4 <- mel4 %>% separate(Date,sep = "/",into = c("Day","Month","Year"))

#converting month into season.
#spring (March, April, May),
#summer (June, July, August),
#autumn (September, October, November)
#winter (December, January, February).

mel4$Season <- mel4$Month
mel4$Season <- as.numeric(mel4$Season)
mel4$Season[which(mel4$Season == 3 | mel4$Season == 4 | mel4$Season == 5)] = "Spring"
mel4$Season[which(mel4$Season == 6 |mel4$Season == 7 | mel4$Season == 8)] = "Summer"
mel4$Season[which(mel4$Season == 9 | mel4$Season == 10 | mel4$Season == 11)] = "Autumn"
mel4$Season[which(mel4$Season == 12 | mel4$Season == 1 | mel4$Season == 2)] = "Winter"

mel4$Season <- as.character(mel4$Season)
mel4$Season[mel4$Season == 'Spring'] <- 1
mel4$Season[mel4$Season == 'Summer'] <- 2
mel4$Season[mel4$Season == 'Autumn'] <- 3
mel4$Season[mel4$Season == 'Winter'] <- 4

#correlation checking of data
my_corrdata <- mel4[,-c(1,2,7,18)]
#converting the datacolumn into numeric
my_corrdata$Regionname <- as.numeric(my_corrdata$Regionname)
my_corrdata$Method <- as.numeric(my_corrdata$Method)
my_corrdata$Type <- as.numeric(my_corrdata$Type)
my_corrdata$Rooms <- as.numeric(my_corrdata$Rooms)
my_corrdata$Distance <- as.numeric(my_corrdata$Distance)
my_corrdata$Postcode <- as.numeric(my_corrdata$Postcode)
my_corrdata$Bedroom2 <- as.numeric(my_corrdata$Bedroom2)
my_corrdata$Bathroom <- as.numeric(my_corrdata$Bathroom)
my_corrdata$Car <- as.numeric(my_corrdata$Car)
my_corrdata$YearBuilt <- as.numeric(my_corrdata$YearBuilt)
my_corrdata$Day <- as.numeric(my_corrdata$Day)
my_corrdata$Month <- as.numeric(my_corrdata$Month)
my_corrdata$Year <- as.numeric(my_corrdata$Year)
my_corrdata$Propertycount <- as.numeric(my_corrdata$Propertycount)
my_corrdata$Season <- as.numeric(my_corrdata$Season)
corr <- round(cor(my_corrdata),1)

corrplot(corr)


```
![png](/images/melbourne/melbourne-housing-price-prediction_16_0.png)
```R
#normalization of data
normalize <- function(x){
  return ((x - min(x)) / (max(x) - min(x)))
}

mel4$Landsize <- normalize(mel4$Landsize)
mel4$Distance <- normalize(mel4$Distance)

mel4$Rooms <- as.numeric(mel4$Rooms)
mel4$Rooms <- normalize(mel4$Rooms)

mel4$Bathroom <- as.numeric(mel4$Bathroom)
mel4$Bathroom <- normalize(mel4$Bathroom)
```


```R
mel4$Car <- as.numeric(mel4$Car)
mel4$Car <- normalize(mel4$Car)

mel4$Propertycount <- as.numeric(mel4$Propertycount)
mel4$Propertycount <- normalize(mel4$Propertycount)
colnames(mel4)

head(mel4)

#one hot encoding and datatype corrections
mel4$Regionname <- as.factor(mel4$Regionname) #one hot needed
mel4$Method <- as.factor(mel4$Method) #one hot needed
mel4$Type <- as.factor(mel4$Type) #one hot needed
mel4$Rooms <- as.factor(mel4$Rooms)
mel4$Car <- as.numeric(mel4$Car)
mel4$Bedroom2 <- as.factor(mel4$Bedroom2)
mel4$Bathroom <- as.factor(mel4$Bathroom)
mel4$YearBuilt <- as.numeric(mel4$YearBuilt)
mel4$Day <- as.numeric(mel4$Day)
mel4$Month <- as.factor(mel4$Month) #as we have create a new variable season using Month we will not be using month
mel4$Propertycount <- as.character(mel4$Propertycount)
mel4$Propertycount <- as.numeric(mel4$Propertycount)
mel4$Season <- as.numeric(mel4$Season) #one hot encoding needed

#one hot encoding of type
type_ <- factor(mel4$Type)
dumm <- as.data.frame(model.matrix(~type_)[,-1])
mel4 <- cbind(dumm,mel4)

#one hot encoding of Method
Method_ <- factor(mel4$Method)
dumm <- as.data.frame(model.matrix(~Method_)[,-1])
mel4 <- cbind(dumm,mel4)

#one hot encoding of season
Season_ <- factor(mel4$Season)
dumm <- as.data.frame(model.matrix(~Season_)[,-1])
mel4 <- cbind(dumm,mel4)


mel4$CouncilArea <- str_replace_all(mel4$CouncilArea,c(" "="_"))
Council_ <- factor(mel4$CouncilArea)
dumm <- as.data.frame(model.matrix(~Council_)[,-1])
mel4 <- cbind(dumm,mel4)

#test_df <- mel4[,-c(40,41,42,43,44,45,47,48,51,54,55,56,58,59,60,61)]

colnames(mel4)

#removing the Rooms from the final dataset as we observed high collinearity between Bedroom2 and Rooms
test_df <- mel4[,-c(42,43,45,47,48,49,50,51,53,54,58,59,62,63,64)]
colnames(test_df)
#splitting the data into train and test
create_train_test <- function(data, size = 0.7, train = TRUE) {
  n_row = nrow(data)
  total_row = size * n_row
  train_sample <- 1: total_row
  if (train == TRUE) {
    return (data[train_sample, ])
  } else {
    return (data[-train_sample, ])
  }
}

set.seed(123)
train_data <- create_train_test(test_df,train = TRUE)
test_data <- create_train_test(test_df,train = FALSE)
```

```R
#####################################################
#### ---------Linear Regression
######################################################

colnames(train_data)

#full model
model_l <- lm(Price ~ .,data = train_data)
summary(model_l)
plot(model_l)
```


    Call:
    lm(formula = Price ~ ., data = train_data)

    Residuals:
         Min       1Q   Median       3Q      Max
    -1538214  -219931   -36880   158636  7781226

    Coefficients:
                                            Estimate Std. Error t value
    (Intercept)                            -57447697   16811374  -3.417
    Council_Bayside_City_Council              262166      37704   6.953
    Council_Boroondara_City_Council           469599      21343  22.003
    Council_Brimbank_City_Council            -341206      37457  -9.109
    Council_Cardinia_Shire_Council          -1201870     131798  -9.119
    Council_Casey_City_Council              -1174178      66498 -17.657
    Council_Darebin_City_Council               65725      19654   3.344
    Council_Frankston_City_Council          -1310758      71064 -18.445
    Council_Glen_Eira_City_Council            -38199      31728  -1.204
    Council_Greater_Dandenong_City_Council   -736451      55248 -13.330
    Council_Hobsons_Bay_City_Council         -210445      40100  -5.248
    Council_Hume_City_Council                -233834      31367  -7.455
    Council_Kingston_City_Council            -475474      42439 -11.204
    Council_Knox_City_Council                -513749      46325 -11.090
    Council_Macedon_Ranges_Shire_Council      -86124     147498  -0.584
    Council_Manningham_City_Council            65985      25220   2.616
    Council_Maribyrnong_City_Council         -186285      33168  -5.616
    Council_Maroondah_City_Council           -249594      42169  -5.919
    Council_/images/melbourne_City_Council            336677      27986  12.030
    Council_Melton_City_Council              -617262      67437  -9.153
    Council_Mitchell_Shire_Council            138011     383196   0.360
    Council_Monash_City_Council               -77261      30761  -2.512
    Council_Moonee_Valley_City_Council         70536      28235   2.498
    Council_Moorabool_Shire_Council          -688421     387795  -1.775
    Council_Moreland_City_Council              52472      23779   2.207
    Council_Nillumbik_Shire_Council          -232477      90307  -2.574
    Council_Port_Phillip_City_Council         308289      31245   9.867
    Council_Stonnington_City_Council          475326      29180  16.289
    Council_Whitehorse_City_Council            59731      33624   1.776
    Council_Whittlesea_City_Council          -162129      30540  -5.309
    Council_Wyndham_City_Council             -899228      67910 -13.241
    Council_Yarra_City_Council                347223      27225  12.754
    Council_Yarra_Ranges_Shire_Council       -440423      91918  -4.791
    Season_2                                  -26292       8374  -3.140
    Season_3                                   -7951       8612  -0.923
    Season_4                                    9903      13449   0.736
    Method_3                                   86492      10393   8.322
    Method_4                                    3201      39959   0.080
    Method_6                                   43909      13248   3.314
    Method_8                                   36701      14325   2.562
    type_2                                   -274286      12322 -22.260
    type_3                                   -471495      10322 -45.678
    Rooms0.166666666666667                    301723      16681  18.088
    Rooms0.333333333333333                    466227      18173  25.655
    Rooms0.5                                  629918      20426  30.840
    Rooms0.666666666666667                    725742      25946  27.971
    Rooms0.833333333333333                    576574      51430  11.211
    Rooms1                                    541834     116879   4.636
    Distance                                  172816      12333  14.012
    Bathroom0.111111111111111                  57574      36955   1.558
    Bathroom0.222222222222222                 333552      95466   3.494
    Bathroom0.333333333333333                 190064      37490   5.070
    Bathroom0.444444444444444                 422765      39796  10.623
    Bathroom0.666666666666667                1165338      53509  21.778
    Bathroom0.777777777777778                 936938      84855  11.042
    Bathroom0.888888888888889                 589075     223126   2.640
    Bathroom1                                2036404     382858   5.319
    Car                                       294508      69814   4.218
    Landsize                                  363054      21666  16.757
    Lattitude                               -2315643     142830 -16.213
    Longtitude                               -205653     121415  -1.694
                                                       Pr(>|t|)    
    (Intercept)                                        0.000634 ***
    Council_Bayside_City_Council             0.0000000000037242 ***
    Council_Boroondara_City_Council        < 0.0000000000000002 ***
    Council_Brimbank_City_Council          < 0.0000000000000002 ***
    Council_Cardinia_Shire_Council         < 0.0000000000000002 ***
    Council_Casey_City_Council             < 0.0000000000000002 ***
    Council_Darebin_City_Council                       0.000828 ***
    Council_Frankston_City_Council         < 0.0000000000000002 ***
    Council_Glen_Eira_City_Council                     0.228630    
    Council_Greater_Dandenong_City_Council < 0.0000000000000002 ***
    Council_Hobsons_Bay_City_Council         0.0000001559753846 ***
    Council_Hume_City_Council                0.0000000000000951 ***
    Council_Kingston_City_Council          < 0.0000000000000002 ***
    Council_Knox_City_Council              < 0.0000000000000002 ***
    Council_Macedon_Ranges_Shire_Council               0.559298    
    Council_Manningham_City_Council                    0.008895 **
    Council_Maribyrnong_City_Council         0.0000000198657832 ***
    Council_Maroondah_City_Council           0.0000000033140992 ***
    Council_/images/melbourne_City_Council         < 0.0000000000000002 ***
    Council_Melton_City_Council            < 0.0000000000000002 ***
    Council_Mitchell_Shire_Council                     0.718735    
    Council_Monash_City_Council                        0.012027 *  
    Council_Moonee_Valley_City_Council                 0.012495 *  
    Council_Moorabool_Shire_Council                    0.075883 .  
    Council_Moreland_City_Council                      0.027356 *  
    Council_Nillumbik_Shire_Council                    0.010055 *  
    Council_Port_Phillip_City_Council      < 0.0000000000000002 ***
    Council_Stonnington_City_Council       < 0.0000000000000002 ***
    Council_Whitehorse_City_Council                    0.075682 .  
    Council_Whittlesea_City_Council          0.0000001120368891 ***
    Council_Wyndham_City_Council           < 0.0000000000000002 ***
    Council_Yarra_City_Council             < 0.0000000000000002 ***
    Council_Yarra_Ranges_Shire_Council       0.0000016721240706 ***
    Season_2                                           0.001693 **
    Season_3                                           0.355888    
    Season_4                                           0.461536    
    Method_3                               < 0.0000000000000002 ***
    Method_4                                           0.936161    
    Method_6                                           0.000921 ***
    Method_8                                           0.010418 *  
    type_2                                 < 0.0000000000000002 ***
    type_3                                 < 0.0000000000000002 ***
    Rooms0.166666666666667                 < 0.0000000000000002 ***
    Rooms0.333333333333333                 < 0.0000000000000002 ***
    Rooms0.5                               < 0.0000000000000002 ***
    Rooms0.666666666666667                 < 0.0000000000000002 ***
    Rooms0.833333333333333                 < 0.0000000000000002 ***
    Rooms1                                   0.0000035860354886 ***
    Distance                               < 0.0000000000000002 ***
    Bathroom0.111111111111111                          0.119269    
    Bathroom0.222222222222222                          0.000477 ***
    Bathroom0.333333333333333                0.0000004034108199 ***
    Bathroom0.444444444444444              < 0.0000000000000002 ***
    Bathroom0.666666666666667              < 0.0000000000000002 ***
    Bathroom0.777777777777778              < 0.0000000000000002 ***
    Bathroom0.888888888888889                          0.008297 **
    Bathroom1                                0.0000001059361533 ***
    Car                                      0.0000247491780222 ***
    Landsize                               < 0.0000000000000002 ***
    Lattitude                              < 0.0000000000000002 ***
    Longtitude                                         0.090324 .  
    ---

    Residual standard error: 380100 on 14300 degrees of freedom
    Multiple R-squared:  0.6329,	Adjusted R-squared:  0.6314
    F-statistic: 410.9 on 60 and 14300 DF,  p-value: < 0.00000000000000022


![png](/images/melbourne/melbourne-housing-price-prediction_19_3.png)

![png](/images/melbourne/melbourne-housing-price-prediction_19_5.png)

![png](/images/melbourne/melbourne-housing-price-prediction_19_6.png)

![png](/images/melbourne/melbourne-housing-price-prediction_19_7.png)

```R
vif(model_l)
options(warn=-1)
predicted_ys <- predict(model_l,newdata=test_data)
observed_ys <- test_data$Price
SSE <- sum((observed_ys - predicted_ys) ^ 2)
SST <- sum((observed_ys - mean(observed_ys)) ^ 2)
r2 <- 1 - SSE/SST
r2
```
0.627673721231063

```R
##############################################
###--------STEPWISE MODEL
###############################################

step_model <- stepAIC(model_l,direction="both",trace=1)
summary(step_model)
step_model$anova
options(warn=-1)

```
    Start:  AIC=369089.8
    Price ~ Council_Bayside_City_Council + Council_Boroondara_City_Council +
        Council_Brimbank_City_Council + Council_Cardinia_Shire_Council +
        Council_Casey_City_Council + Council_Darebin_City_Council +
        Council_Frankston_City_Council + Council_Glen_Eira_City_Council +
        Council_Greater_Dandenong_City_Council + Council_Hobsons_Bay_City_Council +
        Council_Hume_City_Council + Council_Kingston_City_Council +
        Council_Knox_City_Council + Council_Macedon_Ranges_Shire_Council +
        Council_Manningham_City_Council + Council_Maribyrnong_City_Council +
        Council_Maroondah_City_Council + Council_/images/melbourne_City_Council +
        Council_Melton_City_Council + Council_Mitchell_Shire_Council +
        Council_Monash_City_Council + Council_Moonee_Valley_City_Council +
        Council_Moorabool_Shire_Council + Council_Moreland_City_Council +
        Council_Nillumbik_Shire_Council + Council_Port_Phillip_City_Council +
        Council_Stonnington_City_Council + Council_Whitehorse_City_Council +
        Council_Whittlesea_City_Council + Council_Wyndham_City_Council +
        Council_Yarra_City_Council + Council_Yarra_Ranges_Shire_Council +
        Season_2 + Season_3 + Season_4 + Method_3 + Method_4 + Method_6 +
        Method_8 + type_2 + type_3 + Rooms + Distance + Bathroom +
        Car + Landsize + Lattitude + Longtitude

                                             Df       Sum of Sq              RSS
    - Method_4                                1       927113108 2066478130105480
    - Council_Mitchell_Shire_Council          1     18744636821 2066495947629193
    - Council_Macedon_Ranges_Shire_Council    1     49268403045 2066526471395417
    - Season_4                                1     78352322202 2066555555314574
    - Season_3                                1    123180234364 2066600383226736
    - Council_Glen_Eira_City_Council          1    209464061696 2066686667054068
    <none>                                                      2066477202992372
    - Longtitude                              1    414593643310 2066891796635682
    - Council_Moorabool_Shire_Council         1    455406012286 2066932609004658
    - Council_Whitehorse_City_Council         1    456031432552 2066933234424924
    - Council_Moreland_City_Council           1    703643591148 2067180846583520
    - Council_Moonee_Valley_City_Council      1    901847246847 2067379050239219
    - Council_Monash_City_Council             1    911627863792 2067388830856164
    - Method_8                                1    948526404748 2067425729397120
    - Council_Nillumbik_Shire_Council         1    957655441616 2067434858433988
    - Council_Manningham_City_Council         1    989251740321 2067466454732693
    - Season_2                                1   1424728270543 2067901931262915
    - Method_6                                1   1587491251696 2068064694244068
    - Council_Darebin_City_Council            1   1616011561297 2068093214553669
    - Car                                     1   2571594455856 2069048797448228
    - Council_Yarra_Ranges_Shire_Council      1   3317685731365 2069794888723737
    - Council_Hobsons_Bay_City_Council        1   3979959504132 2070457162496504
    - Council_Whittlesea_City_Council         1   4072651127649 2070549854120021
    - Council_Maribyrnong_City_Council        1   4558312444100 2071035515436472
    - Council_Maroondah_City_Council          1   5062728645007 2071539931637379
    - Council_Bayside_City_Council            1   6986686307564 2073463889299936
    - Council_Hume_City_Council               1   8031113427613 2074508316419985
    - Method_3                                1  10008686805594 2076485889797966
    - Council_Brimbank_City_Council           1  11991058701207 2078468261693579
    - Council_Cardinia_Shire_Council          1  12016797617070 2078494000609442
    - Council_Melton_City_Council             1  12106908430450 2078584111422822
    - Council_Port_Phillip_City_Council       1  14068904850189 2080546107842561
    - Council_Knox_City_Council               1  17773375854119 2084250578846491
    - Council_Kingston_City_Council           1  18138941972030 2084616144964402
    - Council_Yarra_City_Council              1  23505822664693 2089983025657065
    - Council_Wyndham_City_Council            1  25337239087412 2091814442079784
    - Council_Greater_Dandenong_City_Council  1  25677364703476 2092154567695848
    - Distance                                1  28373339330050 2094850542322422
    - Lattitude                               1  37983588998992 2104460791991364
    - Council_Stonnington_City_Council        1  38343518831948 2104820721824320
    - Landsize                                1  40577639530173 2107054842522545
    - Council_Casey_City_Council              1  45055015826795 2111532218819167
    - Council_Frankston_City_Council          1  49163204067671 2115640407060043
    - Council_Boroondara_City_Council         1  69959006353134 2136436209345506
    - type_2                                  1  71607513732619 2138084716724991
    - Rooms                                   6 157668685918302 2224145888910674
    - Bathroom                                8 177650084648302 2244127287640674
    - type_3                                  1 301515423517621 2367992626509993
                                                AIC
    - Method_4                               369088
    - Council_Mitchell_Shire_Council         369088
    - Council_Macedon_Ranges_Shire_Council   369088
    - Season_4                               369088
    - Season_3                               369089
    - Council_Glen_Eira_City_Council         369089
    <none>                                   369090
    - Longtitude                             369091
    - Council_Moorabool_Shire_Council        369091
    - Council_Whitehorse_City_Council        369091
    - Council_Moreland_City_Council          369093
    - Council_Moonee_Valley_City_Council     369094
    - Council_Monash_City_Council            369094
    - Method_8                               369094
    - Council_Nillumbik_Shire_Council        369094
    - Council_Manningham_City_Council        369095
    - Season_2                               369098
    - Method_6                               369099
    - Council_Darebin_City_Council           369099
    - Car                                    369106
    - Council_Yarra_Ranges_Shire_Council     369111
    - Council_Hobsons_Bay_City_Council       369115
    - Council_Whittlesea_City_Council        369116
    - Council_Maribyrnong_City_Council       369119
    - Council_Maroondah_City_Council         369123
    - Council_Bayside_City_Council           369136
    - Council_Hume_City_Council              369144
    - Method_3                               369157
    - Council_Brimbank_City_Council          369171
    - Council_Cardinia_Shire_Council         369171
    - Council_Melton_City_Council            369172
    - Council_Port_Phillip_City_Council      369185
    - Council_Knox_City_Council              369211
    - Council_Kingston_City_Council          369213
    - Council_Yarra_City_Council             369250
    - Council_Wyndham_City_Council           369263
    - Council_Greater_Dandenong_City_Council 369265
    - Distance                               369284
    - Lattitude                              369349
    - Council_Stonnington_City_Council       369352
    - Landsize                               369367
    - Council_Casey_City_Council             369398
    - Council_Frankston_City_Council         369426
    - Council_Boroondara_City_Council        369566
    - type_2                                 369577
    - Rooms                                  370134
    - Bathroom                               370258
    - type_3                                 371044

    Step:  AIC=369087.8
    Price ~ Council_Bayside_City_Council + Council_Boroondara_City_Council +
        Council_Brimbank_City_Council + Council_Cardinia_Shire_Council +
        Council_Casey_City_Council + Council_Darebin_City_Council +
        Council_Frankston_City_Council + Council_Glen_Eira_City_Council +
        Council_Greater_Dandenong_City_Council + Council_Hobsons_Bay_City_Council +
        Council_Hume_City_Council + Council_Kingston_City_Council +
        Council_Knox_City_Council + Council_Macedon_Ranges_Shire_Council +
        Council_Manningham_City_Council + Council_Maribyrnong_City_Council +
        Council_Maroondah_City_Council +
        Council_Melton_City_Council + Council_Mitchell_Shire_Council +
        Council_Monash_City_Council + Council_Moonee_Valley_City_Council +
        Council_Moorabool_Shire_Council + Council_Moreland_City_Council +
        Council_Nillumbik_Shire_Council + Council_Port_Phillip_City_Council +
        Council_Stonnington_City_Council + Council_Whitehorse_City_Council +
        Council_Whittlesea_City_Council + Council_Wyndham_City_Council +
        Council_Yarra_City_Council + Council_Yarra_Ranges_Shire_Council +
        Season_2 + Season_3 + Season_4 + Method_3 + Method_6 + Method_8 +
        type_2 + type_3 + Rooms + Distance + Bathroom + Car + Landsize +
        Lattitude + Longtitude

                                             Df       Sum of Sq              RSS
    - Council_Mitchell_Shire_Council          1     18744739114 2066496874844594
    - Council_Macedon_Ranges_Shire_Council    1     48840479482 2066526970584961
    - Season_4                                1     78326508802 2066556456614282
    - Season_3                                1    123225530597 2066601355636076
    - Council_Glen_Eira_City_Council          1    209478456675 2066687608562154
    <none>                                                      2066478130105480
    - Longtitude                              1    414030094078 2066892160199558
    - Council_Moorabool_Shire_Council         1    455284477576 2066933414583056
    - Council_Whitehorse_City_Council         1    456069659264 2066934199764743
    + Method_4                                1       927113108 2066477202992372
    - Council_Moreland_City_Council           1    704034731921 2067182164837400
    - Council_Moonee_Valley_City_Council      1    902493460240 2067380623565720
    - Council_Monash_City_Council             1    910818548998 2067388948654477
    - Council_Nillumbik_Shire_Council         1    957854258442 2067435984363921
    - Method_8                                1    963064769588 2067441194875067
    - Council_Manningham_City_Council         1    988647436674 2067466777542153
    - Season_2                                1   1423930677204 2067902060782684
    - Council_Darebin_City_Council            1   1616315674779 2068094445780258
    - Method_6                                1   1623786438798 2068101916544278
    - Car                                     1   2571831949752 2069049962055232
    - Council_Yarra_Ranges_Shire_Council      1   3318987913447 2069797118018927
    - Council_Hobsons_Bay_City_Council        1   3979050262668 2070457180368147
    - Council_Whittlesea_City_Council         1   4071984573746 2070550114679225
    - Council_Maribyrnong_City_Council        1   4557420871952 2071035550977432
    - Council_Maroondah_City_Council          1   5062765314494 2071540895419973
    - Council_Bayside_City_Council            1   6989499255116 2073467629360595
    - Council_Hume_City_Council               1   8030232533526 2074508362639006
    - Method_3                                1  10490125720186 2076968255825665
    - Council_Brimbank_City_Council           1  11991489617615 2078469619723095
    - Council_Cardinia_Shire_Council          1  12017810666742 2078495940772222
    - Council_Melton_City_Council             1  12107090245554 2078585220351034
    - Council_Port_Phillip_City_Council       1  14071448312222 2080549578417702
    - Council_Knox_City_Council               1  17776235278216 2084254365383696
    - Council_Kingston_City_Council           1  18138190243000 2084616320348479
    - Council_Yarra_City_Council              1  23507807700050 2089985937805529
    - Council_Wyndham_City_Council            1  25339373660398 2091817503765878
    - Council_Greater_Dandenong_City_Council  1  25680130965791 2092158261071270
    - Distance                                1  28372573729582 2094850703835062
    - Lattitude                               1  37982670146802 2104460800252281
    - Council_Stonnington_City_Council        1  38344195826705 2104822325932184
    - Landsize                                1  40589969719014 2107068099824493
    - Council_Casey_City_Council              1  45058628254792 2111536758360272
    - Council_Frankston_City_Council          1  49163284848312 2115641414953792
    - Council_Boroondara_City_Council         1  69966368794184 2136444498899663
    - type_2                                  1  71607153772594 2138085283878074
    - Rooms                                   6 157671189674595 2224149319780074
    - Bathroom                                8 177650664161619 2244128794267099
    - type_3                                  1 301514575764888 2367992705870368
                                                AIC
    - Council_Mitchell_Shire_Council         369086
    - Council_Macedon_Ranges_Shire_Council   369086
    - Season_4                               369086
    - Season_3                               369087
    - Council_Glen_Eira_City_Council         369087
    <none>                                   369088
    - Longtitude                             369089
    - Council_Moorabool_Shire_Council        369089
    - Council_Whitehorse_City_Council        369089
    + Method_4                               369090
    - Council_Moreland_City_Council          369091
    - Council_Moonee_Valley_City_Council     369092
    - Council_Monash_City_Council            369092
    - Council_Nillumbik_Shire_Council        369093
    - Method_8                               369093
    - Council_Manningham_City_Council        369093
    - Season_2                               369096
    - Council_Darebin_City_Council           369097
    - Method_6                               369097
    - Car                                    369104
    - Council_Yarra_Ranges_Shire_Council     369109
    - Council_Hobsons_Bay_City_Council       369113
    - Council_Whittlesea_City_Council        369114
    - Council_Maribyrnong_City_Council       369117
    - Council_Maroondah_City_Council         369121
    - Council_Bayside_City_Council           369134
    - Council_Hume_City_Council              369142
    - Method_3                               369159
    - Council_Brimbank_City_Council          369169
    - Council_Cardinia_Shire_Council         369169
    - Council_Melton_City_Council            369170
    - Council_Port_Phillip_City_Council      369183
    - Council_Knox_City_Council              369209
    - Council_Kingston_City_Council          369211
    - Council_Yarra_City_Council             369248
    - Council_Wyndham_City_Council           369261
    - Council_Greater_Dandenong_City_Council 369263
    - Distance                               369282
    - Lattitude                              369347
    - Council_Stonnington_City_Council       369350
    - Landsize                               369365
    - Council_Casey_City_Council             369396
    - Council_Frankston_City_Council         369424
    - Council_Boroondara_City_Council        369564
    - type_2                                 369575
    - Rooms                                  370132
    - Bathroom                               370256
    - type_3                                 371042

    Step:  AIC=369086
    Price ~ Council_Bayside_City_Council + Council_Boroondara_City_Council +
        Council_Brimbank_City_Council + Council_Cardinia_Shire_Council +
        Council_Casey_City_Council + Council_Darebin_City_Council +
        Council_Frankston_City_Council + Council_Glen_Eira_City_Council +
        Council_Greater_Dandenong_City_Council + Council_Hobsons_Bay_City_Council +
        Council_Hume_City_Council + Council_Kingston_City_Council +
        Council_Knox_City_Council + Council_Macedon_Ranges_Shire_Council +
        Council_Manningham_City_Council + Council_Maribyrnong_City_Council +
        Council_Maroondah_City_Council+
        Council_Melton_City_Council + Council_Monash_City_Council +
        Council_Moonee_Valley_City_Council + Council_Moorabool_Shire_Council +
        Council_Moreland_City_Council + Council_Nillumbik_Shire_Council +
        Council_Port_Phillip_City_Council + Council_Stonnington_City_Council +
        Council_Whitehorse_City_Council + Council_Whittlesea_City_Council +
        Council_Wyndham_City_Council + Council_Yarra_City_Council +
        Council_Yarra_Ranges_Shire_Council + Season_2 + Season_3 +
        Season_4 + Method_3 + Method_6 + Method_8 + type_2 + type_3 +
        Rooms + Distance + Bathroom + Car + Landsize + Lattitude +
        Longtitude

                                             Df       Sum of Sq              RSS
    - Council_Macedon_Ranges_Shire_Council    1     50654560190 2066547529404783
    - Season_4                                1     78311778261 2066575186622854
    - Season_3                                1    122237456173 2066619112300766
    - Council_Glen_Eira_City_Council          1    201301619880 2066698176464474
    <none>                                                      2066496874844594
    - Longtitude                              1    412562314402 2066909437158995
    - Council_Moorabool_Shire_Council         1    455755419626 2066952630264220
    - Council_Whitehorse_City_Council         1    462156223188 2066959031067782
    + Council_Mitchell_Shire_Council          1     18744739114 2066478130105479
    + Method_4                                1       927215400 2066495947629193
    - Council_Moreland_City_Council           1    700972850729 2067197847695322
    - Council_Monash_City_Council             1    897669819793 2067394544664386
    - Council_Moonee_Valley_City_Council      1    901272221876 2067398147066470
    - Method_8                                1    962869673302 2067459744517896
    - Council_Nillumbik_Shire_Council         1    964804959715 2067461679804308
    - Council_Manningham_City_Council         1    991707188226 2067488582032820
    - Season_2                                1   1424352409138 2067921227253732
    - Council_Darebin_City_Council            1   1609568652084 2068106443496678
    - Method_6                                1   1623441303856 2068120316148449
    - Car                                     1   2568881928446 2069065756773040
    - Council_Yarra_Ranges_Shire_Council      1   3319690306353 2069816565150946
    - Council_Hobsons_Bay_City_Council        1   3964198408382 2070461073252976
    - Council_Whittlesea_City_Council         1   4119762094876 2070616636939470
    - Council_Maribyrnong_City_Council        1   4548882938659 2071045757783253
    - Council_Maroondah_City_Council          1   5055724354881 2071552599199475
    - Council_Bayside_City_Council            1   7087587886864 2073584462731458
    - Council_Hume_City_Council               1   8090269482780 2074587144327374
    - Method_3                                1  10491168321372 2076988043165965
    - Council_Brimbank_City_Council           1  11986881032299 2078483755876893
    - Council_Cardinia_Shire_Council          1  11999841819792 2078496716664385
    - Council_Melton_City_Council             1  12113470074600 2078610344919194
    - Council_Port_Phillip_City_Council       1  14154748094894 2080651622939488
    - Council_Knox_City_Council               1  17757720708268 2084254595552862
    - Council_Kingston_City_Council           1  18163528538739 2084660403383332
    - Council_Yarra_City_Council              1  23567394802347 2090064269646940
    - Council_Wyndham_City_Council            1  25322179850107 2091819054694701
    - Council_Greater_Dandenong_City_Council  1  25686911241210 2092183786085803
    - Distance                                1  28435610559176 2094932485403769
    - Lattitude                               1  38299012901570 2104795887746163
    - Council_Stonnington_City_Council        1  38527432300729 2105024307145322
    - Landsize                                1  40578910442157 2107075785286750
    - Council_Casey_City_Council              1  45111961789015 2111608836633609
    - Council_Frankston_City_Council          1  49338937143118 2115835811987711
    - Council_Boroondara_City_Council         1  70174847078876 2136671721923469
    - type_2                                  1  71614101427744 2138110976272338
    - Rooms                                   6 157672290527259 2224169165371852
    - Bathroom                                8 177637408407030 2244134283251623
    - type_3                                  1 301518083017206 2368014957861799
                                                AIC
    - Council_Macedon_Ranges_Shire_Council   369084
    - Season_4                               369085
    - Season_3                               369085
    - Council_Glen_Eira_City_Council         369085
    <none>                                   369086
    - Longtitude                             369087
    - Council_Moorabool_Shire_Council        369087
    - Council_Whitehorse_City_Council        369087
    + Council_Mitchell_Shire_Council         369088
    + Method_4                               369088
    - Council_Moreland_City_Council          369089
    - Council_Monash_City_Council            369090
    - Council_Moonee_Valley_City_Council     369090
    - Method_8                               369091
    - Council_Nillumbik_Shire_Council        369091
    - Council_Manningham_City_Council        369091
    - Season_2                               369094
    - Council_Darebin_City_Council           369095
    - Method_6                               369095
    - Car                                    369102
    - Council_Yarra_Ranges_Shire_Council     369107
    - Council_Hobsons_Bay_City_Council       369112
    - Council_Whittlesea_City_Council        369113
    - Council_Maribyrnong_City_Council       369116
    - Council_Maroondah_City_Council         369119
    - Council_Bayside_City_Council           369133
    - Council_Hume_City_Council              369140
    - Method_3                               369157
    - Council_Brimbank_City_Council          369167
    - Council_Cardinia_Shire_Council         369167
    - Council_Melton_City_Council            369168
    - Council_Port_Phillip_City_Council      369182
    - Council_Knox_City_Council              369207
    - Council_Kingston_City_Council          369210
    - Council_/images/melbourne_City_Council         369229
    - Council_Yarra_City_Council             369247
    - Council_Wyndham_City_Council           369259
    - Council_Greater_Dandenong_City_Council 369261
    - Distance                               369280
    - Lattitude                              369348
    - Council_Stonnington_City_Council       369349
    - Landsize                               369363
    - Council_Casey_City_Council             369394
    - Council_Frankston_City_Council         369423
    - Council_Boroondara_City_Council        369564
    - type_2                                 369573
    - Rooms                                  370130
    - Bathroom                               370254
    - type_3                                 371040

    Step:  AIC=369084.3
    Price ~ Council_Bayside_City_Council + Council_Boroondara_City_Council +
        Council_Brimbank_City_Council + Council_Cardinia_Shire_Council +
        Council_Casey_City_Council + Council_Darebin_City_Council +
        Council_Frankston_City_Council + Council_Glen_Eira_City_Council +
        Council_Greater_Dandenong_City_Council + Council_Hobsons_Bay_City_Council +
        Council_Hume_City_Council + Council_Kingston_City_Council +
        Council_Knox_City_Council + Council_Manningham_City_Council +
        Council_Maribyrnong_City_Council + Council_Maroondah_City_Council + Council_Melton_City_Council +
        Council_Monash_City_Council + Council_Moonee_Valley_City_Council +
        Council_Moorabool_Shire_Council + Council_Moreland_City_Council +
        Council_Nillumbik_Shire_Council + Council_Port_Phillip_City_Council +
        Council_Stonnington_City_Council + Council_Whitehorse_City_Council +
        Council_Whittlesea_City_Council + Council_Wyndham_City_Council +
        Council_Yarra_City_Council + Council_Yarra_Ranges_Shire_Council +
        Season_2 + Season_3 + Season_4 + Method_3 + Method_6 + Method_8 +
        type_2 + type_3 + Rooms + Distance + Bathroom + Car + Landsize +
        Lattitude + Longtitude

                                             Df       Sum of Sq              RSS
    - Season_4                                1     78369543839 2066625898948622
    - Season_3                                1    123978305940 2066671507710724
    - Council_Glen_Eira_City_Council          1    208849913974 2066756379318758
    <none>                                                      2066547529404783
    - Longtitude                              1    362703331198 2066910232735981
    - Council_Whitehorse_City_Council         1    435010202989 2066982539607772
    - Council_Moorabool_Shire_Council         1    437270538561 2066984799943344
    + Council_Macedon_Ranges_Shire_Council    1     50654560190 2066496874844594
    + Council_Mitchell_Shire_Council          1     20558819822 2066526970584961
    + Method_4                                1       492486478 2066547036918306
    - Council_Moreland_City_Council           1    844901325166 2067392430729950
    - Council_Monash_City_Council             1    960222189578 2067507751594362
    - Council_Nillumbik_Shire_Council         1    963031506573 2067510560911356
    - Council_Manningham_City_Council         1    965148615133 2067512678019916
    - Method_8                                1    966856158631 2067514385563414
    - Council_Moonee_Valley_City_Council      1   1088881416181 2067636410820964
    - Season_2                                1   1433597420340 2067981126825123
    - Method_6                                1   1621312647616 2068168842052400
    - Council_Darebin_City_Council            1   1795272823396 2068342802228180
    - Car                                     1   2571019159249 2069118548564032
    - Council_Yarra_Ranges_Shire_Council      1   3438951027080 2069986480431864
    - Council_Hobsons_Bay_City_Council        1   3927866769148 2070475396173931
    - Council_Whittlesea_City_Council         1   4070184456025 2070617713860808
    - Council_Maribyrnong_City_Council        1   4561157096224 2071108686501007
    - Council_Maroondah_City_Council          1   5370179810053 2071917709214836
    - Council_Bayside_City_Council            1   7074856806680 2073622386211464
    - Council_Hume_City_Council               1   8407910273771 2074955439678554
    - Method_3                                1  10497881302516 2077045410707299
    - Council_Cardinia_Shire_Council          1  12435680908474 2078983210313258
    - Council_Brimbank_City_Council           1  12463635709193 2079011165113976
    - Council_Melton_City_Council             1  12597962233646 2079145491638429
    - Council_Port_Phillip_City_Council       1  14386361746946 2080933891151729
    - Council_Kingston_City_Council           1  18484428394426 2085031957799210
    - Council_Knox_City_Council               1  18663431836755 2085210961241538
    - Council_Yarra_City_Council              1  23996227444119 2090543756848902
    - Council_Wyndham_City_Council            1  25875078639768 2092422608044552
    - Council_Greater_Dandenong_City_Council  1  26483025161888 2093030554566671
    - Distance                                1  28392669300577 2094940198705360
    - Council_Stonnington_City_Council        1  38622079169626 2105169608574409
    - Lattitude                               1  39345052052118 2105892581456901
    - Landsize                                1  40542678183540 2107090207588324
    - Council_Casey_City_Council              1  47276233087074 2113823762491858
    - Council_Frankston_City_Council          1  50553988927595 2117101518332378
    - Council_Boroondara_City_Council         1  70164717372695 2136712246777478
    - type_2                                  1  71597272185279 2138144801590062
    - Rooms                                   6 157677192487091 2224224721891874
    - Bathroom                                8 177661347432710 2244208876837493
    - type_3                                  1 301581568649322 2368129098054105
                                                AIC
    - Season_4                               369083
    - Season_3                               369083
    - Council_Glen_Eira_City_Council         369084
    <none>                                   369084
    - Longtitude                             369085
    - Council_Whitehorse_City_Council        369085
    - Council_Moorabool_Shire_Council        369085
    + Council_Macedon_Ranges_Shire_Council   369086
    + Council_Mitchell_Shire_Council         369086
    + Method_4                               369086
    - Council_Moreland_City_Council          369088
    - Council_Monash_City_Council            369089
    - Council_Nillumbik_Shire_Council        369089
    - Council_Manningham_City_Council        369089
    - Method_8                               369089
    - Council_Moonee_Valley_City_Council     369090
    - Season_2                               369092
    - Method_6                               369094
    - Council_Darebin_City_Council           369095
    - Car                                    369100
    - Council_Yarra_Ranges_Shire_Council     369106
    - Council_Hobsons_Bay_City_Council       369110
    - Council_Whittlesea_City_Council        369111
    - Council_Maribyrnong_City_Council       369114
    - Council_Maroondah_City_Council         369120
    - Council_Bayside_City_Council           369131
    - Council_Hume_City_Council              369141
    - Method_3                               369155
    - Council_Cardinia_Shire_Council         369168
    - Council_Brimbank_City_Council          369169
    - Council_Melton_City_Council            369170
    - Council_Port_Phillip_City_Council      369182
    - Council_Kingston_City_Council          369210
    - Council_Yarra_City_Council             369248
    - Council_Wyndham_City_Council           369261
    - Council_Greater_Dandenong_City_Council 369265
    - Distance                               369278
    - Council_Stonnington_City_Council       369348
    - Lattitude                              369353
    - Landsize                               369361
    - Council_Casey_City_Council             369407
    - Council_Frankston_City_Council         369429
    - Council_Boroondara_City_Council        369562
    - type_2                                 369571
    - Rooms                                  370128
    - Bathroom                               370253
    - type_3                                 371039

    Step:  AIC=369082.9
    Price ~ Council_Bayside_City_Council + Council_Boroondara_City_Council +
        Council_Brimbank_City_Council + Council_Cardinia_Shire_Council +
        Council_Casey_City_Council + Council_Darebin_City_Council +
        Council_Frankston_City_Council + Council_Glen_Eira_City_Council +
        Council_Greater_Dandenong_City_Council + Council_Hobsons_Bay_City_Council +
        Council_Hume_City_Council + Council_Kingston_City_Council +
        Council_Knox_City_Council + Council_Manningham_City_Council +
        Council_Maribyrnong_City_Council + Council_Maroondah_City_Council + Council_Melton_City_Council +
        Council_Monash_City_Council + Council_Moonee_Valley_City_Council +
        Council_Moorabool_Shire_Council + Council_Moreland_City_Council +
        Council_Nillumbik_Shire_Council + Council_Port_Phillip_City_Council +
        Council_Stonnington_City_Council + Council_Whitehorse_City_Council +
        Council_Whittlesea_City_Council + Council_Wyndham_City_Council +
        Council_Yarra_City_Council + Council_Yarra_Ranges_Shire_Council +
        Season_2 + Season_3 + Method_3 + Method_6 + Method_8 + type_2 +
        type_3 + Rooms + Distance + Bathroom + Car + Landsize + Lattitude +
        Longtitude

                                             Df       Sum of Sq              RSS
    - Council_Glen_Eira_City_Council          1    207271919913 2066833170868535
    - Season_3                                1    229744635454 2066855643584076
    <none>                                                      2066625898948622
    - Longtitude                              1    363173657105 2066989072605727
    - Council_Whitehorse_City_Council         1    433214767536 2067059113716158
    - Council_Moorabool_Shire_Council         1    437457346240 2067063356294862
    + Season_4                                1     78369543839 2066547529404783
    + Council_Macedon_Ranges_Shire_Council    1     50712325767 2066575186622855
    + Council_Mitchell_Shire_Council          1     20544460865 2066605354487757
    + Method_4                                1       473525402 2066625425423220
    - Council_Moreland_City_Council           1    840366823619 2067466265772241
    - Method_8                                1    959429177030 2067585328125652
    - Council_Monash_City_Council             1    961383999788 2067587282948411
    - Council_Nillumbik_Shire_Council         1    963278709916 2067589177658538
    - Council_Manningham_City_Council         1    966164405663 2067592063354286
    - Council_Moonee_Valley_City_Council      1   1087401711463 2067713300660085
    - Method_6                                1   1627968839048 2068253867787670
    - Council_Darebin_City_Council            1   1795547934134 2068421446882756
    - Season_2                                1   1924486484320 2068550385432943
    - Car                                     1   2580252260042 2069206151208664
    - Council_Yarra_Ranges_Shire_Council      1   3442237373051 2070068136321674
    - Council_Hobsons_Bay_City_Council        1   3927869478202 2070553768426824
    - Council_Whittlesea_City_Council         1   4080756616289 2070706655564912
    - Council_Maribyrnong_City_Council        1   4547228266609 2071173127215232
    - Council_Maroondah_City_Council          1   5373420268044 2071999319216667
    - Council_Bayside_City_Council            1   7083858772394 2073709757721016
    - Council_Hume_City_Council               1   8421565160127 2075047464108749
    - Method_3                                1  10514825652608 2077140724601230
    - Council_Cardinia_Shire_Council          1  12434229421287 2079060128369909
    - Council_Brimbank_City_Council           1  12467099765773 2079092998714395
    - Council_Melton_City_Council             1  12607597600132 2079233496548754
    - Council_Port_Phillip_City_Council       1  14416680809732 2081042579758354
    - Council_Kingston_City_Council           1  18481110833294 2085107009781916
    - Council_Knox_City_Council               1  18669863731462 2085295762680084
    - Council_Yarra_City_Council              1  23994577885421 2090620476834043
    - Council_Wyndham_City_Council            1  25888239347616 2092514138296239
    - Council_Greater_Dandenong_City_Council  1  26496297049022 2093122195997644
    - Distance                                1  28376853731774 2095002752680396
    - Council_Stonnington_City_Council        1  38642454499428 2105268353448051
    - Lattitude                               1  39341874925490 2105967773874113
    - Landsize                                1  40564207800020 2107190106748643
    - Council_Casey_City_Council              1  47284315521246 2113910214469868
    - Council_Frankston_City_Council          1  50561924161656 2117187823110278
    - Council_Boroondara_City_Council         1  70212536818974 2136838435767596
    - type_2                                  1  71652329501551 2138278228450173
    - Rooms                                   6 157616485518750 2224242384467373
    - Bathroom                                8 177718239125468 2244344138074090
    - type_3                                  1 301686770292193 2368312669240815
                                                AIC
    - Council_Glen_Eira_City_Council         369082
    - Season_3                               369082
    <none>                                   369083
    - Longtitude                             369083
    - Council_Whitehorse_City_Council        369084
    - Council_Moorabool_Shire_Council        369084
    + Season_4                               369084
    + Council_Macedon_Ranges_Shire_Council   369085
    + Council_Mitchell_Shire_Council         369085
    + Method_4                               369085
    - Council_Moreland_City_Council          369087
    - Method_8                               369088
    - Council_Monash_City_Council            369088
    - Council_Nillumbik_Shire_Council        369088
    - Council_Manningham_City_Council        369088
    - Council_Moonee_Valley_City_Council     369088
    - Method_6                               369092
    - Council_Darebin_City_Council           369093
    - Season_2                               369094
    - Car                                    369099
    - Council_Yarra_Ranges_Shire_Council     369105
    - Council_Hobsons_Bay_City_Council       369108
    - Council_Whittlesea_City_Council        369109
    - Council_Maribyrnong_City_Council       369112
    - Council_Maroondah_City_Council         369118
    - Council_Bayside_City_Council           369130
    - Council_Hume_City_Council              369139
    - Method_3                               369154
    - Council_Cardinia_Shire_Council         369167
    - Council_Brimbank_City_Council          369167
    - Council_Melton_City_Council            369168
    - Council_Port_Phillip_City_Council      369181
    - Council_Kingston_City_Council          369209
    - Council_Knox_City_Council              369210
    - Council_Yarra_City_Council             369247
    - Council_Wyndham_City_Council           369260
    - Council_Greater_Dandenong_City_Council 369264
    - Distance                               369277
    - Council_Stonnington_City_Council       369347
    - Lattitude                              369352
    - Landsize                               369360
    - Council_Casey_City_Council             369406
    - Council_Frankston_City_Council         369428
    - Council_Boroondara_City_Council        369561
    - type_2                                 369570
    - Rooms                                  370126
    - Bathroom                               370252
    - type_3                                 371038

    Step:  AIC=369082.3
    Price ~ Council_Bayside_City_Council + Council_Boroondara_City_Council +
        Council_Brimbank_City_Council + Council_Cardinia_Shire_Council +
        Council_Casey_City_Council + Council_Darebin_City_Council +
        Council_Frankston_City_Council + Council_Greater_Dandenong_City_Council +
        Council_Hobsons_Bay_City_Council + Council_Hume_City_Council +
        Council_Kingston_City_Council + Council_Knox_City_Council +
        Council_Manningham_City_Council + Council_Maribyrnong_City_Council +
        Council_Maroondah_City_Council +
        Council_Melton_City_Council + Council_Monash_City_Council +
        Council_Moonee_Valley_City_Council + Council_Moorabool_Shire_Council +
        Council_Moreland_City_Council + Council_Nillumbik_Shire_Council +
        Council_Port_Phillip_City_Council + Council_Stonnington_City_Council +
        Council_Whitehorse_City_Council + Council_Whittlesea_City_Council +
        Council_Wyndham_City_Council + Council_Yarra_City_Council +
        Council_Yarra_Ranges_Shire_Council + Season_2 + Season_3 +
        Method_3 + Method_6 + Method_8 + type_2 + type_3 + Rooms +
        Distance + Bathroom + Car + Landsize + Lattitude + Longtitude

                                             Df       Sum of Sq              RSS
    - Longtitude                              1    224722196779 2067057893065314
    - Season_3                                1    230153664819 2067063324533354
    <none>                                                      2066833170868536
    + Council_Glen_Eira_City_Council          1    207271919913 2066625898948622
    - Council_Moorabool_Shire_Council         1    402434453690 2067235605322225
    + Season_4                                1     76791549778 2066756379318758
    + Council_Macedon_Ranges_Shire_Council    1     58234431746 2066774936436790
    + Council_Mitchell_Shire_Council          1     11920122463 2066821250746072
    + Method_4                                1       458114122 2066832712754413
    - Council_Monash_City_Council             1    877744683578 2067710915552114
    - Council_Whitehorse_City_Council         1    878179826606 2067711350695142
    - Method_8                                1    976102219918 2067809273088454
    - Council_Nillumbik_Shire_Council         1   1046452805946 2067879623674482
    - Council_Manningham_City_Council         1   1574396789918 2068407567658454
    - Method_6                                1   1647278362402 2068480449230938
    - Council_Moreland_City_Council           1   1754795417736 2068587966286272
    - Season_2                                1   1934695826047 2068767866694582
    - Council_Moonee_Valley_City_Council      1   2274444528594 2069107615397129
    - Car                                     1   2549462564298 2069382633432833
    - Council_Darebin_City_Council            1   3160297184509 2069993468053045
    - Council_Yarra_Ranges_Shire_Council      1   3450873676694 2070284044545230
    - Council_Whittlesea_City_Council         1   4204318711129 2071037489579664
    - Council_Maroondah_City_Council          1   5169114938224 2072002285806760
    - Council_Hobsons_Bay_City_Council        1   5172427989930 2072005598858466
    - Council_Maribyrnong_City_Council        1   5488302290512 2072321473159047
    - Council_Hume_City_Council               1   8215310788546 2075048481657082
    - Method_3                                1  10563581002149 2077396751870684
    - Council_Cardinia_Shire_Council          1  12285404388914 2079118575257450
    - Council_Melton_City_Council             1  12939415912158 2079772586780694
    - Council_Brimbank_City_Council           1  15138827786162 2081971998654697
    - Council_Knox_City_Council               1  19480814730124 2086313985598659
    - Distance                                1  31062426483013 2097895597351549
    - Council_Bayside_City_Council            1  31175881545535 2098009052414070
    - Council_Greater_Dandenong_City_Council  1  32645687769566 2099478858638102
    - Council_Wyndham_City_Council            1  35399467807457 2102232638675993
    - Council_Port_Phillip_City_Council       1  38830545993218 2105663716861754
    - Council_Kingston_City_Council           1  39385201336899 2106218372205435
    - Landsize                                1  40554699240174 2107387870108709
    - Council_Yarra_City_Council              1  48333579764412 2115166750632947
    - Council_Casey_City_Council              1  55979663381643 2122812834250178
    - type_2                                  1  71812693038019 2138645863906554
    - Council_Frankston_City_Council          1  83053602966906 2149886773835442
    - Council_Stonnington_City_Council        1  92100177573308 2158933348441843
    - Lattitude                               1  97531156164307 2164364327032842
    - Rooms                                   6 157541399532939 2224374570401475
    - Council_Boroondara_City_Council         1 165270734031223 2232103904899758
    - Bathroom                                8 177546289299032 2244379460167568
    - type_3                                  1 302176275544608 2369009446413144
                                                AIC
    - Longtitude                             369082
    - Season_3                               369082
    <none>                                   369082
    + Council_Glen_Eira_City_Council         369083
    - Council_Moorabool_Shire_Council        369083
    + Season_4                               369084
    + Council_Macedon_Ranges_Shire_Council   369084
    + Council_Mitchell_Shire_Council         369084
    + Method_4                               369084
    - Council_Monash_City_Council            369086
    - Council_Whitehorse_City_Council        369086
    - Method_8                               369087
    - Council_Nillumbik_Shire_Council        369088
    - Council_Manningham_City_Council        369091
    - Method_6                               369092
    - Council_Moreland_City_Council          369093
    - Season_2                               369094
    - Council_Moonee_Valley_City_Council     369096
    - Car                                    369098
    - Council_Darebin_City_Council           369102
    - Council_Yarra_Ranges_Shire_Council     369104
    - Council_Whittlesea_City_Council        369109
    - Council_Maroondah_City_Council         369116
    - Council_Hobsons_Bay_City_Council       369116
    - Council_Maribyrnong_City_Council       369118
    - Council_Hume_City_Council              369137
    - Method_3                               369154
    - Council_Cardinia_Shire_Council         369165
    - Council_Melton_City_Council            369170
    - Council_Brimbank_City_Council          369185
    - Council_Knox_City_Council              369215
    - Distance                               369295
    - Council_Bayside_City_Council           369295
    - Council_Greater_Dandenong_City_Council 369305
    - Council_Wyndham_City_Council           369324
    - Council_Port_Phillip_City_Council      369348
    - Council_Kingston_City_Council          369351
    - Landsize                               369359
    - Council_Yarra_City_Council             369412
    - Council_Casey_City_Council             369464
    - type_2                                 369571
    - Council_Frankston_City_Council         369646
    - Council_Stonnington_City_Council       369706
    - Lattitude                              369742
    - Rooms                                  370125
    - Council_Boroondara_City_Council        370185
    - Bathroom                               370250
    - type_3                                 371040

    Step:  AIC=369081.9
    Price ~ Council_Bayside_City_Council + Council_Boroondara_City_Council +
        Council_Brimbank_City_Council + Council_Cardinia_Shire_Council +
        Council_Casey_City_Council + Council_Darebin_City_Council +
        Council_Frankston_City_Council + Council_Greater_Dandenong_City_Council +
        Council_Hobsons_Bay_City_Council + Council_Hume_City_Council +
        Council_Kingston_City_Council + Council_Knox_City_Council +
        Council_Manningham_City_Council + Council_Maribyrnong_City_Council +
        Council_Maroondah_City_Council  +
        Council_Melton_City_Council + Council_Monash_City_Council +
        Council_Moonee_Valley_City_Council + Council_Moorabool_Shire_Council +
        Council_Moreland_City_Council + Council_Nillumbik_Shire_Council +
        Council_Port_Phillip_City_Council + Council_Stonnington_City_Council +
        Council_Whitehorse_City_Council + Council_Whittlesea_City_Council +
        Council_Wyndham_City_Council + Council_Yarra_City_Council +
        Council_Yarra_Ranges_Shire_Council + Season_2 + Season_3 +
        Method_3 + Method_6 + Method_8 + type_2 + type_3 + Rooms +
        Distance + Bathroom + Car + Landsize + Lattitude

                                             Df       Sum of Sq              RSS
    - Season_3                                1    229504140183 2067287397205497
    <none>                                                      2067057893065314
    - Council_Moorabool_Shire_Council         1    316012171957 2067373905237272
    + Longtitude                              1    224722196779 2066833170868536
    + Season_4                                1     77787856487 2066980105208828
    + Council_Glen_Eira_City_Council          1     68820459588 2066989072605726
    + Council_Mitchell_Shire_Council          1     12755156402 2067045137908913
    + Council_Macedon_Ranges_Shire_Council    1      4481860586 2067053411204729
    + Method_4                                1       329475808 2067057563589507
    - Council_Whitehorse_City_Council         1    678435828553 2067736328893868
    - Method_8                                1    970009563031 2068027902628346
    - Council_Nillumbik_Shire_Council         1   1199939755959 2068257832821273
    - Council_Monash_City_Council             1   1282672024655 2068340565089970
    - Council_Manningham_City_Council         1   1350314205331 2068408207270646
    - Method_6                                1   1648291902285 2068706184967600
    - Season_2                                1   1947818461910 2069005711527224
    - Car                                     1   2578754750488 2069636647815803
    - Council_Moreland_City_Council           1   3426480061147 2070484373126462
    - Council_Darebin_City_Council            1   4008847927436 2071066740992751
    - Council_Whittlesea_City_Council         1   4259019527340 2071316912592655
    - Council_Yarra_Ranges_Shire_Council      1   4627656241647 2071685549306961
    - Council_Moonee_Valley_City_Council      1   6034896780845 2073092789846159
    - Council_Hobsons_Bay_City_Council        1   7628959565344 2074686852630658
    - Council_Maribyrnong_City_Council        1   8130561548597 2075188454613911
    - Council_Maroondah_City_Council          1   8976047341239 2076033940406554
    - Council_Hume_City_Council               1   9012310370526 2076070203435840
    - Method_3                                1  10555269888884 2077613162954198
    - Council_Cardinia_Shire_Council          1  14592726876016 2081650619941331
    - Council_Melton_City_Council             1  21467282308977 2088525175374291
    - Council_Knox_City_Council               1  28136906308036 2095194799373351
    - Distance                                1  31430008000389 2098487901065704
    - Council_Brimbank_City_Council           1  33366797231676 2100424690296990
    - Council_Bayside_City_Council            1  35549095829236 2102606988894551
    - Council_Greater_Dandenong_City_Council  1  36875202263571 2103933095328886
    - Council_Kingston_City_Council           1  39953368375922 2107011261441237
    - Landsize                                1  40455356432059 2107513249497374
    - Council_Port_Phillip_City_Council       1  50145078011557 2117202971076872
    - Council_Yarra_City_Council              1  55761627271551 2122819520336866
    - Council_Casey_City_Council              1  69898625225689 2136956518291004
    - type_2                                  1  71713898788858 2138771791854173
    - Council_Wyndham_City_Council            1  77068665193020 2144126558258334
    - Council_Frankston_City_Council          1  85686127561536 2152744020626851
    - Council_Stonnington_City_Council        1  96672146644955 2163730039710269
    - Lattitude                               1  98284725402106 2165342618467421
    - Rooms                                   6 157444304850790 2224502197916104
    - Council_Boroondara_City_Council         1 165440684227749 2232498577293064
    - Bathroom                                8 177418682287228 2244476575352543
    - type_3                                  1 302230942208734 2369288835274048
                                                AIC
    - Season_3                               369081
    <none>                                   369082
    - Council_Moorabool_Shire_Council        369082
    + Longtitude                             369082
    + Season_4                               369083
    + Council_Glen_Eira_City_Council         369083
    + Council_Mitchell_Shire_Council         369084
    + Council_Macedon_Ranges_Shire_Council   369084
    + Method_4                               369084
    - Council_Whitehorse_City_Council        369085
    - Method_8                               369087
    - Council_Nillumbik_Shire_Council        369088
    - Council_Monash_City_Council            369089
    - Council_Manningham_City_Council        369089
    - Method_6                               369091
    - Season_2                               369093
    - Car                                    369098
    - Council_Moreland_City_Council          369104
    - Council_Darebin_City_Council           369108
    - Council_Whittlesea_City_Council        369109
    - Council_Yarra_Ranges_Shire_Council     369112
    - Council_Moonee_Valley_City_Council     369122
    - Council_Hobsons_Bay_City_Council       369133
    - Council_Maribyrnong_City_Council       369136
    - Council_Maroondah_City_Council         369142
    - Council_Hume_City_Council              369142
    - Method_3                               369153
    - Council_Cardinia_Shire_Council         369181
    - Council_Melton_City_Council            369228
    - Council_Knox_City_Council              369274
    - Distance                               369297
    - Council_Brimbank_City_Council          369310
    - Council_Bayside_City_Council           369325
    - Council_Greater_Dandenong_City_Council 369334
    - Council_Kingston_City_Council          369355
    - Landsize                               369358
    - Council_Port_Phillip_City_Council      369424
    - Council_Yarra_City_Council             369462
    - Council_Casey_City_Council             369557
    - type_2                                 369570
    - Council_Wyndham_City_Council           369606
    - Council_Frankston_City_Council         369663
    - Council_Stonnington_City_Council       369736
    - Lattitude                              369747
    - Rooms                                  370124
    - Council_Boroondara_City_Council        370186
    - Bathroom                               370248
    - type_3                                 371040

    Step:  AIC=369081.5
    Price ~ Council_Bayside_City_Council + Council_Boroondara_City_Council +
        Council_Brimbank_City_Council + Council_Cardinia_Shire_Council +
        Council_Casey_City_Council + Council_Darebin_City_Council +
        Council_Frankston_City_Council + Council_Greater_Dandenong_City_Council +
        Council_Hobsons_Bay_City_Council + Council_Hume_City_Council +
        Council_Kingston_City_Council + Council_Knox_City_Council +
        Council_Manningham_City_Council + Council_Maribyrnong_City_Council +
        Council_Maroondah_City_Council + Council_/images/melbourne_City_Council +
        Council_Melton_City_Council + Council_Monash_City_Council +
        Council_Moonee_Valley_City_Council + Council_Moorabool_Shire_Council +
        Council_Moreland_City_Council + Council_Nillumbik_Shire_Council +
        Council_Port_Phillip_City_Council + Council_Stonnington_City_Council +
        Council_Whitehorse_City_Council + Council_Whittlesea_City_Council +
        Council_Wyndham_City_Council + Council_Yarra_City_Council +
        Council_Yarra_Ranges_Shire_Council + Season_2 + Method_3 +
        Method_6 + Method_8 + type_2 + type_3 + Rooms + Distance +
        Bathroom + Car + Landsize + Lattitude

                                             Df       Sum of Sq              RSS
    <none>                                                      2067287397205497
    - Council_Moorabool_Shire_Council         1    316327861500 2067603725066998
    + Season_3                                1    229504140183 2067057893065314
    + Longtitude                              1    224072672143 2067063324533354
    + Season_4                                1    183219127444 2067104178078054
    + Council_Glen_Eira_City_Council          1     69166279920 2067218230925578
    + Council_Mitchell_Shire_Council          1     11600629021 2067275796576476
    + Council_Macedon_Ranges_Shire_Council    1      5258787892 2067282138417605
    + Method_4                                1       351444633 2067287045760864
    - Council_Whitehorse_City_Council         1    650873185288 2067938270390786
    - Method_8                                1    953705420066 2068241102625564
    - Council_Nillumbik_Shire_Council         1   1224064744921 2068511461950418
    - Council_Monash_City_Council             1   1321078390379 2068608475595876
    - Council_Manningham_City_Council         1   1336546311560 2068623943517057
    - Method_6                                1   1641465039082 2068928862244579
    - Season_2                                1   1801124076313 2069088521281810
    - Car                                     1   2527402649700 2069814799855198
    - Council_Moreland_City_Council           1   3441600691236 2070728997896734
    - Council_Darebin_City_Council            1   3991325970075 2071278723175572
    - Council_Whittlesea_City_Council         1   4325924600560 2071613321806057
    - Council_Yarra_Ranges_Shire_Council      1   4648981284406 2071936378489903
    - Council_Moonee_Valley_City_Council      1   6034085419496 2073321482624993
    - Council_Hobsons_Bay_City_Council        1   7618002749218 2074905399954716
    - Council_Maribyrnong_City_Council        1   8112463002428 2075399860207925
    - Council_Hume_City_Council               1   9068468821274 2076355866026772
    - Council_Maroondah_City_Council          1   9106996829592 2076394394035090
    - Method_3                                1  10544342396748 2077831739602245
    - Council_Cardinia_Shire_Council          1  14620199627462 2081907596832959
    - Council_Melton_City_Council             1  21577118230942 2088864515436439
    - Council_Knox_City_Council               1  28312185980379 2095599583185876
    - Distance                                1  31461885519240 2098749282724737
    - Council_Brimbank_City_Council           1  33395488736497 2100682885941994
    - Council_Bayside_City_Council            1  35453778095810 2102741175301308
    - Council_Greater_Dandenong_City_Council  1  37055656005140 2104343053210637
    - Council_Kingston_City_Council           1  40295402305829 2107582799511326
    - Landsize                                1  40495170963294 2107782568168792
    - Council_Port_Phillip_City_Council       1  50157708349637 2117445105555134
    - Council_Yarra_City_Council              1  55660501106600 2122947898312098
    - Council_Casey_City_Council              1  70121992621949 2137409389827446
    - type_2                                  1  71665785249062 2138953182454559
    - Council_Wyndham_City_Council            1  77564013595650 2144851410801148
    - Council_Frankston_City_Council          1  85955456502886 2153242853708384
    - Council_Stonnington_City_Council        1  96632459700594 2163919856906091
    - Lattitude                               1  98397247216367 2165684644421864
    - Rooms                                   6 157431506150090 2224718903355588
    - Council_Boroondara_City_Council         1 165307343261364 2232594740466861
    - Bathroom                                8 177332715466038 2244620112671535
    - type_3                                  1 302299823565276 2369587220770773
                                                AIC
    <none>                                   369081
    - Council_Moorabool_Shire_Council        369082
    + Season_3                               369082
    + Longtitude                             369082
    + Season_4                               369082
    + Council_Glen_Eira_City_Council         369083
    + Council_Mitchell_Shire_Council         369083
    + Council_Macedon_Ranges_Shire_Council   369083
    + Method_4                               369083
    - Council_Whitehorse_City_Council        369084
    - Method_8                               369086
    - Council_Nillumbik_Shire_Council        369088
    - Council_Monash_City_Council            369089
    - Council_Manningham_City_Council        369089
    - Method_6                               369091
    - Season_2                               369092
    - Car                                    369097
    - Council_Moreland_City_Council          369103
    - Council_Darebin_City_Council           369107
    - Council_Whittlesea_City_Council        369109
    - Council_Yarra_Ranges_Shire_Council     369112
    - Council_Moonee_Valley_City_Council     369121
    - Council_Hobsons_Bay_City_Council       369132
    - Council_Maribyrnong_City_Council       369136
    - Council_Hume_City_Council              369142
    - Council_Maroondah_City_Council         369143
    - Method_3                               369153
    - Council_Cardinia_Shire_Council         369181
    - Council_Melton_City_Council            369229
    - Council_Knox_City_Council              369275
    - Distance                               369296
    - Council_Brimbank_City_Council          369310
    - Council_Bayside_City_Council           369324
    - Council_Greater_Dandenong_City_Council 369335
    - Council_Kingston_City_Council          369357
    - Landsize                               369358
    - Council_Port_Phillip_City_Council      369424
    - Council_Yarra_City_Council             369461
    - Council_Casey_City_Council             369559
    - type_2                                 369569
    - Council_Wyndham_City_Council           369608
    - Council_Frankston_City_Council         369665
    - Council_Stonnington_City_Council       369736
    - Lattitude                              369747
    - Rooms                                  370123
    - Council_Boroondara_City_Council        370184
    - Bathroom                               370247
    - type_3                                 371039




    Call:
    lm(formula = Price ~ Council_Bayside_City_Council + Council_Boroondara_City_Council +
        Council_Brimbank_City_Council + Council_Cardinia_Shire_Council +
        Council_Casey_City_Council + Council_Darebin_City_Council +
        Council_Frankston_City_Council + Council_Greater_Dandenong_City_Council +
        Council_Hobsons_Bay_City_Council + Council_Hume_City_Council +
        Council_Kingston_City_Council + Council_Knox_City_Council +
        Council_Manningham_City_Council + Council_Maribyrnong_City_Council +
        Council_Maroondah_City_Council +
        Council_Melton_City_Council + Council_Monash_City_Council +
        Council_Moonee_Valley_City_Council + Council_Moorabool_Shire_Council +
        Council_Moreland_City_Council + Council_Nillumbik_Shire_Council +
        Council_Port_Phillip_City_Council + Council_Stonnington_City_Council +
        Council_Whitehorse_City_Council + Council_Whittlesea_City_Council +
        Council_Wyndham_City_Council + Council_Yarra_City_Council +
        Council_Yarra_Ranges_Shire_Council + Season_2 + Method_3 +
        Method_6 + Method_8 + type_2 + type_3 + Rooms + Distance +
        Bathroom + Car + Landsize + Lattitude, data = train_data)

    Residuals:
         Min       1Q   Median       3Q      Max
    -1536403  -219249   -36930   158319  7779409

    Coefficients:
                                            Estimate Std. Error t value
    (Intercept)                            -81843808    3149733 -25.984
    Council_Bayside_City_Council              307669      19642  15.664
    Council_Boroondara_City_Council           487024      14399  33.824
    Council_Brimbank_City_Council            -282023      18551 -15.203
    Council_Cardinia_Shire_Council          -1228850     122165 -10.059
    Council_Casey_City_Council              -1175009      53338 -22.029
    Council_Darebin_City_Council               84914      16156   5.256
    Council_Frankston_City_Council          -1268328      52002 -24.390
    Council_Greater_Dandenong_City_Council   -724034      45212 -16.014
    Council_Hobsons_Bay_City_Council         -147832      20360  -7.261
    Council_Hume_City_Council                -207032      26134  -7.922
    Council_Kingston_City_Council            -440815      26397 -16.699
    Council_Knox_City_Council                -527647      37695 -13.998
    Council_Manningham_City_Council            65894      21666   3.041
    Council_Maribyrnong_City_Council         -134907      18005  -7.493
    Council_Maroondah_City_Council           -274031      34517  -7.939
    Council_Melton_City_Council              -528422      43242 -12.220
    Council_Monash_City_Council               -61402      20307  -3.024
    Council_Moonee_Valley_City_Council        111323      17227   6.462
    Council_Moorabool_Shire_Council          -563033     380532  -1.480
    Council_Moreland_City_Council              82768      16959   4.880
    Council_Nillumbik_Shire_Council          -259467      89147  -2.911
    Council_Port_Phillip_City_Council         349334      18750  18.631
    Council_Stonnington_City_Council          506152      19572  25.860
    Council_Whitehorse_City_Council            60266      28396   2.122
    Council_Whittlesea_City_Council          -164969      30150  -5.472
    Council_Wyndham_City_Council             -793865      34264 -23.169
    Council_Yarra_City_Council                378024      19261  19.627
    Council_Yarra_Ranges_Shire_Council       -487071      85870  -5.672
    Season_2                                  -23605       6686  -3.531
    Method_3                                   86513      10127   8.542
    Method_6                                   43952      13040   3.370
    Method_8                                   36325      14139   2.569
    type_2                                   -274298      12317 -22.270
    type_3                                   -471962      10318 -45.740
    Rooms0.166666666666667                    301497      16678  18.078
    Rooms0.333333333333333                    466247      18170  25.660
    Rooms0.5                                  629397      20422  30.819
    Rooms0.666666666666667                    725029      25939  27.951
    Rooms0.833333333333333                    577212      51381  11.234
    Rooms1                                    541269     116866   4.632
    Distance                                  176900      11988  14.756
    Bathroom0.111111111111111                  58672      36945   1.588
    Bathroom0.222222222222222                 331611      95205   3.483
    Bathroom0.333333333333333                 190187      37480   5.074
    Bathroom0.444444444444444                 423364      39785  10.641
    Bathroom0.666666666666667                1165888      53486  21.798
    Bathroom0.777777777777778                 936445      84841  11.038
    Bathroom0.888888888888889                 589048     223111   2.640
    Bathroom1                                2032443     382820   5.309
    Car                                       291636      69732   4.182
    Landsize                                  362515      21655  16.741
    Lattitude                               -2171290      83206 -26.095
                                                       Pr(>|t|)    
    (Intercept)                            < 0.0000000000000002 ***
    Council_Bayside_City_Council           < 0.0000000000000002 ***
    Council_Boroondara_City_Council        < 0.0000000000000002 ***
    Council_Brimbank_City_Council          < 0.0000000000000002 ***
    Council_Cardinia_Shire_Council         < 0.0000000000000002 ***
    Council_Casey_City_Council             < 0.0000000000000002 ***
    Council_Darebin_City_Council            0.00000014956275813 ***
    Council_Frankston_City_Council         < 0.0000000000000002 ***
    Council_Greater_Dandenong_City_Council < 0.0000000000000002 ***
    Council_Hobsons_Bay_City_Council        0.00000000000040415 ***
    Council_Hume_City_Council               0.00000000000000251 ***
    Council_Kingston_City_Council          < 0.0000000000000002 ***
    Council_Knox_City_Council              < 0.0000000000000002 ***
    Council_Manningham_City_Council                    0.002359 **
    Council_Maribyrnong_City_Council        0.00000000000007131 ***
    Council_Maroondah_City_Council          0.00000000000000219 ***
    Council_Melton_City_Council            < 0.0000000000000002 ***
    Council_Monash_City_Council                        0.002501 **
    Council_Moonee_Valley_City_Council      0.00000000010654025 ***
    Council_Moorabool_Shire_Council                    0.139004    
    Council_Moreland_City_Council           0.00000107020047843 ***
    Council_Nillumbik_Shire_Council                    0.003613 **
    Council_Port_Phillip_City_Council      < 0.0000000000000002 ***
    Council_Stonnington_City_Council       < 0.0000000000000002 ***
    Council_Whitehorse_City_Council                    0.033823 *  
    Council_Whittlesea_City_Council         0.00000004535207426 ***
    Council_Wyndham_City_Council           < 0.0000000000000002 ***
    Council_Yarra_City_Council             < 0.0000000000000002 ***
    Council_Yarra_Ranges_Shire_Council      0.00000001436905008 ***
    Season_2                                           0.000416 ***
    Method_3                               < 0.0000000000000002 ***
    Method_6                                           0.000752 ***
    Method_8                                           0.010206 *  
    type_2                                 < 0.0000000000000002 ***
    type_3                                 < 0.0000000000000002 ***
    Rooms0.166666666666667                 < 0.0000000000000002 ***
    Rooms0.333333333333333                 < 0.0000000000000002 ***
    Rooms0.5                               < 0.0000000000000002 ***
    Rooms0.666666666666667                 < 0.0000000000000002 ***
    Rooms0.833333333333333                 < 0.0000000000000002 ***
    Rooms1                                  0.00000366168746462 ***
    Distance                               < 0.0000000000000002 ***
    Bathroom0.111111111111111                          0.112286    
    Bathroom0.222222222222222                          0.000497 ***
    Bathroom0.333333333333333               0.00000039356090691 ***
    Bathroom0.444444444444444              < 0.0000000000000002 ***
    Bathroom0.666666666666667              < 0.0000000000000002 ***
    Bathroom0.777777777777778              < 0.0000000000000002 ***
    Bathroom0.888888888888889                          0.008296 **
    Bathroom1                               0.00000011179149185 ***
    Car                                     0.00002903447746647 ***
    Landsize                               < 0.0000000000000002 ***
    Lattitude                              < 0.0000000000000002 ***
    ---

    Residual standard error: 380100 on 14307 degrees of freedom
    Multiple R-squared:  0.6328,	Adjusted R-squared:  0.6314
    F-statistic: 465.1 on 53 and 14307 DF,  p-value: < 0.00000000000000022


```R
predicted_ys <- predict(step_model,newdata=test_data)
observed_ys <- test_data$Price
lm.SSE <- sum((observed_ys - predicted_ys) ^ 2)
lm.SST <- sum((observed_ys - mean(observed_ys)) ^ 2)
lm.r2 <- 1 - lm.SSE/lm.SST
lm.r2
vif(step_model)
```


0.628092787466896

```R
############################################################
############-----------Lasso regression
############################################################

x <- model.matrix(Price~.,data=train_data)
x_train <- x[,-1]
y_train <- train_data$Price
crossval <-  cv.glmnet(x = x_train, y = y_train)
plot(crossval)
penalty <- crossval$lambda.min
penalty
fit1 <-glmnet(x = x_train, y = y_train, alpha = 1, lambda = penalty ) #estimate the model with that
c <- coef(fit1)
inds<-which(c!=0)
variables<-row.names(c)[inds]
variables<-variables[variables %ni% '(Intercept)']
variables
summary(fit1)

x_1 <-model.matrix(Price~.,data=test_data)
x_test <- x_1[,-1]
y_test <- test_data$Price

summary(x_1)
predicted_ys <- predict(fit1, s = penalty, newx = x_test)
observed_ys <- test_data$Price
lm.SSE <- sum((observed_ys - predicted_ys) ^ 2)
lm.SST <- sum((observed_ys - mean(observed_ys)) ^ 2)
lm.r2 <- 1 - lm.SSE/lm.SST
lm.r2

```


229.577232222248



              Length Class     Mode   
    a0         1     -none-    numeric
    beta      60     dgCMatrix S4     
    df         1     -none-    numeric
    dim        2     -none-    numeric
    lambda     1     -none-    numeric
    dev.ratio  1     -none-    numeric
    nulldev    1     -none-    numeric
    npasses    1     -none-    numeric
    jerr       1     -none-    numeric
    offset     1     -none-    logical
    call       5     -none-    call   
    nobs       1     -none-    numeric



      (Intercept) Council_Bayside_City_Council Council_Boroondara_City_Council
     Min.   :1    Min.   :0.00000              Min.   :0.00000                
     1st Qu.:1    1st Qu.:0.00000              1st Qu.:0.00000                
     Median :1    Median :0.00000              Median :0.00000                
     Mean   :1    Mean   :0.04988              Mean   :0.06044                
     3rd Qu.:1    3rd Qu.:0.00000              3rd Qu.:0.00000                
     Max.   :1    Max.   :1.00000              Max.   :1.00000                
     Council_Brimbank_City_Council Council_Cardinia_Shire_Council
     Min.   :0.00000               Min.   :0.000000              
     1st Qu.:0.00000               1st Qu.:0.000000              
     Median :0.00000               Median :0.000000              
     Mean   :0.06499               Mean   :0.001787              
     3rd Qu.:0.00000               3rd Qu.:0.000000              
     Max.   :1.00000               Max.   :1.000000              
     Council_Casey_City_Council Council_Darebin_City_Council
     Min.   :0.000000           Min.   :0.00000             
     1st Qu.:0.000000           1st Qu.:0.00000             
     Median :0.000000           Median :0.00000             
     Mean   :0.009586           Mean   :0.07051             
     3rd Qu.:0.000000           3rd Qu.:0.00000             
     Max.   :1.000000           Max.   :1.00000             
     Council_Frankston_City_Council Council_Glen_Eira_City_Council
     Min.   :0.00000                Min.   :0.00000               
     1st Qu.:0.00000                1st Qu.:0.00000               
     Median :0.00000                Median :0.00000               
     Mean   :0.01755                Mean   :0.03802               
     3rd Qu.:0.00000                3rd Qu.:0.00000               
     Max.   :1.00000                Max.   :1.00000               
     Council_Greater_Dandenong_City_Council Council_Hobsons_Bay_City_Council
     Min.   :0.00000                        Min.   :0.00000                 
     1st Qu.:0.00000                        1st Qu.:0.00000                 
     Median :0.00000                        Median :0.00000                 
     Mean   :0.01316                        Mean   :0.02535                 
     3rd Qu.:0.00000                        3rd Qu.:0.00000                 
     Max.   :1.00000                        Max.   :1.00000                 
     Council_Hume_City_Council Council_Kingston_City_Council
     Min.   :0.00000           Min.   :0.00000              
     1st Qu.:0.00000           1st Qu.:0.00000              
     Median :0.00000           Median :0.00000              
     Mean   :0.07701           Mean   :0.03477              
     3rd Qu.:0.00000           3rd Qu.:0.00000              
     Max.   :1.00000           Max.   :1.00000              
     Council_Knox_City_Council Council_Macedon_Ranges_Shire_Council
     Min.   :0.0000            Min.   :0.000000                    
     1st Qu.:0.0000            1st Qu.:0.000000                    
     Median :0.0000            Median :0.000000                    
     Mean   :0.0195            Mean   :0.001462                    
     3rd Qu.:0.0000            3rd Qu.:0.000000                    
     Max.   :1.0000            Max.   :1.000000                    
     Council_Manningham_City_Council Council_Maribyrnong_City_Council
     Min.   :0.00000                 Min.   :0.00000                 
     1st Qu.:0.00000                 1st Qu.:0.00000                 
     Median :0.00000                 Median :0.00000                 
     Mean   :0.03282                 Mean   :0.03558                 
     3rd Qu.:0.00000                 3rd Qu.:0.00000                 
     Max.   :1.00000                 Max.   :1.00000                 
     Council_Maroondah_City_Council
     Min.   :0.00000                               
     1st Qu.:0.00000                               
     Median :0.00000                             
     Mean   :0.02356                            
     3rd Qu.:0.00000                        
     Max.   :1.00000              
     Council_Melton_City_Council Council_Mitchell_Shire_Council
     Min.   :0.00000             Min.   :0.000000              
     1st Qu.:0.00000             1st Qu.:0.000000              
     Median :0.00000             Median :0.000000              
     Mean   :0.01787             Mean   :0.001462              
     3rd Qu.:0.00000             3rd Qu.:0.000000              
     Max.   :1.00000             Max.   :1.000000              
     Council_Monash_City_Council Council_Moonee_Valley_City_Council
     Min.   :0.00000             Min.   :0.00000                   
     1st Qu.:0.00000             1st Qu.:0.00000                   
     Median :0.00000             Median :0.00000                   
     Mean   :0.04289             Mean   :0.04663                   
     3rd Qu.:0.00000             3rd Qu.:0.00000                   
     Max.   :1.00000             Max.   :1.00000                   
     Council_Moorabool_Shire_Council Council_Moreland_City_Council
     Min.   :0.0000000               Min.   :0.00000              
     1st Qu.:0.0000000               1st Qu.:0.00000              
     Median :0.0000000               Median :0.00000              
     Mean   :0.0004874               Mean   :0.04582              
     3rd Qu.:0.0000000               3rd Qu.:0.00000              
     Max.   :1.0000000               Max.   :1.00000              
     Council_Nillumbik_Shire_Council Council_Port_Phillip_City_Council
     Min.   :0.000000                Min.   :0.00000                  
     1st Qu.:0.000000                1st Qu.:0.00000                  
     Median :0.000000                Median :0.00000                  
     Mean   :0.006336                Mean   :0.01673                  
     3rd Qu.:0.000000                3rd Qu.:0.00000                  
     Max.   :1.000000                Max.   :1.00000                  
     Council_Stonnington_City_Council Council_Whitehorse_City_Council
     Min.   :0.00000                  Min.   :0.00000                
     1st Qu.:0.00000                  1st Qu.:0.00000                
     Median :0.00000                  Median :0.00000                
     Mean   :0.01966                  Mean   :0.02405                
     3rd Qu.:0.00000                  3rd Qu.:0.00000                
     Max.   :1.00000                  Max.   :1.00000                
     Council_Whittlesea_City_Council Council_Wyndham_City_Council
     Min.   :0.00000                 Min.   :0.00000             
     1st Qu.:0.00000                 1st Qu.:0.00000             
     Median :0.00000                 Median :0.00000             
     Mean   :0.05589                 Mean   :0.04322             
     3rd Qu.:0.00000                 3rd Qu.:0.00000             
     Max.   :1.00000                 Max.   :1.00000             
     Council_Yarra_City_Council Council_Yarra_Ranges_Shire_Council
     Min.   :0.00000            Min.   :0.000000                  
     1st Qu.:0.00000            1st Qu.:0.000000                  
     Median :0.00000            Median :0.000000                  
     Mean   :0.01998            Mean   :0.005361                  
     3rd Qu.:0.00000            3rd Qu.:0.000000                  
     Max.   :1.00000            Max.   :1.000000                  
        Season_2          Season_3         Season_4         Method_3     
     Min.   :0.00000   Min.   :0.0000   Min.   :0.0000   Min.   :0.0000  
     1st Qu.:0.00000   1st Qu.:0.0000   1st Qu.:0.0000   1st Qu.:0.0000  
     Median :0.00000   Median :0.0000   Median :0.0000   Median :1.0000  
     Mean   :0.05053   Mean   :0.4543   Mean   :0.3209   Mean   :0.6288  
     3rd Qu.:0.00000   3rd Qu.:1.0000   3rd Qu.:1.0000   3rd Qu.:1.0000  
     Max.   :1.00000   Max.   :1.0000   Max.   :1.0000   Max.   :1.0000  
        Method_4           Method_6         Method_8          type_2      
     Min.   :0.000000   Min.   :0.0000   Min.   :0.0000   Min.   :0.0000  
     1st Qu.:0.000000   1st Qu.:0.0000   1st Qu.:0.0000   1st Qu.:0.0000  
     Median :0.000000   Median :0.0000   Median :0.0000   Median :0.0000  
     Mean   :0.007636   Mean   :0.1293   Mean   :0.1126   Mean   :0.0567  
     3rd Qu.:0.000000   3rd Qu.:0.0000   3rd Qu.:0.0000   3rd Qu.:0.0000  
     Max.   :1.000000   Max.   :1.0000   Max.   :1.0000   Max.   :1.0000  
         type_3        Rooms0.166666666666667 Rooms0.333333333333333
     Min.   :0.00000   Min.   :0.000          Min.   :0.0000        
     1st Qu.:0.00000   1st Qu.:0.000          1st Qu.:0.0000        
     Median :0.00000   Median :0.000          Median :1.0000        
     Mean   :0.02242   Mean   :0.103          Mean   :0.5011        
     3rd Qu.:0.00000   3rd Qu.:0.000          3rd Qu.:1.0000        
     Max.   :1.00000   Max.   :1.000          Max.   :1.0000        
        Rooms0.5     Rooms0.666666666666667 Rooms0.833333333333333
     Min.   :0.000   Min.   :0.00000        Min.   :0.000000      
     1st Qu.:0.000   1st Qu.:0.00000        1st Qu.:0.000000      
     Median :0.000   Median :0.00000        Median :0.000000      
     Mean   :0.323   Mean   :0.06109        Mean   :0.006336      
     3rd Qu.:1.000   3rd Qu.:0.00000        3rd Qu.:0.000000      
     Max.   :1.000   Max.   :1.00000        Max.   :1.000000      
         Rooms1             Distance       Bathroom0.111111111111111
     Min.   :0.0000000   Min.   :0.01402   Min.   :0.0000           
     1st Qu.:0.0000000   1st Qu.:0.21495   1st Qu.:0.0000           
     Median :0.0000000   Median :0.38318   Median :0.0000           
     Mean   :0.0009748   Mean   :0.46153   Mean   :0.4029           
     3rd Qu.:0.0000000   3rd Qu.:0.79673   3rd Qu.:1.0000           
     Max.   :1.0000000   Max.   :0.99065   Max.   :1.0000           
     Bathroom0.222222222222222 Bathroom0.333333333333333 Bathroom0.444444444444444
     Min.   :0.000000          Min.   :0.0000            Min.   :0.00000          
     1st Qu.:0.000000          1st Qu.:0.0000            1st Qu.:0.00000          
     Median :0.000000          Median :0.0000            Median :0.00000          
     Mean   :0.002599          Mean   :0.4936            Mean   :0.08627          
     3rd Qu.:0.000000          3rd Qu.:1.0000            3rd Qu.:0.00000          
     Max.   :1.000000          Max.   :1.0000            Max.   :1.00000          
     Bathroom0.666666666666667 Bathroom0.777777777777778 Bathroom0.888888888888889
     Min.   :0.000000          Min.   :0.000000          Min.   :0.0000000        
     1st Qu.:0.000000          1st Qu.:0.000000          1st Qu.:0.0000000        
     Median :0.000000          Median :0.000000          Median :0.0000000        
     Mean   :0.009423          Mean   :0.002599          Mean   :0.0003249        
     3rd Qu.:0.000000          3rd Qu.:0.000000          3rd Qu.:0.0000000        
     Max.   :1.000000          Max.   :1.000000          Max.   :1.0000000        
       Bathroom1      Car             Landsize        Lattitude     
     Min.   :0   Min.   :0.00000   Min.   :0.0000   Min.   :-38.19  
     1st Qu.:0   1st Qu.:0.05556   1st Qu.:0.2810   1st Qu.:-37.87  
     Median :0   Median :0.11111   Median :0.3651   Median :-37.79  
     Mean   :0   Mean   :0.10909   Mean   :0.3574   Mean   :-37.80  
     3rd Qu.:0   3rd Qu.:0.11111   3rd Qu.:0.4429   3rd Qu.:-37.72  
     Max.   :0   Max.   :1.00000   Max.   :0.9959   Max.   :-37.40  
       Longtitude   
     Min.   :144.4  
     1st Qu.:144.9  
     Median :145.0  
     Mean   :145.0  
     3rd Qu.:145.1  
     Max.   :145.5  



0.627307833966311



![png](/images/melbourne/melbourne-housing-price-prediction_23_5.png)



```R
######################################################
#### ---------Decision Tree
######################################################

#test_df <- mel4[,-c(17,18,19,20,22,23,25,26,33,34,35,37)]
#test_df$Postcode <- as.numeric(test_df$Postcode)
set.seed(123)


tree.model <- rpart(Price ~ .,data = train_data)
summary(tree.model)
rpart.plot(tree.model,type = 5,extra=101)

# prune the tree
pfit<- prune(tree.model, cp=tree.model$cptable[which.min(tree.model$cptable[,"xerror"]),"CP"])
rpart.plot(pfit,type = 5,extra=101)


```

    Call:
    rpart(formula = Price ~ ., data = train_data)
      n= 14361

               CP nsplit rel error    xerror       xstd
    1  0.15979222      0 1.0000000 1.0001983 0.02723581
    2  0.08241533      1 0.8402078 0.8405480 0.02358225
    3  0.07875580      2 0.7577924 0.7520467 0.02294884
    4  0.04902198      3 0.6790366 0.6795674 0.02228897
    5  0.04372912      4 0.6300147 0.6306230 0.02173550
    6  0.03620536      5 0.5862855 0.5879856 0.02070802
    7  0.02608737      6 0.5500802 0.5429320 0.01988319
    8  0.02295836      7 0.5239928 0.5195067 0.01916080
    9  0.01600244      8 0.5010345 0.5074235 0.01889009
    10 0.01460007      9 0.4850320 0.4856895 0.01801009
    11 0.01356944     10 0.4704319 0.4790948 0.01780655
    12 0.01199327     11 0.4568625 0.4610549 0.01768474
    13 0.01000000     13 0.4328760 0.4469059 0.01779862

    Variable importance
                               Rooms                        Lattitude
                                  25                               13
     Council_Boroondara_City_Council                           type_3
                                  12                               11
                          Longtitude                         Bathroom
                                  10                                5
                            Distance Council_Stonnington_City_Council
                                   4                                3
        Council_Bayside_City_Council    Council_Brimbank_City_Council
                                   3                                2
    Council_Maribyrnong_City_Council                         Landsize
                                   2                                2
           Council_Hume_City_Council     Council_Darebin_City_Council
                                   2                                1
     Council_Whittlesea_City_Council Council_Hobsons_Bay_City_Council
                                   1                                1
        Council_Wyndham_City_Council      Council_Monash_City_Council
                                   1                                1
     Council_Manningham_City_Council
                                   1

    Node number 1: 14361 observations,    complexity param=0.1597922
      mean=1067729, MSE=3.919743e+11
      left son=2 (10829 obs) right son=3 (3532 obs)
      Primary splits:
          Rooms                           splits as  LLLRRRR, improve=0.15979220, (0 missing)
          type_3                          < 0.5        to the right, improve=0.15459880, (0 missing)
          Bathroom                        splits as  LLRRRRRRR, improve=0.14001180, (0 missing)
          Council_Boroondara_City_Council < 0.5        to the left,  improve=0.09172619, (0 missing)
          Car                             < 0.08333333 to the left,  improve=0.07839241, (0 missing)
      Surrogate splits:
          Bathroom                       splits as  LLRLRRRRR, agree=0.807, adj=0.214, (0 split)
          Car                            < 0.1944444  to the left,  agree=0.755, adj=0.005, (0 split)
          Longtitude                     < 145.3309   to the left,  agree=0.755, adj=0.003, (0 split)
          Landsize                       < 0.4076987  to the left,  agree=0.754, adj=0.002, (0 split)
          Council_Cardinia_Shire_Council < 0.5        to the left,  agree=0.754, adj=0.001, (0 split)

    Node number 2: 10829 observations,    complexity param=0.08241533
      mean=924799.4, MSE=2.288443e+11
      left son=4 (3152 obs) right son=5 (7677 obs)
      Primary splits:
          type_3    < 0.5        to the right, improve=0.18720690, (0 missing)
          Rooms     splits as  LLR----, improve=0.13125350, (0 missing)
          Bathroom  splits as  LL-RRRRR-, improve=0.07908479, (0 missing)
          Lattitude < -37.7562   to the right, improve=0.06078446, (0 missing)
          Car       < 0.08333333 to the left,  improve=0.05404936, (0 missing)
      Surrogate splits:
          Rooms                             splits as  LLR----, agree=0.780, adj=0.245, (0 split)
          Council_Port_Phillip_City_Council < 0.5        to the right, agree=0.716, adj=0.026, (0 split)
          Council_/images/melbourne_City_Council    < 0.5        to the right, agree=0.716, adj=0.023, (0 split)
          Landsize                          < 0.08981788 to the left,  agree=0.714, adj=0.019, (0 split)
          Bathroom                          splits as  LR-RRRRR-, agree=0.714, adj=0.016, (0 split)

    Node number 3: 3532 observations,    complexity param=0.0787558
      mean=1505947, MSE=6.37456e+11
      left son=6 (3019 obs) right son=7 (513 obs)
      Primary splits:
          Council_Boroondara_City_Council < 0.5        to the left,  improve=0.1969037, (0 missing)
          Lattitude                       < -37.74891  to the right, improve=0.1773264, (0 missing)
          Longtitude                      < 144.9493   to the left,  improve=0.1147543, (0 missing)
          Distance                        < 0.7219626  to the left,  improve=0.1064398, (0 missing)
          Bathroom                        splits as  LLLLRRRRR, improve=0.1030338, (0 missing)
      Surrogate splits:
          Distance < 0.9696262  to the left,  agree=0.877, adj=0.150, (0 split)
          Bathroom splits as  RLLLLLLLR, agree=0.855, adj=0.004, (0 split)

    Node number 4: 3152 observations
      mean=601776.3, MSE=6.323728e+10

    Node number 5: 7677 observations,    complexity param=0.04902198
      mean=1057425, MSE=2.364079e+11
      left son=10 (2122 obs) right son=11 (5555 obs)
      Primary splits:
          Lattitude                        < -37.75211  to the right, improve=0.15204760, (0 missing)
          Council_Boroondara_City_Council  < 0.5        to the left,  improve=0.10542930, (0 missing)
          Longtitude                       < 144.944    to the left,  improve=0.10420270, (0 missing)
          Council_Stonnington_City_Council < 0.5        to the left,  improve=0.04714486, (0 missing)
          Distance                         < 0.5654206  to the left,  improve=0.04599326, (0 missing)
      Surrogate splits:
          Council_Darebin_City_Council    < 0.5        to the right, agree=0.762, adj=0.138, (0 split)
          Council_Hume_City_Council       < 0.5        to the right, agree=0.756, adj=0.115, (0 split)
          Council_Whittlesea_City_Council < 0.5        to the right, agree=0.742, adj=0.068, (0 split)
          Distance                        < 0.9976636  to the right, agree=0.734, adj=0.039, (0 split)
          Council_Melton_City_Council     < 0.5        to the right, agree=0.730, adj=0.022, (0 split)

    Node number 6: 3019 observations,    complexity param=0.04372912
      mean=1359905, MSE=5.105211e+11
      left son=12 (794 obs) right son=13 (2225 obs)
      Primary splits:
          Lattitude                        < -37.74835  to the right, improve=0.15971150, (0 missing)
          Council_Stonnington_City_Council < 0.5        to the left,  improve=0.12121360, (0 missing)
          Council_Bayside_City_Council     < 0.5        to the left,  improve=0.11079890, (0 missing)
          Longtitude                       < 144.872    to the left,  improve=0.09716043, (0 missing)
          Bathroom                         splits as  LLLLRRRL-, improve=0.08318964, (0 missing)
      Surrogate splits:
          Council_Hume_City_Council       < 0.5        to the right, agree=0.773, adj=0.136, (0 split)
          Council_Whittlesea_City_Council < 0.5        to the right, agree=0.763, adj=0.097, (0 split)
          Council_Darebin_City_Council    < 0.5        to the right, agree=0.753, adj=0.059, (0 split)
          Distance                        < 0.9883178  to the right, agree=0.751, adj=0.054, (0 split)
          Council_Moreland_City_Council   < 0.5        to the right, agree=0.748, adj=0.040, (0 split)

    Node number 7: 513 observations
      mean=2365406, MSE=5.202804e+11

    Node number 10: 2122 observations
      mean=750671.4, MSE=5.683417e+10

    Node number 11: 5555 observations,    complexity param=0.03620536
      mean=1174605, MSE=2.553285e+11
      left son=22 (1115 obs) right son=23 (4440 obs)
      Primary splits:
          Longtitude                      < 144.8936   to the left,  improve=0.14369170, (0 missing)
          Council_Boroondara_City_Council < 0.5        to the left,  improve=0.08627518, (0 missing)
          Council_Brimbank_City_Council   < 0.5        to the right, improve=0.05417094, (0 missing)
          Lattitude                       < -37.80239  to the right, improve=0.04379666, (0 missing)
          Bathroom                        splits as  LL-RRRRR-, improve=0.04333133, (0 missing)
      Surrogate splits:
          Council_Maribyrnong_City_Council < 0.5        to the right, agree=0.870, adj=0.354, (0 split)
          Council_Brimbank_City_Council    < 0.5        to the right, agree=0.847, adj=0.239, (0 split)
          Council_Hobsons_Bay_City_Council < 0.5        to the right, agree=0.839, adj=0.196, (0 split)
          Council_Wyndham_City_Council     < 0.5        to the right, agree=0.813, adj=0.070, (0 split)
          Distance                         < 0.9929907  to the right, agree=0.801, adj=0.009, (0 split)

    Node number 12: 794 observations
      mean=881903, MSE=8.15504e+10

    Node number 13: 2225 observations,    complexity param=0.02608737
      mean=1530482, MSE=5.529684e+11
      left son=26 (2138 obs) right son=27 (87 obs)
      Primary splits:
          Council_Stonnington_City_Council  < 0.5        to the left,  improve=0.11935550, (0 missing)
          Longtitude                        < 144.8831   to the left,  improve=0.11056570, (0 missing)
          Council_Bayside_City_Council      < 0.5        to the left,  improve=0.08958585, (0 missing)
          Bathroom                          splits as  -LLLRRRL-, improve=0.08654541, (0 missing)
          Council_Port_Phillip_City_Council < 0.5        to the left,  improve=0.05289840, (0 missing)

    Node number 22: 1115 observations
      mean=792379.3, MSE=6.04901e+10

    Node number 23: 4440 observations,    complexity param=0.01356944
      mean=1270592, MSE=2.583554e+11
      left son=46 (3856 obs) right son=47 (584 obs)
      Primary splits:
          Council_Boroondara_City_Council < 0.5        to the left,  improve=0.06658920, (0 missing)
          Longtitude                      < 145.1613   to the right, improve=0.06467657, (0 missing)
          Rooms                           splits as  LLR----, improve=0.05411615, (0 missing)
          Lattitude                       < -37.95461  to the left,  improve=0.04367245, (0 missing)
          type_2                          < 0.5        to the right, improve=0.03951833, (0 missing)
      Surrogate splits:
          Distance < 0.9696262  to the left,  agree=0.896, adj=0.211, (0 split)

    Node number 26: 2138 observations,    complexity param=0.02295836
      mean=1478658, MSE=4.498782e+11
      left son=52 (1902 obs) right son=53 (236 obs)
      Primary splits:
          Council_Bayside_City_Council      < 0.5        to the left,  improve=0.13436330, (0 missing)
          Longtitude                        < 144.8727   to the left,  improve=0.12142690, (0 missing)
          Bathroom                          splits as  -LLLRRRL-, improve=0.08708185, (0 missing)
          Council_Port_Phillip_City_Council < 0.5        to the left,  improve=0.07409553, (0 missing)
          Distance                          < 0.1028037  to the right, improve=0.04603139, (0 missing)

    Node number 27: 87 observations
      mean=2804031, MSE=1.398455e+12

    Node number 46: 3856 observations,    complexity param=0.01199327
      mean=1219547, MSE=2.325101e+11
      left son=92 (1099 obs) right son=93 (2757 obs)
      Primary splits:
          Longtitude                        < 145.0617   to the right, improve=0.06889617, (0 missing)
          Council_Stonnington_City_Council  < 0.5        to the left,  improve=0.05304292, (0 missing)
          Rooms                             splits as  LLR----, improve=0.04707172, (0 missing)
          Lattitude                         < -37.95643  to the left,  improve=0.04413913, (0 missing)
          Council_Port_Phillip_City_Council < 0.5        to the left,  improve=0.04386360, (0 missing)
      Surrogate splits:
          Landsize                        < 0.3431291  to the right, agree=0.783, adj=0.238, (0 split)
          Council_Monash_City_Council     < 0.5        to the right, agree=0.775, adj=0.211, (0 split)
          Lattitude                       < -37.95498  to the left,  agree=0.765, adj=0.177, (0 split)
          Council_Manningham_City_Council < 0.5        to the right, agree=0.757, adj=0.148, (0 split)
          Distance                        < 0.3528037  to the left,  agree=0.749, adj=0.120, (0 split)

    Node number 47: 584 observations
      mean=1607625, MSE=2.982105e+11

    Node number 52: 1902 observations,    complexity param=0.01600244
      mean=1392054, MSE=3.62664e+11
      left son=104 (229 obs) right son=105 (1673 obs)
      Primary splits:
          Longtitude                        < 144.8727   to the left,  improve=0.13059110, (0 missing)
          Council_Port_Phillip_City_Council < 0.5        to the left,  improve=0.11939820, (0 missing)
          Bathroom                          splits as  -LRLRRRL-, improve=0.05513764, (0 missing)
          Distance                          < 0.567757   to the left,  improve=0.05312046, (0 missing)
          Council_Wyndham_City_Council      < 0.5        to the right, improve=0.05105073, (0 missing)
      Surrogate splits:
          Council_Brimbank_City_Council < 0.5        to the right, agree=0.917, adj=0.314, (0 split)
          Council_Wyndham_City_Council  < 0.5        to the right, agree=0.908, adj=0.236, (0 split)
          Distance                      < 0.978972   to the right, agree=0.883, adj=0.031, (0 split)
          Council_Melton_City_Council   < 0.5        to the right, agree=0.881, adj=0.009, (0 split)

    Node number 53: 236 observations
      mean=2176629, MSE=6.051562e+11

    Node number 92: 1099 observations
      mean=1019082, MSE=1.74309e+11

    Node number 93: 2757 observations,    complexity param=0.01199327
      mean=1299457, MSE=2.333057e+11
      left son=186 (992 obs) right son=187 (1765 obs)
      Primary splits:
          Rooms                            splits as  LLR----, improve=0.11388600, (0 missing)
          Lattitude                        < -37.81776  to the right, improve=0.10425410, (0 missing)
          Bathroom                         splits as  LL-RRRRR-, improve=0.07883109, (0 missing)
          Council_Stonnington_City_Council < 0.5        to the left,  improve=0.05212816, (0 missing)
          Landsize                         < 0.0906457  to the left,  improve=0.04760861, (0 missing)
      Surrogate splits:
          Landsize  < 0.0906457  to the left,  agree=0.738, adj=0.273, (0 split)
          Car       < 0.02777778 to the left,  agree=0.662, adj=0.059, (0 split)
          Distance  < 0.03504673 to the left,  agree=0.646, adj=0.017, (0 split)
          Bathroom  splits as  LR-RRRRR-, agree=0.643, adj=0.007, (0 split)
          Lattitude < -37.75227  to the right, agree=0.641, adj=0.001, (0 split)

    Node number 104: 229 observations
      mean=803834.9, MSE=6.288341e+10

    Node number 105: 1673 observations,    complexity param=0.01460007
      mean=1472570, MSE=3.498545e+11
      left son=210 (784 obs) right son=211 (889 obs)
      Primary splits:
          Longtitude                        < 145.0688   to the right, improve=0.14041510, (0 missing)
          Council_Port_Phillip_City_Council < 0.5        to the left,  improve=0.12384610, (0 missing)
          Lattitude                         < -37.92425  to the left,  improve=0.08511412, (0 missing)
          Bathroom                          splits as  -LRLRRRL-, improve=0.04400034, (0 missing)
          Council_/images/melbourne_City_Council    < 0.5        to the left,  improve=0.04208519, (0 missing)
      Surrogate splits:
          Distance                        < 0.567757   to the left,  agree=0.816, adj=0.608, (0 split)
          Landsize                        < 0.4110099  to the right, agree=0.678, adj=0.314, (0 split)
          Council_Manningham_City_Council < 0.5        to the right, agree=0.653, adj=0.259, (0 split)
          Council_Monash_City_Council     < 0.5        to the right, agree=0.644, adj=0.240, (0 split)
          Lattitude                       < -37.84785  to the left,  agree=0.634, adj=0.218, (0 split)

    Node number 186: 992 observations
      mean=1082029, MSE=1.299196e+11

    Node number 187: 1765 observations
      mean=1421660, MSE=2.49909e+11

    Node number 210: 784 observations
      mean=1236552, MSE=1.378909e+11

    Node number 211: 889 observations
      mean=1680711, MSE=4.443354e+11




![png](/images/melbourne/melbourne-housing-price-prediction_24_1.png)



```R
tree.pred <- predict(pfit,newdata = test_data)
dt.SSE <- sum((test_data[,"Price"]-tree.pred)^2)
dt.SST <- sum((test_data[,"Price"] - mean(test_data[,"Price"]))^2)
dt.r2 <- 1 - dt.SSE/dt.SST
dt.r2
```


0.574209775190866



```R
#######################################################
#### ---------Random Forests
#######################################################

rf <- randomForest(Price ~ .,data = train_data)
rf.pred <- predict(rf,newdata = test_data)
SSE.rf <- sum((test_data[,"Price"]-rf.pred)^2)
SST.rf <- sum((test_data[,"Price"] - mean(test_data[,"Price"]))^2)
r2.rf <- 1 - SSE.rf/SST.rf
r2.rf
```


0.808423681957345


With compared Analysis of accuracy for the above 3 models it can be observed that Random Forest Works best with maximum accuracy.


```R

```
