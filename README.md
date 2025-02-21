# Online Retail Store Analysis

## Project Overview
This project involved a Exploratory Data Analysis at an online retail company, helping interpret real-world data to help make a key business decision.

This project is done on Jupyter Notebook (`omedina_online_retail.ipynb`) uploaded to this repository.

## Objective

- Describe data to answer key questions to uncover insights

- Gain valuable insights that will help improve online retail performance

- Provide analytic insights and data-driven recommendations

## Dataset

The dataset contains all the transactional data of an UK-based online retail store from 2010 to 2011. The dataset is included as `Online Retail.xlsx` uploaded to this repository. 

The dataset contains the following columns:

- **`InvoiceNo:`** Invoice number of the transaction  
- **`StockCode:`** Unique code of the product  
- **`Description:`** Description of the product  
- **`Quantity:`** Quantity of the product in the transaction  
- **`InvoiceDate:`** Date and time of the transaction  
- **`UnitPrice:`** Unit price of the product  
- **`CustomerID:`** Unique identifier of the customer  
- **`Country:`** Country where the transaction occurred

## 1. Importing Required Python Libraries
```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker  # Import ticker for formatting
```
## 2. Loading the Dataset
```python
df = pd.read_excel('Online_Retail.xlsx')
```
## 3. Data Validation

```python
df.head()
```
![image](https://github.com/user-attachments/assets/b69eed20-00bc-42fd-bb27-330395154dc0)
The `df.head()` method in Pandas show us the 5 first row of the Dataframe. Usefull to get a quick overview of a large dataset.

```python
df.info()
```
![image](https://github.com/user-attachments/assets/afdfe38a-8d7c-44c2-910b-8f65b0bd7509)

The `df.info()` method in Pandas provides a concise summary of the Dataframe, including the number of non-null values, the data types of each column, and the memory usage. It's useful for getting a quick overview of the structure and content of a dataset. In this case we see missing or null values for `Description` and `CustomerID` columns. Later we will need to drop rows with missing `CustomerID` values (It is a relevant column).

```python
df.describe()
```
![image](https://github.com/user-attachments/assets/c0383984-1320-44cd-8aae-0d4f90f7d628)

The `df.describe()` method in Pandas provides a statistical summary of the numerical columns in a Dataframe. This includes important statistics like the count, mean, standard deviation, minimum, maximum, and the quartiles (25%, 50%, 75%) of the data. It is useful to detect outliers, as we can see for `Quantity` and `UnitPrice` columns. They have negative values (they shouldn't), min and max values are a way off percentiles 25% and 75% respectively (outliers detected).

```python
df[['Quantity', 'UnitPrice']].boxplot()
plt.title("Box Plot of Numerical Columns")
plt.show()
```
![image](https://github.com/user-attachments/assets/5228fca7-f4ba-4cf9-85f2-2a353150b0cd)

A boxplot is a graphical representation used to summarize the distribution of a dataset. It is particularly useful for detecting outliers. We can see the outliers for `Quantity` and `UnitPrice`.

## 4a. Data Cleaning

```python
print(f'Original Count: {df.shape[0]}\n')

df_filtered = df[(df['Quantity'] > 0) & (df['UnitPrice'] > 0)] # Removing rows based on Quantity and UnitPrice negative values
print(f'Count after dealing with negative values: {df_filtered.shape[0]}\n')

df_filtered = df_filtered.dropna(subset=['CustomerID']) # Removing rows base on CustomerID missing values
print(f'Count after dealing with missing values: {df_filtered.shape[0]}\n')

df_filtered = df_filtered.drop_duplicates() # Removing duplicates rows
print(f'Count after dealing with duplicates rows: {df_filtered.shape[0]}\n')

df_filtered[['Quantity', 'UnitPrice']].boxplot()
plt.title("Box Plot of Numerical Columns")
plt.show()
```

Original Count: 541909

Count after dealing with negative values: 530104

Count after dealing with missing values: 397884

Count after dealing with duplicates rows: 392692

![image](https://github.com/user-attachments/assets/84c06d75-2c18-4bd9-b9da-84aac056b33d)
We proceed to remove rows with negative values in `Quantity` or `Unitprice`, we removed rows with missing `CustomerID` value and also removed duplicate rows. After cleaning the Dataframe,
the boxplot shows a good dataframe shape!

## 4b. Dealing with outliers
```python
lower_bound= df_filtered[['Quantity','UnitPrice']].quantile(0.01)
upper_bound = df_filtered[['Quantity','UnitPrice']].quantile(0.99)

no_outliers_df = df_filtered[
    (df_filtered['UnitPrice'].between(lower_bound['UnitPrice'], upper_bound['UnitPrice'])) &
    (df_filtered['Quantity'].between(lower_bound['Quantity'], upper_bound['Quantity']))
]
no_outliers_df.info()
no_outliers_df.describe()
```
![image](https://github.com/user-attachments/assets/a093b245-1be5-4419-b34a-1baa393e58dd)
We proceed to remove outliers below 1st percentile and above 99th percentile. Dataframe count 392692 reduced to 382280. Now we have a clean dataset ready for analysis.

```python
no_outliers_df[['Quantity', 'UnitPrice']].boxplot()
plt.title("Box Plot of Numerical Columns")
plt.show()
```
![image](https://github.com/user-attachments/assets/dd344747-be85-4b48-a23d-bd8a77884589)
Boxplot now shows very close values for `Quantity` and `Unitprice`.

## 4b. Data Formating
```python
df1 = no_outliers_df.drop(['StockCode'], axis=1) # Drop Irrelevant "StockCode" column
df1 = df1.rename(columns={'Description': 'Product_Description'}) # Rename "Description" column
df1['TotalSales'] = df1['Quantity'] * df1['UnitPrice'] # Calculating TotalSales = Quantity * UnitPrice
df1['Month_Name'] = df1['InvoiceDate'].dt.month_name() # Adding "Month_Name" column
df1['CustomerID'] = df1['CustomerID'].astype(int) # Converting "CustomerID" column to integer
df1.head()
```
![image](https://github.com/user-attachments/assets/5460c019-b9d9-46d6-ab74-c411adcdc2c6)

As we have `Product_Description`, we dont need `StockCode` (Product Code), We add `TotalSales` and `Month_Name` columns and convert `CustomerID` values to integer.

