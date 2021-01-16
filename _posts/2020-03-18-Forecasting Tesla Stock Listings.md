---
title: "Forecasting Tesla Stock Listings."
#date: 2018-01-28
#tags: [data wrangling, data science, messy data]
header:
  #image: "/images/churn/churn1.png"
excerpt: "Python programming:Time Series and Forecasting:Simple Moving Average,Weighted Moving Average and ARIMA modeling."
mathjax: "true"
---

# Time series & forecasting .

**Forecasting the values for TESLA stock listings.**

**Given : Values till March 07 2017**

**Predict : Values for next 7 business days after March 07 2017**


```python
import pandas as pd
import numpy as np
import random
import seaborn as sns
import matplotlib.pyplot as plt
from numpy import mean, absolute
from statsmodels.tsa.stattools import adfuller
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error
from statsmodels.tsa.holtwinters import ExponentialSmoothing
from statsmodels.tsa.arima_model import ARIMA
```


```python
data = pd.read_csv("/Users/abhishekbidap/Desktop/my_stuff/Quarter 2/Intro to Enterprise/Module 4/TSLA__.csv")
tesla = data.copy()
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)
```
```python
tesla.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>6/29/10</td>
      <td>19.000000</td>
      <td>25.00</td>
      <td>17.540001</td>
      <td>23.889999</td>
      <td>23.889999</td>
      <td>18766300</td>
    </tr>
    <tr>
      <td>1</td>
      <td>6/30/10</td>
      <td>25.790001</td>
      <td>30.42</td>
      <td>23.299999</td>
      <td>23.830000</td>
      <td>23.830000</td>
      <td>17187100</td>
    </tr>
    <tr>
      <td>2</td>
      <td>7/1/10</td>
      <td>25.000000</td>
      <td>25.92</td>
      <td>20.270000</td>
      <td>21.959999</td>
      <td>21.959999</td>
      <td>8218800</td>
    </tr>
    <tr>
      <td>3</td>
      <td>7/2/10</td>
      <td>23.000000</td>
      <td>23.10</td>
      <td>18.709999</td>
      <td>19.200001</td>
      <td>19.200001</td>
      <td>5139800</td>
    </tr>
    <tr>
      <td>4</td>
      <td>7/6/10</td>
      <td>20.000000</td>
      <td>20.00</td>
      <td>15.830000</td>
      <td>16.110001</td>
      <td>16.110001</td>
      <td>6866900</td>
    </tr>
  </tbody>
</table>
</div>

```python
tesla.isna().sum()
```

    Date         0
    Open         0
    High         0
    Low          0
    Close        0
    Adj Close    0
    Volume       0
    dtype: int64


```python
data['Date']= pd.to_datetime(data['Date'])
```

```python
#plotting each column.
for i in data:
    if i not in ['Date','Adj Close','Volume']:
        print("------------------------------------------")
        print('Test Column:',i)
        X=data.loc[:,i]
        X.plot()
        plt.show()
```

    ------------------------------------------
    Test Column: Open

![png](/images/TESLA/TESLA_STOCK_FORECAST_6_1.png)

    ------------------------------------------
    Test Column: High

![png](/images/TESLA/TESLA_STOCK_FORECAST_6_3.png)


    ------------------------------------------
    Test Column: Low



![png](/images/TESLA/TESLA_STOCK_FORECAST_6_5.png)


    ------------------------------------------
    Test Column: Close



![png](/images/TESLA/TESLA_STOCK_FORECAST_6_7.png)


```python
def metric_cal(Actual_Values,Predicted_values):
    print("\nMean Absolute Deviation Error (MAD)")
    MAD_Open = mean(absolute(Actual_Values['Open'] - Predicted_values['Open_Predicted']))    
    MAD_Close = mean(absolute(Actual_Values['Close'] - Predicted_values['Close_Predicted']))
    MAD_High = mean(absolute(Actual_Values['High'] - Predicted_values['High_Predicted']))
    MAD_Low = mean(absolute(Actual_Values['Low'] - Predicted_values['Low_Predicted']))
    print("Opening Price",MAD_Open,"\nClosing Price",MAD_Close,"\nHighest Price",MAD_High,"\nLowest Price",MAD_Low)

    print("\nMean Square Error (MSE)")
    MSE_Open = mean((Actual_Values['Open'] - Predicted_values['Open_Predicted'])**2)
    MSE_Close = mean((Actual_Values['Close'] - Predicted_values['Close_Predicted'])**2)
    MSE_High = mean((Actual_Values['High'] - Predicted_values['High_Predicted'])**2)
    MSE_Low = mean((Actual_Values['Low'] - Predicted_values['Low_Predicted'])**2)
    print("Opening Price",MSE_Open,"\nClosing Price",MSE_Close,"\nHighest Price",MSE_High,"\nLowest Price",MSE_Low)

    print("\nMean Average Percentage Error (MAPE)")
    MAPE_Open=mean(absolute(Actual_Values['Open'] - Predicted_values['Open_Predicted'])/data['Open']) * 100
    MAPE_Close=mean(absolute(Actual_Values['Close'] - Predicted_values['Close_Predicted'])/data['Close']) * 100
    MAPE_High=mean(absolute(Actual_Values['High'] - Predicted_values['High_Predicted'])/data['High']) * 100
    MAPE_Low=mean(absolute(Actual_Values['Low'] - Predicted_values['Low_Predicted'])/data['Low']) * 100
    print("Opening Price",MAPE_Open,"\nClosing Price",MAPE_Close,"\nHighest Price",MAPE_High,"\nLowest Price",MAPE_Low)
```


```python
def forcast_plot(train,valid):
    plt.style.use("fivethirtyeight")
    for i in train:
        if i in ['Open','Close','High','Low']:
            train.set_index('Date')[i].plot(figsize=(12, 10), linewidth=2.5, color='Black')
            valid.set_index('Date')[i+'_Predicted'].plot(figsize=(12, 10), linewidth=2.5, color='Orange')
            plt.xlabel('Datetime')
            plt.ylabel(i)
            plt.legend(['Training','Validation'])
            plt.show()
```
**Simple Moving Average.**

**Simple Moving Average helps to identify the trends in the data and smoothen the data and emphasizes more on the long term trends.**


```python

def my_simple_average(data,k):

    data['Open_Predicted'] = data['Open'].rolling(window=k).mean()
    data['Close_Predicted'] = data['Close'].rolling(window=k).mean()
    data['High_Predicted'] = data['High'].rolling(window=k).mean()
    data['Low_Predicted'] = data['Low'].rolling(window=k).mean()

    #plotting
    train = data[(data.Date < 'Mar 03 2017') & (data.Date >= 'Nov 01 2016')]
    valid = data[(data.Date <= 'Mar 03 2017') & (data.Date >= 'Nov 01 2016')]
    forcast_plot(train,valid)

```

**Simple Moving Average with window K=10.**


```python
my_simple_average(data,10)
```


![png](/images/TESLA/TESLA_STOCK_FORECAST_13_0.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_13_1.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_13_2.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_13_3.png)


**a.It can be observed that there has been an increasing trend for the last 5 months from the most recent data i.e from  'Nov 01 2016' & 'Mar 03 2017' for all the variables Open,Close,High and Low for TESLA stock listings.**

**Weighted Moving Average.**

**With weighted moving average we can emphasize more on the required lags by assigning higher weights.**


```python
def my_weighted_average1(data,k,weights):
    if (k != len(weights)):
        print("Error: Value of K is not equal to number of Weights or sum of weights not equals 1")
    else:
        df = pd.DataFrame(data)
        sum_weights = np.sum(weights)

        data['Open_Predicted'] = (data['Open'].rolling(window=k, center=True).apply(lambda x: np.sum(weights*x) / sum_weights, raw=False))

        data['Close_Predicted'] = (data['Close'].rolling(window=k, center=True).apply(lambda x: np.sum(weights*x) / sum_weights, raw=False))

        data['High_Predicted'] = (data['High'].rolling(window=k, center=True).apply(lambda x: np.sum(weights*x) / sum_weights, raw=False))

        data['Low_Predicted'] = (data['Low'].rolling(window=k, center=True).apply(lambda x: np.sum(weights*x) / sum_weights, raw=False))

        #plotting
        train = data[(data.Date < 'Mar 03 2017') & (data.Date >= 'Jan 01 2017')]
        valid = data[(data.Date <= 'Mar 03 2017') & (data.Date >= 'Feb 27 2017')]
        forcast_plot(train,valid)

        #Error metrics
        Actual_Values = data[(data.Date >= 'Feb 27 2017') & (data.Date <= 'Mar 03 2017')]
        Predicted_Values = valid[(data.Date >= 'Feb 27 2017') & (data.Date <= 'Mar 03 2017')]
        metric_cal(Actual_Values,Predicted_Values)
```

**Weighted Moving Average with k=2 and weights 0.10,0.90.**


```python
my_weighted_average1(data,2,[0.10,0.90])
```


![png](/images/TESLA/TESLA_STOCK_FORECAST_21_0.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_21_1.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_21_2.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_21_3.png)



    Mean Absolute Deviation Error (MAD)
    Opening Price 0.4791995399999962
    Closing Price 0.3222002999999972
    Highest Price 0.3866003199999966
    Lowest Price 0.3372000200000002

    Mean Square Error (MSE)
    Opening Price 0.3136842258806621
    Closing Price 0.26306254080056435
    Highest Price 0.24794711384021745
    Lowest Price 0.19806178208015973

    Mean Average Percentage Error (MAPE)
    Opening Price 0.19140518230030412
    Closing Price 0.13013901897308836
    Highest Price 0.15424636334739167
    Lowest Price 0.13764050948250642


**Weighted Moving Average with k=3 and weights 0.10 0.20,0.70.**


```python
my_weighted_average1(data,3,[0.10,0.20,0.70])
```


![png](/images/TESLA/TESLA_STOCK_FORECAST_23_0.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_23_1.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_23_2.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_23_3.png)



    Mean Absolute Deviation Error (MAD)
    Opening Price 3.421595379999991
    Closing Price 1.0922024799999803
    Highest Price 1.512601840000002
    Lowest Price 1.6840006199999777

    Mean Square Error (MSE)
    Opening Price 16.567181254635777
    Closing Price 2.925787462190145
    Highest Price 3.3630216422927965
    Lowest Price 3.875064512609213

    Mean Average Percentage Error (MAPE)
    Opening Price 1.3782707555443663
    Closing Price 0.44113298910125115
    Highest Price 0.602665915552746
    Lowest Price 0.6871837923711224

Error metric : Mean Square Error Metric for the K=2 and Weights 0.10,0.90 are far more better where maximum emphasis is on the recent lag 1 data.

<br>

**Checking Stationarity of Data.**


```python
#Dicky fuller test to check if the data is stationary or not.
#Null hypothesis : We have a non stationary dataset.
#Alternate Hypothesis :We have a stationary dataset.
#https://machinelearningmastery.com/time-series-data-stationary-python/
from statsmodels.tsa.stattools import adfuller

def dicky_fuller_test(input_col):
    X=input_col
    result = adfuller(X)
    print('ADF Statistic: %f' % result[0])
    print('p-value: %f' % result[1])
    print('Critical Values:')
    for key, value in result[4].items():
        print('\t%s: %.3f' % (key, value))
for i in data:
    if i in ['Open','Close','High','Low']:
        print("------------------------------------------")
        print('Test Column:',i)
        X=data.loc[:,i]
        dicky_fuller_test(X)

#p-value > 0.05: Fail to reject the null hypothesis (H0).
#p-value <= 0.05: Reject the null hypothesis (H0).
```

    ------------------------------------------
    Test Column: Open
    ADF Statistic: -0.895025
    p-value: 0.789606
    Critical Values:
    	1%: -3.434
    	5%: -2.863
    	10%: -2.568
    ------------------------------------------
    Test Column: High
    ADF Statistic: -0.887391
    p-value: 0.792096
    Critical Values:
    	1%: -3.434
    	5%: -2.863
    	10%: -2.568
    ------------------------------------------
    Test Column: Low
    ADF Statistic: -0.929777
    p-value: 0.778008
    Critical Values:
    	1%: -3.434
    	5%: -2.863
    	10%: -2.568
    ------------------------------------------
    Test Column: Close
    ADF Statistic: -0.897490
    p-value: 0.788798
    Critical Values:
    	1%: -3.434
    	5%: -2.863
    	10%: -2.568


**P-value for all the variables in the data > 0.05: Fail to reject the null hypothesis (H0).i.e We have a non stationary dataset**

<br>


```python
#KPSS test to check if the data is stationary or not.
#Null hypothesis : We have a trend stationary dataset.
#Alternate Hypothesis :We have a non stationary dataset.
#https://www.analyticsvidhya.com/blog/2018/09/non-stationary-time-series-python/

#define function for kpss test
from statsmodels.tsa.stattools import kpss
#define KPSS
def kpss_test(timeseries):
    print ('Results of KPSS Test:')
    kpsstest = kpss(timeseries, regression='c')
    kpss_output = pd.Series(kpsstest[0:3], index=['Test Statistic','p-value','Lags Used'])
    for key,value in kpsstest[3].items():
        kpss_output['Critical Value (%s)'%key] = value
    print (kpss_output)
for i in data:
    if i in ['Open','Close','High','Low']:
        print("------------------------------------------")
        print('Test Column:',i)
        X=data.loc[:,i]
        kpss_test(X)

#p-value > 0.05: Fail to reject the null hypothesis (H0).
#p-value <= 0.05: Reject the null hypothesis (H0).
```

    ------------------------------------------
    Test Column: Open
    Results of KPSS Test:
    Test Statistic            5.768672
    p-value                   0.010000
    Lags Used                25.000000
    Critical Value (10%)      0.347000
    Critical Value (5%)       0.463000
    Critical Value (2.5%)     0.574000
    Critical Value (1%)       0.739000
    dtype: float64
    ------------------------------------------
    Test Column: High
    Results of KPSS Test:
    Test Statistic            5.765299
    p-value                   0.010000
    Lags Used                25.000000
    Critical Value (10%)      0.347000
    Critical Value (5%)       0.463000
    Critical Value (2.5%)     0.574000
    Critical Value (1%)       0.739000
    dtype: float64
    ------------------------------------------
    Test Column: Low
    Results of KPSS Test:
    Test Statistic            5.774164
    p-value                   0.010000
    Lags Used                25.000000
    Critical Value (10%)      0.347000
    Critical Value (5%)       0.463000
    Critical Value (2.5%)     0.574000
    Critical Value (1%)       0.739000
    dtype: float64
    ------------------------------------------
    Test Column: Close
    Results of KPSS Test:
    Test Statistic            5.769559
    p-value                   0.010000
    Lags Used                25.000000
    Critical Value (10%)      0.347000
    Critical Value (5%)       0.463000
    Critical Value (2.5%)     0.574000
    Critical Value (1%)       0.739000
    dtype: float64


**p-value <= 0.05: Reject the null hypothesis (H0).i.e We have a non stationary dataset.**

**Checking the ADF and KPSS after differencing of the columns**


```python
#checking the ADF and KPSS after differencing of the columns.
for i in data:
    if i in ['Open','Close','High','Low']:
        print("Column:",i)
        x = data[i] - data[i].shift(1)
        Y = x.dropna()
        Y.plot()
        plt.show()
        print("Results of ADF Test:")
        dicky_fuller_test(Y)
        kpss_test(Y)
        print("-------------------------------------")
```

    Column: Open



![png](/images/TESLA/TESLA_STOCK_FORECAST_33_1.png)


    Results of ADF Test:
    ADF Statistic: -42.687738
    p-value: 0.000000
    Critical Values:
    	1%: -3.434
    	5%: -2.863
    	10%: -2.568
    Results of KPSS Test:
    Test Statistic            0.059567
    p-value                   0.100000
    Lags Used                25.000000
    Critical Value (10%)      0.347000
    Critical Value (5%)       0.463000
    Critical Value (2.5%)     0.574000
    Critical Value (1%)       0.739000
    dtype: float64
    -------------------------------------
    Column: High


![png](/images/TESLA/TESLA_STOCK_FORECAST_33_4.png)


    Results of ADF Test:
    ADF Statistic: -17.168738
    p-value: 0.000000
    Critical Values:
    	1%: -3.434
    	5%: -2.863
    	10%: -2.568
    Results of KPSS Test:
    Test Statistic            0.064424
    p-value                   0.100000
    Lags Used                25.000000
    Critical Value (10%)      0.347000
    Critical Value (5%)       0.463000
    Critical Value (2.5%)     0.574000
    Critical Value (1%)       0.739000
    dtype: float64
    -------------------------------------
    Column: Low


![png](/images/TESLA/TESLA_STOCK_FORECAST_33_7.png)


    Results of ADF Test:
    ADF Statistic: -28.950677
    p-value: 0.000000
    Critical Values:
    	1%: -3.434
    	5%: -2.863
    	10%: -2.568
    Results of KPSS Test:
    Test Statistic            0.058196
    p-value                   0.100000
    Lags Used                25.000000
    Critical Value (10%)      0.347000
    Critical Value (5%)       0.463000
    Critical Value (2.5%)     0.574000
    Critical Value (1%)       0.739000
    dtype: float64
    -------------------------------------
    Column: Close


![png](/images/TESLA/TESLA_STOCK_FORECAST_33_10.png)


    Results of ADF Test:
    ADF Statistic: -39.728000
    p-value: 0.000000
    Critical Values:
    	1%: -3.434
    	5%: -2.863
    	10%: -2.568
    Results of KPSS Test:
    Test Statistic            0.06279
    p-value                   0.10000
    Lags Used                25.00000
    Critical Value (10%)      0.34700
    Critical Value (5%)       0.46300
    Critical Value (2.5%)     0.57400
    Critical Value (1%)       0.73900
    dtype: float64
    -------------------------------------


**It can be seen that after differencing our data is now stationary as required and tested using the ADF and the KPSS tests.**

**Durbin-Watson Statistic -**
**Test statistic used to detect the presence of autocorrelation at lag 1 in the residuals from a regression analysis.**


```python
from statsmodels.stats.stattools import durbin_watson
print('DW statistic for Opening Price is: ',durbin_watson(tesla['Open']))
print('DW statistic for Closing Price is: ',durbin_watson(tesla['Close']))
print('DW statistic for Highest Price is: ',durbin_watson(tesla['High']))
print('DW statistic for Lowest Price is: ',durbin_watson(tesla['Low']))
```

    DW statistic for Opening Price is:  0.0008402831823626482
    DW statistic for Closing Price is:  0.0007333546150387083
    DW statistic for Highest Price is:  0.0005890708602447673
    DW statistic for Lowest Price is:  0.0007372830417820543


**Since DW statistic is close to 0, it mean that first order positive autocorrelation exists for opening price, closing price, highest price, lowest price.**

**ARIMA MODEL :**

**1.Check for the ACF AND PACF plots.**
**Help to detect the Autoregressive terms and moving average terms.**

**Autoregressive - forecast the next timestamp value by regressing over the previous values.**
**Moving average - forecast the nest timestamp value by regressing over the previous values.**

```python
from statsmodels.graphics.tsaplots import plot_acf
from statsmodels.graphics.tsaplots import plot_pacf
from matplotlib import pyplot

for i in data:
    if i in ['Open','Close','High','Low']:
        pyplot.figure()
        plt.figure(figsize=(12,20))
        pyplot.subplot(211)
        plot_acf(data[i], ax=pyplot.gca(),lags=30)
        pyplot.subplot(212)

        plot_pacf(data[i], ax=pyplot.gca())
        pyplot.show()
```


    <Figure size 432x288 with 0 Axes>

![png](/images/TESLA/TESLA_STOCK_FORECAST_42_1.png)

    <Figure size 432x288 with 0 Axes>

![png](/images/TESLA/TESLA_STOCK_FORECAST_42_3.png)

    <Figure size 432x288 with 0 Axes>

![png](/images/TESLA/TESLA_STOCK_FORECAST_42_5.png)

    <Figure size 432x288 with 0 Axes>

![png](/images/TESLA/TESLA_STOCK_FORECAST_42_7.png)

ARIMA(p,d,q)

as 1 differencing is sufficient to bring the data to stationary d = 1
q 1
P

```python
data.head()
data1 = data.copy()
```

```python
data[(data.Date < 'Mar 03 2017') & (data.Date >= 'Jan 01 2017')].head(5)
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
      <th>Open_Predicted</th>
      <th>Close_Predicted</th>
      <th>High_Predicted</th>
      <th>Low_Predicted</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1640</td>
      <td>2017-01-03</td>
      <td>214.860001</td>
      <td>220.330002</td>
      <td>210.960007</td>
      <td>216.990005</td>
      <td>216.990005</td>
      <td>5923300</td>
      <td>214.927000</td>
      <td>223.660005</td>
      <td>225.416000</td>
      <td>213.376999</td>
    </tr>
    <tr>
      <td>1641</td>
      <td>2017-01-04</td>
      <td>214.750000</td>
      <td>228.000000</td>
      <td>214.309998</td>
      <td>226.990005</td>
      <td>226.990005</td>
      <td>11213500</td>
      <td>222.929999</td>
      <td>225.822001</td>
      <td>226.868997</td>
      <td>219.322998</td>
    </tr>
    <tr>
      <td>1642</td>
      <td>2017-01-05</td>
      <td>226.419998</td>
      <td>227.479996</td>
      <td>221.949997</td>
      <td>226.750000</td>
      <td>226.750000</td>
      <td>5911700</td>
      <td>225.609995</td>
      <td>228.355997</td>
      <td>229.512998</td>
      <td>223.635997</td>
    </tr>
    <tr>
      <td>1643</td>
      <td>2017-01-06</td>
      <td>226.929993</td>
      <td>230.309998</td>
      <td>225.449997</td>
      <td>229.009995</td>
      <td>229.009995</td>
      <td>5527900</td>
      <td>228.306999</td>
      <td>230.372998</td>
      <td>231.153998</td>
      <td>226.884999</td>
    </tr>
    <tr>
      <td>1644</td>
      <td>2017-01-09</td>
      <td>228.970001</td>
      <td>231.919998</td>
      <td>228.000000</td>
      <td>231.279999</td>
      <td>231.279999</td>
      <td>3979500</td>
      <td>230.886999</td>
      <td>230.065996</td>
      <td>231.814999</td>
      <td>226.967999</td>
    </tr>
  </tbody>
</table>
</div>




```python
def ARIMA_model(data):

    ARIMA_model_1 = ARIMA(data['Open'],order=(0,1,0)).fit(transparam=False)
    ARIMA_model_2 = ARIMA(data['High'],order=(1,1,0)).fit(transparam=False)
    ARIMA_model_3 = ARIMA(data['Low'],order=(1,1,0)).fit(transparam=False)
    ARIMA_model_4 = ARIMA(data['Close'],order=(0,1,0)).fit(transparam=False)
    ARIMA_model_5 = ARIMA(data['Volume'],order=(1,1,1)).fit(transparam=False)

    print(ARIMA_model_1.summary())
    print(ARIMA_model_2.summary())
    print(ARIMA_model_3.summary())
    print(ARIMA_model_4.summary())
    print(ARIMA_model_5.summary())                    

    data['Open'+'_Predicted'] = ARIMA_model_1.predict(typ='levels')     
    data['Close'+'_Predicted'] = ARIMA_model_2.predict(typ='levels')     
    data['High'+'_Predicted'] = ARIMA_model_3.predict(typ='levels')     
    data['Low'+'_Predicted'] = ARIMA_model_4.predict(typ='levels')          

    train = data[(data.Date < 'Mar 07 2017') & (data.Date >= 'Jan 01 2017')]
    valid = data[(data.Date <= 'Mar 07 2017') & (data.Date >= 'Mar 03 2017')]
    forcast_plot(train,valid)

    #Error metrics
    Actual_Values = data[(data.Date >= 'Feb 27 2017') & (data.Date <= 'Mar 03 2017')]
    Predicted_Values = valid[(data.Date >= 'Feb 27 2017') & (data.Date <= 'Mar 03 2017')]

    metric_cal(Actual_Values,Predicted_Values)

    #forecasting next week values
    print("----------------------------------------------------------------------")
    print("As the MSE observed with the validation data is comparatively less we be Using the above performed ARIMA model we would be making the predictions for the week Mar 6 to Mar 12 2017.")
    print("Forecasts for the next 7 Business Days Week from 6th March 2017:")

    forecast_1 = ARIMA_model_1.forecast(steps=7,exog=None, alpha=0.05)
    forecast_2 = ARIMA_model_2.forecast(steps=7,exog=None, alpha=0.05)
    forecast_3 = ARIMA_model_3.forecast(steps=7,exog=None, alpha=0.05)
    forecast_4 = ARIMA_model_4.forecast(steps=7,exog=None, alpha=0.05)

    date=['2017-03-06','2017-03-07','2017-03-08','2017-03-09','2017-03-10','2017-03-11','2017-03-12']
    pred=pd.DataFrame({'Date':date,'Open_Predicted':forecast_1[0],'Close_Predicted':forecast_2[0],'High_Predicted':forecast_3[0],'Low_Predicted':forecast_4[0]})

    pred['Date']= pd.to_datetime(pred['Date'])

    def prediction_plot(train,valid):
        plt.style.use("fivethirtyeight")
        for i in train:
            if i in ['Open','Close','High','Low']:
                train.set_index('Date')[i].plot(figsize=(12, 10), linewidth=2.5, color='Black')
                valid.set_index('Date')[i+'_Predicted'].plot(figsize=(12, 10), linewidth=2.5, color='Orange')
                plt.xlabel('Datetime')
                plt.ylabel(i)
                plt.legend(['Training','Forecast'])
                plt.show()

    prediction_plot(train,pred)

```


```python
ARIMA_model(data)
```

                                 ARIMA Model Results                              
    ==============================================================================
    Dep. Variable:                 D.Open   No. Observations:                 1683
    Model:                 ARIMA(0, 1, 0)   Log Likelihood               -4991.269
    Method:                           css   S.D. of innovations              4.696
    Date:                Sat, 21 Mar 2020   AIC                           9986.538
    Time:                        17:21:49   BIC                           9997.394
    Sample:                             1   HQIC                          9990.559

    ==============================================================================
                     coef    std err          z      P>|z|      [0.025      0.975]
    ------------------------------------------------------------------------------
    const          0.1384      0.114      1.209      0.227      -0.086       0.363
    ==============================================================================
                                 ARIMA Model Results                              
    ==============================================================================
    Dep. Variable:                 D.High   No. Observations:                 1683
    Model:                 ARIMA(1, 1, 0)   Log Likelihood               -4704.443
    Method:                       css-mle   S.D. of innovations              3.960
    Date:                Sat, 21 Mar 2020   AIC                           9414.887
    Time:                        17:21:49   BIC                           9431.172
    Sample:                             1   HQIC                          9420.919

    ================================================================================
                       coef    std err          z      P>|z|      [0.025      0.975]
    --------------------------------------------------------------------------------
    const            0.1367      0.112      1.225      0.221      -0.082       0.355
    ar.L1.D.High     0.1350      0.024      5.587      0.000       0.088       0.182
                                        Roots                                    
    =============================================================================
                      Real          Imaginary           Modulus         Frequency
    -----------------------------------------------------------------------------
    AR.1            7.4079           +0.0000j            7.4079            0.0000
    -----------------------------------------------------------------------------
                                 ARIMA Model Results                              
    ==============================================================================
    Dep. Variable:                  D.Low   No. Observations:                 1683
    Model:                 ARIMA(1, 1, 0)   Log Likelihood               -4846.742
    Method:                       css-mle   S.D. of innovations              4.310
    Date:                Sat, 21 Mar 2020   AIC                           9699.485
    Time:                        17:21:49   BIC                           9715.770
    Sample:                             1   HQIC                          9705.517

    ===============================================================================
                      coef    std err          z      P>|z|      [0.025      0.975]
    -------------------------------------------------------------------------------
    const           0.1374      0.113      1.211      0.226      -0.085       0.360
    ar.L1.D.Low     0.0740      0.024      3.042      0.002       0.026       0.122
                                        Roots                                    
    =============================================================================
                      Real          Imaginary           Modulus         Frequency
    -----------------------------------------------------------------------------
    AR.1           13.5184           +0.0000j           13.5184            0.0000
    -----------------------------------------------------------------------------
                                 ARIMA Model Results                              
    ==============================================================================
    Dep. Variable:                D.Close   No. Observations:                 1683
    Model:                 ARIMA(0, 1, 0)   Log Likelihood               -4876.558
    Method:                           css   S.D. of innovations              4.387
    Date:                Sat, 21 Mar 2020   AIC                           9757.116
    Time:                        17:21:49   BIC                           9767.973
    Sample:                             1   HQIC                          9761.138

    ==============================================================================
                     coef    std err          z      P>|z|      [0.025      0.975]
    ------------------------------------------------------------------------------
    const          0.1335      0.107      1.249      0.212      -0.076       0.343
    ==============================================================================
                                 ARIMA Model Results                              
    ==============================================================================
    Dep. Variable:               D.Volume   No. Observations:                 1683
    Model:                 ARIMA(1, 1, 1)   Log Likelihood              -27217.917
    Method:                       css-mle   S.D. of innovations        2553533.029
    Date:                Sat, 21 Mar 2020   AIC                          54443.833
    Time:                        17:21:50   BIC                          54465.547
    Sample:                             1   HQIC                         54451.876

    ==================================================================================
                         coef    std err          z      P>|z|      [0.025      0.975]
    ----------------------------------------------------------------------------------
    const          -9094.9495   8732.245     -1.042      0.298   -2.62e+04    8019.937
    ar.L1.D.Volume     0.5120      0.028     18.580      0.000       0.458       0.566
    ma.L1.D.Volume    -0.9390      0.013    -69.834      0.000      -0.965      -0.913
                                        Roots                                    
    =============================================================================
                      Real          Imaginary           Modulus         Frequency
    -----------------------------------------------------------------------------
    AR.1            1.9530           +0.0000j            1.9530            0.0000
    MA.1            1.0650           +0.0000j            1.0650            0.0000
    -----------------------------------------------------------------------------



![png](/images/TESLA/TESLA_STOCK_FORECAST_47_1.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_47_2.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_47_3.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_47_4.png)


    Mean Absolute Deviation Error (MAD)
    Opening Price 0.8916022792631964
    Closing Price 1.6162856763898503
    Highest Price 3.5648699624862275
    Lowest Price 1.6135075846702307

    Mean Square Error (MSE)
    Opening Price 0.7949546243873268
    Closing Price 2.612379387702996
    Highest Price 12.708297849436557
    Lowest Price 2.6034067257883615

    Mean Average Percentage Error (MAPE)
    Opening Price 0.3555883630389161
    Closing Price 0.6424794814231771
    Highest Price 1.415192555537031
    Lowest Price 0.6479950139237874
    ----------------------------------------------------------------------
    As the MSE observed with the validation data is comparatively less we be Using the above performed ARIMA model we would be making the predictions for the week Mar 6 to Mar 12 2017.
    Forecasts for the next 7 Business Days Week from 6th March 2017:



![png](/images/TESLA/TESLA_STOCK_FORECAST_47_7.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_47_8.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_47_9.png)



![png](/images/TESLA/TESLA_STOCK_FORECAST_47_10.png)
