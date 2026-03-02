# E-Wallet Payment Performance Analysis (Python)
Author: Vo Van Pon

Date: ... 2025

Tools Used: Python

##Table of Contents

## Overview
### Objectives 
This project analyzes e-wallet transaction and payment data to:
- Track transaction/payment trends over time.
- Surface anomalies in product ownership and team-level performance.
- Identify key drivers behind refunds and transaction volume.
- Classify transactions into meaningful categories to support deeper analysis.

### 👤 Who this is for
- Data Analysts / Business Analysts who need a practical fintech analytics case.
- Fintech stakeholders looking to monitor payment performance.
- Product teams aiming to optimize e-wallet flows and reduce refund-related issues.

## Dataset Description & Data Structure
📌 Data Source
- Source: Internal e-wallet transaction records 
- Format: `CSV`

📏 Dataset Size
- `product.csv`: 493 rows × 3 columns
- `payment_report.csv`: 920 rows × 5 columns
- `transactions.csv`: 1,324,002 rows × 9 columns
---

## 📊 Data Structure & Relationships
1) Tables Used

This project uses three datasets:
- `product.csv`: Product metadata (category, owning team).
- `payment_report.csv`: Monthly payment volumes by payment group and source.
- `transactions.csv`: Detailed transaction-level records.

Key Relationships
	•	**product_id** connects 'product.csv' <-> 'payment_report.csv'.
	•	**transaction_id** is the unique identifier in `transactions.csv` used to track individual transactions.
	•	**source_id** in `payment_report.csv` is used to analyze payment sources, especially in refund-related breakdowns.
  
## 🧾 Table Schemas & Snapshots

### Table 1 — Products (`product.csv`)
**493 rows × 3 columns**

| Column | Type | Description |
|---|---|---|
| `product_id` | INT | Unique identifier for each product |
| `category` | TEXT | Product category |
| `team_own` | TEXT | Team responsible for the product |



### Table 2 — Payment Report (`payment_report.csv`)
**920 rows × 5 columns**

| Column | Type | Description |
|---|---|---|
| `report_month` | DATE | Month of the report |
| `payment_group` | TEXT | Payment type (e.g., purchase, refund) |
| `product_id` | INT | Associated product ID |
| `source_id` | INT | Transaction source identifier |
| `volume` | FLOAT | Total payment volume |



### Table 3 — Transactions (`transactions.csv`)
**1,324,002 rows × 9 columns**

| Column | Type | Description |
|---|---|---|
| `transaction_id` | INT | Unique transaction identifier |
| `merchant_id` | INT | Merchant involved in the transaction |
| `volume` | FLOAT | Transaction amount |
| `transType` | INT | Transaction type code |
| `transStatus` | TEXT | Transaction status (e.g., completed, failed) |
| `sender_id` | INT | Sender account ID |
| `receiver_id` | INT | Receiver account ID |
| `extra_info` | TEXT | Additional transaction details |
| `timeStamp` | TIMESTAMP | Transaction timestamp |

---
## 🗝️ Main Process
## 1️⃣. Dataset payment_enriched
- The `payment_enriched` dataset is created by merging:
  - `payment_report.csv` : monthly payment volume by product
  - `product.csv`: which contains product metadata such as (category, team ownership)
Each record represents the payment volume of a specific product in a given month and source.

**Cell Code**
```pyhton
import pandas as pd

payment = pd.read_csv('/content/payment_report.csv')
product = pd.read_csv('/content/product.csv')

payment_enriched = payment.merge(
    product,
    on = 'product_id',
    how = 'left',
    validate = 'm:1')
```


### **1.1 Overview**

Check the overall data

**Cell code**
```python
payment_enriched.head()
```
**Output**

<img width="824" height="210" alt="Ảnh màn hình 2026-03-02 lúc 13 27 14" src="https://github.com/user-attachments/assets/1cf0ed07-2776-4f57-9dab-d288974db7c2" />

---
**Cell code**
```python
payment_enriched.info()
```

**Output**

<img width="802" height="252" alt="Ảnh màn hình 2026-03-02 lúc 13 30 20" src="https://github.com/user-attachments/assets/8dbdbbf5-617e-4a8f-aa4a-d38796d970a3" />

- Observed data types:

  - **report_month:** object (string – year-month format)
  - **payment_group:** object
  - **category, team_own:** object
  - **product_id, source_id:** int64 (identifiers)
  - **volume:** int64 (numeric metric)
- Assessment:
  - All data types are consistent with their business meaning.
  - No type conversion is required at this stage.
-> Next step: No action


### 1.2 Data Validation
**Missing Data**
**Cell code**
```python
payment_enriched.isna().sum()
```
**Output**

<img width="851" height="304" alt="Ảnh màn hình 2026-03-02 lúc 13 37 29" src="https://github.com/user-attachments/assets/cf13425a-0fcb-4c42-9292-f3f90acffe22" />

---


**Cell code**
```python
payment_enriched[payment_enriched['category'].isna()][['product_id']].head()
```
**Output**

<img width="714" height="198" alt="Ảnh màn hình 2026-03-02 lúc 13 40 00" src="https://github.com/user-attachments/assets/f58de7a0-7cf2-4802-a2bb-157ceb50976d" />

---

**Cell code**
```python
(payment_enriched['category'].isna() & payment_enriched['team_own'].isna()).sum()
```
**Output**

<img width="833" height="34" alt="Ảnh màn hình 2026-03-02 lúc 13 40 45" src="https://github.com/user-attachments/assets/36641de6-a696-4f79-a262-7f59c300b02b" />

- Missing data:

  - **category:** 22 rows missing
  - **team_own:** 22 rows missing
  - Missing values occur together on the same records.
Interpretation: Because `payment_enriched` is created using a LEFT JOIN on `product_id`, these missing values likely come from `product_id` that do not exist in `product.csv` (unmatched product metadata).

-> Next step: No action


**Duplicates**

**Cell code**
```python
payment_enriched.duplicated(
    subset = ['report_month', 'product_id', 'source_id']).sum()
```

**Output**

<img width="553" height="32" alt="Ảnh màn hình 2026-03-02 lúc 14 26 47" src="https://github.com/user-attachments/assets/0596ddc5-fb5d-4703-a692-c8f3bccee0b7" />

---

**Cell code**
```python
# Inspect duplicated records
payment_enriched[
    payment_enriched.duplicated(
        subset=['report_month', 'product_id', 'source_id'],
        keep = False)
].sort_values(['product_id', 'report_month'])
```
**Output**

<img width="876" height="332" alt="Ảnh màn hình 2026-03-02 lúc 14 28 09" src="https://github.com/user-attachments/assets/6715fc0b-90cb-4c10-9a10-4c0f7cdd07ef" />

**Duplicate analysis:** `payment_enriched`
**Key checked:** (`report_month`, `product_id`, `source_id`)

- Findings:

  - 5 duplicated records detected.
  - Duplicates are mainly related to `refund` payment_group.
  - Volumes are different across duplicated rows, indicating valid business events rather than data duplication errors.
- Interpretation:

  - Multiple payment/refund records for the same product and month are possible in real payment operations.
-> Next step: No action

**Value Check**
**Cell code**
```python
payment_enriched.dtypes
```

**Output** 

<img width="842" height="309" alt="Ảnh màn hình 2026-03-02 lúc 14 42 54" src="https://github.com/user-attachments/assets/42cd893e-a6b8-4417-9dfb-65f34126572c" />

- **Incorrect data types:**

  - None detected. All data is consistent with business sense.
-> Next step: No action

**Cell code**
```python
payment_enriched['volume'].describe()
```

**Output**

<img width="767" height="338" alt="Ảnh màn hình 2026-03-02 lúc 14 45 00" src="https://github.com/user-attachments/assets/513e14bd-0fea-40df-aa9b-f5e57088c9d7" />

