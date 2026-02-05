# 📊 Azure Data Factory Incremental Ingestion Pipeline
## 🚀 Overview

A production-ready data pipeline that demonstrates enterprise data engineering practices using Azure services. This solution dynamically ingests CSV files, transforms data with business logic, and maintains data integrity with comprehensive error handling.

## 🏗️ Architecture
<p align="center">
  <img src="diagram.png" alt="Description" width="400" style="border-radius: 10px; border: 1px solid #ddd;">
</p>

### 📁 Data Model
#### 🗃️ Staging Layer (Raw Ingestion)

| Table | Purpose | Characteristics |
| :--- | :--- | :--- |
| **stg_customers** | Raw customer data | • Accepts duplicates<br>• No constraints<br>• Full history |
| **stg_products** | Raw product data | • Accepts duplicates<br>• No constraints<br>• Full history |
| **stg_orders** | Raw order transactions | • Accepts invalid FKs<br>• No constraints<br>• Full history |


### 🎯 Final Layer (Curated Data)

| Table | Type | Key Constraints | Purpose |
| :--- | :--- | :--- | :--- |
| **`customers`** | Dimension | `customer_id` (PK) | Deduplicated customer master |
| **`products`** | Dimension | `product_id` (PK) | Deduplicated product catalog |
| **`orders`** | Fact | `order_id` (PK)<br>`customer_id` (FK)<br>`product_id` (FK) | Validated transactions |

### ⚠️ Error Handling

| Table | Columns | Purpose |
| :--- | :--- | :--- |
| **`orders_error`** | • Invalid order data<br>• Error reason<br>• Source file<br>• Timestamp | Quarantined invalid records with full traceability |

## 🔄 Incremental Loading Strategy

### 📂 File-Level Processing

| Aspect | Detail |
| :--- | :--- |
| **File Pattern** | `customers_YYYY-MM-DD.csv` |
| **Processing** | Dynamic discovery via **Get Metadata** |
| **Advantage** | No hardcoded file lists |

---

### 📊 Record-Level Processing

To ensure data integrity, a `MERGE` strategy is used to prevent duplicates and maintain a clean final layer.

```sql
MERGE final_table AS target
USING staging_table AS source
ON target.business_key = source.business_key
WHEN NOT MATCHED THEN
    INSERT (...) 
    VALUES (...);
```

#### Key Features:

* **✅ Idempotent:** Safe for multiple executions; won't create duplicate records if re-run.
* **✅ Incremental:** Processes only new or changed data to save on compute costs.
* **✅ Scalable:** Optimized to handle growing data volumes efficiently.

### ⚙️ Azure Data Factory Pipeline

#### 🔧 Datasets
| Dataset | Type | Configuration | Purpose |
| :--- | :--- | :--- | :--- |
| **`InputFolderDataset`** | Azure Blob | Container path | File listing only |
| **`InputFileDataset`** | Azure Blob | Parameterized path | Read specific CSV |
| **`SqlStagingDataset`** | Azure SQL | Parameterized table | Write to staging |



