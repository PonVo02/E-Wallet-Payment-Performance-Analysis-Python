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
- **Analytical grain:** 1 row = 1 transaction event  
- **Fields:** amount (`volume`), transaction type (`transType`), sender/receiver (`sender_id`, `receiver_id`), status (`transStatus`), timestamp (`timeStamp`)

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
- Missing `sender_id` / `receiver_id`:
  - Treat as potential **system-generated transactions** (e.g., top-up, refund, cashback)  
  - Validate by checking distribution of `transType` and `transStatus` for missing-ID records
- Validate `volume`, `transStatus`, `transType` are consistent with business meaning


---


---
