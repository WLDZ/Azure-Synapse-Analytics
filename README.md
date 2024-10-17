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

## Solution
[data_ingestion.md](WikiPage)
