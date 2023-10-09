# RFM-analysis

## **Overview**

This project on RFM (Recency, Frequency, Monetary) analysis is used to separate customer behavior based on customer segmentation. It groups customers based on their transaction history – how recently, how often, and how much did they buy. 
This concept will aim to divide customers into different segments, like high-value customers, medium-value customers, or low-value customers. 

In RFM Data Analysis, the definitions for each of the analyses are: 

- **Recency**: _How recently has the customer made a transaction?_
+ **Frequency**: _How frequent is the customer in ordering/buying some product?_
* **Monetary**: _How much does the customer spend on purchasing products?_

RFM Analysis becomes highly useful in understanding the responsiveness of the customers of a particular product and provides segmentation for driven database marketing.


## **Data Preprocessing**

In order to conduct RFM analysis using Python and to segment the customers accordingly, Python libraries such as pandas, numpy, and seaborn were used for the data visualization and preprocessing - _just to name a few_. 

```python
# importing all the necessary libraries
import pandas as pd
import numpy as np
from datetime import datetime
import matplotlib.pyplot as plt
import plotly.express as px
import plotly.graph_objects as go
import seaborn as sns

# Mounting the Google Drive to the notebook environment
drive.mount('/content/gdrive')
# reading the excel file to pandas
rfmdata = pd.read_excel(r'/content/gdrive/My Drive/RFM_Analysis/rfm_maindata.xlsx')
```

Additionally, the Data Preprocessing steps involved converting the purchase date column to datetime, dropping null rows, and checking for duplicate Order IDs within the dataset as duplicate order IDs would represent redundancies within our dataset. 

```python
# Setting the PurchaseDate column values to Datatime
rfmdata['PurchaseDate'] = pd.to_datetime(rfmdata['PurchaseDate'])
#Dropping null rows in the dataset
rfmdata.dropna(inplace = True)
# Checking for duplicate Order IDs
rfmdata.drop_duplicates(subset='OrderID', keep='first', inplace=True, ignore_index=False)

```

## **RFM Values Calculation**

The RFM Calculations involve creating a copy of the data and calculating Recency, Frequency, and Monetary Value using groupby and lambda. 
The column CustomerID was grouped by purchase date. The lambda function calculates the Recency value for each customer and calculates the number of days between the current date (obtained using datetime.now().date()) and the maximum purchase date within the group. For the OrderID, the lambda function calculates the Frequency value for each customer by calculating the number of unique OrderIDs within the group, which represents how frequently a customer has made purchases. Lastly, for the Transaction Amount, the lambda function calculates the Monetary Value for each customer and sums the transaction amount values within the group, representing the total monetary value spent by the customer.

```python
# Calculating Recency, Frequency, and Monetary Value using groupby and lambda
RFM_data = RFM_data.groupby('CustomerID').agg({
    'PurchaseDate': lambda x: (datetime.now().date() - x.max().date()).days,  # Recency Value
    'OrderID': lambda x: len(x.unique()),  # Frequency Value
    'TransactionAmount': lambda x: x.sum()  # Monetary Value
}).reset_index()
```

For the new columns that were formed in the copy of the data (RFM_data), the columns named were also renamed so that the Purchase Date is called Recency_value, OrderID is called Frequency_Value, and TranscationAmount is called Monetary_Value. 

```python
# Renaming the columns
RFM_data.rename(columns={
    'PurchaseDate': 'Recency_value',
    'OrderID': 'Frequency_value',
    'TransactionAmount': 'Monetary_value'
}, inplace=True)
```


## **RFM Scores Calculation**

In this section, the scoring criteria were defined for recency scores, frequency scores, and Monetary scores.
These scores were added to a RecencyScore list, a FrequencyScore list, and a MonetaryScore list, with values ranging from 1 to 5. The Recency Score values were derived from the Recency_value Column, the Frequency Score values from the Frequency_value column, and lastly, the MonetaryScores values from the Monetary_value column. Each was assigned a label from each of their respective lists to each customer based on which bin their RFM value falls into. 

```python
# scoring criteria for each RFM value
recency_scores =  [5,4,3,2,1]
frequency_scores =[1,2,3,4,5]
monetary_scores = [1,2,3,4,5]

# Calculating RFM scores
RFM_data['RecencyScore']= pd.cut(RFM_data['Recency_value'], bins=5, labels=recency_scores)
RFM_data['FrequencyScore'] = pd.cut(RFM_data['Frequency_value'], bins = 5, labels=frequency_scores)
RFM_data['MonetaryScore'] = pd.cut(RFM_data['Monetary_value'], bins = 5, labels=monetary_scores)
```
Lastly, the lists were converted to integers for the RFM Data Segmentation Process. 

```python
# Converting RFM scores to a numeric type
RFM_data['RecencyScore'] = RFM_data['RecencyScore'].astype(int)
RFM_data['FrequencyScore'] = RFM_data['FrequencyScore'].astype(int)
RFM_data['MonetaryScore'] = RFM_data['MonetaryScore'].astype(int)
```

## **RFM Value Segmentation**

This process involved calculating the sum of the RFM Values (Recency, Frequency, and Monetary) Values, categorizing the segment into Low-value, Mid-Value, and High-Value labels, and creating the value segment based on the RFM Score. 

```python
# Calculate the RFM score by combining the individual scores
RFM_data['RFM_Score'] = RFM_data[['RecencyScore', 'FrequencyScore', 'MonetaryScore']].sum(axis=1)

# Define segment labels
segment_labels = ['Low-Value', 'Mid-Value', 'High-Value']

# Create the Value Segment based on the RFM score
RFM_data['Value Segment'] = pd.qcut(RFM_data['RFM_Score'], q=3, labels=segment_labels)
```









