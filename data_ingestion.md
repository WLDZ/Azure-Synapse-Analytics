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


## Pipeline Overview

- **Pipeline Name:** `Ingest`
- **Type:** `Microsoft.Synapse/workspaces/pipelines`
- **Last Published:** `2024-10-13`

The pipeline is designed to securely extract theme and set data from the Rebrickable API and store it in Azure Blob Storage in JSON format. It fetches the necessary credentials from Azure Key Vault and handles data ingestion through a series of REST API calls.

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

The following image shows the overall structure of the final data ingestion pipeline. <br>

   ![Pipeline](images/synapse.png)

**In the same way, the detials of user profile were fecthed by using the user endpoints provided by Rebrickable api.**

---

## Summary

The **Ingest** pipeline provides a secure, efficient solution for extracting data from the Rebrickable API. It leverages Azure Key Vault and Managed Service Identity (MSI) for secure credential management and uses REST-based API calls for data extraction. With well-defined retry and timeout policies, the pipeline ensures reliable and secure data ingestion into Azure Blob Storage for future use.

**[Go to Next Section](./solution.md)**
