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
### Phase 0. ENVIRONMENT SETUP
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

Create a file named `.env` in the project root directory. This file stores configuration values and sensitive credentials used by the Python scripts.

Example:

```dotenv
# PostgreSQL connection settings
PG_USER='postgres'
PG_PWD='your_database_password'
PG_DATABASE_NAME='postgres'
PG_HOST='localhost'
PG_PORT='5432'

# Google Service Account credentials
GOOGLE_SERVICE_ACCOUNT_FILE=/absolute/path/to/service_account.json

# Google Drive source folder
GOOGLE_DRIVE_FOLDER_ID=your_google_drive_folder_id
```

**Security Precautions**

Never commit `.env` or service account JSON files to GitHub.

Add the following entries to `.gitignore`:

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

If `.env` and `credentials/` are not already listed in `.gitignore`, you can add them from the terminal:

```bash
echo '.env' >> .gitignore
echo 'credentials/' >> .gitignore
```

**Important:** If a password or service account key is accidentally committed to a public repository, treat it as compromised. Change the exposed password or revoke and replace the affected key, then remove the exposed secret from the repository history where appropriate.



