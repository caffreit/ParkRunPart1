
## Analysis of Park Run Data 1
### Since the beginning of this year my girlfriend has dragged me out to the local [Park Run](http://www.parkrun.ie/malahide/). To say I am a reluctant participant is an understatement. However, she also told me that all past runs are recorded and are publically available.<br> The records go back several years and with roughly 80 thousand entries I thought it would make for an interesting dataset and a great opportunity to brush up my data science skills.<br> 
This is the first in a series of posts. At the moment I have another couple in the works. This post will begin by looking at how the "Finish Times" and "Runner Count" vary over time. It then moves on to finding a relationship (if any) of the "Finish Times" with "Age" and "Runner Count". My hope is that I can tell a fun story and that, possibly, by the end we arrive at an interesting conclusion. 


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

We need to convert some numerical values from strings to floats so we can work with them.<br>Then print out the first few entries to check it worked as expected.


```python
data['Time'] = ((pd.to_numeric(data['Time'].str.slice(0,2)))*60)+\
                (pd.to_numeric(data['Time'].str.slice(3,5)))+((pd.to_numeric(data['Time'].str.slice(6,8)))/60)
data['Date'] = pd.to_datetime(data['Date'],errors='coerce', format='%d-%m-%Y')
data['Age_Cat'] = pd.to_numeric(data['Age_Cat'].str.slice(2,4),errors='coerce', downcast='signed')
data['Age_Grade'] = pd.to_numeric(data['Age_Grade'].str.slice(0,5),errors='coerce')
data.head()
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
  </tbody>
</table>
</div>



The most obvious thing to try is how does the number of runners "Runner Count" change over time. Is there a trend of increasing attendence? Is there any seasonality, where the count drops in the winter?


```python
ax = data.groupby('Date').count()['Pos'].plot.line(figsize=(15, 6), fontsize=20)
ax.set_xlabel("Date", fontsize=20)
ax.set_ylabel("Runner Count", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
```


![png](output_10_0.png)


Even by eye we can see some periodicity in the Runner Count. There also seems to be a drop in average attendence from roughly 350 runners to somewhere between 2 and 3 hundred runners after 2015.<br>We can use a tool from statsmodel to decompose the trend and the seasonality from the data. Below are the outputted plots.

#### Seasonality and Trend of Runner Count


```python
result = seasonal_decompose(data.groupby('Date').count()['Pos'],\
                            model='additive',freq=52)
fig = result.plot()
fig.set_size_inches(15,6)
```


![png](output_13_0.png)


As we thought there is a definite periodicity in attendence. Peaking just after the beginning of a new year, no doubt reflecting all the New Year Resolutions made. This is followed by a steady decline over the course of the year reaching a minimum around Christmas.

We can also observe more clearly the drop in average attendence. I've no insight into what may have caused the drop across 2015. If anybody has any thoughts feel free to get in contact. ivan.caffrey@gmail.com

There doesn't seem to be anything interesting left over in the residuals.

## Let's move onto "Finish Times".

The first thing I'm going to do is, for each race, find the Minimum (first place), Maximum (last place) and Mean Finish times. I'll then plot these times for each date.


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




    <matplotlib.legend.Legend at 0x10614358>




![png](output_17_1.png)


There doesn't seem to be much going on in any of the plots. But let's get a closer look at the Mean and Minimum times.


```python
ax = data.groupby('Date').mean()['Time'].plot.line(figsize=(15, 6), fontsize=20)
ax = data.groupby('Date').min()['Time'].plot.line()
ax.set_xlabel("Date", fontsize=20)
ax.set_ylabel("Finish Time (minute)", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
ax.legend(['Mean','Min'], fontsize=18)
```




    <matplotlib.legend.Legend at 0x10608748>




![png](output_19_1.png)


There is a distinct drop in the first place finishing times in mid 2014. I looked at the raw data to see why this was. It turns out there was a small group of runners who all ran the course in about 15 minutes. They appear suddenly, dominate the leaderboard and then leave never to be seen again. This drop shows up in the decomposition of trend and seasonality as well.

#### Seasonality and Trend of Minimum times


```python
result = seasonal_decompose(data.groupby('Date').min()['Time'],\
                            model='additive',freq=52)
fig = result.plot()
fig.set_size_inches(15,6)
```


![png](output_22_0.png)


Repeating the decomposition on the mean times, we see a seasonality. Just like the runner count, it peaks just after New Years Day and gradually decreases. Again, this relationship is not that surprising. If all the new "athletes" are only running because of a  New Years resolution and do not ordinarily spend their free time pounding the pavements it stands to reason the mean finish time would increase.

#### Seasonality and Trend of Mean times


```python
result = seasonal_decompose(data.groupby('Date').mean()['Time'],\
                            model='additive',freq=52)
fig = result.plot()
fig.set_size_inches(15,6)
```


![png](output_25_0.png)


Given that the mean finish time and the runner count both display seasonality, we should see a  positive dependence of mean time with runner count. So let's plot the Max, Mean and Min times against Runner Count.

### Dependence of Finish times on Runner Count


```python
x = data.groupby('Date').count()['Pos']
ymax = data.groupby('Date').max()['Time']
ymean = data.groupby('Date').mean()['Time']
ymin = data.groupby('Date').min()['Time']
```


```python
plt.figure(figsize=(12,6))
plt.subplot(121)
plt.scatter(x,ymax, label='Max')
plt.scatter(x,ymean, label='Mean')
plt.scatter(x,ymin, label='Min')
plt.xlabel('Runner Count', fontsize=18)
plt.ylabel('Finish Time', fontsize=18)
plt.xticks(fontsize=14)
plt.yticks(fontsize=14)
plt.legend(fontsize=16)

plt.subplot(122)
plt.scatter(x,ymean, label='Mean')
plt.scatter(x,ymin, label='Min')
plt.xlabel('Runner Count', fontsize=18)
plt.xticks(fontsize=14)
plt.yticks(fontsize=14)
plt.legend(fontsize=16)
plt.tight_layout()
```


![png](output_29_0.png)


Zooming in on the mean and minimum times we observe a clear increase in mean finish time with runner count. Interestingly, we also see a decrease in the min finish time. While we would expect an increase in runner count to "drag up" the mean time. 

This effect would not apply to the first place times since the slow fairweather runners won't affect the times of the hardcore roadrunners. In fact we would expect that as attendence increases there would be more hardcore runners who run the course in ever faster times.

#### Standard deviation of times

For fun I said I'd plot the standard deviation of the finish times. When we decompose the time series we see the same seasonality as in Runner Count and mean finish time.


```python
ax = data.groupby('Date').std()['Time'].plot.line(figsize=(15, 6), fontsize=20)
ax.set_xlabel("Date", fontsize=20)
ax.set_ylabel("Std Time (minute)", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
```


![png](output_33_0.png)


#### Seasonality and Trend of Standard Devaition of Finish times


```python
result = seasonal_decompose(data.groupby('Date').std()['Time'],\
                            model='additive',freq=52)
fig = result.plot()
fig.set_size_inches(15,6)
```


![png](output_35_0.png)


We see the positive dependence of the standard deviation of the finish times and the mean finish times and the runner count. As a larger number of slower people attend, naturally we should expect an increase in the standard deviation.

#### Standard Deviation of Finish times versus Runner Count and Mean time


```python
ystd = data.groupby('Date').std()['Time']
plt.figure(figsize=(10,5))
plt.subplot(121)
plt.scatter(x,ystd)
plt.xlabel('Runner Count', fontsize=18)
plt.ylabel('Std Time', fontsize=18)
plt.xticks(fontsize=16)
plt.yticks(fontsize=16)


plt.subplot(122)
plt.scatter(ymean,ystd)
plt.xlabel('Mean Time', fontsize=18)
plt.xticks(fontsize=16)
plt.yticks(fontsize=16)
plt.tight_layout()
```


![png](output_38_0.png)


## Now I want us to look at how "Age Grade" and "Age Category" affects times and runner count.

Age Grade and Age Category are distinct and mean very different things. Age Category is imply what age bracket a runner falls into, e.g. 30-34 years old. Age Grade on the other hand is a measure of how good that runner is *relative* to other runners of the same age and gender. See here for more detail. [What is Age Grade?](https://support.parkrun.com/hc/en-us/articles/200565263-What-is-age-grading-)

First we need to do some data cleaning. 

The age grade is very granular, in order to smooth the data and allow us to see trends I'm going to categorise the data by making boxes composed of a particular range of age grades. For example age grade 70 includes age grades from 70 up to 72. See the table below.

Any NaNs in the Age Grade column have also been dropped.


```python
df = data.dropna(subset=['Age_Grade'])
df['Rounded_Age_Grade'] = df['Age_Grade'].apply(lambda x: x//2)
df['Rounded_Age_Grade'] = df['Rounded_Age_Grade'].apply(lambda x: int(x*2))
df.head()
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
x = range(22,96,2)
ymax = df.groupby('Rounded_Age_Grade').max()['Time']
ymean = df.groupby('Rounded_Age_Grade').mean()['Time']
ymin = df.groupby('Rounded_Age_Grade').min()['Time']
ystd = df.groupby('Rounded_Age_Grade').std()['Time']
```


```python
plt.figure(figsize=(12,6))
plt.subplot(121)
plt.plot(x,ymax, label='Max', marker = 'o')
plt.plot(x,ymean, label='Mean', marker = 'o')
plt.plot(x,ymin, label='Min', marker = 'o')
plt.xlabel('Rounded Age Grade', fontsize=18)
plt.ylabel('Finish Time', fontsize=18)
plt.xticks(fontsize=14)
plt.yticks(fontsize=14)
plt.legend(fontsize=16)
plt.grid()

plt.subplot(122)
plt.plot(x,ystd, label='Std time', marker = 'o')
plt.xlabel('Rounded Age Grade', fontsize=18)
plt.xticks(fontsize=14)
plt.yticks(fontsize=14)
plt.legend(fontsize=16)
plt.tight_layout()
plt.grid()
```


![png](output_43_0.png)


The mean and minimum finish times have a really smooth trend with Age Grade, I'm sure the shape of this curve is due to whatever formula is used to calculate the age grade. But I'm not too sure. Obviously those with the best age grade have the lowest times. It should be noted here that I'm considering each race run by someone individually. It might be better to take all the runners with age grade e.g. 80 and then consider each runner, find the average minimum of all the athletes with age grade 80.

The maximum is far more jumpy. These data are necessarily outliers, as the maximum is just a single point with the largest value. all we need is for someone to have a terrible day. So we should expect some jumpiness. The reason for these jumps might be an athlete accompanying a friend who might be a newcomer or who knows maybe they were hungover or ahd a long hiatus.

As people get better they run times which are more tightly bunched together. People at the very top have all come up against the limits of what's possible. It would be interesting to see the count versis age grade.


```python
x = [10,11,15,18,20,25,30,35,40,45,50,55,60,65,70,75,80,85]
ymax = df.groupby('Age_Cat').max()['Time']
ymean = df.groupby('Age_Cat').mean()['Time']
ymin = df.groupby('Age_Cat').min()['Time']
ystd = df.groupby('Age_Cat').std()['Time']
```


```python
plt.figure(figsize=(12,6))
plt.subplot(121)
plt.plot(x,ymax, label='Max', marker = 'o')
plt.plot(x,ymean, label='Mean', marker = 'o')
plt.plot(x,ymin, label='Min', marker = 'o')
plt.xlabel('Age Category', fontsize=18)
plt.ylabel('Finish Time', fontsize=18)
plt.xticks(fontsize=14)
plt.yticks(fontsize=14)
plt.legend(fontsize=16)
plt.grid()

plt.subplot(122)
plt.plot(x,ystd, label='Std time', marker = 'o')
plt.xlabel('Age Category', fontsize=18)
plt.xticks(fontsize=14)
plt.yticks(fontsize=14)
plt.legend(fontsize=16)
plt.tight_layout()
plt.grid()
```


![png](output_46_0.png)


The maximum time stays roughly constant with age. Walking pace is about 5 km an hour and most people can walk for an hour. So I would say that all the max times we see here are just from people walking around.

The mean time stays fairly flat up to about 70 years of age where it jumps up. The minimum time steadily rises from about 20 years of age with an exponential shape. This line probably represents the limit of ability at a given age. This then shows that the limit of ability decreases with age. This is an interesting contrast to the mean time. Which shows that average people regardless of age have the same ability/fitness.

The standard deviation is plotted to the left. At the old end of the scale the number of athletes is quite small and the standard deviation probably isn't as reliable. It also may not be surprising that the standard deviation drops, because the type of 80 year old people who take part are probably quite similar in mobility.

Regardless of age the spread in abilties seems more or less constant. At older ages there isn't a lot of data, I can check the exact number of entries but the std figures aren't as reliable


```python
x = [10,11,15,18,20,25,30,35,40,45,50,55,60,65,70,75,80,85]
y = data.groupby('Age_Cat').count()['Pos']
y
```




    Age_Cat
    10.0     1082
    11.0     2668
    15.0     1896
    18.0      581
    20.0     1641
    25.0     4934
    30.0     9796
    35.0    12928
    40.0    11312
    45.0    10186
    50.0     8933
    55.0     4056
    60.0     2243
    65.0      750
    70.0      208
    75.0       60
    80.0       40
    85.0        1
    Name: Pos, dtype: int64



### Runner Count for each Age Category

Below are two plots showing the distribution of number athletes per age category. The mean is clearly somewhere around 35 years of age. Funnily, young children are clearly dragged along by eaer parents, but once the children reach their teenage years they're not so easily roused from bed at half 8 on a Saturday morning.


```python
plt.figure(figsize=(12,6))

plt.bar(x,y, width=2, color=(0.2588,0.4433,1.0))
plt.xlabel('Age Category', fontsize=18)
plt.ylabel('Runner Count', fontsize=18)
plt.xticks(fontsize=14)
plt.yticks(fontsize=14)
plt.grid(True)
```


![png](output_51_0.png)



```python
sns.factorplot("Age_Cat", data=data,kind='count', aspect=2)
```




    <seaborn.axisgrid.FacetGrid at 0xc9edcc0>




![png](output_52_1.png)


I've added this final plot mostly because it looks cool. It tells the same story as the previous plots. Now, we can also see how the distribution of finish times change between age categories.

And if anyone knows how to fix it, please let me know.


```python
sns.set(style="white", rc={"axes.facecolor": (0, 0, 0, 0)})

# Initialize the FacetGrid object
pal = sns.cubehelix_palette(18, rot=-.25, light=.7)
g = sns.FacetGrid(data, row="Age_Cat", hue="Age_Cat", aspect=15, size=.5, palette=pal)

# Draw the densities in a few steps
g.map(sns.kdeplot, "Time", clip_on=False, shade=True, alpha=1, lw=1.5, bw=.7)
g.map(sns.kdeplot, "Time", clip_on=False, color="w", lw=2, bw=.7)
g.map(plt.axhline, y=0, lw=2, clip_on=False)

# Define and use a simple function to label the plot in axes coordinates
def label(x, color, label):
    ax = plt.gca()
    ax.text(0, .2, label, fontweight="bold", color=color, 
            ha="left", va="center", transform=ax.transAxes)

g.map(label, "Time")

# Set the subplots to overlap
g.fig.subplots_adjust(hspace=-.1)

# Remove axes details that don't play will with overlap
g.set_titles("")
g.set(yticks=[])
g.despine(bottom=True, left=True)
```




    <seaborn.axisgrid.FacetGrid at 0xfa650b8>




![png](output_54_1.png)


That's all for now, be sure to check out the next post, **_"Battle of the Sexes"_**, where I compare the performance and attendence of both genders.
