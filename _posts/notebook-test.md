---
layout:     notebook
title:      Titanic Notebook test
author:     Palavi Sahoopa
tags:       jupyter workflows template
subtitle:   Showcasing Jupyter Notebook Translator Layout
category:   project1
notebookfilename: intro
visualworkflow: true
---
```python
import numpy as np
import pandas as pd 
import matplotlib.pyplot as plt
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split, cross_val_score,KFold
from sklearn import metrics
import brewer2mpl
from sklearn import tree
from sklearn.neighbors import KNeighborsClassifier
from sqlalchemy import create_engine

%matplotlib inline
```

Every data scientist begins their journey of exploring the world of data science using Titanic data and I am no exception. I am learning data science using Python and it has been a great experience working on titanic data for a few reasons..
1. Even though the data sample is as small as 891 observations and 11 features, this data has the beauty to experiment various techniques of data science very easily. It has a good mix of categorical variables and numeric variables. The categories in each variable is just 3-4, which is a good opportunity for a novice data scientist to experiement. 
The data has a few missing values and a few outliers too  which is great for learning.

### Objective 
This is a classification problem where the goal is to identify the likelihood of passenger being survived. 


### 2. Technique
Since it's a classificatin problem, I will be using Logistic Regression and Decision trees

### 3. Model evaluation criteria
AUC, accuracy

## Part 1: Aquire the Data


```python
psql -h dsi.c20gkj5cvu3l.us-east-1.rds.amazonaws.com -p 5432 -U dsi_student titanic
password: gastudents
```


      File "<ipython-input-2-b0485d6fae25>", line 1
        psql -h dsi.c20gkj5cvu3l.us-east-1.rds.amazonaws.com -p 5432 -U dsi_student titanic
                  ^
    SyntaxError: invalid syntax




```python
engine = create_engine('postgresql://postgres:postgres@localhost:5432')
pd.read_sql("select * from information_schema.tables limit 5;" , engine)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>table_catalog</th>
      <th>table_schema</th>
      <th>table_name</th>
      <th>table_type</th>
      <th>self_referencing_column_name</th>
      <th>reference_generation</th>
      <th>user_defined_type_catalog</th>
      <th>user_defined_type_schema</th>
      <th>user_defined_type_name</th>
      <th>is_insertable_into</th>
      <th>is_typed</th>
      <th>commit_action</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>postgres</td>
      <td>pg_catalog</td>
      <td>pg_statistic</td>
      <td>BASE TABLE</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>YES</td>
      <td>NO</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>postgres</td>
      <td>pg_catalog</td>
      <td>pg_type</td>
      <td>BASE TABLE</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>YES</td>
      <td>NO</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>postgres</td>
      <td>public</td>
      <td>table1</td>
      <td>BASE TABLE</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>YES</td>
      <td>NO</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>postgres</td>
      <td>pg_catalog</td>
      <td>pg_authid</td>
      <td>BASE TABLE</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>YES</td>
      <td>NO</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>postgres</td>
      <td>public</td>
      <td>evictions_simple</td>
      <td>BASE TABLE</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>YES</td>
      <td>NO</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
%load_ext sql
```

    The sql extension is already loaded. To reload it, use:
      %reload_ext sql



```python
%%sql postgresql://postgres:postgres@localhost:5432
```

    UsageError: %%sql is a cell magic, but the cell body is empty. Did you mean the line magic %sql (single %)?


```python
# sqlalchemy
conn = engine.connect()
conn.execute("commit")
conn.execute("create database titanic")
conn.close()

```


```python
%%sql create database titanic;
```

    UsageError: %%sql is a cell magic, but the cell body is empty. Did you mean the line magic %sql (single %)?


```python
%%sql
DROP TABLE IF EXISTS titanic;
CREATE TABLE titanic
("Passenger ID" integer,
 "Survived" varchar,
 "Pclass" varchar,
 "Name" varchar,
 "Sex" varchar,
 "Age" varchar,
 "SibSp" varchar,
 "Parch" varchar,
 "Ticket" varchar,
 "Fare" varchar,
 "Cabin" varchar,
 "Embarked" varchar);


COPY evictions_simple FROM 'postgresql://dsi_student:gastudents@dsi.c20gkj5cvu3l.us-east-1.rds.amazonaws.com/titanic' delimiter ',' csv header;
```

    Done.
    Done.
    (psycopg2.OperationalError) could not open file "postgresql://dsi_student:gastudents@dsi.c20gkj5cvu3l.us-east-1.rds.amazonaws.com/titanic" for reading: No such file or directory
     [SQL: "COPY evictions_simple FROM 'postgresql://dsi_student:gastudents@dsi.c20gkj5cvu3l.us-east-1.rds.amazonaws.com/titanic' delimiter ',' csv header;"]


#### 1. Connect to the remote database


```python
from sqlalchemy import create_engine
engine = create_engine('postgresql://dsi_student:gastudents@dsi.c20gkj5cvu3l.us-east-1.rds.amazonaws.com/titanic')

```

#### 2. Query the database and aggregate the data


```python
# We will pull in the Titanic data as before
titanic = pd.read_sql('SELECT * FROM train', engine)
titanic.shape
```




    (891, 13)



### Data Description - 

Passenger ID : A seemingly unique number assigned to each passenger

Survived : A binary indicator of survival (0 = died, 1 = survived)

Pclass : A proxy for socio-economic status (1 = upper, 3 = lower)

Name : Passenger’s Name. For wedded women, her husband’s name appears first and her maiden name appears in parentheses

Sex : General indication of passenger’s sex

Age : Age of passenger (or approximate age). Passengers under the age of 1 year have fractional ages

SibSp : A count of the passenger’s siblings or spouses aboard

Parch : A count of the passenger’s parents or siblings aboard

Ticket : The number printed on the ticket. The numbering system is not immediately apparent

Fare : The price for the ticket (presumably in pounds, shillings, and pennies)

Cabin : Cabin number occupied by the passenger (this field is quite empty)

Embarked : The port from which the passenger boarded the ship



## Data Exploration :


```python
titanic.describe()
```

    /Users/ipm/anaconda/lib/python2.7/site-packages/numpy/lib/function_base.py:3834: RuntimeWarning: Invalid value encountered in percentile
      RuntimeWarning)





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Pclass</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>Fare</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>891.000000</td>
      <td>891.000000</td>
      <td>891.000000</td>
      <td>891.000000</td>
      <td>714.000000</td>
      <td>891.000000</td>
      <td>891.000000</td>
      <td>891.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>445.000000</td>
      <td>446.000000</td>
      <td>0.383838</td>
      <td>2.308642</td>
      <td>29.699118</td>
      <td>0.523008</td>
      <td>0.381594</td>
      <td>32.204208</td>
    </tr>
    <tr>
      <th>std</th>
      <td>257.353842</td>
      <td>257.353842</td>
      <td>0.486592</td>
      <td>0.836071</td>
      <td>14.526497</td>
      <td>1.102743</td>
      <td>0.806057</td>
      <td>49.693429</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.420000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>222.500000</td>
      <td>223.500000</td>
      <td>0.000000</td>
      <td>2.000000</td>
      <td>NaN</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>7.910400</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>445.000000</td>
      <td>446.000000</td>
      <td>0.000000</td>
      <td>3.000000</td>
      <td>NaN</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>14.454200</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>667.500000</td>
      <td>668.500000</td>
      <td>1.000000</td>
      <td>3.000000</td>
      <td>NaN</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>31.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>890.000000</td>
      <td>891.000000</td>
      <td>1.000000</td>
      <td>3.000000</td>
      <td>80.000000</td>
      <td>8.000000</td>
      <td>6.000000</td>
      <td>512.329200</td>
    </tr>
  </tbody>
</table>
</div>




```python
titanic.dtypes
```




    index            int64
    PassengerId      int64
    Survived         int64
    Pclass           int64
    Name            object
    Sex             object
    Age            float64
    SibSp            int64
    Parch            int64
    Ticket          object
    Fare           float64
    Cabin           object
    Embarked        object
    dtype: object




```python
# Finding missing values
for col in titanic.columns:
    print col , titanic[col].isnull().sum()
```

    index 0
    PassengerId 0
    Survived 0
    Pclass 0
    Name 0
    Sex 0
    Age 177
    SibSp 0
    Parch 0
    Ticket 0
    Fare 0
    Cabin 687
    Embarked 2


1. 20% of the age data has missing values
2. 75% of cabins have missing values
3. Only 2 customers have missing values in Embarked

Hypothesis I would like to test:
1. Is Sex important in predicting the survival? My hypothesis is if the passenger is female, then passenger will have higher survival rate
2. Is age important in predicting the survival? My hypothesis is that if the passenger is child/baby, then passenger will have higher survival rate
3. Is nobility an important variable in predicting the survival? If passenger is a common person from class 3, can have less chance of survival, compared to the passenger from class 1/passenger having noble title
4. Do family members have higher chances of survival than people who were travelling alone?
5. Passengers who paid more might have higher chance of survival, as it also reflects that they were rich. So would be intersting to test, if the passengers who paid more had higher chances of survival or not
6. Is there any signifant difference in the survival rates based on their embarked port?

## Part 2: Exploratory Data Analysis


```python
# Let's have a look at the numeric/categorical variables distribution

# Set up a grid of plots
fig = plt.figure(figsize=(17,8))
fig_dims = (2, 3)

# Plot death and survival counts
plt.subplot2grid(fig_dims, (0, 0))
titanic['Survived'].value_counts().plot(kind='bar', 
                                         title='Survived Vs Died Counts')

# Plot Pclass counts
plt.subplot2grid(fig_dims, (0, 1))
titanic['Pclass'].value_counts().plot(kind='bar', 
                                       title='Passenger Class Counts')

# Plot Sex counts
plt.subplot2grid(fig_dims, (0, 2))
titanic['Sex'].value_counts().plot(kind='bar', 
                                    title='Gender Counts')

# Plot Embarked counts
plt.subplot2grid(fig_dims, (1, 0))
titanic['Embarked'].value_counts().plot(kind='bar', 
                                         title='Port of Embarkation Counts')

# Plot the Age histogram
plt.subplot2grid(fig_dims, (1, 1))
titanic['Age'].hist(bins=10)
plt.title('Age Distribution')

# Plot the Fare histogram
plt.subplot2grid(fig_dims, (1, 2))
titanic['Fare'].hist(bins=10)
plt.title('Fare Distribution')
```




    <matplotlib.text.Text at 0x11db55950>




![png](output_24_1.png)
<img src = "/Assets/output_24_1.png">

1. Let's test first hypothesis - 
### IS sex important in predicting the survival


```python
group = titanic.groupby(["Sex"])["Survived"].value_counts(normalize=True).unstack()
group
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Survived</th>
      <th>0</th>
      <th>1</th>
    </tr>
    <tr>
      <th>Sex</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>female</th>
      <td>0.257962</td>
      <td>0.742038</td>
    </tr>
    <tr>
      <th>male</th>
      <td>0.811092</td>
      <td>0.188908</td>
    </tr>
  </tbody>
</table>
</div>



Almost 75% of the female passengers survived and only 19% male passengers survived. These is a significant difference between the survival rates of male and female. So it looks like our hypothesis is true


```python
#normalize each row by transposing, normalizing each column, and un-transposing
group = (1. * group.T / group.T.sum()).T

print "Transpose" , group.T

plt.bar([0, 1], group[0], color = 'b', label='Died')
plt.bar([0, 1], group[1], bottom=group[0], color= 'g', label='Survived')
plt.xticks([0.5, 1.5], ['female', 'male'], rotation='horizontal')
plt.ylabel("Fraction")
plt.title("Survival Rates by Gender")
plt.legend(loc='right')            
```

    Transpose Sex         female      male
    Survived                    
    0         0.257962  0.811092
    1         0.742038  0.188908





    <matplotlib.legend.Legend at 0x11e7da710>




![png](output_28_2.png)



```python
group.plot.bar(stacked=True);
plt.title("Seuvival rates by gender")
plt.legend(loc="upper right")
```




    <matplotlib.legend.Legend at 0x11b31aad0>




![png](output_29_1.png)


Sex looks to be a very important factor in survival rates... 75% of all females survived while only 19% male survived

### Is age important in predicting the survival?


```python
titanic.Age.isnull().sum()
```




    177



We have 20% missing values in Age. We have to impute the missing values for age. 
Let's see how can we impute the missing value for age... 
We can impute the age based on the title.. If the title is Mrs. then we can impute the age by taking the median for all the passengers having title as Mrs and so on...


```python
def extract_Salutation(string):
    return string.split(',')[1].split('.')[0].strip()
titanic["Salutation"] = titanic["Name"].apply(extract_Salutation)
titanic["Salutation"].value_counts()
```




    Mr              517
    Miss            182
    Mrs             125
    Master           40
    Dr                7
    Rev               6
    Mlle              2
    Col               2
    Major             2
    Lady              1
    Jonkheer          1
    Don               1
    Ms                1
    Mme               1
    Capt              1
    the Countess      1
    Sir               1
    Name: Salutation, dtype: int64



Oh.. we have so many titles... But 13 of these 17 titles are very rare.. Only 1 or 2 passengers have these titles. 
Also these title reflects their nobility... Let's keep the titles as Mr, Mrs, Miss, Master. 

1. I am going to combine Ms with Miss, 
2. Mme, Mlle, Lady, the countess with Mrs  
3. Dr with Mr


```python
def GroupSalutations(Salutation):
    if Salutation == "Mr":
        return 'Mr'
    elif Salutation in(["Mrs", "Mlle", "Lady", "Mme", "the Countess"]):
        return 'Mrs'
    elif Salutation in(["Miss", "Ms"]):
        return "Miss"
    elif Salutation == "Master":
        return "Master"
#    elif Salutation == 'Dr':
#        if Sex == 'male':
#            return "Mr"
#        else:
#            return "Mrs"
    else:
        return "Mr"
titanic["Salut_group"] = titanic["Salutation"].apply(GroupSalutations)
titanic["Salut_group"].value_counts()
```




    Mr        538
    Miss      183
    Mrs       130
    Master     40
    Name: Salut_group, dtype: int64




```python
titanic.loc[titanic.Age.isnull()].head(2)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Pclass</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>Ticket</th>
      <th>Fare</th>
      <th>Cabin</th>
      <th>Embarked</th>
      <th>Salutation</th>
      <th>Salut_group</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>5</td>
      <td>6</td>
      <td>0</td>
      <td>3</td>
      <td>Moran, Mr. James</td>
      <td>male</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>330877</td>
      <td>8.4583</td>
      <td>None</td>
      <td>Q</td>
      <td>Mr</td>
      <td>Mr</td>
    </tr>
    <tr>
      <th>17</th>
      <td>17</td>
      <td>18</td>
      <td>1</td>
      <td>2</td>
      <td>Williams, Mr. Charles Eugene</td>
      <td>male</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>244373</td>
      <td>13.0000</td>
      <td>None</td>
      <td>S</td>
      <td>Mr</td>
      <td>Mr</td>
    </tr>
  </tbody>
</table>
</div>



### Lets find the median Age for each salutation groups


```python
a = titanic.groupby(["Salut_group"])["Age"].median()
a
```




    Salut_group
    Master     3.5
    Miss      21.0
    Mr        30.0
    Mrs       35.0
    Name: Age, dtype: float64




```python
table = titanic.pivot_table(values='Age', index=['Salut_group'], columns=['Pclass', 'Sex'], aggfunc=np.median).unstack()
#print titanic[titanic['Age'].isnull()]
print table
# Define function to return value of this pivot_table
def impute_age(x):
    return table[x['Pclass']][x['Sex']][x['Salut_group']]
# Replace missing values
titanic['Age'].fillna(titanic[titanic['Age'].isnull()].apply(impute_age, axis=1), inplace=True)
```

    Pclass  Sex     Salut_group
    1       female  Master          NaN
                    Miss           30.0
                    Mr             49.0
                    Mrs            39.0
            male    Master          4.0
                    Miss            NaN
                    Mr             42.0
                    Mrs             NaN
    2       female  Master          NaN
                    Miss           24.0
                    Mr              NaN
                    Mrs            32.0
            male    Master          1.0
                    Miss            NaN
                    Mr             31.0
                    Mrs             NaN
    3       female  Master          NaN
                    Miss           18.0
                    Mr              NaN
                    Mrs            31.0
            male    Master          4.0
                    Miss            NaN
                    Mr             26.0
                    Mrs             NaN
    dtype: float64



```python
ind =titanic.loc[titanic.Salut_group == 'Mr'][titanic.Sex == 'female'].index
```

    /Users/ipm/anaconda/lib/python2.7/site-packages/ipykernel/__main__.py:1: UserWarning: Boolean Series key will be reindexed to match DataFrame index.
      if __name__ == '__main__':


We have 1 doctor who is female and we have replaced all doctors with Mr. So now we need to change this particular observation to be Mrs


```python
titanic.set_value(ind, "Salut_group", "Mrs").head(2)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Pclass</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>Ticket</th>
      <th>Fare</th>
      <th>Cabin</th>
      <th>Embarked</th>
      <th>Salutation</th>
      <th>Salut_group</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>Braund, Mr. Owen Harris</td>
      <td>male</td>
      <td>22.0</td>
      <td>1</td>
      <td>0</td>
      <td>A/5 21171</td>
      <td>7.2500</td>
      <td>None</td>
      <td>S</td>
      <td>Mr</td>
      <td>Mr</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>Cumings, Mrs. John Bradley (Florence Briggs Th...</td>
      <td>female</td>
      <td>38.0</td>
      <td>1</td>
      <td>0</td>
      <td>PC 17599</td>
      <td>71.2833</td>
      <td>C85</td>
      <td>C</td>
      <td>Mrs</td>
      <td>Mrs</td>
    </tr>
  </tbody>
</table>
</div>




```python
titanic.loc[ind]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Pclass</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>Ticket</th>
      <th>Fare</th>
      <th>Cabin</th>
      <th>Embarked</th>
      <th>Salutation</th>
      <th>Salut_group</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>796</th>
      <td>796</td>
      <td>797</td>
      <td>1</td>
      <td>1</td>
      <td>Leader, Dr. Alice (Farnham)</td>
      <td>female</td>
      <td>49.0</td>
      <td>0</td>
      <td>0</td>
      <td>17465</td>
      <td>25.9292</td>
      <td>D17</td>
      <td>S</td>
      <td>Dr</td>
      <td>Mrs</td>
    </tr>
  </tbody>
</table>
</div>




```python
titanic.Age.isnull().sum()
```




    0




```python
# Plot the Age histogram
# Set up a grid of plots
fig = plt.figure(figsize=(10,8))
plt.subplot2grid(fig_dims, (0, 0))
titanic['Age'].hist(bins=10)
plt.title('Age Distribution')

#normalize each row by transposing, normalizing each column, and un-transposing
group = (1. * group.T / group.T.sum()).T

print "Transpose" , group.T
plt.subplot2grid(fig_dims, (0, 1))
plt.bar([0, 1], group[0], color = 'g', label='Died')
plt.bar([0, 1], group[1], bottom=group[0], color= 'b', label='Survived')
plt.xticks([0.5, 1.5], ['female', 'male'], rotation='horizontal')
plt.ylabel("Fraction")
plt.title("Survival Rates by Gender")
plt.legend(loc='best') 


```

    Transpose Sex         female      male
    Survived                    
    0         0.257962  0.811092
    1         0.742038  0.188908





    <matplotlib.legend.Legend at 0x11eb95390>




![png](output_46_2.png)


Age looks pretty much normally distributed...
Almost 3 quarters of females survived while only 1 quarter of male survived

### Do family members have higher chances of survival than people who were travelling alone?


```python
titanic["Family_size"] = titanic["Parch"] + titanic["SibSp"] + 1
fm = titanic.groupby(["Family_size"])["Survived"].value_counts(normalize=True).unstack()
fm.plot.bar();
plt.title('Family Size Vs Survival rate')
plt.ylabel("Survival Rate")
```




    <matplotlib.text.Text at 0x11edf54d0>




![png](output_49_1.png)


If the passenger was travelling alone, the chances of passenger being survived was 30%. However, if the passenger was travelling with a family of upto 4 people, the chances of survival has increased upto 72%. However, if the familt is more than 4 members, the chances of survival dimishes again.


```python
# Replace missing values for port of Embankment by the maximum occuring value for Embarked
titanic["Embarked"] = titanic.Embarked.fillna('S')
```


```python
titanic.groupby("Embarked")["Survived"].value_counts(normalize=True).unstack()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Survived</th>
      <th>0</th>
      <th>1</th>
    </tr>
    <tr>
      <th>Embarked</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>C</th>
      <td>0.446429</td>
      <td>0.553571</td>
    </tr>
    <tr>
      <th>Q</th>
      <td>0.610390</td>
      <td>0.389610</td>
    </tr>
    <tr>
      <th>S</th>
      <td>0.660991</td>
      <td>0.339009</td>
    </tr>
  </tbody>
</table>
</div>



It looks like there is some difference in survival rates based on their port of embarkment

#### 2. Visualize the Data


```python
plt.hist(titanic.Fare, bins=10)
```




    (array([ 732.,  106.,   31.,    2.,   11.,    6.,    0.,    0.,    0.,    3.]),
     array([   0.     ,   51.23292,  102.46584,  153.69876,  204.93168,
             256.1646 ,  307.39752,  358.63044,  409.86336,  461.09628,
             512.3292 ]),
     <a list of 10 Patch objects>)




![png](output_55_1.png)



```python
titanic.Fare.describe()
```




    count    891.000000
    mean      32.204208
    std       49.693429
    min        0.000000
    25%        7.910400
    50%       14.454200
    75%       31.000000
    max      512.329200
    Name: Fare, dtype: float64




```python
titanic.PassengerId.loc[titanic.Fare == 0].count()
```




    15




```python
titanic.boxplot(column='Age', by = 'Salut_group')
plt.title("Age by Salutation")
```




    <matplotlib.text.Text at 0x1006371d0>




![png](output_58_1.png)



```python
titanic.boxplot(column='Fare', by = 'Pclass')
plt.ylim((0,250))
```




    (0, 250)




![png](output_59_1.png)


15 passengers did not pay any fare... while some passngers paid over 500. Data seems to be highly skewed on both sides. It might be data issue, so let's fix it.. 
Let's treat the outliers... 


```python
titanic.Fare.describe(percentiles=[0.995,.02])
```




    count    891.000000
    mean      32.204208
    std       49.693429
    min        0.000000
    2%         6.397500
    50%       14.454200
    99.5%    263.000000
    max      512.329200
    Name: Fare, dtype: float64




```python
titanic.ix[titanic.Fare > 263, "Fare"] = 263
titanic.ix[titanic.Fare <6.39, "Fare"] = 6.39
```


```python
titanic.Fare.describe()
```




    count    891.000000
    mean      31.476691
    std       43.184946
    min        6.390000
    25%        7.910400
    50%       14.454200
    75%       31.000000
    max      263.000000
    Name: Fare, dtype: float64




```python
plt.hist(np.log(titanic.Fare),bins=10)
```




    (array([ 316.,  113.,   82.,  128.,   77.,   53.,   56.,   28.,   18.,   20.]),
     array([ 1.85473427,  2.22647624,  2.59821822,  2.9699602 ,  3.34170217,
             3.71344415,  4.08518613,  4.4569281 ,  4.82867008,  5.20041206,
             5.57215403]),
     <a list of 10 Patch objects>)




![png](output_64_1.png)



```python
titanic["logFare"] = np.log(titanic.Fare)
```


```python
titanic.head(2)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Pclass</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>Ticket</th>
      <th>Fare</th>
      <th>Cabin</th>
      <th>Embarked</th>
      <th>Salutation</th>
      <th>Salut_group</th>
      <th>Family_size</th>
      <th>logFare</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>Braund, Mr. Owen Harris</td>
      <td>male</td>
      <td>22.0</td>
      <td>1</td>
      <td>0</td>
      <td>A/5 21171</td>
      <td>7.2500</td>
      <td>None</td>
      <td>S</td>
      <td>Mr</td>
      <td>Mr</td>
      <td>2</td>
      <td>1.981001</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>Cumings, Mrs. John Bradley (Florence Briggs Th...</td>
      <td>female</td>
      <td>38.0</td>
      <td>1</td>
      <td>0</td>
      <td>PC 17599</td>
      <td>71.2833</td>
      <td>C85</td>
      <td>C</td>
      <td>Mrs</td>
      <td>Mrs</td>
      <td>2</td>
      <td>4.266662</td>
    </tr>
  </tbody>
</table>
</div>




```python
## Passengers survived
```


```python
a = titanic.groupby(["Salut_group","Pclass"])["Survived"].sum().unstack()
a.columns = ["Pclass1", "Pclass2", "Pclass3"]
a
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Pclass1</th>
      <th>Pclass2</th>
      <th>Pclass3</th>
    </tr>
    <tr>
      <th>Salut_group</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Master</th>
      <td>3</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>Miss</th>
      <td>44</td>
      <td>33</td>
      <td>51</td>
    </tr>
    <tr>
      <th>Mr</th>
      <td>42</td>
      <td>8</td>
      <td>36</td>
    </tr>
    <tr>
      <th>Mrs</th>
      <td>47</td>
      <td>37</td>
      <td>21</td>
    </tr>
  </tbody>
</table>
</div>




```python
## Total passengers 
```


```python
b = titanic.groupby(["Salut_group","Pclass"])["Survived"].count().unstack()
b.columns = ["Pclass1", "Pclass2", "Pclass3"]
b
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Pclass1</th>
      <th>Pclass2</th>
      <th>Pclass3</th>
    </tr>
    <tr>
      <th>Salut_group</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Master</th>
      <td>3</td>
      <td>9</td>
      <td>28</td>
    </tr>
    <tr>
      <th>Miss</th>
      <td>46</td>
      <td>35</td>
      <td>102</td>
    </tr>
    <tr>
      <th>Mr</th>
      <td>119</td>
      <td>99</td>
      <td>319</td>
    </tr>
    <tr>
      <th>Mrs</th>
      <td>48</td>
      <td>41</td>
      <td>42</td>
    </tr>
  </tbody>
</table>
</div>




```python
## Percantage passengers survived
```


```python
c = a/b
c
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Pclass1</th>
      <th>Pclass2</th>
      <th>Pclass3</th>
    </tr>
    <tr>
      <th>Salut_group</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Master</th>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.392857</td>
    </tr>
    <tr>
      <th>Miss</th>
      <td>0.956522</td>
      <td>0.942857</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>Mr</th>
      <td>0.352941</td>
      <td>0.080808</td>
      <td>0.112853</td>
    </tr>
    <tr>
      <th>Mrs</th>
      <td>0.979167</td>
      <td>0.902439</td>
      <td>0.500000</td>
    </tr>
  </tbody>
</table>
</div>




```python
c.plot.bar();
plt.title('Salutation group Vs Survival rate')
plt.ylabel("Survival Rate")
plt.legend(loc="best")
```




    <matplotlib.legend.Legend at 0x11f6d9210>




![png](output_73_1.png)



```python
### plot the class distribution ###
import brewer2mpl
tclass = titanic.groupby(['Pclass', 'Survived']).size().unstack()
print tclass
print "tclass0", tclass[0]

dark2_colors = brewer2mpl.get_map('Dark2', 'Qualitative', 7).mpl_colors

fig = plt.figure(figsize=(10, 6)) 
plt.subplot(121)
plt.bar([0, 1, 2], tclass[0], color=dark2_colors[2], label='Died')
plt.bar([0, 1, 2], tclass[1], bottom=tclass[0], color=dark2_colors[0], label='Survived')
plt.xticks([0.5, 1.5, 2.5], ['1st Class', '2nd Class', '3rd Class'], rotation='horizontal')
plt.ylabel("Count")
plt.xlabel("")
plt.legend(loc='upper left')

#normalize each row by transposing, normalizing each column, and un-transposing
tclass = (1. * tclass.T / tclass.T.sum()).T

print "Transpose" , tclass.T
plt.subplot(122)
plt.bar([0, 1, 2], tclass[0], color=dark2_colors[2], label='Died')
plt.bar([0, 1, 2], tclass[1], bottom=tclass[0], color=dark2_colors[0], label='Survived')
plt.xticks([0.5, 1.5, 2.5], ['1st Class', '2nd Class', '3rd Class'], rotation='horizontal')
plt.ylabel("Fraction")
plt.xlabel("")


```

    Survived    0    1
    Pclass            
    1          80  136
    2          97   87
    3         372  119
    tclass0 Pclass
    1     80
    2     97
    3    372
    Name: 0, dtype: int64
    Transpose Pclass          1         2         3
    Survived                             
    0         0.37037  0.527174  0.757637
    1         0.62963  0.472826  0.242363





    <matplotlib.text.Text at 0x11fc805d0>




![png](output_74_2.png)


High percenatge of class 1 and class 2 people survived and relatively small percentage of class 3 people survived

## Part 3: Data Wrangling

#### 1. Create Dummy Variables for *Sex* 


```python
titanic.head(2)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Pclass</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>Ticket</th>
      <th>Fare</th>
      <th>Cabin</th>
      <th>Embarked</th>
      <th>Salutation</th>
      <th>Salut_group</th>
      <th>Family_size</th>
      <th>logFare</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>Braund, Mr. Owen Harris</td>
      <td>male</td>
      <td>22.0</td>
      <td>1</td>
      <td>0</td>
      <td>A/5 21171</td>
      <td>7.2500</td>
      <td>None</td>
      <td>S</td>
      <td>Mr</td>
      <td>Mr</td>
      <td>2</td>
      <td>1.981001</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>Cumings, Mrs. John Bradley (Florence Briggs Th...</td>
      <td>female</td>
      <td>38.0</td>
      <td>1</td>
      <td>0</td>
      <td>PC 17599</td>
      <td>71.2833</td>
      <td>C85</td>
      <td>C</td>
      <td>Mrs</td>
      <td>Mrs</td>
      <td>2</td>
      <td>4.266662</td>
    </tr>
  </tbody>
</table>
</div>




```python
titanic.Sex = titanic.Sex.replace({'male': 1, 'female':0})
```

Lets create a dummy variable to Pclass and Embarked


```python
dummies = ["Pclass", "Embarked"]
for col in dummies:
    print col
    res = pd.get_dummies(titanic[col], prefix=col)
    res = res.drop(res.columns[-1], axis=1)
    titanic = pd.concat([titanic, res], axis=1)
    titanic.head()
```

    Pclass
    Embarked



```python
titanic.head(2)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Pclass</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>Ticket</th>
      <th>...</th>
      <th>Cabin</th>
      <th>Embarked</th>
      <th>Salutation</th>
      <th>Salut_group</th>
      <th>Family_size</th>
      <th>logFare</th>
      <th>Pclass_1</th>
      <th>Pclass_2</th>
      <th>Embarked_C</th>
      <th>Embarked_Q</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>Braund, Mr. Owen Harris</td>
      <td>1</td>
      <td>22.0</td>
      <td>1</td>
      <td>0</td>
      <td>A/5 21171</td>
      <td>...</td>
      <td>None</td>
      <td>S</td>
      <td>Mr</td>
      <td>Mr</td>
      <td>2</td>
      <td>1.981001</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>Cumings, Mrs. John Bradley (Florence Briggs Th...</td>
      <td>0</td>
      <td>38.0</td>
      <td>1</td>
      <td>0</td>
      <td>PC 17599</td>
      <td>...</td>
      <td>C85</td>
      <td>C</td>
      <td>Mrs</td>
      <td>Mrs</td>
      <td>2</td>
      <td>4.266662</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>2 rows × 21 columns</p>
</div>



## Part 4: Logistic Regression and Model Validation

#### 1. Define the variables that we will use in our classification analysis


```python
X = titanic[["Sex", "Age", "SibSp", "Parch", "Fare", "Family_size", "Pclass_2", "Pclass_1", "Embarked_C", "Embarked_Q"]]
```

#### 2. Transform "Y" into a 1-Dimensional Array for SciKit-Learn


```python
y= titanic.Survived
```

#### 3. Conduct the logistic regression


```python
import seaborn as sns
correlationMatrix = X.corr().abs()

plt.subplots(figsize=(10, 6))
sns.heatmap(correlationMatrix,annot=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x120155810>




![png](output_89_1.png)


Family size is highly correlated to Parch and Sibsp. So I am going to remove Parch and Sibsp. It perfectly makes sense as I derived family size based on Sibsp and Parch
We have high negative correlation between Pclass2 and Pclass3. I will keep this as when I run the model, we will compare the results of Pclass2 Vs Pclass1 and Pclass3 vs Pclass1


```python
X = titanic[["Sex", "Age", "logFare", "Family_size", "Pclass_2", "Pclass_1", "Embarked_C", "Embarked_Q"]]
```


```python
import seaborn as sns
correlationMatrix = X.corr().abs()

plt.subplots(figsize=(10, 6))
sns.heatmap(correlationMatrix,annot=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11f5f4850>




![png](output_92_1.png)



```python
#from sklearn.linear_model import LogisticRegression
#from sklearn.model_selection import cross_val_score

model = LogisticRegression()
# that's accuracy score for logistic regression
scores = cross_val_score(model, X, y)

print('CV scores: {}'.format(scores))
print('Average CVScore: {:0.3f} +/- {:0.3f}'.format(scores.mean(), scores.std()))
```

    CV scores: [ 0.78787879  0.7979798   0.8013468 ]
    Average CVScore: 0.796 +/- 0.006


#### 6. Test the Model by introducing a *Test* or *Validaton* set 


```python
from sklearn.model_selection import train_test_split
from sklearn import linear_model
from sklearn.metrics import mean_squared_error, mean_absolute_error

X_train, X_test, y_train, y_test = train_test_split(X,y, test_size=0.2, random_state=10)

logreg=LogisticRegression()
model=logreg.fit(X_train,y_train)
predictions_test=logreg.predict(X_test)

scores = cross_val_score(model, X, y)


print('CV scores: {}'.format(scores))
print('Average CVScore: {:0.3f} +/- {:0.3f}'.format(scores.mean(), scores.std()))
```

    CV scores: [ 0.78787879  0.7979798   0.8013468 ]
    Average CVScore: 0.796 +/- 0.006


#### 7. Predict the class labels for the *Test* set


```python
%matplotlib inline
sns.pairplot( data = titanic[["Survived", "Fare", "Pclass", "Age"]],hue = "Survived")
```




    <seaborn.axisgrid.PairGrid at 0x11fa086d0>




![png](output_97_1.png)


#### 9. Evaluate the *Test* set


```python
from sklearn.model_selection import cross_val_score
from sklearn import model_selection, linear_model, metrics
from sklearn.metrics import accuracy_score

model = linear_model.LogisticRegression()
model.fit(X_train,y_train)

gs = model_selection.GridSearchCV(
    estimator = model,
    param_grid = {'C': [10**-i for i in range(-2, 4)],
                 'penalty':['l1','l2'],
                 'fit_intercept':[True]},
    cv = 10,
    scoring = 'roc_auc'
    )
gs.fit(X_train, y_train)
print gs.best_params_, gs.best_score_
#gs.grid_scores_
```

    {'penalty': 'l2', 'C': 1, 'fit_intercept': True} 0.849270042572


We had a wide range of C parameter.. From the first run, we have found that C = 1... Now, lets try to tune C with a smaller range and see if we get any improvement in predictions



```python
from sklearn.model_selection import train_test_split, cross_val_predict, cross_val_score
from sklearn.metrics import accuracy_score, average_precision_score, f1_score, recall_score, precision_score

# Fit the model
model = linear_model.LogisticRegression()
model.fit(X_train,y_train)

# Tune the hyper parameters using grid search

gs = model_selection.GridSearchCV(
    estimator = model,
    param_grid = {'C': np.arange(1,100,1),
                 'penalty':['l1','l2'],
                 'fit_intercept':[True],
                 'solver':['liblinear']},
    cv = 10,
    scoring = 'roc_auc'
    )

        
# Find the optimum parameters
gs.fit(X_train, y_train)
print gs.best_params_, gs.best_score_
#gs.grid_scores_
```

    {'penalty': 'l2', 'C': 2, 'solver': 'liblinear', 'fit_intercept': True} 0.849352865745



```python
# Test the model performance with tunned parameters
model_new = linear_model.LogisticRegression(C=51,penalty = 'l2', fit_intercept=True)
model_new.fit(X_train, y_train)

scores = cross_val_score(model_new, X_train, y_train)
predicted = cross_val_predict(model_new,X_test,y_test, cv=5)
print "accuracy_score", accuracy_score(y_test, predicted)
print "precision_score", precision_score(y_test,predicted)
print "f1_score", f1_score(y_test,predicted)
print "recall_score", recall_score(y_test,predicted)
print "Score", np.mean(scores)


```

    accuracy_score 0.810055865922
    precision_score 0.75
    f1_score 0.71186440678
    recall_score 0.677419354839
    Score 0.797698826366



```python
from sklearn.model_selection import cross_val_score, KFold

# cross-validation where the model is fit to a portion of the training data, and the 
# is tested against the rest of the training data

for metric in ['accuracy', 'precision', 'recall', 'roc_auc']:
    scores = cross_val_score(model_new, X_test, y_test, scoring=metric)
    print("mean {}: {}, all: {}".format(metric, scores.mean(), scores))

```

    mean accuracy: 0.821092278719, all: [ 0.83333333  0.83333333  0.79661017]
    mean precision: 0.757936507937, all: [ 0.76190476  0.76190476  0.75      ]
    mean recall: 0.707936507937, all: [ 0.76190476  0.76190476  0.6       ]
    mean roc_auc: 0.862657712658, all: [ 0.84859585  0.85347985  0.88589744]



```python
# Check the model performance on test data

model_score = model_new.score(X_test, y_test)
print ("Model Score %.2f \n" % (model_score))

confusion_matrix = metrics.confusion_matrix(y_test, predicted)

print confusion_matrix
```

    Model Score 0.83 
    
    [[103  14]
     [ 20  42]]


The logistic regression model has been able to identify 103 passengers who were actually survived correctly, 
and model predicted correctly that 43 passengers would have died. We know that the model can not predict 100% correct results all the time(False positives). In the same way, we had 14 passengers who died were incorrectly classified as passngers survived and 19 passsngers who survived were incorrectly classified as died(True Negatives).

#### 15. Plot the ROC curve


```python
def auc_plotting_function(rate1, rate2, rate1_name, rate2_name, curve_name):
    AUC = auc(rate1, rate2)
    # Plot of a ROC curve for class 1 (has_cancer)
    #plt.figure(figsize=[11,9])
    plt.plot(rate1, rate2, label=curve_name + ' (area = %0.2f)' % AUC, linewidth=4)
    plt.plot([0, 1], [0, 1], 'k--', linewidth=4)
    plt.xlim([0.0, 1.0])
    plt.ylim([0.0, 1.05])
    plt.xlabel(rate1_name, fontsize=18)
    plt.ylabel(rate2_name, fontsize=18)
    plt.title(curve_name, fontsize=18)
    plt.legend(loc="lower right")
    plt.show()

# plot receiving operator characteristic curve
def plot_roc(y_true, y_score):
    fpr, tpr, _ = roc_curve(y_true, y_score)
    auc_plotting_function(fpr, tpr, 'False Positive Rate (1-specificity)', 'True Positive Rate (sensitivity)', 'ROC')
```


```python
from sklearn.metrics import roc_curve, auc
Y_score = logreg.decision_function(X_test)
print y_test.shape
print Y_score.shape
plot_roc(y_test, Y_score)
```

    (179,)
    (179,)



![png](output_108_1.png)


Mean accuracy of logistic regression model is 82.66%
Roc-AUC of 86.59 shows that the area under the curve covers 86.5% of the area. This seems to be a pretty good model, We have got a really good lift in the first 2 deciles where 20% data gives us 80% correct predictions. 


#### 16. What does the ROC curve tell us?

RoC curve gives us a clear indication about the impact of the model. If we did not have any model, the random prediction is specified as dotted line. And the blue thick line shows how much the model has been able to predict correctly. The larger the area covered by the blue line, suggest that the model has been able to predict correctly

#### 3. Explain the difference between the difference between the L1 (Lasso) and L2 (Ridge) penalties on the model coefficients.

Lasso and Ridge are the regularisation techniques. 
Ridge uses L2 regularisation while lasso uses L1 regularisation. In ridge regression, the penalty is the sum of the squares of the coefficients and for the Lasso, it's the sum of the absolute values of the coefficients.
Ridge allows all parameters to be there in the model, however some parameters might have very small coefficents.
Lasso regression excludes some parameters who do not add more value to the model. So Lasso regression is effectively reduces the number of predictors. 




#### 4. What hypothetical situations are the Ridge and Lasso penalties useful?

When we have a lot of predictors, say 500, then we need to reduce the dimension of data. As the number of predictors increases, there r square increases, may be in a vary small amount. We do not want to have 500 variables predicting the outcome of the model. So Ridge and Lasso comes into picture. These regularisation technique, penalises the parameter coefficents based on the number of parameters being added to the regression. 
So effectively, it helps in reducing the dimensions of the data and also helps in avoiding the overfitting. 

#### 5. [BONUS] Explain how the regularization strength (C) modifies the regression loss function. Why do the Ridge and Lasso penalties have their respective effects on the coefficients?

C is inverse of regularization strength. It must be a positive float. Smaller values specify stronger regularization. This means, if we have bigger value of C, we might not be able to reach global optima or it might take a very long time to reach global optima, as the model will keep on searching with longer steps, while if we have a very small C, we have more chance to reach global optima and it might take much longer again, as we are taking very small steps.

Lasso excludes the parameters while ridge keep all parameters even if they have a very small parameter estimates

#### 6.a. [BONUS] You decide that you want to minimize false positives. Use the predicted probabilities from the model to set your threshold for labeling the positive class to need at least 90% confidence. How and why does this affect your confusion matrix?

False positives are primarily false alarms when actually the event has not happened. E.g. if we are testing for cancer, we do not want any patient to be told that they have cancer when actually they don't. In certain cases, we would like to reduce false postives. There are many applications where we would like to reduce false positives like in fraud, airline failures, surgery failures etc. It depends on the business objective on how much false postives could be allowed. 

Without any threshold, sklearn selects observations having score >0.5 to be 1 and observations having score <=0.5 to be 0. But for airline failure that score could be 0.95. If we change the threshold to say, 0.95, the confusion matrix will change, as we will select any observation having score >0.95 to be 1 and rest all to be 0.

## Part 6: Gridsearch and kNN

#### 1. Perform Gridsearch for the same classification problem as above, but use KNeighborsClassifier as your estimator

At least have number of neighbors and weights in your parameters dictionary.


```python
#from sklearn.neighbors import KNeighborsClassifier
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=42)

scores_test = []
predictions = []
for k in range(1,101):
    model = KNeighborsClassifier(n_neighbors=k)
    model.fit(X_train,y_train)
    predict = model.predict(X_test) 
    predictions.append(predict)
    scores_test.append(model.score(X_test,y_test))
[(i+1,scores_test[i]) for i in range(len(scores_test)) if scores_test[i] == max(scores_test)]
```




    [(3, 0.78731343283582089)]




```python
score_list = []
score_means = []
accuracy_list = []
precision_list = []
f1_list = []
recall_list = []
for k in range(1,101):
    model = KNeighborsClassifier(n_neighbors=k)
    scores = cross_val_score(model, X, y, cv=5)
    predicted = cross_val_predict(model,X,y, cv=5)
    accuracy_list.append(accuracy_score(y, predicted))
    precision_list.append(precision_score(y,predicted))
    f1_list.append(f1_score(y,predicted))
    recall_list.append(recall_score(y,predicted))
    score_list.append(scores)
    score_means.append(np.mean(scores))
# Best k
[(i+1,score_means[i]) for i in range(len(score_means)) if score_means[i] == max(score_means)]
```




    [(7, 0.77340820423670309)]




```python
line_1, = plt.plot(range(1,101),scores_test,c='r')
line_2, = plt.plot(range(1,101),score_means,c='b')
plt.xlabel('k')
plt.legend([line_1, line_2], ['Test set','5 CV'],loc='upper right')
plt.show()
```


![png](output_125_0.png)



```python
line_1, = plt.plot(range(1,101),scores_test,c='r')
line_2, = plt.plot(range(1,101),score_means,c='b')
plt.xlabel('k')
plt.legend([line_1, line_2], ['Test set','5 CV'],loc='upper right')
plt.show()
```


![png](output_126_0.png)


#### 5. Fit a new kNN model with the optimal parameters found in gridsearch. 


```python
from sklearn import grid_search
parameters = {'n_neighbors':(3,4,5,6,7,8), 'leaf_size':(3,4,5,6,10)}
knn = KNeighborsClassifier()
print knn.get_params().keys()
clf = grid_search.GridSearchCV(knn, param_grid=parameters, cv=4)
clf.fit(X_train,y_train)
clf.best_params_
```

    /Users/ipm/anaconda/lib/python2.7/site-packages/sklearn/cross_validation.py:44: DeprecationWarning: This module was deprecated in version 0.18 in favor of the model_selection module into which all the refactored classes and functions are moved. Also note that the interface of the new CV iterators are different from that of this module. This module will be removed in 0.20.
      "This module will be removed in 0.20.", DeprecationWarning)
    /Users/ipm/anaconda/lib/python2.7/site-packages/sklearn/grid_search.py:43: DeprecationWarning: This module was deprecated in version 0.18 in favor of the model_selection module into which all the refactored classes and functions are moved. This module will be removed in 0.20.
      DeprecationWarning)


    ['n_neighbors', 'n_jobs', 'algorithm', 'metric', 'metric_params', 'p', 'weights', 'leaf_size']





    {'leaf_size': 3, 'n_neighbors': 3}



#### 6. Construct the confusion matrix for the optimal kNN model. Is it different from the logistic regression model? If so, how?


```python
knn = KNeighborsClassifier(n_neighbors= 8, leaf_size= 10)
knn.fit(X_train,y_train)
output_dt = knn.predict(X_test)
print "fit to all test data:",knn.score(X_test,y_test)
```

    fit to all test data: 0.738805970149


Knn model does not look very good, as we have got accurancy of 73%. Regression was much better model in this case.

#### 7. [BONUS] Plot the ROC curves for the optimized logistic regression model and the optimized kNN model on the same plot.


```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=42)
Mean_List=[]
N=[]
for a in range(1,100):
    knn=KNeighborsClassifier(n_neighbors=a)
    knn_model=knn.fit(X_train,y_train)
    Mean_List.append(knn_model.score(X,y))
    N.append(a)
Output1=pd.DataFrame({'Number':N,'Mean_Accuracy':Mean_List})
Output1.head()

Mean_List=[]
Mean_List2=[]
N=[]
Recall=[]
Precision=[]
f1=[]
for a in range(1,100):
    knn=KNeighborsClassifier(n_neighbors=a)
    accuracies = cross_val_score(knn, X_train, y_train, cv=5)
    y_pred = cross_val_predict(knn, X_train, y_train, cv=5)
    accuracies2=np.mean(accuracy_score(y_train,y_pred))
    Mean_List.append(accuracies.mean())
    Mean_List2.append(accuracies2)
    N.append(a)
    Precision.append(np.mean(average_precision_score(y_train,y_pred)))
    f1.append(np.mean(f1_score(y_train,y_pred)))
    Recall.append(np.mean(recall_score(y_train,y_pred)))
Output2=pd.DataFrame({'Number':N,'Mean_Accuracy':Mean_List,'Mean_Accuracy_2':Mean_List2,
                      'Recall':Recall,'F1':f1,'Precision':Precision})
Output2.head()

# Best k
Output2.sort_values(by='F1',ascending=False).head()
# plot mean accuracy versus k
fig=plt.figure(figsize=(16,12))
ax1=fig.add_subplot(221)
ax2=fig.add_subplot(222)
ax3=fig.add_subplot(223)
ax4=fig.add_subplot(224)
Output2.plot(x='Number', y='Mean_Accuracy',ax=ax1)
Output2.plot(x='Number', y='Recall',ax=ax2)
Output2.plot(x='Number', y='F1',ax=ax3)
Output2.plot(x='Number', y='Precision',ax=ax4)


```




    <matplotlib.axes._subplots.AxesSubplot at 0x1249da2d0>




![png](output_133_1.png)



```python
from sklearn.metrics import roc_curve, auc
knn_model=KNeighborsClassifier(n_neighbors=3)
knn_model.fit(X_train,y_train)
y_score = knn_model.predict_proba(X_test)
#print y_score
Y_score = []
for item in y_score:
    Y_score.append(item[1])
print y_test.shape
Y_score = pd.Series(Y_score)
#print Y_score
plot_roc(y_test, Y_score)
```

    (268,)



![png](output_134_1.png)


Regularised Knn is much better than knn model. Regularised knn has shown the accuracy of 83%.. not bad...

## Part 7: [BONUS] Precision-recall

#### 1. Gridsearch the same parameters for logistic regression but change the scoring function to 'average_precision'

`'average_precision'` will optimize parameters for area under the precision-recall curve instead of for accuracy.


```python
# Fit the model
model = linear_model.LogisticRegression()
model.fit(X_train,y_train)

# Tune the hyper parameters using grid search

gs = model_selection.GridSearchCV(
    estimator = model,
    param_grid = {'C': np.arange(1,100,1),
                 'penalty':['l1','l2'],
                 'fit_intercept':[True],
                 'solver':['liblinear']},
    cv = 10,
    scoring = 'average_precision'
    )

        
# Find the optimum parameters
gs.fit(X_train, y_train)
print gs.best_params_, gs.best_score_
```

    {'penalty': 'l2', 'C': 2, 'solver': 'liblinear', 'fit_intercept': True} 0.796636959996



```python

# Fit the model
model = linear_model.LogisticRegression()
model.fit(X_train,y_train)

# Tune the hyper parameters using grid search

gs = model_selection.GridSearchCV(
    estimator = model,
    param_grid = {'C': np.arange(0.1,2,.1),
                 'penalty':['l1','l2'],
                 'fit_intercept':[True],
                 'solver':['liblinear']},
    cv = 10,
    scoring = 'average_precision'
    )

        
# Find the optimum parameters
gs.fit(X_train, y_train)
print gs.best_params_, gs.best_score_
```

    {'penalty': 'l2', 'C': 0.30000000000000004, 'solver': 'liblinear', 'fit_intercept': True} 0.797890063391


#### 2. Examine the best parameters and score. Are they different than the logistic regression gridsearch in part 5?

the parameters are same but the scores are different. 

#### 3. Create the confusion matrix. Is it different than when you optimized for the accuracy? If so, why would this be?


```python
model_new = linear_model.LogisticRegression(C=0.3 ,penalty = 'l2', fit_intercept=True)
model_new.fit(X_train, y_train)

predicted = cross_val_predict(model_new,X_test,y_test, cv=5)

model_score = model_new.score(X_test, y_test)
print ("Model Score %.2f \n" % (model_score))

confusion_matrix = metrics.confusion_matrix(y_test, predicted)

print confusion_matrix
```

    Model Score 0.81 
    
    [[137  20]
     [ 32  79]]


## Part 8: Decision trees, ensembles, bagging

#### 1. Gridsearch a decision tree classifier model on the data, searching for optimal depth. Create a new decision tree model with the optimal parameters.


```python
#from sklearn import tree
#from sklearn.cross_validation import cross_val_score, KFold

# build model
#dtc = tree.DecisionTreeClassifier()
dtc = tree.DecisionTreeClassifier()
dtc.fit(X_train,y_train)
output_dt = dtc.predict(X_test)
print "fit to all test data:",dtc.score(X_test,y_test)
```

    fit to all test data: 0.761194029851



```python
parameters = {'max_depth':(3,4,5,6), 'min_samples_leaf':(3,4,5,6), 'max_features' : (4,5,6,8)}
dtc = tree.DecisionTreeClassifier()
print dtc.get_params().keys()
clf = grid_search.GridSearchCV(dtc, param_grid=parameters, cv=4)
clf.fit(X_train,y_train)
clf.best_params_
```

    ['presort', 'splitter', 'max_leaf_nodes', 'min_samples_leaf', 'min_samples_split', 'min_weight_fraction_leaf', 'criterion', 'random_state', 'min_impurity_split', 'max_features', 'max_depth', 'class_weight']





    {'max_depth': 4, 'max_features': 5, 'min_samples_leaf': 3}




```python
dtc = tree.DecisionTreeClassifier(max_depth= 4, max_features= 5, min_samples_leaf= 6)
dtc.fit(X_train,y_train)
output_dt = dtc.predict(X_test)
print "fit to all test data:",dtc.score(X_test,y_test)
```

    fit to all test data: 0.813432835821


#### 2. Compare the performace of the decision tree model to the logistic regression and kNN models.


```python
# Random Forests
from sklearn.ensemble import RandomForestClassifier
random_forest = RandomForestClassifier()

random_forest.fit(X_train, y_train)

Y_pred = random_forest.predict(X_test)

random_forest.score(X_test, y_test)

```




    0.80223880597014929



#### 3. Plot all three optimized models' ROC curves on the same plot. 


```python
plot_roc(y_test, Y_pred)
```


![png](output_152_0.png)



```python
from sklearn import grid_search
# grid search the parameters
parameters = {'n_estimators':(10,50,100), 'max_features':(2,3,4,5,6), 'max_depth':(2,3,4,5,6)}
rfc = RandomForestClassifier()
print rfc.get_params().keys()
clf = grid_search.GridSearchCV(rfc, param_grid=parameters, cv=5)
clf.fit(X_train,y_train)
clf.best_params_
```

    ['warm_start', 'oob_score', 'n_jobs', 'verbose', 'max_leaf_nodes', 'bootstrap', 'min_samples_leaf', 'n_estimators', 'min_samples_split', 'min_weight_fraction_leaf', 'criterion', 'random_state', 'min_impurity_split', 'max_features', 'max_depth', 'class_weight']





    {'max_depth': 6, 'max_features': 3, 'n_estimators': 100}




```python
rfc1 = RandomForestClassifier(max_depth= 6, max_features= 6, n_estimators= 100)
rfc1.fit(X_train,y_train)
y_rfc_pred = rfc1.predict(X_test)
print y_rfc_pred.shape
rfc1.score(X_test,y_test)
print ("Model Score %.2f \n" % (rfc1.score(X_test,y_test)))

confusion_matrix = metrics.confusion_matrix(y_test, y_rfc_pred)

print confusion_matrix
```

    (268,)
    Model Score 0.82 
    
    [[141  16]
     [ 33  78]]



```python
plot_roc(y_test, y_rfc_pred)
```


![png](output_155_0.png)



```python

```


```python

```


```python


