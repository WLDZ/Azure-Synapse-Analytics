# Building a Dimensional Model Using Data from Rebrickable API in Azure Synapse Analytics

## Objective:
The objective of this activity is to practice data engineering skills by ingesting data from an external API and transforming it into a dimensional model within Azure Synapse Analytics. This data will represent Lego sets and their attributes, organized into tables for querying and analysis.

## Steps to Complete the Lab:

1. **Ingest Data into a Data Lake:**
   - Access the "Rebrickable" API, which provides data about Lego sets (e.g., set number, name, release year, number of parts, theme ID).
   - Store the data in the **Raw Layer** of the Azure Data Lake. The data will likely be in JSON format.

2. **Create a Dimensional Model:**
   - **Lego Sets Dimension**: Transform the raw Lego set data into a table format, where each row represents a Lego set. The dimension should include:
     - Set number (as the primary key).
     - Name of the set.
     - Year of release.
     - Number of parts.
     - A theme ID (such as Star Wars or Ninjago).
   - **Enhance the Dimension**: Fetch the corresponding theme names from the API and add them to the Lego Sets dimension. This addition will allow users to easily understand the theme of each Lego set.

3. **Build Additional Fact and Dimension Tables (Optional – Advanced):**
   - **Fact Table: Owned Sets**: Create a fact table that stores which users own which Lego sets. Query the API to obtain this information for specific user profiles.
   - **Profile Dimension**: Create a user profile dimension that includes user IDs and relevant information from the Rubber Cable API.
   - **Date Dimension**: Add a date dimension that records when each Lego set was added to a user’s collection. Since the API does not provide an explicit purchase date, infer it by observing when a new Lego set appears in a user’s collection.

4. **Data Transformation:**
   - Clean and format the data to ensure it is user-friendly. Ensure column names are readable and clear.
   - Apply appropriate data types to each column (e.g., dates, numbers, strings).
   - Implement Slowly Changing Dimensions (SCD) Type 1, meaning old values can be overwritten with new ones when updating the data.

5. **Register the Tables in a Catalog:**
   - After processing the data, store the tables in the **Final Zone** of Azure Data Lake. 
   - Use Azure Synapse Analytics to register the tables in a catalog. This registration will allow SQL queries to be executed using table names without referencing file paths directly.

6. **Partitioning and Structure:**
   - **Final Layer**: For the final dimensional and fact tables, avoid applying partitioning unless it benefits query performance. Keep the data structure simple.

7. **Security Considerations:**
   - Set proper access permissions for the Azure Data Lake.
   - Use secure methods to store secrets or access keys.

8 **Solution Overview**

The solution is divided into three parts:

#### 1. Data Ingestion from Rebrickable Website and Storing in Raw Layer
The first part involves extracting data from the Rebrickable website. This data is ingested into the raw layer of the data lake, preserving its original form without any transformations. This layer acts as the foundational storage, where all raw, unprocessed data is saved for further processing. Following is the link to
the data ingestion process that is used for this activity.
[Data Ingestion](./data_ingestion.md)



#### 2. Data Warehouse Structure Creation
In this phase, we design and create the structure of the data warehouse where the processed data will be housed. This includes setting up necessary tables, defining schemas, and applying any relevant business rules. The warehouse serves as the structured environment for storing transformed data. Following is the link to
the data warehouse creation process for this activity.
[Data Warehouse](./table_creation.md)


#### 3. Data Transformation and Loading into Curated Layer
The final part involves loading the raw data from the data lake, performing transformations to clean, enrich, and structure the data according to business requirements. This transformed data is then loaded into the curated layer of the data lake. The curated layer serves as the final repository from which data can be accessed and served to meet business needs. Following is the link to the final activity of the whole process
[Solution](./solution.md)




