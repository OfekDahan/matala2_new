# Cosmetics Dataset Analysis

This project analyzes a dataset of user events from a cosmetics e-commerce website. The dataset includes various user interactions such as viewing products, adding products to the cart, and purchasing products. The goal is to perform data processing and analysis using `pandas` and regular expressions.

## Data Preparation and Cleaning

1. **Reading the Data:**
   The dataset is read from a CSV file into a pandas DataFrame.

   ```python
   import pandas as pd
   import numpy as np
   import re
   from datetime import datetime
   import seaborn as sns
   import matplotlib.pyplot as plt 
   import datetime as dt
   from datetime import timedelta
   
   datafile = "path_to_csv_file/matala2_cosmetics_2019-Nov.csv"
   data = pd.read_csv(datafile)

# Duration to Next Event:
A new column duration_to_next_event is added, which calculates the time in seconds until the next event for each user session. For the last event in a session, the value is set to 0.
 ```python
data['event_time'] = pd.to_datetime(data['event_time'])
data['duration_to_next_event'] = abs((data.groupby('user_session')['event_time'].shift(-1) - data['event_time']).dt.seconds.fillna(0))

# Funnel Number:
A funnel is a sequence of sessions by the same user with no more than 5 days between sessions. The funnel_number column is added to indicate the funnel to which each session belongs.
 ```python
data = data.sort_values(['user_id', 'event_time'])
data['days_diff'] = data.groupby('user_id')['event_time'].diff().dt.days
new = data['days_diff'] > 5
data['funnel_number'] = new.groupby(data['user_id']).cumsum() + 1
data['funnel_number'] = data['funnel_number'].fillna(0)

# Index in Funnel:
The index_in_funnel column is added to indicate the session number within each funnel for a user.
 ```python
data = data.sort_values(['user_id', 'funnel_number', 'event_time'])
data['temp'] = data.groupby(['user_id', 'funnel_number', 'user_session'])['user_session'].shift().ne(0).astype(int)
data['index_in_funnel'] = data.groupby(['user_id' , 'funnel_number'])['temp'].cumsum()
data = data.drop(columns=['temp'])

# Price Cleaning:
The price column is cleaned to remove any non-numeric characters and convert the values to floats using regular expressions.
data['price'] = data['price'].apply(lambda x: float(re.findall(r'\d+\.\d+', x)[0]) if isinstance(x, str) else x)

# Analysis and Visualization
Event Types Distribution:
A bar plot is created to visualize the distribution of different event types.
event_counts = data['event_type'].value_counts()

plt.figure(figsize=(8, 6))
sns.barplot(x=event_counts.index, y=event_counts.values, palette='rocket')

for index, value in enumerate(event_counts.values):
    plt.text(index, value + 1000, str(value), ha='center')

plt.xlabel('Event Type', fontsize=12)
plt.ylabel('Number of Events', fontsize=12)
plt.title('Number of Events by Type', fontsize=16)
plt.show()

# Session-Level Data Aggregation
Session Data Aggregation:
A new DataFrame session_data is created where each row represents a session and includes:

user_id
user_session
funnel_number
index_in_funnel
Number of events in the session
Duration of the session
List of viewed products
List of products added to cart
List of purchased products
new_data = data[['user_id', 'user_session', 'funnel_number', 'index_in_funnel']].copy()
new_data['num_of_events'] = data.groupby('user_session')[['event_type']].transform('count')
new_data['visit_duration'] = data.groupby(['funnel_number', 'user_session'])[["duration_to_next_event"]].transform('sum')

events_by_session = data.groupby(['user_id', 'user_session']).apply(
    lambda x: pd.Series({
        'list_of_viewed': list(x.loc[x['event_type'] == 'view', 'product_id']),
        'list_of_added_to_cart': list(x.loc[x['event_type'] == 'cart', 'product_id']),
        'list_of_purchased': list(x.loc[x['event_type'] == 'purchase', 'product_id'])
    })
).reset_index()

session_data = pd.merge(new_data, events_by_session , how = 'left', on = ['user_id', 'user_session'])

# Conclusion
This project demonstrates the use of pandas for data manipulation and regular expressions for data cleaning.
It includes tasks such as calculating the duration to the next event, identifying user funnels, indexing sessions within funnels, cleaning price data, visualizing event types, and aggregating session-level data. 
The resulting session_data DataFrame provides a comprehensive view of user sessions and their interactions with the e-commerce platform.
