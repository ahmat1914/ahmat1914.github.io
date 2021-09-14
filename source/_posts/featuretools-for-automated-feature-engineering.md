---
title: featuretools for automated feature engineering
category:
  - MachineLearning
tags:
  - 特征工程
mathjax: false
date: 2021-09-14 14:49:46
img:
---

# Reading Notes

### Overview

> One of the holy grails of machine learning is to automate more and more of the feature engineering process. ― Pedro Domingos

Featuretools is a python library for automated feature engineering.

* [Examples](https://featuretools.alteryx.com/en/stable/getting_started/afe.html)
* [define your own custom primitives](https://featuretools.alteryx.com/en/stable/getting_started/primitives.html#defining-custom-primitives)
* [demos](https://www.featuretools.com/demos/)
* Paper: [Deep feature synthesis: Towards automating data science endeavors](https://dai.lids.mit.edu/wp-content/uploads/2017/10/DSAA_DSM_2015.pdf)


### Quick Start

install with pip
```bash
python -m pip install featuretools
```

Below is an example of using Deep Feature Synthesis (DFS) to perform automated feature engineering. 

```python
>> import featuretools as ft
>> es = ft.demo.load_mock_customer(return_entityset=True)
>> es.plot()
```

```python
>> feature_matrix, features_defs = ft.dfs(entityset=es, target_entity="customers")
>> feature_matrix.head(5)
```

```
            zip_code  COUNT(transactions)  COUNT(sessions)  SUM(transactions.amount) MODE(sessions.device)  MIN(transactions.amount)  MAX(transactions.amount)  YEAR(join_date)  SKEW(transactions.amount)  DAY(join_date)                   ...                     SUM(sessions.MIN(transactions.amount))  MAX(sessions.SKEW(transactions.amount))  MAX(sessions.MIN(transactions.amount))  SUM(sessions.MEAN(transactions.amount))  STD(sessions.SUM(transactions.amount))  STD(sessions.MEAN(transactions.amount))  SKEW(sessions.MEAN(transactions.amount))  STD(sessions.MAX(transactions.amount))  NUM_UNIQUE(sessions.DAY(session_start))  MIN(sessions.SKEW(transactions.amount))
customer_id                                                                                                                                                                                                                                  ...
1              60091                  131               10                  10236.77               desktop                      5.60                    149.95             2008                   0.070041               1                   ...                                                     169.77                                 0.610052                                   41.95                               791.976505                              175.939423                                 9.299023                                 -0.377150                                5.857976                                        1                                -0.395358
2              02139                  122                8                   9118.81                mobile                      5.81                    149.15             2008                   0.028647              20                   ...                                                     114.85                                 0.492531                                   42.96                               596.243506                              230.333502                                10.925037                                  0.962350                                7.420480                                        1                                -0.470007
3              02139                   78                5                   5758.24               desktop                      6.78                    147.73             2008                   0.070814              10                   ...                                                      64.98                                 0.645728                                   21.77                               369.770121                              471.048551                                 9.819148                                 -0.244976                               12.537259                                        1                                -0.630425
4              60091                  111                8                   8205.28               desktop                      5.73                    149.56             2008                   0.087986              30                   ...                                                      83.53                                 0.516262                                   17.27                               584.673126                              322.883448                                13.065436                                 -0.548969                               12.738488                                        1                                -0.497169
5              02139                   58                4                   4571.37                tablet                      5.91                    148.17             2008                   0.085883              19                   ...                                                      73.09                                 0.830112                                   27.46                               313.448942                              198.522508                                 8.950528                                  0.098885                                5.599228                                        1                                -0.396571

[5 rows x 69 columns]
```

### Concepts

Feature engineering requires that we use what we understand about the data to build numeric rows (feature vectors) which we can use as input into machine learning algorithms. The primary benefit of Featuretools is that it does not require you make those features by hand. The requirement instead is that you pass in what you know about the data.

That knowledge is stored in a Featuretools [EntitySet](https://docs.featuretools.com/loading_data/using_entitysets.html). 

`EntitySets` are a collection of tables with  information about relationships between tables and semantic typing for every column.

We're going to show how to
+ pass ininformation about semantic types of columns,
+ load in a dataframe to an `EntitySet` and
+ tell the `EntitySet` about reasonable new `Entities` to make from that dataframe.

```python
# List the semantic type for each column

import featuretools.variable_types as vtypes
variable_types = {'gender': vtypes.Categorical,
                  'patient_id': vtypes.Categorical,
                  'age': vtypes.Ordinal,
                  'scholarship': vtypes.Boolean,
                  'hypertension': vtypes.Boolean,
                  'diabetes': vtypes.Boolean,
                  'alcoholism': vtypes.Boolean,
                  'handicap': vtypes.Boolean,
                  'no_show': vtypes.Boolean,
                  'sms_received': vtypes.Boolean}
```
The `variable_types` dictionary is a place to store information about the semantic type of each column. 
(While many types can be detected automatically, some are necessarily tricky.)

Changing the variable type will change which functions are automatically applied to generate features.

Make an entity `appointments`:

```python
# Make an entity named 'appointments' which stores dataset metadata with the dataframe
es = ft.EntitySet('Appointments')
es = es.entity_from_dataframe(entity_id="appointments",
                              dataframe=data,
                              index='appointment_id',
                              time_index='scheduled_time',
                              secondary_time_index={'appointment_day': ['no_show', 'sms_received']},
                              variable_types=variable_types)
es['appointments']
```
The time index and secondary time index notate what time the data is recorded. By doing that, we can avoid using data from the future while creating features.

```python
# Make a patients entity with patient-specific variables
es.normalize_entity('appointments', 'patients', 'patient_id',
                    additional_variables=['scholarship',
                                          'hypertension',
                                          'diabetes',
                                          'alcoholism',
                                          'handicap'])

# Make locations, ages and genders
es.normalize_entity('appointments', 'locations', 'neighborhood',
                    make_time_index=False)
es.normalize_entity('appointments', 'ages', 'age',
                    make_time_index=False)
es.normalize_entity('appointments', 'genders', 'gender',
                    make_time_index=False)

es.plot()
```

![](/images/appointment.svg)

Finally, we build new entities from our existing one using `normalize_entity`. We take unique values from `patient`, `age`, `neighborhood` and `gender` and make a new `Entity` for each whose rows are the unique values. To do that we only need to specify where we start (`appointments`), the name of the new entity (e.g. `patients`) and what the index should be (e.g. `patient_id`). Having those additional `Entities` and `Relationships` tells the algorithm about reasonable groupings which allows for some neat aggregations.
