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

