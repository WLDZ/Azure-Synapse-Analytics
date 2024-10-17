# Azure Synapse Pipeline: Detailed Documentation

## Table of Contents

0. [Solution](#Solution)
1. [Pipeline Overview](#pipeline-overview)
2. [Pipeline Steps](#pipeline-steps)
   - [Retrieve API Key (Get Key)](#1-retrieve-api-key-get-key)
   - [Retrieve API Username (Get User Name from Vault)](#2-retrieve-api-username-get-user-name-from-vault)
   - [Retrieve API Password (Get Password from Vault)](#3-retrieve-api-password-get-password-from-vault)
   - [Download Themes Data (Download and Themes Data)](#4-download-themes-data-download-and-themes-data)
   - [Download Sets Data (Download Sets Data)](#5-download-sets-data-download-sets-data)
3. [Security Considerations](#security-considerations)
4. [Execution Flow](#execution-flow)
5. [Summary](#summary)

---

## Solution

The solution is divided into three parts:

#### 1. Data Ingestion from Rebrickable Website and Storing in Raw Layer

The first part involves extracting data from the Rebrickable website and rest of this page will discuss this part in detail.

#### 2. Data Warehouse Structure Creation

In this phase, we design and create the structure of the data warehouse where the processed data will be housed. This includes setting up necessary tables, defining schemas, and applying any relevant business rules. The warehouse serves as the structured environment for storing transformed data. Following is the link to the data warehouse creation process for this activity. <br>
[Data Warehouse](./table_creation.md)

#### 3. Data Transformation and Loading into Curated Layer

The final part involves loading the raw data from the data lake, performing transformations to clean, enrich, and structure the data according to business requirements. This transformed data is then loaded into the curated layer of the data lake. The curated layer serves as the final repository from which data can be accessed and served to meet business needs. Following is the link to the final activity of the whole process. <br>
[Solution](./solution.md)

---

<br>
<br>

## Pipeline Overview

The following image shows the overall structure of the final data ingestion pipeline. <br>

![Pipeline](images/synapse.png)

The pipeline is designed to securely extract theme and set data from the Rebrickable API and store it in Azure Datalake in JSON format. It fetches the necessary credentials from **Azure Key Vault** and handles data ingestion through a series of REST API calls.

---

## Pipeline Steps

### 1. Retrieve API Key (Get Key)

- **Activity Type:** Web Activity
- **Description:** Fetches the API key from Azure Key Vault to authenticate subsequent API requests.

  - **HTTP Method:** `GET`
  - **URL:** `https://dp203.vault.azure.net/secrets/Rebrickable?api-version=7.0`
  - **Authentication:** Managed Service Identity (MSI)
  - **Runtime Environment:** `AutoResolveIntegrationRuntime`
  - **Timeout:** 12 hours
  - **Retries:** 0 retries, 30-second retry interval

---

### 2. Retrieve API Username (Get User Name from Vault)

- **Activity Type:** Web Activity
- **Description:** Fetches the API username from Azure Key Vault.

  - **HTTP Method:** `GET`
  - **URL:** `https://dp203.vault.azure.net/secrets/REBRICKABLE-API-UNAME?api-version=7.0`
  - **Authentication:** Managed Service Identity (MSI)
  - **Depends On:** Success of "Get Key" activity
  - **Runtime Environment:** `AutoResolveIntegrationRuntime`
  - **Timeout:** 12 hours
  - **Retries:** 0 retries, 30-second retry interval

---

### 3. Retrieve API Password (Get Password from Vault)

- **Activity Type:** Web Activity
- **Description:** Fetches the API password from Azure Key Vault.

  - **HTTP Method:** `GET`
  - **URL:** `https://dp203.vault.azure.net/secrets/REBRICKABLE-API-PASSWORD?api-version=7.0`
  - **Authentication:** Managed Service Identity (MSI)
  - **Depends On:** Success of "Get User Name from Vault" activity
  - **Runtime Environment:** `AutoResolveIntegrationRuntime`
  - **Timeout:** 12 hours
  - **Retries:** 0 retries, 30-second retry interval

---

### 4. Download Themes Data (Download and Themes Data)

- **Activity Type:** Copy Activity
- **Description:** Downloads themes data from the Rebrickable API and stores it in Azure Blob Storage as JSON.

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

---

### 5. Download Sets Data (Download Sets Data)

- **Activity Type:** Copy Activity
- **Description:** Downloads sets data from the Rebrickable API and stores it in Azure Blob Storage as JSON.

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

  - **Depends On:** Success of "Download and Themes Data"
  - **Timeout:** 12 hours
  - **Retries:** 0 retries, 30-second retry interval

---

## Security Considerations

1. **Managed Identity (MSI):** The pipeline uses Azure Managed Service Identity to securely access Azure Key Vault. This eliminates the need for storing credentials in the pipeline configuration.
2. **Secure Key Retrieval:** API credentials (key, username, and password) are dynamically fetched from Azure Key Vault during runtime, ensuring that sensitive information is not hardcoded or exposed.

3. **Confidential Output:** Input and output of web activities are non-secure by default but can be configured as secure to prevent logging sensitive information.

---

## Execution Flow

1. **Credential Fetching:**

   - The pipeline starts by retrieving the API key, username, and password from Azure Key Vault in three consecutive steps.
   - Each step depends on the successful execution of the previous one, ensuring that credentials are fetched in the correct order.

2. **Data Extraction:**

   - After retrieving the credentials, two `Copy Activities` are executed to download theme and set data from the Rebrickable API.
   - The data is stored in Azure Blob Storage in JSON format.

3. **Data Storage:**
   - Both the themes and sets data are stored in Azure Blob Storage in JSON format, making the data available for further analysis or processing.

**In the same way, the detials of user profile were fecthed by using the user endpoints provided by Rebrickable api.**

---

## Summary

The **Ingest** pipeline provides a secure, efficient solution for extracting data from the Rebrickable API. It leverages Azure Key Vault and Managed Service Identity (MSI) for secure credential management and uses REST-based API calls for data extraction. With well-defined retry and timeout policies, the pipeline ensures reliable and secure data ingestion into Azure Blob Storage for future use.

# Data Load Process in Dimension and Fact Tables

## Introduction

This document explains the process of loading data into dimension and fact tables in a data warehouse using Spark with PySpark and Azure Synapse. The notebook utilizes various Spark transformations and SQL commands to merge, read, and process data from JSON files into relevant tables. The approach ensures that the data is effectively handled for analytics purposes.

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
