# Azure Synapse Pipeline: Detailed Documentation

## Table of Contents

1. [Overview](#Overview)  
2. [Pipeline Overview](#pipeline-overview)  
   2.0 - [Pipeline Steps](#pipeline-steps)  
   2.1 - [Retrieve API Key (Get Key)](#21-retrieve-api-key-get-key)  
   2.2 - [Retrieve API Username (Get User Name from Vault)](#22-retrieve-api-username-get-user-name-from-vault)  
   2.3 - [Retrieve API Password (Get Password from Vault)](#23-retrieve-api-password-get-password-from-vault)  
   2.4 - [Download Themes Data (Download Themes Data)](#24-download-themes-data)  
   2.5 - [Download Sets Data (Download Sets Data)](#25-download-sets-data)  
   2.6 - [Security Considerations](#26-security-considerations)  
3. [Data Load Process in Dimension and Fact Tables](#data-load-process-in-dimension-and-fact-tables)  
   3.1 - [Environment Setup](#31-environment-setup)  
   3.2 - [Loading and Processing JSON Files](#32-loading-and-processing-json-files)  
   3.3 - [Data Insertion into Dimension and Fact Tables](#33-data-insertion-into-dimension-and-fact-tables)  
   3.3.1 - [Inserting into Lego_Sets_Dimension](#331-inserting-into-lego_sets_dimension)  
   3.3.2 - [Inserting into Rebrickable_Profile_Dimension](#332-inserting-into-rebrickable_profile_dimension)  
   3.3.3 - [Inserting into Lego_Date_Dimension](#333-inserting-into-lego_date_dimension)  
   3.4 - [Inserting into the Fact Table: Owned_Sets_Fact](#34-inserting-into-the-fact-table-owned_sets_fact)  
   3.5 - [Use of createOrReplaceTempView](#35-use-of-createorreplacetempview)  
   3.6 - [Data Validation Using SQL Queries](#36-data-validation-using-sql-queries)  
   3.7 - [Register the Tables in a Catalog](#37-register-the-tables-in-a-catalog)  
   3.8 - [Final Structure of Tables in Data Lake](#38-final-structure-of-tables-in-data-lake)  
   3.9 - [Conclusion](#39-conclusion)

---

## Overview

The overall solution is divided into three parts:

### 1. Data Ingestion from Rebrickable Website to Raw Layer

This part focuses on extracting data from the Rebrickable website. The pipeline is designed to securely retrieve theme and set data from the Rebrickable API and store it in the raw layer of the Azure Data Lake in JSON format. Credentials for accessing the API are securely managed using Azure Key Vault.

### 2. Data Warehouse Structure Creation

In this phase, the data warehouse is designed and structured, including setting up tables, defining schemas, and applying any business rules. This warehouse provides the foundation for storing processed data. Further details are provided [here](./table_creation.md).

### 3. Data Transformation and Loading into the Curated Layer

After ingestion, data is transformed according to business requirements. This process includes cleaning, enriching, and structuring the data. The transformed data is then loaded into the curated layer of the data lake, making it available for reporting and analytics. The detailed solution can be found [here](./solution.md).

---

## Pipeline Overview

The following diagram shows the overall structure of the data ingestion pipeline:

![Pipeline](images/synapse.png)

The pipeline is built to securely extract themes and sets data from the Rebrickable API, fetch the necessary credentials from **Azure Key Vault**, and store the extracted data in Azure Data Lake in JSON format. It consists of several sequential steps to ensure that data retrieval and storage are performed reliably.

---

### 2.0 - Pipeline Steps

#### 2.1 - Retrieve API Key (Get Key)

- **Activity Type:** Web Activity  
- **Description:** Fetches the API key from Azure Key Vault to authenticate API requests.  
  - **HTTP Method:** `GET`  
  - **URL:** `https://dp203.vault.azure.net/secrets/Rebrickable?api-version=7.0`  
  - **Authentication:** Managed Service Identity (MSI)  
  - **Runtime Environment:** `AutoResolveIntegrationRuntime`  
  - **Timeout:** 12 hours  
  - **Retries:** 0 retries, 30-second retry interval

#### 2.2 - Retrieve API Username (Get User Name from Vault)

- **Activity Type:** Web Activity  
- **Description:** Fetches the API username from Azure Key Vault.  
  - **HTTP Method:** `GET`  
  - **URL:** `https://dp203.vault.azure.net/secrets/REBRICKABLE-API-UNAME?api-version=7.0`  
  - **Authentication:** Managed Service Identity (MSI)  
  - **Depends On:** Success of "Get Key" activity  
  - **Runtime Environment:** `AutoResolveIntegrationRuntime`  
  - **Timeout:** 12 hours  
  - **Retries:** 0 retries, 30-second retry interval

#### 2.3 - Retrieve API Password (Get Password from Vault)

- **Activity Type:** Web Activity  
- **Description:** Fetches the API password from Azure Key Vault.  
  - **HTTP Method:** `GET`  
  - **URL:** `https://dp203.vault.azure.net/secrets/REBRICKABLE-API-PASSWORD?api-version=7.0`  
  - **Authentication:** Managed Service Identity (MSI)  
  - **Depends On:** Success of "Get User Name from Vault" activity  
  - **Runtime Environment:** `AutoResolveIntegrationRuntime`  
  - **Timeout:** 12 hours  
  - **Retries:** 0 retries, 30-second retry interval

#### 2.4 - Download Themes Data (Download Themes Data)

- **Activity Type:** Copy Activity  
- **Description:** Downloads theme data from the Rebrickable API and stores it in Azure Blob Storage as JSON.  
  - **Source:**
    - **Type:** `RestSource`
    - **HTTP Method:** `GET`
    - **Authorization:** Dynamically generated using the API key.
    - **Pagination:** Follows `$.next` for paginated responses.
    - **Timeout:** 1 minute 40 seconds
    - **Request Interval:** 1 second
  - **Sink:**
    - **Type:** `JsonSink`
    - **Storage Type:** Azure Blob FS
    - **Data Format:** JSON
  - **Depends On:** Success of "Get Password from Vault"  
  - **Timeout:** 12 hours  
  - **Retries:** 0 retries, 30-second retry interval

#### 2.5 - Download Sets Data (Download Sets Data)

- **Activity Type:** Copy Activity  
- **Description:** Downloads set data from the Rebrickable API and stores it in Azure Blob Storage as JSON.  
  - **Source:**
    - **Type:** `RestSource`
    - **HTTP Method:** `GET`
    - **Authorization:** API key dynamically generated.
    - **Pagination:** Uses `$.next` for handling paginated data.
    - **Timeout:** 1 minute 40 seconds
    - **Request Interval:** 2 seconds
  - **Sink:**
    - **Type:** `JsonSink`
    - **Storage Type:** Azure Blob FS
    - **Data Format:** JSON
  - **Depends On:** Success of "Download Themes Data"  
  - **Timeout:** 12 hours  
  - **Retries:** 0 retries, 30-second retry interval

#### 2.6 - Security Considerations

1. **Managed Identity (MSI):** The pipeline uses Azure Managed Service Identity to securely access Azure Key Vault. This approach eliminates the need for storing sensitive credentials in the pipeline configuration.
2. **Secure Key Retrieval:** API credentials (key, username, and password) are dynamically fetched from Azure Key Vault at runtime, ensuring that sensitive information is not hardcoded or exposed.
3. **Confidential Output:** Input and output of web activities can be marked as secure to prevent logging of sensitive information.

---

# Data Load Process in Dimension and Fact Tables

## Introduction
This section explains the process of loading data into dimension and fact tables in a data warehouse using Spark with PySpark and Azure Synapse. The notebook utilizes various Spark transformations and SQL commands to merge, read, and process data from JSON files into relevant tables. The approach ensures that the data is effectively handled for analytics purposes.

### 1. Environment Setup
The notebook begins by setting up the Spark environment. This involves defining the Spark pool that provides the required computational resources.

```python 
spark_pool = 'MySparkPool'
``` 
Explanation:
- This line specifies the name of the Spark pool as MySparkPool. The resources for running the Spark jobs will be drawn from this pool.
### 2. Loading and Processing JSON Files
The notebook retrieves JSON files from an Azure Data Lake storage location. These files are then processed and prepared for insertion into dimension and fact tables.

#### 2.1 Listing Files in the Azure Data Lake Folder
First, the code lists all the files in a specified folder of the Azure Data Lake using mssparkutils.fs.ls.

```python 
files = mssparkutils.fs.ls("abfss://raw@adlg2dev.dfs.core.windows.net/SomeFolder")
```
Explanation:
- mssparkutils.fs.ls: Lists all the files in the specified Azure Data Lake folder.
- The path "abfss://raw@adlg2dev.dfs.core.windows.net/SomeFolder" specifies the storage location.
- Dummy Information: Folder path: abfss://raw@some_account_name.dfs.core.windows.net/DummyFolder

#### 2.2 Reading and Exploding JSON Files
Next, the notebook reads each file and processes its contents. This involves exploding any nested arrays and adding additional metadata, such as the user and date_added.

```python 
for file in files:
    temp_df = (spark.read
               .option("multiline", "true")
               .format('json')
               .load(file.path)
               .withColumn("explodedArray", explode(col('results')))
               .withColumn("user", lit(file.name.split('.')[0]))
               .withColumn("date_added", current_timestamp()))
```
Explanation:
- spark.read: Reads the files in JSON format.
- option("multiline", "true"): Ensures that multi-line JSON files are read correctly. 
- explode(col('results')): Breaks down the nested results array in each file into individual rows.  
- withColumn("user", lit(file.name.split('.')[0])): Adds a new column user using the filename (before the extension).
- withColumn("date_added", current_timestamp()): Adds a column for the current timestamp when the data is added. 
- This step transforms the data into a format suitable for insertion into the target tables.


#### 2.3 Creating a Temporary View for SQL Queries
After processing the data into a DataFrame, the notebook creates a temporary view of the data. This allows the data to be accessed using SQL.

```python 
temp_df.createOrReplaceTempView("source_data")
``` 
Explanation:
- createOrReplaceTempView("source_data"): Registers the DataFrame temp_df as a temporary SQL view called source_data. This makes the data available for querying using SQL commands.
### 3. Data Insertion into Dimension and Fact Tables
Once the data is ready, it is inserted into dimension and fact tables. This involves using the MERGE INTO statement to update existing records or insert new ones.

#### 3.1 Dimension Tables
The dimension tables store reference data to support the fact table. The code performs upserts (merge operations) into various dimension tables, ensuring they are populated to cater to **SC 1**.

##### 3.1.1 Inserting into Lego_Sets_Dimension
The Lego_Sets_Dimension table stores information about Lego sets. The following SQL query merges data into this dimension table:

```python 
spark.sql("""
    MERGE INTO Lego.Lego_Sets_Dimension AS target
    USING updated_keys AS source
    ON target.SetNumber = source.SetNumber
    WHEN MATCHED THEN
        UPDATE SET target.Set_Dim_Key = source.Set_Dim_Key
    WHEN NOT MATCHED THEN
        INSERT (SetNumber, Set_Dim_Key)
        VALUES (source.SetNumber, source.Set_Dim_Key)
""")
```
Explanation:
- MERGE INTO: Performs an upsert operation (i.e., update or insert).
- WHEN MATCHED THEN UPDATE: Updates existing records where SetNumber matches.
- WHEN NOT MATCHED THEN INSERT: Inserts new records if no match is found on SetNumber.


The following images show the populated data in the Lego_Sets_Dimension. The second image clearly displays the name of each theme, which is fetched from the themes dataset. <br>

![sets](images/sets.png)

<br>
<br>

![sets2](images/set2.png)

<br>

##### 3.1.2 Inserting into Rebrickable_Profile_Dimension
The Rebrickable_Profile_Dimension table stores information about user profiles and their associated LEGO sets.

```python 
spark.sql("""
        MERGE INTO lego.Rebrickable_Profile_Dimension AS rp
        USING (
            SELECT DISTINCT c.user AS User_ID, c.list_id, c.name AS Set_Name, ...
        )
        ON rp.Set_Num = c.set_num
           AND rp.List_ID = c.list_id
        WHEN MATCHED AND (
            rp.User_ID <> c.User_ID OR ...
        ) THEN
            UPDATE ...
""")
```
This query performs an upsert to update or insert user profile records. When matched, it updates the User_ID, Set_Name, and Set_Num fields if there are differences. If no match is found, a new record is inserted.

Table: Rebrickable_Profile_Dimension
Columns: User_ID, List_ID, Set_Name, Set_Num, Date_Added

<br>

![rebrickable_profile](images/rebrickable_profile.png)

<br>

##### 3.1.3 Inserting into Lego_Date_Dimension
The Lego_Date_Dimension table stores date-related information, typically used to track transactions in the fact table.
```python 
spark.sql("""
    MERGE INTO Lego.Lego_Date_Dimension AS target
    USING date_keys AS source
    ON target.Date_ID = source.Date_ID
    WHEN MATCHED THEN
        UPDATE SET target.Date_Dim_Key = source.Date_Dim_Key
    WHEN NOT MATCHED THEN
        INSERT (Date_ID, Date_Dim_Key)
        VALUES (source.Date_ID, source.Date_Dim_Key)
""")
```
Explanation:
- Similar to the other dimension tables, this query performs an upsert to update or insert date records.
- The following images show the populated data in the Lego_Date_Dimension.

<br>

![date_dimension](images/date_dimension.png)

<br>

#### 3.2 Inserting into the Fact Table: Owned_Sets_Fact
The fact table stores transactional data about Lego sets, linking users, sets, and dates. The code inserts new records into this table:

```
spark.sql("""
    INSERT INTO Lego.Owned_Sets_Fact (Owned_Fact_ID,Set_ID, User_ID, Date_ID, Price)
    SELECT Owned_Fact_ID,Set_ID, User_ID, Date_ID, Price
    FROM source_data
""")
```
Explanation:
- INSERT INTO: Inserts new records into the fact table.
- The SELECT statement retrieves the necessary fields (SetNumber, User_ID, Date_ID, Price) from the source_data temp view.

<br>

![fact](images/fact.png)

<br>

### 4. Use of createOrReplaceTempView
The function createOrReplaceTempView allows a DataFrame to be registered as a temporary SQL view, making it accessible via SQL queries. In the code, this is used to enable SQL-based operations on the DataFrame data.

```python 
temp_df.createOrReplaceTempView("source_data")
```
Explanation:
createOrReplaceTempView creates a temporary view called source_data from the DataFrame temp_df. This enables the data to be queried directly using SQL, such as in the fact table insertion described above.
### 5. Data Validation Using SQL Queries
After data insertion, validation queries are run to verify the data was successfully loaded into the **Fact** table.

```python

spark.sql("SELECT * FROM Lego.Owned_Sets_Fact").show()
```
Explanation:
- This query retrieves and displays the contents of the Owned_Sets_Fact table, showing the inserted records. Following images shows the data contained in it. 
<br>

![fact](images/fact.png)

<br>

### 6. Register the Tables in a Catalog:
Azure Synapse Analytics was used to register the tables in a catalog. This allows SQL queries to be executed using table names without needing to reference the file paths directly, simplifying data access and query execution. The following image shows how all the tables look after being registered in the catalog.


<br>

![catalog](images/catalog.png)

<br>

### 7. Final Structure of tables in Data Lake
The final structure of the dimensional and fact tables is shown in the image below. This structure has been kept simple, as it utilizes Delta format underneath, which facilitates efficient data management and query performance.

<br>

![partition](images/partition.png)

<br>

### 7. Conclusion
This notebook automates the process of loading, merging, and inserting data into dimension and fact tables using PySpark and SQL in an Azure Synapse environment. The dimension tables are populated to cater to SC 1, ensuring that reference data is available to support the fact table. This process guarantees data consistency and efficient loading from the raw files stored in an Azure Data Lake to the appropriate data warehouse tables.

**[Infromation on Warehouse Creation](./table_creation.md)**



