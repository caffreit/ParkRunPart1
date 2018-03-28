
## Since the beginning of this year my girlfriend has dragged me out to the local Park Run(link). To say I am a reluctant participant is an understatement. However, she also told me that all past runs are recorded and are publically available.<br> Given the records go back several years and have roughly 80 thousand entries I thought it would make for an interesting dataset and a great opportunity to brush up my data science skills.

## This is the first in a series of posts. At the moment I have another six in the works.<br> Though that might change.

## This post will begin by looking at how the "Finish Times" and "Runner Count" vary over time.<br> It then moves on to finding a relationship (if any) of the "Finish Times" with "Age" and "Runner Count".<br>It finshes with a similar analysis of the Ratio between male and female athletes/runners.

## My hope is that I can tell a fun story and that, possibly, by the end we arrive at an interesting conclusion. 


```python

```


```python

```


```python

```


```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
import scipy.stats as stats
%matplotlib inline
from statsmodels.tsa.seasonal import seasonal_decompose
```

    C:\ProgramData\Anaconda2\lib\site-packages\statsmodels\compat\pandas.py:56: FutureWarning: The pandas.core.datetools module is deprecated and will be removed in a future version. Please use the pandas.tseries module instead.
      from pandas.core import datetools
    

### Import the data file.<br>Some cleaning has already been done in Notepad and Excel.<br>Excel did not want to play nice with unicode datetime. CHrist!


```python
path_to_file = 'C:\Users\Administrator\Documents\Python Scripts\examplepark.csv'
data = pd.read_csv(path_to_file)
```

#### We need to convert some numerical values from strings to floats so we can work with them.<br>Then print out the first few entries to check it worked as expected.


```python
data['Time'] = ((pd.to_numeric(data['Time'].str.slice(0,2)))*60)+\
                (pd.to_numeric(data['Time'].str.slice(3,5)))+((pd.to_numeric(data['Time'].str.slice(6,8)))/60)
data['Date'] = pd.to_datetime(data['Date'],errors='coerce', format='%d-%m-%Y')
data['Age_Cat'] = pd.to_numeric(data['Age_Cat'].str.slice(2,4),errors='coerce', downcast='signed')
data['Age_Grade'] = pd.to_numeric(data['Age_Grade'].str.slice(0,5),errors='coerce')
data.head(10)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Pos</th>
      <th>Name</th>
      <th>Time</th>
      <th>Age_Cat</th>
      <th>Age_Grade</th>
      <th>Gender</th>
      <th>Gen_Pos</th>
      <th>Club</th>
      <th>Note</th>
      <th>Total_Runs</th>
      <th>Run_No.</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2012-11-10</td>
      <td>1</td>
      <td>Michael MCSWIGGAN</td>
      <td>18.316667</td>
      <td>35.0</td>
      <td>73.43</td>
      <td>M</td>
      <td>1.0</td>
      <td>Portmarnock Athletic Club</td>
      <td>First Timer!</td>
      <td>29.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2012-11-10</td>
      <td>2</td>
      <td>Alan FOLEY</td>
      <td>18.433333</td>
      <td>30.0</td>
      <td>71.16</td>
      <td>M</td>
      <td>2.0</td>
      <td>Raheny Shamrock AC</td>
      <td>First Timer!</td>
      <td>99.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2012-11-10</td>
      <td>3</td>
      <td>Matt SHIELDS</td>
      <td>18.533333</td>
      <td>55.0</td>
      <td>85.07</td>
      <td>M</td>
      <td>3.0</td>
      <td>North Belfast Harriers</td>
      <td>First Timer!</td>
      <td>274.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2012-11-10</td>
      <td>4</td>
      <td>David GARGAN</td>
      <td>18.650000</td>
      <td>40.0</td>
      <td>73.73</td>
      <td>M</td>
      <td>4.0</td>
      <td>Raheny Shamrock AC</td>
      <td>First Timer!</td>
      <td>107.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2012-11-10</td>
      <td>5</td>
      <td>Paul SINTON-HEWITT</td>
      <td>18.900000</td>
      <td>50.0</td>
      <td>79.28</td>
      <td>M</td>
      <td>5.0</td>
      <td>Ranelagh Harriers</td>
      <td>First Timer!</td>
      <td>369.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2012-11-10</td>
      <td>6</td>
      <td>John Gerard MURPHY</td>
      <td>20.250000</td>
      <td>40.0</td>
      <td>68.97</td>
      <td>M</td>
      <td>6.0</td>
      <td>North Belfast Harriers</td>
      <td>First Timer!</td>
      <td>342.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2012-11-10</td>
      <td>7</td>
      <td>Conor FITZPATRICK</td>
      <td>20.283333</td>
      <td>20.0</td>
      <td>64.26</td>
      <td>M</td>
      <td>7.0</td>
      <td>Portmarnock Athletic Club</td>
      <td>First Timer!</td>
      <td>40.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2012-11-10</td>
      <td>8</td>
      <td>Rachael BECK</td>
      <td>20.450000</td>
      <td>40.0</td>
      <td>76.37</td>
      <td>F</td>
      <td>1.0</td>
      <td>Fingal Triathlon Club</td>
      <td>First Timer!</td>
      <td>9.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2012-11-10</td>
      <td>9</td>
      <td>Des HUSIN</td>
      <td>20.533333</td>
      <td>45.0</td>
      <td>69.07</td>
      <td>M</td>
      <td>8.0</td>
      <td>NaN</td>
      <td>First Timer!</td>
      <td>296.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2012-11-10</td>
      <td>10</td>
      <td>John COLEMAN</td>
      <td>20.816667</td>
      <td>30.0</td>
      <td>63.01</td>
      <td>M</td>
      <td>9.0</td>
      <td>NaN</td>
      <td>First Timer!</td>
      <td>87.0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



#### The most obvious thing to try is how does the number of runners "Runner Count" change over time. Is there a trend of increasing attendence? Is there any seasonality, where the count drops in the winter?


```python
ax = data.groupby('Date').count()['Pos'].plot.line(figsize=(15, 6), fontsize=20)
ax.set_xlabel("Date", fontsize=20)
ax.set_ylabel("Runner Count", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
```


![png](/img/output_13_0.png)


#### Even by eye we can see some periodicity in the Runner Count. There also seems to be a drop in average attendence from roughly 350 runners to somewhere between 2 and 3 hundred runners after 2015.<br>We can use a tool from statsmodel to decompose the trend and the seasonality from the data. Below are the outputted plots.


```python
result = seasonal_decompose(data.groupby('Date').count()['Pos'],\
                            model='additive',freq=52)
fig = result.plot()
fig.set_size_inches(15,6)
```


![png](/img/output_15_0.png)


#### As we thought there is a definite periodicity in attendence. Peaking just after the beginning of a new year, no doubt reflecting all the New Year Resolutions made. This is followed by a steady decline over the course of the year reaching a minimum around Christmas.

#### We can also observe more clearly the drop in average attendence. I've no insight into what may ahve caused the drop across 2015. If anybody has any thoughts feel free to get in contact. ivan.caffrey@gmail.com

#### There doesn't seem to be anything interesting left over in the residuals.

## Let's move onto "Finish Times".

#### The first thing I'm going to do is, for each race, find the Minimum (first place), Maximum(last place) and Mean Finish times. I'll then plot these times for each date.


```python

```


```python
ax = data.groupby('Date').max()['Time'].plot.line(figsize=(15, 6), fontsize=20)
ax = data.groupby('Date').mean()['Time'].plot.line()
ax = data.groupby('Date').min()['Time'].plot.line()
ax.set_xlabel("Date", fontsize=20)
ax.set_ylabel("Finish Time (minute)", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
ax.legend(['Max','Mean','Min'], fontsize=18)
```




    <matplotlib.legend.Legend at 0x11476ba8>




![png](/img/output_20_1.png)


#### There doesn't seem to be much going on in any of the plots. But first I'm going to get a closer look at the Mean and Minimum times.


```python
ax = data.groupby('Date').mean()['Time'].plot.line(figsize=(15, 6), fontsize=20)
ax = data.groupby('Date').min()['Time'].plot.line()
ax.set_xlabel("Date", fontsize=20)
ax.set_ylabel("Finish Time (minute)", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
ax.legend(['Mean','Min'], fontsize=18)
```




    <matplotlib.legend.Legend at 0x117bee48>




![png](/img/output_22_1.png)


#### There is a distinct drop in the first place finishing times in mid 2014. I looked at the raw data to see why this may be. The cause is a small group of runners who all run the course in about 15 minutes. They appear suddenly, dominate the leaderboard and then leave never to be seen again. This drop shows up in the decomposition of trend and seasonality as well.


```python
result = seasonal_decompose(data.groupby('Date').min()['Time'],\
                            model='additive',freq=52)
fig = result.plot()
fig.set_size_inches(15,6)
```


![png](/img/output_24_0.png)


#### Repeating the decomposition on the mean times, we see a seasonality. Just like the runner count, it peaks just after New Years Day and gradually decreases. Again, this relationship is not that surprising. If all the new "athletes" are only running because of a resolution and do not ordinarily spend their free time pounding the pavements it stands to reason the mean finish time would increase.


```python
result = seasonal_decompose(data.groupby('Date').mean()['Time'],\
                            model='additive',freq=52)
fig = result.plot()
fig.set_size_inches(15,6)
```


![png](/img/output_26_0.png)


#### Given that the mean finish time and the runner count both display seasonality, we should see a  positive dependence of mean time with runner count. So let's plot the Max, Mean and Min times against Runner Count.


```python
x = data.groupby('Date').count()['Pos']
ymax = data.groupby('Date').max()['Time']
ymean = data.groupby('Date').mean()['Time']
ymin = data.groupby('Date').min()['Time']
```


```python
plt.figure(figsize=(16,12))
plt.scatter(x,ymax, label='Max')
plt.scatter(x,ymean, label='Mean')
plt.scatter(x,ymin, label='Min')
#plt.title('sometitle')
plt.xlabel('Runner Count', fontsize=18)
plt.ylabel('Finish Time', fontsize=18)
plt.xticks(fontsize=18)
plt.yticks(fontsize=18)
plt.legend(fontsize=18)
plt.show()
```


![png](/img/output_29_0.png)


#### Zooming in on the mean and min times we observe a clear increase in mean finish time with runner count. Interestingly, we also see a decrease in the min finish time. While we would expect an increase in runner count to "drag up" the mean time. This effect would not apply to the first place times since the slow fairweather runners won't affect the times of the hardcore roadrunners. In fact we would expect that as attendence increases there would be more hardcore runners who run the course in ever faster times.


```python
plt.figure(figsize=(16,12))
plt.scatter(x,ymean, label='Mean')
plt.scatter(x,ymin, label='Min')
plt.xlabel('Runner Count', fontsize=18)
plt.ylabel('Finish Time', fontsize=18)
plt.xticks(fontsize=18)
plt.yticks(fontsize=18)
plt.legend(fontsize=18)
plt.show()
```


![png](/img/output_31_0.png)


#### For fun I said I'd plot the standard deviation of the finish times. When we decompose the time series we see the same seasonality as in Runner Count and mean finish time.


```python
ax = data.groupby('Date').std()['Time'].plot.line(figsize=(15, 6), fontsize=20)
ax.set_xlabel("Date", fontsize=20)
ax.set_ylabel("Std Time (minute)", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
```


![png](/img/output_33_0.png)



```python
result = seasonal_decompose(data.groupby('Date').std()['Time'],\
                            model='additive',freq=52)
fig = result.plot()
fig.set_size_inches(15,6)
```


![png](/img/output_34_0.png)


#### We see the positive dependence of the standard deviation of the finish times and the mean finish times and the runner count. As a larger number of slower people attend, naturally we should expect an increase in the standard deviation.

## Put these as subplots side by side.


```python
ystd = data.groupby('Date').std()['Time']
plt.figure(figsize=(8,6))
plt.scatter(x,ystd)
#plt.title('sometitle')
plt.xlabel('Runner Count', fontsize=18)
plt.ylabel('Std Time', fontsize=18)
plt.xticks(fontsize=18)
plt.yticks(fontsize=18)
plt.show()

plt.figure(figsize=(8,6))
plt.scatter(ymean,ystd)
plt.xlabel('Mean Time', fontsize=18)
plt.ylabel('Std Time', fontsize=18)
plt.xticks(fontsize=18)
plt.yticks(fontsize=18)
plt.show()
```


![png](/img/output_36_0.png)



![png](/img/output_36_1.png)


## Now I want us to look at how "Age Grade" and "Age Category" affects times and runner count.

### Age Grade and Age Category are distinct and mean very different things. Age Category is imply what age bracket a runner falls into, e.g. 30-34 years old. Age Grade on the other hand is a measure of how good that runner is _relative_ to other runners of the same age and gender. See here for more detial https://support.parkrun.com/hc/en-us/articles/200565263-What-is-age-grading-

#### First we need to do some data cleaning. 

#### The age grade is very granular, in order to smooth the data and allow us to see trends I'm going to categorise the data by making boxes composed of a particular range of age grades. For example age grade 70 includes age grades from 70 up to 72. See the table below.

#### Any NaNs in the Age Grade column have also been dropped.


```python
df = data.dropna(subset=['Age_Grade'])
df['Rounded_Age_Grade'] = df['Age_Grade'].apply(lambda x: x//2)
df['Rounded_Age_Grade'] = df['Rounded_Age_Grade'].apply(lambda x: int(x*2))
df3.head()
```

    C:\ProgramData\Anaconda2\lib\site-packages\ipykernel_launcher.py:2: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      
    C:\ProgramData\Anaconda2\lib\site-packages\ipykernel_launcher.py:3: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      This is separate from the ipykernel package so we can avoid doing imports until
    




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Pos</th>
      <th>Name</th>
      <th>Time</th>
      <th>Age_Cat</th>
      <th>Age_Grade</th>
      <th>Gender</th>
      <th>Gen_Pos</th>
      <th>Club</th>
      <th>Note</th>
      <th>Total_Runs</th>
      <th>Run_No.</th>
      <th>Rounded_Age_Grade</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2012-11-10</td>
      <td>1</td>
      <td>Michael MCSWIGGAN</td>
      <td>18.316667</td>
      <td>35.0</td>
      <td>73.43</td>
      <td>M</td>
      <td>1.0</td>
      <td>Portmarnock Athletic Club</td>
      <td>First Timer!</td>
      <td>29.0</td>
      <td>1</td>
      <td>72</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2012-11-10</td>
      <td>2</td>
      <td>Alan FOLEY</td>
      <td>18.433333</td>
      <td>30.0</td>
      <td>71.16</td>
      <td>M</td>
      <td>2.0</td>
      <td>Raheny Shamrock AC</td>
      <td>First Timer!</td>
      <td>99.0</td>
      <td>1</td>
      <td>70</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2012-11-10</td>
      <td>3</td>
      <td>Matt SHIELDS</td>
      <td>18.533333</td>
      <td>55.0</td>
      <td>85.07</td>
      <td>M</td>
      <td>3.0</td>
      <td>North Belfast Harriers</td>
      <td>First Timer!</td>
      <td>274.0</td>
      <td>1</td>
      <td>84</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2012-11-10</td>
      <td>4</td>
      <td>David GARGAN</td>
      <td>18.650000</td>
      <td>40.0</td>
      <td>73.73</td>
      <td>M</td>
      <td>4.0</td>
      <td>Raheny Shamrock AC</td>
      <td>First Timer!</td>
      <td>107.0</td>
      <td>1</td>
      <td>72</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2012-11-10</td>
      <td>5</td>
      <td>Paul SINTON-HEWITT</td>
      <td>18.900000</td>
      <td>50.0</td>
      <td>79.28</td>
      <td>M</td>
      <td>5.0</td>
      <td>Ranelagh Harriers</td>
      <td>First Timer!</td>
      <td>369.0</td>
      <td>1</td>
      <td>78</td>
    </tr>
  </tbody>
</table>
</div>




```python

```


```python

```


```python
ax = df3.groupby('Rounded_Age_Grade').max()['Time'].plot.line(figsize=(8, 6), fontsize=20, marker='o')
ax = df3.groupby('Rounded_Age_Grade').mean()['Time'].plot.line(marker='o')
ax = df3.groupby('Rounded_Age_Grade').min()['Time'].plot.line(marker='o')


ax.set_xlabel("Age Grade", fontsize=20)
ax.set_ylabel("Finish Time (minute)", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
ax.legend(['Max','Mean','Min'], fontsize=18)

## it would be nice to have these two plost side by side
```




    <matplotlib.legend.Legend at 0x12f4a278>




![png](/img/output_42_1.png)


#### the mean and min finsih times have areally smooth treand with age grade, I'm sure the shape of this curve is due to whatever formula is used to calculate the age grade. But I'm not too sure. Obviously those with the best age grade have the lowest times. It should be noted here that I'm considering each race run by someone individually. It might be better to take all the runners with age grade e.g. 80 and then consider each runner, find the average minimum of all the athletes with age grade 80.

#### the maximum is far more jumpy. These data are necessarily outliers, as the maximum is just a single point with the largest value, all we need is for someone to have aterribleday. So we should expect some jumpiness. The reason for these jumps might be an athlete accompanying a friend who might be a newcomer or who knows maybe they were hungover or ahd a long hiatus.


```python

```


```python
ax = df3.groupby('Rounded_Age_Grade').std()['Time'].plot.line(marker = 'o', figsize=(8, 6), fontsize=20)
ax.set_xlabel("Age Grade", fontsize=20)
ax.set_ylabel("Std Time (minute)", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
```


![png](/img/output_45_0.png)


#### As people get better they run times which are more tightly bunched together. People at the very top have all come up against the limits of what's possible. It would be interesting to see the count versis age grade.


```python
ax = data.groupby('Age_Cat').max()['Time'].plot.line(marker='o', figsize=(8, 6), fontsize=20)
ax = data.groupby('Age_Cat').mean()['Time'].plot.line(marker='o', figsize=(8, 6), fontsize=20)
ax = data.groupby('Age_Cat').min()['Time'].plot.line(marker='o', figsize=(8, 6), fontsize=20)
ax.legend(['Max','Mean','Min'], fontsize=18)
ax.set_xlabel("Age Category", fontsize=20)
ax.set_ylabel("Finish Time (minute)", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')

## same again, have these two plots side by side
```


![png](/img/output_47_0.png)



```python
ax = data.groupby('Age_Cat').std()['Time'].plot.line(marker='o', figsize=(8, 6), fontsize=20)
ax.set_xlabel("Age Category", fontsize=20)
ax.set_ylabel("Std of Time (minute)", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
```


![png](/img/output_48_0.png)


### regardless of age the spread in abilties seems more or less constant. At older ages there isn't a lot of data, I can check the exact number of entries but the std figures aren't as reliable


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python
dft = data.groupby(['Date','Gender']).count()['Pos']
dft = dft.unstack()
dft['Ratio'] = dft['M']/dft['F']
dft['Date'] = dft.index
dft.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Gender</th>
      <th>F</th>
      <th>M</th>
      <th>Ratio</th>
      <th>Date</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2012-11-10</th>
      <td>81</td>
      <td>69</td>
      <td>0.851852</td>
      <td>2012-11-10</td>
    </tr>
    <tr>
      <th>2012-11-17</th>
      <td>85</td>
      <td>117</td>
      <td>1.376471</td>
      <td>2012-11-17</td>
    </tr>
    <tr>
      <th>2012-11-24</th>
      <td>122</td>
      <td>131</td>
      <td>1.073770</td>
      <td>2012-11-24</td>
    </tr>
    <tr>
      <th>2012-12-01</th>
      <td>90</td>
      <td>132</td>
      <td>1.466667</td>
      <td>2012-12-01</td>
    </tr>
    <tr>
      <th>2012-12-08</th>
      <td>57</td>
      <td>92</td>
      <td>1.614035</td>
      <td>2012-12-08</td>
    </tr>
  </tbody>
</table>
</div>




```python
ax = dft.plot(x='Date',y='Ratio', figsize=(15, 6), fontsize=20)
ax.set_xlabel("Date", fontsize=20)
ax.set_ylabel("Ratio", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
ax.legend(['Ratio'], fontsize=18)
```




    <matplotlib.legend.Legend at 0x117a40b8>




![png](/img/output_57_1.png)



```python
series = dft['Ratio']
result = seasonal_decompose(series,\
                            model='additive',freq=52)
fig = result.plot()
fig.set_size_inches(15,6)
```


![png](/img/output_58_0.png)



```python
## peak comes just before new year, cf count peak is just after NYE.
```


```python
# the seasonality is kind of expected when we consider the weather data.
# the ratio of men to women goes up the colder and rainier it is
```


```python
dft = data.groupby(['Age_Cat','Gender']).count()['Pos']
dft = dft.unstack()
dft['Ratio'] = dft['M']/dft['F']
dft['Age_Cat'] = dft.index
dft.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Gender</th>
      <th>F</th>
      <th>M</th>
      <th>Ratio</th>
      <th>Age_Cat</th>
    </tr>
    <tr>
      <th>Age_Cat</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>10.0</th>
      <td>468.0</td>
      <td>614.0</td>
      <td>1.311966</td>
      <td>10.0</td>
    </tr>
    <tr>
      <th>11.0</th>
      <td>1069.0</td>
      <td>1599.0</td>
      <td>1.495790</td>
      <td>11.0</td>
    </tr>
    <tr>
      <th>15.0</th>
      <td>900.0</td>
      <td>996.0</td>
      <td>1.106667</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>18.0</th>
      <td>352.0</td>
      <td>229.0</td>
      <td>0.650568</td>
      <td>18.0</td>
    </tr>
    <tr>
      <th>20.0</th>
      <td>1006.0</td>
      <td>635.0</td>
      <td>0.631213</td>
      <td>20.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
ax = dft.plot(x='Age_Cat',y='Ratio', figsize=(8, 6), fontsize=20, marker='o')

ax.set_xlabel("Age_Cat", fontsize=20)
ax.set_ylabel("Ratio", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
ax.legend(['Ratio'], fontsize=18)

## again plots side by side
```




    <matplotlib.legend.Legend at 0x1084c3c8>




![png](/img/output_62_1.png)



```python
ax = dft.plot(x='Age_Cat',y='Ratio', figsize=(8, 6), fontsize=20, marker='o')

ax.set_xlabel("Age_Cat", fontsize=20)
ax.set_ylabel("Ratio", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
ax.legend(['Ratio'], fontsize=18)
ax.set_xlim(xmax=65)
ax.set_ylim(ymax=5)
```




    (-0.32722664015904579, 5)




![png](/img/output_63_1.png)



```python
# that dip coincides with the dip/bump in mean times of male and females
```


```python
dft = df3.groupby(['Rounded_Age_Grade','Gender']).count()['Pos']
dft = dft.unstack()
dft['Ratio'] = dft['M']/dft['F']
dft['Rounded_Age_Grade'] = dft.index
dft.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Gender</th>
      <th>F</th>
      <th>M</th>
      <th>Ratio</th>
      <th>Rounded_Age_Grade</th>
    </tr>
    <tr>
      <th>Rounded_Age_Grade</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>22</th>
      <td>4.0</td>
      <td>1.0</td>
      <td>0.250000</td>
      <td>22</td>
    </tr>
    <tr>
      <th>24</th>
      <td>17.0</td>
      <td>16.0</td>
      <td>0.941176</td>
      <td>24</td>
    </tr>
    <tr>
      <th>26</th>
      <td>15.0</td>
      <td>23.0</td>
      <td>1.533333</td>
      <td>26</td>
    </tr>
    <tr>
      <th>28</th>
      <td>57.0</td>
      <td>29.0</td>
      <td>0.508772</td>
      <td>28</td>
    </tr>
    <tr>
      <th>30</th>
      <td>125.0</td>
      <td>48.0</td>
      <td>0.384000</td>
      <td>30</td>
    </tr>
  </tbody>
</table>
</div>




```python
ax = dft.plot(x='Rounded_Age_Grade',y='Ratio', figsize=(8, 6), fontsize=20, marker='o')

ax.set_xlabel("Rounded_Age_Grade", fontsize=20)
ax.set_ylabel("Ratio", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
ax.legend(['Ratio'], fontsize=18)
```




    <matplotlib.legend.Legend at 0xcbe4d68>



    C:\ProgramData\Anaconda2\lib\site-packages\matplotlib\transforms.py:661: RuntimeWarning: invalid value encountered in absolute
      inside = ((abs(dx0 + dx1) + abs(dy0 + dy1)) == 0)
    


![png](/img/output_66_2.png)



```python
## so the men who come along tend to be good and hardcore runners
# this jibes well with the weather data and other data findings
# like the runner count and the seasonality
```


```python
dft = data.groupby(['Date','Gender']).count()['Pos']
dft = dft.unstack()
dft['Ratio'] = dft['M']/dft['F']
dft['Count'] = dft['M']+dft['F']
dft.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Gender</th>
      <th>F</th>
      <th>M</th>
      <th>Ratio</th>
      <th>Count</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2012-11-10</th>
      <td>81</td>
      <td>69</td>
      <td>0.851852</td>
      <td>150</td>
    </tr>
    <tr>
      <th>2012-11-17</th>
      <td>85</td>
      <td>117</td>
      <td>1.376471</td>
      <td>202</td>
    </tr>
    <tr>
      <th>2012-11-24</th>
      <td>122</td>
      <td>131</td>
      <td>1.073770</td>
      <td>253</td>
    </tr>
    <tr>
      <th>2012-12-01</th>
      <td>90</td>
      <td>132</td>
      <td>1.466667</td>
      <td>222</td>
    </tr>
    <tr>
      <th>2012-12-08</th>
      <td>57</td>
      <td>92</td>
      <td>1.614035</td>
      <td>149</td>
    </tr>
  </tbody>
</table>
</div>




```python
ax = dft.plot.scatter(x='Count',y='Ratio', figsize=(8, 6), fontsize=20,)
ax.set_xlabel("Count", fontsize=20)
ax.set_ylabel("Ratio M/F", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
ax.grid('on', which='minor', axis='x')
ax.grid('on', which='minor', axis='y')
ax.set_xscale('log')

## maybe throw in ratio vs mean time
```


![png](/img/output_69_0.png)



```python
## and the fact that when few people come out the ratio of men to women goes up.
## and we already know the count is seasonal so since count and the ratio are 
## correlated we expect some seasonality in the ratio.
```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```
