---
title: "Fabric "
description: Technical Notes
date: 2025-01-29T00:00:00.000Z
---
# #2Read:

- https://medium.com/@jacobrnnowjensen/putting-microsoft-fabric-to-the-test-34b7383a9546
- [X] [Nice article for  Data Ingestion](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/scenarios/cloud-scale-analytics/best-practices/automated-ingestion-pattern)
- [X] [Metadata driven] (https://techcommunity.microsoft.com/blog/fasttrackforazureblog/microsoft-fabric-metadata-driven-pipelines-with-mirrored-databases/4222480)

- [X] https://milescole.dev/
- [X] https://www.serverlesssql.com/category/fabric/page/3/
- [X] [Usecase Fabric End to End flow](https://debruyn.dev/2023/fabric-end-to-end-use-case-analytics-engineering-part-1-dbt-with-the-lakehouse/)
- [X] [Data Lake Implementation Fabric](
https://learn-it-all.medium.com/data-lake-implementation-using-microsoft-fabric-ccea72a8d162)
- [X] [Fabric code to Get token](https://learn.microsoft.com/en-us/fabric/data-engineering/microsoft-spark-utilities#get-token)
- [X] [Pandas Vs DuckDB Vs Pyspark in Fabric](https://medium.com/@mariusz_kujawski/exploring-python-libraries-for-data-engineering-in-ms-fabric-pandas-polars-duckdb-and-pyspark-8a7df3326193)
- [X] [Sempy](https://learn.microsoft.com/en-us/python/api/semantic-link-sempy/sempy.fabric?view=semantic-link-python)
- [X] [Invoke Fabric pipeline using Rest API](https://learn-it-all.medium.com/calling-a-fabric-data-pipeline-from-azure-data-factory-pipeline-b739418e6b34#:~:text=We%20essentially%20have%20two%20web,data%20pipeline%20using%20that%20token.&text=From%20the%20Fabric%20data%20pipeline,invoke%20from%20Azure%20Data%20factory.)
- [X] [Connecting to Warehouse using JDBC](https://milescole.dev/data-engineering/2024/09/27/Another-Way-to-Connect-to-the-SQL-Endpoint.html)
- [X] [Service Principal usage for Authentication](https://www.youtube.com/watch?v=_RXpvWjgZE8)

# [[Synapse]]
+ Dedicated SQL Pool - use when you have large scale data warehousing solutions.
+ Serverless SQl Pool - Pricing about the query. For 1 Tb of data it is free; This is applicable  when you  have lot of adhoc query and inconsistent volume.
	+  Query data from ADLS, CosmoDB
	+ it create  a querying surface area means a LOGICAL DATAWAREHOUSE over the files
	+  If you create database, all are external.
	+  LDW  store the metadata about the underlying
	+ You cannot create Table or Materilaized View or Index on the view
- Why Azure Synapse:[Ref](https://www.youtube.com/watch?v=DL36iVJK8w4) (37:24)
	  1) Point is integration b/w ADLS and DataWarehouse is seemless
	  2) Need not worry about the compute, say for example you need a seperate DBricks or HDInsight
	  
# Capacity 
[[FabricConcepts]]

#Azure/Fabric #Concept

## Capacity - Billing

1. Tenant - it is tied to an organisation; A tenant can have multiple CAPACITY; A Workspace is tied to a CAPACITY  
2. Compute + Storage (Similar to ADLS) + Mirroring + Network  
3. Choose the capacity  
	- Warehouse F64 ~ 32 cores  
	- Spark Compute ~ 64*2*3  
4. Except Power BI (as it is completely independent), Fabric capacity is sufficient or free trial for running all the other experiences.  
	- If you have a POWER BI premium license (P2..P5), then you can enable Fabric Capacity at the TENANT level  
	- For PPU (Premium per user), you cannot enable the Fabric capacity; You need to purchase FKU  
5. Capacity - Pay as you go OR Reserved  
	- **Pay as You Go** - there is no automatic pause/resume option  
	- **Reserved** - if you are purchasing 64 SKU and using only 32 SKU, then the remaining 32 SKU is wasted  
6. Licensing  
	- Two licenses are needed: one for Capacity License and one for User License  
	- **User Licensing:**  
			- F64 >= Needs a developer license but does not need a Viewer license  
			- Pro/Premium Per User - Capacity is shared; Premium Per User allows you to use Power BI datamart  
			- Power BI Premium Per Capacity - Dedicated capacity  



### Concept: [[Architecture & Design]]
#Azure/Fabric #Architecture
1. If your Workspaces are allocated to a Capacity in the UK and the company is located in the UK, assume a new capacity is allocated in the USA and assigned to a WAREHOUSE, then consider the following:
   - This may break data protection law (as data needs to be transferred to WAREHOUSE in the USA, which certain organizations do not adhere to).
   - There could be egress fees across regions (networking cost).

   > If we now look at a scenario in which we have the same 3 Workspaces with Lakehouses and a Warehouse in each workspace,
   >except now we have an additional Fabric Capacity that has been created in the East US region.
   > If we allocate the Workspace that contains the Warehouse to this new capacity, even though it is in the same Fabric tenant, we will be moving data across regions if we load the Warehouse from the Cleansed Lakehouse.
   > Not ideal as this may break any data protection, plus there could be egress fees across regions.
   > [Pricing page](https://www.serverlesssql.com/microsoft-fabric-warehouse-for-the-database-administrator-dba-part-3-fabric-compute-and-the-sql-engine/)

---

# Fabric - Capacity Metric App[[FabricConcepts]]
#Azure/Fabric #Concept 
## Throttling
- It means if the resource consumption exceeds the allocated capacity. In Fabric, it happens in stages and is handled using the **SMOOTHING** technique. The process is stalled only when the consumption reaches the last stage.

## Smoothing
- This technique balances the capacity. For example, there can be hours in a day where capacity is overutilized and underutilized. This technique balances the usage based on throttle stages.

## Throttling Stages
- If the capacity overutilization is **< 10 mins (overage protection)**, the job can consume 10 mins of future capacity use without throttling.
- If capacity overutilization is **> 10 mins and < 60 mins**, then there will be a delay of 20s in the interactive job.
- If capacity overutilization is **> 60 mins and < 24 hours**, then interactive requests will be rejected.
- If capacity overutilization is **> 24 hours**, then nothing is allowed to run (background rejection).
# Elements
[[FabricConcepts]]
1. Choose the capacity  
2. Except Power BI (as it is completely independent), Fabric capacity is sufficient or free trial for running all the other experiences.  
3. Create Workspace pointing to Capacity  
4. Microsoft has an app called **Microsoft Fabric Capacity Metrics App** to monitor the capacity. It provides an easy plug-and-use experience.  

**Reference:** [Is Microsoft Fabric Just a Rebranding?](https://debruyn.dev/2023/is-microsoft-fabric-just-a-rebranding/)  

## Onelake  
- There will be a single ADLS Gen2 storage account.  
- This removes the burden of extra management and the ability to fine-tune every knob and switch to your liking.  

## Lakehouse  
- A **Lakehouse** is a **DELTA table**.  

## Apache Spark in Lakehouse  
- **High Concurrency Mode**: This allows you to share a Spark session between multiple Spark Notebooks.  
  - Even after the very short start-up time, the next notebook you run doesn’t have to take that short hit.  
  - **Configuration:** [Configure High Concurrency Session Notebooks](https://learn.microsoft.com/en-us/fabric/data-engineering/configure-high-concurrency-session-notebooks?source=recommendations)  

## T-SQL  
- **Lakehouse T-SQL Endpoint**: A read-only version of the Delta Lake tables in your Lakehouse on OneLake (similar to Serverless SQL).  
- **Data Warehouse**: A read-write version of Delta Lake tables on OneLake (similar to Dedicated Pool).  
- **Comparison to Synapse**: The key difference is that the execution engine in Fabric uses **POLARIS**.  

## Workspace  
- If you select a **Power BI experience**, it will not impact the overall experience.  
- Meanwhile, a **Data Warehouse** created in Power BI experience is the same as Data Warehouse in Data Factory.  

## Data Warehouse  
- **One use case for choosing a Warehouse is Multi-Table Transactions**, for example:  



## Additional Resources  
- **Exploring Delta Lakes Logs:** [Query Delta Lake Logs](https://learn.microsoft.com/en-us/fabric/data-warehouse/query-delta-lake-logs?source=recommendations)  
- **Limitations:** [Fabric Data Warehouse Limitations](https://learn.microsoft.com/en-us/fabric/data-warehouse/limitations?source=recommendations)
- **Reference:** [Working with JSON and Nested Arrays in Microsoft Fabric](https://medium.com/@jacobrnnowjensen/working-with-json-and-nested-arrays-in-microsoft-fabric-2487bd83f930)
# Fabric - Data Engineering - Spark  


- Spark settings are specific to a **Workspace**.  

## Spark Pools  
5. **Starter Pools**  
   - Machines are pre-warmed in the background, making **session initialization faster**.  
   - Pre-warmed machines are **available** for immediate use.  
   - In a **Starter Pool**, you can change the **number** of machines, but **not** the **size** or **type** of machines.  

6. **Custom Pools**  
   - Custom Pools **take longer** to initialize (**2-3 minutes**) as they do not have pre-warmed machines.  
   - Recommended **only for development**, where longer cluster spawn times are acceptable.  
   - **Spark session initialization** takes **2-3 minutes** for Custom Pools, but **only seconds** for Starter Pools.  

## Node Size  
- Available node sizes: **S, M, L, XL, XXL**  
- Example: XXL = **64 cores, 512GB RAM**  

## High Concurrency vs. Standard  
- **High Concurrency (HC) Mode**  
  - **Reduces Spark session time**.  
  - **Single-user boundary**: The session **cannot** be shared with multiple users.  
  - **Use Case**: If a notebook triggers multiple notebooks, **HC mode** avoids reinitializing Spark sessions each time.  

## Notebook Features  
- **Built-In Code Snippets**: Provides sample code snippets for quick reference.  
- **Freeze Cell**: Prevents execution of a specific cell.  
- **Lock Cell**: Prevents editing but allows execution.  
- **Resource Upload**:  
  - Upload artifacts related to code.  
  - Each file **must be <100MB**, with a **maximum of 500MB**.  
- **Lakehouse**: Default lakehouse is **automatically attached**.  
- **Shortcuts vs. Mount Points**:  
  - Shortcuts provide a **better, more persistent** way to access data.  
  - Users can directly reference and interact with **data lakes or Lakehouses**.  
  - Unlike **mount points**, shortcuts **do not** have limitations.  

## MSSPARKUTIL  
- **Fastcp**: Uses **underlying AzCOPY**, enabling **fast data transfer**.  
- Example: **70 million records copied in 20 seconds**.  

## Performance: Stored Procedures vs. Notebooks  
- **Stored Procedures** are observed to be **faster** compared to **Notebooks**.  
- Read the full comparison:  
  [SQL Stored Procedures in Fabric Warehouse Offer Blazing Speed and Power at Scale](https://techcommunity.microsoft.com/blog/healthcareandlifesciencesblog/sql-stored-procedures-in-fabric-warehouse-offer-blazing-speed-and-power-at-scale/4287247)  
# Fabric - Item Level Permission  
7. **Intent of Item Level Permission**  
   - Allows sharing of items **without providing access** to the **Workspace**.  

8. **Sharing Requirements**  
   - To share an item, you **must have** a **Member-level role**.  

## Warehouse  
- **ReadData** → Allows querying tables but **does not** permit creating shortcuts (i.e., no underlying OneLake access).  
- **ReadAll** → Grants access to **OneLake files** under the Warehouse.  
  - Users can **create shortcuts** and **read data** using **Spark Notebooks**.  
- **Build** → Enables creating the **default Semantic Model** and **building reports**.  
- **Table-Level Permissions** → Requires using `GRANT` and `REVOKE` statements.  

## Notebook  
- **Permissions:** Share, Edit, Run  
- **Run Permission:**  
  - Any user with **Edit** permission must also be granted **Run** permission.  

## Lakehouse  
- **ReadData** → Can only read data using the **SQL Analytics Endpoint**.  
  - Admin must use `GRANT`/`MODIFY` permissions to allow changes.  
- **ReadAll & Build** → Same as **Warehouse** permissions.  

## OneLake  
9. **Currently in Preview Mode**.  
10. Uses **Role-Based Access Control (RBAC)** to grant access to specific folders.  
11. **Default Access:**  
   - By default, all users have the **DefaultReader** role, allowing them to read all folders.  
12. **OneLake Shortcuts:**  
   - **Permissions must be defined on the destination table**.  
   - **Defining permissions on the shortcut itself is not allowed**.  
# Fabric - Row Level Security (RLS)  

## Key Considerations  
13. **Applied at the Database Level**  
   - If a user tries to read data via the **OneLake path**, **RLS will not be enforced**.  
   - To prevent bypassing RLS, **grant only `READDATA` access** to the user.  

## Steps to Implement RLS  
14. **Input** → The **USERNAME** of the user.  
15. **Create a Function**  
   - Takes **USERNAME** as input.  
   - Defines logic to **restrict records** displayed.  
   - Must include **Schema Binding** when creating the function.  
16. **Create a Security Policy**  
   - Implement the **security policy** on the table.  
   - Pass the **column name** that should be restricted within the policy.  
# Fabric - Dynamic Data Masking (DDM)  

## Key Considerations  
-  **Applied at the Database Level**  
   - Similar to **Row-Level Security (RLS)**, it is enforced at the **database level**.  
   - Original data can still be accessed using **Spark Notebooks**.  

-  **Combine with Object-Level Security**  
   - Dynamic Data Masking should be **used alongside Object-Level Security** for better protection.  

## Steps to Apply Dynamic Data Masking  
17. **Remove Existing Security Policies**  
   - If there are any **security policies** with **Schema Binding**, they must be **removed** before applying **Dynamic Data Masking**.  

18. **Apply Masking Function**  
   - Use `ALTER` to modify the table and **apply the masking function** to the relevant columns.  
# Fabric - Column & Object Level Security  

## Column-Level Security  
- Requires **Warehouse permission** as **READ** (**not** `READDATA`).  
- Use `GRANT SELECT` to specify column-level access:  

  ```sql
  GRANT SELECT (<column_name>) ON <table_name> TO <user_name>;
    ```
# Fabric - OneLake - Shortcut  

## Shortcut Overview  
- **Source:** Can be **internal** (Lakehouse/Warehouse) or **external** (Azure Data Lake, S3, GCP, Dataverse).  
- **Permissions:** Required to connect to the source.  
- **Destination:** Can be **Lakehouse** or **KQL**.  

## Caveats for Table Shortcuts  
- Shortcuts **can only be created at the top level** of a table.  
- **Sub-directory level shortcuts are not supported** for tables.  
- In the **File section**, shortcuts **can be created at any level**.  
  - **Use Case:** If data is stored in a sub-directory, place the shortcut under the **File section**.  

## Scenario: Azure Data Lake to Lakehouse Shortcut  
- **Azure Datalake** ---> **Shortcut** -----> **Lakehouse**  

---

### Updating the Scenario
- **Update a Record in the Shortcut:**  
  - If you have the **Data Contributor** role, the changes will be reflected in the source.
- **Update the Azure Data Lake:**  
  - Any updates in **Azure Data Lake** will immediately be available in the Shortcut.

---

### Deleting Scenario
This applies to both **Table** and **File**:
- **Delete a Record in Azure Data Lake:**  
  - The record will be deleted in the Shortcut file.
- **Remove a Record in the Lakehouse Shortcut (if Write access is granted):**  
  - The record will be deleted from Azure Data Lake.
- **Delete a Shortcut (File/Table):**  
  - Deleting the Shortcut will **not** delete the original file/table.

---

### Accessing Shortcuts Internal to Fabric  
#### Scenario:
- **W1--[LH1]** ----> **Shortcut [LH2]** created by **Alice** (Contributor role in LH1).  
- **Bob**, another user in **LH2**, can read it using **T-SQL/Power BI**.  
  - This works because **T-SQL uses the identity of the owner (Alice)**.
- **Bob will get an access violation issue** when trying to read the **Shortcut** using a **Notebook**.  
  - This happens because **Notebooks use the identity of the user (Bob)**.
- **Solution:**  
  - To allow **Bob** to access the Shortcut using **Spark Notebook**, assign **Bob the Contributor role**.

---

### Accessing Shortcut to ADLS (Delegated Authorization Model)  
#### Scenario:
- **Alice** should have the **Storage Blob Contributor Role** to create a **Shortcut on ADLS**.
- **Alice** and **Bob** are both on **W2--[LH2]**.  
- **Bob** can access the **Shortcut in W2** using **T-SQL** and **Spark Notebook**.


#Fabric - Onelake - Managed Vs External Table

19. Both **Managed** and **External** tables can be created on any file format.
20. We should attach **Lakehouse** to the notebook if we need to use the **saveTableAs** Spark API.
21. When you mention **LOCATION** in **CREATE TABLE** or **path** in the **saveTableAs** Spark API, the table will be created as an **External table**.
22. Both the **External** and **Managed** tables will be available under the **Table section**.
23. All the **Shortcut** created are like **Managed tables**.

#Fabric - In Relation with Delta Table
[Reference](https://learn.microsoft.com/en-us/fabric/get-started/delta-lake-interoperability)
## Delta Table Column Mapping:
24. Even in Databricks, this is a preview feature. In MS Fabric, you can create a **Delta table with Column mapping** using Spark Notebooks.
   - Unfortunately, it is not compatible with the **Data Warehouse**. This means it cannot be queried in the Warehouse.
25. This feature allows us to **rename or drop** columns in a Delta table. By default, you cannot rename or drop a column in a Delta table.

## Pausing Delta Lake Logs:
26. If you pause the **Delta Lake logs**, any changes made to the Delta table will **not** be reflected in the Data Warehouse.
   
   **Use Case**:  
   When publishing is paused, Microsoft Fabric engines that read tables outside of the Warehouse see the data as it was before the pause. It ensures that reports remain stable and consistent, reflecting data from all tables as they existed before any changes were made to the tables. Once your data updates are complete, you can resume **Delta Lake Log publishing** to make all recent data changes visible to other analytical engines.

## Cloning:
27. Only the **metadata** (such as schema) of the source table is copied, not the underlying Parquet data files. This means the cloned table still references the original Parquet data files in One Lake without duplicating the data files. Cloning is sometimes referred to as a **"Zero Copy Clone"**.
28. A cloned table is **separate and independent** from its source table.
29. Any changes made in the source table are **not** reflected in the cloned table and vice-versa.
30. The clone is based on a **point-in-time** up to thirty days in the past or the current point-in-time. The new table is created with a timestamp based on **UTC**.

   **Limitation**:  
   - Table clones across warehouses in a workspace are not currently supported.  
   - Changes to the table schema prevent a clone from being created before the table schema change.

## Time Travel (Preview Feature):
31. Similar to the one in **Delta tables**.

   **Limitation**:
   - Supply at most three digits of **fractional seconds** in the timestamp.
   - Currently, only the **Coordinated Universal Time (UTC)** time zone is used for time travel.
   - Time travel is **not supported** for the **SQL analytics** endpoint of the Lakehouse.
   - In views, you cannot query past data from tables in a view from before the view was created. But you can create a View and then apply the time travel feature. **You cannot create a View with the time travel feature**.
   - The **OPTION FOR TIMESTAMP AS OF** syntax can only be used in queries that begin with the **SELECT** statement. Queries like **INSERT INTO SELECT** and **CREATE TABLE AS SELECT** cannot be used with the **OPTION FOR TIMESTAMP AS OF**.

## Deletion Vector:
Ref: [Deletion Vectors](https://milescole.dev/data-engineering/2024/11/04/Deletion-Vectors.html)

- It is an optimization technique where, when you delete or update a Delta table, usually a new Parquet file is created with impacted and non-impacted records. When **Deletion Vector** is enabled, this will **not** happen.

   **Copy-on-Write**:
   > If we had a Delta table with one massive Parquet file containing 1,000,000,000 rows and we delete or update one row, copy-on-write would result in 999,999,999 rows of data being written to a new Parquet file, even though only one row is being updated or deleted. While in real-world scenarios, files aren’t usually this large, the example shows that unchanged data in files with changes must be copied, which can have a massive performance impact.

   **Merge on Read**:
   > In the example of deleting 1 row from a table with a single Parquet file containing 1,000,000,000 rows, with **Deletion Vectors**, we now only record the position of the single row that has been soft-deleted (one very small .bin file) instead of rewriting a new Parquet file with 999,999,999 rows.

   **Updates**:
   - With update operations, the changed record is **soft-deleted** in the existing Parquet files via a new deletion vector, and the new version of the changed record is written to a new Parquet file added to the Delta log.
   
  **To do a HARD delete**:
  - Usually, all the deleted records will be available in the .bin file until a **VACUUM** is run. In the case of **PII data**, run the **VACUUM** to clean up the invalidated files.

  **Configure VACUUM manually**:
  
  ```text
  spark.conf.set("spark.databricks.delta.retentionDurationCheck.enabled", "false")
  VACUUM user_data RETAIN 0 HOURS
  ```
  **Enable DELETION VECTOR**:

  ```text
  spark.conf.set("spark.databricks.delta.properties.defaults.enableDeletionVectors", "true")
  ```
  **Scenario to Disable DELETION VECTOR**:
  - **Infrequent Writes, Frequent Reads**
  - **Fabric Pipeline COPY activity** does not support **deletion vector**.



## Change Data Feed:
- This feature is useful when we need to identify the kind of operation at the **SOURCE**, based on the **ACTION type**, and decide if we need to perform some logic in the **TARGET**.
- If the use case is simply **incremental reads**, add a **timestamp column** and read it incrementally.

**Reference**: [Delta Change Data Feed in Fabric Lakehouses](https://www.serverlesssql.com/delta-change-data-feed-in-fabric-lakehouses/)

#Fabric - Warehousing

### Foreign Key:
32) **Fabric Warehouse** supports **foreign key constraints** but they **can't be enforced**. Therefore, it's important that your **ETL process** tests for integrity between related tables when data is loaded.
33) It's still a good idea to create **foreign keys**. One good reason to create unenforced foreign keys is to allow **modeling tools**, like **Power BI Desktop**, to automatically detect and create relationships between tables in the semantic model.

**Reference**: [Dimensional Modeling in Fabric Data Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/dimensional-modeling-dimension-tables)

#Fabric - Warehousing

### Foreign Key:
34) **Fabric Warehouse** supports **foreign key constraints** but they **can't be enforced**. Therefore, it's important that your **ETL process** tests for integrity between related tables when data is loaded.
35) It's still a good idea to create **foreign keys**. One good reason to create unenforced foreign keys is to allow **modeling tools**, like **Power BI Desktop**, to automatically detect and create relationships between tables in the semantic model.

**Reference**: [Dimensional Modeling in Fabric Data Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/dimensional-modeling-dimension-tables)

# Fabric - Mirroring - Shortcut

## Mirroring  
Mirroring is a data replication method where data is brought to the lakehouse using CDC (Change Data Capture) incrementally.  

## Mirroring vs. Shortcut  
- In **mirroring**, the data is replicated and stored in the lakehouse.  
- In **shortcut**, the data remains in the source and is not copied to the lakehouse.  

## When Should You Consider Mirroring?  
- If a source table has multiple ETL processes populating/writing to it, mirroring can be beneficial as it reads data incrementally.  
- It reduces the load on the source by eliminating the need for an ETL process to ingest the latest data into the lakehouse.  
- **Use Case:** Create a mirror for production reporting.  

## When Should You Not Use Mirroring?  
- Mirroring is **not** an alternative to ingestion.  

## Limitations of Mirroring (Azure SQL)  
- Mirroring does not capture permissions in the SQL database:  
  - Row-level security  
  - Object-level permissions  
  - Dynamic data masking  

## Pricing  
- **Power BI Premium**, **Fabric Capacity**, or **Trial Capacity** licensing is required to use mirroring.  

## Best Practices - Mirroring & Capacity  
**Reference:** [Fabric Mirroring - Replacing E-ETL](https://www.element61.be/en/resource/fabric-mirroring-replacing-e-etl)  

36. If your source has **frequent changes** and requires **24/7 data availability**, a **dedicated lower-grade capacity** for mirroring may be more cost-effective than using a high-end compute resource.  
37. If your source has **small, infrequent changes**, consider a separate capacity with scheduled start and pause times to avoid unnecessary costs.  
   - Capacity start and pause can be managed via **Azure REST APIs**.  
#Fabric - Warehousing - Data Recovery

### Restore Points:
- **Warehouse deletes** both the system-created and user-defined restore points at the expiry of the **30 calendar day retention period**.
- System-created and user-generated restore points **can't be created** when the **Microsoft Fabric capacity** is paused.
- At any point in time, **Warehouse** is guaranteed to be able to store up to **180 system-generated restore points** as long as these restore points haven't reached the thirty-day retention period.
- There will be **Storage** and **Compute** costs associated with the Restore Point.

### Limitation:
38) A **recovery point** can't be restored to create a new warehouse with a different name, either within or across the Microsoft Fabric workspaces.
39) **Restore points** can't be retained beyond the default **thirty calendar day** retention period. This retention period isn't currently configurable.

### Clone Table:
#Fabric -ALTER

### Reference:
- [The Reality of ALTER Table in Fabric Warehouses](https://www.serverlesssql.com/the-reality-of-alter-table-in-fabric-warehouses-2/)

### Key Points:
40) **ALTER Table Usage**:
   - **Supports adding a column** but **does not support** dropping a column or changing the datatype of a column.

41) **To Add a Column for a Table in the Lakehouse**:
   - You need to **change the protocol version**. Use the following code:
   
   ```sql
   ALTER TABLE table_name SET TBLPROPERTIES (
       'delta.columnMapping.mode' = 'name',
       'delta.minReaderVersion' = '2',
       'delta.minWriterVersion' = '5'
   )
  ```

- **Before the ALTER Command (Adding a New Column)**:
  The schema configuration before adding the new column looks like this:

  ```json
  {
  "protocol": {
    "minReaderVersion": 1,
    "minWriterVersion": 2
  }
  }
  {
  "metaData": {
    "id": "f439cc23-498d-4713-80e8-5f732032b0ab",
    "format": {
      "provider": "parquet",
      "options": {}
    },
    "schemaString": "{\"type\":\"struct\",\"fields\":[{\"name\":\"ProductKey\",\"type\":\"integer\",\"nullable\":true,\"metadata\":{}},{\"name\":\"ProductName\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}}]}",
    "partitionColumns": [],
    "configuration": {},
    "createdTime": 1721598597504
  }
  }
  ```

- **After the ALTER Command (With Column Mapping)**

  Once the `ALTER` command has been run, the schema string is updated to include the new column with column mapping information:

  ```json
  {
    "schemaString": "{\"type\":\"struct\",\"fields\":[{\"name\":\"ProductKey\",\"type\":\"integer\",\"nullable\":true,\
    \"metadata\":{\"delta.columnMapping.id\":1,\"delta.columnMapping.physicalName\":\"ProductKey\"}},{\"name\":\"ProductName\",\"type\":\"string\",\"nullable\":true,\"metadata\":{\"delta.columnMapping.id\":2,\"delta.columnMapping.physicalName\":\"ProductName\"}},{\"name\":\"ProductSubCategory\",\"type\":\"string\",\"nullable\":true,\
    \"metadata\":{\"delta.columnMapping.id\":3,\"delta.columnMapping.physicalName\":\"col-f772a929-55a7-477c-924c-5dc1b8aeb86e\"}}]}",
    "partitionColumns": [],
    "configuration": {
      "delta.columnMapping.mode": "name",
      "delta.columnMapping.maxColumnId": "3"
    },
    "createdTime": 1721643187063
  }
	```
	# Fabric - Work Experience & Issues Faced

### 1) **Spark Notebook Cannot Read Data Warehouse Tables**  
**Issue:**  
- Spark notebooks in Fabric cannot directly read tables from Fabric Data Warehouse.  

**Solution:**  
- Create a **SHORTCUT** for the required table in the **GOLD** layer tables to enable access in the Spark notebook.  



### 2) **Fabric Data Pipeline Does Not Support Triggers with Parameters**  
**Issue:**  
- A common ingestion framework was planned to handle multiple tables dynamically.  
- Fabric pipelines (as of now) **do not support passing parameters in triggers**, which limits dynamic execution.  

**Solution:**  
- Use a **Parent Pipeline** that triggers child pipelines using the **External Pipeline Trigger** functionality.  
- Each **Parent Pipeline** will be specific to a data source and trigger the appropriate ingestion process.  



### 3) **Fabric Pipeline Does Not Support Dynamic Notebook Names**  
**Issue:**  
- The requirement was to dynamically pass the **notebook name** to be executed, but Fabric pipelines do not allow this.  

**Solution:**  
- Create a **Parent Notebook** that accepts the notebook name as a parameter.  
- Use `mssparkutils` in the parent notebook to **dynamically trigger** the appropriate notebook.  



# Fabric Notebook - Limitations  
[Microsoft Fabric Notebook Limitations](https://learn.microsoft.com/en-us/fabric/data-engineering/notebook-limitation)  

### 1) **Returning Parameters from a Notebook**  
- The number of rows that can be returned is **limited to 10K rows or 5MB**.  

### 2) **Accessing Return Parameters as JSON**  
- Notebook return values can be accessed in **JSON format** using the following syntax:  
  ```text
  @Json(activity('Test Notebook').output.result.exitValue).message
  ```
  [Reference](https://community.fabric.microsoft.com/t5/Data-Pipeline/Referencing-notebook-exit-value-as-a-variable-in-a-data-pipeline/m-p/3507053)

  # Fabric Notebook - Spark Session  

## 1) Shared Spark Session Between Master and Child Notebooks  
- A **Master Notebook** shares its **Spark session** with child notebooks triggered using `mssparkutils.run()`.  
- Example:  
  ```python
  mssparkutil.run('notebook1')  # Creates a temp view
  mssparkutil.run('notebook2')  # Temp view from 'notebook1' is available in 'notebook2'
  ```
  **Reference:** [Microsoft Fabric - Shared Spark Sessions](https://christianhenrikreich.medium.com/microsoft-fabric-utilize-shared-sparksessions-fully-with-mssparkutils-notebook-run-and-runmultiple-79c780f5af1c)  

> SparkSQL, which is a module in Spark Core, has an optimizer called **Catalyst**.  
> SparkSQL has yet another component within Catalyst called **Catalog**.  
> **Catalog is not Unity Catalog**, but a **meta datastore** for all known tables and views within the SparkSessions.  
> Known tables can be **Hive tables, Unity Catalog tables, etc.**  

# Fabric SQL - Limitations in Warehouse  

## Temporary Tables  
42. **Limited Usage:** Temporary tables are supported but with restrictions:  
   - You **cannot join** a temporary table with a normal table.  
   - `INSERT INTO` with `SELECT * FROM` a normal table **is not supported**.  
   - **Reference:** [Temp Tables in Fabric Warehouses](https://www.serverlesssql.com/temp-tables-in-fabric-warehouses/)  

## Cursor Support  
- **Not Supported:** Fabric SQL **does not support cursors** because **Synapse uses a columnar format** for data storage.  

---

# ALTER Statement Limitations  
43. **Dropping Columns & Changing Datatypes:**  
   - You **cannot drop columns** or **change the datatype** using `ALTER TABLE`.  
   - **Time Travel Functionality is Lost:** When you apply an `ALTER TABLE`, **time travel tracking is reset** to the timestamp of the alteration.  

> _“It’s worth noting that when issuing an ALTER TABLE statement, it resets the date/time the table is being tracked to the date/time the table was altered.  
> For the existing functionality of altering a table to add primary, unique, and foreign key constraints, it probably did not surface that regularly as these are often implemented when a table is first created.  
> But now we can add new columns, this issue may surface more regularly.”_  
# Fabric Notebook - Warehouse Table  

## Sync Issues Between Warehouse Table and Notebook  
44. **Data Discrepancy:** There may be a synchronization issue between **Warehouse tables and Notebooks**, leading to:  
   - **Count mismatches** when querying data in the notebook versus querying via `SELECT * FROM` in the Lakehouse table.  
   - **Duplicated rows** or **inconsistent results** between Notebook and SQL Endpoint.  

### References:  
- [Duplicated Rows Between Notebook and SQL Endpoint](https://community.fabric.microsoft.com/t5/Data-Engineering/Duplicated-rows-between-notebook-and-SQL-Endpoint/m-p/3707317)  
- [Lakehouse Data Discrepancy Between Notebook and SQL Endpoint](https://community.fabric.microsoft.com/t5/Data-Science/Lakehouse-data-discrepancy-between-notebook-and-SQL-endpoint-or/m-p/4069421)  

# Fabric Warehouse & Delta Table

## 1. Singleton Data Manipulation Statements  

- The second rule of the warehouse is **“singleton data manipulation statements are not optimal.”**  
- Large batches of inserts that fail must be rolled back.  
- The underlying storage mechanism is a **delta file**.  
- **Comparison of Inserts:**
  - **1000 singleton inserts:** Creates multiple Parquet and log files.  
  - **Batch of 1000 inserts:** Creates **one** Parquet file and **one** log file.  

## 2. Snapshot Isolation in Fabric Warehouse  

- **Fabric Warehouse tables support only SNAPSHOT ISOLATION**.  
- Unlike **Delta tables in Databricks**, which support **SERIALIZABLE** and **WRITE SERIALIZABLE** isolation levels.  
- **Key Difference:**  
  - In **SQL Server**, multiple transactions can modify data in a table unless they attempt to update the same row simultaneously.  
  - In **Fabric Warehouse**, **multiple transactions cannot modify any rows in the same table**, even if they are different rows.  

🔗 **Reference:** [Transaction Isolation in Fabric Warehouses](https://www.serverlesssql.com/transaction-isolation-in-fabric-warehouses/)  

> *“A basic comparison between SQL Server and Fabric Warehouse snapshot isolation is that multiple transactions can modify data in a SQL Server table as long as no transactions are trying to update the same row at the same time, while in Fabric this isn’t possible – multiple transactions cannot modify any rows in the same table even if they are not the same row.”*  

## 3. WRITE CONFLICT  

🔗 **Reference:** [Exploring Time Travel and Row-Level Concurrency – Data Engineering with Fabric](https://www.sqlservercentral.com/articles/exploring-time-travel-and-row-level-concurrency-data-engineering-with-fabric)  

> *“We are going to focus on two or more parallel processes writing to the same row of data. However, write conflicts can happen when the metadata for the Delta table is changed at the same time by more than one process.”*  

- **INSERT operations do not create WRITE conflicts**.  
- **Update, Delete, and Merge Into operations create WRITE conflicts**.  

### Solutions for WRITE Conflicts:  

45. **Append-Only Table for Metadata**  
46. **Monitor Lock & Retry the INSERT** (requires privileged access)  
47. **In Databricks:**  
   - Handled using **Isolation Levels**  
   - **Partitioning the table**  

# Fabric Warehouse - Performance  

## Automatic Statistics & Caching  

- **Automatic Statistics feature** is available in Fabric.  
- **Caching is managed automatically** by Fabric, with limited user control.  

---

## V-Order  

- **V-Order is a read optimization technique** that enables faster reads.  
- **Tradeoff:** It adds **overhead during writes**.  
- **V-Order Control:**  
  - Can be enabled/disabled at the **table level** in **Lakehouse**.  
  - Can be disabled at the **warehouse level**, but **once disabled, it cannot be re-enabled**.  
- **When to avoid V-Order?**  
  - **High write workloads**.  
  - **Best practice:** Microsoft recommends using a **separate staging warehouse** with V-Order **disabled** and a **read-intensive warehouse** with V-Order **enabled**.  

🔗 **References:**  
- [Scenarios Where V-Order Might Not Be Beneficial](https://learn.microsoft.com/en-us/fabric/data-warehouse/v-order#scenarios-where-v-order-might-not-be-beneficial)  
- [Performance Analysis of V-Ordering in Fabric Warehouse](https://www.serverlesssql.com/performance-analysis-of-v-ordering-in-fabric-warehouse-on-or-off/)  

### V-Order in Lakehouse  

#### **How does it work?**  
> *"V-Order applies special sorting, row group distribution, dictionary encoding, and compression on Parquet files. This reduces network, disk, and CPU usage, improving cost efficiency and performance."*  
> *"V-Order sorting has a 15% impact on average write times but provides up to 50% more compression."*  

#### **How is V-Order enabled?**  
48. **Automatically enabled by Microsoft Fabric:**  
   ```sql
   spark.conf.set("spark.microsoft.delta.optimizeWrite.enabled", "true")
   ```
    [OR]  
```sql
ALTER TABLE <table_name>  
SET TBLPROPERTIES ('delta.autoOptimize.optimizeWrite' = 'true');
```
### 3) V-Order at the Session Level  

- When **enabled at the session level**, **all Parquet writes** use **V-Order**.  
- This applies to:  
  - **Non-Delta Parquet tables**.  
  - **Delta tables**, regardless of whether the `parquet.vorder.enabled` table property is set to `true` or `false`.  



### 4) Why is V-Order Enabled in Lakehouse by Default?  

> *"By pushing Power Query transformations and VertiPaq optimizations back to the lake, Direct Lake Semantic Models can directly query Power BI-optimized data in the lake and quickly hydrate Analysis Services models to cache data for faster results."*  

#### **Common Misconception:**  
- Many assume that **when a Direct Lake model caches data**, VertiPaq **sorting and encoding are reapplied** in the **Analysis Services database**.  
- **Reality:** If a **Delta table does not have V-Order enabled**, the **cached data will not** have **VertiPaq sorting and encoding applied**.  
- This is why **V-Order is enabled by default** in Microsoft Fabric today.  

#### **Power BI Optimizations in the Lake**  
V-Order enables **Direct Lake Semantic Models** to:  
- ✅ **Reduce data redundancy**.  
- ✅ **Eliminate the need for Semantic Model refreshes**.  
- ✅ **Ensure business logic is stored consistently** in the lake for **multi-workload** use cases.  



### 5) Impact of V-Order in Fabric Spark Notebooks  

- **V-Order adds overhead in Spark Notebooks**.  
- **Best suited for:**  
  - ✅ **Power BI Direct Lake Semantic Models**.  
- **Warehouse Impact:**  
  - 📈 **Read Performance:** ~10% improvement.  
  - 📉 **Write Performance:** 10-20% slowdown.  

🔗 **Reference:** [To V-Order or Not?](https://milescole.dev/data-engineering/2024/09/17/To-V-Order-or-Not.html)  


# Fabric Database
[[FabricConcepts]]
#inprogress
### Requirement
 1) Fabric Database is not available on all the region. [[Fabric region availability - Microsoft Fabric | Microsoft Learn](https://learn.microsoft.com/en-us/fabric/admin/region-availability)]
 
# Optimize Write for Apache Spark  

🔗 **Reference:** [A Deep Dive into Optimized Write in Microsoft Fabric](https://milescole.dev/data-engineering/2024/08/16/A-Deep-Dive-into-Optimized-Write-in-Microsoft-Fabric.html)  

### 1) What is Optimize Write?  

- **Enabled by default** in Fabric for **partitioned Delta tables**.  
- **Purpose:**  
  - Merges **small files** into **128MB** (or a specified target size) to **avoid small file problems**.  
  - This **reduces the number of files** but **adds extra processing cost** during writes.  
- **How does it work?**  
  - **Fabric shuffles data before writing**, ensuring that **each partition is written by a single executor**.  
  - This means **fewer but larger files are created**.  
  - **(Unclear if skewness is addressed.)**  

- #### **Considerations**  
📌 **Partitioned tables:**  
  - If you have **fewer than 10 partitions**, the **overhead of shuffling may outweigh** the benefits.  

- #### **Benefits**  
  - ✅ Queries scan **fewer, larger files**, improving **read performance** & **resource efficiency**.  
  - ✅ Optimized Write is important for:  
    - **Power BI Direct Lake**.  
    - **Fabric Warehouse / SQL Endpoint performance**.  

### 2) Optimize Write Configuration  

```sql
spark.conf.set("spark.databricks.delta.optimizeWrite.binSize", 1073741824)
spark.conf.set("spark.databricks.delta.optimizeWrite.enabled", "<true/false>")
```
### When to Avoid Optimize Write  

Avoid using **Optimize Write** in the following scenarios:  
- ❌ **Non-partitioned tables**.  
- ❌ **Use cases where extra write latency isn't acceptable**.  
- ❌ **Large tables with well-defined optimization schedules & read patterns**.  

---

## Low Shuffle Merge  

### What is it?  
> "The current algorithm isn't fully optimized for handling unmodified rows. With Low Shuffle Merge optimization, unmodified rows are excluded from an expensive shuffling operation that is needed for updating matched rows."*  



### Need for Low Shuffle Merge  

> "Currently, the MERGE operation is done by two join executions:"*  
1️⃣ **First Join:**  
   - Uses the **whole target table and source data** to find **touched files** that include **matched rows**.  
2️⃣ **Second Join:**  
   - Reads only the **touched files and source data** to perform the actual **table update**.  

#### **Challenges with the Current Approach**  
- Even though the **first join reduces data for the second join**, it may still include **a large number of unmodified rows**.  
- **The second join** requires **all columns** to be loaded, leading to **an expensive shuffling process**.  



### How Low Shuffle Merge Improves Performance  

✅ **Keeps matched row results from the first join** and reuses them in the second join.  
✅ **Excludes unmodified rows** from the **expensive shuffling process**.  
✅ **Creates two separate write jobs** for:  
   - **Matched rows**.  
   - **Unmodified rows**.  
✅ May result in **twice the number of output files**, but the **performance gain outweighs** the potential **small file problem**.  

# Fabric - Load Tables - DA  
[Dimensional Modelling](https://learn.microsoft.com/en-us/fabric/data-warehouse/dimensional-modeling-load-tables)  

## Stage Data  
✅ Allows restarting the ETL process without reloading from source systems.  
✅ Alternatives to staging: **Virtualization (Mirroring, OneLake shortcuts)**.  

## Transform & Load Data  

### Logging  
- Unique ID for each ETL execution.  
- Start & end time.  
- Status & error messages.  

### Surrogate Keys  
> *Identity columns are not available in Fabric Warehouse.*  

- **Never truncate & fully reload a dimension table** with surrogate keys, as it would invalidate fact table data.  
- **For SCD Type 2 dimensions**, regenerating historical versions might be impossible. 
> -  **Snip:** When a dimension table includes automatically generated surrogate keys, you should never perform a truncate and full reload of it. That's because it would invalidate the data loaded into fact tables that use the dimension.
    Also, if the dimension table supports SCD type 2 changes, it might not be possible to regenerate the historical versions.

## Managing Historical Change  
✅ Use **SCD Type 1** or **SCD Type 2**.  
✅ **Inferred Members** (rows inserted by fact load process) should be treated as **late-arriving dimensions**.  

### Dimension Member Deletions  
- If a record is **deleted in the source**, it should **not** be removed from the dimension table unless it was created in error and has no related fact records.  
- **Use soft deletes** with an `IsActive` or `IsCurrent` column.  

## Processing Fact Tables  
- Facts can be sourced from **both source systems & dimensions**.  
- Fabric Warehouse **does not enforce foreign keys**, so ETL must check data integrity.  
- **Always prefer incremental fact loading**.  
  > *Truncating and reloading a large fact table should be a last resort.*  
  - It's costly in terms of time, compute, and may disrupt source systems.  
  - **For SCD Type 2 dimensions**, key lookups must match within the validity period.  
  > - **Snip:** Especially for a large fact table, it should be a last resort to truncate and reload a fact table.
        That approach is expensive in terms of process time, compute resources, and possible disruption to the source systems.
         It also involves complexity when the fact table dimensions apply SCD type 2.
        That's because dimension key lookups will need to be done within the validity period of the dimension member versions. 

	**Incremental Data Reading**
	- **From source:**  
		- Use a **Watermark column** or **CDC (Change Data Capture)** to track inserts, updates, and deletes.  This can be implemented **reading logs** or **adding Trigger** that populate the keys into table
	- **From dimension table:**  
		- If the **dimension key exists**, insert the fact row.  
		- If the **dimension key is missing**, two approaches:  
			1️⃣ Insert a special "Unknown" dimension key (requires later update).  
	> 		it could indicate an integrity issue with the source system. In this case, the fact row must still get inserted into the fact table. A valid dimension key must still be stored. One approach is to store a **special dimension member (like Unknown)**. This approach requires a later update to correctly assign the true dimension key value, when known.

		2️⃣ If the natural key is trusted, insert a new dimension member and store its surrogate key.  

### Fact Updates & Deletions  
- Include **attributes to track modifications**, and **index these columns** for better performance.  
> - When you anticipate fact updates or deletions, you should include attributes in the fact table to help identify the fact rows to modify. Be sure to index these columns to support efficient modification operations.

## Dimensions  

### Handling Hierarchical Structures  
- **Balanced hierarchy**  
- **Unbalanced hierarchy (Parent-Child, e.g., Employee-Manager)**
- In a dimension it is better to transform and store the hierarchy levels in the dimension as columns.  
 
  - **If using parent-child relationships in Power BI**, avoid them for large dimensions.  
  > - **Snip:** If you choose not to naturalize the hierarchy, you can still create a hierarchy based on a parent-child relationship in a Power BI semantic model.However, this approach isn't recommended for large dimensions.

### Managing Historical Change  
- **SCD Type 1**  
- **SCD Type 2 (Time-based versioning)**  
  - Requires a **surrogate key** to manage duplicate natural keys.  
   
  > **Snip:**  Also, too many versions could indicate that a changing attribute might be better stored in the fact table. Extending the earlier example,   if sales region changes were frequent,the sales region could be stored as a dimension key   in the fact table rather than implementing an SCD type 2.

### Date Dimension  
- Fact tables store measures over time, requiring a **Date (Calendar) dimension**.  
- **Time-based analysis** should use separate **Date & Time dimensions**.  

### Conformed Dimensions  
> *Shared by multiple fact tables (e.g., Date Dimension).*  

### Role-Playing Dimensions  
> *Same dimension referenced multiple times in a fact table (e.g., Order Date, Ship Date, Delivery Date).*  

### Junk Dimensions  
Consolidate small dimensions with low cardinality (e.g., Flags, Indicators, Order Status) into one dimension.
> **Snip:**  A junk dimension is useful when there are many independent dimensions, especially when they comprise a few attributes (perhaps one), and  when these attributes have low cardinality (few values). The objective of a junk dimension is to consolidate many small dimensions into a  single dimension. This design approach can reduce the number of dimensions, and decrease the number of fact  table keys and thus fact table storage size. They also help to reduce Data pane clutter because they present fewer tables to users.
So candidates include flags and indicators, order status, and customer demographic states (gender, age group, and others).

### Degenerate Dimensions  
- A dimension that exists at the same grain as a fact table (e.g., Sales Order Number).
> A degenerate dimension can occur when the dimension is at the same grain as the related facts. A common example of a degenerate dimension  is a sales order number dimension that relates to a sales fact table. Typically, the invoice number is a single, non-hierarchical attribute in the fact table. So, it's an accepted practice not to copy this data to create a separate dimension table.

- **No separate dimension table required**, can be handled using views.  

### Outrigger Dimensions  
> *A dimension referencing another dimension (e.g., Geography dimension referenced by Customer & Salesperson dimensions).*  

### Multivalued Dimensions  
> *Used when a dimension attribute has multiple values (e.g., Salesperson assigned to multiple regions).*  
- Requires a **bridge table** to establish many-to-many relationships.
>   -   For example, consider there's a salesperson dimension, and that each salesperson is assigned to one or possibly more sales regions. In this case,    it makes sense to create a sales region dimension. That dimension stores each sales region only once. A separate table, known as the bridge table, stores a row for each salesperson and sales region relationship. Physically, there's a one-to-many relationship from the salesperson dimension to the bridge table, and  another one-to-many relationship from the sales region dimension to the bridge table. Logically,    there's a many-to-many relationship between salespeople and sales regions.**Another example**, is in the following diagram, the Account dimension table relates to the Transaction fact table. Because customers can have multiple accounts and accounts can have multiple customers, the Customer dimension table is related via the Customer Account bridge table.

---

## Facts  

### Primary Key  
- **Fact tables typically do not need a primary key**.  That's because it doesn't typically serve a useful purpose, and it would unnecessarily increase the table storage size.
- Uniqueness is implied by **dimension keys & attributes**.  

### Audit Attributes  
- Track changes in the fact table.  

### Types of Fact Tables  
1️⃣ **Transaction Fact Tables** → Tracks individual transactions.  
2️⃣ **Periodic Snapshot Fact Tables** → Stores measurements at predefined intervals.  
   > Example: There might be millions of stock movements in a day (which could be stored in a transaction fact table), but your analysis is only concerned with trends of end-of-day stock levels. 
  
3️⃣ **Accumulating Snapshot Fact Tables** → Captures milestones in a process.  

### Measure Types  
- **Semi-additive Measures** → Aggregate across some dimensions but not all (e.g., Account Balance).  
- **Non-additive Measures** → Cannot be aggregated (e.g., Profit Margin) because summing or averaging profit margins across products is meaningless because it ignores the underlying revenue and profit values 

### Factless Fact Table  
> *Records events/occurrences (e.g., Tracking student participation, Absence tracking).*  

### Aggregate Fact Tables  
- Can be pre-aggregated in **Warehouse tables** or **Power BI DirectQuery storage mode**.  
- Power BI can create **user-defined aggregations** to achieve same result.  

---

## Loading Data into Warehouse  

1️⃣ **COPY statement** only supports Azure Storage, **OneLake sources are not supported**.  
2️⃣ Supports **PARQUET & CSV file formats**.  

### Best Practices  
❌ Avoid singleton `INSERT` statements for ingestion → Poor performance.  
✅ Use `CREATE TABLE AS SELECT (CTAS)` or `INSERT...SELECT`.  
🚨 Dropping an existing table may affect **semantic models** (e.g., custom measures).  

# Fabric - Best Practices (BP)  

- **Microsoft Fabric workspace** provides a **natural isolation boundary** for the distributed compute system.  
- Workloads can leverage this **isolation boundary** to optimize **cost & performance**.  


# Fabric - Use Case 1: Learning Management System (LMS)  

## BRONZE to SILVER  

### 1) Data Cleansing  
- **Handling duplicates**  
- **Handling missing or NULL values**  
  - Delete rows with missing critical values  
- **Standardize date formats**  
- **Check for logical consistency**  
  - Example: Ensure a student's course completion date is after their enrollment date  

### 2) Data Validation  

### 3) Implement Business Logic  
- Compute **Completion Time in Days**  
- Calculate **Performance Score**  

### 4) Upsert Logic to SILVER  

---

# Fabric - Git Integration  
**#Fabric #UseCase1 #LMS #GitIntg**

### **Parameterize All Pipelines**  
- If a Notebook references a Storage Account, **parameterize it in the Pipeline**  

### **Reports & Semantic Model Refresh**  
- **Direct Lake is not supported for reports**, so use **IMPORT mode**  
- **Connect to SQL Endpoint**  
- **Refresh the SEMANTIC model** when using IMPORT mode  

### **Setup Git Integration with Azure DevOps**  
49. Create a **Project & Repo** in Azure DevOps  
50. In **Fabric Workspace**, enable **Git Integration**  
   - Specify the **Project & Branch**  
   - Ensure the **Azure DevOps account matches** the Fabric workspace user  
51. **Lock the main branch** using **Branch Policies** (Settings → Branch Policy)  

### **Continuous Integration (CI) in Fabric**  
- Multiple users can update an object  
- Option to **create a new Workspace** when checking out a branch  

### **Continuous Deployment (CD) in Fabric**  
- **Switch ADLS** using **Deployment Pipeline**  
- **Create a Deployment**  
  - Up to **10 stages** (e.g., DEV, PREPROD, PROD)  
  - If there is DEV Lakehouse attach to the notebook and then it needs to changed to PROD i.e. **Modify Notebook connections** (e.g., DEV to PROD Lakehouse)  use **Deployment Rules**
- **Deployment Rules**  
  - **Semantic Model:** Can be overridden  
  - **Data Pipelines & Notebooks:** Cannot override parameters  
  - **Dataflow:** Can be overridden  
- **Power BI Development**  
  - Convert report to **Power BI Project**  
  - Modify using **VS Code**  
  - **Push changes** to Azure DevOps