# LEGO Data Ingestion Pipeline Using Azure Data Factory


## Table of Contents
1. Overview of Resource Groups
2. Development Environment Resources
3. Production Environment Resources
4. Data Ingestion and Process Flow
5. Setup Guide
6. Customization


## Overview

This Azure Data Factory (ADF) pipeline is designed to automate the ingestion of LEGO data files and user-specific data from the Rebrickable API into **Azure Data Lake Storage (ADLS)**. The pipeline securely retrieves credentials from **Azure Key Vault**, handles large data volumes with **pagination** and **throttling**, and includes error-handling mechanisms. Additionally, a **Logic App** is used to send custom notifications to the admin in case of failures or critical events.

## 1. Overview of Resource Groups

The image illustrates three Azure Resource Groups associated with an Azure subscription:

- **DP203_PROD** (Production)
- **DP203_DEV** (Development)
- **SHAREDRG** (Shared)

Each resource group holds specific resources required for different stages of the workflow:

- **DP203_PROD**: Contains all necessary resources for tasks in the production environment.
- **DP203_DEV**: Contains all necessary resources for tasks in the development environment.
- **SHAREDRG**: Contains shared resources such as Logic Apps, which handle error management for both the development and production environments.

![Resource Groups](images/resource%20groups.png)

## 2. Development Environment Resources

In the development environment, three core resources are used for data integration, storage, and security:

- **adfdev203** (Data Factory V2): Handles data integration and transformation processes.
- **adlg2dev** (Storage Account): Manages the storage needs for the development environment.
- **DP203** (Key Vault): Stores and manages secrets, such as keys and connection strings, securely.

![Development Resources](images/dev.png)

## 3. Production Environment Resources

In the production environment, two core resources are used:

- **adf203prod** (Data Factory V2): Handles data integration and transformation processes.
- **adlg2prod** (Storage Account): Manages the storage needs for the production environment.
- **DP203** (Key Vault) from the development environment will be used to fetch all secrets required in production.

![Production Resources](images/prod.png)

## 4. Shared Resource Group

In the shared resource group, one key resource is utilized:

- **dp203logicapp** (Logic App): This Logic App is used by both the development and production environments to handle failure notifications via emails.

![Logic App](images/logicapp.png)

## 5. Git Repository Overview

The image shows the Git repository for the project named **Rebrickable_CICD**. This repository contains folders and files related to Data Factory configurations and DevOps setup.

The directory structure includes:

- **data-factory**: Contains datasets, factory settings, linked services, pipelines, triggers, and a publish configuration file.
- **devops**: Contains pipeline and job YAML files, `package.json`, and `azure-pipelines.yml`.

![DevOps Repository](images/devopps.png)

## 6. Azure DevOps CI/CD Pipeline

The following image illustrates the CI/CD pipeline for the **Rebrickable_CICD** repository, specifically on the main branch. The pipeline has three stages:

- **BUILD**: Status - SUCCESS
- **DEV**: Status - SUCCESS
- **PROD**: Status - SUCCESS

![CI/CD Pipeline](images/CICD.png)

## 7. Git Configuration in Development Data Factory

The following image shows the Git configuration for the development Data Factory (`adfdev203`). This configuration is essential for maintaining version control and enabling CI/CD workflows.

![Git Configuration](images/git.png)

## 8. Linked Services

The following image shows all the linked services required for the data integration tasks. These linked services allow communication between Data Factory and various Azure services or external APIs:

- **ADLG2**: Type - Azure Data Lake Storage Gen2
- **AzureKeyVault**: Type - Azure Key Vault
- **RebrickableAPI**: Type - REST
- **RebrickableHTTP**: Type - HTTP

![Linked Services](images/linked_services.png)

## 9. Final Pipeline Workflow

The following image represents the final workflow of the pipeline, covering all the tasks and configurations required for the data integration process. The explanation of this workflow is detailed in the following sections.

![Final Pipeline](images/final%20pipeline.png)

#Section 2 - Pipeline Breakdown

### 1. **Download LEGO Zip Files**

- **Activity**: `ForEach`
- **Description**: This step downloads zipped LEGO data files such as `themes.csv.gz` and `colors.csv.gz` from the Rebrickable API.
- **Storage**: The files are stored in **Azure Data Lake Storage (ADLS)** in binary format.

### 2. **Get API Key, Username, and Password from Key Vault**

- **Activities**:
  - `Get Key For Key Vault`
  - `Get Username From Key Vault`
  - `Get Password From Key Vault`
- **Description**: These steps securely retrieve the API Key, Username, and Password from **Azure Key Vault** using **Managed Service Identity (MSI)**. This ensures that sensitive credentials are not hardcoded and are accessed securely at runtime.

### 3. **Generate User Token**

- **Activity**: `Generate User Token`
- **Description**: This step generates a user token for Rebrickable API authentication. The token is then used in subsequent API requests to fetch user-specific data.

### 4. **Looping Through Users and Fetching Data**

- **Activity**: `Loop Over Users`
- **Description**: This step iterates over a list of users defined in the `userlist` variable (e.g., `allparts`, `partlists`). It sends a GET request to the Rebrickable API for each user, fetching relevant LEGO part lists or inventory data.
- **Pagination and Throttling**: The pipeline automatically handles **pagination** by following the next page links provided in the API response. It also manages **API throttling** to avoid hitting rate limits, ensuring smooth data fetching without errors.
- **Storage**: The user-specific data is stored in **ADLS** in **JSON format**, with directories organized by date to support easy retrieval and further processing.

### 5. **Error Handling with Failure Pipeline**

- **Activity**: `Failure Pipeline`
- **Description**: If any activity fails, the pipeline triggers a **Failure Pipeline**. This ensures that the error is logged and proper failure workflows are initiated.

### 6. **Custom Admin Notification via Logic App**

- **Logic App**: A **Logic App** is configured to send a custom alert to the admin when failures occur, or when other critical events require attention. The alert message includes details about the pipeline's execution status, such as:
  - Failed activity
  - Timestamp of the failure
  - Possible reasons for the failure

## Pipeline Activities and Flow

### **Activities Overview**:

- **ForEach (Download All Zip Files)**: Downloads LEGO zip files from the Rebrickable API.
- **WebActivity (Key Vault)**: Retrieves credentials (API key, username, password) from Azure Key Vault.
- **WebActivity (Token Generation)**: Generates a token to authenticate further API requests.
- **ForEach (Loop Over Users)**: Loops through the list of users and retrieves data.
- **Failure Pipeline**: Executes if any activity fails during the pipeline run.

### **Failure Handling**:

- **Wait and Error Steps**: The pipeline uses a **Wait** activity as a dummy step to control the flow and ensure that dependent activities run only if prior steps succeed. If a failure occurs, the **Failure Pipeline** is triggered, and the **Logic App** sends alerts to the admin.

## Variables

- **result**: Holds the list of LEGO zip files to download (e.g., `themes.csv.gz`, `colors.csv.gz`).
- **userlist**: Contains the user-specific identifiers for API data retrieval (e.g., `allparts`, `partlists`).
- **token**: Stores the user token generated for API authentication.
- **value** and **uservalue**: Temporary placeholders for dynamic values during execution.

## Error Management and Notifications

- **Error Handling**: The pipeline includes robust error management through the **Failure Pipeline**, which is executed when any step fails.
- **Admin Notifications**: A **Logic App** is configured to send custom error notifications to the admin, ensuring prompt action in case of pipeline failures. The message includes details about the failed activity, the reason for failure, and the time of occurrence.

## Key Features

1. **Secure Credential Management**:

   - API keys and passwords are securely stored in **Azure Key Vault** and accessed using **Managed Service Identity (MSI)**, ensuring security and compliance.

2. **Automatic Pagination and Throttling**:

   - The pipeline automatically handles pagination for large data sets and prevents API throttling issues by adjusting the frequency of API calls.

3. **Failure Handling and Admin Alerts**:

   - The pipeline is equipped with robust failure handling, ensuring that if any part of the process fails, the **Failure Pipeline** is triggered. The **Logic App** sends detailed failure notifications to the admin for timely intervention.

4. **Customizable Data Fetching**:
   - Both the list of downloadable LEGO zip files (`result`) and user data (`userlist`) can be easily customized to fetch additional datasets or user information.

## Setup Guide

### Prerequisites

- **Azure Subscription**: Ensure you have access to create and manage Azure Data Factory, Data Lake, Key Vault, and Logic Apps.
- **Azure Key Vault**: Set up Key Vault to securely store API credentials (API Key, Username, Password).
- **Logic App**: Create a Logic App that can send custom email or message notifications to the admin when the pipeline encounters errors.

### Steps

1. **Deploy Azure Data Factory**:

   - Create a new **Azure Data Factory** instance for both Development and Production environments.
   - Deploy the pipeline configuration for data ingestion.

2. **Configure Azure Data Lake Storage (ADLS)**:

   - Create **Azure Data Lake Storage** to store downloaded data and fetched API results.

3. **Set Up Azure Key Vault**:

   - Store your Rebrickable API credentials in **Azure Key Vault**.
   - Grant **Managed Service Identity (MSI)** permissions to the ADF pipeline to access these credentials.

4. **Set Up the Logic App**:

   - Create a **Logic App** that listens for specific events (such as pipeline failures) and sends custom alerts to the admin.
   - Ensure that the Logic App is integrated with ADF and is triggered appropriately in case of errors.

5. **Deploy and Test**:
   - After deploying the pipeline and configuring the Logic App, trigger a test run to verify that all components (data download, API fetching, failure handling, and notifications) are functioning as expected.

## Customization

- **Adding More Files**: Update the `result` variable to include additional file names that need to be downloaded (e.g., `sets.csv.gz`, `minifigs.csv.gz`).
- **Adding More Users**: Modify the `userlist` variable to add more user-specific data fetching (e.g., `user123`, `user456`).
