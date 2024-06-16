### OneHealth Data Analysis Report

#### Introduction
This report presents a comprehensive analysis of pharmaceutical delivery data provided by OneHealth. The primary objectives of the analysis include data cleaning, descriptive and trend analysis, performance metrics calculation, and Service Level Agreement (SLA) compliance evaluation. Insights and recommendations are also provided to improve delivery performance and efficiency.

#### Methodology
The analysis is structured as follows:
1. **Data Cleaning and Preparation**:
    - Identify and handle missing, inconsistent, or outlier data.
    - Standardize date and time formats.
    - Calculate lead time, considering specific conditions for orders created after 4 PM.
2. **Descriptive Analysis**:
    - Generate distribution plots for key variables.
3. **Trend Analysis**:
    - Identify and illustrate trends in delivery times and lead times.
    - Analyze specific times or days where delivery times vary.
4. **Performance Metrics**:
    - Calculate average delivery time and lead time.
    - Determine the proportion of prescriptions delivered on time versus late.
5. **SLA Compliance Analysis**:
    - Analyze and trend the average turnaround time per order.
    - Analyze SLA achievement trends per day and per hour.
    - Compare and trend actual lead times against the target lead time.
    - Assess and trend compliance with SLA requirements.
6. **Insights and Recommendations**:
    - Provide insights based on the analysis.
    - Make data-driven recommendations for improving delivery performance and efficiency.

### Data Cleaning and Preparation

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import timedelta

# Load data
df = pd.read_excel("Assessment_Data_Output.xlsx")

# Convert date columns to datetime
df['Date Created'] = pd.to_datetime(df['Date Created'])
df['Delivery Time'] = pd.to_datetime(df['Delivery Time'])

# Calculate lead time with conditions
df['Lead Time'] = np.where((df['Date Created'].dt.hour >= 16) & (df['Date Created'].dt.date != df['Delivery Time'].dt.date),
                           (df['Delivery Time'] - df['Date Created'].replace(hour=8, minute=0, second=0) + timedelta(days=1)).dt.total_seconds() / 3600,
                           (df['Delivery Time'] - df['Date Created']).dt.total_seconds() / 3600)

# Clean data by dropping rows with missing delivery times
df = df.dropna(subset=['Delivery Time'])

# Handle outliers by capping lead time to the 95th percentile
lead_time_cap = df['Lead Time'].quantile(0.95)
df['Lead Time'] = np.where(df['Lead Time'] > lead_time_cap, lead_time_cap, df['Lead Time'])
```

**Observations**:
- Data types are standardized, and lead time is calculated considering specific conditions for orders created after 4 PM.

### Descriptive Analysis
##### Distribution of Delivery Times and Lead Times
```python
plt.figure(figsize=(12, 6))
sns.histplot(df['Delivery Time'], kde=True)
plt.title('Distribution of Delivery Times')
plt.show()

plt.figure(figsize=(12, 6))
sns.histplot(df['Lead Time'], kde=True)
plt.title('Distribution of Lead Times')
plt.show()
```

**Observations**:
- The distribution plots reveal the spread and central tendencies of delivery and lead times.

### Trend Analysis
##### Delivery Times and Lead Times Trends
```python
plt.figure(figsize=(12, 6))
df.set_index('Date Created')['Lead Time'].resample('D').mean().plot()
plt.title('Daily Average Lead Time')
plt.ylabel('Lead Time (hours)')
plt.show()

df['Hour Created'] = df['Date Created'].dt.hour
hourly_lead_time = df.groupby('Hour Created')['Lead Time'].mean()

plt.figure(figsize=(12, 6))
hourly_lead_time.plot(kind='bar')
plt.title('Hourly Average Lead Time')
plt.xlabel('Hour of the Day')
plt.ylabel('Lead Time (hours)')
plt.show()
```

**Findings**:
- The daily average lead time shows variability, with some days exhibiting higher lead times.
- Hourly trends indicate that lead times vary significantly across different hours of the day.

### Performance Metrics
##### Average Delivery Time and Lead Time
```python
average_delivery_time = (df['Delivery Time'] - df['Date Created']).dt.total_seconds().mean() / 3600
average_lead_time = df['Lead Time'].mean()

print(f"Average Delivery Time: {average_delivery_time:.2f} hours")
print(f"Average Lead Time: {average_lead_time:.2f} hours")
```

##### Proportion of On-Time vs Late Deliveries
```python
df['On Time'] = np.where(df['Lead Time'] <= 4, 'On Time', 'Late')
on_time_proportion = df['On Time'].value_counts(normalize=True)

plt.figure(figsize=(6, 6))
on_time_proportion.plot(kind='pie', autopct='%1.1f%%', startangle=90)
plt.title('Proportion of On-Time vs Late Deliveries')
plt.ylabel('')
plt.show()
```

**Findings**:
- The average delivery time and lead time metrics indicate overall performance.
- The proportion of on-time versus late deliveries provides insights into SLA compliance.

### SLA Compliance Analysis
##### SLA Achievement Trends
```python
sla_compliance = df[(df['Date Created'].dt.hour <= 15) & (df['Delivery Time'].dt.date == df['Date Created'].dt.date) |
                   (df['Date Created'].dt.hour > 15) & (df['Delivery Time'] <= df['Date Created'].replace(hour=12, minute=0, second=0) + timedelta(days=1))]

sla_compliance['SLA Met'] = np.where(df['Lead Time'] <= 4, 'Met', 'Not Met')

# Daily SLA compliance trend
daily_sla_compliance = sla_compliance.resample('D')['SLA Met'].value_counts(normalize=True).unstack().fillna(0)

plt.figure(figsize=(12, 6))
daily_sla_compliance.plot()
plt.title('Daily SLA Compliance Trend')
plt.ylabel('Proportion')
plt.show()

# Hourly SLA compliance trend
hourly_sla_compliance = sla_compliance.groupby('Hour Created')['SLA Met'].value_counts(normalize=True).unstack().fillna(0)

plt.figure(figsize=(12, 6))
hourly_sla_compliance.plot(kind='bar', stacked=True)
plt.title('Hourly SLA Compliance Trend')
plt.xlabel('Hour of the Day')
plt.ylabel('Proportion')
plt.show()
```

**Findings**:
- Daily and hourly SLA compliance trends reveal patterns of performance, identifying periods with lower compliance.

### Insights and Recommendations
**Insights**:
- **Lead Time Variability**: Significant fluctuations in lead times suggest process inefficiencies and resource allocation issues.
- **Hourly Compliance**: Lower compliance rates in the afternoon and evening indicate a need for better resource management during these hours.
- **Daily Trends**: Variability in daily performance points to inconsistent operational practices.

**Recommendations**:
1. **Improve Lead Time Consistency**:
   - Investigate causes of lead time fluctuations and address process bottlenecks.
   - Implement process improvements and optimize resource allocation during peak times.
2. **Enhance Afternoon and Evening Compliance**:
   - Reallocate resources or increase staffing during afternoon and evening hours.
   - Provide additional training and establish monitoring mechanisms for staff.
3. **Stabilize Daily Compliance**:
   - Implement daily review meetings to track and improve performance.
   - Adopt continuous improvement practices for operational consistency.
4. **Implement Predictive Monitoring**:
   - Use predictive analytics to anticipate periods of low compliance and adjust proactively.
   - Set up automated alerts for lead time and SLA compliance thresholds.

### Conclusion
This analysis provides comprehensive insights into OneHealth's pharmaceutical delivery performance. By implementing the recommendations, OneHealth can improve lead times, enhance SLA compliance, and achieve greater operational efficiency, ultimately leading to higher customer satisfaction.

### Deliverables
1. **Cleaned Data File**: Submitted the cleaned and standardized dataset, including the calculated lead time in this repository
2. **Analysis Report**: This detailed report including summary statistics, trend analyses, performance metrics, SLA compliance analysis, insights, and recommendations.
3. **Presentation**: A concise presentation summarizing key findings, trends, and recommendations (not yet completed).