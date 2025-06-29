---
date: 2025-01-31
---
# Data Engineering Resources

## Do We Need SCD Type 2 in This Decade?
- [Functional Data Engineering: A Modern Paradigm for Batch Data Processing](https://maximebeauchemin.medium.com/functional-data-engineering-a-modern-paradigm-for-batch-data-processing-2327ec32c42a)
- [Lambda and Kappa Architecture with Databricks](https://nileshprajapati.net/blog/2020/lambda-and-kappa-architecture-with-databricks/)

## Must Read for Data Engineers
- [10 Things I Learned from Reading Fundamentals of Data Engineering](https://blog.det.life/10-things-i-learned-from-reading-fundamentals-of-data-engineering-eea5dc8e5fb7)

## Try It for Fun
- [How I Turned My Company's Docs into a Searchable Database with OpenAI](https://towardsdatascience.com/how-i-turned-my-companys-docs-into-a-searchable-database-with-openai-4f2d34bd8736)

## Reference: Data Lake Architecture
- [AWS Data Lake Architecture Reference](https://aws-reference-architectures.gitbook.io/datalake/)

## Read
- [Spark Data Pipeline End-to-End](https://medium.com/everything-full-stack/spark-data-pipeline-end-to-end-3154cf95ded7)
## Data Warehouse
- [GitHub - lxxxng/sales_data_engineer: End-to-end data warehouse project in SQL Server using Medallion Architecture (Bronze, Silver, Gold). ETL pipeline integrates ERP & CRM CSV data into a star schema for analytics.](https://github.com/lxxxng/sales_data_engineer)
- [GitHub - TungPhamDuy/azure-data-warehouse-implementation: A comprehensive data warehousing project on Azure, utilizing Azure Synapse Analytics to build a scalable and performant data warehouse. Focuses on star schema design, ELT processes, and optimizing data for business intelligence and analytics on bikeshare data.](https://github.com/TungPhamDuy/azure-data-warehouse-implementation)
- [GitHub - sidsax23/Dairy-Data-Warehouse: A data warehouse for a milk marketing company akin to Amul with a well-defined STAR Schema, information package diagrams and with business queries implemented on the same.](https://github.com/sidsax23/Dairy-Data-Warehouse/tree/main)
- 

## Must Read
- [Learning Notes on Designing Data-Intensive Applications](https://elfi-y.medium.com/learning-notes-on-designing-data-intensive-applications-vi-f505eec740e7)

## Schema Design
- Predicate Pushdown Works for Nested Schema:  
  - [TechMagie - Spark](https://techmagie.wordpress.com/category/spark/)
  - [Metadata-Driven ETL](https://www.linkedin.com/pulse/metadata-driven-etl-yogaraj-kathirvelu?articleId=6628072537530093568)
  - [Democratizing Data](https://databricks.com/session_na20/democratizing-data)
  - [Handling Upstream Data Changes via Change Data Capture](https://medium.com/swlh/handling-upstream-data-changes-via-change-data-capture-4b22b8c75363)
  - [How We Built Our New Modern ETL Pipeline](https://inside.getyourguide.com/blog/2019/12/11/how-we-built-our-new-modern-etl-pipeline-part-1)
  - [Building a Modern Batch Data Warehouse Without Updates](https://towardsdatascience.com/building-a-modern-batch-data-warehouse-without-updates-7819bfa3c1ee)
  - [Data Engineering Digest](https://medium.com/data-plumbers/data-engineering-digest-11-april-2020-80e8079a305)

## Data Lake Essentials
- [Part 1: Storage and Data Processing](https://www.qubole.com/blog/data-lake-essentials-part-1-storage-and-data-processing/)
- [Data Lake Ingestion Strategies](https://sbhoracle.wordpress.com/2019/01/13/data-lake-ingestion-strategies/)

## **How to Choose the Right ETL Tool**
#### **Key Steps in the Evaluation Process**:

1. **Assess Current Needs & Use Cases**    
    - Identify why an ETL tool is needed (e.g., data integration, automation).        
    - Define **must-have features** (e.g., specific data source connectors, ease of use).        
    - Determine who will use the tool (technical vs. business users).
     
2. **Create an Evaluation Matrix**    
    - Compare tools based on criteria such as:        
        - **Cost & Budget**            
        - **Available Connectors** (range and compatibility)            
        - **Ease of Use** (user-friendliness for the team)            
        - **Additional Capabilities** (e.g., data quality checks, scheduling)          
         - Assign **importance weights** (e.g., 1-5 scale) to prioritize key factors.
3. **Research & Reviews**    
    - Check **customer reviews** and industry reports (e.g., Gartner Magic Quadrant).        
    - Evaluate **vendor support** (training, dedicated assistance).
        
4. **Shortlist & Test**    
    - Eliminate tools that don’t meet must-have criteria.        
    - Contact vendors for **demos, trials, or proof-of-concept testing**.        
    - Test shortlisted tools in real-world scenarios before committing.
# Bi-Temporal Modelling

🔗 [Bi-Temporal Data Modeling with Envelope](https://blog.cloudera.com/bi-temporal-data-modeling-with-envelope/)

## Key Takeaways:
5. **Purpose of Temporal Modeling**  
   - Helps **avoid expensive aggregations** when retrieving the latest information.  
   - Efficiently tracks changes in data over time.

6. **Capturing All Changes in Data**  
   - Add **start and end date** at the source.  
   - These dates represent **business time** (not system time).

7. **Solution: SCD Type 2 Table**  
   - Stores **system-generated dates** for each record version.  
   - Enables **rollback to a previous point in time** for historical analysis.

# Infra Provisioning

8. **Auto Scaling Feature**  
   - Works only on **ETL workloads** and **SQL Warehouses**.  
   - **Not effective** for **iterative workloads** like **Machine Learning**.

# AWS Lake House Architecture and Metadata Management

## Recommended Reading

- **AWS Data Lake Reference Architecture**  
  [Read here](https://aws-reference-architectures.gitbook.io/datalake/)

- **Discovering Metadata with AWS Lake Formation (Part 1)**  
  [Read here](https://aws.amazon.com/blogs/big-data/discovering-metadata-with-aws-lake-formation-part-1/)

- **Building a Lake House Architecture on AWS**  
  [Read here](https://aws.amazon.com/blogs/big-data/build-a-lake-house-architecture-on-aws/)

## AWS Lake House Architecture Overview

AWS Lake Formation provides a **centralized management** system for **data lake administration**, offering **granular table- and column-level permissions**.  
Key features include:

- Controlled access to **databases and tables** in the data lake.  
- Permissions enforced across multiple services, including:
  - **AWS Glue**
  - **Amazon EMR**
  - **Amazon Athena**
  - **Redshift Spectrum**

This ensures **secure and governed** access to data, allowing users and groups to work only with **authorized tables and columns**.

# Ingestion Strategy: CDC & SCD  

## Change Data Capture (CDC)  

CDC is a method of detecting and extracting **new or updated records** in a source and loading **only** this new information into the destination.  
CDC can be implemented as either **PUSH or PULL**, for example, **querying from a source as a nightly job**.

### Primary CDC Implementation Methods  

9. **Log-Based CDC**  
   - One of the most **efficient** CDC strategies.  
   - Every **new database transaction** is recorded in a **log file**.  
   - The polling system extracts data **without impacting** the source database.  

10. **Query-Based CDC**  
   - The database is **queried** to fetch changes.  
   - Requires **additional metadata** like **timestamps** to track changes.  
   - More **resource-intensive** than log-based CDC.  

11. **Trigger-Based CDC**  
   - Uses **database triggers** to notify when data is written or updated.  
   - Relies on **auditing metadata** like timestamps.  
   - **Higher performance impact** due to additional write operations.  

---

### Log-Based CDC vs. Query-Based CDC  

| Feature                  | Query-Based CDC ✅/🛑 | Log-Based CDC ✅/🛑 |
|--------------------------|---------------------|---------------------|
| **Ease of Setup**        | ✅ Simple JDBC connection  | 🛑 More setup required |
| **Permissions**          | ✅ Requires only read access | 🛑 Needs higher privileges |
| **Tracking DELETEs**     | 🛑 Cannot track deletes | ✅ Captures all operations (INSERT, UPDATE, DELETE) |
| **Multiple Events in Interval** | 🛑 Only captures latest state | ✅ Captures every change |
| **Performance Impact**   | 🛑 Polling increases load | ✅ Lower impact, reads from logs |
| **Latency**              | 🛑 Higher latency (depends on polling) | ✅ Lower latency |

---

## Slowly Changing Dimensions (SCD)  

SCD is primarily used for **data storage and historical tracking** at the **target system**.

- **SCD Type 1:**  
  - **Overwrites** data.  
  - When a change occurs, **existing data is updated** (no history).  

- **SCD Type 2:**  
  - **Maintains history** of changes.  
  - Commonly used in scenarios where historical tracking is necessary.  
 --- 
# AWS Data Lakes  

**Reference:** [AWS Data Lake Best Practices](https://docs.aws.amazon.com/whitepapers/latest/best-practices-building-data-lake-for-games/monitoring.html)  
### In-place Querying
- Directly query data in S3 using 
- Athena, Redshift Spectrum
-  **Amazon Athena**
       - **Serverless** interactive SQL query service        
    - Supports formats like **CSV, JSON, Parquet, ORC, Avro**        
    - **Pay-per-query**, based on data scanned        
    - Ideal for **ad hoc querying and data discovery**
        
- **Redshift Spectrum**    
    - Queries **S3 directly using Redshift SQL**        
    - Supports **complex queries** and **large user bases**        
    - Can span queries across **Redshift + S3**        
    - Optimized and parallelized for performance
        
- **Kinesis Data Analytics**    
    - Use **SQL on streaming data**        
    - Supports **continuous queries** that output results in real time
## Cost Optimization for Storage
 - **S3 Lifecycle Management**
	 -  Automate storage tier transitions and deletions
	 - Rules based on object age, tags, or paths
- S3 Storage Class Analysis
- Intelligent Tiering
- Glacier and Glacier Deep Archive
- Data Formats
## **S3 Encryption Options**

S3 supports **encryption to protect data at rest** or during transmission, using:
### **1. Server-Side Encryption (SSE)**
S3 handles encryption **before storing** and decryption **during retrieval**.
#### a. **SSE-S3 (S3 Managed Keys)**
- S3 manages encryption keys automatically.    
- Encryption/decryption is **transparent** to users.    
- If a bucket is public, **even anonymous users** can access encrypted data.
#### b. **SSE-KMS (AWS KMS Managed Keys)**

- You manage keys using **AWS Key Management Service (KMS)**.    
- Access control is stricter — users need:    
    - Permission to the **S3 bucket** and        
    - Access to the **KMS key**        
- Prevents public access even if the bucket is exposed.
    
#### c. **SSE-C (Customer-Provided Keys)**
- The **client provides and manages** the encryption key.    
- Every request must include the correct key.    
- If the wrong key is used, access is denied.    
- Suitable for organizations that **must control key storage**.  
### **2. Client-Side Encryption**

- Data is encrypted **before uploading to S3**.    
- S3 stores encrypted data **as-is**.    
- Encryption/decryption handled **entirely by the client**.    
- Clients can use **KMS or custom key management systems**.
## Security and Protection in a Data Lake (S3)
### **Access Control Mechanisms**

- **User-based policies** (recommended):    
    - Assigned to users or user groups based on roles.        
    - Easier to manage with roles and permissions.        
- **Resource-based policies**:    
    - Attached directly to S3 buckets or objects.        
    - Useful for enforcing restrictions (e.g., access only from corporate IPs).        
- Use **both in combination** for robust access control.
### **Data Protection Features in S3**
- **11 9s Durability**:  
    Data is redundantly stored across multiple **AZs** and devices.    
- **Versioning**:    
    - Retains previous versions of objects.        
    - Allows recovery from accidental overwrites/deletes.        
- **MFA Delete**:    
    - Requires multi-factor authentication for delete operations.        
    - Prevents accidental or malicious deletions.        
- **Lifecycle Policies**:    
    - Automatically transition or expire old versions or unused data.
### **Advanced Data Protection**
- **Cross-Region Replication (CRR)**:    
    - Automatically replicates data to a second AWS region for **disaster recovery**.        
- **Object Tagging**:    
    - Classify data (e.g., PHI – Protected Health Information).        
    - Create **tag-based access controls** to restrict sensitive data.
### Protect Your Data Using Data Perimeters
#### **Scenario 1: Accessing an Untrusted Resource**
- **Threat**: Users/applications might unknowingly access an attacker-controlled AWS resource (e.g., misleading S3 bucket names).    
- **Risk**:    
    - **Reading**: Malicious files/configurations could infect systems or cause harm.        
    - **Writing**: Sensitive data could be exfiltrated to an attacker's bucket.        
- **Solution**:    
    - Add **conditional checks** in **IAM policies** (e.g., validate account ownership or enforce VPC access).        
    - **Limitation**: Not scalable across many accounts and policies.        
    - **Better Approach**: Use **Service Control Policies (SCPs)** for centralized, org-level enforcement.      
####  **Scenario 2: Untrusted Principal Accessing Your Resource**
- **Threat**: External (untrusted) accounts access your resource via misconfigured **resource-based policies** (e.g., S3, KMS).    
- **Risk**: Data lake breaches and large-scale data exposure.    
- **Limitation**:    
    - SCPs **do not apply** to external principals or **resource-based policies**.        
- **Solution**:    
    - Use **Resource Control Policies (RCPs)** to block access by untrusted external principals.        
    - RCPs apply organization-wide and complement SCPs.       
####  **Recommended Approach**
- Use **SCPs** for internal IAM roles/users.    
- Use **RCPs** for external principals via resource-based policies.    
- Combine both for a comprehensive **data perimeter**.    
- Enforce **VPC endpoint policies/checks** to validate trusted network access.   
####  **Additional Security Consideration: Confused Deputy Attacks**
- **Issue**: AWS services acting on your behalf could be misused if not properly scoped.    
- **Solution**:    
    - Use conditions like `SourceAccount` and `SourceArn` to ensure AWS service access originates from **your account only**.
    - 
## Data Catalogue  
- You can trigger **AWS Glue Crawler** whenever new objects land in **S3** using the following approach:  
  **S3 EVENT → SQS → Glue CRAWLER**  
### Schema Management
 #### **Why Schema Management Matters**
- Data structures change over time: columns may be added, removed, renamed, or reordered.    
- Glue Catalog must stay in sync with these changes to avoid failures in downstream systems like Athena.    
- The **goal** is to maintain **query stability** and **schema compatibility** to minimize disruption.    
#### **Tools for Managing Schema Changes**
- **Glue Crawler**: Automates catalog updates on a schedule.    
- **Manual Catalog Updates**: Possible alternative to crawlers.    
- However, **query engines like Athena** may still break or return **incorrect data** if schema issues aren't handled properly.    
#### **Data Format Matters (Athena Behavior)**
- Athena accesses data **by index** or **by column name**, depending on the format:    
    - **CSV/TXT**: Access by **column index** (0-based).        
    - **ORC**: Access by **index (default)**, but supports name-based too.        
    - **Parquet**: Default is **name-based**, but index-based is optional.
        
#### **Schema Issues with CSV Files**
Using the **Iris dataset** example, Athena queries can silently fail due to:
1. **Column Order Changes**:    
    - Column values may map incorrectly if the order is shuffled.        
    - Example: Column 0 and 3 are swapped → incorrect attribute values.        
2. **Missing Columns**:    
    - Missing columns shift all values left → all mappings are wrong.        
3. **New Columns in the Middle**:    
    - Breaks index mapping → Athena reads the wrong column values.        
4. **New Columns at the End** :    
    - Safe: Existing queries work correctly, new columns can be ignored.    
5. **Renaming Columns in the Catalog**:    
    -  Allowed when using index-based formats like CSV—data remains accessible as mapping is based on position.      
##### **Best Practices for CSV-Based Data**
- Maintain consistent **column order**.    
- Add new columns **only at the end**.    
- Avoid missing columns.    
- Test any **data type changes**.    
- You **can rename** catalog column names (as index-based access allows this).    
- If you **can’t enforce these constraints**, consider **post-processing or using a better format** (like Parquet/ORC).

#### **Parquet File Handling in Athena**

#####  **Default Access: By Column Name**
- Athena reads **Parquet files by column name** (not by index, like in CSV).    
- This makes schema evolution **much easier to manage**.   

##### **Schema Change Scenarios and How Athena Handles Them**
1. **Shuffled Column Order**  
     No problem — Athena accesses data by name, so order doesn’t matter.    
2. **Missing Columns in New Files**  
     Athena retrieves the existing columns correctly; missing ones are ignored.    
3. **New Columns Added (Anywhere)**  
     Handled gracefully — Athena reads known columns; new ones don’t break queries.    
4. **Renamed Columns in Glue Catalog**  
     Problem — If the catalog name changes (e.g., adding units), but the file still uses old names, Athena **can’t find** the column and returns **empty values**.   
##### **Cautions & Considerations**
- **Don't rename** catalog columns unless the file schema is also updated.    
- **Data type changes** still require **testing** for compatibility.    
- You can **enable index-based access** in table config, but it's **not the default**.


### Organizing Data in S3 for Glue and Athena
#### **1. File Structure Matters for Querying**
- **Glue Crawler** can handle complex folder structures and detect different schemas.    
- **Athena expects similar schema files** in the same S3 path — **don’t mix unrelated data (e.g., employee and movie JSONs) in one folder**.   
#####  **Best Practice: Organize Data by Schema**
- Example:    
    - `s3://bucket/sales/region/` → one schema, one table (partitioned by region)        
    - `s3://bucket/hr/employee/` and `s3://bucket/hr/training/` → separate folders and separate tables        
- **Configure crawlers with specific S3 paths**, not the whole bucket.
#### **2. Data Classification and Security**
- Use **data classification** (Tier 1 to Tier 3) to guide storage and access:    
    - Tier 1: Internal use (employee info)        
    - Tier 2: Business-sensitive (sales data)        
    - Tier 3: Highly confidential (trade secrets)        
- **Do not store data of different sensitivity levels in the same folder**.    
- Use separate **buckets or top-level folders** for each classification.    
- Follow **least privilege** access control.  
####  **3. Partitioning for Performance**
- Improves **query speed** and **reduces costs** in Athena.    
- Avoids **throughput errors** when querying large datasets.
 ##### Common Partition Strategies:
- **By region**: e.g., `/sales/region=US/`    
- **By time**: e.g., `/sales/year=2024/month=01/`    
- Use **Hive-style folder names (key=value)** for compatibility.
#### **4. Maintaining Partitions in Glue Catalog**
- Athena queries only recognize partitions defined in the **Glue Catalog**.
- #### Ways to Update Partitions:
1. **Glue Crawler**    
    - Can be scheduled (e.g., daily)        
    - Handles both schema and partition changes        
    - **Slower**, scans entire structure        
2. **Athena MSCK REPAIR TABLE**    
    - Fast, works only with **Hive-style** folders        
    - Doesn’t work with regular folder names        
    - Can be triggered by **Lambda**        
3. **Manual Add/Drop Partition Commands**    
    - Fastest method        
    - Requires tracking partition changes manually        
    - Can also be automated with Lambda 
##### Schema Drift Prevention
- **Crawler can enforce a single schema at table level** to prevent drift between table and partitions.
## Data Security and Governance  

### **Data Protection**  
- **Data at Rest** – Ensure encryption is applied.  
- **Data in Transit** – AWS services use **TLS encryption by default**.  

---

### **Regulatory Compliance (GDPR, CCPA, and COPPA)**  

To comply with regulations, organizations must support the **Right to Forget**—the ability to **delete Personally Identifiable Information (PII)** upon request within a specified period.  

#### **Challenges in Deleting PII from Data Lakes**  
4. **Identifying User Records:** Requires scanning all data partitions to locate the records containing the user ID.  
5. **Parquet File Limitations:**  
   - Cannot delete a **single record** from a **Parquet file**.  
   - Requires **re-writing the entire partition**, which is **time-consuming**.  

#### **Solutions to Overcome These Challenges**  

6. **Avoid Storing PII in the Data Lake** (Recommended)  
   - If PII is **not required** for analytics, do not store it.  
   - If PII **must be stored**, use **data masking, hashing, blurring, or random modifications** to **IMPERSONATE** the data.  
   - **Collibra** supports such data protection techniques.  

7. **Implement an Additional Metadata Layer**  
   - Reduces **the number of operations** and **scanning volume** required to find and delete a specific user ID.  
---
# Questions to Consider  

### **1. Do you need real-time insights or model updates?**  
- Determine if your application requires **real-time processing** or if batch processing suffices.  

### **2. What is the staleness tolerance of your application?**  
- Define how **fresh** the data needs to be for your use case.  

### **3. What are the cost constraints?**  
- Assess the budgetary limitations for **storage, compute, and data processing**.  

### **4. Are we ingesting any sensitive data?**  
- If yes, consider **data security measures** such as:  
  - **Tokenization**: Replace character **X** with character **Y**.  
  - **Masking**: Hide sensitive information in **DEV and higher environments**.  

#### **Masking & Tokenization Approach (for MRL)**  
- **Tokenization** is applied to **selected fields**.  
- This is enforced across **all layers** in the **DEV environment**.

---
# Medallion Architecture  

## **Data Platform Strategy**  
- **How is the data platform used?**  
  - **Centralized and shared data platform**: A single, unified platform for all data needs.  
  - **Federated multi-platform structure**: Multiple platforms serving different domains.  

- **How do you align the data platform?**  
  - **Source-system aligned platform**: Easier to standardize in terms of **layering and structure**.  
  - **Consumer-aligned platform**: More **diverse data usage** characteristics on the consumption side.  

## **Landing Zone**  
- Required when extracting data from a source is **difficult or inconsistent**.  
- Useful when working with **external customers or SaaS vendors**.  
- Helps manage **data dependencies** and supports **various file formats**.  
- Facilitates **data refilling and replay** to ensure **data integrity and consistency**.  


# **Data Lake - Design Pattern**  
[Medium Article](https://medium.com/@lackshub/design-patterns-for-data-lakes-d6da14a0af1f)  

## **Landing Zone**  
- Required when it is difficult to extract data from the source (e.g., working with external customers or SaaS vendors).  
- Helps in **data refilling and replay** to ensure consistency.  

## **Raw Zone**  
- Sensitive data can be **masked or tokenized** during ingestion based on regulations.  
- Supports **JDBC sources** in ORC, PARQUET, or AVRO formats.  
- Access is **restricted to a few authorized users**.  

## **Structured Zone**  
- **Additional columns** for auditing and lineage added during transformation in the **RAW layer**.  

## **Trusted Zone**  
- Data is **ready for general consumption** and can be queried via **Apache Hive**.  
- **Reporting tools** can connect directly to the **Trusted Layer**.  
- Uses **SCD Type 1** (no historical data retention).  


# **MRL: Architecture**  

### **Assumptions (Data Lake)**  
- **No SCD Type 2**, meaning **historical changes are not retained**.  
- Supports **data up to the confidential level**.  
- Classified as **Business Essential** (no major risks from unavailability).  
- Captures only **additions/updates** from the source (no deletion tracking).  
- Deleted records remain in the Data Lake (**soft deletes at the source**).  
- Records inserted and deleted within the refresh cycle are **not available**.  

### **Layers - Work, Base, Curated (Data Ingestion)**  
#### **System Folder**  
- **Staging folder**: Stores JAR/code uploaded by the **DI framework** (Data Lifecycle enabled - 30 days expiration).  
- **Stores EMR logs**.  

#### **Raw (Work/Base Layer)**  
- All files in the **WORK layer are archived**.  
- **Data cleansing**: Removing special characters from columns.  

#### **Trusted (Curated Layer)**  
- No **Data Quality (DQ) checks** when moving data.  
- **DQ is needed** for ML applications or **core business logic** derived from the Data Lake.  
- **No SCD specification**, meaning only the **latest version is available**.  



# **Ingestion Strategy**  
- **FULL LOAD**.  
- For **Incremental Load**, use **QUERY-based CDC**.  



# **Data Product Layer**  
- **Encapsulation & Governance** applied via **Redshift groups/accounts**.  

### **Data Processing (ETL)**  
- **Spark & EMR** used for data processing.  

### **Data Transformation & Aggregation**  
- Happens at the **Redshift Layer**.  
- Data moved from **Data Lake → Data Warehouse** using **Redshift Spectrum**.  
- **ELT processes** use **Data Vault, Dimensional (Kimball), or both approaches**.  
- **Data profiling** depends on time windows for engineering processes.  
- **Logging** tracks business unit, process, source, target, row counts, etc. for **governance**.  

### **Data Consumption**  
- **Power BI/Tableau** connect to:  
  - **Redshift internal tables/staging layer** OR  
  - **Data Product Layer (S3 bucket)**.  



# **Data Publication**  
8. **DPL Layer (S3)** sends **incremental data to Greenphire**.  
   - **Push Mechanism**.  
   - **Custom Lambda code** uses **Greenphire API** for **payment settlement** (logistics, payments, etc.).  
   - **Only incremental data** sent to Greenphire (due to **payload limitations**).  



# **Data Lake Management**  
- **Ensuring data quality & consistency** for business decisions.  
- **Policies & regulations** for data ingestion, transformation, and consumption.  
- **Security, privacy, and compliance**which ties to how data is laid out as well as authentication and authorization of users.
- **Data lifecycle management** (archiving aged data).  



# **Data Governance**  
- **Proposed Solution**: **Collibra**.  
- **Glue Table** used for Data Catalog.  
  - **Two Glue Catalogs**:  
    1. **Analytical Account**.  
    2. **DataHub Account**.  



# **Data Provision**  
- **AIM team** provisions data to **XXX** (Clinical Trials Vendor) as **XML/CSV files**.  
- **SNS (Simple Notification Service)** used for delivery notifications.  



# **Data Scrambling vs. Data Masking**  
- **Data Masking** in **DEV & SIT** as **PROD data is used** in DEV and SIT.  
- **No masking needed** if PROD data is not used in lower environments.  
- **Highly sensitive data** (e.g., blood reports) **not stored in Data Lake**.  
- For **DMW/CDR**, data is **masked at the source** via a **triggered Stored Procedure**.  



# **MRL Infrastructure**  
### **Data Warehouse (DWH)**  
- **DEV, SIT, and PROD** environments exist in **separate AWS accounts** (Analytical).  
- **Redshift uses RA3 machines**.  

### **Data Hub (DH)**  
- Same **multi-account structure** as Analytical.  
- **EMR workload is memory and network-bound**.  

---
# **Data Masking Techniques**  

## **1. Encryption**  
- **Process**: Encrypts the data and allows decryption using a **key**.  
- **Use Case**: Securely stores and transmits sensitive information.  
- **Example**: AES (Advanced Encryption Standard) encryption for storing passwords or PII data.  

## **2. Scrambling**  
- **Process**: Jumbles characters and numbers into a **random order** to hide the original content.  
- **Use Case**: Used for **basic obfuscation**, but **not highly secure**.  
- **Limitations**: Only applicable to certain types of data.  

## **3. Nulling Out**  
- **Process**: Replaces data with **NULL values** so unauthorized users cannot access it.  
- **Use Case**: Used when certain users **should not see** the sensitive data at all.  
- **Example**: Masking salary data in HR reports for unauthorized employees.  

## **4. Substitution**  
- **Process**: Replaces sensitive data with **realistic but fake** values.  
- **Use Case**: Used when masked data needs to **retain realistic properties**.  
- **Example**: Replacing real customer names with randomly generated names in test environments.  

## **5. Date Aging**  
- **Process**: Alters **dates** while maintaining relative timeframes.  
- **Use Case**: Ensures **time-sensitive trends** are preserved while masking actual dates.  
- **Example**: Shifting transaction dates by a random number of days to protect user data.  
---
# **Application Design Considerations**  

## **Difference Between YAML and JSON as Configuration Files**  

### **YAML**   
✅ Supports **multi-line statements**, making it ideal for SQL queries in configurations.  
✅ Allows **comments**, making configurations easier to understand and maintain.  
✅ More **human-readable** with indentation-based formatting.  
🛑 Slightly more complex parsing than JSON.  

### **JSON**  
✅ Lightweight and widely used for **machine-to-machine** communication.  
✅ Strict structure with **key-value pairs**, making it easy to parse.  
🛑 **No multi-line support**, making it difficult to store long SQL statements.  
🛑 **No comments**, reducing readability and maintainability.  

## **When to Use CLI Over API**  
- **Use CLI when API changes frequently**: APIs might get updated often, but CLIs tend to remain stable. 
---
# **Designing Incremental Ingestion for File-Based Push Strategy (Hourly File Flow)**  

## **Key Considerations**  

### **How Do We Identify New Files?**  
9. **M1: Maintain a Control Table (RDBMS)**
   - Keeps track of processed files.
   - **Issue:** As the number of files increases, scanning and maintaining the table becomes cumbersome.  

10. **M2: Clean the INBOUND Location After Successful Processing**
   - Only retains unprocessed files.
   - **Issue:** No automatic replay mechanism. To replay, we need to restore all files manually.  

### **How to Handle Schema Evolution and Schema Drift?**  
- **Need to find**

### **How Easy Is It to Switch from Batch to Streaming?**  
- The architecture should support **both batch and streaming** to allow easy migration if needed.  

---

## **Approaches for Incremental Ingestion**  

### **M1: Polling-Based Approach**  
- **Constantly poll** the INBOUND location to look for new files.  
- Store metadata of processed files in a **metadata store**.  
- **Pros:** Works well for scheduled ingestion.  
- **Cons:** Can be inefficient due to continuous polling.  

### **M2: Spark Structured Streaming**  
- Use **Spark Structured Streaming** to read files incrementally.  
- **Cons:** It scans all objects in the INBOUND location, which may be **resource-intensive**.  

### **M3: Event-Driven Approach**  
- **Lambda triggers ingestion** when a new file lands.  
- The file is placed in **SQS**, and **another Lambda** triggers the **Databricks job**.  
- **Pros:** Efficient and event-driven, reducing the need for polling.  
- **Cons:** Requires calling **Databricks API** every time a job is triggered, adding **operational overhead**.  

---
# Medallion Architecture

## Bronze:

### Do we need an exact copy of the source?
- **A1:** Do we have a plan to use data in **BRONZE** as the **Source of Truth** in the future, so downstream applications use this data?  
- **A2:** Sometimes, ingestion (**COPY ACTIVITY**) has issues with **NUMERIC** data types (e.g., **Oracle**).  
  - To handle this, we choose to **convert** those columns to **STRING**.  
  - **Problem:** If **BRONZE** data is used as **Source of Truth**, then this conversion would not work.  
---
# Ingestion Decision

## Incremental Ingestion Strategy:
- **Large Volume of Data**
- **When Source Does Not Support Large Volume of Data Ingestion**  
  - Example: **API sources like VEEVA** do not support extracting large volumes of data.

---
# Infrastructure Rules
[Databricks Cluster Configuration Best Practices](https://docs.databricks.com/clusters/cluster-config-best-practices.html#cluster-mode)

## Typical Use Cases:

### **Data Analyst**
- Shared cluster for multiple users to execute **READ-ONLY** queries.
- **Enable Auto Termination** (Cluster will terminate when idle).

### **Machine Learning**
- [Details not provided]

### **Batch Ingestion ETL**
- **No need for caching**, as data is not being reprocessed.
- Use a **combination of On-Demand instances + Spot Instances** for cost optimization.
- **Enable Autoscaling** if the data volume is high.
- Choose a **Compute-Intensive Cluster** for better performance.

# AWS Big Data White Paper
[Big Data Analytics Options on AWS](http://d0.awsstatic.com/whitepapers/Big_Data_Analytics_Options_on_AWS.pdf)

## AWS Kinesis
### **Pricing**
- Based on the **number of shards**
- Charges for each **1 million PUT transactions**

### **Design Pointer**
- Use **DynamoDB** to store a cursor to track what has been read from a Kinesis stream.
  - If an application fails while reading data, it can restart using the stored cursor.

### **Anti-Patterns**
11. **Small Scale Consistent Throughput**  
   - Kinesis Data Streams is optimized for large data throughputs, not for streaming at **200 KB/sec or less**.
12. **Long-Term Data Storage and Analytics**  
   - By default, **data is retained for 24 hours**, extendable up to **7 days**.
   - Move long-term data to **Amazon S3, Glacier, Redshift, or DynamoDB**.



## AWS Lambda
### **Usage Patterns**
- **Real-time File Processing** (e.g., Process files uploaded in S3)
- **Real-time Stream Processing**
- **Replacing Cron Jobs** with scheduled Lambda functions.
- **Processing AWS Events** (e.g., AWS CloudTrail logs)

### **Pricing**
- Based on `# of requests * duration of execution`

### **Durability and Availability**
- **Synchronous invocations** respond with an exception if they fail.
- **Asynchronous invocations** are retried at least **3 times** before rejection.

### **Scalability and Elasticity**
- Default **soft limit of 1,000 concurrent executions per account per region**.

### **Anti-Patterns**
13. **Long-Running Applications**  
   - Lambda functions **must complete within 900 seconds**.  
   - For long-running tasks, use **EC2 or a chain of Lambda functions**.
14. **Dynamic Websites**
15. **Stateful Applications**  
   - Lambda is stateless. Persistent data should be stored in **S3 or DynamoDB**.



## AWS EMR
### **Anti-Patterns**
- Not suited for **small files**.
- Not designed for **ACID transactions**.



## AWS Glue
### **Cost Model**
- Hourly billing for **crawler jobs** and **ETL jobs** (billed by the minute).
- Monthly fees for **Glue Data Catalog**.

### **Durability and Availability**
- AWS Glue **pushes job statuses** to **CloudWatch**.
- SNS notifications can be set up for job failures/completions.

### **Interfaces**
- **Schedule the crawler**
- **Import metadata from Hive Metastore** into AWS Glue Data Catalog.

### **Anti-Patterns**
16. **Streaming Data**  
   - Glue is not suited for streaming. Use **Kinesis for ingestion**, then process with Glue.
17. **NoSQL Databases**  
   - AWS Glue **does not support NoSQL databases or Amazon DynamoDB**.



## Amazon Machine Learning
### **Anti-Patterns**
- Not suited for **large datasets**.  
  - **Amazon ML supports up to 100 GB** but does not support terabyte-scale ingestion.  
  - **Amazon EMR with Spark MLlib** is a better alternative.



## Amazon DynamoDB
### **Ideal Usage Patterns**
- **Metadata storage for Amazon S3 objects**
- **Log ingestion**
- **Highly available, scalable database** for business-critical applications.

### **Cost Model**
- **Provisioned throughput capacity**
- **Indexed data storage**
- **Data transfer (in/out)**

### **Anti-Patterns**
18. **Prewritten applications tied to relational databases**  
   - Use **RDS or EC2** with an installed RDBMS instead.
19. **Joins or Complex Transactions**
20. **Binary Large Objects (BLOB) Data**  
   - Store large files **(400 KB+) in S3** instead.
21. **Large data with low I/O rate**  
   - DynamoDB **uses SSDs**, optimized for high I/O workloads.  
   - For large but infrequently accessed data, **Amazon S3 is a better choice**.



## Amazon Redshift
### **Cost Model**
- **No extra charge** for backup storage **up to 100% of provisioned storage**.

### **Anti-Patterns**
22. **Small Datasets**  
   - Redshift is optimized for **parallel processing** across clusters.
   - If data is **< 100GB**, **RDS is a better option**.
23. **OLTP (Online Transaction Processing)**
24. **Unstructured Data**  
   - Redshift **requires a defined schema**.
   - **ETL with Amazon EMR** is recommended for structuring data before ingestion.
25. **BLOB Data**  
   - Store large binary files in **S3** and reference them in Redshift.



## Amazon Athena
### **Cost Model**
- **$5 per TB of data scanned**

### **Ideal Usage Patterns**
- **Ad-hoc querying for web logs** (e.g., troubleshooting performance issues).
- **Querying staging data before loading into Redshift**.
- **Notebook-based analytical solutions** (e.g., **RStudio, Jupyter, Zeppelin**).

### **Anti-Patterns**
26. **Enterprise BI & Reporting**  
   - **Redshift is better** for large-scale **business intelligence workloads**.
27. **ETL Workloads**
28. **RDBMS Use Cases**  
   - Athena is **not a transactional database**.



## Solving Big Data Problems on AWS
### **Key Considerations**
- **How quickly do you need results?** (Real-time, seconds, hours?)
- **What is the budget for analytics?**
- **How large is the dataset and its growth rate?**
- **How is the data structured?**
- **What integration capabilities exist for producers and consumers?**
- **What is the acceptable latency?**
- **What is the cost of downtime?**
- **Is the workload consistent or elastic?**

---

# Use Case: AWS Glue

## **Problem 1**  
### **Reference:**  
[YouTube Video](https://www.youtube.com/watch?v=S_xeHvP7uMo&list=PLqR0Vb5cBGe8aA5gj7MpR6ABJNeh2SRXb&index=3&t=4s)  
**Timestamp:** 1:01  

### **Use Case: Convert CSV/JSON to Parquet for Querying Layer**

#### **Architecture 1:**
29. **Data Pipeline** → **ECS** (Generates a unique application ID for each file)
30. **ECS Updates the State Table**:
   - Performs **data quality checks**.
   - **Unnests fields**.
   - Stores **last successfully processed partition information**.
31. **ECS Execution:**
   - ECS contains a script that **triggers AWS Glue**.
   - Before triggering, a **soft lock** is acquired.
   - After processing, the **lock is released**.

#### **Architecture 2:**
- Replace **ECS** with **Python Shell in AWS Glue**.



## **Problem 2: Build a User Profile**

### **Architecture**
32. **Users Click Stream → Amazon Athena (CTAS) → Consumer Analytics Profile**.
33. **Data Flow**:
   - Click Stream → **10-minute intervals** → **S3** → **DynamoDB (Update User Profile)**.

---

# **Spark-Based Ingestion**

## **Source:**
### **JDBC - DB2**
- If a table does **not** have a **primary key** or **unique key**, DB2 provides an internal key called **RRN** to uniquely identify a row.
## **Audit Tables**
- If the source has AUDIT table, used by **data warehouses** to track **deletions**.
- Example: `SOMESYSTEMNAME_DELETE_LOGS_S`.
- **Recommendation**: **Do not ingest these tables** into the **Data Lake**.
- **Reason**:
  - No significant **business value**.
  - **Dashboards are not built** on them.



## **Key Questions to Ask in Mxxx**
### **Data Ingestion Priorities**
34.  Some it is Data Quality
35.  Security
- **Sanity Checks**:
  - Number of records received.
  - Data type validation.
- Any masking performed in the ingestion layer.
- History capture.
  - **Schema Changes**
- **Max Connections for JDBC** (Check source limitations or parallel connection limits).
- **Parallelism for JDBC & REST API**:
  - For JDBC sources, verify if **separate environments (DEV, PROD, SIT)** exist.  
    - If not, environments **cannot run simultaneously**.
  - Identify source limitations such as **maximum parallel connections**.

 - Handling Dependencies Within a Job
- **Checkpointing Mechanism**
- **Failure Handling / Retry / Backoff**
- **Monitoring**
- **Schema Changes**:
   - New column added.
   - Data type change.
   - Column deleted.
- **Inconsistent Reads While Writes in Progress**
- **Backfill / Re-ingestion**
- **Handling Empty Tables**:
  - Should the data be standardized (e.g., standardizing date formats)?



## **Data Ingestion Framework**
**Reference:** [Databricks Session on Spark-Based Reliable Data Ingestion](https://databricks.com/session/spark-based-reliable-data-ingestion-in-datalake)

### **Metadata Storage**
Stores the following information:
36. **Source details & connection parameters**
37. **State Checkpointing**
38. **Max records / Processing time**
39. **Registered transformations**

### **Spark Ingestion Framework**
#### **Source Layer**
- Connects to the **data source** using **metadata from the last checkpoint**.

#### **Processing Layer**
- **Transformations**:
  - **Masking PII data**
- **Schema Evolution**:
  - How to Identifying schema changes:
    -Compare  Latest run schema vs. Today's schema.
- **Compaction / Deduplication**

#### **Sink Layer**
- Ensuring **read-time consistency** while updating the **sink**.
- **Explanation**:
  >If you are writing to directory, when people are
					reading the  directory,there is a chance of inconsistency.
					To avoid this, one solution is lock;**Drawback** to  this approach is
							1)this affect READS while writing
							2)Reduce the parallelism 

- **Solutions**:
  - **HDFS Approach**:
    1. Take an **HDFS snapshot** and create a **Hive view** on the snapshot.
    2. Write directly to the directory.
  - **S3 Approach**:
    - Use **prefix-based partitioning (Epoch/Date Partitioning)**.

#### **Sanity Checks**
- If using **Parquet**, then it is easy to get:
  - **Missing records**
  - **Count differences**
  - **Duplicates**

---
## **Datawarehouse**
### Data Mart
- Data Mart is a subset of DWH.
- It is typically built to **serve a specific use case** — a department, region, or a particular tool.
-  **Cubes** it is used for analytical purposes . Use case is when you have lot of hierarchy 
### ODS
- ODS integrate multiple system in an  organization into single database. This data is used for Operational decision-making. It only maintain the current state of the data.
- Common pattern in Data Architecture when ODS used is , it serves as a **staging layer** for the Data Warehouse.
- Example, In a financial services company, to evaluate if a customer qualifies for a credit, data from multiple systems (crypto, stocks, ETFs) is combined **in real-time** to get a current view of their balance. This **quick, operational** decision requires an ODS—not a warehouse.
- Implementation ODS  [Building an Operational Data Store with Kafka and Snowflake | by Vladimir Pasman | Medium](https://medium.com/@vlad-pasman/building-an-operational-data-store-with-kafka-and-snowflake-fac1d7361c81)
## **Design Considerations**
### **Use Case: Ingesting Oracle Tables to S3**
40. **Handling Numeric Data Types**:
   - Example: `NUMERIC(6,2)` in **Oracle** can store `9999.99, -9999.99, 1000, 1`,  
     but in **Spark**, it might display `1.00` when read directly.
   - **Solution 1**: Store as **STRING**.
   - **Solution 2**: Handle in the **visualization layer**.

	### **Ingestion Folder Structure**
	- **Recommended S3 Naming Convention**:
	source_system/raw/table/data --if we follow this naming, in case we need to build Glue Catalog, it is easy  
	source_system/raw/table/keys  
	source_system/raw/table/schema  
	
	- **Earlier S3 naming convention:**  
	source_system/raw/table/<<<data files>>>  
	source_system/raw/table/keys  
	
	  - **Note:** Here we cannot build Glue data catalog on `source_system/raw/table/` as there are extra folders under `table` like `keys`.
	
	
	## **Handling Missing Data in Curated Layer**
	### **Scenario: Base Layer Has No Data**
	- **Option 1**: Copy **previous run data from Curated layer ** to **Current Curated Layer**. This is the case when the GLUE Catalog refers the latest partition for all the FULL snapshot of the data.
	  - **Con**: Data duplication.
	- **Option 2**: Checking  for **Base Layer data in the Processing Layer(Spark)** (Not Orchestration Layer).**Ex:** Airlfow using Python Operator - if file exist - Task 1 dag else Trigger another task 
	  - **Pro**:
	    1. Above would introduce multiple tasks. If **failure occurs**, only **one task** in an **Airflow DAG** needs restarting.
	    2. **Parallelism** can be achieved using **Spark** (vs. an **Airflow operator**).
	
	
	
	## **Deployment Considerations**
	- **Configurations** stored in **Bitbucket**.
	- **Data** resides in **S3**.
	- Use **Service User** for deployment instead of individual users.
	  - **Service User** has **specific project permissions**.
	
	
	
	## **Infrastructure**
	- **CloudFormation Templates** are **preferred** for **spawning EMR/Infra**.
	- **Alternative**: Boto3 (not recommended since **CFTs are easier to maintain**).
	
	
	
	## **Source Profiling**
	- **Classify Tables by Record Volume**:
	  - **Small**, **Medium**, **Large**.
	- **Tables Without a Primary Key**:
	  - Perform a **full load** (if data volume is manageable).
	
	---
	
	## **Issues Faced**
	- **Oracle to Redshift Data Ingestion**:
	  - **BLOB data type** can cause **memory issues**.
	  - **Solution**:
	    - If **Mxxk** does not use the column **downstream**, it **can be deleted**.

---
### **Use Case:  2 ETL jobs updating same table **
      #Architecture/Orchestration 
     [Use Fabric Notebook code based orchestration tool to avoid concurrent write conflicts. – Small Data And self service](https://datamonkeysite.com/2024/01/27/use-fabric-notebook-code-based-orchestration-tool-to-avoid-concurrency-write-conflicts/)
	  
	> I had a simple data ingestion use case, Notebook A inserts data to a Delta Table every 5 minutes and Notebook B backfills the same table with new fields but only at 4 am. Initially I just scheduled Notebook A to run every 5 minutes and Notebook B to run at 4 AM , did not work as I got a write conflict, basically Notebook B take longer time to process the data, when it is ready to update the table, it is a bit too late as it was already modified by Notebook A and you get this error.
	- Solution 1: Schedule Notebook A every 5 minutes ,except 4 AM - 4:15 AM. This is not available in Fabric pipeline, but is available in Data Factory.
	- Solution 2: Partition it based on time, for not so large volume of  data this approach can create multiple small files.
	- Solution 3: Check if a refill file is available in the Notebook A and use `notebook.run()` utility  to  run the Notebook B from Notebook A.
	---