# Online Retail Store Analysis

## Project Overview
This project involved a Exploratory Data Analysis at an online retail company, helping interpret real-world data to help make a key business decision.

This project is done on Jupyter Notebook (`omedina_online_retail.ipynb`) uploaded to this repository.

## Objective

- Describe data to answer key questions to uncover insights

- Gain valuable insights that will help improve online retail performance

- Provide analytic insights and data-driven recommendations

## Dataset

The dataset contains all the transactional data of an **UK-based** online retail store from 2010 to 2011. The dataset is included as `Online Retail.xlsx` uploaded to this repository. 

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

Original Count: **541909**

Count after dealing with negative values: **530104**

Count after dealing with missing values: **397884**

Count after dealing with duplicates rows: **392692**

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

We proceed to remove outliers below 1st percentile and above 99th percentile. Dataframe count reduced from **392692** to **382280**. 

```python
no_outliers_df[['Quantity', 'UnitPrice']].boxplot()
plt.title("Box Plot of Numerical Columns")
plt.show()
```
![image](https://github.com/user-attachments/assets/dd344747-be85-4b48-a23d-bd8a77884589)

Boxplot now shows very close values for `Quantity` and `Unitprice`.

## 5. Data Formating
```python
df1 = no_outliers_df.drop(['StockCode'], axis=1) # Drop Irrelevant "StockCode" column
df1 = df1.rename(columns={'Description': 'Product_Description'}) # Rename "Description" column
df1['TotalSales'] = df1['Quantity'] * df1['UnitPrice'] # Calculating TotalSales = Quantity * UnitPrice
df1['Month_Name'] = df1['InvoiceDate'].dt.month_name() # Adding "Month_Name" column
df1['CustomerID'] = df1['CustomerID'].astype(int) # Converting "CustomerID" column to integer
df1.head()
```
![image](https://github.com/user-attachments/assets/5f6d719d-34b7-4682-9a38-c86dc58965d1)

As we have `Product_Description`, we do not need `StockCode` (Product Code), we then add `TotalSales` and `Month_Name` columns and convert `CustomerID` values to integer. Now we have a clean Dataframe ready for analysis.

## 6a. Analizing the Data (Sales Summary)
```python
total_sales = df1["TotalSales"].sum() # Total Sales
total_transactions = df1["InvoiceNo"].nunique() # Total number of transactions
avg_order_value = total_sales / total_transactions # Average order value
total_unique_customers = df1["CustomerID"].nunique() # Total number of unique Customers

print(f"Total Sales: ${total_sales:,.2f}")
print(f"Total Transactions: {total_transactions}")
print(f"Average Order Value: ${avg_order_value:,.2f}")
print(f"Total Unique Customers: {total_unique_customers}")
```
Total Sales: **$6,883,273.81**

Total Transactions: **17972**

Average Order Value: **$383.00**

Total Unique Customers: **4290**

## 6b. Analizing the Data (Customer Analysis)
```python
# Top 10 customers by total sales
top_countries = df1.groupby('CustomerID')['TotalSales'].sum().nlargest(10)

# Create plot
plt.figure(figsize=(10, 5))
ax = top_countries.plot(kind='bar', color='lightcoral', edgecolor='black')

# Add labels and title
plt.xlabel('CustomerID', fontsize=12)
plt.ylabel('Total Sales', fontsize=12)
plt.title('Top 10 Customers by Total Sales', fontsize=14)
plt.xticks(rotation=45, ha='right')

# Format Y-axis to show normal numbers (no scientific notation)
ax.yaxis.set_major_formatter(mticker.StrMethodFormatter('{x:,.0f}'))  # Adds commas for large numbers

# Add grid
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.show()
```
![image](https://github.com/user-attachments/assets/9541194b-9b45-42e3-b2a6-bc93ad0c082a)

## 6c. Analizing the Data (Best-Selling Products)
```python
# Top 10 Selling Products by Quantity Sold
top_products = df1.groupby('Product_Description')['Quantity'].sum().nlargest(10)

# Create plot
plt.figure(figsize=(10, 5))
ax = top_products.plot(kind='bar', color='lightcoral', edgecolor='black')

# Add labels and title
plt.xlabel('Product Description', fontsize=12)
plt.ylabel('Quantity Sold', fontsize=12)
plt.title('Top 10 Best-Selling Products', fontsize=14)
plt.xticks(rotation=45, ha='right')

# Format Y-axis to show normal numbers (no scientific notation)
ax.yaxis.set_major_formatter(mticker.StrMethodFormatter('{x:,.0f}'))  # Adds commas for large numbers

# Add grid
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.show()
```
![image](https://github.com/user-attachments/assets/5f3cb5e5-b784-4aef-87cd-f34218cd3780)

## 6d. Analizing the Data (Sales by Country)
```python
# Top 10 countries by total sales
top_countries = df1.groupby('Country')['TotalSales'].sum().nlargest(10)

# Create plot
plt.figure(figsize=(10, 5))
ax = top_countries.plot(kind='bar', color='lightcoral', edgecolor='black')

# Add labels and title
plt.xlabel('Country', fontsize=12)
plt.ylabel('Total Sales', fontsize=12)
plt.title('Top 10 Countries by Total Sales', fontsize=14)
plt.xticks(rotation=45, ha='right')

# Format Y-axis to show normal numbers (no scientific notation)
ax.yaxis.set_major_formatter(mticker.StrMethodFormatter('{x:,.0f}'))  # Adds commas for large numbers

# Add grid
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.show()
```
![image](https://github.com/user-attachments/assets/61f23de0-2123-415d-8c75-f6582340dbcc)

## 6e. Analizing the Data (Sales Trend over Time)
```python
# Top 10 countries by total sales
top_countries = df1.groupby('Month_Name')['TotalSales'].sum().nlargest(12)

# Create plot
plt.figure(figsize=(10, 5))
ax = top_countries.plot(kind='bar', color='lightcoral', edgecolor='black')

# Add labels and title
plt.xlabel('Month', fontsize=12)
plt.ylabel('Total Sales', fontsize=12)
plt.title('Sales Trend Over Time', fontsize=14)
plt.xticks(rotation=45, ha='right')

# Format Y-axis to show normal numbers (no scientific notation)
ax.yaxis.set_major_formatter(mticker.StrMethodFormatter('{x:,.0f}'))  # Adds commas for large numbers

# Add grid
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.show()
```
![image](https://github.com/user-attachments/assets/a5c99a59-611d-4e25-a3f9-cdc9a0c924cb)

## 7. Findings and Recommendations

### 1. Sales Summary
- ✅ `Total Revenue ($6,883,273.81):` The company has generated significant revenue.
- ✅ `Average Order Value (AOV) ($383.00):` Shows how much customers typically spend per transaction.
- ✅ `Total Transactions (17972):` Helps measure sales activity and business performance.

📌 **Recommendations:**
 - If AOV is low, consider upselling or bundling products.
 - Increasing transaction volume can improve overall revenue.

### 2. Customer Analysis
- ✅ `Total Unique Customers (4290):` Helps understand customer base size.
- ✅ `Top 10 Customers:` A small percentage of customers that contribute to a large portion of sales.

📌 **Recommendations:**
 - Focus on customer retention by offering loyalty programs or discounts.
 - Identify high-value customers for personalized marketing.

### 3. Product Performance
- ✅ `Best-Selling Products:` Products that drive most of the revenue.
- ✅ `Least-Selling Products:` These may need discounts or promotions.

📌 **Recommendations:**
 - Stock up on best-selling products to avoid stockouts.
 - Consider removing or repackaging slow-moving items.

### 4. Geographic Analysis
- ✅ `Top Countries by Sales:` Some regions outperform others, in this case UK.
- ✅ `Low-Sales Regions:` Need targeted marketing or expansion efforts.

📌 **Recommendations:**
 - Invest more in high-performing markets.
 - Improve marketing in underperforming regions to boost sales.

### 5. Sales Trends Over Time
- ✅ `Seasonal Trends:` Sales may peak during specific months (e.g., holidays). November on this case (Black Friday)
- ✅ `Declining Sales Periods:` Can indicate low customer demand.

📌 **Recommendations:**
- Plan marketing campaigns around high-sales seasons.
- Offer discounts or promotions during slow months to maintain revenue.
