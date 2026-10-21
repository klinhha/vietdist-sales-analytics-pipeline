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
### 1. Objective

Install and configure all required tools, then verify that they work correctly before working with any data.

### 2. Install Required Tools

1. **Python 3.11 or later**
   - Download and install Python from [python.org](https://www.python.org/downloads/).
   - Verify the installation in the terminal:

   ```bash
   python3 --version
   ```

2. **PostgreSQL 16**
   - Download and install PostgreSQL from [postgresql.org](https://www.postgresql.org/download/).
   - Set a password for the `postgres` user during installation and store it securely.

3. **Visual Studio Code**
   - Install [VS Code](https://code.visualstudio.com/).
   - Install the **Python** and **SQLTools** extensions.

4. **Git and GitHub**
   - Install [Git](https://git-scm.com/downloads).
   - Create a repository on [GitHub](https://github.com/).
   - Clone the repository to your local machine.

   Verify Git is installed:

   ```bash
   git --version
   ```

### 3. Create a Python Virtual Environment

Open the terminal in the project root directory and run:

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

After installation, verify the environment and installed packages:

```bash
python --version
pip list
```

### 4. Initialize the PostgreSQL Database

Open pgAdmin and connect to your PostgreSQL server.

Create a dedicated database for the project:

```sql
CREATE DATABASE postgres;
```

Connect to `vietdist_dw`, open the Query Tool, and create the three schemas:

```sql
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

### 5. Configure Environment Variables

**Create** a file named **`.env`** in the project root directory. This file stores **configuration values** and **sensitive credentials** used by the Python scripts.

```dotenv
# PostgreSQL connection settings
PG_USER='postgres'
PG_PWD='database_password'
PG_DATABASE_NAME='postgres'
PG_HOST='localhost'
PG_PORT='5432'
```

**Security Precautions**

**Never commit** `.env` or service account JSON files to GitHub.

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

If `.env` and `credentials/` are not already listed in `.gitignore`, add them from the terminal:

```bash
echo '.env' >> .gitignore
echo 'credentials/' >> .gitignore
```

**Important:** If a password or service account key is **accidentally committed** to a public repository, **treat it as compromised**. **Change** the exposed password or revoke and replace the affected key, then **remove** the exposed secret from the repository history where appropriate.

### Phase 1 — BRONZE LAYER: Source Connection & Ingestion

### 1. Objective

Establish a connection between Google Drive and PostgreSQL, automate the retrieval of source files, and load raw data into the Bronze layer.

The Bronze layer stores ingested source data with minimal transformation, preserving the original information for subsequent cleaning and modeling in the Silver and Gold layers.

**Data flow:**

Google Drive → Python ingestion modules → PostgreSQL Bronze layer

### Task 1.1 — Connect to Google Drive and Read Source Files

### Subtask 1.1.1 — Set Up Google Service Account

The Google Service Account credentials are provided by the course.

1. **Obtain** the service account JSON file.
2. **Store** the JSON file locally in the **'config'** folder.
3. **Share** the source Google Drive folder with the service account email address, granting it permission to read the required files.
4. **Store** the JSON file path and Google Drive folder ID in **`.env`**.

Example `.env` configuration:

```dotenv
# Google Service Account credentials
GOOGLE_SERVICE_ACCOUNT_FILE=/absolute/path/to/service_account.json

# Google Drive source folder
GOOGLE_DRIVE_FOLDER_ID=your_google_drive_folder_id
```

The service account JSON contains **sensitive credentials** and **must not be committed** to GitHub.

### Subtask 1.1.2 — Create the Google Drive Utility Module

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
# Retrieve the service account JSON path and Google Drive folder ID from .env
# This avoids hardcoding configuration values directly into the Python code

SERVICE_ACCOUNT_FILE = os.getenv("GOOGLE_SERVICE_ACCOUNT_FILE")
FOLDER_DRIVE_ID = os.getenv("GOOGLE_DRIVE_FOLDER_ID")

# Define the service account permissions (OAuth scope)
# Read-only access is sufficient because the pipeline only needs to retrieve source files

SCOPES = ["https://www.googleapis.com/auth/drive.readonly"]
```

#### 3. Create the Google Drive Connection

```python
# Create a reusable function that authenticates the service account
# and returns a Google Drive API service client

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

The function loads the service account credentials, applies the read-only permission scope, and returns a service client that other functions can use to access Google Drive.

#### 4. Download a File into Memory

```python
# Download a source file from Google Drive and return it as an in-memory buffer

def download_data(service, file_id):
    request = service.files().get_media(
        fileId=file_id,
        supportsAllDrives=True
    )

    # Create a buffer in RAM to store the downloaded file
    buffer = BytesIO()

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

`BytesIO` allows the downloaded file to be read directly from memory, avoiding the need to save a temporary copy to the local disk.

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

**Supported formats in this implementation:**

- `.csv`
- `.xlsx`

Other formats, such as `.xlsb` and `.xlsm`, require additional handling before they can be supported reliably.

## Task 1.2 — Connect to PostgreSQL

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

The connection string combines the database username, password, host, port, and database name. These values are retrieved from `.env` rather than hardcoded into the script.

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

Parameters:

| Parameter | Purpose |
|---|---|
| `df` | The DataFrame to load into PostgreSQL. |
| `table_name` | The destination table name. |
| `schema` | The destination schema; defaults to `bronze`. |
| `if_exists` | Determines what happens when the table already exists. |

**Important:** The current default, `if_exists="replace"`, **drops** and **recreates** an existing table. Use it only when replacing the table is **intentional**. 
For an **initial raw-data load**, `replace` can be convenient. 
For **repeated ingestion**, choose an appropriate strategy (such as incremental or append) to avoid overwriting existing data or duplicating records.

## Task 1.3 — Load Raw Data into the Bronze Layer

**Estimated time:** 3–5 hours

### Bronze Layer Principles

The Bronze layer stores source data with minimal transformation. It preserves source values so that cleaning and standardization can happen in the Silver layer.

For this project, the Bronze tables are stored in the PostgreSQL `bronze` schema.

| Principle | Description |
|---|---|
| Preserve source data | Avoid business transformations during ingestion. |
| Retain source identifiers | Keep original identifiers and values available for downstream processing. |
| Track ingestion | Record when data was loaded and which source file it came from where applicable. |
| Support reprocessing | Retain raw data so downstream transformations can be rerun when needed. |

### Subtask 1.3.1 — Create a Loader Script

Each source table should have its own loader script in the `loader/` directory, for example:

- `load_customer_master.py`
- `load_distributor_master.py`
- `load_employee_master.py`
- `load_product_master.py`
- `load_sales_transaction.py`
- `load_sales_target_auto.py`

A typical loader follows these steps:

1. Connect to Google Drive.
2. Retrieve the required source file.
3. Download the file into memory.
4. Read the file into a pandas DataFrame.
5. Add ingestion metadata if required.
6. Load the DataFrame into the appropriate Bronze table.

Example loader workflow:

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

# Replace these values with the actual source file information
file_id = "YOUR_FILE_ID"
file_name = "YOUR_SOURCE_FILE.csv"

# Download the source file into memory
buffer = download_data(service, file_id)

# Read the source file into a DataFrame
data = read_file(file_name, buffer)

# Record the ingestion timestamp
data["recorded_at"] = pd.Timestamp.now()

# Load the data into the Bronze schema
load_to_postgres(
    df=data,
    table_name="your_table_name",
    schema="bronze",
    if_exists="replace"
)
```

Replace the example file ID, file name, and table name with the actual values. The import statements assume that the scripts are run with the `loader/` directory on Python's import path.

**Note:** `recorded_at` records the ingestion time, but it is not the same as source-file metadata or a unique batch identifier. If the pipeline needs source tracking, batch IDs, or ingestion logs, implement those explicitly.

### Subtask 1.3.2 — Verify the Bronze Tables

After running a loader, connect to the project database in pgAdmin and check the destination table.

```sql
-- Check the number of rows loaded into a Bronze table
SELECT COUNT(*)
FROM bronze.your_table_name;

-- Inspect a sample of the loaded data
SELECT *
FROM bronze.your_table_name
LIMIT 10;
```

Replace `your_table_name` with the actual table name.

Check that:

- The expected number of rows has been loaded.
- The column names are correct.
- The source values are preserved.
- No unexpected truncation or encoding issues occurred.
- The data was loaded into the intended schema and table.


