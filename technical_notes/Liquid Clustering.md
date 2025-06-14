

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



