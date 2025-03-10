### Project Idea: AWS S3 to Snowflake Data Pipeline

├── infrastructure/
│   ├── 3_setup.md
│   ├── trust_policy.json
│   ├── bucket_policy.json
│   ├── iam_policy.jsons
├── snowflake/
│   ├── snowflake_setup.sql
│   ├── data_loading.sql
│   ├── post_load_validation.sql
├── data/
│   ├── Consumer_Complaintss.csv
└── scripts/
    ├── data_cleaning.sql
    ├── etl_process.md

### 1. **AWS Components**
- **IAM Role & Trust Policy**: 
  - `iam_policy.json`: Defines permissions for accessing S3.
  - `trust_policy.json`: Configures trust between AWS IAM and Snowflake.
  - `bucket_policy.json`: Sets access control for S3 Bucket.
  - `s3_setup.md`: Step-by-step guide to setting up S3 bucket permissions and uploading data.
  
### 2. **Snowflake Components**
- **snowflake_setup.sql**:
  
  -- Set Context explicitly
USE WAREHOUSE DWH_CLIENT;
USE DATABASE CLIENT_DATA;
USE SCHEMA DWH_PROD;

-- Step 1: Create Warehouse (efficient settings)
CREATE OR REPLACE WAREHOUSE DWH_CLIENT 
    WAREHOUSE_SIZE = 'XSMALL'
    AUTO_SUSPEND = 300
    AUTO_RESUME = TRUE;

-- Step 2: Create Database and Schema
CREATE OR REPLACE DATABASE CLIENT_DATA;
CREATE OR REPLACE SCHEMA CLIENT_DATA.DWH_PROD;

-- Step 3: Create Storage Integration (Correct ARN and path)
CREATE OR REPLACE STORAGE INTEGRATION SNOW_OBJ
    TYPE = EXTERNAL_STAGE
    STORAGE_PROVIDER = 'S3'
    ENABLED = TRUE
    STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::908027391407:role/Snowflake'
    STORAGE_ALLOWED_LOCATIONS = ('s3://isaclient/');

-- Verify Storage Integration configuration
DESC INTEGRATION SNOW_OBJ;

-- Step 4: File Format Definition (robust CSV handling)
CREATE OR REPLACE FILE FORMAT CLIENT_DATA.DWH_PROD.CSV_FORMAT
    TYPE = CSV
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1
    NULL_IF = ('NULL', 'null', '')
    EMPTY_FIELD_AS_NULL = TRUE
    FIELD_OPTIONALLY_ENCLOSED_BY = '"'
    TRIM_SPACE = TRUE
    ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE
    DATE_FORMAT = 'AUTO'
    TIMESTAMP_FORMAT = 'AUTO';

-- Step 5: External Stage (targeted to specific CSV folder if needed)
CREATE OR REPLACE STAGE CLIENT_DATA.DWH_PROD.SNOW_STAGE_2023
    STORAGE_INTEGRATION = SNOW_OBJ
    URL = 's3://isaclient/'
    FILE_FORMAT = CLIENT_DATA.DWH_PROD.CSV_FORMAT;

-- Step 6: Create Table with optimized data types and sizes
CREATE OR REPLACE TABLE CLIENT_DATA.DWH_PROD.Consumer_Complaintss (
    CustomerId INT,
    Surname VARCHAR(100),
    CreditScore INT,
    Firstname VARCHAR(100),
    Gender VARCHAR(20),
    Age FLOAT,
    Tenure INT,
    EstimatedSalary VARCHAR(50), -- Stored as VARCHAR due to currency format (€ symbol and commas)
    NumOfProducts INT,
    HasCrCard VARCHAR(10),
    IsActiveMember VARCHAR(10)
);

-- Optional: Verify files staged (recommended before COPY)
LIST @CLIENT_DATA.DWH_PROD.SNOW_STAGE_2023;

-- Step 7: COPY data into table with robust error handling
COPY INTO CLIENT_DATA.DWH_PROD.Consumer_Complaintss
FROM @CLIENT_DATA.DWH_PROD.SNOW_STAGE_2023/Consumer_Complaintss.csv
FILE_FORMAT = (FORMAT_NAME = 'CLIENT_DATA.DWH_PROD.CSV_FORMAT')
ON_ERROR = 'CONTINUE'  -- continue loading despite errors; alternative: 'ABORT_STATEMENT'
PURGE = FALSE; -- set TRUE if you want to delete files post-load

-- Step 8: Post-load validation checks
SELECT COUNT(*) AS Rows_Loaded FROM CLIENT_DATA.DWH_PROD.Consumer_Complaintss;

-- Review COPY history and errors
SELECT * FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME=>'Consumer_Complaintss', 
  START_TIME=>DATEADD('hours', -1, CURRENT_TIMESTAMP())
)); 
  - `snowflake_stage_screenshot.  <img width="510" alt="image" src="https://github.com/user-attachments/assets/d2fa1b93-46bc-413f-8189-a355fcacf428" />

  
### 3. **Data Cleaning & Processing in Snowflake**
-- Step 1 Data cleaning: Add a new numeric column to store cleaned salary values
ALTER TABLE CLIENT_DATA.DWH_PROD.Consumer_Complaintss
ADD COLUMN EstimatedSalary_Clean FLOAT;

-- Step 2: Clean and populate the new numeric column
UPDATE CLIENT_DATA.DWH_PROD.Consumer_Complaintss
SET EstimatedSalary_Clean = 
    TRY_CAST(
        REPLACE(
            REPLACE(EstimatedSalary, '€', ''), 
            ',', ''
        ) AS FLOAT
    );

-- Verify data transformation
SELECT CustomerId, EstimatedSalary, EstimatedSalary_Clean
FROM CLIENT_DATA.DWH_PROD.Consumer_Complaintss
LIMIT 10;

<img width="672" alt="image" src="https://github.com/user-attachments/assets/41807fce-a0a0-4887-8ed2-880f5d7505bb" />
<img width="938" alt="image" src="https://github.com/user-attachments/assets/b31b0ea1-86ae-4704-9374-a89e9ca437f9" />
<img width="877" alt="image" src="https://github.com/user-attachments/assets/f5e94b94-b25f-4f12-adf0-c90608aeb82d" />

## Key Skills Highlighted
✅ **AWS IAM & Security**: Setting up and managing permissions, trust policies.
✅ **AWS S3**: Bucket creation, data upload, and security controls.
✅ **Snowflake Integration**: Creating external stages, warehouses, databases, schemas, and file formats.
✅ **Data Transformation & Cleaning**: SQL-based transformations to prepare structured data.

## Usage & Setup Guide
1. **Set up AWS S3** following `s3_setup.md`.
2. **Create IAM Role** and assign the correct trust policy to enable connection to Snowflake.
3. **Configure Snowflake** using `snowflake_setup.sql`.
4. **Stage and Load Data** with `data_loading.sql`.
5. **Verify Data** using `post_load_validation.sql`.
6. **Clean & Transform Data** using `data_cleaning.sql`.
