# User Document for LEGO Database Tables

## Overview
This document provides an overview of the SQL code for creating four tables in the LEGO database, focusing on user profiles, owned sets, set dimensions, and date dimensions. The tables use Delta Lake format for improved performance and data reliability.

### Tables Overview

1. **Rebrickable_Profile_Dimension**
2. **Owned_Sets_Fact**
3. **Lego_Sets_Dimension**
4. **Date_Dimension**

---

## 1. Rebrickable_Profile_Dimension

### Purpose
The `Rebrickable_Profile_Dimension` table is designed to store information about users and their interactions with LEGO sets, particularly from the Rebrickable platform.

### Structure
| Column Name        | Data Type      | Constraints     | Description                                        |
|--------------------|----------------|------------------|----------------------------------------------------|
| Profile_Dim_Key    | BIGINT         | NOT NULL         | Unique identifier for each profile dimension entry.|
| User_ID            | BIGINT         | NOT NULL         | Unique identifier for each user.                   |
| Set_Name           | VARCHAR(128)   |                  | Name of the LEGO set associated with the user.     |
| Set_Num            | VARCHAR(128)   |                  | Number identifying the LEGO set.                    |
| List_ID            | BIGINT         |                  | Identifier for the list associated with the set.    |
| Date_added         | DATE           |                  | Date the set was added to the user’s profile.      |

### SQL Code
```sql
CREATE TABLE lego.Rebrickable_Profile_Dimention
(
    Profile_Dim_Key bigint NOT NULL,
    User_ID bigint NOT NULL,
    Set_Name varchar(128),
    Set_Num varchar(128),
    List_ID bigint,
    Date_added date
)
USING DELTA;
```
## Owned_Sets_Fact

### Purpose
The Owned_Sets_Fact table records factual data regarding sets owned by users, including pricing information. This table links users with their owned LEGO sets.

### Structure
| Column Name     | Data Type       | Constraints     | Description                                      |
|------------------|------------------|------------------|--------------------------------------------------|
| Owned_Fact_ID    | BIGINT           | NOT NULL         | Unique identifier for each owned fact entry.     |
| Set_ID           | VARCHAR(128)     | NOT NULL         | Unique identifier for the set owned.             |
| User_ID          | BIGINT           | NOT NULL         | Unique identifier for each user.                 |
| Price            | DECIMAL(19,4)    |                  | Price of the owned LEGO set.                      |

### SQL Code
```sql
CREATE TABLE lego.Owned_Sets_Fact
(
    Owned_Fact_ID BIGINT NOT NULL,
    Set_ID varchar(128) NOT NULL,
    User_ID BIGINT NOT NULL,
    Price DECIMAL(19,4)
)
USING DELTA;
```
## Lego_Sets_Dimension

### Purpose
The Lego_Sets_Dimension table contains descriptive attributes of LEGO sets. This dimension is essential for providing detailed information about each set available in the database.

### Structure
| Column Name                    | Data Type       | Constraints     | Description                                              |
|--------------------------------|------------------|------------------|----------------------------------------------------------|
| Set_Dim_Key                    | BIGINT           | NOT NULL         | Unique identifier for each set dimension entry.          |
| SetNumber                      | VARCHAR(128)     |                  | Unique number identifying the LEGO set.                  |
| SetName                        | VARCHAR(128)     |                  | Name of the LEGO set.                                    |
| NumberOfParts                  | INT              |                  | Total number of parts in the LEGO set.                   |
| ImageURL                       | STRING           |                  | URL link to the image of the LEGO set.                   |
| ThemeID                        | INT              |                  | Identifier for the theme associated with the set.        |
| ThemeName                      | VARCHAR(128)     |                  | Name of the theme associated with the set.               |
| LastModifiedDateOriginal       | DATE             |                  | Original last modified date of the set entry.            |
| LastModifiedTimeOriginal       | TIMESTAMP        |                  | Original last modified timestamp of the set entry.       |
| Year                           | INT              |                  | Year the LEGO set was released.                          |

### SQL Code
```sql
CREATE TABLE Lego.Lego_Sets_Dimension
(
    Set_Dim_Key BIGINT NOT NULL,
    SetNumber VARCHAR(128),
    SetName VARCHAR(128),
    NumberOfParts INT,
    ImageURL STRING,
    ThemeID INT,
    ThemeName VARCHAR(128),
    LastModifiedDateOriginal DATE,
    LastModifiedTimeOriginal TIMESTAMP,
    Year INT
)
USING DELTA;
```

## Date_Dimension

### Purpose
The Date_Dimension table is used to store detailed information about dates. This table facilitates time-based analyses and allows users to easily filter data by different date attributes.

### Structure
| Column Name      | Data Type       | Constraints     | Description                                               |
|------------------|------------------|------------------|-----------------------------------------------------------|
| Date_ID          | INT              |                  | Unique identifier for each date entry.                    |
| Date             | DATE             | NOT NULL         | Specific date.                                           |
| Year             | INT              | NOT NULL         | Year of the date.                                       |
| Month            | INT              | NOT NULL         | Month of the date.                                      |
| Day              | INT              | NOT NULL         | Day of the date.                                        |
| Quarter          | INT              | NOT NULL         | Quarter of the year (1-4).                              |
| Day_of_Week     | INT              | NOT NULL         | Day of the week (1=Monday, 7=Sunday).                  |
| Week_of_Year     | INT              | NOT NULL         | Week number of the year.                                |
| Is_Weekend       | BOOLEAN          | NOT NULL         | Indicates if the date falls on a weekend (true/false).  |
| Month_Name       | STRING           | NOT NULL         | Name of the month (e.g., January).                      |
| Day_Name         | STRING           | NOT NULL         | Name of the day (e.g., Monday).                         |

### SQL Code
```sql
CREATE TABLE lego.Date_Dimension
(
    Date_ID INT,  -- ID for each date
    Date DATE NOT NULL,
    Year INT NOT NULL,
    Month INT NOT NULL,
    Day INT NOT NULL,
    Quarter INT NOT NULL,
    Day_of_Week INT NOT NULL,
    Week_of_Year INT NOT NULL,
    Is_Weekend BOOLEAN NOT NULL,
    Month_Name STRING NOT NULL,
    Day_Name STRING NOT NULL
)
USING DELTA;
```
## Conclusion
These tables form a foundational structure for managing data related to LEGO sets, users, ownership, and dates. They allow for detailed queries and analyses regarding user interactions with LEGO products, facilitating better insights and data-driven decisions.


