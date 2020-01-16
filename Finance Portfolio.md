```python
#Dashboard article
#https://towardsdatascience.com/python-for-finance-dash-by-plotly-ccf84045b8be

# Importing initial libraries.
import pandas as pd
import numpy as np
import datetime
import os
import matplotlib.pyplot as plt
import plotly.graph_objs as go
import bs4
import requests
from bs4 import BeautifulSoup
from pandas_datareader import data, wb
from alpha_vantage.timeseries import TimeSeries
import time
%matplotlib inline

```


```python
"""

I have embarked on a project to explore the difference between investing in ETFs vs individual stocks¶
The goal of this project is to eventually create an application that houses all of an individual's investment accounts, with the ability to compare against benchmarks, and also visualize predicted gains for retirement, showing the importance of investing early and often.

In this notebook, I will be working with an API, and visualizing some basic EDA

"""


```


```python
api_key = 'TN6KAGZRKLYYKWVL'
```


```python
ts = TimeSeries(key=api_key, output_format='pandas')
VTI, meta_data = ts.get_daily_adjusted(symbol='VTI', outputsize = 'full')
VTI.tail()
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
      <th>1. open</th>
      <th>2. high</th>
      <th>3. low</th>
      <th>4. close</th>
      <th>5. adjusted close</th>
      <th>6. volume</th>
      <th>7. dividend amount</th>
      <th>8. split coefficient</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2001-06-06</td>
      <td>117.5</td>
      <td>117.8</td>
      <td>116.7</td>
      <td>116.8</td>
      <td>41.2633</td>
      <td>278500.0</td>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>2001-06-05</td>
      <td>116.4</td>
      <td>118.0</td>
      <td>116.4</td>
      <td>117.8</td>
      <td>41.6166</td>
      <td>562400.0</td>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>2001-06-04</td>
      <td>116.1</td>
      <td>116.2</td>
      <td>115.3</td>
      <td>116.1</td>
      <td>41.0160</td>
      <td>1018200.0</td>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>2001-06-01</td>
      <td>115.1</td>
      <td>115.9</td>
      <td>114.4</td>
      <td>115.6</td>
      <td>40.8394</td>
      <td>2542200.0</td>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <td>2001-05-31</td>
      <td>114.5</td>
      <td>115.5</td>
      <td>114.5</td>
      <td>114.8</td>
      <td>40.5568</td>
      <td>2457200.0</td>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
close_data = VTI['4. close']
close_data
```




    date
    2020-01-14    166.6068
    2020-01-13    166.5900
    2020-01-10    165.4600
    2020-01-09    165.9400
    2020-01-08    164.9100
                    ...   
    2001-06-06    116.8000
    2001-06-05    117.8000
    2001-06-04    116.1000
    2001-06-01    115.6000
    2001-05-31    114.8000
    Name: 4. close, Length: 4685, dtype: float64




```python
percentage_change = close_data.pct_change()
percentage_change.tail()
```




    date
    2001-06-06   -0.005111
    2001-06-05    0.008562
    2001-06-04   -0.014431
    2001-06-01   -0.004307
    2001-05-31   -0.006920
    Name: 4. close, dtype: float64




```python

TSLA, meta_data = ts.get_daily_adjusted(symbol='TSLA', outputsize = 'full')
TSLA = TSLA['4. close']

BND, meta_data = ts.get_daily_adjusted(symbol='BND', outputsize = 'full')
BND = BND['4. close']

FB, meta_data = ts.get_daily_adjusted(symbol='FB', outputsize = 'full')
FB = FB['4. close']

VXUS, meta_data = ts.get_daily_adjusted(symbol='VXUS', outputsize = 'full')
VXUS = VXUS['4. close']

VTI = VTI['4. close']
```


```python
tickers = ['TSLA', 'BND', 'FB', 'VTI', 'VXUS']
```


```python
stock_close = pd.concat([TSLA, BND, FB, VTI, VXUS], axis=1, keys=tickers)
stock_close.head()
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
      <th>TSLA</th>
      <th>BND</th>
      <th>FB</th>
      <th>VTI</th>
      <th>VXUS</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2001-05-31</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>114.8</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2001-06-01</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>115.6</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2001-06-04</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>116.1</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2001-06-05</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>117.8</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2001-06-06</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>116.8</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
stock_close.columns.names = ['Bank Ticker']
stock_close.dropna(thresh=5, inplace=True)
stock_close.head()
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
      <th>Bank Ticker</th>
      <th>TSLA</th>
      <th>BND</th>
      <th>FB</th>
      <th>VTI</th>
      <th>VXUS</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2012-05-18</td>
      <td>27.56</td>
      <td>84.09</td>
      <td>38.2318</td>
      <td>66.42</td>
      <td>39.96</td>
    </tr>
    <tr>
      <td>2012-05-21</td>
      <td>28.77</td>
      <td>84.10</td>
      <td>34.0300</td>
      <td>67.65</td>
      <td>40.73</td>
    </tr>
    <tr>
      <td>2012-05-22</td>
      <td>30.79</td>
      <td>83.96</td>
      <td>31.0000</td>
      <td>67.67</td>
      <td>40.61</td>
    </tr>
    <tr>
      <td>2012-05-23</td>
      <td>31.02</td>
      <td>84.02</td>
      <td>32.0000</td>
      <td>67.79</td>
      <td>40.29</td>
    </tr>
    <tr>
      <td>2012-05-24</td>
      <td>30.32</td>
      <td>83.93</td>
      <td>33.0300</td>
      <td>67.92</td>
      <td>40.11</td>
    </tr>
  </tbody>
</table>
</div>




```python
for tick in tickers:
    print(str(tick) + ' ' + str(stock_close[tick].max()))
```

    TSLA 541.05
    BND 85.36
    FB 221.91
    VTI 166.6068
    VXUS 61.17



```python
returns = pd.DataFrame()
returns
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
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
for tick in tickers:
    returns[tick + ' Return'] = stock_close[tick].pct_change()
returns.tail()
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
      <th>TSLA Return</th>
      <th>BND Return</th>
      <th>FB Return</th>
      <th>VTI Return</th>
      <th>VXUS Return</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2020-01-08</td>
      <td>0.049205</td>
      <td>-0.001546</td>
      <td>0.010138</td>
      <td>0.004936</td>
      <td>0.001799</td>
    </tr>
    <tr>
      <td>2020-01-09</td>
      <td>-0.021945</td>
      <td>0.001191</td>
      <td>0.014311</td>
      <td>0.006246</td>
      <td>0.004488</td>
    </tr>
    <tr>
      <td>2020-01-10</td>
      <td>-0.006627</td>
      <td>0.001665</td>
      <td>-0.001099</td>
      <td>-0.002893</td>
      <td>-0.001251</td>
    </tr>
    <tr>
      <td>2020-01-13</td>
      <td>0.097689</td>
      <td>-0.000594</td>
      <td>0.017656</td>
      <td>0.006829</td>
      <td>0.007516</td>
    </tr>
    <tr>
      <td>2020-01-14</td>
      <td>0.030846</td>
      <td>0.001129</td>
      <td>-0.009148</td>
      <td>0.000101</td>
      <td>-0.000355</td>
    </tr>
  </tbody>
</table>
</div>




```python
#returns[1:]
import seaborn as sns
sns.pairplot(returns[1:])
```




    <seaborn.axisgrid.PairGrid at 0x1c2c763550>




![png](output_13_1.png)



```python
# Worst Drop 
returns.idxmin()
# returns.min()
```




    TSLA Return   2013-11-06
    BND Return    2013-07-05
    FB Return     2018-07-26
    VTI Return    2015-08-24
    VXUS Return   2016-06-24
    dtype: datetime64[ns]




```python
returns.idxmax()

```




    TSLA Return   2013-05-09
    BND Return    2013-09-18
    FB Return     2013-07-25
    VTI Return    2018-12-26
    VXUS Return   2012-06-29
    dtype: datetime64[ns]




```python
returns.std()
```




    TSLA Return    0.031136
    BND Return     0.002063
    FB Return      0.022751
    VTI Return     0.008242
    VXUS Return    0.009066
    dtype: float64




```python
returns.loc['2015-01-01':'2015-12-31'].std()
```




    TSLA Return    0.024470
    BND Return     0.002395
    FB Return      0.016174
    VTI Return     0.009654
    VXUS Return    0.010589
    dtype: float64




```python
sns.distplot(returns.ix['2015-01-01':'2015-12-31']['VXUS Return'],color='green',bins=50)
```

    //anaconda3/lib/python3.7/site-packages/ipykernel_launcher.py:1: FutureWarning:
    
    
    .ix is deprecated. Please use
    .loc for label based indexing or
    .iloc for positional indexing
    
    See the documentation here:
    http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#ix-indexer-is-deprecated
    





    <matplotlib.axes._subplots.AxesSubplot at 0x1c2c933978>




![png](output_18_2.png)



```python
sns.distplot(returns.ix['2015-01-01':'2015-12-31']['FB Return'],color='blue',bins=50)
```

    //anaconda3/lib/python3.7/site-packages/ipykernel_launcher.py:1: FutureWarning:
    
    
    .ix is deprecated. Please use
    .loc for label based indexing or
    .iloc for positional indexing
    
    See the documentation here:
    http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#ix-indexer-is-deprecated
    





    <matplotlib.axes._subplots.AxesSubplot at 0x1c2d191c50>




![png](output_19_2.png)



```python
# Optional Plotly Method Imports
import plotly
import cufflinks as cf
cf.go_offline()
```


<script type="text/javascript">
window.PlotlyConfig = {MathJaxConfig: 'local'};
if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}
if (typeof require !== 'undefined') {
require.undef("plotly");
requirejs.config({
    paths: {
        'plotly': ['https://cdn.plot.ly/plotly-latest.min']
    }
});
require(['plotly'], function(Plotly) {
    window._Plotly = Plotly;
});
}
</script>




```python
for tick in tickers:
    stock_close[tick].plot(figsize=(24,8),label=tick)
plt.legend()
```




    <matplotlib.legend.Legend at 0x1c2d288f28>




![png](output_21_1.png)



```python
stock_close.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1c2d4149b0>




![png](output_22_1.png)



```python
#bank_stocks.xs(key='Close',axis=1,level='Stock Info').iplot()
AAPL2, meta_data = ts.get_daily_adjusted(symbol='AAPL', outputsize = 'full')
AAPL2.ix['2014-01-01':'2014-12-31']
```

    //anaconda3/lib/python3.7/site-packages/ipykernel_launcher.py:3: FutureWarning:
    
    
    .ix is deprecated. Please use
    .loc for label based indexing or
    .iloc for positional indexing
    
    See the documentation here:
    http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#ix-indexer-is-deprecated
    





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
      <th>1. open</th>
      <th>2. high</th>
      <th>3. low</th>
      <th>4. close</th>
      <th>5. adjusted close</th>
      <th>6. volume</th>
      <th>7. dividend amount</th>
      <th>8. split coefficient</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
stock_close
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
      <th>Bank Ticker</th>
      <th>TSLA</th>
      <th>BND</th>
      <th>FB</th>
      <th>VTI</th>
      <th>VXUS</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2012-05-18</td>
      <td>27.56</td>
      <td>84.090</td>
      <td>38.2318</td>
      <td>66.4200</td>
      <td>39.96</td>
    </tr>
    <tr>
      <td>2012-05-21</td>
      <td>28.77</td>
      <td>84.100</td>
      <td>34.0300</td>
      <td>67.6500</td>
      <td>40.73</td>
    </tr>
    <tr>
      <td>2012-05-22</td>
      <td>30.79</td>
      <td>83.960</td>
      <td>31.0000</td>
      <td>67.6700</td>
      <td>40.61</td>
    </tr>
    <tr>
      <td>2012-05-23</td>
      <td>31.02</td>
      <td>84.020</td>
      <td>32.0000</td>
      <td>67.7900</td>
      <td>40.29</td>
    </tr>
    <tr>
      <td>2012-05-24</td>
      <td>30.32</td>
      <td>83.930</td>
      <td>33.0300</td>
      <td>67.9200</td>
      <td>40.11</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>2020-01-08</td>
      <td>492.14</td>
      <td>83.970</td>
      <td>215.2200</td>
      <td>164.9100</td>
      <td>55.70</td>
    </tr>
    <tr>
      <td>2020-01-09</td>
      <td>481.34</td>
      <td>84.070</td>
      <td>218.3000</td>
      <td>165.9400</td>
      <td>55.95</td>
    </tr>
    <tr>
      <td>2020-01-10</td>
      <td>478.15</td>
      <td>84.210</td>
      <td>218.0600</td>
      <td>165.4600</td>
      <td>55.88</td>
    </tr>
    <tr>
      <td>2020-01-13</td>
      <td>524.86</td>
      <td>84.160</td>
      <td>221.9100</td>
      <td>166.5900</td>
      <td>56.30</td>
    </tr>
    <tr>
      <td>2020-01-14</td>
      <td>541.05</td>
      <td>84.255</td>
      <td>219.8800</td>
      <td>166.6068</td>
      <td>56.28</td>
    </tr>
  </tbody>
</table>
<p>1926 rows × 5 columns</p>
</div>




```python

```


```python

```


```python

```
