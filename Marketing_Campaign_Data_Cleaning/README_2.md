# 🧹 Marketing Campaign Data Cleaning Project

This project demonstrates an **end-to-end data cleaning workflow** on a [sourced](https://drive.google.com/drive/folders/135-0Mi2697S0-1INCyfPb03qc2EgQPvt) marketing campaign dataset.  
This project documents the cleaning and preprocessing steps applied to a raw marketing campaign dataset. Each step is explained with both the technical implementation and the **business reasoning** behind why it was necessary.


---

## 📊 Project Overview
- **Raw Data**: 2020 records of various campaign marketing with many errors (missing values, inconsistent formats, outliers).
- **Goal**: Clean and standardize the dataset to make it analysis-ready.
- **Tools**: Jupyter Notebook, Python
- **Libraries**: Pandas, NumPy
- For more details checkout the complete [cleaning script](cleaning_script.ipynb).

---

## 🛠️ Data Cleaning Workflow


### 1. Renaming Column Names

**Code:**
```python
df.columns = df.columns.str.strip().str.lower()
df = df.rename(columns={'campaignid':'campaign_id',
                        'campeigntag':'campaign_tag'})
```

**Business Reason:**
- Standardized column names reduce confusion and errors in downstream analysis.
- Lowercase and snake_case naming improves readability and ensures compatibility with SQL/BI tools.


### 2. Cleaning the `Spend` Column

**Code:**
```python
df['spend'] = df['spend'].str.replace(r'[-$, ]', '', regex=True)
df['spend'] = pd.to_numeric(df['spend'], errors='coerce')
```

**Business Reason:**
- Removing symbols like $ and , ensures numeric consistency.
- Converting to numeric allows aggregation, statistical analysis, and financial reporting.


### 3. Correcting Categorical Typos

**Code:**
```python
df['channel'] = df['channel'].replace({ 'Tik_Tok'   : 'TikTok',
                                        'Facebok'   : 'Facebook',
                                        'E-mail'    : 'Email',
                                        'Insta_gram': 'Instagram',
                                        'Gogle'     : 'Google Ads',
                                         np.nan     : 'Unknown' })

tag_map = { 'TikTok'    : 'TI',
            'Facebook'  : 'FA',
            'Email'     : 'EM',
            'Instagram' : 'IN',
            'Google Ads': 'GO',
            'Unknown'   : 'Unknown' }

df['campaign_tag'] = df['campaign_tag'].replace({
    'E-':'EM','INVALID':np.nan,'XX':'Unknown'
})
df['campaign_tag'] = df['campaign_tag'].fillna(df['channel'].map(tag_map))
```

**Business Reason:**
- Correcting typos ensures consistent grouping and reporting across marketing channels.
- Mapping campaign tags to channels prevents misclassification and improves campaign attribution.


### 4. Cleaning Mixed Boolean Values in `Active`

**Code:**
```python
df['active'] = df['active'].replace({ 'Y'    : True,
                                      '1'    : True,
                                      'Yes'  : True,
                                      'TRUE' : True,
                                      'No'   : False,
                                      '0'    : False,
                                      'FALSE': False }).astype('bool')
```

**Business Reason:**
- Standardizing boolean values ensures accurate filtering of active vs inactive campaigns.
- Prevents misinterpretation in dashboards and reporting.


### 5. Changing Data Types

**Code:**
```python
df = df.astype({ 'channel'     : 'category',
                 'campaign_tag': 'category' })

def parse_date(x):
    for fmt in ("%m/%d/%Y %H:%M", "%m/%d/%Y", "%d/%m/%Y"):
        try:
            return pd.to_datetime(x, format=fmt)
        except:
            continue
    return pd.NaT

df['start_date'] = df['start_date'].apply(parse_date)
df['end_date'] = pd.to_datetime(df['end_date'])
```

**Business Reason:**
- Converting categorical columns reduces memory usage and improves performance.
- Parsing dates ensures accurate time-based analysis (e.g., campaign duration, seasonality).


### 6. Data Integrity: `Start_Date` vs `End_Date`

**Code:**
```python
mask = df['end_date'] < df['start_date']
df.loc[mask, 'end_date'] = df.loc[mask, 'end_date'] + pd.Timedelta(days=30)
```

**Business Reason:**
- Ensures logical consistency: campaigns cannot end before they start.
- Adjusting end dates prevents misleading ROI and duration calculations.


### 7. Handling Outliers in `Spend`

**Code:**
```python
Q1 = df['spend'].quantile(0.25)
Q3 = df['spend'].quantile(0.75)
IQR = Q3 - Q1
upper_limit = Q3 + (3 * IQR)

df.loc[df['spend'] > upper_limit, 'spend'] = upper_limit
```

**Business Reason:**

- Winsorization prevents extreme values from skewing averages and trend analysis.
- Ensures more reliable budgeting and forecasting.


### 8. Feature Engineering: Extracting `Season`

**Code:**
```python
df['season'] = df['campaign_name'].apply(lambda x: x.split('_')[1])
```

**Business Reason:**

- Extracting seasonality enables performance comparison across quarters or seasonal campaigns.
- Helps marketing teams align spend with seasonal demand.









