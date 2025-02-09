---
title: SQL_Programming
date: 2025-01-30T00:00:00.000Z
---
# Varchar Vs String

- With ORC, we will consider `STRING` for most of the datatype.
- `VARCHAR` and `STRING` are string values. `VARCHAR` is simply a layer on top of `STRING` that checks values during insert.  
  **Example:** Even if the data coming in is only `VARCHAR(30)` in length, your ELT/ETL processing will not  
  fail if you send in 31 characters while using a `STRING` datatype.
	
	# Exact Numeric Vs Approximate Numeric

- **Decimal** – Used when rounding error is not accepted.  
- **Float** – Used when rounding errors are acceptable, hence called approximate numeric.  
  - More performant in order.
  
# Difference Between CASE and WHERE

Consider a program that loops over multiple rows in a table:

### **CASE:**
- It is an operation applied to all rows (without using `WHERE`).
- **Use Case:**
  - When there are multiple filter conditions, consider using `CASE`.
  - Without `CASE`, we would have to use `WHERE` with `UNION`, leading to multiple table scans.
- CASE WHEN condition1 THEN a.some_column ELSE null END  ; when NULL it will not be counted
 
 **WHERE:** It is used to reduce the number of rows that are satisfied
 
 # WITH Clause (Common Table Expression) OR Statement-Scoped Views

## **Need for CTE**
- Even though SQL has functions and procedures, they are not the right tools for building easily understandable and reusable units.  
- **SQL-92** introduced **views**. Once created, a view has a name in the database schema so that other queries can use it like a table.  
- **SQL:1999** added the **WITH** clause to define **"statement-scoped views"**.  
  - They are **not** stored in the database schema.  
  - They are only **valid within the query** they belong to.  
  - This improves the structure of a statement without polluting the global namespace.  

## **Syntax**
```sql
WITH query_name (column_name1, ...) AS  
    (SELECT ...)  

SELECT ...
```

# Difference Between `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()`

When there is a tie in ranking, these functions behave differently:

## **1. `ROW_NUMBER()`**
- Assigns a **unique** rank to each row.
- Tied records are assigned ranks **randomly**.
- **Example:**  
  - Rimsha and Tiah may be ranked **2 or 3** in different query runs.

## **2. `RANK()`**
- Assigns the **same** rank to tied records.
- The **next rank is skipped** based on the number of tied records.
- **Example:**  
  - Rimsha and Tiah may both be ranked **2**.  
  - The next rank will be **4** instead of 3.

## **3. `DENSE_RANK()`**
- Assigns the **same** rank to tied records.
- **No gaps** in ranking.
- **Example:**  
  - Rimsha and Tiah may both be ranked **2**.  
  - The next rank will be **3**.

# Dealing with NULL in SQL

## **Boolean Logic with NULL**
When dealing with NULL values in logical conditions, the following rules apply:

- **`AND`**  
  - If **any** column is `FALSE`, the result is `FALSE`.  
  - For all other combinations, the output is `NULL`.

- **`OR`**  
  - If **any** column is `TRUE`, the result is `TRUE`.  
  - For all other combinations, the output is `NULL`.

- **`NOT`**  
  - `NOT NULL` evaluates to `NULL`.

## **NULL in Aggregations**
- **SUM Aggregation:**  
  - `NULL` values are ignored.  
  - **Example:** `SUM(1,2,3,4,NULL)` returns **10**.
  
- **AVG Aggregation:**  
  - Since `AVG(id) = SUM(id) / COUNT(id)`, the result is different if NULLs are present.

## **NULL in Joins**
- **INNER JOIN:** `NULL` values are ignored.  
- **OUTER JOIN:** `NULL` values will appear in the result.

## **`NOT IN` Clause and NULL**
- If the subquery (inner query) returns `NULL`, then the `NOT IN` condition evaluates to **false or unknown**.

### **Further Reading**
- [Demystifying NULL Values in SQL](https://towardsdatascience.com/demystify-null-values-in-sql-bc7e7e1b913a)  
- [Deep Dive into NULL in SQL](https://medium.com/@chengzhizhao/the-naughty-null-deep-dive-of-null-in-sql-9dbf4c39128)

# Sub-Query in SQL

### **Performance Considerations**
- Subqueries are executed as **separate `SELECT` statements**, increasing query execution time.  
- It is best to **minimize** the use of subqueries when possible.  
- If multiple subqueries perform the **same** computation, they will be executed **multiple times**, leading to inefficiency.

### **Example of a Subquery**
```sql
SELECT course, score, (
    SELECT ROUND(AVG(score),2) FROM results
    WHERE student_id = 9
) AS average_score
FROM results
WHERE student_id = 9 
AND score > (
    SELECT AVG(score) FROM results
    WHERE student_id = 9
)
ORDER BY score;
```
In this query, the average score for `student_id = 9` is computed twice.  
This can be highly inefficient for large tables.  

### Optimizing Subqueries  
Instead of using multiple subqueries, a better approach is to use **Common Table Expressions (CTEs)** to compute the value once and reuse it.  

# Recursive CTE in SQL

## **Overview**
- Used to handle **hierarchical data** in relational databases.  
- Can be used as a **for loop** for generating sequences (e.g., generating dates in a date dimension).  

## **Syntax**
```sql
WITH seq(n) AS
(
    SELECT 0 
    UNION ALL 
    SELECT n + 1 FROM seq   -- Like for loop initialization; UNION ALL is required (UNION is not supported)
    WHERE n < DATEDIFF(DAY, @StartDate, @CutoffDate)  -- Condition to terminate the loop
)
SELECT n FROM seq
ORDER BY n
OPTION (MAXRECURSION 0);
```
## Use Cases  

- **List down the manager for an employee.**  
- **Finding routes between cities.**  

## **Reference**  
[Recursive CTE in SQL Server](https://learnsql.com/blog/recursive-cte-sql-server/) 

# SQL Server - `GO` Statement

The `GO` statement is a **batch separator** in SQL Server.

- It's only a command for the **SQL Server tools** like SSMS, SQLCMD, and Azure Data Studio to separate batches for execution.
- Each batch separated by `GO` is executed independently.  
- If an error occurs in one batch, it will **not** affect other batches, unless the batch depends on data or objects from the previous one.