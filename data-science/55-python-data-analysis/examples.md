# Python Data Analysis - Examples

---

## Ejemplo 1: Sales Analysis

```python
import pandas as pd
import numpy as np

# Load data
sales = pd.read_csv('sales.csv', parse_dates=['date'])

# Clean data
sales = sales.drop_duplicates()
sales['amount'] = sales['amount'].fillna(sales['amount'].median())

# Analysis
monthly_sales = (sales
    .groupby(sales['date'].dt.to_period('M'))
    .agg({
        'amount': ['sum', 'mean', 'count'],
        'customer_id': 'nunique'
    })
    .round(2)
)

# Top products
top_products = (sales
    .groupby('product')['amount']
    .sum()
    .sort_values(ascending=False)
    .head(10)
)
```

---

## Ejemplo 2: Customer Segmentation

```python
# RFM Analysis (Recency, Frequency, Monetary)
rfm = sales.groupby('customer_id').agg({
    'date': lambda x: (pd.Timestamp.now() - x.max()).days,
    'order_id': 'count',
    'amount': 'sum'
}).rename(columns={
    'date': 'recency',
    'order_id': 'frequency',
    'amount': 'monetary'
})

# Quartile-based segmentation
rfm['R_score'] = pd.qcut(rfm['recency'], 4, labels=[4,3,2,1])
rfm['F_score'] = pd.qcut(rfm['frequency'], 4, labels=[1,2,3,4])
rfm['M_score'] = pd.qcut(rfm['monetary'], 4, labels=[1,2,3,4])

rfm['RFM_score'] = (rfm['R_score'].astype(str) +
                     rfm['F_score'].astype(str) +
                     rfm['M_score'].astype(str))
```

---

**Versión:** 1.0.0
