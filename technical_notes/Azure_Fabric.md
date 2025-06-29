---
title: "Fabric "
description: Technical Notes
date: 2025-01-29T00:00:00.000Z
---
# #2Read:

- [Delta Lake Tables For Optimal Direct Lake Performance In Fabric Python Notebook](https://fabric.guru/delta-lake-tables-for-optimal-direct-lake-performance-in-fabric-python-notebook)
- [What is Direct Lake mode in Fabric? - Lytix](https://lytix.be/what-is-direct-lake-mode-in-fabric/)
- [Power BI Direct Lake Mode in Microsoft Fabric - MSSQLTips.com](https://www.mssqltips.com/sqlservertip/7894/power-bi-direct-lake-mode-in-microsoft-fabric/)
- [Direct Lake overview - Microsoft Fabric | Microsoft Learn](https://learn.microsoft.com/en-us/fabric/fundamentals/direct-lake-overview#known-issues-and-limitations)
- 
- [Small Data And self service – PowerBI & Fabric and Data in General](https://datamonkeysite.com/page/2/)
- [#directlake - Microsoft Fabric | Power BI | Data Analytics | Data Science | GenAI](https://fabric.guru/tag/directlake)
- [fabric-samples-healthcare/analytics-bi-directlake-starschema at main · gregbeaumont/fabric-samples-healthcare](https://github.com/gregbeaumont/fabric-samples-healthcare/tree/main/analytics-bi-directlake-starschema)
- https://www.youtube.com/watch?v=U8FxNcerLa0
-  [ ] https://medium.com/@jacobrnnowjensen/putting-microsoft-fabric-to-the-test-34b7383a9546
- [ ] [Nice article for  Data Ingestion](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/scenarios/cloud-scale-analytics/best-practices/automated-ingestion-pattern)
- [ ] [Metadata driven] (https://techcommunity.microsoft.com/blog/fasttrackforazureblog/microsoft-fabric-metadata-driven-pipelines-with-mirrored-databases/4222480)

- [ ] https://milescole.dev/
- [ ] https://www.serverlesssql.com/category/fabric/page/3/
- [ ] [Usecase Fabric End to End flow](https://debruyn.dev/2023/fabric-end-to-end-use-case-analytics-engineering-part-1-dbt-with-the-lakehouse/)
- [ ] [Data Lake Implementation Fabric](
https://learn-it-all.medium.com/data-lake-implementation-using-microsoft-fabric-ccea72a8d162)
- [ ] [Fabric code to Get token](https://learn.microsoft.com/en-us/fabric/data-engineering/microsoft-spark-utilities#get-token)
- [ ] [Pandas Vs DuckDB Vs Pyspark in Fabric](https://medium.com/@mariusz_kujawski/exploring-python-libraries-for-data-engineering-in-ms-fabric-pandas-polars-duckdb-and-pyspark-8a7df3326193)
- [ ] [Sempy](https://learn.microsoft.com/en-us/python/api/semantic-link-sempy/sempy.fabric?view=semantic-link-python)
- [ ] [Invoke Fabric pipeline using Rest API](https://learn-it-all.medium.com/calling-a-fabric-data-pipeline-from-azure-data-factory-pipeline-b739418e6b34#:~:text=We%20essentially%20have%20two%20web,data%20pipeline%20using%20that%20token.&text=From%20the%20Fabric%20data%20pipeline,invoke%20from%20Azure%20Data%20factory.)
- [ ] [Connecting to Warehouse using JDBC](https://milescole.dev/data-engineering/2024/09/27/Another-Way-to-Connect-to-the-SQL-Endpoint.html)
- [ ] [Service Principal usage for Authentication](https://www.youtube.com/watch?v=_RXpvWjgZE8)

# Summary [[Architecture & Design]] [[Summarize]]
## **Overview of Data Ingestion Methods in Microsoft Fabric**

  #### **1.Data Ingestion**
-  **Data Flow Gen 2**:
	        	        - No/low-code, 300+ connectors, supports on-premise data via gateway.
		          	        - Good for small-medium datasets, but struggles with large data.
		            	        - Can embed in pipelines for orchestration.
	            
  - **Data Pipeline**:
               - Best for large datasets & orchestration (e.g., scheduling workflows).     
        - Limited transformation capabilities (requires notebooks/data flows).        
        - No on-premise data support yet.
            
   - **Fabric Notebooks**:        
        - Python-based, ideal for APIs, custom libraries, and complex transformations.            
        - Requires coding skills.
            
   - **Event Stream**:        
        - Real-time data ingestion (not covered in detail here).
            

#### **2. Shortcuts (File Replication)**

- **Purpose**: Create live syncs to external/internal files without ETL.    
- **Types**:    
    - **External Shortcuts**: Link to ADLS, Amazon S3, Dataverse.        
    - **Internal Shortcuts**: Link tables across Fabric (Lakehouse, KQL DB, but not Data Warehouse).        
- **Pros**: Near real-time, no merge logic needed.    
- **Cons**: Limited to supported storage systems; cross-region costs apply.   

#### **3. Database Mirroring (Coming Soon)**
- **Purpose**: Live sync of external databases (Snowflake, Cosmos DB, Azure SQL) into Fabric as Delta tables.    
- **Pros**:    
    - Real-time updates via CDC (Change Data Capture).        
    - No ETL needed; supports time-travel with Delta format.        
- **Cons**: Limited to specific databases (in preview as of recording).
    

#### **Key Considerations When Choosing a Method**

1. **Real-Time Need** → Shortcuts or Database Mirroring.    
2. **Team Skills** → Low-code (Data Flows/Pipelines) vs. Code (Notebooks).    
3. **Data Size** → Large datasets favor Pipelines/Notebooks.    
4. **Cross-Workspace Needs** → Data Pipelines have limitations.    
5. **Cost & Capacity** → Test different methods for efficiency.    
6. **On-Premise Data** → Only Data Flows support gateways currently.

# Medallion  Architecture [[Architecture & Design]]
#medallion #Architecture 
1. If there are different persona consuming the data at each layer, say for example  BI user need to access the **SILVER** layer data , Data Scientist for cleansed data in the BRONZE layer; In this scenario  the recommended pattern is to have each layer has a WORKSPACE. [The great "number of workspaces for medallion architecture in Microsoft Fabric" debate - Kevin Chant](https://www.kevinrchant.com/2024/05/03/the-great-number-of-workspaces-for-medallion-architecture-in-microsoft-fabric-debate/)
2. Security , if your QA data is production grade data then you need to keep them separate.
3.  Microsoft documentation recommends to save the notebook in a separate workspace. Also when using Deployment pipeline for DEV to higher environment  movement, with increase in number of items it is difficult to select the items, as there is no automated way to identify the changed items (i.e)  from a drop down list items needs to be selected manually.
>   When we implement DevOps, I do see a lot of suggestions online as well as from Microsoft that we need to have our notebooks in a different workspace than the lakehouse or warehouse. If you want to leverage Deployment Pipelines for Dev, Test, and Prod, having fewer workspaces with more Fabric items can also make the use of Deployment Pipelines more challenging. You can easily reach a point where you have to be very careful about Selective Deployments to ensure that you don’t accidentally publish something to a higher-level environment that is not ready. This only becomes more challenging as you increase the number of developers working in a space.
---


# [[Synapse]]
+ Dedicated SQL Pool - use when you have large scale data warehousing solutions.
+ Serverless SQl Pool - Pricing about the query. For 1 Tb of data it is free; This is applicable  when you  have lot of adhoc query and inconsistent volume.
	+  Query data from ADLS, CosmoDB
	+ it create  a querying surface area means a LOGICAL DATAWAREHOUSE over the files
	+  If you create database, all are external.
	+  LDW  store the metadata about the underlying
	+ You cannot create Table or Materilaized View or Index on the view
+ Fabric Vs Synapse
	+ Synapse we need to provision cluster then run the query 
	+ Synapase is Paas, client is responsible for managing the Platform
	+ Number of nodes provisioned are determined by the end user
- Why Azure Synapse:[Ref](https://www.youtube.com/watch?v=DL36iVJK8w4) (37:24)
	  1) Point is integration b/w ADLS and DataWarehouse is seemless
	  2) Need not worry about the compute, say for example you need a seperate DBricks or HDInsight
	  
# Capacity
[[FabricConcepts]]


#Azure/Fabric #Concept

## Capacity - Billing

4. Tenant - it is tied to an organisation; A tenant can have multiple CAPACITY; A Workspace is tied to a CAPACITY  
5. Compute + Storage (Similar to ADLS) + Mirroring + Network  
6. Choose the capacity  
	- Warehouse F64 ~ 32 cores  
	- Spark Compute ~ 64*2*3  
7. Except Power BI (as it is completely independent), Fabric capacity is sufficient or free trial for running all the other experiences.  
	- If you have a POWER BI premium license (P2..P5), then you can enable Fabric Capacity at the TENANT level  
	- For PPU (Premium per user), you cannot enable the Fabric capacity; You need to purchase FKU  
8. Capacity - Pay as you go OR Reserved  
	- **Pay as You Go** - there is no automatic pause/resume option  
	- **Reserved** - if you are purchasing 64 SKU and using only 32 SKU, then the remaining 32 SKU is wasted  
9. Licensing  
	- Two licenses are needed: one for Capacity License and one for User License  
	- **User Licensing:**  
			- F64 >= Needs a developer license but does not need a Viewer license  
			- Pro/Premium Per User - Capacity is shared; Premium Per User allows you to use Power BI datamart  
			- Power BI Premium Per Capacity - Dedicated capacity  



### Concept:
[[Architecture & Design]]
#Azure/Fabric #Architecture
10. If your Workspaces are allocated to a Capacity in the UK and the company is located in the UK, assume a new capacity is allocated in the USA and assigned to a WAREHOUSE, then consider the following:
   - This may break data protection law (as data needs to be transferred to WAREHOUSE in the USA, which certain organizations do not adhere to).
   - There could be egress fees across regions (networking cost).

   > If we now look at a scenario in which we have the same 3 Workspaces with Lakehouses and a Warehouse in each workspace,
   >except now we have an additional Fabric Capacity that has been created in the East US region.
   > If we allocate the Workspace that contains the Warehouse to this new capacity, even though it is in the same Fabric tenant, we will be moving data across regions if we load the Warehouse from the Cleansed Lakehouse.
   > Not ideal as this may break any data protection, plus there could be egress fees across regions.
   > [Pricing page](https://www.serverlesssql.com/microsoft-fabric-warehouse-for-the-database-administrator-dba-part-3-fabric-compute-and-the-sql-engine/)

---

# Fabric - Capacity Metric App
[[FabricConcepts]]
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
# Fabric - Power BI 
[[FabricConcepts]]
#PowerBI
## Semantic mode
1. Import mode:
	1. Store the data in DISK when querying the data is brought into memory
	2. **Uses VertiPaq storage engine**, so the data is compressed and optimized. For a 10GB dataset, it can be compressed to about 1GB in size.
	3. Supports Power Query (M) formula, Post-Calculated Column, Calculated tables or measures.
	4. **Disadvantage**
		1. Model need to be refreshed on scheduled basis
		2. **Full refresh:** Removes all data from the table and reloads it from the data source. This is expensive, so prefer **incremental refresh**
2.  Direct Query mode:
	1. Data is **not imported** but retrieved **on demand** via native queries. So less memory is needed, as only metadata is stored in memory.
	2. It can handle large volume of  data, as native query corresponding to the source is used to extract the data from Source.
	3. Support **automatic page refresh** (real-time updates every 15 minutes).
	4. **Limitations:**
		 1. **Query Performance:** Queries depend on the source system, which may cause delays.
		 2.  **DAX & Power Query Restrictions:** Only functions that can be translated into native queries are allowed.
		 3.  **Lack of Features:** No support for **Quick Insights** or **calculated tables**.
 3. Composite mode:
	 1. Composite mode allows mixing **Import and DirectQuery** storage modes within a single model.
	 2. Tables can be set to **Import, DirectQuery, or Dual** mode.
	 3. Frequently used data can be cached in memory, while real-time data is retrieved via DirectQuery.
	 4. Unlike DirectQuery, this mode allows calculated tables in DAX.
   ----
### Hybrid tables:

Hybrid tables combine **Import and DirectQuery partitions**, enabling efficient querying while keeping real-time updates.
##### **Key Benefits:**
- **Fast Access to Historical Data:** Older data remains in Import mode for quick retrieval.
    
- **Real-Time Updates:** Latest data is fetched via DirectQuery.
    
- **Optimized Performance:** Reduces database load while maintaining fresh data.
    

##### **Requirements:**

- **Premium Capacity Needed:** Hybrid tables require a **Premium** Power BI workspace.
    
- **Incremental Refresh:** Must enable **Get the latest data in real-time with DirectQuery**.
    
**Hybrid tables provide the best of both worlds—fast historical data access and real-time updates—making them ideal for handling large datasets efficiently.**

---


# Fabric - Data Engineering - Spark  


- Spark settings are specific to a **Workspace**.  

## Spark Pools  
4. **Starter Pools**  
   - Machines are pre-warmed in the background, making **session initialization faster**.  
   - Pre-warmed machines are **available** for immediate use.  
   - In a **Starter Pool**, you can change the **number** of machines, but **not** the **size** or **type** of machines.  

5. **Custom Pools**  
   - Custom Pools **take longer** to initialize (**2-3 minutes**) as they do not have pre-warmed machines.  
   - Recommended **only for development**, where longer cluster spawn times are acceptable.  
   - **Spark session initialization** takes **2-3 minutes** for Custom Pools, but **only seconds** for Starter Pools.  

## Node  
- Available node sizes: **S, M, L, XL, XXL**  
- Example: XXL = **64 cores, 512GB RAM**  
- Rule of thump is that each core is associated with **8GB** of **memory**. For node size that is small, it takes 4 cores and 32GB memory.

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
## Installing Library
1) Library can be installed at **Workspace level** or at the **Notebook**
2) In notebook, Use %pip commands to directly install feed libraries into your notebook. Library installed will be available for the current spark session. 
3) When libraries are installed at **Workspace level**,can be used by all notebooks and Spark jobs within that workspace, and are accessible across different sessions. So, if you need to create a common environment for all sessions in a workspace, it's best to use workspace-level libraries.

## MSSPARKUTIL  
- **Fastcp**: Uses **underlying AzCOPY**, enabling **fast data transfer**.  
	- Example: **70 million records copied in 20 seconds**.  
- **RunMultiple**: Using this one all the Notebook can be run under a single cluster . There by avoiding `TooManyRequestForCapacity`

## Performance: Stored Procedures vs. Notebooks  
- **Stored Procedures** are observed to be **faster** compared to **Notebooks**.  
- Read the full comparison:  
  [SQL Stored Procedures in Fabric Warehouse Offer Blazing Speed and Power at Scale](https://techcommunity.microsoft.com/blog/healthcareandlifesciencesblog/sql-stored-procedures-in-fabric-warehouse-offer-blazing-speed-and-power-at-scale/4287247)  
# Fabric - Item Level Permission  
4. **Intent of Item Level Permission**  
   - Allows sharing of items **without providing access** to the **Workspace**.  

5. **Sharing Requirements**  
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
6. **Currently in Preview Mode**.  
7. Uses **Role-Based Access Control (RBAC)** to grant access to specific folders.  
8. **Default Access:**  
   - By default, all users have the **DefaultReader** role, allowing them to read all folders.  
9. **OneLake Shortcuts:**  
   - **Permissions must be defined on the destination table**.  
   - **Defining permissions on the shortcut itself is not allowed**.  
# Fabric - Row Level Security (RLS)  

## Key Considerations  
10. **Applied at the Database Level**  
   - If a user tries to read data via the **OneLake path**, **RLS will not be enforced**.  
   - To prevent bypassing RLS, **grant only `READDATA` access** to the user.  
   - There is a risk of information leakage if an attacker writes a query with a specially crafted `WHERE` clause and, for example, a divide-by-zero error, to force an exception if the `WHERE` condition is true. This is known as a _side-channel attack_.

## Steps to Implement RLS  
11. **Input** → The **USERNAME** of the user.  
12. **Create a Function**  
   - Takes **USERNAME** as input.  
   - Defines logic to **restrict records** displayed.  
   - Must include **Schema Binding** when creating the function.  
13. **Create a Security Policy**  
   - Implement the **security policy** on the table.  
   - Pass the **column name** that should be restricted within the policy.  
# Fabric - Dynamic Data Masking (DDM)  

## Key Considerations  
-  **Applied at the Database Level**  
   - Similar to **Row-Level Security (RLS)**, it is enforced at the **database level**.  
   - **Unprivileged users with query permissions** can infer the actual data since the data isn’t physically obfuscated. So, original data can still be accessed using **Spark Notebooks**.  

-  **Combine with Object-Level Security**  
   - Dynamic Data Masking should be **used alongside Object-Level Security** for better protection.  

## Steps to Apply Dynamic Data Masking  
14. **Remove Existing Security Policies**  
   - If there are any **security policies** with **Schema Binding**, they must be **removed** before applying **Dynamic Data Masking**.  

15. **Apply Masking Function**  
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

16. Both **Managed** and **External** tables can be created on any file format.
17. We should attach **Lakehouse** to the notebook if we need to use the **saveTableAs** Spark API.
18. When you mention **LOCATION** in **CREATE TABLE** or **path** in the **saveTableAs** Spark API, the table will be created as an **External table**.
19. Both the **External** and **Managed** tables will be available under the **Table section**.
20. All the **Shortcut** created are like **Managed tables**.
---
# Fabric - In Relation with Delta Table
[Delta Lake Interoperability](https://learn.microsoft.com/en-us/fabric/get-started/delta-lake-interoperability)
[[delta_table_concepts]]

## Delta Table Column Mapping:
21. Even in Databricks, this is a preview feature. In MS Fabric, you can create a **Delta table with Column mapping** using Spark Notebooks.
   - Unfortunately, it is not compatible with the **Data Warehouse**. This means it cannot be queried in the Warehouse.
22. This feature allows us to **rename or drop** columns in a Delta table. By default, you cannot rename or drop a column in a Delta table.

## Pausing Delta Lake Logs:
23. If you pause the **Delta Lake logs**, any changes made to the Delta table will **not** be reflected in the Data Warehouse.
   
   **Use Case**:  
   When publishing is paused, Microsoft Fabric engines that read tables outside of the Warehouse see the data as it was before the pause. It ensures that reports remain stable and consistent, reflecting data from all tables as they existed before any changes were made to the tables. Once your data updates are complete, you can resume **Delta Lake Log publishing** to make all recent data changes visible to other analytical engines.

## Cloning:
24. Only the **metadata** (such as schema) of the source table is copied, not the underlying Parquet data files. This means the cloned table still references the original Parquet data files in One Lake without duplicating the data files. Cloning is sometimes referred to as a **"Zero Copy Clone"**.
25. A cloned table is **separate and independent** from its source table.
26. Any changes made in the source table are **not** reflected in the cloned table and vice-versa.
27. The clone is based on a **point-in-time** up to thirty days in the past or the current point-in-time. The new table is created with a timestamp based on **UTC**.

   **Limitation**:  
   - Table clones across warehouses in a workspace are not currently supported.  
   - Changes to the table schema prevent a clone from being created before the table schema change.

## Time Travel (Preview Feature):
28. Similar to the one in **Delta tables**.

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
## Restore to Previous Version:
29) Use this feature when you want to restore data. We can achieve this with a `DELETE` but this is  a file operation. When you use `RESTORE` it is a metadata operation.
30) To restore a delta table to  previous version, use `TIME TRAVEL` functionality or `RESTORE`
31) **Time Travel** we use read the previous version using `asOfVersion`, then `overwrite` target table
32) **Restore** we need to mention version(in this case it is previous version) that needs to be restored.

**Reference**: [Delta Change Data Feed in Fabric Lakehouses](https://www.serverlesssql.com/delta-change-data-feed-in-fabric-lakehouses/)
---

# Fabric - Warehousing [[FabricConcepts]]
### Foreign Key:
33) **Fabric Warehouse** supports **foreign key constraints** but they **can't be enforced**. Therefore, it's important that your **ETL process** tests for integrity between related tables when data is loaded.
34) It's still a good idea to create **foreign keys**. One good reason to create unenforced foreign keys is to allow **modeling tools**, like **Power BI Desktop**, to automatically detect and create relationships between tables in the semantic model.

**Reference**: [Dimensional Modeling in Fabric Data Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/dimensional-modeling-dimension-tables)

# Fabric - Mirroring - Shortcut

## Mirroring  
Mirroring is a data replication method where data is brought to the lakehouse using CDC (Change Data Capture) incrementally.  
### **How it works**
1. **Connect**: Link to an external database via a supported connector.    
2. **Mirror**: Fabric creates a read-only snapshot that stays synchronized.    
3. **Query**: Use T-SQL, Spark, or Power BI to analyze the mirrored data.
### **Types of Mirroring in Microsoft Fabric**

Microsoft Fabric provides **three mirroring approaches** to bring data into **OneLake**, enabling analytics without complex ETL pipelines.
#### 1. **Database Mirroring**
- **What it is:** Replicates **entire databases and tables** from source systems into OneLake.    
- **Purpose:** Consolidates data from multiple systems into a unified analytics platform.    
- **Example:** Replicating a full Azure SQL Database or SQL Server into Fabric for centralized reporting.  

#### 2. **Metadata Mirroring**
- **What it is:** Syncs only the **metadata** (catalogs, schemas, tables), not the actual data.    
- **How it works:** Uses **shortcuts** so the data stays at the source but appears accessible within Fabric.    
- **Example:** Mirroring metadata from **Azure Databricks** lets you analyze data stored in Databricks without copying it to Fabric.
#### 3. **Open Mirroring**
- **What it is:** Allows developers to **write change data directly** into a mirrored database using the **Delta Lake format** and public APIs.    
- **Use case:** Enables custom or third-party applications to push updates to Fabric seamlessly.    
- **Example:** A retail app writes transaction updates directly into Fabric using open mirroring.
### Cost of Mirroring in Microsoft Fabric

| **Aspect**              | **Cost Details**                                                          |
| ----------------------- | ------------------------------------------------------------------------- |
| **Storage**             | Free up to 1 TB per CU; billed beyond limit or if capacity is **paused**. |
| **Replication Compute** | Free (no capacity consumption).                                           |
| **Query Compute**       | Charged at standard rates (SQL, Spark, Power BI- Direct Lake).            |
| **Setup Requirement**   | Requires active Fabric capacity **only during initial setup**.            |
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

4. If your source has **frequent changes** and requires **24/7 data availability**, a **dedicated lower-grade capacity** for mirroring may be more cost-effective than using a high-end compute resource.  
5. If your source has **small, infrequent changes**, consider a separate capacity with scheduled start and pause times to avoid unnecessary costs.  
   - Capacity start and pause can be managed via **Azure REST APIs**.  
#Fabric - Warehousing - Data Recovery

### Restore Points:
- **Warehouse deletes** both the system-created and user-defined restore points at the expiry of the **30 calendar day retention period**.
- System-created and user-generated restore points **can't be created** when the **Microsoft Fabric capacity** is paused.
- At any point in time, **Warehouse** is guaranteed to be able to store up to **180 system-generated restore points** as long as these restore points haven't reached the thirty-day retention period.
- There will be **Storage** and **Compute** costs associated with the Restore Point.

### Limitation:
6) A **recovery point** can't be restored to create a new warehouse with a different name, either within or across the Microsoft Fabric workspaces.
7) **Restore points** can't be retained beyond the default **thirty calendar day** retention period. This retention period isn't currently configurable.

### Clone Table:
#Fabric -ALTER

### Reference:
- [The Reality of ALTER Table in Fabric Warehouses](https://www.serverlesssql.com/the-reality-of-alter-table-in-fabric-warehouses-2/)

### Key Points:
8) **ALTER Table Usage**:
   - **Supports adding a column** but **does not support** dropping a column or changing the datatype of a column.

9) **To Add a Column for a Table in the Lakehouse**:
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



# Fabric Notebook 
#Architecture 
 ## Methods for Parallel Spark Orchestration [[Architecture & Design]]
 ([Cluster Configuration Secrets for Spark: Unlocking Parallel Processing Power | Miles Cole](https://milescole.dev/optimization/2024/02/19/Unlocking-Parallel-Processing-Power.html))
 1) ## Running Many Jobs on a Single High-Concurrency Cluster:
	 Issues with this approach:
		**Concurrency Limited by Executors/Workers**: 
		    >In environments like Databricks and Fabric, as opposed to open-source Spark, the number of executors is directly tied to the number of workers; a cluster with 4 workers equates to having 4 executors. This configuration limits the ability to run concurrent operations, as each executor is dedicated to a single job. Consequently, attempting to run 8 notebooks on a 4-worker cluster results in half of the notebooks being queued until an executor becomes available.
        **Shared Driver Node Becomes a Bottleneck:**
        Contrary to the distinct driver-executor model in open-source Spark, platforms like Databricks and Fabric utilize a shared driver across all jobs in a high-concurrency cluster. All operations are executed through a single JVM, which can quickly become overwhelmed by resource demands.
 2) ## Dedicated Job Cluster for Each Job:
	 Issues with this approach:
	- **Risk of Underutilized Compute Resources:**
		- When each job runs on its dedicated compute resources without the possibility of sharing, there’s a risk that even minor tasks might not fully utilize the allocated resources. For instance, a small job running on a single-node, 4-core cluster might not use all available computing power, leading to inefficiencies.
 3) ## Use Multithreading for concurrent job execution:
	 - Idea here is to run each job as a thread. **Things to keeps in mind** are how do we cancel the each job, because there is no inherent feature to cancel a thread that is already in sleep mode. `Concurrent.Futures` support cancelling of threads that have not started.
	 - Capturing the thread status.
	
   

 ## Limitations
- [Microsoft Fabric Notebook Limitations](https://learn.microsoft.com/en-us/fabric/data-engineering/notebook-limitation)  
	### Size/Number of Parameters return:
	 1) **Returning Parameters from a Notebook**  
	- The number of rows that can be returned is **limited to 10K rows or 5MB**.  
	
	1) **Accessing Return Parameters as JSON**  
	- Notebook return values can be accessed in **JSON format** using the following syntax:  
	  ```text
	  @Json(activity('Test Notebook').output.result.exitValue).message
	  ```
	  [Reference](https://community.fabric.microsoft.com/t5/Data-Pipeline/Referencing-notebook-exit-value-as-a-variable-in-a-data-pipeline/m-p/3507053)

## Spark Session  
10) Shared Spark Session Between Master and Child Notebooks  
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

## Fabric Notebook - Concurrency
 - [Fabric Notebook Concurrency Explained: Update — Advancing Analytics](https://www.advancinganalytics.co.uk/blog/2023/12/13/fabric-notebook-concurrency-explained-whjpa)
 - With Capacity there is always a QUEUE capacity associated. This QUEUE capacity is only applicable for BATCH jobs. 
 - From reference link, seems like NOTEBOOK related activities , even if it is triggered from PIPELINE is treated as not BATCH job, but as **Interactive** job.
 - Degree of Parallelism mean number of concurrent notebook that can be run , it can notebook triggered from **Fabric Pipeline** and **Interactive Notebook** session. If DOP is 6 , it can be 6 notebooks triggered from the Pipeline or it can 4 notebooks triggered from the Pipeline and 2 Spark Session.
 - ` Cores Available >= number of nodes * number of cores * DOP`
 -  ### Capacity Calculation example  (F64 Equivalent, 128 cores available)
		 **Default Starter Pool**
		 10 nodes  
		  Node size: Medium (8 vCores per node)  
		Result: Only 1 notebook can run at once in the default starter pool. 
			
	$$
		
		DOP = \frac{128}{(10 \times 8)} = \frac{128}{80} = 1.6 \Rightarrow \text{Rounded Down to } 1
		
	$$
		 
	**Optimized Setup for Higher DOP**
			To improve concurrency, a custom pool is used with:  
			2 nodes instead of 10  
			Small node size (4 vCores per node)  
	 - Result:** **16 notebooks** can run simultaneously in this optimized setup
		
			
		$$
			
			DOP = \frac{128}{(2 \times 4)} = \frac{128}{8} = 16
			
		$$
			
		
## Fabric Notebook - Warehouse Table  

## Sync Issues Between Warehouse Table and Notebook  
11. **Data Discrepancy:** There may be a synchronization issue between **Warehouse tables and Notebooks**, leading to:  
   - **Count mismatches** when querying data in the notebook versus querying via `SELECT * FROM` in the Lakehouse table.  
   - **Duplicated rows** or **inconsistent results** between Notebook and SQL Endpoint.  

### References:  
- [Duplicated Rows Between Notebook and SQL Endpoint](https://community.fabric.microsoft.com/t5/Data-Engineering/Duplicated-rows-between-notebook-and-SQL-Endpoint/m-p/3707317)  
- [Lakehouse Data Discrepancy Between Notebook and SQL Endpoint](https://community.fabric.microsoft.com/t5/Data-Science/Lakehouse-data-discrepancy-between-notebook-and-SQL-endpoint-or/m-p/4069421)  
# Fabric SQL - Limitations in Warehouse  

## Temporary Tables  
12. **Limited Usage:** Temporary tables are supported but with restrictions:  
   - You **cannot join** a temporary table with a normal table.  
   - `INSERT INTO` with `SELECT * FROM` a normal table **is not supported**.  
   - **Reference:** [Temp Tables in Fabric Warehouses](https://www.serverlesssql.com/temp-tables-in-fabric-warehouses/)  

## Cursor Support  
- **Not Supported:** Fabric SQL **does not support cursors** because **Synapse uses a columnar format** for data storage.  

---

# ALTER Statement Limitations  
13. **Dropping Columns & Changing Datatypes:**  
   - You **cannot drop columns** or **change the datatype** using `ALTER TABLE`.  
   - **Time Travel Functionality is Lost:** When you apply an `ALTER TABLE`, **time travel tracking is reset** to the timestamp of the alteration.  

> _“It’s worth noting that when issuing an ALTER TABLE statement, it resets the date/time the table is being tracked to the date/time the table was altered.  
> For the existing functionality of altering a table to add primary, unique, and foreign key constraints, it probably did not surface that regularly as these are often implemented when a table is first created.  
> But now we can add new columns, this issue may surface more regularly.”_  


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

14. **Append-Only Table for Metadata**  
15. **Monitor Lock & Retry the INSERT** (requires privileged access)  
16. **In Databricks:**  
   - Handled using **Isolation Levels**  
   - **Partitioning the table**  

# Fabric Warehouse - Performance  
[Warehouse performance guidelines - Microsoft Fabric | Microsoft Learn](https://learn.microsoft.com/en-us/fabric/data-warehouse/guidelines-warehouse-performance)
## Automatic Statistics & Caching  

-  **Cold run  (cold cache) performance** - Caching is managed automatically by Fabric, with limited user control.  [Caching with local SSD and memory](https://learn.microsoft.com/en-us/fabric/data-warehouse/caching) is automatic. The first 1-3 executions of a query perform noticeably slower than subsequent executions. If you are experiencing cold run performance issues, then try creating **manual statistics** for the column used in GROUP BY, SORT ,filter condition. However, if the first run's performance is not critical, you can rely on **automatic statistics** that will be generated in the first query and will continue to be leveraged in subsequent runs (so long as underlying data does not change significantly).
## Group INSERT statements into batches (avoid trickle inserts)
- Use singleton INSERTS incase if the target table has less than a million or thousand records. But incase of large table ,it is recommended to have INSERTS in a batch.
  
## Minimize transaction sizes
- *INSERT, UPDATE, and DELETE statements run in a transaction. When they fail, they must be rolled back. To reduce the potential for a long rollback, minimize transaction sizes whenever possible. Minimizing transaction sizes can be done by dividing INSERT, UPDATE, and DELETE statements into parts.*
- If your INSERT is expected to take 1 hour, it  is better to break up the INSERTS into 4 parts.
- Use CTAS , to write the data you want to keep in a table rather than using DELETE. *If a CTAS takes the same amount of time, it's safer to run since it has minimal transaction logging and can be canceled quickly if needed.*
## Collocate client applications and Microsoft Fabric
 - Client application should be in same region as Fabric Warehouse
 ## Utilize star schema data design
 ## Choose the best data type for performance
 - Use integer-based  data types if possible. *SORT, JOIN, and GROUP BY operations complete faster on integers than on character data*
 - When defining the table, use **smallest data type** that will improve the query performance.*If the **longest value** in a column is **25 characters**, then define your column as **VARCHAR(25)**. Avoid defining all character columns with a large default length.*
 ## Data Compaction
 [Announcing: Automatic Data Compaction for Fabric Warehouse | Microsoft Fabric Blog | Microsoft Fabric](https://blog.fabric.microsoft.com/en-US/blog/announcing-automatic-data-compaction-for-fabric-warehouse//)
 - It is process of merging the smaller parquet file into larger one. This will reduce the number of parquet files , there by improving the read performance.  Also, use Delta Lake feature called Delete Vectors, so that records that are delete are stored in a separate file and during this compaction process, these deleted records are  removed from the main parquet file . Data Compaction is *seamlessly integrated into the warehouse. As queries are executed, the system identifies tables that could benefit from compaction and performs necessary evaluations. There is no manual way to trigger data compaction.*
 

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
17. **Automatically enabled by Microsoft Fabric:**  
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
- Natural key Vs Surrogate key
	- **Artificial** keys created during the **ETL process**
	- Simple **integer** values (e.g., `1`, `2`, `3`) used as **primary keys (PK)** and **foreign keys (FK)**
	- Smaller in size , as it is integer→ **better performance** and **storage efficiency**.Help handle **dummy/missing values**.
- **Practical Guideline**
	- **Always use surrogate keys** for **primary** and **foreign keys** in both **fact** and **dimension** tables. Except Date dimensions, since the natural key like `YYYYMMDD` since it's consistent and predictable.
	- You **can optionally keep natural keys** in dimension tables for reference, especially if they are small and useful.
	- 
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
### NULL in Dimensions
- NULL should not be there in the FK
- **Always replace nulls** in dimension attributes with **descriptive placeholders**:
	- E.g., “No Promotion Available”, “No Category”, or “Unknown”.
    - For date dimensions, use a valid placeholder like `1900-01-01`.
- **Why replace nulls in dimensions?**
	- Nulls are **ambiguous** and confusing to end users.
    - Descriptive values improve **clarity and usability** in reports.
    - BI tools may **ignore nulls** by default in visualizations and groupings, causing misleading results.

### Handling Hierarchical Structures  
- **Balanced hierarchy**  
- **Unbalanced hierarchy (Parent-Child, e.g., Employee-Manager)**
- In a dimension it is better to transform and store the hierarchy levels in the dimension as columns.  
- Never normalize hierarchical data by building a Star Schema like pattern. Rather denormalize the data . Include the `product` and `category` in the same table. Create a `single flattened table` combining all the `product` and `categories`.
- **Create combined attributes for better usability**:
	- - Examples:    
	    - `Year_Quarter` = “2023-Q2”
         - `City_State` = “Nashville, TN”
	- Especially helpful when:    
	    - Attribute names are **ambiguous** (e.g., same city name in multiple states).        
	    - Users want **quick filtering/grouping** in BI tools.
 
  - **If using parent-child relationships in Power BI**, avoid them for large dimensions.  
  > - **Snip:** If you choose not to naturalize the hierarchy, you can still create a hierarchy based on a parent-child relationship in a Power BI semantic model. However, this approach isn't recommended for large dimensions.

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
- - A **degenerate dimension (DD)** is a **dimension attribute stored directly in the fact table** without a corresponding dimension table.
    - It typically contains **transaction identifiers** like: 
	    - **Order ID**        
	    - **Invoice number**
		- **Payment ID**
	- These attributes are useful for grouping, summarizing or aggregation
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
#### Basics/**Terminology**
- **Grain** defines the **most atomic level** of data stored in the table. Example: "Profit by region and by day" → Grain is **one row per region per day**.
- Fact usually contain **numeric** and **additive** (can be aggregated, e.g., summed up).
- Facts can be often derived from events or transaction.

### Measures
- **Fact** measures can be classified based on how they can be aggregated across dimensions in data analytics.
	- Fully-Additive Facts, means can be added across **all dimensions**.e.g: `Units Sold`, `Revenue`
	- Semi-Additive Facts, it can be added only few dimensions.eg: `Account balance` it has to be summed up based on the Portfolio, portfolio can be  Cash ,Stocks.
		-  Can sum across **portfolio types**    
		-  Cannot sum across **dates**
	- Non-Additive Facts, cannot be summed across **any dimension**.
		- **Examples**: `Price per Unit`, `Ratios`, `Percentages`, `Inventory Level`
		-  Adding up prices or percentages gives **meaningless results**
### Nulls in Measures (Fact Values)
- SQL and BI tools (Power BI, Tableau, etc.) **handle nulls gracefully** during aggregations like `SUM`, `AVG`, `MIN`
-  While nulls are excluded from aggregates, the **interpretation can be misleading** (e.g., average _per day_ vs. average _when transaction occurred_).
-  If nulls represent **"zero"**, consider replacing them with `0` to reflect actual absence of activity.
- Nulls in Foreign Keys (Dimensions)
	- - **Problematic**: Null foreign keys **break relationships** between fact and dimension tables.
	- **Solution:** 
	- Use **dummy keys** (e.g., `999`, `-1`) to indicate special cases like:        
        - Unknown Portfolio            
        - Missing Date
    - Add corresponding **dummy rows in dimension tables** (e.g., Portfolio = "Unknown", Date = "01-Jan-1900").
 
### Primary Key  
- **Fact tables typically do not need a primary key**.  That's because it doesn't typically serve a useful purpose, and it would unnecessarily increase the table storage size.
- Uniqueness is implied by **dimension keys & attributes**.  

### Audit Attributes  
- Track changes in the fact table.  

### Types of Fact Tables  
1️⃣ **Transaction Fact Tables** → Tracks individual transactions. 
	- The **grain** of the table is **one transaction per row**.
	- Each row includes **Measurements** (e.g., units sold, call duration) and **Foreign keys** linking to dimensions (e.g., product, time, location)
	-  Example: Sales Transaction, Customer calls
	
2️⃣ **Periodic Snapshot Fact Tables** → Stores measurements at predefined intervals.  
   > Example: There might be millions of stock movements in a day (which could be stored in a transaction fact table), but your analysis is only concerned with trends of end-of-day stock levels. 
   - Often **built on top of transactional tables** through scheduled summarization.
   - If no events occur in a period, Use **0** if "nothing happened" is meaningful (e.g., 0 sales) **OR** Use **NULL** if the absence should be excluded from calculations (e.g., no business on weekends)
  
3️⃣ **Accumulating Snapshot Fact Tables** → Captures milestones in a process.  
### Factless Fact Table  
- Focus is solely on **capturing events** with dimensions—**no measures** are present.
- Example: **Product Promotion Tracking**:
	- Captures: promo code, product, campaign details.
    - No metrics like sales uplift.
    -  Useful for tracking **when** and **where** promotions occurred.
> *Records events/occurrences (e.g., Tracking student participation, Absence tracking, Employee registeration).*  

### Measure Types  
- **Semi-additive Measures** → Aggregate across some dimensions but not all (e.g., Account Balance).  
- **Non-additive Measures** → Cannot be aggregated (e.g., Profit Margin) because summing or averaging profit margins across products is meaningless because it ignores the underlying revenue and profit values 



### Aggregate Fact Tables  
- Can be pre-aggregated in **Warehouse tables** or **Power BI DirectQuery storage mode**.  
- Power BI can create **user-defined aggregations** to achieve same result.  

---
### Steps to Design a Fact Table
18. Identify the Business Process
	1. Determine what process you want to analyze (e.g., **sales**, **order fulfillment**).
	2. This defines the scope and focus of your fact table.
19. Define the Grain
	1. Decide the **level of detail** each row represents. E.g., one row = one transaction, one daily summary per location, etc.
	2.  **Finer grain (more detail)** is preferred:
		1.  Keeps analysis flexible.
		2.  Avoids limitations caused by early aggregations.
		3.  Allows later aggregation in data marts for specific use cases.
20. Identify Dimensions
21. Identify the Facts
	1.  Determine what **metrics or measures** need to be captured.
	2. Facts are the **numerical values** used for analysis (e.g., quantity sold, revenue).
22. 
______________________

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
23. Create a **Project & Repo** in Azure DevOps  
24. In **Fabric Workspace**, enable **Git Integration**  
   - Specify the **Project & Branch**  
   - Ensure the **Azure DevOps account matches** the Fabric workspace user  
25. **Lock the main branch** using **Branch Policies** (Settings → Branch Policy)  

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
### Fabric Administration and Governance
1) Global Admin - user who can provide access to any service like Fabric, PowerApps
2) Fabric Admin - user who has full control over the Fabric
3) PowerBI admin portal is same as Fabric Admin portal
4) 


