

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

## Introduction

This feature is available in Delta Lake 3.1.0 and above. See [Limitations](https://docs.delta.io/latest/delta-clustering.html#-limitations).

Announced at the 2023 Data + AI Summit \[[1](https://www.databricks.com/blog/announcing-delta-lake-30-new-universal-format-and-liquid-clustering)\], Delta Lake liquid clustering introduces an innovative optimization technique aimed at streamlining data layout in Delta Lake tables. Its primary goal is to enhance the efficiency of read and write operations while minimizing the need for tuning and data management overhead. Liquid clustering is specifically designed to address the challenges posed by Hive-style partitioning and Z-ordering. It provides a highly adaptive solution, particularly in the context of evolving data patterns, scaling demands, and data skew complexities.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*Q7T3A_GDswARU0-RfdZCZw.png)

In this article, we will compare partitioning, Z-ordering, and liquid clustering using a practical dataset example. We will examine the inner workings, strengths, and weaknesses of traditional data-layout methods before contrasting them with the benefits of liquid clustering. Additionally, we will provide examples to guide the reader through the implementation of liquid clustering.

## Background

## Example Dataset

To better illustrate our discussion, we will refer to the following example of a transaction dataset throughout this write-up:

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*ftwCGwGhQU7eIGkGzBnwUA.png)

Example Transaction Dataset

## What is a Data Layout? Why is It Important?

Data layout is the way data is stored in memory or on disk. It is the physical arrangement of the data, which can have a great impact on the performance of the queries and data operations.

A data layout is considered balanced when the data is evenly distributed across the storage medium. In the context of Apache Parquet and Delta Lake \[[2](https://www.databricks.com/glossary/what-is-parquet)\]\[[3](https://docs.databricks.com/en/delta/index.html)\], the goal of optimizing a data layout is to arrive at a uniform file size and an appropriate number of files across a given dataset. A balanced data layout is essential for efficient data skipping during query execution. Data skipping ensures irrelevant data files are ignored and not read into memory for query processing. Data skew and other inconsistencies can lead to suboptimal data skipping, increased execution times, and potential failures in execution.

## Hive-style Partitioning

Hive-style partitioning is a common way to improve the performance of queries on large datasets stored in data lakes \[[4](https://docs.databricks.com/en/sql/language-manual/sql-ref-partition.html)\]. It involves dividing the data into smaller partitions, i.e., sets of files, where the partition information is stored as part of each file’s path (see example below). This technique can help speed up queries significantly by enabling data skipping during scanning.

```c
/transactions/month=2023-1/country=Germany/{1.parquet, 2.parquet,...}
/transactions/month=2023-2/country=USA/{30.parquet, 31.parquet,...}
/transactions/month=2023-3/country=Mexico/{53.parquet, 54.parquet,...}
...
```

Partitioning also comes with a few drawbacks. Storing the partition information as part of the file path introduces physical boundaries between partitions and makes partitioning overall a less flexible, static layout technique. As a result, it can be difficult to manage and update partitions, where a complete rewrite of the data may be necessary.

Partitioning can also lead to data skew, meaning some partitions are much larger than others. Furthermore, partitioning can be effectively applied on low-cardinality fields, i.e., columns with a low number of distinct values (*country* and *month* in our example) \[[5](https://docs.databricks.com/en/tables/partitions.html)\]. Partitioning by high cardinality columns (*transactionId, transactionTime, senderId/recipientId*, etc.) would create many small files that cannot be combined, resulting in poor scan performance.

The following examples demonstrate some of the typical challenges that can arise with partitioning:

1\. ==Some partitions may have small data or no data at all.==

2\. Other partitions may have multiple, large files that should ideally be broken down further.

3\. Yet another group of partitions may each have one smaller file that should be ideally merged across these partitions.

4\. Data may get ingested on a frequent basis, resulting in small, fragmented data files. A background maintenance job needs to be scheduled on a regular basis to optimize these files.

The diagram below illustrates these scenarios. Please note that the number of files is simplified for demonstration and may not be truly representative of an actual transaction dataset.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*u1w9IsT3qmZqnk9NghQ-fA.png)

Partitioning (Example: Unbalanced/Unoptimized Transaction Dataset)

## Z-ordering

Z-ordering is a data-layout optimization technique that can be used to cluster data within partitions for improved query performance (note that using partitions is not a prerequisite though) \[[6](https://docs.databricks.com/en/delta/data-skipping.html)\]. Z-ordering is recommended to be used on high-cardinality columns. Thus, it is complementary to partitioning. Z-ordering co-locates related data in the same set of files and this co-locality is in turn leveraged to further improve data skipping during scanning.

In order to further explore Z-ordering, we should first review the building block of this clustering method. A Z-cube is a collection of clustered files produced as a result of Z-ordering. Once a Z-cube reaches a certain threshold in size, the cube is considered stable (sealed). This means that a stable Z-cube’s data files are not considered for clustering when the dataset is Z-ordered next time. Furthermore, an unstable Z-cube gets combined and repeatedly re-clustered with newly ingested data files until the Z-cube reaches a stable state.

Although Z-ordering is a highly effective data-clustering technique, the above attributes lead to a few implications:

1. Z-ordering is not idempotent; nevertheless, it aims to be an incremental operation \[[6](https://docs.databricks.com/en/delta/data-skipping.html)\]. Consequently, the time it takes to Z-order is not guaranteed to lessen over multiple runs.
2. Z-ordering clusters data within partitions and not across partition boundaries.
3. Stable Z-cubes may still need to be opened up for future clustering when DML operations (merge, update, delete) are performed on existing records.
4. Query patterns may change over time, requiring a new set of clustering (and partitioning) keys. Applying a new set of keys can be compute-intensive for large datasets.

While the maintenance time of large tables can be significantly reduced by Z-ordering targeted partitions, data management overhead can increase in parallel:

```c
-- Example of Z-ordering in targeted partitions
OPTIMIZE transactionsTable
WHERE month >= to_date('2023-02-01')
ZORDER BY (senderId)
```

Revisiting our example of the transaction table, file sizes are optimized and data is co-located by *senderId* within partitions after performing Z-ordering:

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*mhKMq2fDDpqHQjxenzEZBQ.png)

Partitioning plus Z-ordering (Example: Optimized Transaction Dataset)

The combination of partitioning and Z-ordering can overall still be considered a static layout technique. We will see how liquid clustering can overcome the above limitations next.

## How is Liquid Clustering Different?

Unlike Hive partitioning and Z-ordering, liquid clustering keys can be chosen purely based on the query predicate pattern, with no worry about future changes, cardinality, key order, file size, and potential data skew. Liquid clustering is flexible and adaptive (hence, its name), meaning that

1. Clustering keys can be changed without necessarily rebuilding the entire table
2. It eliminates the concept of partitions and can dynamically merge or further divide files in order to arrive at a balanced dataset with the ideal number and size of files.

Liquid clustering accomplishes the above by leveraging a tree-based algorithm to incrementally map the data layout and maintain the associated metadata as part of the table’s Delta Lake logs. Consequently, liquid clustering is stateful and does not need to be recomputed each time an `OPTIMIZE` command is executed.

This property also makes a truly incremental clustering possible, meaning that newly ingested data gets clustered as necessary — ignoring previously clustered data. Operating on larger blocks compared to Z-ordering also makes table maintenance significantly more efficient. Moreover, cluster-on-write (also called eager clustering) becomes possible following this approach, with clustering performed as part of the ingestion.

Eliminating the need for physical partition boundaries and dynamically merging/dividing data files can dramatically boost data skipping. Persisting the metadata also allows for skew detection and correction as well as better concurrency support at the record rather than partition level.

Revisiting our earlier example of the transaction dataset, liquid clustering provides a much higher degree of flexibility across previous partition boundaries, achieving the desired file size in a more consistent way:

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*Lr4lfzNGILCxrP1XWf7UYw.png)

Liquid Clustering (Example: Balanced Transaction Dataset)

Databricks recommends liquid clustering for all new Delta tables \[[7](https://docs.databricks.com/en/delta/clustering.html)\]. Based on the above, it can be particularly beneficial for the following scenarios:

- Tables are often filtered by columns with different cardinalities
- Tables have a significant skew in data distribution
- Tables grow quickly and require maintenance and tuning effort
- Tables have concurrent write requirements
- Tables have access patterns that change over time
- Tables where a typical partition key could leave the table with too many or too few partitions.

## Benefits

To recap, liquid clustering offers a number of benefits over Hive-style partitioning and Z-ordering:

- **Improved performance**: Liquid clustering can significantly improve the performance of read and write operations on Delta Lake tables. It also provides better concurrency support and is more efficient at data maintenance due to its incremental approach \[[8](https://medium.com/closer-consulting/liquid-clustering-first-impressions-113e2517b251)\].
- **Easier management**: Liquid clustering is easier to manage and update than partitioning or Z-ordering (see next section for examples). It supports the evolution of clustering keys and reduces the required maintenance on tables.
- **Reduced data skew and fragmentation**: Liquid clustering can minimize data skew and fragmentation, which can significantly improve query performance.

## How to Use It?

To use liquid clustering \[[7](https://docs.databricks.com/en/delta/clustering.html)\], you need to create a Delta Lake table with the `CLUSTER BY` phrase specifying the clustering columns:

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

In order to ensure sufficient clustering across the dataset regardless of ingestion size, you can trigger clustering by running `OPTIMIZE` as the following:

```c
OPTIMIZE transactions
```

For tables with frequent updates or inserts, a background `OPTIMIZE` job can be scheduled to optimize data layout on a regular basis. Due to the incremental nature of clustering, table maintenance is computationally cheap and runs in a short time. Downstream processes and users can continue accessing the table simultaneously without an impact due to Delta Lake’s ACID transactions \[[9](https://docs.databricks.com/en/lakehouse/acid.html)\].

When query predicates change over time, you can set new clustering keys on the table as below. Subsequent `OPTIMIZE` will cluster newly ingested data according to the new set of clustering keys.

```c
ALTER TABLE transactions CLUSTER BY (country, month, senderId);
OPTIMIZE transactions;
```

## Current Limitations

As of writing this article, Delta liquid clustering is in Public Preview. The following limitations exist \[[7](https://docs.databricks.com/en/delta/clustering.html)\]:

- You can only specify columns with statistics collected for clustering keys. By default, the first 32 columns in a Delta table have statistics collected.
- You can specify up to 4 columns as clustering keys.
- Structured Streaming workloads do not support clustering-on-write.

## Conclusion

This write-up has compared Hive-style partitioning, Z-ordering, and Delta Lake liquid clustering using a practical example with a transaction dataset. Liquid clustering, an innovative approach to data layout optimization in Delta Lake tables, stands out as a superior option. It offers enhanced efficiency and flexibility compared to Hive-style partitioning and Z-ordering, leading to significant improvements in query performance and data maintenance. In addition, liquid clustering excels in handling data skew and evolving query patterns. Databricks recommends adopting liquid clustering for all new Delta tables.

## References

\[1\] Databricks, [Announcing Delta Lake 3.0 with New Universal Format and Liquid Clustering](https://www.databricks.com/blog/announcing-delta-lake-30-new-universal-format-and-liquid-clustering)

\[2\] Databricks, [What is Parquet?](https://www.databricks.com/glossary/what-is-parquet)

\[3\] Databricks, [What is Delta Lake?](https://docs.databricks.com/en/delta/index.html)

\[4\] Databricks, [Partitions](https://docs.databricks.com/en/sql/language-manual/sql-ref-partition.html)

\[5\] Databricks, [When to partition tables on Databricks](https://docs.databricks.com/en/tables/partitions.html)

\[6\] Databricks, [Data skipping with Z-order indexes for Delta Lake](https://docs.databricks.com/en/delta/data-skipping.html)

\[7\] Databricks, [Use liquid clustering for Delta Table](https://docs.databricks.com/en/delta/clustering.html)

\[8\] Gustavo Martins (Closer Consulting), [Liquid Clustering: First Impressions](https://medium.com/closer-consulting/liquid-clustering-first-impressions-113e2517b251)

\[9\] Databricks, [What are ACID guarantees on Databricks?](https://docs.databricks.com/en/lakehouse/acid.html)


---
## Introduction to Delta Lake Liquid Clustering

As your Delta tables grow in size, the need for performance tuning in Microsoft Fabric becomes essential. In this post, I’ll explore two powerful optimisation techniques — **Delta Lake Partitioning** and **Liquid Clustering**. Both can help improve query speed and reduce costs, but they work in very different ways.

We’ll break them down, compare them, and help you decide which one fits your scenario best. Earlier, I already wrote about partitioning, so I will focus on liquid clustering today and compare that to partitioning.

## Reminder: What is Delta Lake Partitioning?

[Partitioning](https://thatfabricguy.com/delta-lake-partitioning-for-microsoft-fabric/) is the classic way to optimise large Delta tables. It splits your data physically into subfolders based on one or more columns — often a date or category.

If your query filters on the partitioned columns, only relevant partitions are scanned, which can dramatically speed things up. However, this approach comes with trade-offs that we’ve mentioned in the post on partitioning.

## What is Liquid Clustering in Delta Lake?

Liquid Clustering is a newer, more dynamic approach available in Delta Lake and therefore in Microsoft Fabric. Instead of physically splitting data into folders like traditional partitioning does, Liquid Clustering logically clusters your data by values in one or more columns — all without changing the table’s physical layout.

So what does that mean in practice? Under the hood, Liquid Clustering organises your data into what’s effectively a logical sort order during writes. It groups together rows with similar values in the clustering columns. These grouped rows end up stored in the same files or file ranges. Then, when you run queries with filters on those clustered columns, Spark can quickly skip over unrelated data by consulting file-level statistics (like min and max values for each file).

This behaviour is very similar to Z-Ordering in traditional Delta Lake, but with two big differences: you don’t need to explicitly run a maintenance command like `OPTIMIZE ZORDER BY`, and the clustering is maintained incrementally with each write. In effect, Liquid Clustering creates a kind of soft indexing on your data — it doesn’t guarantee perfect ordering, but it improves data locality, reduces I/O, and allows Spark to prune files much more efficiently at query time.

The result? Faster queries, especially on large tables with high-cardinality filter columns like product IDs, user IDs, or timestamps — and far less engineering effort to maintain it.

## Partitioning vs Liquid Clustering – How They Work Under the Hood

### Storage Layout

Partitioning results in physical folders like `/PickupDate=2025-04-01/`. Clustering, on the other hand, leaves your data in place but adds metadata about how it’s logically grouped.

### Write-Time Complexity

Partitioning must be defined when the table is created. Changing it later involves rewriting the data. Liquid Clustering allows for adjustments without reloading the whole table. This makes maintenance much easier, and I like that!

### Query-Time Optimisation

Both techniques enable data skipping, but the mechanics differ. Partitioning uses folder-based pruning, and Liquid Clustering relies on Spark’s file statistics to skip files. This also implies you need quite large tables before Liquid Clustering starts to make sense. Because if you can compact your delta table into one single parquet file, clustering won’t add any benefits.

### Maintenance Effort

Partitioning requires careful planning and can degrade performance if overused. Liquid Clustering is more forgiving and easier to evolve over time.

## Performance Considerations

We’ve tested both approaches on large datasets — in some cases with billions of rows — and the results may surprise you.

For highly selective filters (e.g. one day’s worth of data), partitioning often wins. But for broader queries or when filter values vary, Liquid Clustering can shine thanks to its adaptive design.

### Querying a Single Partition

Partitioning performs well here. Spark scans only what it needs which is fast and efficient. In the previous post we saw good improvements in read performance for this exact scenario on a very large scale table (± 4 billion rows).

### Querying Across Many Partitions

This is where partitioning can backfire. Reading hundreds of small files adds overhead, whereas Liquid Clustering can streamline the process. We have seen this in the previous post as well, where reading data over multiple partitions actually took longer than the same query on the unpartitioned table.

Liquid clustering has less problems in this scenario, because it doesn’t necessarily split the data into to too many small files. Rather, it keeps the file size in Spark’s optimum, and based on your query, selects the appropriate files it expects the data in.

### Full Table Scans

When scanning all data, partitions add no benefit — and might even slow things down. Clustering keeps things leaner. But even then, I expect not that big of a difference with clustering versus a not optimised table.

## Best Practices for Partitioning

- Use only when your query patterns consistently filter on the partition key.
- Choose low-cardinality columns (like date or category) to avoid file explosion. And even then, be wary. Keys like data could quickly blow up on tables with lots of history, and if your filters usually are on year and month, partitioning on date won’t do a thing.
- Avoid partitioning small or medium sized tables because the overhead isn’t worth it. Only the largest tables benefit from partitioning.
- Once set, changing the partitioning requires rewriting the entire table. This is a major drawback.

## Best Practices for Liquid Clustering

- Use it when queries vary or evolve over time.
- Apply Liquid Clustering to high-cardinality columns without performance penalties.
- Also easy to use if you plan on minimising your maintenance efforts.
- Liquid Clustering can be ideal when working with medium sized tables.

## Can I Use Both?

Absolutely. A hybrid approach is often the sweet spot.

For example, coarse partitioning by year, combined with Liquid Clustering by product or region, can give you the best of both worlds. Just make sure your query patterns align with how your data is structured and clustered.

## When to Use Liquid Clustering or Partitioning for Delta Lake?

The choice between these two techniques is largely driven by the factors I’ve mentioned above in this article. But another big factor we haven’t discussed yet is the table size.

As mentioned, the partitioning of small tables is not efficient because of the overhead when reading from multiple small files. Spark prefers larger Parquet files, so too much partitioning doesn’t work that well.

The same goes for liquid clustering, where file elimination is triggered based on the metadata of those files. If there are just 1-2 files in a delta table, this will not make sense.

From research, I’ve found the following general guidelines. There is no definitive answer as to which strategy to use for which table size, but when looking at the orders of magnitude you can come to the following guidelines:

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

1. Collecting **telemetry** to determine if the table will benefit from introducing or evolving Liquid Clustering Keys.
2. **Modeling the workload** to understand and identify eligible columns.
3. **Applying the column selection** and evolving the clustering schemes based on **cost-benefit analysis.**

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

1) Frequently filtered columns

2) Joins

3) Skewed data

4) Column cardinality.

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
