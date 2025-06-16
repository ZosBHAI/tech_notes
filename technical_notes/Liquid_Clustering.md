

title: "Liquid Clustering: An Innovative Approach to Data Layout in Delta Lake"
source: "https://medium.com/@stevejvoros/liquid-clustering-an-innovative-approach-to-data-layout-in-delta-lake-1a277f57af99"
"[Delta Lake Liquid Clustering vs Partitioning - That Fabric Guy](https://thatfabricguy.com/delta-lake-liquid-clustering-partitioning/)"
tags:
  - "[[delta_table_concepts]]"
---
## What is Liquid Clustering in Delta Lake?

Liquid Clustering is a  optimization technique aimed at streamlining data layout in Delta Lake tables. Liquid Clustering logically clusters your data by values in one or more columns — all without changing the table’s physical layout.
- This feature is available in Delta Lake 3.1.0 and above

## How Liquid Clustering works?
Under the hood, Liquid Clustering organizes your data into what’s effectively a logical sort order during writes. It groups together rows with similar values in the clustering columns. These grouped rows end up stored in the same files or file ranges. Then, when you run queries with filters on those clustered columns, Spark can quickly skip over unrelated data by consulting file-level statistics (like min and max values for each file).

## LC Vs Hive-style Partitioning Vs  Z-ordering

#### Example Dataset

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*ftwCGwGhQU7eIGkGzBnwUA.png)

#### Hive-style Partitioning
Hive-style partitioning improves query performance on large datasets by organizing data into directory-based partitions.
```c
/transactions/month=2023-1/country=Germany/{1.parquet, 2.parquet,...}
/transactions/month=2023-2/country=USA/{30.parquet, 31.parquet,...}
/transactions/month=2023-3/country=Mexico/{53.parquet, 54.parquet,...}
...
```
This structure enables **data skipping**, reducing scan time.
However, it has **limitations**:
- It's **rigid and static**, making partition management and updates challenging.    
- It can cause **data skew**, with some partitions oversized and others nearly empty.    
- It's only effective on **low-cardinality columns** (e.g., `month`, `country`); high-cardinality fields lead to many small files, degrading performance.   
**Common issues are :**
1. Partitions with little or no data.    
2. Oversized partitions needing further breakdown.    
3. Multiple small partitions needing file merges.    
4. Frequent data ingestion creating small, fragmented files—requiring scheduled optimization jobs.
![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*u1w9IsT3qmZqnk9NghQ-fA.png)


### Z-Ordering 
**Z-ordering** is a **data-layout optimization** that clusters data within (or across) files to improve **query performance**, especially on **high-cardinality columns** like `senderId`. Unlike partitioning, Z-ordering does **not require** partitions but is often used **in combination** with them to enhance **data skipping**.
Key concepts:
- **Z-cube**: A group of clustered files. Once it reaches a size threshold, it becomes **stable** and excluded from future clustering unless modified.    
- **Unstable Z-cubes** are re-clustered with new data until they stabilize.   

**Implications & Limitations:**
1. Z-ordering is **not idempotent**; clustering results vary across runs.    
2. It clusters **within partitions only**, not across them.    
3. **Stable Z-cubes** may need to be reopened after DML operations.    
4. Changes in **query patterns** may require re-Z-ordering, which is **compute-intensive**.    
```c
-- Example of Z-ordering in targeted partitions
OPTIMIZE transactionsTable
WHERE month >= to_date('2023-02-01')
ZORDER BY (senderId)
```
![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*mhKMq2fDDpqHQjxenzEZBQ.png)
**Conclusion**: Z-ordering complements partitioning but both are **static layout techniques**.
### LC(Liquid Clustering)
**Liquid clustering** is a **flexible, adaptive** data layout technique that overcomes the limitations of **Hive partitioning** and **Z-ordering** by:
- **Removing the need for static partitions**   and  can dynamically merge or further divide files in order to arrive at a balanced dataset with the ideal number and size of files.
- **Allowing clustering keys to be based purely on query patterns** without concern for cardinality, file size, or skew    
- **Enabling clustering key changes** without full table rewrites.   

Key features:
- Supports **stateful, incremental clustering**, only processing newly ingested data .Due to the incremental nature of clustering, table maintenance is computationally cheap and runs in a short time. Downstream processes and users can continue accessing the table simultaneously without an impact due to Delta Lake’s ACID transactions.  
- Allows **cluster-on-write (also called eager clustering)** becomes possible following this approach, with clustering performed as part of the ingestion.    
- Improves **data skipping, skew correction**, and **concurrency** at the record level
![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*Lr4lfzNGILCxrP1XWf7UYw.png)
## How to Use It?

To use liquid clustering, you need to create a Delta Lake table with the `CLUSTER BY` phrase specifying the clustering columns:

```c
CREATE TABLE transactions (
  id BIGINT,
  country STRING,
  month DATE,
  transactionTime TIMESTAMP,
  senderId INT,
  recipientId INT,
  amount DECIMAL,
  note STRING
)
CLUSTER BY (country, month)
```

Ingesting data to the table (for example, using CTAS, INSERT or COPY INTO statements) can automatically trigger clustering as long as the ingestion size does not exceed 512 GB. This is also called eager clustering:

```c
INSERT INTO transactions WHERE country = "Germany" 
SELECT * FROM transactions_staging
```

In order to ensure sufficient clustering across the dataset regardless of **ingestion size**, you can trigger clustering by running `OPTIMIZE` as the following:

```c
OPTIMIZE transactions
```

For tables with frequent updates or inserts, a background `OPTIMIZE` job can be scheduled to optimize data layout on a regular basis. Due to the incremental nature of clustering, table maintenance is computationally cheap and runs in a short time. Downstream processes and users can continue accessing the table simultaneously without an impact due to Delta Lake’s ACID transactions \[[9](https://docs.databricks.com/en/lakehouse/acid.html)\].

When query predicates change over time, you can set new clustering keys on the table as below. Subsequent `OPTIMIZE` will cluster newly ingested data according to the new set of clustering keys.

```c
ALTER TABLE transactions CLUSTER BY (country, month, senderId);
OPTIMIZE transactions;
```

## Can I Use Both Partitioning and Liquid Clustering ?
Absolutely. A hybrid approach is often the sweet spot. For **example**, coarse partitioning by year, combined with Liquid Clustering by product or region, can give you the best of both worlds. Just make sure your query patterns align with how your data is structured and clustered.
## When to Use Liquid Clustering or Partitioning for Delta Lake?

Partitioning of small tables is not efficient because of the overhead when reading from multiple small files. Spark prefers larger Parquet files, so too much partitioning doesn’t work that well.
The same goes for liquid clustering, where file elimination is triggered based on the metadata of those files. If there are just 1-2 files in a delta table, this will not make sense.

| **Table size / row count** | **Optimisation strategy** |
| --- | --- |
| < 1 GB / < 10 million rows | Nothing – keep things simple! |
| 10-100 GB / 10-100 million rows | Liquid Clustering based on columns that are often used in filters |
| \> 100 GB / billion+ rows | Partitioning + Liquid Clustering |

## How To Create Liquid Clustering in Microsoft Fabric Delta Tables

Creating new Delta tables with Liquid Clustering is quite easy! Just refer to the below example. In the second-last line, you see that we call the.clusterBy() method to enable clustering on these columns. It’s as easy as that!

```python
#ClusteringDateTaxiType
DeltaTable.create(spark) \
    .tableName("gold_facttaxidata_ClusteringDateTaxitype") \
    .addColumn("PickupDate", "DATE") \
    .addColumn("PickUpTime", "STRING") \
    .addColumn("DropoffDate", "DATE") \
    .addColumn("DropoffTime", "STRING") \
    .addColumn("PickupLocationId", "INT") \
    .addColumn("DropoffLocationId", "INT") \
    .addColumn("TaxiTypeId", "INT") \
    .addColumn("PassengerCount", "INT") \
    .addColumn("TripDistance", "DOUBLE") \
    .addColumn("FareAmount", "DOUBLE") \
    .addColumn("SurchargeAmount", "DOUBLE") \
    .addColumn("MTATaxAmount", "DOUBLE") \
    .addColumn("TipAmount", "DOUBLE") \
    .addColumn("TollsAmount", "DOUBLE") \
    .addColumn("ExtraAmount", "DOUBLE") \
    .addColumn("EhailFeeAmount", "DOUBLE") \
    .addColumn("AirportFeeAmount", "DOUBLE") \
    .addColumn("CongestionSurchargeAmount", "DOUBLE") \
    .addColumn("ImprovementSurchargeAmount", "DOUBLE") \
    .addColumn("_PickupYear", "INT", generatedAlwaysAs="YEAR(PickupDate)") \
    .addColumn("_PickupMonth", "INT", generatedAlwaysAs="MONTH(PickupDate)") \
    .clusterBy("PickupDate", "TaxiTypeId") \
    .execute()
```
## Maintenance for Liquid Clustering on Delta Tables

Simply running the OPTIMIZE command against your table will trigger the clustering. I usually have a maintenance job that I run once per week, in order to COMPACT, OPTIMIZE and VACUUM my tables.

For tables that are updated very frequently (think multiple times per hour), you might want to experiment with daily maintenance jobs. However, keep in mind that maintenance introduces overhead compute usage.
## Current Limitations & Recommendation

Databricks recommends liquid clustering for all new Delta tables \[[7](https://docs.databricks.com/en/delta/clustering.html)\]. Based on the above, it can be particularly beneficial for the following scenarios:
- Tables are often filtered by columns with different cardinalities
- Tables have a significant skew in data distribution
- Tables grow quickly and require maintenance and tuning effort
- Tables have concurrent write requirements
- Tables have access patterns that change over time
- Tables where a typical partition key could leave the table with too many or too few partitions.

Limitations:

- You can only specify columns with statistics collected for clustering keys. By default, the first 32 columns in a Delta table have statistics collected.
- You can specify up to 4 columns as clustering keys.
- Structured Streaming workloads do not support clustering-on-write.

---
## Challenges  with Liquid clustering 

As data volumes and analytics needs grow, collaboration across teams and understanding query behavior becomes increasingly complex—even with Liquid Clustering in place. Data teams need to figure out:
- Which tables will benefit from Liquid Clustering?
- What are the best clustering columns for this table?
- What if my query patterns change as business needs evolve?
Moreover, within an organization, data engineers often have to work with multiple downstream consumers to find answers to above queries. This challenge becomes exponentially more complex as your data volume scales with more analytics needs.
## Automatic Liquid Clustering 

With Automatic Liquid Clustering, Databricks **takes care of all data layout-related decisions for you** – from table creation, to clustering your data and evolving your data layout – enabling you to focus on extracting insights from your data.
- ``Databricks Runtime 15.4+
#### How to enable it ?
- Enabled Predictive Optimization , you can do so by selecting Enabled next to Predictive Optimization in the account console under Settings > Feature enablement.

![Cost-benefit Optimization](https://www.databricks.com/sites/default/files/inline-images/announcing-automatic-liquid-clustering-blog-img-7.png?v=1741166280)

- Then configure your **UC managed unpartitioned or Liquid tables** by setting the parameter `CLUSTER BY AUTO`.
```
-- Creating a new table
CREATE TABLE tbl_name CLUSTER BY AUTO;
-- Applying on an existing table
ALTER TABLE tbl_name CLUSTER BY AUTO;
```


Once enabled, Predictive Optimization analyzes how your tables are queried and **intelligently selects the most effective clustering keys** based on your workload. It then clusters the table automatically, ensuring data is organized for optimal query performance. Any engine reading from the Delta table benefits from these enhancements, leading to **significantly faster queries.** Additionally, as query patterns change, Predictive Optimization dynamically adjusts the clustering scheme, **completely eliminating the need for manual tuning or data layout decisions** when setting up your Delta tables.
## Behind the Scenes of Automatic Liquid Clustering: How It Works
Once enabled, Automatic Liquid Clustering continuously performs the following three steps:
1. Collecting **telemetry**(like **collects and analyzes query scan statistics**, such as query predicates and JOIN filters) to determine if the table will benefit from introducing or evolving Liquid Clustering Keys.
2. **Modeling the workload** to understand and identify eligible columns. This is done by  learning from past query patterns and estimates the potential performance gains of different clustering schemes. By simulating past queries, it predicts how effectively each option would **reduce the amount of data scanned.**
3. **Once new clustering key candidates are identified, Predictive Optimization evaluates whether the performance gains outweigh the costs.** If the benefits are significant, it updates the clustering keys on Unity Catalog managed tables

![Predictive Optimization](https://www.databricks.com/sites/default/files/inline-images/predictive-optimization.png?v=1741200723)


