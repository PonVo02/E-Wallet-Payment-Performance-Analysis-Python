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
	- Action: keep as “Unknown” or leave missing (no hard drop unless needed)
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
- **Fields:** `transaction_id`, `merchant_id`, `volume`, `transType`, `sender_id`, `receiver_id`,`transStatus`,`extra_info` ,`timeStamp`)

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

<details>

<details>
<summary>👉🏻 Show code & output (Show duplicate)</summary>

```python
df[df['transaction_id'].duplicated(keep=False)].head(10)
```
<img width="1251" height="355" alt="Ảnh màn hình 2026-03-03 lúc 13 01 23" src="https://github.com/user-attachments/assets/c1eb4287-65c1-47c6-9116-3a07b42486d4" />
<details>

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


<details>

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

