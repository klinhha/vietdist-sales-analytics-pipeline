# [PYTHON] VietDist Sales Analytics Pipeline

## 1. Project Overview

VietDist Analytics is an **end-to-end** data warehousing and analytics engineering project designed to transform sales, customer, product, employee, distributor, sales target, and return data into a structured analytical platform.

The project uses **Python** to ingest Excel and CSV files from Google Drive into PostgreSQL, SQL to clean and transform data through a Medallion Architecture, and dimensional modeling to prepare analytical datasets for business reporting.

The platform aims to support sales performance monitoring, actual-versus-target analysis, employee performance evaluation, return analysis, and distributor delivery performance.

## 2. Project Objectives

- **Automate data ingestion** from Google Drive into PostgreSQL using Python.

- **Organize data** into Bronze, Silver, and Gold layers.

- **Clean**, **standardize**, and **validate** data before analytical use.

- **Build dimensional models** and **fact tables** for business analysis.

- **Preserve** employee history and **manage** sales target versions.

- **Prepare** analytical datasets for reporting and decision-making.

## 3. Medallion Architecture

The project follows a three-layer architecture to separate raw data ingestion, data transformation, and business-oriented modeling.

| Layer | Purpose | Implementation |
|---|---|---|
| **Bronze** | Stores source data as ingested, preserving the original information for downstream processing. | Python loaders and PostgreSQL |
| **Silver** | Cleans and standardizes data, applies appropriate data types, handles duplicates, and validates data quality. | SQL transformations in PostgreSQL |
| **Gold** | Organizes validated data into dimensional models, fact tables, and data marts for analytical queries and reporting. | SQL-based data modeling |

### Data Flow

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

## 4. Data Modeling

The Gold layer follows dimensional modeling principles, separating descriptive dimensions from measurable business events and analytical summaries.

### Dimension Tables

| Table | Description |
|---|---|
| `dim_date` | Calendar attributes for date-based analysis, including month, quarter, and year. |
| `dim_employees` | Employee attributes with historical tracking using SCD Type 2. |
| `dim_customers` | Customer profiles, classifications, regions, and credit information. |
| `dim_products` | Product attributes, categories, pricing, and cost information. |
| `dim_distributors` | Distributor profiles, tiers, channels, and historical attributes. |

### Fact Tables

| Table | Description |
|---|---|
| `fact_sales` | Sales transactions at the individual order-line level. |
| `fact_targets` | Monthly employee sales targets with plan version information. |
| `fact_returns` | Product return records and associated return amounts and statuses. |
| `fact_distributor_orders` | Distributor order lines, ordered and delivered quantities, delivery performance, and payment terms. |

### Analytical Data Marts

| Table | Description |
|---|---|
| `mart_sales_vs_target` | Combines actual sales and target data to support monthly performance comparisons. |
| `mart_distributor_performance` | Consolidates distributor performance metrics, including order fulfillment, fill rates, and on-time delivery. |

## 5. Project Structure

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

## 6. Technology Stack

- **Python:** Data ingestion and reusable loading utilities.
- **Pandas & OpenPyXL:** Reading and processing CSV and Excel files.
- **PostgreSQL:** Data storage, SQL transformations, and warehouse modeling.
- **SQLAlchemy & psycopg2:** Python–PostgreSQL connectivity.
- **Google Drive API:** Accessing source data files.
- **Git & GitHub:** Version control and project documentation.
- **VS Code & pgAdmin & DBeaver:** Development and database management.

## 7. Analytical Journey
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
**Libraries in `requirement.txt`**

| Library | Purpose |
|---|---|
| `pandas` | Reads, processes, and transforms tabular data using DataFrames. |
| `sqlalchemy` | Provides database connectivity and tools for interacting with PostgreSQL. |
| `psycopg2-binary` | PostgreSQL database adapter for Python. |
| `openpyxl` | Reads and writes Excel `.xlsx` files. |
| `google-api-python-client` | Provides Python tools for accessing Google APIs, including Google Drive. |
| `google-auth` | Handles authentication with Google services, including service account credentials. |
| `python-dotenv` | Loads environment variables from a `.env` file. |


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

**Result:** Three rows containing `bronze`, `silver`, and `gold`.

- **Bronze:** Stores source data with minimal transformation.
- **Silver:** Stores cleaned, standardized, and validated data.
- **Gold:** Stores dimensional models, fact tables, and analytical marts.

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

### Subtask 1.1.1 — Create the Google Drive Utility Module

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

### Subtask 1.2.1 — Create the Database Utility Module

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

### Subtask 1.3.1 — Create a Loader Script

Each source table has its own loader script in the `loader/` directory:

- `load_customer_master.py`
- `load_distributor_master.py`
- `load_distributor_orders.py`
- `load_employee_master.py`
- `load_product_master.py`
- `load_promotion_program.py`
- `load_return_transactions.py`
- `load_sales_transaction.py`
- `load_sales_target_auto.py`
- `load_territory_mapping.py`

**A sample loader workflow from `load_sales_transaction.py`:**

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

### Subtask 1.3.2 — Verify the Bronze Tables

After running a loader, connect to the project database in pgAdmin and check the destination table.

```sql
-- Check the number of rows loaded into a Bronze table
SELECT COUNT(*)
FROM bronze.Sales_Transaction;

-- Inspect a sample of the loaded data
SELECT *
FROM bronze.Sales_Transaction
LIMIT 10;
```
-- Results: COUNT: 119,101 rows

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



### Task 1.4 — Special Handling: Multi-Version Sales Target Plans

The `sales_target_plan` file requires special handling because it may **contain multiple versions**, such as v1 and v2, either in separate files or in different sheets within the same Excel workbook.

Unlike the other source tables, sales target data must preserve its version history. All versions must be extracted and loaded without unintentionally overwriting previously ingested versions.

### Subtask 1.4.1 — Read Multiple Versions from Excel and create metadata

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
```

### Subtask 1.4.3 — Combine Versions into a Raw Dataset and load the Data into PostgreSQL Bronze

```python
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

### Subtask 1.4.5 — Validate the Loaded Data

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
-- Result: There are 2 available versions (v1: 1332 rows, v2: 618 rows), 12 months (T1 -> T12)


