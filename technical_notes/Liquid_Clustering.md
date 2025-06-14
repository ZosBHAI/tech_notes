

title: "Liquid Clustering: An Innovative Approach to Data Layout in Delta Lake"
source: "https://medium.com/@stevejvoros/liquid-clustering-an-innovative-approach-to-data-layout-in-delta-lake-1a277f57af99"
author:
  - "[[Steve Voros]]"
published: 2023-09-21
created: 2025-06-14
description: "Announced at the 2023 Data + AI Summit [1], Delta Lake liquid clustering introduces an innovative optimization technique aimed at streamlining data layout in Delta Lake tables. Its primary goal is to…"
tags:
  - "[[delta_table_concepts]]"
---
## What is Liquid Clustering in Delta Lake?

Liquid Clustering is a  optimization technique aimed at streamlining data layout in Delta Lake tables. Liquid Clustering logically clusters your data by values in one or more columns — all without changing the table’s physical layout.
- This feature is available in Delta Lake 3.1.0 and above
- 
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

## Current Limitations & Recommendation

Databricks recommends liquid clustering for all new Delta tables \[[7](https://docs.databricks.com/en/delta/clustering.html)\]. Based on the above, it can be particularly beneficial for the following scenarios:
- Tables are often filtered by columns with different cardinalities
- Tables have a significant skew in data distribution
- Tables grow quickly and require maintenance and tuning effort
- Tables have concurrent write requirements
- Tables have access patterns that change over time
- Tables where a typical partition key could leave the table with too many or too few partitions.

As of writing this article, Delta liquid clustering is in Public Preview. The following limitations exist \[[7](https://docs.databricks.com/en/delta/clustering.html)\]:

- You can only specify columns with statistics collected for clustering keys. By default, the first 32 columns in a Delta table have statistics collected.
- You can specify up to 4 columns as clustering keys.
- Structured Streaming workloads do not support clustering-on-write.

---
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

## Conclusion

Whether you lean towards partitioning or embrace the flexibility of Liquid Clustering, the key takeaway is this: there’s no silver bullet. Each approach has its strengths and weaknesses. Partitioning can offer fast query performance when done right, but it’s rigid and difficult to maintain. Liquid Clustering, on the other hand, brings adaptability and ease of use, particularly as your data volumes grow and usage patterns change over time..

In most real-world scenarios, a thoughtful combination of the two will give you the best results. Start simple, monitor your query patterns, and don’t be afraid to iterate. As always in data engineering, context is king, so let your use case guide your optimisation strategy.

---
---

#### Summary

- Automatic Liquid Clustering, powered by Predictive Optimization, automates clustering key selection to continuously improve query performance and lowering costs.
- Robust selection processes and continuous monitoring keep tables optimized.
- TCO is minimized by automatically evaluating whether the performance gains outweigh the costs.

We’re excited to announce the Public Preview of Automatic Liquid Clustering, powered by [Predictive Optimization](https://docs.databricks.com/aws/en/optimizations/predictive-optimization). This feature automatically applies and updates Liquid Clustering columns on [Unity Catalog](https://www.databricks.com/product/unity-catalog)  managed tables, **improving query performance and reducing costs.**

Automatic Liquid Clustering simplifies data management by eliminating the need for manual tuning. Previously, data teams had to manually design the specific data layout for each of their tables. Now, Predictive Optimization harnesses the power of Unity Catalog to monitor and analyze your data and query patterns.

To [enable Automatic Liquid Clustering](https://docs.databricks.com/en/delta/clustering.html#automatic-liquid-clustering), configure your UC managed unpartitioned or Liquid tables by setting the parameter `CLUSTER BY AUTO`.
```
-- Creating a new table
CREATE TABLE tbl_name CLUSTER BY AUTO;
-- Applying on an existing table
ALTER TABLE tbl_name CLUSTER BY AUTO;
```


Once enabled, Predictive Optimization analyzes how your tables are queried and **intelligently selects the most effective clustering keys** based on your workload. It then clusters the table automatically, ensuring data is organized for optimal query performance. Any engine reading from the Delta table benefits from these enhancements, leading to **significantly faster queries.** Additionally, as query patterns change, Predictive Optimization dynamically adjusts the clustering scheme, **completely eliminating the need for manual tuning or data layout decisions** when setting up your Delta tables.

During the Private Preview, dozens of customers tested Automatic Liquid Clustering and saw strong results. Many **appreciated its simplicity and performance gains**, with some already using it for their gold tables and planning to expand it across all Delta tables.

Preview customers like Healthrise have reported **significant query performance improvement** with Automatic Liquid Clustering:

> — Li Zou, Principal Data Engineer, Brian Allee, Director, Data Services | Technology & Analytics, Healthrise

![Customer Workload](https://www.databricks.com/sites/default/files/inline-images/announcing-automatic-liquid-clustering-blog-img-1.png?v=1741166280)

Customer Workload

## Choosing the best data layout is a hard problem

**Applying the best data layout to your tables significantly improves query performance and cost efficiency.** Traditionally, with partitioning, customers have found it difficult to design the right partitioning strategy to avoid data skews and concurrency conflicts. To further enhance performance, customers might use ZORDER atop partitioning, but ZORDERing is both expensive and even more complicated to manage.

[Liquid Clustering](https://www.databricks.com/blog/announcing-general-availability-liquid-clustering) significantly simplifies data layout-related decisions and provides the flexibility to redefine clustering keys without data rewrites. Customers only have to **choose clustering keys purely based on query access patterns,** without having to worry about cardinality, key order, file size, potential data skew, concurrency, and future access pattern changes. We've worked with thousands of customers who benefited from better query performance with Liquid Clustering, and we now have **3000+ active monthly customers** writing **200+ PB data** to Liquid-clustered tables per month.

However, even with the advances in Liquid Clustering, you still have to choose the columns to cluster by based on how you query your table. Data teams need to figure out:

- Which tables will benefit from Liquid Clustering?
- What are the best clustering columns for this table?
- What if my query patterns change as business needs evolve?

Moreover, within an organization, data engineers often have to work with multiple downstream consumers to understand how tables are being queried, while also keeping up with changing access patterns and evolving schemas. This challenge becomes exponentially more complex as your data volume scales with more analytics needs.

## How Automatic Liquid Clustering evolves your Data Layout

With Automatic Liquid Clustering, Databricks **takes care of all data layout-related decisions for you** – from table creation, to clustering your data and evolving your data layout – enabling you to focus on extracting insights from your data.

Let’s see Automatic Liquid Clustering is in action with an example table.

Consider a table `example_tbl`, which is frequently queried by `date` and `customer ID`. It contains data from `Feb 5-6` and `customer IDs A to F`. Without any data layout configuration, the data is stored in insertion order, resulting in the following layout:

![Automatic Liquid Clustering](https://www.databricks.com/sites/default/files/inline-images/announcing-automatic-liquid-clustering-blog-img-2.png?v=1741166280)

Automatic Liquid Clustering

Suppose the customer runs `SELECT * FROM example_tbl WHERE date = '2025-02-05' AND customer_id = 'B'`. The query engine leverages [Delta data skipping statistics](https://docs.databricks.com/aws/en/delta/data-skipping) (min/max values, null counts, and total records per file) to identify the relevant files to scan. Pruning unnecessary file reads is crucial, as it reduces the number of files scanned during query execution, directly improving query performance and lowering compute costs. **The fewer files a query needs to read, the faster and more efficient it becomes.**

In this case, the engine identifies 5 files for `Feb 5`, as half of the files have a min/max value for the `date` column matching that date. However, since data skipping statistics only provide min/max values, these 5 files all have a min/max `customer_id` that suggest `customer B` is somewhere in the middle. As a result, the query must scan all 5 files to extract entries for `customer B`, leading to a 50% file pruning rate (reading 5 out of 10 files).

![Data Layout](https://www.databricks.com/sites/default/files/inline-images/announcing-automatic-liquid-clustering-blog-img-3.png?v=1741166280)

Data Layout

As you see, the core issue is that `customer B` ’s data is not colocated in a single file. This means that extracting all entries for `customer B` also requires reading a significant amount of entries for other customers.

Is there a way to improve file pruning and query performance here? Automatic Liquid Clustering can enhance both. Here’s how:

## Behind the Scenes of Automatic Liquid Clustering: How It Works

Once enabled, Automatic Liquid Clustering continuously performs the following three steps:

11. Collecting **telemetry** to determine if the table will benefit from introducing or evolving Liquid Clustering Keys.
12. **Modeling the workload** to understand and identify eligible columns.
13. **Applying the column selection** and evolving the clustering schemes based on **cost-benefit analysis.**

![Predictive Optimization](https://www.databricks.com/sites/default/files/inline-images/predictive-optimization.png?v=1741200723)

Predictive Optimization

### Step 1: Telemetry Analysis

Predictive Optimization **collects and analyzes query scan statistics**, such as query predicates and JOIN filters, to determine if a table would benefit from Liquid Clustering.

With our example, Predictive Optimization detects that the columns `‘date’` and `‘customer_id’` are frequently queried.

### Step 2: Workload Modeling

Predictive Optimization evaluates the query workload and identifies the best clustering keys to **maximize data skipping.**

It learns from past query patterns and estimates the potential performance gains of different clustering schemes. By simulating past queries, it predicts how effectively each option would **reduce the amount of data scanned.**

In our example, using registered scans on `‘date’` and `‘customer_id’` and assuming consistent queries, Predictive Optimization calculates that:

- Clustering by `‘date’` reads 5 files with 50% pruning rates.
- Clustering by `‘customer_id’`, reads ~2 files (an estimate) with an 80% pruning rate.
	- Clustering by both `‘date’` and `‘customer_id’` (see data layout below) reads just 1 file with a 90% pruning rate.

![Workload Modeling](https://www.databricks.com/sites/default/files/inline-images/announcing-automatic-liquid-clustering-blog-img-5.png?v=1741166280)

Workload Modeling

![Clustering Keys](https://www.databricks.com/sites/default/files/inline-images/announcing-automatic-liquid-clustering-blog-img-6.png?v=1741166280)

Clustering Keys

### Step 3: Cost-benefit Optimization

The Databricks Platform ensures that any changes to clustering keys provide a clear performance benefit, as clustering can introduce additional overhead. Once new clustering key candidates are identified, Predictive Optimization **evaluates whether the performance gains outweigh the costs.** If the benefits are significant, it updates the clustering keys on Unity Catalog managed tables.

In our example, clustering by `‘date’` and `‘customer_id’` results in a 90% data pruning rate. Since these columns are frequently queried, the reduced compute costs and improved query performance justify the clustering overhead.

Preview customers have highlighted [Predictive Optimization's cost-effectiveness](https://www.databricks.com/blog/predictive-optimization-automatically-delivers-faster-queries-and-lower-tco), particularly its low overhead compared to manually designing data layouts. Companies like CFC Underwriting have reported **lower total cost of ownership** and significant efficiency gains.

> — Nikos Balanis, Head of Data Platform, CFC

The capability in a nutshell: **Predictive Optimization chooses liquid clustering keys on your behalf,** such that **the predicted cost savings from data skipping outweigh the predicted cost of clustering.**

## Get Started Today

If you haven’t enabled Predictive Optimization yet, you can do so by selecting Enabled next to Predictive Optimization in the account console under Settings > Feature enablement.

![Cost-benefit Optimization](https://www.databricks.com/sites/default/files/inline-images/announcing-automatic-liquid-clustering-blog-img-7.png?v=1741166280)

Cost-benefit Optimization

New to Databricks? Since November 11th, 2024, Databricks has enabled Predictive Optimization **by default** on all new Databricks accounts, running optimizations for all your Unity Catalog managed tables.

[Get started today](https://docs.databricks.com/en/delta/clustering.html#automatic-liquid-clustering) by setting `CLUSTER BY AUTO` on your Unity Catalog managed tables. Databricks Runtime 15.4+ is required to CREATE new AUTO tables or ALTER existing Liquid / unpartitioned tables. In the near future, Automatic Liquid Clustering will be enabled by default for newly created Unity Catalog managed tables. Stay tuned for more details.

---
Optimizing query performance of evolving data layouts as per the needs over time is always a challenging area in data landscape. Achieving optimal query performance without rewriting the whole existing data and that too in the context of changing query access patterns is one of the most desirable options. This is what it makes Automatic Liquid Clustering of Databricks Delta tables unique compared to the traditional approaches like Partitioning and Z-ordering

Partitioning is the process of physically dividing a table into subdirectories (partitions) based on the value of one or more columns. This helps limit the amount of data scanned during queries, especially when filtering on the partitioned columns. This is useful when query patterns frequently filter on partitioned columns.

Z-ordering on the other hand optimize data layout by co-locating related information across multiple columns — so that queries filtering on those columns scan less data and run faster.
Automatic liquid clustering is an advanced Delta Lake feature (currently in public preview and available in Databricks Runtime 15.4 LTS and above) that provides a smarter and more flexible alternative to Partitioning and Z-ordering. It reshapes how data is physically laid out on disk. It is just like reorganizing the books being just added in a library and gradually reshuffling the old books in background when needed. Even though liquid clustering involves physical re-write of data to disk to optimize the data layout for faster reads, the key point is that it happens incrementally in the background with minimal impact.

Behind the scenes in automatic liquid clustering,

· Monitors write patterns and queries

Once auto clustering is specified at table creation time – “CREATE TABLE my_table CLUSTER BY AUTO”, Delta Lake tracks query patterns and data distributions using metadata. It analyses

14) Frequently filtered columns

15) Joins

16) Skewed data

17) Column cardinality.

· Automatic Clustering Column selection

Based on these stats/metrics Databricks determines which columns are best to cluster by. It adapts over time as Data characteristics change and query patterns evolve. These clustering columns are not exposed in UI, but they drive the file layout decisions.

· Write-time optimization

Delta · buffers new rows in memory, it groups rows based on the chosen clustering keys and writes files with co-located rows

· Background Re-clustering (Adaptive Optimization)

Delta performs asynchronous re-clustering by checking the clustering drift, i.e. when file layout no longer matches optimal clustering. Only the affected files and re-written and this is done automatically, no need to manually schedule the jobs

· Faster queries via file skipping

Delta skips all irrelevant files using metadata collected during clustering dramatically improving the query performance

In summary the following scenarios gets benefited from Auto Liquid Clustering:

Ø Tables often filtered by high cardinality columns.

Ø Tables with significant skew in data distribution.

Ø Tables that grow quickly and require maintenance and tuning effort.
Ø Tables with concurrent write requirements.

Ø Tables with access patterns that change over time.

Ø Tables where a typical partition key could leave the table with too many or too few partitions.
