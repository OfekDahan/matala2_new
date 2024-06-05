# Data Analysis on Cosmetics Transactions

This project performs data analysis on a dataset containing cosmetic transaction events. The steps include data cleaning, feature engineering, and visualization.

## Data

The dataset contains the following columns:

- `event_time`: The timestamp of the event.
- `event_type`: The type of event (view, cart, purchase, remove_from_cart).
- `product_id`: The ID of the product.
- `category_id`: The ID of the category.
- `category_code`: The code of the category.
- `brand`: The brand of the product.
- `price`: The price of the product.
- `user_id`: The ID of the user.
- `user_session`: The session ID of the user.

## Steps and Code Explanation

### 1. Import Libraries and Load Data

```python
import pandas as pd
import numpy as np
import re
import seaborn as sns
import matplotlib.pyplot as plt
from datetime import timedelta

datafile = "C:\\Users\\ofekd\\Desktop\\שנה ג תעשייה וניהול\\כריה וניתוח בפייתון\\מטלות\\matala2\\matala2_cosmetics_2019-Nov.csv"
data = pd.read_csv(datafile)
```

### 2. Convert `event_time` to datetime and Calculate Duration to Next Event

```python
data['event_time'] = pd.to_datetime(data['event_time'])
data['duration_to_next_event'] = abs((data.groupby('user_session')['event_time'].shift(-1) - data['event_time']).dt.seconds.fillna(0))
```

### 3. Assign Funnel Numbers Based on Time Differences

```python
data = data.sort_values(['user_id', 'event_time'])
data['days_diff'] = data.groupby('user_id')['event_time'].diff().dt.days
new = data['days_diff'] > 5
data['funnel_number'] = new.groupby(data['user_id']).cumsum() + 1
data['funnel_number'] = data['funnel_number'].fillna(0)
```

### 4. Assign Index in Funnel

```python
data = data.sort_values(['user_id', 'funnel_number', 'event_time'])
data['temp'] = data.groupby(['user_id', 'funnel_number', 'user_session'])['user_session'].shift().ne(0).astype(int)
data['index_in_funnel'] = data.groupby(['user_id', 'funnel_number'])['temp'].cumsum()
data = data.drop(columns=['temp'])
```

### 5. Clean and Convert `price` Column

```python
data['price'] = data['price'].apply(lambda x: float(re.findall(r'\d+\.\d+', x)[0]) if isinstance(x, str) else x)
```

### 6. Plot Number of Events by Type

```python
event_counts = data['event_type'].value_counts()

plt.figure(figsize=(8, 6))
sns.barplot(x=event_counts.index, y=event_counts.values, palette='rocket')

for index, value in enumerate(event_counts.values):
    plt.text(index, value + 1000, str(value), ha='center')

plt.xlabel('Event Type', fontsize=12)
plt.ylabel('Number of Events', fontsize=12)
plt.title('Number of Events by Type', fontsize=16)
plt.show()
```

### 7. Create a Summary DataFrame

```python
new_data = data[['user_id','user_session','funnel_number','index_in_funnel']].copy()
new_data['num_of_events'] = data.groupby('user_session')['event_type'].transform('count')
new_data['visit_duration'] = data.groupby(['funnel_number', 'user_session'])[['duration_to_next_event']].transform('sum')

events_by_session = data.groupby(['user_id', 'user_session']).apply(
    lambda x: pd.Series({
        'list_of_viewed': list(x.loc[x['event_type'] == 'view', 'product_id']),
        'list_of_added_to_cart': list(x.loc[x['event_type'] == 'cart', 'product_id']),
        'list_of_purchased': list(x.loc[x['event_type'] == 'purchase', 'product_id'])
    })
).reset_index()

session_data = pd.merge(new_data, events_by_session, how='left', on=['user_id', 'user_session'])
session_data.head()
```

## Summary

This project processes the raw data to extract meaningful insights. The steps include cleaning the data, calculating the duration between events, assigning funnel numbers, plotting event types, and summarizing the sessions. The final result is a comprehensive DataFrame that provides a detailed view of each user session, including the list of viewed, added to cart, and purchased products.

## Prerequisites

Make sure you have the following libraries installed:
- pandas
- numpy
- seaborn
- matplotlib

You can install them using pip:
```bash
pip install pandas numpy seaborn matplotlib
```
