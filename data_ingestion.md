# Azure Synapse Pipeline: Detailed Documentation

## Table of Contents

1. [Solution](#solution)  
2. [Pipeline Overview](#pipeline-overview)  
3. -[Pipeline Steps](#pipeline-steps)  
   - [Retrieve API Key (Get Key)](#1-retrieve-api-key-get-key)  
   - [Retrieve API Username (Get User Name from Vault)](#2-retrieve-api-username-get-user-name-from-vault)  
   - [Retrieve API Password (Get Password from Vault)](#3-retrieve-api-password-get-password-from-vault)  
   - [Download Themes Data (Download Themes Data)](#4-download-themes-data)  
   - [Download Sets Data (Download Sets Data)](#5-download-sets-data)  
4. [Security Considerations](#security-considerations)  
5. [Execution Flow](#execution-flow)

---

## Solution

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

## Pipeline Steps

### 1. Retrieve API Key (Get Key)

- **Activity Type:** Web Activity  
- **Description:** Fetches the API key from Azure Key Vault to authenticate API requests.  
  - **HTTP Method:** `GET`  
  - **URL:** `https://dp203.vault.azure.net/secrets/Rebrickable?api-version=7.0`  
  - **Authentication:** Managed Service Identity (MSI)  
  - **Runtime Environment:** `AutoResolveIntegrationRuntime`  
  - **Timeout:** 12 hours  
  - **Retries:** 0 retries, 30-second retry interval

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

### 4. Download Themes Data (Download Themes Data)

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

### 5. Download Sets Data (Download Sets Data)

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

---

## Security Considerations

1. **Managed Identity (MSI):** The pipeline uses Azure Managed Service Identity to securely access Azure Key Vault. This approach eliminates the need for storing sensitive credentials in the pipeline configuration.
2. **Secure Key Retrieval:** API credentials (key, username, and password) are dynamically fetched from Azure Key Vault at runtime, ensuring that sensitive information is not hardcoded or exposed.
3. **Confidential Output:** Input and output of web activities can be marked as secure to prevent logging of sensitive information.

---

## Execution Flow

1. **Credential Fetching:**  
   The pipeline first retrieves the API key, username, and password from Azure Key Vault. Each step is dependent on the success of the previous one, ensuring the credentials are fetched securely in sequence.

2. **Data Extraction:**  
   After retrieving the credentials, the pipeline proceeds with two `Copy Activities` to download the themes and sets data from the Rebrickable API.

3. **Data Storage:**  
   The extracted data is stored in Azure Blob Storage in JSON format for further processing or analysis.

