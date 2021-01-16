---
title: "Data Science survey 2019 Results by Numbers."
#date: 2018-01-28
#tags: [data wrangling, data science, messy data]
header:
  #image: "/images/churn/churn1.png"
excerpt: "Python:Exploratory Data Analysis,Data Visualization using Seaborn library"
mathjax: "true"
---

<b><font size="+2">Data Science survey 2019 Results by Numbers.<br>

<b>Kaggle set out to conduct an industry-wide survey that presents a truly comprehensive view of the state of data science and machine learning. The survey was live for three weeks in October 2019, and after cleaning the data included 19,717 responses.
This survey received 19,717 usable respondents from 171 countries and territories.
Most of the respondents were found primarily through Kaggle channels, email list, discussion forums and social media channels.Multiple choice single response questions fit into individual columns whereas multiple choice multiple response questions were split into multiple columns.</b>

Following is the analysis conducted over the survey results.


```python
#libraries
import pandas as pd
import numpy as np
import seaborn as sns
from matplotlib import pyplot
```


```python
#configs
pd.set_option("display.max_columns",999)
pd.set_option("display.max_rows",999)

****Files importing****
data1=pd.read_csv("/Users/../survey_schema.csv")
data2=pd.read_csv("/Users/../other_text_responses.csv")
data3=pd.read_csv("/Users/../questions_only.csv")
data4=pd.read_csv("/Users/../multiple_choice_responses.csv")               
```
<b>1.Educational Analysis of the Respondents.</b>
```python
data_degree = data4[data4["Q4"].notna()]
my_dataframe = pd.DataFrame(data_degree.loc[1:,"Q4"])
my_dataframe["Q4"] = my_dataframe["Q4"].astype('category')
figure1 = sns.countplot(x="Q4",data=my_dataframe)
figure1.set_xticklabels(figure1.get_xticklabels(), rotation=75)
pyplot.title("Educational Analysis")
pyplot.xlabel("Degree")
```

![png](/images/DS_survey/kaggle_DS_survey_2019_4_1.png)


<b>Following are the insights from the above plot.<br>
1.There are considerably larger number of kaggle survey respondents with educational qualification of bachelors and Masters degree.
<br>
For starters, you don’t need a Master’s or a Ph.D. degree , if you already have it – great! It’s certainly a plus! However, a Bachelor’s degree is good enough to get you on this journey.<br>
<br>

<b>2.Job Profile of the surveyors with a Master's Degree.</b>

```python
data4["Q4"].unique()
Masters_data = data4[data4["Q4"]=="Master’s degree"]
Masters_data = Masters_data[Masters_data != 'Student']
Masters_data = Masters_data[Masters_data != 'Not employed']
```
```python
Masters_data["Q5"].unique()
figure2 = sns.countplot(x='Q5',data=Masters_data,order = Masters_data['Q5'].value_counts().index)
figure2.set_xticklabels(figure2.get_xticklabels(), rotation=75)
figure2.set(xlabel='Job Profile',ylabel='Count')
pyplot.title("Job Profile for the Surveyors with a Master's Degree")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_8_1.png)

<b>Following are the insights from the above plot.<br>
It can be observed that the among the survey respondents with a Master's degree, maximum respondents are working with a job profile of Data Scientist followed by Software engineer and Data analyst.<br>
<br>
<b>3.Job Profile of the surveyors with a Bachelor's Degree.</b>


```python
bachelors_data = data4[data4["Q4"]=="Bachelor’s degree"]
bachelors_data = bachelors_data[bachelors_data != 'Student']
bachelors_data = bachelors_data[bachelors_data != 'Not employed']

bachelors_data["Q5"].unique()

figure3 = sns.countplot(x='Q5',data=bachelors_data,order = bachelors_data['Q5'].value_counts().index)
figure3.set_xticklabels(figure3.get_xticklabels(), rotation=75)
figure3.set(xlabel='Job Profile',ylabel='Count')
pyplot.title("Job Profile for the Surveyors with a Bachelors's Degree")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_11_1.png)

<b>Following are the insights from the above plot.<br>
It can be observed that the among the survey respondents with a Bachelor's degree, maximum respondents are working with a job profile of Software engineer followed by Data Scientist and Data analyst.<br>

<b>We can see that the respondents with a bachelors degree are working actively in the data science field.</b>
<br>
<br>
<b>4.Country based Analysis of the Respondents.</b>


```python
Masters_data["Q3"].unique()
```

    array(['France', 'Australia', 'India', 'Netherlands', 'Germany',
           'United States of America', 'Ireland', 'Russia', 'Greece',
           'Pakistan', 'Japan', 'Other', 'South Korea', 'Brazil', 'Belarus',
           'Nigeria', 'Sweden', 'Mexico', 'Canada', 'Poland', 'Indonesia',
           'Czech Republic',
           'United Kingdom of Great Britain and Northern Ireland', 'Morocco',
           'Spain', 'Chile', 'Portugal', 'Italy', 'Taiwan', 'Egypt', 'Norway',
           'South Africa', 'Thailand', 'China', 'Argentina', 'Denmark',
           'Colombia', 'Viet Nam', 'Israel', 'Ukraine', 'Singapore',
           'Iran, Islamic Republic of...', 'Bangladesh', 'Turkey',
           'Switzerland', 'Peru', 'Algeria', 'Belgium', 'Kenya', 'Hungary',
           'New Zealand', 'Tunisia', 'Austria', 'Romania', 'Malaysia',
           'Philippines', 'Hong Kong (S.A.R.)', 'Republic of Korea',
           'Saudi Arabia'], dtype=object)




```python
a4_dims = (11.7, 8.27)
fig, ax = pyplot.subplots(figsize=a4_dims)
figure4=sns.countplot(x="Q3",data=Masters_data,palette="Greens_d",order = Masters_data['Q3'].value_counts().index)
figure4.set_xticklabels(figure4.get_xticklabels(), rotation=90)
figure4.set(xlabel="Countries",ylabel="Count")
pyplot.title("Countries of Surveyors with Master's Degree")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_15_1.png)

<b>Following are the insights from the above plot.<br>
<b>1.Maximum number of surveyors having master degree are residing in India followed by United States.</b>


```python

```


```python
bachelors_data["Q3"].unique()
```




    array(['India', 'United States of America', 'Germany', 'Australia',
           'Brazil', 'United Kingdom of Great Britain and Northern Ireland',
           'Russia', 'Pakistan', 'South Africa', 'Israel', 'Nigeria', 'Japan',
           'Ukraine', 'Bangladesh', 'Canada', 'South Korea', 'Other', 'Egypt',
           'Netherlands', 'Indonesia', 'Turkey', 'Viet Nam', 'Chile',
           'Colombia', 'France', 'Mexico', 'Norway', 'Italy', 'New Zealand',
           'Iran, Islamic Republic of...', 'Kenya', 'Denmark', 'Spain',
           'Argentina', 'Malaysia', 'Romania', 'Algeria', 'Ireland',
           'Hungary', 'Philippines', 'Morocco', 'Sweden', 'Greece', 'Tunisia',
           'Singapore', 'Taiwan', 'Belarus', 'Poland', 'Portugal', 'China',
           'Hong Kong (S.A.R.)', 'Peru', 'Republic of Korea', 'Saudi Arabia',
           'Thailand', 'Czech Republic', 'Switzerland', 'Belgium', 'Austria'],
          dtype=object)




```python
a4_dims = (11.7, 8.27)
fig, ax = pyplot.subplots(figsize=a4_dims)
figure5=sns.countplot(x="Q3",data=bachelors_data,palette="Greens_d",order = bachelors_data['Q3'].value_counts().index)
figure5.set_xticklabels(figure5.get_xticklabels(), rotation=90)
figure5.set(xlabel="Countries",ylabel="Count")
pyplot.title("Countries of Surveyors with Bachelor's Degree")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_19_1.png)

<b>Following are the insights from the above plot.<br>
<b>1.Maximum number of surveyors having bachelor's degree are residing in India followed by United States similar to which was observed in case of Master's degree surveyors.</b>
<br>
<br>
<b>5.Gender check for the survey respondents.</b>


```python
# Gender check for the surveyors.

column=['Q2']
data4_gender = pd.DataFrame(data_degree.loc[1:,'Q2'],columns=column)
gend = data4_gender["Q2"].value_counts()
gender_data = pd.DataFrame((gend.values/len(data4_gender["Q2"]) ) * 100)
gender_data.insert(1,"Gender",['Male', 'Female', 'Prefer not to say', 'Prefer to self-describe'],True)
columns=["Percent","Gender"]
gender_data.columns=columns
labels = gender_data["Gender"]
percent= gender_data["Percent"]
colors = ['gold', 'yellowgreen', 'lightcoral', 'lightskyblue']
explode = (0,0.1, 0, 0)  # explode 2nd slice
pyplot.pie(percent, explode=explode, labels=labels, colors=colors,
autopct='%1.1f%%', shadow=True, startangle=180)
pyplot.axis('equal')
pyplot.show()
```


![png](/images/DS_survey/kaggle_DS_survey_2019_22_0.png)

<b>Following are the insights from the above plot.<br>
<b>The above plot shows that the 81.9% are the Male respondents from the total survey outcomes provided here.</b>

 <b>6.Age analysis of the respondents with Master's Degree.</b>


```python
Masters_data["Q1"].unique()
```


    array(['22-24', '40-44', '50-54', '55-59', '30-34', '35-39', '25-29',
           '45-49', '18-21', '60-69', '70+'], dtype=object)



```python
a4_dims = (11.7, 8.27)
fig, ax = pyplot.subplots(figsize=a4_dims)
figure4=sns.countplot(x="Q1",data=Masters_data,order = Masters_data['Q1'].value_counts().index)
figure4.set_xticklabels(figure4.get_xticklabels())
pyplot.title("Age Group for Surveyors with Master's Degree")
figure4.set(xlabel="Age",ylabel="Count")
```

![png](/images/DS_survey/kaggle_DS_survey_2019_26_1.png)

<b>Following are the insights from the above plot.<br>
From the above plot it can be observed that the maximum kaggle survey respondents with a Masters degree are within the age group of 25-29.<br>
<br>
<b>7.Age analysis of the respondents with Bachelor's Degree.</b>


```python
bachelors_data["Q1"].unique()
```


    array(['22-24', '30-34', '25-29', '18-21', '35-39', '55-59', '40-44',
           '50-54', '60-69', '45-49', '70+'], dtype=object)




```python
a4_dims = (11.7, 8.27)
fig, ax = pyplot.subplots(figsize=a4_dims)
figure4=sns.countplot(x="Q1",data=bachelors_data,order = bachelors_data['Q1'].value_counts().index)
figure4.set_xticklabels(figure4.get_xticklabels())
pyplot.title("Age Group for Surveyors with Bachelors's Degree")
figure4.set(xlabel="Age",ylabel="Count")
```

![png](/images/DS_survey/kaggle_DS_survey_2019_31_1.png)


```python
job_data = data_degree.loc[1:,:]
```


```python
job_data['Q10'].value_counts().mean
```




    <bound method Series.mean of $0-999             1513
    10,000-14,999       833
    100,000-124,999     750
    30,000-39,999       728
    40,000-49,999       719
    50,000-59,999       704
    1,000-1,999         599
    60,000-69,999       576
    5,000-7,499         536
    15,000-19,999       529
    20,000-24,999       526
    70,000-79,999       524
    125,000-149,999     483
    25,000-29,999       482
    150,000-199,999     434
    7,500-9,999         408
    80,000-89,999       405
    2,000-2,999         390
    90,000-99,999       377
    3,000-3,999         305
    4,000-4,999         289
    200,000-249,999     165
    > $500,000           83
    300,000-500,000      74
    250,000-299,999      65
    Name: Q10, dtype: int64>



 <b>8.Salary by job profile.</b>


```python
Job_Sal_data = pd.crosstab(job_data['Q10'],job_data['Q5'])
```


```python
figure5 = Job_Sal_data.plot.bar(stacked=True,figsize=(10,10))
```


![png](/images/DS_survey/kaggle_DS_survey_2019_36_0.png)

<b>Following are the insights from the above plot.<br>

<b>1.The respondents have more number of Data Scientists and less number of Statisticians.<br>
<b>2.Salaries of these job profiles are diverse.

<b>9.Hot Skills for 2019 among the Data Science Enthusiasts.</b>


```python
my_data = data_degree.loc[1:,]
```


```python
my_data["Q19"].unique()
```




    array(['Python', nan, 'Java', 'R', 'SQL', 'C++', 'None', 'Other', 'C',
           'MATLAB', 'TypeScript', 'Javascript', 'Bash'], dtype=object)




```python
my_data["Q19"].value_counts()
```

    Python        11316
    R              1343
    SQL             817
    C++             199
    MATLAB          162
    C               153
    Other           127
    Java            104
    None             69
    Javascript       47
    Bash             35
    TypeScript        5
    Name: Q19, dtype: int64


```python
data_job_skill = pd.crosstab(my_data["Q5"],my_data["Q19"])
```


```python
figure6 = data_job_skill.plot.bar(figsize=(10,10),stacked=True,grid=True)
pyplot.title("Recommended Skills by the Kaggle Surveyors")
figure6.set(xlabel="Job Role",ylabel="Count")
```

![png](/images/DS_survey/kaggle_DS_survey_2019_43_1.png)

<b>Following are the insights from the above plot.<br>
The above plot tells that among all the kaggle surveyors Python is the most widely recommended language followed by R and SQL.



```python

```


```python
#function
def my_function_plot(mydataframe):
    list_notebook_sources=[]
    for i in mydataframe:
        list_notebook_sources.append(mydataframe[i].value_counts())

    test =pd.DataFrame(list_notebook_sources)

    l1=[]
    for i in test:
        l1.append(i)
    l2=[]
    for i in range(0,12):
        l2.append(test.iloc[i,i])

    data_1= {}
    data_1 = {
        'Source' : l1,
        'Value' : l2
    }

    data_source_df = pd.DataFrame(data_1,index=[0,1,2,3,4,5,6,7,8,9,10,11])
    figure9 = sns.barplot(x='Source',y='Value',data=data_source_df,palette="Greens_d")
    figure9.set_xticklabels(figure9.get_xticklabels(), rotation=90)

```


```python

```

<b>10.Favorite media sources that report on data science topics.</b>


```python
data_sources = my_data.loc[:,"Q12_Part_1":"Q12_Part_12"]
my_function_plot(data_sources)
pyplot.title("Favorite media sources that report on data science topics.")
pyplot.xlabel("Media Sources")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_49_1.png)

<b>Following are the insights from the above plot.<br>
According the survey results it can be seen that the Kaggle has been most widely accepted media source followed by blogs like that of Towards Data Science,Medium,Analytics Vidya,KDnuggets.

<b>11.Open Online Courses for Data Science Learning.</b>


```python
data_online_sources = my_data.loc[:,"Q13_Part_1":"Q13_Part_12"]

my_function_plot(data_online_sources)
pyplot.title("On which platforms have you begun or completed data science courses")
pyplot.xlabel("Data Science Course")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_52_1.png)

<b>Following are the insights from the above plot.<br>
It can be seen that the most widely accepted open online course among the respndents is Coursera followed by Kaggle courses and Udemy courses and university courses(resulting in a university degree).

<b>12.Trending Primary Analysis Tools among the respondents.</b>


```python
my_data["Q14"].value_counts()
figure8=sns.countplot(x="Q14",data=my_data)
figure8.set_xticklabels(figure8.get_xticklabels(), rotation=90)
pyplot.title("Primary tool that you use at work or school to analyze data")
pyplot.xlabel("Analysing Tools")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_55_1.png)

<b>Following are the insights from the above plot.<br>
<b>It can be observed that the local development environments like that Rstudio,JupyterLab are trending among the respondents.

<b>13.Coding Experience among the Respondents.</b>


```python
Masters_data["Q15"].value_counts()
figure9=sns.countplot(x="Q15",data=Masters_data,order = Masters_data['Q15'].value_counts().index)
figure9.set_xticklabels(figure9.get_xticklabels(), rotation=90)
pyplot.title("Masters data having experience in coding using any of the programming languages")
pyplot.xlabel("Analysing Tools")

```


![png](/images/DS_survey/kaggle_DS_survey_2019_58_1.png)

<b>Following are the insights from the above plot.<br>
It can be observed that the kagglers with a masters degree are having a minimum of 1-2 years / 3-5 years of the coding experience.


```python
#Masters_data["Q15"].value_counts()
figure9=sns.countplot(x="Q15",data=bachelors_data,order = bachelors_data['Q15'].value_counts().index)
figure9.set_xticklabels(figure9.get_xticklabels(), rotation=90)
pyplot.title("Bachelors data having experience in coding using any of the programming languages")
pyplot.xlabel("Analysing Tools")

```


![png](/images/DS_survey/kaggle_DS_survey_2019_60_1.png)

<b>Following are the insights from the above plot.<br>
It can be observed that the kagglers with a bachelors degree are having a minimum of 0-2 years of the coding experience.


```python
my_data["Q15"].value_counts()
figure9=sns.countplot(x="Q15",data=my_data)
figure9.set_xticklabels(figure9.get_xticklabels(), rotation=90)
pyplot.title("Experience with Coding")
pyplot.xlabel("In Years")
```



![png](/images/DS_survey/kaggle_DS_survey_2019_62_1.png)

<b>Following are the insights from the above plot.<br>
While in case of all kagglers it can be observed that the maximum surveyors are having 0-2 years of experience.

<b>14.Trending IDE's in 2019..!!</b>


```python
data_IDE_sources = my_data.loc[:,"Q16_Part_1":"Q16_Part_12"]
my_function_plot(data_IDE_sources)
pyplot.title("Integrated development environments (IDE's) do you use on a regular basis")
pyplot.xlabel("DIfferent IDE's")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_65_1.png)

<b>Following are the insights from the above plot.<br>
It can be observed that the Jupyter IDE's are the most acceptable and widely used by respondents.


```python

```

<b>15.Trending NoteBook's of 2019.!!</b>


```python
data_notebook_sources = my_data.loc[:,"Q17_Part_1":"Q17_Part_12"]

my_function_plot(data_notebook_sources)
pyplot.title("Notebook's Do you use on a regular basis")
pyplot.xlabel("DIfferent Notebook's")
```

![png](/images/DS_survey/kaggle_DS_survey_2019_69_1.png)

<b>Following are the insights from the above plot.<br>
We can see that there many respondents who havnt been using any notebook for the data analysis development while among the respondents who have been using notebook's for development kaggle Notebooks and google colab are the most trending notebooks.


```python

```

<b>16.Trending Programming Languages of 2019 in the field of Data Analytics.</b>


```python
data_language_sources = my_data.loc[:,"Q18_Part_1":"Q18_Part_12"]

my_function_plot(data_language_sources)
pyplot.title("Popular Programming Languages")
pyplot.xlabel("Programming Language")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_73_1.png)

<b>Following are the insights from the above plot.<br>
We can see that the Python top's the chart followed by R programming and SQL(Structured Query Language)


```python

```

<b>17.Trending Data Visualization Packages..!!</b>


```python
data_visualization_sources = my_data.loc[:,"Q20_Part_1":"Q20_Part_12"]

my_function_plot(data_visualization_sources)
pyplot.title("Popular Data Visualization Packages")
pyplot.xlabel("Visualization Package")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_77_1.png)

<b>Following are the insights from the above plot.<br>
Python's Matplotlib and seaborn are the trending data visualization packages followed by R's ggplot2.
Data science professionals out there must be well versed with these visualization packages.

<b>18.Trending Popular ML algorithms..!!</b>


```python
data_modeling_sources = my_data.loc[:,"Q24_Part_1":"Q24_Part_12"]

my_function_plot(data_modeling_sources)
pyplot.title("Popular ML algoirthms")
pyplot.xlabel("Algorithm")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_80_1.png)

<b>Following are the insights from the above plot.<br>
Based on the results we could see that the classification algorithms- logistic algorithms , Decision trees and Random Forests are the trending ML algorithms.

<b>19.Trending Machine Learning Frameworks..!!</b>


```python
data_cat_sources = my_data.loc[:,"Q28_Part_1":"Q28_Part_12"]

my_function_plot(data_cat_sources)
pyplot.title(" Machine Learning Frameworks")
pyplot.xlabel("Frameworks")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_83_1.png)

<b>Following are the insights from the above plot.<br>
It can be observed that the popular machine learning frameworks out there include pythons Scikit learn and followed by Tensorflow.


```python

```

<b>20.Trending Cloud Computing Platforms..!!!</b>


```python
data_cloud_sources = my_data.loc[:,"Q29_Part_1":"Q29_Part_12"]

my_function_plot(data_cloud_sources)
pyplot.title(" Popular cloud computing platforms")
pyplot.xlabel("Cloud Computing platforms")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_87_1.png)



```python
data_computing_sources = my_data.loc[:,"Q30_Part_1":"Q30_Part_12"]

my_function_plot(data_computing_sources)
pyplot.title(" Popular cloud computing platforms")
pyplot.xlabel("cloud computing platforms")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_88_1.png)

<b>Following are the insights from the above plot.<br>
We can see that the popular cloud computing platforms which are widely accepted by the respondents include AWS(amazon web services) also we could see that many respondents are not using clouding computing platforms yet.

<b>Thus having hands-on knowledge of cloud computing platforms like AWS and GCP will definitely add to the applicants profile.

<b>21.Trending Big Data Analytics Products..!</b>


```python
data_computing_sources = my_data.loc[:,"Q31_Part_1":"Q31_Part_12"]

my_function_plot(data_computing_sources)
pyplot.title(" Popular big data / analytics products")
pyplot.xlabel("big data / analytics products")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_91_1.png)

<b>Following are the insights from the above plot.<br>
We can see that the many respondents are not yet not working on the big data analytics products but of the respondents who are having some experience in these products are using Google Bigquery and Databricks.

<b>22.Trending Clouding ML platforms.!!</b>


```python
data_ML_sources = my_data.loc[:,"Q32_Part_1":"Q32_Part_12"]

my_function_plot(data_ML_sources)
pyplot.title("machine learning products")
pyplot.xlabel("big data / analytics products")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_94_1.png)



```python
data_ML_sources = my_data.loc[:,"Q33_Part_1":"Q33_Part_12"]

my_function_plot(data_ML_sources)
pyplot.title("Automated machine learning tools")
pyplot.xlabel("Automated machine learning tools")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_95_1.png)

<b>Following are the insights from the above plot.<br>
Very less respondents are experienced working with the new machine learning cloud platforms.
Google Cloud Machine learning Engine is gaining popularity and is being used to some extent meanwhile automated machine learning tools are also gaining popularity. Auto keras , Auto sk learn and Google AutoML are being used to some extent.

<b>23.Trending Databases in 2019..!!</b>


```python
data_DB_sources = my_data.loc[:,"Q34_Part_1":"Q34_Part_12"]

my_function_plot(data_DB_sources)
pyplot.title("Popular Databases for Data Analysis.")
pyplot.xlabel("Databases")
pyplot.ylabel("Count")
```


![png](/images/DS_survey/kaggle_DS_survey_2019_98_1.png)

<b>Following are the insights from the above plot.<br>
MySQL,PostgreSQL,SQLite and MS SQL are most trending databases based upon the survey results by the respondents.Thus having knowledge of the one these database query languages is of utmost necessity.
