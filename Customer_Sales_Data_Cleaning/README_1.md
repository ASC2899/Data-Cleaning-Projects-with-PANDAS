# 🧹 Customer Sales Data Cleaning Project

This project demonstrates an **end-to-end data cleaning workflow** on a simulated customer sales dataset.  
The dataset was intentionally generated with errors, inconsistencies, and missing values to mimic real-world messy data.

---

## 📊 Project Overview
- **Raw Data**: 10,200 simulated customer records with injected errors (missing values, inconsistent formats, outliers).
- **Goal**: Clean and standardize the dataset to make it analysis-ready.
- **Tools**: Jupyter Notebook, Python
- **Libraries**: Pandas, NumPy, Regex, Matplotlib.
- Checkout the complete [cleaning_script](Customer_Sales_Data_Cleaning/cleaning_script.ipynb).

---

## 🛠️ Data Cleaning Workflow

### 1. Initial Exploration

**Reasoning**: To understand dataset structure, numeric distributions, and missing value patterns before cleaning.

```python
# Getting overall idea of total columns and rows, not_null value counts per column, data types, memory usage.
df.info()

# Getting statistical info of all numeric columns and also finding possible oultiers.
df.describe()

# Getting total NaN counts per column.
df.isna().sum()
```

### 2. Drop Invalid Customer Records
**Reasoning**: Rows without CustomerID are ghost customers which are unusable since this is going to be primary key of the table.
```python
df = df.dropna(subset=['CustomerID'], ignore_index=True)
```

### 3. Handling NaN Values in Numeric Columns.

- Cleaning and Standardizing Age. 
  
  **Reasoning**: Extract numeric age values from messy strings and impute missing ages with the median to maintain realistic segmentation.
```python
# Creating numbers extracting function.
def extract_age(age):
    age_num = re.findall('[0-9]+', str(age))
    return age_num[0] if len(age_num) > 0 else age

# Applying the function on Age column.
df['Age'] = df['Age'].apply(lambda x: extract_age(x))

# Filling NaN with median because of ouliers.
df['Age'] = df['Age'].fillna(int(df['Age'].dropna().astype('int64').median()))
```

- Handle Missing Purchase Amounts.

  **Reasoning**: Using Median for imputation which prevents skewing analysis due to extreme outliers.
```python
df['PurchaseAmount'] = df['PurchaseAmount'].fillna(df['PurchaseAmount'].median())
```

- Handle Missing Feedback Scores.

  **Reasoning**: Since `PurchaseAmount` columns is categorical, Mode imputation ensures the most common feedback score fills gaps, preserving survey consistency.
```python
df['FeedbackScore'] = df['FeedbackScore'].fillna(df['FeedbackScore'].mode()[0])
```

### 4. Standardizing Categorical Columns.

**Reasoning**: Normalize casing and fix inconsistent labels to ensure reliable grouping and reporting.
```python
# Changing value formattings in Gender, City and Country to Title Case while also removing leading and trailing spaces.
for col in ['Gender','City','Country']:
    df[col] = df[col].str.title().str.strip()

# Replacing inconsistent values with proper values.
df['Gender'] = df['Gender'].replace({'M': 'Male', 'F':'Female'})
df['Country'] = df['Country'].replace({'Usa': 'USA', 'Uk':'UK'})
```

### 5. Handling NaN Values in Categorical Columns.


**Reasoning**: Since there is no proper way to know Gender and City, we use `'Unknown'` as a safe placeholder to avoid losing records while acknowledging missing info.
```python
df['Gender'] = df['Gender'].fillna('Unknown')
df['City'] = df['City'].fillna('Unknown')
```

**Reasoning**: we can fill country name names where city name is present and rest will be filled with Unkown.

```python
# Finding records where city name is present but not the country name.
x = df.loc[(df['City'] != 'Unknown') & (df['Country'].isnull()), ['City','Country']]

# From the records we got earlier finding unique city names.
x['City'].unique()
# Output: ['Chennai', 'Hyderabad', 'Kolkata', 'Mumbai', 'Delhi', 'Bangalore']

# Since the unique cities are from India, let's a map using City column.
city_to_country = { 'Chennai':'India',
                    'Hyderabad':'India',
                    'Kolkata':'India',
                    'Mumbai':'India',
                    'Delhi':'India',
                    'Bangalore':'India' }

# Filling country names using map()
df['Country'] = df['Country'].fillna(df['City'].map(city_to_country))
# Rest of the NaN will be filled with Unkown.
df['Country'] = df['Country'].fillna('Unknown')
```

### 6. Remove Duplicates.

**Reasoning**: Ensure each customer record is unique to avoid double-counting in analysis. We are using the primary key logic.
```python
df = df.drop_duplicates(subset=['CustomerID'], ignore_index=True)
```

### 7. Rename Columns for Consistency.

Reasoning: Use consistent naming conventions for clarity and professionalism.
```python
df = df.rename(columns={ 'CustomerID': 'Customer_ID',
                         'SignupDate': 'Signup_Date',
                         'LastPurchaseDate': 'Last_Purchase_Date',
                         'PurchaseAmount': 'Purchase_Amount',
                         'FeedbackScore': 'Feedback_Score',
                         'PhoneNumber': 'Phone_Number' })

```

### 8. Optimize Data Types.

**Reasoning**: Converting columns to appropriate types for memory efficiency and accurate analysis.

```python
# Gender, City and Country to category dtype so that each value gets stored once, thus reducing memory usage.
df = df.astype({'Gender': 'category','City': 'category','Country': 'category'})

# Age column consists of int and string values so forcing the column to int and then converting to Int64.
df['Age'] = pd.to_numeric(df['Age'], errors='coerce').astype('int64')

# Signup_Date and Signup_Date to datetime
df['Signup_Date'] = pd.to_datetime(df['Signup_Date'])
df['Last_Purchase_Date'] = pd.to_datetime(df['Last_Purchase_Date'])

```