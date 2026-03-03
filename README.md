# E-Wallet Payment Performance Analysis (Python)
Author: Vo Van Pon

Date: ... 2025

Tools Used: Python

## 📑 Table of Contents
- [📖 Overview](Overview)
- [📂 Dataset Description & Data Structure](Dataset-Description-&-Data-Structure)
- [🔎 Final Conclusion & Recommendations](🔎-Final-Conclusion-&-Recommendations)
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

## 📂 Dataset Description & Data Structure
📌 Data Source
- Source: Internal e-wallet transaction records 
- Format: `CSV`

📏 Dataset Size
- `product.csv`: 493 rows × 3 columns
- `payment_report.csv`: 920 rows × 5 columns
- `transactions.csv`: 1,324,002 rows × 9 columns
---

### 📊 Data Structure & Relationships
1) Tables Used

This project uses three datasets:
- `product.csv`: Product metadata (category, owning team).
- `payment_report.csv`: Monthly payment volumes by payment group and source.
- `transactions.csv`: Detailed transaction-level records.

Key Relationships
- **product_id** connects 'product.csv' <-> 'payment_report.csv'.
- **transaction_id** is the unique identifier in `transactions.csv` used to track individual transactions.
- **source_id** in `payment_report.csv` is used to analyze payment sources, especially in refund-related breakdowns.
  
## 🧾 Table Schemas & Snapshots


### Table 1 — Products (`product.csv`)

<details>
	<summary> 👉🏻 click here </summary>
	
**493 rows × 3 columns**

| Column | Type | Description |
|---|---|---|
| `product_id` | INT | Unique identifier for each product |
| `category` | TEXT | Product category |
| `team_own` | TEXT | Team responsible for the product |
</details>


### Table 2 — Payment Report (`payment_report.csv`)

<details>
	<summary> 👉🏻 click here </summary>
	
**920 rows × 5 columns**

| Column | Type | Description |
|---|---|---|
| `report_month` | DATE | Month of the report |
| `payment_group` | TEXT | Payment type (e.g., purchase, refund) |
| `product_id` | INT | Associated product ID |
| `source_id` | INT | Transaction source identifier |
| `volume` | FLOAT | Total payment volume |
</details>


### Table 3 — Transactions (`transactions.csv`)

<details>
	<summary> 👉🏻 click here </summary>
	
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

</details>

---
## 🛠️ Main Process

This project is analyzed through **two complementary tracks**:

- **Track A — `payment_enriched` (Monthly Payment Performance):** combines `payment_report.csv` with product metadata from `product.csv` to analyze **payment volume by month / product / source / payment_group**.
- **Track B — `transactions` (Transaction-level Behavior):** uses `transactions.csv` to analyze **user/system transaction behavior, status outcomes, and operational patterns**.

---

### ✅ Track A — Build & Analyze `payment_enriched` (Payment Report + Product)

#### A1) Data Loading & Join 🔗
- Load `product.csv` and `payment_report.csv`
- Create `payment_enriched` using **LEFT JOIN on `product_id`**
- Ensure product metadata (`category`, `team_own`) is available for segmentation

#### A2) Data Quality Checks 🧪
- **Missing values:** check `category`, `team_own` after join
	- Interpretation: missing values likely come from `product_id` not found in `product.csv` (unmatched metadata)
	- Action: keep as “Unknown” or leave missing
- **Duplicates:** check key (`report_month`, `product_id`, `source_id`)  
  - If duplicates appear mostly in `payment_group` = refund and `volumes` differ -> treat as **valid business events**, not data errors

- 🔎 Data Validation

	- Verified `report_month` is a proper datetime field with no invalid or missing months.

	- Checked `volume` is numeric and contains no negative or unrealistic values.
<details>
	<summary>👉🏻 Show value missing </summary>
<img width="294" height="307" alt="Ảnh màn hình 2026-03-02 lúc 16 12 53" src="https://github.com/user-attachments/assets/6aec230d-e796-4b90-8fc1-aae6d1571481" />
</details>

<details>
	<summary>👉🏻 Show Duplicates </summary>
<img width="1008" height="337" alt="Ảnh màn hình 2026-03-02 lúc 16 25 06" src="https://github.com/user-attachments/assets/da691bcf-2cb4-4119-9078-961228e7b4fa" />  
</details>


<details>
	<summary>👉🏻 Show describe "volume" </summary>
<img width="550" height="334" alt="Ảnh màn hình 2026-03-02 lúc 16 37 06" src="https://github.com/user-attachments/assets/9292f94d-aa5a-4c5a-92e2-73d714709fa4" />
</details>


---

### ✅ Track B — Analyze `transactions` Dataset (Transaction-level)

**📌 Dataset Overview (`transactions.csv`)**
- **Size:** 1,324,002 rows × 9 columns
- **Analytical grain:** 1 row = 1 transaction
- **Fields:** `transaction_id`, `merchant_id`, `volume`, `transType`, `sender_id`, `receiver_id`,`transStatus`,`extra_info` ,`timeStamp`

#### B1) Data Loading & Preparation 🧹

- Load `transactions.csv`
- Convert `timeStamp` from milliseconds to datetime for time-based analysis (trend, peak time)

<details>
<summary>👉🏻 Show code & output (timestamp conversion + sample rows)</summary>

```python
df["timeStamp"] = pd.to_datetime(df["timeStamp"], unit="ms")
df.head()
```
<img width="1136" height="203" alt="Ảnh màn hình 2026-03-02 lúc 16 45 47" src="https://github.com/user-attachments/assets/c4ca1bfb-992b-479a-a713-e1a91c8c5bf2" />

</details>

#### B2) Data Quality Checks 🧪
**1️⃣ Missing Values Validation** 

**📍Observation**

<details>
<summary>👉🏻 Output (Missing Data)</summary>

<img width="799" height="363" alt="Ảnh màn hình 2026-03-03 lúc 12 20 42" src="https://github.com/user-attachments/assets/60ebcc00-d8b1-4b54-a6f1-f7dcff4c7403" />

</details>

Missing values were detected in:
- `sender_id`
-  `receiver_id`
	- Missing `sender_id` or `receiver_id` is expected in system-related transactions such as top-ups, refunds, or cashback.
	- `extra_info` has a high number of missing values and does not affect core analysis, so it is ignored.
  
To determine whether this is a data quality issue or a system-defined behavior, we analyzed missing patterns by **transaction type**.

**📍Evidence**

<details>
<summary>👉🏻 Show code & output (Transaction Type)</summary>

```python
df['transType'].value_counts()
```
<img width="751" height="326" alt="Ảnh màn hình 2026-03-03 lúc 12 17 40" src="https://github.com/user-attachments/assets/b7a8864d-2729-4ecb-9bfc-73dec64e1eee" />
</details>

<details>
<summary>👉🏻 Show code & output (Missing by transType)</summary>

```python
df.groupby('transType')[['sender_id', 'receiver_id']].apply(lambda x: x.isna().sum())
```
<img width="613" height="293" alt="Ảnh màn hình 2026-03-03 lúc 12 24 49" src="https://github.com/user-attachments/assets/503743a6-8b23-4686-bd44-60590b03b491" />

</details>

**💡Key Patterns**:
- `receiver_id` is missing exclusively in **transType 2**
- `sender_id` is missing exclusively in **transType 22**
- **TransType 30**: has both `sender_id` and `receiver_id` missing in a consistent pattern, all sender_id are missing and a portion of recevier_id are missing

**❗️Interpretation**
- Missing IDs are strongly associated with specific transaction types rather than occurring randomly across data set
- This indicates that missing values are part of the system's transaction logic, rather than data quality errors.

**❗️Conclusion**

Missing `sender_id` and `receiver_id` are treated as structural characteristic of certain transaction types, not as data errors.

**2️⃣Duplicate Transaction ID Check**

<details>
<summary>👉🏻 Show code & output (Check duplicate)</summary>

```python
df['transaction_id'].duplicated().sum()
```
<img width="464" height="28" alt="Ảnh màn hình 2026-03-03 lúc 13 00 14" src="https://github.com/user-attachments/assets/e1e710da-8b7a-464a-8f09-c2b50eca027f" />

</details>

<details>
<summary>👉🏻 Show code & output (Show duplicate)</summary>

```python
df[df['transaction_id'].duplicated(keep=False)].head(10)
```
<img width="1251" height="355" alt="Ảnh màn hình 2026-03-03 lúc 13 01 23" src="https://github.com/user-attachments/assets/c1eb4287-65c1-47c6-9116-3a07b42486d4" />
</details>

**Finding🔎**
Inspection shows that duplicates rows share identical:
- `transaction_id`
- `volume`
- `sender_id`
- `receiver_id`
- `timeStamp`
This suggests exact duplicate records rather than legitimate repeated business events.


**3️⃣Transaction Amount Validation**

<details>
<summary>👉🏻 Show code & output (Numerical)</summary>

```python
df['volume'].describe()
```
<img width="371" height="329" alt="Ảnh màn hình 2026-03-03 lúc 19 27 23" src="https://github.com/user-attachments/assets/02c07ae4-27f3-4fbc-b11d-9be9e4e907cc" />


</details>

**📍Observation**
- **Minimum value:** 1
- **25%:** 10,000
- **Median:** 30,000
- **75%:** 100,000
- Maximum value: 78,691,480

**📌Assessment**
- No negative values detected.
- Maximum value remains within a plausible range for high-value transactions.
- Distributionn is right-skewed, which is typical for financial transaction data

---

### 📊 Summary – Transactions Dataset

#### 🔎 Data Types
All columns have appropriate data types and align with business meaning.  
No type conversion required.

---

#### 💰 Transaction Amount (`volume`)
- All values are positive.
- No unrealistic or abnormal negative values detected.

---

#### 🧩 Missing Data
- Missing `sender_id` and `receiver_id` are structurally linked to specific transaction types (system-defined behavior).
- `extra_info` is largely missing (~99%) and does not impact core analysis.

No corrective cleaning is required for missing values.

---

#### 🔁 Duplicate Transactions
- 28 duplicated `transaction_id` records detected.
- Duplicated rows are **exactly identical across all fields**.

Given the very small proportion relative to total records (1.3M+),  
and since duplicates do not alter aggregated results materially,  
they are documented but not removed for this analysis.

---

#### ✅ Final Assessment

The transactions dataset is structurally consistent and analytically reliable.

Observed data characteristics (missing IDs, minor duplicates) are either:
- System-defined transaction logic, or  
- Exact duplicated records with negligible analytical impact.

The dataset is ready for further analysis.

---


## 🧠 Analysis

### ✅ Top 3 Products by Total Revenue

**Goal** Identify the **top 3 products generating the highest total revenue** to understand which products contribute most to overall income. This insight helps prioritize **sales focus, marketing budget, and product strategy** around the highest-performing items.

**Cell code**
```python
top_3_products = (
    payment_enriched.groupby('product_id')['volume'].sum().sort_values(ascending=False).head(3)
)

print('Top 3 product_ids with the highest volume:')
print(top_3_products)
```
**Output**
<img width="449" height="108" alt="Ảnh màn hình 2026-03-03 lúc 15 10 22" src="https://github.com/user-attachments/assets/7b5eb277-58ea-4d06-9135-1281c9065442" />


### ✅ Validate Product Ownership (One Team per Product)

Confirm that each `product_id` is assigned to **exactly one** owning team (`team_own`) to ensure clear accountability and prevent overlapping ownership across teams.

**Code cell**
```python
abnormal_products = (
    payment_enriched.groupby('product_id')['team_own']
    .nunique()
    .reset_index(name='team_count')
    .query('team_count > 1'))

if abnormal_products.empty:
  print('No abnormal products found.')
else:
  print('Abnormal products found:')
  print(abnormal_products)
```

**Output**

<img width="310" height="25" alt="Ảnh màn hình 2026-03-03 lúc 15 20 57" src="https://github.com/user-attachments/assets/39de0292-e9a5-4edf-8b14-b04070570d90" />


### ✅ Team Performance Check: Lowest Volume Since Q2 2023 & Category Contribution Breakdown

**Cell code**
```python
# filter Q2.2023
payment_enriched['report_month'] = pd.to_datetime(payment_enriched['report_month'])

payment_enriched_q2 = payment_enriched[payment_enriched['report_month'] >= '2023-04-01']

#lowest pfm
team_volume = (payment_enriched_q2.groupby('team_own')['volume']
                   .sum()
                   .sort_values())

team_lowest = team_volume.index[0]
team_lowest_volume = team_volume.iloc[0]

print(f'Team with the lowest performance: {team_lowest}')
print(f'Total volume: {team_lowest_volume}')

# Category contributing the least to that team

category_contributing = (
    payment_enriched_q2[payment_enriched_q2['team_own'] == team_lowest]
    .groupby('category')['volume']
    .sum()
    .sort_values()
    .head(1))

least_category = category_contributing.index[0]
least_category_volume = category_contributing.iloc[0]

print("Least-contributing category:", least_category)
print("Category volume:", least_category_volume)

```
**Output**

<img width="561" height="76" alt="Ảnh màn hình 2026-03-03 lúc 15 42 23" src="https://github.com/user-attachments/assets/abf47929-eee6-4bd5-94ce-fcd970176c45" />



### ✅ Highest-Contributing Refund Source (source_id)
**Goal:** Measure refund contribution by `source_id` (filter `payment_group = 'refund'`) and identify the source that generates the highest total refund volume.

**Code cell**
```python
# Filter refund transactions
rf_payment_enriched = payment_enriched[payment_enriched['payment_group'] == 'refund']

# Refund volume by source_id + contribution %
rf_contribution = (
    rf_payment_enriched.groupby('source_id')['volume']
    .sum()
    .sort_values(ascending=False)
    .reset_index(name='refund_volume')
)

rf_contribution["pct_contribute"] = rf_contribution['refund_volume'] / rf_contribution['refund_volume'].sum() * 100

top_source_id = rf_contribution.iloc[0]['source_id']
top_pct = rf_contribution.iloc[0]['pct_contribute']

print("Top refund source_id:", top_source_id)
print("Contribution (%):", top_pct)

rf_contribution

```

**Output**

<img width="419" height="173" alt="Ảnh màn hình 2026-03-03 lúc 15 54 19" src="https://github.com/user-attachments/assets/75d6f135-dd61-49ba-a670-4a034f868679" />



### ✅ Define type of transactions ('transaction_type') for each row, given:
- transType = 2 & merchant_id = 1205: Bank Transfer Transaction
- transType = 2 & merchant_id = 2260: Withdraw Money Transaction
- transType = 2 & merchant_id = 2270: Top Up Money Transaction
- transType = 2 & others merchant_id: Payment Transaction
- transType = 8, merchant_id = 2250: Transfer Money Transaction
- transType = 8 & others merchant_id: Split Bill Transaction
- Remained cases are invalid transactions

**Goal:** Create a new `transaction_type` field to classify each transaction based on `transType` and `merchant_id`. This standardizes raw system codes into clear business labels, enabling cleaner analysis by transaction type, easier pattern detection, and flagging invalid/unsupported transactions for data quality checks.

**Code cell**
```python
 #tranTpe = 2 - Bank Transfer - Withdraw Money - Top Up Money
def transaction_type_m(x):
  if x['transType'] == 2:
    if x['merchant_id'] == 1205:
      return 'Bank Transfer'
    elif x['merchant_id'] == 2260:
      return 'Withdraw Money'
    elif x['merchant_id'] == 2270:
      return 'Top Up Money'
    else:
      return 'Payment'
  # trantype = 8 - Tranfers Money - Split Bill
  elif x['transType'] == 8:
    if x['merchant_id'] == 2250:
      return 'Transfer Money'
    else:
       return 'Split Bill'

  # remain
  else:
    return ' Invalid'

df['transaction_type'] = df.apply(transaction_type_m, axis=1)
df['transaction_type'].value_counts()
```

**Output**

<img width="346" height="325" alt="Ảnh màn hình 2026-03-03 lúc 18 13 10" src="https://github.com/user-attachments/assets/9a3db562-ecbd-4e38-bb97-71f28aaefebc" />

### ✅ Performance Metrics by Transaction Type(Invalid).
Summarize each valid transaction_type by:

- Total number of transactions

- Total transaction volume

- Number of unique senders

- Number of unique receivers

This provides a clear comparison of activity and impact across transaction types.

**Code Cell**
```python
# filter valid transactions
df_valid = df[df['transaction_type'] != 'Invalid']

# senders and receivers

senders_receivers = (
    df_valid.groupby('transaction_type').agg(
        num_trans = ('transaction_id', 'count'),
        total_volume = ('volume', 'sum'),
        num_senders = ('sender_id', 'nunique'),
        num_receivers = ('receiver_id','nunique')
    )
).sort_values('num_trans', ascending=False)

senders_receivers
```

**Output**

<img width="740" height="291" alt="Ảnh màn hình 2026-03-03 lúc 18 34 19" src="https://github.com/user-attachments/assets/62d426e1-0e5b-4998-b451-9c7234756597" />

## 🔎 Final Conclusion & Recommendations


