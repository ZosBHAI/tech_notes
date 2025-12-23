![Basics](https://github.com/ZosBHAI/pure_theory/blob/main/Basics_Snapshot_isolation.pnghttps://github.com/ZosBHAI/pure_theory/blob/main/Basics_Snapshot_isolation.png)

  ### Need for Isolation:
  - When multiple transactions run at the same time, one transaction may read or modify data while another is updating it. Without proper control, this can cause dirty reads, inconsistent results, and lost updates. Transaction isolation levels are used by databases to manage concurrency and ensure data consistency.
 ### What is Snapshot isolation
 - Snapshot is called optimistic because it allows multiple transactions to do their work on the same data without much blocking. Reads can happen without blocking data modifications, and data modifications can happen without blocking reads.
