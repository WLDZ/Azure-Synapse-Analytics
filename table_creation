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
