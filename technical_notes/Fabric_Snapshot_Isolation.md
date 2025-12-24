![Basics](https://github.com/ZosBHAI/pure_theory/blob/main/Snapshot_isolation.png)

  ### Need for Isolation:
  - When multiple transactions run at the same time, one transaction may read or modify data while another is updating it. Without proper control, this can cause dirty reads, inconsistent results, and lost updates. Transaction isolation levels are used by databases to manage concurrency and ensure data consistency.
 ### What is Snapshot isolation
 - Snapshot is called optimistic because it allows multiple transactions to do their work on the same data without much blocking. Reads can happen without blocking data modifications, and data modifications can happen without blocking reads.
 - > A basic comparison between SQL Server and Fabric Warehouse snapshot isolation is that multiple transactions can modify data in a SQL Server table as long as no transactions are trying to update the same row at the same time, while in Fabric this isn’t possible – multiple transactions cannot modify any rows in the same table even if they are not the same row.
### Write-Write Conflict
- Write-write conflicts can occur when two transactions attempt to UPDATE, DELETE, MERGE, or TRUNCATE the same table.
- Write-write conflicts or update conflicts are possible at the table level, since Fabric Data Warehouse uses table-level locking. If two transactions attempt to modify different rows in the same table, they can still conflict.
- ### Why concurrent INSERTs in Fabric Warehouse do NOT cause snapshot isolation conflicts
        - Because INSERTs only ADD new files and do NOT modify existing files.
        - Snapshot isolation conflicts happen only when existing files are changed or removed.
## Usecase - Write-Write Conflict in Microsoft Fabric Warehouse - Metadata-driven orchestration framework

### 📌 Overview
![Overview](https://github.com/ZosBHAI/pure_theory/blob/main/orchestration_framework.png)

We built a metadata-driven orchestration framework to control and execute ELT jobs in Fabric.

The framework manages:

- Ingestion jobs
- Transformation jobs
- Execution order based on job dependencies

Based on dependency configuration, the framework determines:

- ✅ Which jobs can run in parallel
- ⏳ Which jobs must wait for others to complete
### 🗄️ Metadata Storage
All orchestration metadata is stored in Microsoft Fabric Warehouse, including:
- Job configuration
- Dependency graph
- Job execution status:
  - Running
  - Success
  - Failed
- Each job updates its execution status back into the Fabric Warehouse.
---
### 🔄 Job Status Update Mechanism
The current implementation updates job status using an `UPDATE` statement.
- **Typical Lifecycle**
    - Job starts    → status = 'Running'
    - Job completes → status = 'Success' | 'Failed'
### ❗ Problem: Write-Write Conflict

A write-write conflict occurs when:

- Two or more jobs complete at the same time
- Each job tries to `UPDATE` its status in the same Warehouse table
- Transactions execute concurrently

#### Why This Happens

Fabric Warehouse uses **Snapshot Isolation**.
Under snapshot isolation:

- Concurrent `UPDATE` operations on the same table can conflict
- Conflicts occur even when different rows are updated

#### Result

- ✅ One transaction succeeds
- ❌ Another transaction fails with a snapshot isolation / write-write conflict

> ⚠️ This behavior is expected and by design in Fabric Warehouse.

---
### ✅ Solution 1: Append-Only Status Table 

The most effective solution is to eliminate `UPDATE` statements.

#### Approach

Use an append-only status table where:

- Every status change is logged as a new row
- Only `INSERT` statements are used
- Existing rows are never updated

**Example**
Job Started → INSERT row
Job Completed → INSERT row
Job Failed → INSERT row
#### Why This Works

- `INSERT` operations do not modify existing files
- Delta Lake allows concurrent `INSERT`s
- No snapshot isolation conflicts
> ⚠️ **Design consideration**
 - In an append-only design, each job execution inserts a new record rather than updating existing rows.As a result, multiple records can exist for the same job, each representing a different run.
 - To derive the current or latest run status, the query must:
    - Partition data by Job_ID (or equivalent)
    - Order runs by `Run_Start_Time`, `Run_End_Time`, or `Run_ID`
    - Select the most recent row using a window function.


### 🔁 Solution 2: Retry Mechanism

Handle conflicts by retrying failed operations.

#### Option A: Fixed Retry (Fabric Pipeline)

Fabric Pipelines support:

- Configurable retry count
- Configurable retry interval

**Behavior:**

1. Snapshot conflict occurs
2. Activity retries automatically
3. Succeeds after the conflicting transaction completes

#### Option B: Programmatic Retry (SQL / Python)

Retry logic can be implemented using:

- SQL
- Python (uses Tenacity,as it available in Fabric's Spark Cluster )


**Typical logic:**

1. Catch snapshot conflict error
2. Wait for a short interval
3. Retry until success
- **Fixed Retry Vs Programmatic Retry**
> Fixed retry is a out-of-box no-code solution, but it offers limited control over retry behavior. In contrast, programmatic retry provides greater implementation flexibility, allowing explicit definition of stop conditions (such as a maximum number of retry attempts) and wait strategies (such as exponential backoff between retries).

### 🏗️ Solution 3: Use an OLTP Database (Microsoft-Recommended)

Microsoft recommends moving transactional metadata to an OLTP system.

#### Recommended Approach

Move job status tracking to:

- SQL Server
- Or another OLTP database

#### Why?

Fabric Warehouse is optimized for:

- Analytics workloads
- Large-scale reads
- Batch processing

It is **not** designed for high-frequency transactional updates.

#### OLTP Advantages

- Row-level locking
- Better concurrency handling
- No snapshot write-write conflicts
---
## Artifacts:
![Notebooks &Pipeline](https://github.com/ZosBHAI/tech_notes/tree/master/technical_notes/ytube_contents/Fabric/Snapshot_Isolation/code)
