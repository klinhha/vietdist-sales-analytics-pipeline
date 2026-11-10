# [PYTHON] VietDist Sales Analytics Pipeline

## I. Project Overview

VietDist Analytics is an **end-to-end** data warehousing and analytics engineering project designed to transform sales, customer, product, employee, distributor, sales target, and return data into a structured analytical platform.

The project uses **Python** to ingest Excel and CSV files from Google Drive into PostgreSQL, SQL to clean and transform data through a **Medallion Architecture**, and dimensional modeling to prepare analytical datasets for business reporting.

The platform aims to support sales performance monitoring, actual-versus-target analysis, employee performance evaluation, return analysis, and distributor delivery performance.

### 1. Objectives

- **Automate data ingestion** from Google Drive into PostgreSQL using Python.

- **Organize data** into Bronze, Silver, and Gold layers.

- **Clean**, **standardize**, and **validate** data before analytical use.

- **Build dimensional models** and **fact tables** for business analysis.

- **Preserve** employee history and **manage** sales target versions.

- **Prepare** analytical datasets for reporting and decision-making.

### 2. Who is this project for?

- **Aspiring Data Engineers & Analytics Engineers**: To gain hands-on experience building data pipelines and data warehouses.
- **Data Analysts & BI Developers**: To understand data transformation, dimensional modeling, and analytical data preparation.
- **Business Stakeholders & Sales Managers**: To analyze sales performance, target achievement, returns, and distributor operations.

### 3. Medallion Architecture

The project follows a three-layer architecture to separate raw data ingestion, data transformation, and business-oriented modeling.

| Layer | Purpose | Implementation |
|---|---|---|
| **Bronze** | Stores source data as ingested, preserving the original information for downstream processing. | Python loaders and PostgreSQL |
| **Silver** | Cleans and standardizes data, applies appropriate data types, handles duplicates, and validates data quality. | SQL transformations in PostgreSQL |
| **Gold** | Organizes validated data into dimensional models, fact tables, and data marts for analytical queries and reporting. | SQL-based data modeling |

### 4. Data Flow

```text
Google Drive (Excel / CSV)
           |
           v
    Python Loaders
           |
           v
   Bronze Layer
    Raw Data
           |
           v
   Silver Layer
 Cleaned & Validated
           |
           v
    Gold Layer
Dimensions / Facts / Marts
           |
           v
 Business Analysis & Reporting
```

## II. Data Modeling

The Gold layer follows dimensional modeling principles, separating descriptive dimensions from measurable business events and analytical summaries.

### Dimension Tables

<details>
<summary>View dimension tables</summary>

| Table | Description |
|---|---|
| `dim_date` | Calendar attributes for date-based analysis, including month, quarter, and year. |
| `dim_employees` | Employee attributes with historical tracking using SCD Type 2. |
| `dim_customers` | Customer profiles, classifications, regions, and credit information. |
| `dim_products` | Product attributes, categories, pricing, and cost information. |
| `dim_distributors` | Distributor profiles, tiers, channels, and historical attributes. |

</details>

### Fact Tables

<details>
<summary>View fact tables</summary>

| Table | Description |
|---|---|
| `fact_sales` | Sales transactions at the individual order-line level. |
| `fact_targets` | Monthly employee sales targets with plan version information. |
| `fact_returns` | Product return records and associated return amounts and statuses. |
| `fact_distributor_orders` | Distributor order lines, ordered and delivered quantities, delivery performance, and payment terms. |

</details>

### Analytical Data Marts

<details>
<summary>View analytical data marts</summary>

| Table | Description |
|---|---|
| `mart_sales_vs_target` | Combines actual sales and target data to support monthly performance comparisons. |
| `mart_distributor_performance` | Consolidates distributor performance metrics, including order fulfillment, fill rates, and on-time delivery. |

</details>

## III. Project Structure

The repository is organized into reusable ingestion modules, supporting configuration, documentation, and sample data.

```text
vietdist_analytics/
├── config/           # Database and connection configuration
├── docs/             # Project documentation and data definitions
├── loader/
│   ├── db_utils.py         # Handles PostgreSQL database connections and operations
│   ├── drive_utils.py      # Provides reusable functions to access and read files from Google Drive
│   ├── load_customer_master.py
│   ├── load_distributor_master.py
│   ├── load_distributor_orders.py
│   ├── load_employee_master.py
│   ├── load_product_master.py
│   ├── load_promotion_program.py
│   ├── load_return_transactions.py
│   ├── load_sales_target_auto.py
│   ├── load_sales_transaction.py
│   └── load_territory_mapping_.py
├── VietDist_SampleData/    # The source data
├── .env                    # Stores environment variables and database credentials
├── .gitignore              # Specifies files and folders Git should ignore
├── requirement.txt         # Python dependencies
└── README.md
```

## IV. Technology Stack

- **Python:** Data ingestion and reusable loading utilities.
- **Pandas & OpenPyXL:** Reading and processing CSV and Excel files.
- **PostgreSQL:** Data storage, SQL transformations, and warehouse modeling.
- **SQLAlchemy & psycopg2:** Python–PostgreSQL connectivity.
- **Google Drive API:** Accessing source data files.
- **Git & GitHub:** Version control and project documentation.
- **VS Code & pgAdmin & DBeaver:** Development and database management.

## V. Main Process
### Phase 0 - ENVIRONMENT SETUP
### 1. Create a Python Virtual Environment

```bash
# Create a virtual environment
python3 -m venv venv

# Activate the virtual environment on macOS/Linux
source venv/bin/activate

# Install all required libraries listed in requirement.txt
pip install -r requirement.txt
```

<details>
<summary>Libraries in 'requirement.txt'</summary>

| Library | Purpose |
|---|---|
| `pandas` | Reads, processes, and transforms tabular data using DataFrames. |
| `sqlalchemy` | Provides database connectivity and tools for interacting with PostgreSQL. |
| `psycopg2-binary` | PostgreSQL database adapter for Python. |
| `openpyxl` | Reads and writes Excel `.xlsx` files. |
| `google-api-python-client` | Provides Python tools for accessing Google APIs, including Google Drive. |
| `google-auth` | Handles authentication with Google services, including service account credentials. |
| `python-dotenv` | Loads environment variables from a `.env` file. |

</details>


### 2. Initialize the PostgreSQL Database


```sql
-- Create a dedicated database for the project om PostgreSQL
CREATE DATABASE postgres;

-- Create the Medallion Architecture schemas
CREATE SCHEMA bronze;
CREATE SCHEMA silver;
CREATE SCHEMA gold;

-- Verify that the schemas exist
SELECT schema_name
FROM information_schema.schemata
WHERE schema_name IN ('bronze', 'silver', 'gold')
ORDER BY schema_name;
```

### 3. Configure Environment Variables

**Create** a file named **`.env`** in the project root directory. This file stores **configuration values** and **sensitive credentials** used by the Python scripts.

```dotenv
# PostgreSQL connection settings
PG_USER='postgres'
PG_PWD='database_password'
PG_DATABASE_NAME='postgres'
PG_HOST='localhost'
PG_PORT='5432'

# Google Service Account credentials
GOOGLE_SERVICE_ACCOUNT_FILE=/absolute/path/to/service_account.json

# Google Drive source folder
GOOGLE_DRIVE_FOLDER_ID=your_google_drive_folder_id
```

Add the entries to `.gitignore`:

```gitignore
# Environment variables and credentials
.env
credentials/

# Python virtual environment and cache
venv/
__pycache__/
*.py[cod]

# macOS system files
.DS_Store
```

### Phase 1 — BRONZE LAYER: Source Connection & Ingestion

### Task 1.1 — Connect to Google Drive and Read Source Files

### 1.1.1 — Create the Google Drive Utility Module

**File:** `loader/drive_utils.py`

This module centralizes Google Drive authentication, file downloading, and file reading. Reusable functions allow individual loader scripts to access source data without duplicating the connection logic.

#### 1. Import Required Libraries and Load Environment Variables

```python
import os  # Access environment variables and operating system information

from urllib.parse import quote_plus  # Encode special characters in the PostgreSQL password for safe use in a URL

from io import BytesIO  # Store downloaded file data in memory (RAM) instead of writing it to disk

import pandas as pd  # Read and process tabular data from CSV and Excel files

from google.oauth2 import service_account  # Authenticate with Google services using a service account

from googleapiclient.discovery import build  # Build a Google Drive API service client

from googleapiclient.http import MediaIoBaseDownload  # Download files from Google Drive into an in-memory buffer

from sqlalchemy import create_engine  # Create a database engine for connecting to PostgreSQL

from dotenv import load_dotenv  # Load environment variables from the .env file

load_dotenv()
```

#### 2. Read Google Drive Configuration

```python
# Retrieve the service account JSON path and Google Drive folder ID from .env to avoid hardcoding configuration values directly into the Python code

SERVICE_ACCOUNT_FILE = os.getenv("GOOGLE_SERVICE_ACCOUNT_FILE")
FOLDER_DRIVE_ID = os.getenv("GOOGLE_DRIVE_FOLDER_ID")

# Define the service account permissions (OAuth scope)

SCOPES = ["https://www.googleapis.com/auth/drive.readonly"]      # Read-only access is sufficient because the pipeline only needs to retrieve source files
```

#### 3. Create the Google Drive Connection

```python
# Create a reusable function that authenticates the service account and returns a Google Drive API service client

def get_service_account():
    credentials = service_account.Credentials.from_service_account_file(
        SERVICE_ACCOUNT_FILE,
        scopes=SCOPES
    )

    # Connect to Google Drive API version 3
    service = build(
        "drive",
        "v3",
        credentials=credentials
    )

    return service
```

#### 4. Download a File into Memory

```python
# Download a source file from Google Drive and return it as an in-memory buffer

def download_data(service, file_id):
    request = service.files().get_media(
        fileId=file_id,
        supportsAllDrives=True
    )

    # Create a buffer in RAM to store the downloaded file
    buffer = BytesIO()       #`BytesIO` allows the downloaded file to be read directly from memory, avoiding the need to save a temporary copy to the local disk.

    # Download the file in chunks to handle larger files
    downloader = MediaIoBaseDownload(
        buffer,
        request
    )

    done = False

    while not done:
        status, done = downloader.next_chunk()

    # Move the buffer pointer back to the beginning so pandas can read the file
    buffer.seek(0)

    return buffer
```

#### 5. Read CSV and Excel Files

```python
# Read a CSV or Excel file from the in-memory buffer and return a DataFrame

def read_file(file_name, buffer):
    if file_name.endswith(".csv"):
        # Read CSV data into a pandas DataFrame
        df = pd.read_csv(buffer)

    elif file_name.endswith(".xlsx"):
        # Use openpyxl to read Excel .xlsx files
        df = pd.read_excel(
            buffer,
            engine="openpyxl"
        )

    else:
        # Raise an error if the file format is not supported
        raise ValueError(
            "File format not supported. Only CSV and .xlsx files are allowed."
        )

    return df
```

### Task 1.2 — Connect to PostgreSQL

### Create the Database Utility Module

**File:** `loader/db_utils.py`

This module centralizes PostgreSQL connection settings and provides a reusable function for loading DataFrames into database tables.

#### 1. Create a PostgreSQL Connection

```python
import os  # Retrieve database settings from environment variables

from urllib.parse import quote_plus  # Encode special characters in the database password

import pandas as pd  # Handle tabular data and DataFrame-to-SQL loading

from sqlalchemy import create_engine  # Create a PostgreSQL database engine

from dotenv import load_dotenv  # Load database settings from .env

load_dotenv()


# Create a reusable function to connect to PostgreSQL
def create_connection_postgres():

    # Retrieve database configuration from .env
    PG_USER = os.getenv("PG_USER")
    PG_PWD = os.getenv("PG_PWD")
    PG_DATABASE_NAME = os.getenv("PG_DATABASE_NAME")
    PG_HOST = os.getenv("PG_HOST")
    PG_PORT = os.getenv("PG_PORT")

    # Encode special characters, such as @, in the password for safe URL usage
    PG_PWD = quote_plus(PG_PWD)

    # Build the PostgreSQL connection string
    connection_string = (
        f"postgresql+psycopg2://{PG_USER}:{PG_PWD}"
        f"@{PG_HOST}:{PG_PORT}/{PG_DATABASE_NAME}"
    )

    # Create an SQLAlchemy engine to manage database connections
    engine = create_engine(connection_string)

    return engine
```

#### 2. Load Data into PostgreSQL

```python
# Load a pandas DataFrame into a PostgreSQL table
def load_to_postgres(
    df,
    table_name,
    schema="bronze",
    if_exists="replace"
):
    # Create the PostgreSQL engine
    engine = create_connection_postgres()

    # Write the DataFrame to the specified database table
    df.to_sql(
        name=table_name,
        con=engine,
        schema=schema,
        if_exists=if_exists,
        index=False
    )
# Before loading into bronze, create timestamp to show the ingestion time
data['recorded_at'] = pd.Timestamp.now()
```

### Task 1.3 — Load Raw Data into the Bronze Layer

#### Create a Loader Script

A sample loader workflow from **`load_sales_transaction.py`**:

```python
import pandas as pd

from drive_utils import (
    get_service_account,
    download_data,
    read_file
)

from db_utils import load_to_postgres

# Connect to Google Drive
service = get_service_account()

# Download the source file into memory
buffer = download_data(service, file_id = '1lpXfu1X3IF2_uPhO1uECSW0M1Ijucfbu')

# Read the source file into a DataFrame
data = read_file('sales_transactions.csv', buffer)

# Record the ingestion timestamp
data["recorded_at"] = pd.Timestamp.now()

# Load the data into the Bronze schema
load_to_postgres(
    data,
    table_name="Sales_Transaction",
    schema="bronze",
    if_exists="replace"
)
```

#### Verify the Bronze Table

```sql
-- Check the number of rows loaded into a Bronze table
SELECT COUNT(*)
FROM bronze.Sales_Transaction;

-- Inspect a sample of the loaded data
SELECT *
FROM bronze.Sales_Transaction
LIMIT 10;
```
- **Results**: COUNT: 119,101 rows

<details>
<summary><strong>View table Sales_Transaction</strong></summary>

| Order ID | Order Date | Customer ID | Region | Province | Sales Channel | Employee ID | Product ID | Product Category | Quantity | Unit Price | Discount (%) | Gross Amount | Net Amount | Order Status | Payment Method | Payment Status | Recorded At |
|---|---|---|---|---|---|---|---|---|---:|---:|---:|---:|---:|---|---|---|---|
| ORD0010001 | 2024-08-22 | CUS01590 | Miền Nam | TP.HCM | Modern Trade | EMP1079 | PRD0055 | Thực phẩm | 109 | 8,500 | 2 | 926,500 | 908,000 | Delivered | Công nợ | Paid | 2026-08-25 00:31:46 |
| ORD0010002 | 2024-12-28 | CUS01166 | Miền Nam | Vũng Tàu | E-commerce | EMP1092 | PRD0057 | Thực phẩm | 59 | 50,000 | 0 | 2,950,000 | 2,950,000 | Delivered | Công nợ | Paid | 2026-08-25 00:31:46 |
| ORD0010002 | 2024-12-28 | CUS01166 | Miền Nam | Vũng Tàu | E-commerce | EMP1092 | PRD0036 | Thực phẩm | 9 | 30,000 | 0 | 270,000 | 270,000 | Delivering | Tiền mặt | Paid | 2026-08-25 00:31:46 |
| ORD0010002 | 2024-12-28 | CUS01166 | Miền Nam | Vũng Tàu | E-commerce | EMP1092 | PRD0087 | Gia dụng | 7 | 15,000 | 0 | 105,000 | 105,000 | Delivering | Công nợ | Paid | 2026-08-25 00:31:46 |
| ORD0010003 | 2024-03-19 | CUS01832 | Miền Trung | Quảng Ngãi | Modern Trade | EMP1047 | PRD0069 | Chăm sóc cá nhân | 34 | 40,000 | 0 | 1,360,000 | 1,360,000 | Delivered | Công nợ | Unpaid | 2026-08-25 00:31:46 |
| ORD0010003 | 2024-03-19 | CUS01832 | Miền Trung | Quảng Ngãi | Modern Trade | EMP1047 | PRD0091 | Gia dụng | 95 | 12,000 | 10 | 1,140,000 | 1,026,000 | Delivered | Tiền mặt | Paid | 2026-08-25 00:31:46 |
| ORD0010003 | 2024-03-19 | CUS01832 | Miền Trung | Quảng Ngãi | Modern Trade | EMP1047 | PRD0002 | Thức uống | 13 | 9,000 | 5 | 117,000 | 111,200 | Delivered | Công nợ | Paid | 2026-08-25 00:31:46 |
| ORD0010004 | 2024-02-13 | CUS00345 | Miền Bắc | Hà Nội | Modern Trade | EMP1014 | PRD0056 | Thực phẩm | 64 | 10,000 | 10 | 640,000 | 576,000 | Delivered | Tiền mặt | Paid | 2026-08-25 00:31:46 |
| ORD0010005 | 2024-03-11 | CUS00564 | Miền Trung | Huế | Traditional Trade | EMP1050 | PRD0035 | Thực phẩm | 86 | 18,000 | 10 | 1,548,000 | 1,393,200 | Delivered | Chuyển khoản | Unpaid | 2026-08-25 00:31:46 |
| ORD0010006 | 2024-11-13 | CUS00573 | Miền Trung | Huế | Modern Trade | EMP1041 | PRD0024 | Thức uống | 19 | 40,000 | 0 | 760,000 | 760,000 | Delivered | Tiền mặt | Paid | 2026-08-25 00:31:46 |

</details>


### Task 1.4 — Special Handling: Multi-Version Sales Target Plans

The `sales_target_plan` file requires special handling because it may **contain multiple versions**, such as v1 and v2, either in separate files or in different sheets within the same Excel workbook.

Unlike the other source tables, sales target data must preserve its version history. All versions must be extracted and loaded without unintentionally overwriting previously ingested versions.

#### Create a Loader Script

**File:** `loader/load_sales_target_auto.py`

The existing workbook contains two sheets:

- `Plan_v1_Original` — Original sales target plan (v1).
- `Plan_v2_Adjustment_H2` — Adjusted sales target plan (v2).

```python
# Connect to Google Drive and download the source workbook
service = get_service_account()

buffer = download_data(
    service,
    file_id='1-BLii_-P0NQ4jlcG9ckbyfkZk54vo19s'
)

# Reset the buffer pointer before reading the workbook
buffer.seek(0)

# Read all sheets into a dictionary of DataFrames
sheets = pd.read_excel(
    buffer,
    sheet_name=None,          # This parameter instructs pandas to read all sheets into a dictionary of DataFrames, allowing **each version** to be processed separately.
    engine="openpyxl"
)

# Extract each version into a separate DataFrame
v1 = sheets["Plan_v1_Original"].copy()
v2 = sheets["Plan_v2_Adjustment_H2"].copy()

# Create metadata for the available sales target versions
sales_target_files = pd.DataFrame([
    {
        "version_label": "v1",
        "sheet_name": "Plan_v1_Original"
    },
    {
        "version_label": "v2",
        "sheet_name": "Plan_v2_Adjustment_H2"
    }
])

# Combine v1 and v2 into a single DataFrame
sales_targets_raw = pd.concat(
    [v1, v2],
    ignore_index=True
)

# Convert numeric month values into labels such as T1, T2, ..., T12
sales_targets_raw["month_col"] = (
    "T" + sales_targets_raw["month"].astype(str)
)

# Record the ingestion timestamp
sales_targets_raw["recorded_at"] = pd.Timestamp.now()
sales_target_files["recorded_at"] = pd.Timestamp.now()

# Load sales target version metadata
load_to_postgres(
    sales_target_files,
    "sales_target_files",
    schema="bronze",
    if_exists="replace"
)

# Load all sales target records
load_to_postgres(
    sales_targets_raw,
    "sales_targets_raw",
    schema="bronze",
    if_exists="replace"
)
```

#### Validate the Loaded Data

```sql
-- Check the available versions
SELECT DISTINCT version_label
FROM bronze.sales_target_files
ORDER BY version_label;

-- Check the months
SELECT DISTINCT month_col
FROM bronze.sales_targets_raw
ORDER BY month_col;

-- Check the number of records by version
SELECT
    plan_version,
    COUNT(*) AS row_count
FROM bronze.sales_targets_raw
GROUP BY plan_version
ORDER BY plan_version;
```
- **Result**: There are 2 available versions (v1: 1332 rows, v2: 618 rows), 12 months (T1 -> T12)

### Phase 2 - SILVER LAYER: Clean & Standardize

> **Note:** The examples below use **`Sales_Transaction`** as the sample dataset. The same profiling, transformation, and testing workflow is applied to the other Bronze tables, with adjustments based on their individual structures and business rules.

### Task 2.1 — Data Profiling

Profile all Bronze tables before applying transformations.

```sql
-- Check total rows
SELECT COUNT(*) AS total_rows
FROM bronze."Sales_Transaction";

-- Check null values in key columns
SELECT
    COUNT(*) AS total_rows,
    COUNT(*) FILTER (WHERE order_id IS NULL) AS null_order_id,
    COUNT(*) FILTER (WHERE order_date IS NULL) AS null_order_date,
    COUNT(*) FILTER (WHERE product_id IS NULL) AS null_product_id
FROM bronze."Sales_Transaction";

-- Identify potential duplicates
SELECT
    order_id,
    product_id,
    COUNT(*) AS duplicate_count
FROM bronze."Sales_Transaction"
GROUP BY order_id, product_id
HAVING COUNT(*) > 1;
```

### Task 2.2 — Build the Silver Layer with SQL

```sql
CREATE TABLE silver."SalesTransaction" AS
SELECT
    TRIM(order_id::TEXT) AS order_id,

    CAST(order_date AS DATE) AS order_date,
    CAST(order_month AS INTEGER) AS order_month,
    CAST(order_quarter AS INTEGER) AS order_quarter,
    CAST(order_year AS INTEGER) AS order_year,

    TRIM(customer_id::TEXT) AS customer_id,
    region,
    province,
    channel,
    TRIM(employee_id::TEXT) AS employee_id,
    TRIM(product_id::TEXT) AS product_id,
    TRIM(product_category::TEXT) AS product_category,

    CAST(quantity AS INTEGER) AS quantity,
    CAST(unit_price AS NUMERIC(18,2)) AS unit_price,
    CAST(discount_pct AS NUMERIC) AS discount_pct,
    CAST(discount_amount AS NUMERIC(18,2)) AS discount_amount,
    CAST(gross_amount AS NUMERIC(18,2)) AS gross_amount,
    CAST(net_amount AS NUMERIC(18,2)) AS net_amount,

    delivery_status,
    payment_method,
    payment_status,

    recorded_at
FROM bronze."Sales_Transaction";
```

### Task 2.3 — Data Quality Testing

```sql
-- Check null primary keys
SELECT COUNT(*) AS null_order_ids
FROM silver."SalesTransaction"
WHERE order_id IS NULL;

-- Check duplicate keys
SELECT order_id, COUNT(*)
FROM silver."SalesTransaction"
GROUP BY order_id
HAVING COUNT(*) > 1;

-- Check invalid values
SELECT *
FROM silver."SalesTransaction"
WHERE order_date IS NULL
   OR quantity < 0
   OR unit_price < 0;

-- Compare Bronze and Silver row counts
SELECT
    (SELECT COUNT(*)
           FROM bronze."Sales_Transaction") AS bronze_rows,
    (SELECT COUNT(*)
           FROM silver."SalesTransaction") AS silver_rows;
```

### Task 2.4 — Special Handling: Sales Target Versioning

Preserve both Sales Target versions (v1 and v2), standardize data types, and identify the effective target for each employee and month.

| Target Period | v1 – Original Plan | v2 – July Adjustment | Effective Version |
|---|---|---|---|
| T1–T6 | Original targets | No adjustment | v1 |
| T7–T12 | Original targets | Adjusted targets | v2 |

**Build the Versioned Silver Table**

```sql
-- Create the versioned Sales Target table in the Silver layer
CREATE TABLE silver."SalesTarget" AS

-- Rank target versions for each employee and month
WITH ranked_targets AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY employee_id, year, month
            ORDER BY CAST(version_date AS DATE) DESC
        ) AS version_rank  -- Rank versions by issue date (newest version = 1)

    FROM bronze."sales_targets_raw"
)

SELECT
    plan_version,  -- Original plan version (v1 or v2)

    CAST(version_date AS DATE) AS version_date,  -- Convert date columns from TEXT to DATE
    CAST(effective_from AS DATE) AS effective_from,
    CAST(effective_to AS DATE) AS effective_to,

    employee_id,
    employee_name,
    region,
    team,

    CAST(year AS INTEGER) AS year,
    CAST(month AS INTEGER) AS month,
    CAST(target_revenue AS NUMERIC(18,2)) AS target_revenue,  -- Monetary value
    CAST(target_quantity AS INTEGER) AS target_quantity,   
    CAST(target_new_customers AS INTEGER) AS target_new_customers,

    recorded_at,

    -- Flag the latest target version for each employee and month
    CASE
        WHEN version_rank = 1 THEN TRUE
        ELSE FALSE
    END AS is_latest

FROM ranked_targets;
```

**Data Quality Testing**

```sql
-- Check data types
SELECT
    column_name,
    data_type
FROM information_schema.columns
WHERE table_schema = 'silver'
  AND table_name = 'SalesTarget'
ORDER BY ordinal_position;

-- Check records across versions
SELECT
    plan_version,
    is_latest,
    COUNT(*) AS row_count
FROM silver."SalesTarget"
GROUP BY plan_version, is_latest
ORDER BY plan_version, is_latest;

-- Check for multiple latest versions per employee and month
SELECT
    employee_id,
    year,
    month,
    COUNT(*) AS latest_count
FROM silver."SalesTarget"
WHERE is_latest = TRUE
GROUP BY employee_id, year, month
HAVING COUNT(*) > 1;
```
-> **Results**:
- Date, numeric, and timestamp columns have the intended data types.
- Both v1 and v2 records are preserved.
- Each employee-month combination has **no more than** one record marked as `is_latest = TRUE`.

## PHASE 3  —  GOLD LAYER: Data Mart & Star Schema 

### Task 3.1 — Build Dimension Tables

#### 3.1.1 — Create `dim_date`

Create a static calendar dimension covering 2022–2026, including calendar attributes and fiscal year information.

**Create the table structure**

```sql
-- Create an empty date dimension table
CREATE TABLE gold.dim_date (
    date_key INTEGER PRIMARY KEY,  -- Date key in YYYYMMDD format

    full_date DATE NOT NULL,       -- Actual calendar date
    day INTEGER,                   -- Day of the month
    month INTEGER,                 -- Month number (1–12)
    month_name TEXT,               -- Month name
    quarter INTEGER,               -- Calendar quarter (1–4)
    year INTEGER,                  -- Calendar year
    fiscal_year INTEGER            -- Fiscal year (ends in August)
);
```

**Populate the calendar table**

```sql
-- Generate dates from 2022-01-01 to 2026-12-31
INSERT INTO gold.dim_date (
    date_key,

    full_date,
    day,
    month,
    month_name,
    quarter,
    year,
    fiscal_year
)
SELECT
    TO_CHAR(d, 'YYYYMMDD')::INTEGER AS date_key, -- Convert date to YYYYMMDD key
    d::DATE AS full_date,                       -- Store the full calendar date
    EXTRACT(DAY FROM d)::INTEGER AS day,        
    EXTRACT(MONTH FROM d)::INTEGER AS month,     
    TRIM(TO_CHAR(d, 'Month')) AS month_name,     
    EXTRACT(QUARTER FROM d)::INTEGER AS quarter, 
    EXTRACT(YEAR FROM d)::INTEGER AS year,       

    -- Calculate fiscal year (September–August)
    CASE
        WHEN EXTRACT(MONTH FROM d) >= 9
            THEN EXTRACT(YEAR FROM d)::INTEGER
        ELSE EXTRACT(YEAR FROM d)::INTEGER - 1
    END AS fiscal_year

FROM generate_series(
    '2022-01-01'::DATE,
    '2026-12-31'::DATE,
    '1 day'::INTERVAL
) AS d;
```

**Validate the date dimension**

```sql
-- Check total records and date range
SELECT
    COUNT(*) AS total_dates,
    MIN(full_date) AS start_date,
    MAX(full_date) AS end_date
FROM gold.dim_date;

-- Validate fiscal year calculation
SELECT
    full_date,
    fiscal_year
FROM gold.dim_date
WHERE full_date IN ('2023-10-01', '2024-01-01');

-- Check for duplicate date keys
SELECT
    date_key,
    COUNT(*) AS duplicate_count
FROM gold.dim_date
GROUP BY date_key
HAVING COUNT(*) > 1;
```

-> **Results:**
- Date range: 2022-01-01 to 2026-12-31.
- Total records: 1,826 dates.
- Fiscal year = 2023 for both 2023-10-01 and 2024-01-01.
- No duplicate date keys.


#### 3.1.2 — Create `dim_employees` with SCD Type 2 (Slowly Changing Dimension Type 2)

Implement Slowly Changing Dimension Type 2 to **preserve employee history** when attributes such as region or team change.

**Create the employee dimension table**

```sql
-- Create employee dimension with a surrogate primary key
CREATE TABLE gold.dim_employees (
    employee_key SERIAL PRIMARY KEY,  -- Unique key for each employee version

    employee_id TEXT NOT NULL,        -- Business key identifying the employee
    full_name TEXT,
    gender TEXT,

    date_of_birth DATE,
    join_date DATE,

    position TEXT,
    region TEXT,

    team TEXT,
    email TEXT,
    phone BIGINT,
    status TEXT,

    effective_from DATE,              -- Start date of this employee version
    effective_to DATE,                -- End date of this employee version

    is_current BOOLEAN                -- Indicates the current employee record
);
```

**Build employee history using `LEAD()`**

```sql
-- Identify the next effective date for each employee
WITH employee_history AS (
    SELECT
        employee_id,
        full_name,
        gender,
        date_of_birth,
        join_date,
        position,
        region,
        team,
        email,
        phone,
        status,
        effective_date,

        -- Retrieve the effective date of the next employee version
        LEAD(effective_date) OVER (
            PARTITION BY employee_id
            ORDER BY effective_date
        ) AS next_effective_date

    FROM silver."EmployeeMaster"
)

-- Insert employee history into the Gold dimension
INSERT INTO gold.dim_employees (
    employee_id,
    full_name,
    gender,
    date_of_birth,
    join_date,
    position,
    region,
    team,
    email,
    phone,
    status,
    effective_from,
    effective_to,
    is_current
)

SELECT
    employee_id,
    full_name,
    gender,
    date_of_birth,
    join_date,
    position,
    region,
    team,
    email,
    phone,
    status,

    effective_date AS effective_from,

    -- Close the old version one day before the next version starts
    COALESCE(
        next_effective_date - INTERVAL '1 day',
        DATE '9999-12-31'
    )::DATE AS effective_to,

    -- Mark the latest version as the current record
    CASE
        WHEN next_effective_date IS NULL THEN TRUE
        ELSE FALSE
    END AS is_current

FROM employee_history;
```
**Validate employee history**

```sql
-- Check SCD Type 2 history for EMP1004
SELECT
    employee_key,
    employee_id,
    full_name,
    region,
    team,
    effective_from,
    effective_to,
    is_current
FROM gold.dim_employees
WHERE employee_id = 'EMP1004'
ORDER BY effective_from;
```

-> **Result**: EMP1004 has two records:

| employee_id | region | team | effective_from | effective_to | is_current |
|---|---|---|---|---|---|
| EMP1004 | Miền Bắc | Team Bắc 1 | 2024-01-01 | 2024-06-30 | FALSE |
| EMP1004 | Miền Trung | Team Trung 2 | 2024-07-01 | 9999-12-31 | TRUE |

```sql
-- Check that each employee has exactly one current record
SELECT
    employee_id,
    COUNT(*) AS total_records,
    SUM(
        CASE WHEN is_current = TRUE THEN 1 ELSE 0 END
    ) AS current_records
FROM gold.dim_employees
GROUP BY employee_id
HAVING SUM(
    CASE WHEN is_current = TRUE THEN 1 ELSE 0 END
) <> 1;

-- Check employees who changed region
SELECT
    employee_id,
    full_name,
    region,
    team,
    effective_from,
    effective_to,
    is_current
FROM gold.dim_employees
WHERE employee_id IN (
    'EMP1018',
    'EMP1004',
    'EMP1092',
    'EMP1026',
    'EMP1078'
)
ORDER BY employee_id, effective_from;
```

-> **Results:**
- EMP1004 has two records representing the old and new regions.
- The five employees identified in the Change Log have their historical versions preserved.
- Each employee has exactly one current record.
- Historical records have `is_current = FALSE`; the latest records have `is_current = TRUE`.


#### 3.1.3 — Build `dim_customers` (SCD Type 1)

Customer dimension stores customer information for sales analysis. It uses **SCD Type 1**, meaning updated customer information overwrites the previous values rather than maintaining historical versions.

**Create the table**

```sql
CREATE TABLE gold.dim_customers (
    customer_key SERIAL PRIMARY KEY,
    customer_id TEXT NOT NULL,
    customer_name TEXT,
    customer_type TEXT,
    channel TEXT,
    province TEXT,
    region TEXT,
    address TEXT,
    phone TEXT,
    tax_code TEXT,
    join_date DATE,
    credit_limit NUMERIC(18,2),
    status TEXT
);
```

**Load data from Silver**

```sql
INSERT INTO gold.dim_customers (
    customer_id,
    customer_name,
    customer_type,
    channel,
    province,
    region,
    address,
    phone,
    tax_code,
    join_date,
    credit_limit,
    status
)
SELECT
    TRIM(customer_id),
    TRIM(customer_name),
    TRIM(customer_type),
    TRIM(channel),
    TRIM(province),
    TRIM(region),
    TRIM(address),
    phone::TEXT,
    tax_code::TEXT,
    join_date,
    credit_limit::NUMERIC(18,2),
    TRIM(status)
FROM silver."CustomerMaster";
```

**Validate the data**

```sql
-- Check loaded records
SELECT *
FROM gold.dim_customers
LIMIT 5;

-- Compare row counts between Silver and Gold
SELECT
    (SELECT COUNT(*) FROM silver."CustomerMaster") AS silver_count,
    (SELECT COUNT(*) FROM gold.dim_customers) AS gold_count;

-- Check duplicate customer IDs
SELECT
    customer_id,
    COUNT(*) AS total_records
FROM gold.dim_customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

#### 3.1.4 — Build `dim_products` (SCD Type 1)

Product dimension stores product information, including category, pricing, cost, and stock quantity. It uses **SCD Type 1**, so changes to product attributes overwrite existing values.

<details>
<summary><strong>View SQL — Create and load dim_products</strong></summary>

**Create the table**

```sql
CREATE TABLE gold.dim_products (
    product_key SERIAL PRIMARY KEY,
    product_id TEXT NOT NULL,
    product_name TEXT,
    category TEXT,
    subcategory TEXT,
    unit TEXT,
    unit_price NUMERIC(18,2),
    cost_price NUMERIC(18,2),
    stock_quantity INTEGER,
    status TEXT,
    created_date DATE
);
```

**Load data from Silver**

```sql
INSERT INTO gold.dim_products (
    product_id,
    product_name,
    category,
    subcategory,
    unit,
    unit_price,
    cost_price,
    stock_quantity,
    status,
    created_date
)
SELECT
    TRIM(product_id),
    TRIM(product_name),
    TRIM(category),
    TRIM(subcategory),
    TRIM(unit),
    unit_price::NUMERIC(18,2),
    cost_price::NUMERIC(18,2),
    stock_quantity::INTEGER,
    TRIM(status),
    created_date
FROM silver."ProductMaster";
```

**Validate the data**

```sql
-- Check loaded records
SELECT *
FROM gold.dim_products
LIMIT 5;

-- Compare row counts between Silver and Gold
SELECT
    (SELECT COUNT(*) FROM silver."ProductMaster") AS silver_count,
    (SELECT COUNT(*) FROM gold.dim_products) AS gold_count;

-- Check duplicate product IDs
SELECT
    product_id,
    COUNT(*) AS total_records
FROM gold.dim_products
GROUP BY product_id
HAVING COUNT(*) > 1;
```

</details>

---

#### 3.1.5 — Build `dim_distributors` (SCD Type 2)

Distributor dimension is designed using **SCD Type 2** to support historical tracking of distributor information.

<details>
<summary><strong>View SQL — Create and load dim_distributors</strong></summary>

**Create the table**

```sql
CREATE TABLE gold.dim_distributors (
    distributor_key SERIAL PRIMARY KEY,

    distributor_id TEXT NOT NULL,
    distributor_name TEXT,

    tier TEXT,
    channel TEXT,

    province TEXT,
    region TEXT,

    contact_person TEXT,
    phone TEXT,
    email TEXT,
    tax_code TEXT,

    join_date DATE,

    credit_limit NUMERIC(18,2),

    status TEXT,

    assigned_supervisor_id TEXT,

    effective_from DATE,
    effective_to DATE,

    is_current BOOLEAN
);
```

**Load data from Silver**

```sql
INSERT INTO gold.dim_distributors (
    distributor_id,
    distributor_name,

    tier,
    channel,

    province,
    region,

    contact_person,
    phone,
    email,
    tax_code,

    join_date,

    credit_limit,

    status,

    assigned_supervisor_id,

    effective_from,
    effective_to,

    is_current
)
SELECT
    TRIM(distributor_id),
    TRIM(distributor_name),

    TRIM(tier),
    TRIM(channel),

    TRIM(province),
    TRIM(region),

    TRIM(contact_person),
    phone::TEXT,
    TRIM(email),
    tax_code::TEXT,

    join_date,

    credit_limit::NUMERIC(18,2),

    TRIM(status),

    TRIM(assigned_supervisor_id),

    join_date AS effective_from,
    DATE '9999-12-31' AS effective_to,

    TRUE AS is_current

FROM silver."DistributorMaster";
```

**Validate the data**

```sql
-- Check loaded records
SELECT *
FROM gold.dim_distributors
LIMIT 5;

-- Check that each distributor has exactly one current record
SELECT
    distributor_id,
    COUNT(*) AS total_records,
    SUM(CASE WHEN is_current = TRUE THEN 1 ELSE 0 END) AS current_records
FROM gold.dim_distributors
GROUP BY distributor_id
HAVING SUM(CASE WHEN is_current = TRUE THEN 1 ELSE 0 END) <> 1;
```

</details>

### Task 3.2 — Build Fact Tables & Data Marts

#### 3.2.1 — Build `fact_sales`

**Create the table**

```sql
CREATE TABLE gold.fact_sales (
    sales_key BIGSERIAL PRIMARY KEY,

    order_id TEXT NOT NULL,
    order_date DATE NOT NULL,
    date_key INTEGER,

    employee_key INTEGER,
    customer_key INTEGER,
    product_key INTEGER,

    region TEXT,
    province TEXT,
    channel TEXT,

    quantity INTEGER,
    unit_price NUMERIC(18,2),
    discount_pct NUMERIC(10,2),
    discount_amount NUMERIC(18,2),
    gross_amount NUMERIC(18,2),
    net_amount NUMERIC(18,2),

    delivery_status TEXT,
    payment_method TEXT,
    payment_status TEXT
);
```

**Load data from Silver**

```sql
INSERT INTO gold.fact_sales (
    order_id,

    order_date,
    date_key,

    employee_key,
    customer_key,
    product_key,

    region,
    province,

    channel,

    quantity,
    unit_price,
    discount_pct,
    discount_amount,
    gross_amount,
    net_amount,

    delivery_status,
    payment_method,
    payment_status
)
SELECT
    s.order_id,
    s.order_date,
    TO_CHAR(s.order_date, 'YYYYMMDD')::INTEGER,

    e.employee_key,
    c.customer_key,
    p.product_key,

    TRIM(s.region),
    TRIM(s.province),
    TRIM(s.channel),

    s.quantity,
    s.unit_price::NUMERIC(18,2),
    s.discount_pct::NUMERIC(10,2),
    s.discount_amount::NUMERIC(18,2),
    s.gross_amount::NUMERIC(18,2),
    s.net_amount::NUMERIC(18,2),

    TRIM(s.delivery_status),
    TRIM(s.payment_method),
    TRIM(s.payment_status)

FROM silver."SalesTransaction" s

LEFT JOIN gold.dim_employees e
    ON s.employee_id = e.employee_id
    AND s.order_date BETWEEN e.effective_from AND e.effective_to

LEFT JOIN gold.dim_customers c
    ON s.customer_id = c.customer_id

LEFT JOIN gold.dim_products p
    ON s.product_id = p.product_id;
```

**Validate the data**

```sql
-- Compare row counts between Silver and Gold
SELECT
    (SELECT COUNT(*) FROM silver."SalesTransaction") AS silver_count,
    (SELECT COUNT(*) FROM gold.fact_sales) AS gold_count;

-- Check missing dimension key mappings
SELECT COUNT(*)
FROM gold.fact_sales
WHERE employee_key IS NULL
   OR customer_key IS NULL
   OR product_key IS NULL;

-- Preview the loaded records
SELECT *
FROM gold.fact_sales
LIMIT 5;
```

#### 3.2.2 — Build `fact_targets`

<details>
<summary><strong>View SQL — Create and load fact_targets</strong></summary>

**Create the table**

```sql
CREATE TABLE gold.fact_targets (
    target_key BIGSERIAL PRIMARY KEY,

    plan_version TEXT,
    version_date DATE,
    effective_from DATE,
    effective_to DATE,

    employee_key INTEGER,
    employee_id TEXT,

    date_key INTEGER,
    year INTEGER,
    month INTEGER,

    target_revenue NUMERIC(18,2),
    target_quantity INTEGER,
    target_new_customers INTEGER,

    recorded_at TIMESTAMP,
    is_latest BOOLEAN
);
```

**Load data from Silver and map employee keys**

```sql
INSERT INTO gold.fact_targets (
    plan_version,

    version_date,
    effective_from,
    effective_to,

    employee_key,
    employee_id,

    date_key,
    year,
    month,

    target_revenue,
    target_quantity,
    target_new_customers,

    recorded_at,
    is_latest
)

SELECT
    t.plan_version,
    t.version_date,
    t.effective_from,
    t.effective_to,

    e.employee_key,
    t.employee_id,

    TO_CHAR(
        MAKE_DATE(t.year, t.month, 1),
        'YYYYMMDD'
    )::INTEGER,

    t.year,
    t.month,

    t.target_revenue,
    t.target_quantity,
    t.target_new_customers,

    t.recorded_at,
    t.is_latest

FROM silver."SalesTarget" t

LEFT JOIN gold.dim_employees e
    ON t.employee_id = e.employee_id
    AND MAKE_DATE(t.year, t.month, 1)
        BETWEEN e.effective_from AND e.effective_to;
```

**Validate the data**

```sql
-- Compare row counts between Silver and Gold
SELECT
    (SELECT COUNT(*) FROM silver."SalesTarget") AS silver_count,
    (SELECT COUNT(*) FROM gold.fact_targets) AS gold_count;

-- Check missing employee key mappings
SELECT COUNT(*)
FROM gold.fact_targets
WHERE employee_key IS NULL;

-- Check target records by version
SELECT
    plan_version,
    is_latest,
    COUNT(*) AS total_records
FROM gold.fact_targets
GROUP BY plan_version, is_latest
ORDER BY plan_version, is_latest;

-- Check duplicate employee-month-version records
SELECT
    employee_id,
    year,
    month,
    plan_version,
    COUNT(*) AS total_records
FROM gold.fact_targets
GROUP BY employee_id, year, month, plan_version
HAVING COUNT(*) > 1;

-- Preview the loaded records
SELECT *
FROM gold.fact_targets
LIMIT 5;
```

</details>

---

#### 3.2.3 — Build `fact_returns`

<details>
<summary><strong>View SQL — Create and load fact_returns</strong></summary>

**1. Create the table**

```sql
CREATE TABLE gold.fact_returns (
    return_key BIGSERIAL PRIMARY KEY,

    return_id TEXT NOT NULL,
    original_order_id TEXT,
    return_date DATE NOT NULL,
    date_key INTEGER,

    customer_key INTEGER,
    employee_key INTEGER,
    product_key INTEGER,

    customer_id TEXT,
    employee_id TEXT,
    product_id TEXT,

    region TEXT,
    province TEXT,

    return_quantity INTEGER,
    unit_price NUMERIC(18,2),
    return_amount NUMERIC(18,2),

    return_reason TEXT,
    status TEXT
);
```

**Load data from Silver and map dimension keys**

```sql
INSERT INTO gold.fact_returns (
    return_id,
    original_order_id,
    return_date,
    date_key,

    customer_key,
    employee_key,
    product_key,

    customer_id,
    employee_id,
    product_id,

    region,
    province,

    return_quantity,
    unit_price,
    return_amount,

    return_reason,
    status
)
SELECT
    r.return_id,
    r.original_order_id,
    r.return_date,

    TO_CHAR(r.return_date, 'YYYYMMDD')::INTEGER,

    c.customer_key,
    e.employee_key,
    p.product_key,

    r.customer_id,
    r.employee_id,
    r.product_id,

    TRIM(r.region),
    TRIM(r.province),

    r.return_quantity::INTEGER,
    r.unit_price::NUMERIC(18,2),
    r.return_amount::NUMERIC(18,2),

    TRIM(r.return_reason),
    TRIM(r.status)

FROM silver."ReturnTransactions" r

LEFT JOIN gold.dim_customers c
    ON r.customer_id = c.customer_id

LEFT JOIN gold.dim_employees e
    ON r.employee_id = e.employee_id
    AND r.return_date BETWEEN e.effective_from AND e.effective_to

LEFT JOIN gold.dim_products p
    ON r.product_id = p.product_id;
```

**Validate the data**

```sql
-- Compare row counts between Silver and Gold
SELECT
    (SELECT COUNT(*) FROM silver."ReturnTransactions") AS silver_count,
    (SELECT COUNT(*) FROM gold.fact_returns) AS gold_count;

-- Check missing dimension key mappings
SELECT COUNT(*)
FROM gold.fact_returns
WHERE customer_key IS NULL
   OR employee_key IS NULL
   OR product_key IS NULL;

-- Preview the loaded records
SELECT *
FROM gold.fact_returns
LIMIT 5;
```

</details>

--- 

#### 3.2.4 — Build `fact_distributor_orders`

<details>
<summary><strong>View SQL — Create and load fact_distributor_orders</strong></summary>

**Create the table**

```sql
CREATE TABLE gold.fact_distributor_orders (
    distributor_order_key BIGSERIAL PRIMARY KEY,

    order_id TEXT,
    order_date DATE,
    date_key INTEGER,

    distributor_key INTEGER,
    product_key INTEGER,

    distributor_id TEXT,
    product_id TEXT,

    region TEXT,
    channel TEXT,

    product_category TEXT,
    qty_ordered INTEGER,
    qty_delivered INTEGER,

    fill_rate_pct NUMERIC(10,2),
    unit_price_list NUMERIC(18,2),
    distributor_price NUMERIC(18,2),
    gross_amount NUMERIC(18,2),
    delivered_amount NUMERIC(18,2),

    expected_delivery_date DATE,
    actual_delivery_date DATE,
    ontime_delivery TEXT,
    delivery_status TEXT,
    payment_terms TEXT
);
```

**Load data from Silver and map dimension keys**

```sql
INSERT INTO gold.fact_distributor_orders (
    order_id,
    order_date,
    date_key,

    distributor_key,
    product_key,

    distributor_id,
    product_id,

    region,
    channel,

    product_category,
    qty_ordered,
    qty_delivered,

    fill_rate_pct,
    unit_price_list,
    distributor_price,
    gross_amount,
    delivered_amount,

    expected_delivery_date,
    actual_delivery_date,
    ontime_delivery,
    delivery_status,
    payment_terms
)
SELECT
    TRIM(d.order_id),

    d.order_date::DATE,

    TO_CHAR(d.order_date::DATE, 'YYYYMMDD')::INTEGER,

    dist.distributor_key,
    p.product_key,

    TRIM(d.distributor_id),
    TRIM(d.product_id),
    TRIM(d.region),
    TRIM(d.channel),
    TRIM(d.product_category),

    d.qty_ordered::INTEGER,
    d.qty_delivered::INTEGER,
    d.fill_rate_pct::NUMERIC(10,2),
    d.unit_price_list::NUMERIC(18,2),
    d.distributor_price::NUMERIC(18,2),
    d.gross_amount::NUMERIC(18,2),
    d.delivered_amount::NUMERIC(18,2),
    d.expected_delivery_date::DATE,
    d.actual_delivery_date::DATE,

    TRIM(d.ontime_delivery),
    TRIM(d.delivery_status),
    TRIM(d.payment_terms)

FROM silver."DistributorOrders" d

LEFT JOIN gold.dim_distributors dist
    ON d.distributor_id = dist.distributor_id
    AND d.order_date::DATE
        BETWEEN dist.effective_from AND dist.effective_to

LEFT JOIN gold.dim_products p
    ON d.product_id = p.product_id;
```

**Validate the data**

```sql
-- Preview loaded records
SELECT *
FROM gold.fact_distributor_orders
LIMIT 5;

-- Check total records and missing dimension keys
SELECT
    COUNT(*) AS total_rows,
    COUNT(*) FILTER (
        WHERE distributor_key IS NULL
    ) AS missing_distributor,
    COUNT(*) FILTER (
        WHERE product_key IS NULL
    ) AS missing_product
FROM gold.fact_distributor_orders;
```

</details>

---

### Task 3.3 — Build Mart Tables

#### 3.3.1 — Build `mart_sales_vs_target`

**Create the table**

```sql
CREATE TABLE gold.mart_sales_vs_target (
    employee_id TEXT,
    year INTEGER,
    month INTEGER,
    plan_version TEXT,

    actual_revenue NUMERIC(18,2),
    target_revenue NUMERIC(18,2),

    variance NUMERIC(18,2),
    achievement_rate NUMERIC(10,4)
);
```

**Load aggregated data**

```sql
INSERT INTO gold.mart_sales_vs_target (
    employee_id,
    year,
    month,
    plan_version,
    actual_revenue,
    target_revenue,
    variance,
    achievement_rate
)

SELECT
    t.employee_id,
    t.year,
    t.month,
    t.plan_version,

    COALESCE(SUM(s.net_amount), 0) AS actual_revenue,

    t.target_revenue,

    COALESCE(SUM(s.net_amount), 0) - t.target_revenue
        AS variance,

    CASE
        WHEN t.target_revenue = 0 THEN NULL
        ELSE COALESCE(SUM(s.net_amount), 0)
             / t.target_revenue
    END AS achievement_rate

FROM gold.fact_targets t

LEFT JOIN gold.dim_employees e
    ON t.employee_id = e.employee_id
    AND MAKE_DATE(t.year, t.month, 1)
        BETWEEN e.effective_from AND e.effective_to

LEFT JOIN gold.fact_sales s
    ON s.employee_key = e.employee_key
    AND EXTRACT(YEAR FROM s.order_date)::INTEGER = t.year
    AND EXTRACT(MONTH FROM s.order_date)::INTEGER = t.month

GROUP BY
    t.employee_id,
    t.year,
    t.month,
    t.plan_version,
    t.target_revenue;
```

**Validate the data**

```sql
-- Preview mart records
SELECT *
FROM gold.mart_sales_vs_target
LIMIT 10;

-- Compare actual revenue and target by plan version
SELECT
    plan_version,
    is_latest,
    SUM(actual_revenue) AS total_actual_revenue,
    SUM(target_revenue) AS total_target_revenue,
    AVG(achievement_pct) AS avg_achievement_pct
FROM gold.mart_sales_vs_target
GROUP BY plan_version, is_latest
ORDER BY plan_version, is_latest;

-- Check duplicate employee-month-version records
SELECT
    employee_id,
    year,
    month,
    plan_version,
    COUNT(*) AS total_records
FROM gold.mart_sales_vs_target
GROUP BY employee_id, year, month, plan_version
HAVING COUNT(*) > 1;
```

</details>

---

#### 3.3.2 — Build `mart_distributor_performance`

<details>
<summary><strong>View SQL — Create and load mart_distributor_performance</strong></summary>

**Create the table**

```sql
CREATE TABLE gold.mart_distributor_performance (
    distributor_id TEXT,
    distributor_name TEXT,
    region TEXT,
    tier TEXT,

    total_orders INTEGER,
    total_qty_ordered BIGINT,
    total_qty_delivered BIGINT,

    fill_rate_pct NUMERIC(10,2),

    gross_revenue NUMERIC(18,2),
    delivered_revenue NUMERIC(18,2),

    ontime_orders INTEGER,
    ontime_rate_pct NUMERIC(10,2)
);
```

**Load aggregated data**

```sql
INSERT INTO gold.mart_distributor_performance (
    distributor_id,
    distributor_name,
    region,
    tier,

    total_orders,
    total_qty_ordered,
    total_qty_delivered,

    fill_rate_pct,

    gross_revenue,
    delivered_revenue,

    ontime_orders,
    ontime_rate_pct
)
SELECT
    d.distributor_id,
    d.distributor_name,
    d.region,
    d.tier,

    COUNT(DISTINCT f.order_id) AS total_orders,

    SUM(f.qty_ordered) AS total_qty_ordered,
    SUM(f.qty_delivered) AS total_qty_delivered,

    CASE
        WHEN SUM(f.qty_ordered) = 0 THEN 0
        ELSE
            SUM(f.qty_delivered)::NUMERIC
            / SUM(f.qty_ordered) * 100
    END AS fill_rate_pct,

    SUM(f.gross_amount) AS gross_revenue,
    SUM(f.delivered_amount) AS delivered_revenue,

    COUNT(DISTINCT f.order_id)
        FILTER (WHERE f.ontime_delivery = 'Yes')
        AS ontime_orders,

    CASE
        WHEN COUNT(DISTINCT f.order_id) = 0 THEN 0
        ELSE
            COUNT(DISTINCT f.order_id)
            FILTER (WHERE f.ontime_delivery = 'Yes')::NUMERIC
            / COUNT(DISTINCT f.order_id) * 100
    END AS ontime_rate_pct

FROM gold.fact_distributor_orders f

LEFT JOIN gold.dim_distributors d
    ON f.distributor_key = d.distributor_key

GROUP BY
    d.distributor_id,
    d.distributor_name,
    d.region,
    d.tier;
```

**Validate the data**

```sql
-- Rank distributors by delivered revenue
SELECT *
FROM gold.mart_distributor_performance
ORDER BY delivered_revenue DESC
LIMIT 10;

-- Check duplicate distributor records
SELECT
    distributor_id,
    COUNT(*) AS total_records
FROM gold.mart_distributor_performance
GROUP BY distributor_id
HAVING COUNT(*) > 1;
```

</details>

--- 

## VI. Conclusions

VietDist Analytics demonstrates the development of an end-to-end pipeline, transforming raw business data from multiple sources into structured, analysis-ready datasets.

By combining Python-based data ingestion, SQL transformations, Medallion Architecture, and dimensional modeling, the project establishes a structured data foundation for analyzing sales, targets, returns, and distributor performance. It also applies SCD Type 2 to preserve employee history and manages multiple sales target versions to support historical and performance comparisons.

The resulting Gold layer, consisting of dimension tables, fact tables, and analytical data marts, provides a foundation for business reporting and data-driven decision-making.

Overall, this project demonstrates practical Data Engineering and Analytics Engineering skills, from source data integration and data quality management to data warehouse design and analytical data preparation.

