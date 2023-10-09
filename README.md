# RFM-analysis

## **Overview**

This project on RFM (Recency, Frequency, Monetary) analysis aims to show how customer behavior can be separated based on customer segmentation. It groups customers based on their transaction history – how recently, how often, and how much did they buy. This concept will aim to divide customers into different segments, like high-value customers, medium-value customers, or low-value customers. 

In RFM Data Analysis, the definitions for each of the analyses are: 

- **Recency**: _How recently has the customer made a transaction?_
+ **Frequency**: _How frequent is the customer in ordering/buying some product?_
* **Monetary**: _How much does the customer spend on purchasing products?_

RFM Analysis becomes highly useful in understanding the responsiveness of the customers of a particular product and provides segmentation for driven database marketing.


## **Data Preprocessing**

In order to conduct RFM analysis using Python and to segment the customers accordingly, Python libraries such as pandas, numpy, and seaborn were used for the data visualization and preprocessing - _just to name a few_. 
These libraries provide the necessary utilities/ resources for conducting the RFM Analysis. 

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

Additionally, the Data Preprocessing steps involved converting the purchase date column to datetime, dropping null rows, and checking for duplicate Order IDs within the dataset as duplicate order IDs would represent redundancies within our dataset. These are steps that are taken towards getting the data ready for the RFM analysis process. 

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

For the new columns that were formed in the copy of the data (RFM_data), the columns named were also renamed so that the Purchase Date is called Recency_value, OrderID is called Frequency_Value, and TranscationAmount is called Monetary_Value. These columns will be used later on for the RFM scores calculation and segmenation process. 

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
Lastly, the lists were converted to integers for the RFM Data Segmentation Process. This allows for the values to be added to generate the final RFM Score per customer. 

```python
# Converting RFM scores to a numeric type
RFM_data['RecencyScore'] = RFM_data['RecencyScore'].astype(int)
RFM_data['FrequencyScore'] = RFM_data['FrequencyScore'].astype(int)
RFM_data['MonetaryScore'] = RFM_data['MonetaryScore'].astype(int)
```

## **RFM Value Segmentation**

This process involved calculating the sum of the RFM Values (Recency, Frequency, and Monetary) Values, categorizing the segment into Low-value, Mid-Value, and High-Value labels, and creating the value segment based on the RFM Score.
By knowing the sum of the RFM Values, the customers can be efficiently segmented. 

```python
# Calculate the RFM score by combining the individual scores
RFM_data['RFM_Score'] = RFM_data[['RecencyScore', 'FrequencyScore', 'MonetaryScore']].sum(axis=1)

# Define segment labels
segment_labels = ['Low-Value', 'Mid-Value', 'High-Value']

# Create the Value Segment based on the RFM score
RFM_data['Value Segment'] = pd.qcut(RFM_data['RFM_Score'], q=3, labels=segment_labels)
```

## **RFM Customer Segments**

This process involves segmenting the customers based on their RFM Values. It creates a new column called RFM Customer Segments based on their RecencyScore, their FrequencyScore, and the MonetaryScore. 
The criteria for segmentation is that once the RFM_Score is 9 and above, those are referred to as _Champions_. Between 6 to 9 are referred to as _Potential Loyalists_, between 5 to 6 are at Risk Customers, between 4 to 5 are _Can't Lose customers_ and 3 to 4 are referred to as _Lost_. Through this process, we can be sure as to which customers are interested in the products the company sells and which customers are not interested. 


```python
# Defining the conditions and corresponding segments
conditions = [
    (RFM_data['RFM_Score'] >= 9),
    ((RFM_data['RFM_Score'] >= 6) & (RFM_data['RFM_Score'] < 9)),
    ((RFM_data['RFM_Score'] >= 5) & (RFM_data['RFM_Score'] < 6)),
    ((RFM_data['RFM_Score'] >= 4) & (RFM_data['RFM_Score'] < 5)),
    ((RFM_data['RFM_Score'] >= 3) & (RFM_data['RFM_Score'] < 4))
]

segments = [
    'Champions',
    'Potential Loyalists',
    'At Risk Customers',
    "Can't Lose",
    "Lost"
]

# Creating a new column for RFM Customer Segments using np.select
RFM_data['RFM Customer Segments'] = np.select(conditions, segments, default='Unknown')

# Printing the updated data with RFM segments
RFM_data.head()
```


## **RFM Analysis**

This final segment of the project involves analyzing the distributions using various data visualization techniques. It examines customers across different RFM customer segments within each value segment using a heatmap. It also examines the distribution of RFM values within the Champions Segment using Summary statistics in Python. Additionally, it examines the number of customers within all the segments using a barplot and examines the recency, frequency, and monetary scores of all the segments using a grouped bar chart. 

The code samples are provided below which examine how the RFM Analysis Charts were created respective to how they were introduced above: 


```python
# Analyzing the distribution of customers across different RFM customer segments within each value segment

# Pivoting the data to create a matrix suitable for the heatmap
heatmap_data = RFM_data.groupby(['Value Segment', 'RFM Customer Segments']).size().unstack()

# Creating the heatmap
plt.figure(figsize=(10, 6))
sns.heatmap(heatmap_data, annot=True, fmt='d', cmap='YlGnBu', cbar=True)
plt.title('Distribution of Customers Across RFM Segments Within Each Value Segment')
plt.xlabel('RFM Customer Segments')
plt.ylabel('Value Segment')

plt.show()
```



```python
# The distribution of RFM values within the Champions segment

champions_data = RFM_data[RFM_data['RFM Customer Segments'] == 'Champions']

# Summary statistics for Recency, Frequency, and Monetary values
recency_stats = champions_data['Recency_value'].describe()
frequency_stats = champions_data['Frequency_value'].describe()
monetary_stats = champions_data['Monetary_value'].describe()

print(recency_stats)
print(frequency_stats)
print(monetary_stats)
```




```python
# The number of customers in all of the segments:

# Calculating the segment counts
segment_counts = RFM_data['RFM Customer Segments'].value_counts().reset_index()
segment_counts.columns = ['RFM Customer Segments', 'Count']

# Sorting the segments by count in descending order
segment_counts = segment_counts.sort_values(by='Count', ascending=False)

# Creating the bar chart
plt.figure(figsize=(10, 6))
sns.barplot(x='RFM Customer Segments', y='Count', data=segment_counts, palette='viridis')
plt.title('Number of Customers in Each RFM Customer Segment')
plt.xlabel('RFM Customer Segments')
plt.ylabel('Number of Customers')
plt.xticks(rotation=45, ha='right')  # Rotating x-axis labels for readability

plt.show()
```



```python
# To examine the recency, frequency, and monetary scores of all the segments

# Calculating the average Recency, Frequency, and Monetary scores for each segment
segment_scores = RFM_data.groupby('RFM Customer Segments')[['RecencyScore', 'FrequencyScore', 'MonetaryScore']].mean().reset_index()


# Creating a grouped bar chart to compare segment scores
fig = go.Figure()

# Adding bars for Recency score
fig.add_trace(go.Bar(
    x=segment_scores['RFM Customer Segments'],
    y=segment_scores['RecencyScore'],
    name='Recency Score',
    marker_color='rgb(158,202,225)'
))

# Adding bars for Frequency score
fig.add_trace(go.Bar(
    x=segment_scores['RFM Customer Segments'],
    y=segment_scores['FrequencyScore'],
    name='Frequency Score',
    marker_color='rgb(94,158,217)'
))

# Adding bars for Monetary score
fig.add_trace(go.Bar(
    x=segment_scores['RFM Customer Segments'],
    y=segment_scores['MonetaryScore'],
    name='Monetary Score',
    marker_color='rgb(32,102,148)'
))

# Updating the layout
fig.update_layout(
    title='Comparison of RFM Segments based on Recency, Frequency, and Monetary Scores',
    xaxis_title='RFM Segments',
    yaxis_title='Score',
    barmode='group',
    showlegend=True
)

fig.show()
```









