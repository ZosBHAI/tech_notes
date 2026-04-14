# Microsoft Fabric Capacity Planning Guide

## Estimating Capacity with Azure Calculator

When estimating Microsoft Fabric capacity using the Azure Calculator, consider the following key components:

## 1. Fabric Compute Capacity (F-SKU)

**References:**
- [Metrics App](https://learn.microsoft.com/en-us/fabric/enterprise/metrics-app)
- [Licenses](https://learn.microsoft.com/en-us/fabric/enterprise/licenses)

-  ### What Is Fabric Capacity and How Does It Work?
-  [Ref](https://tomkeim.nl/fabric-over-capacity/)

Each capacity SKU in Microsoft Fabric defines a specific amount of available Capacity Units (CU). For example:

- The F2 SKU provides 2 CU per second
- The F256 SKU provides 256 CU per second

Capacity Units are measured per second. So with an F256, you have access to 256 CU/s, which translates to:

| Time Period | CU Consumption |
| :--- | :--- |
| Per second | 256 CU |
| Per minute | 15,360 CU |
| Per hour | 921,600 CU |
| Per day | 22,118,400 CU |
| Per 30 days | 663,552,000 CU |

- This is the **primary cost driver**
- Measured in **Compute Units (CUs)**
- All workloads (Power BI, Spark, Data Warehouse, etc.) consume from this pool unless offloaded

  ### Sample calculation:
    - #### Fabric Pipeline :
    - For example, every data movement in a Data Pipeline uses **1.5 CU-hours**, which equals **5,400 CU-seconds**.So for a second, 5400/(60x60) ~1.5 CU
      So, if you have a Copy Data activity (with no parallel throughput) running for 30 minutes, it will consume: **5,400 × 0.5 hour = 2,700 CU-seconds**.
      This means , 75% of an F2 capacity during that time
   - #### Spark Notebook
   - #### Fabric Warehouse 
      


## 2. Spark Autoscaling (Serverless Offloading)

**References:**
- [Spark Compute](https://learn.microsoft.com/en-us/fabric/data-engineering/spark-compute)
- [Spark Job Concurrency and Queueing](https://learn.microsoft.com/en-us/fabric/data-engineering/spark-job-concurrency-and-queueing)

### What is this feature?

When enabled:
- Spark jobs are offloaded from the reserved Fabric capacity
- They run on dedicated serverless resources
- They do **NOT** consume CUs from the fixed capacity pool

### Why was it introduced?

- Fabric uses a fixed-capacity model
- Spark workloads are typically heavy compute consumers
- Offloading enables:
  - True elastic scaling
  - Isolation of workloads
  - Protection for interactive workloads like:
    - Power BI reports
    - Data Warehouse queries

### Autoscaling vs Bursting vs Smoothing

**References:**
- [Throttling](https://learn.microsoft.com/en-us/fabric/enterprise/throttling)
- [Capacity Planning](https://learn.microsoft.com/en-us/fabric/enterprise/capacity-planning)

| Feature | Description |
|---------|-------------|
| **Bursting** | Allows temporary usage up to ~12x the provisioned SKU |
| **Smoothing** | Spreads CU usage over time: 5–64 minutes (interactive), up to 24 hours (background jobs) |
| **True Autoscaling** | Not natively available for SKU. Requires manual scaling in Azure Portal OR automation via Azure Logic Apps / Azure Runbooks |

---

## 3. OneLake Storage

**References:**
- [OneLake Overview](https://learn.microsoft.com/en-us/fabric/onelake/onelake-overview)
- [OneLake Consumption](https://learn.microsoft.com/en-us/fabric/onelake/onelake-consumption)

### Key Concepts
- Uses **ZRS** (Zone-Redundant Storage) where available
- Otherwise **LRS** (Locally Redundant Storage)
- Storage and caching are charged separately

### Important Considerations
> ⚠️ You continue to pay for storage even after workspace deletion (during disaster recovery retention period)

### OneLake BCDR (Business Continuity & Disaster Recovery)

**References:**
- [OneLake Disaster Recovery](https://learn.microsoft.com/en-us/fabric/onelake/onelake-disaster-recovery)
- [Disaster Recovery Operation Types](https://learn.microsoft.com/en-us/fabric/onelake/onelake-consumption#disaster-recovery-operation-types)

- **Why BCDR?** Default storage protects against hardware failure but does **NOT** protect against regional failure
- BCDR enables cross-region disaster recovery

**Key Notes:**
- Replication is **asynchronous**
- Possible data loss during disaster
- Enabling BCDR:
  - Increases CU consumption
  - Increases storage usage

**Example CU Consumption:**
| Operation | CU Seconds |
|-----------|-------------|
| Normal write | 1,626 |
| BCDR write | 3,056 |

### OneLake Cache

**References:**
- [OneLake Shortcuts](https://learn.microsoft.com/en-us/fabric/onelake/onelake-shortcuts)
- [OneLake Consumption](https://learn.microsoft.com/en-us/fabric/onelake/onelake-consumption)

- **Why use cache?** When accessing external data (e.g., S3, GCS, on-prem):
  - Reduces latency
  - Reduces egress costs

**Benefits:**
- Stores a local copy in OneLake
- Eliminates repeated external reads
- Supports caching duration: 1 to 28 days

---

## 4. Mirroring

**Reference:** [Mirroring Overview](https://learn.microsoft.com/en-us/fabric/data-factory/mirroring-overview)

- Example: F64 SKU → 64 TB storage
- Compute for mirroring is included (no extra cost)

---

## 5. Data Transfer Costs

**Reference:** [Azure Bandwidth Pricing](https://azure.microsoft.com/en-in/pricing/details/bandwidth/)

Charges apply for:
- Cross-region data movement
- Cross-cloud data ingestion

### Key Risk
> ⚠️ If Fabric capacity is in a different region than your data, you **may incur egress charges**

---

## 6. Licensing Model (Fabric + Power BI)

**References:**
- [Power BI License Types](https://learn.microsoft.com/en-us/power-bi/fundamentals/service-features-license-type)
- [Fabric Licenses](https://learn.microsoft.com/en-us/fabric/enterprise/licenses)

> Fabric pricing is **NOT** purely user-based (unlike Tableau/OBIEE)

### Pricing Model
