# Microsoft Fabric Capacity Planning Guide

## Estimating Capacity with Azure Calculator

When estimating Microsoft Fabric capacity using the Azure Calculator, consider the following key components:

## 1. Fabric Compute Capacity (F-SKU)

**References:**
- [Metrics App](https://learn.microsoft.com/en-us/fabric/enterprise/metrics-app)
- [Licenses](https://learn.microsoft.com/en-us/fabric/enterprise/licenses)

-  ### What Is Fabric Capacity and How Does It Work?
    -  [Ref](https://tomkeim.nl/fabric-over-capacity/)
    -  Capacities use stock-keeping units (SKUs). Each SKU provides Fabric resources for your organization. Your organization can have as many capacities as needed.
    
    - Each capacity SKU in Microsoft Fabric defines a specific amount of available Capacity Units (CU). For example:
    
    - The F2 SKU provides 2 CU per second
    - The F256 SKU provides 256 CU per second
    
    - Capacity Units are measured per second. So with an F256, you have access to 256 CU/s, which translates to:
    
      | Time Period | CU Consumption |
      | :--- | :--- |
      | Per second | 256 CU |
      | Per minute | 15,360 CU |
      | Per hour | 921,600 CU |
      | Per day | 22,118,400 CU |
      | Per 30 days | 663,552,000 CU |

- This is the **primary cost driver**
- Measured in **Caapacity Units (CUs)**
- All workloads (Power BI, Spark, Data Warehouse, etc.) consume from this pool unless offloaded
- **Note** SKU is charged per hour , even if it is not used. You can avoid this by **Pausing** the Capacity.

  ### Sample calculation:
    - #### Fabric Pipeline :
    - For example, every data movement in a Data Pipeline uses **1.5 CU-hours**, which equals **5,400 CU-seconds**.So for a second, 5400/(60x60) ~1.5 CU
      So, if you have a Copy Data activity (with no parallel throughput) running for 30 minutes, it will consume: **5,400 × 0.5 hour = 2,700 CU-seconds**.
      This means , 75% of an F2 capacity during that time **~0.75 CU-hours × $0.365 per hour = $0.27375**
   - #### Spark Notebook [Reddit Post on this](https://www.reddit.com/r/MicrosoftFabric/comments/1k7easw/is_my_understanding_of_fabricspark_cu_usage_and/)
   - **Concept:** One Capacity Unit = Two Spark VCores So F64 => 128 Spark Vcores and on which a 3x Burst Multiplier is applied which gives a total of 384 Spark VCores.
   - Lets say I want a cluster with 10 Spark Medium nodes. Each medium node is 8vcore. So 10 Medium nodes will take 80 vcores. I want to run this for 2 hours daily. An F16 SKU will give me 96Vcore with 3x bursting. So F16 should be sufficient for this usage.A spark cluster of **10 medium nodes** for 2 hours and stop it. 2 hours of Spark cluster will consume **80*2 = 160 vcores** in total. 1 F16 gives me **16*2 = 32 vcore per hour**. So, ideally I used 160/32 F16 hours, which is 5.
Cost of F16 for one hour is 2.88$. So cost of running my cluster is **2.88*5 = 15$**
        | Component | Value |
        | :--- | :--- |
        | Cluster size | 10 Medium nodes (80 vCores) |
        | Run duration | 2 hours |
        | Total vCore consumption | 160 vCore-hours |
        | F16 base capacity | 32 vCores per hour |
        | F16 hours consumed | 5 hours |
        | F16 hourly rate | $2.88 |
        | **Total cost** | **~$15** |
        
        > **Note:** This calculation assumes the cluster runs at full capacity for 2 hours. F16 with 3× bursting provides 96 vCores, which comfortably supports the 80 vCore requirement.
     
   - #### Fabric Warehouse [Billing example](https://learn.microsoft.com/en-us/fabric/data-warehouse/usage-reporting#billing-example)
       - **Concept:** 1 Fabric capacity unit = 0.5 Warehouse vCore
       - Capacity Metrics App tracks warehouse consumption across three operation categories.
           
            | Operation Category | Description |
            | :--- | :--- |
            | **Warehouse Query** | All user-generated and system-generated T-SQL statements within a warehouse |
            | **SQL Analytics Endpoint Query** | T-SQL queries on Lakehouse SQL endpoints |
            | **OneLake Compute** | All reads and writes for data stored in OneLake |
        - For example a query , consumes 100CU per seconds,and F-SKU is F16, hourly rate is $2.88:
             - **Step-by-step breakdown:**
                | Step | Calculation | Result |
                | :--- | :--- | :--- |
                | 1 | 100 × $2.28 | $228 |
                | 2 | $228 ÷ 3,600 seconds | $0.06333 |
              
                > **Note:** At $2.28 per CU-hour, a 100 CU-second query costs roughly **6.3 cents USD**.
         
- ### Analogy
- ![Anallogy] (https://github.com/ZosBHAI/tech_notes/blob/master/technical_notes/ytube_contents/Fabric/Capacity_Planning_Guide/analogy_with_kitchen.png)
     > Imagine a large food court inside a shopping mall. kitchen infrastructure — cooking space, staff, electricity, and equipment — represents the overall Microsoft Fabric Capacity.Now imagine different food counters such as pizza, burgers, desserts, and beverages. These represent different Fabric workloads like:
Power BI,Spark,Data Warehouse,Data Engineering
Even though customers place orders at different counters, all of them depend on the same kitchen infrastructure behind the scenes.Similarly, in Microsoft Fabric, all workloads consume resources from the same shared compute pool.
The size of the kitchen determines how much work can be handled simultaneously.A small kitchen can process only a limited number of orders, while a larger kitchen can process many more requests at the same time.This kitchen size is similar to an SKU in Microsoft Fabric.
     > Now imagine it is peak time and suddenly customer demand increases significantly.The kitchen owner decides to temporarily bring in extra cooks to handle the surge in orders.However, notice something important:The kitchen itself is not expanded permanently.Only additional staff are temporarily brought in to manage the increased workload.This is similar to **Bursting** in Microsoft Fabric.
     > Imagine suddenly 500 customers arrive at the food court.If the kitchen tries to prepare all 500 orders immediately, the kitchen becomes overloaded and operations slow down.Instead, the kitchen intelligently organizes the work.Urgent customer orders are prepared immediately.Less urgent work, such as ingredient preparation or bulk orders, can be scheduled over time.Microsoft Fabric works in a similar way.Rather than consuming all Capacity Units immediately, Fabric intelligently distributes the workload over time.This concept is called **Smoothing**.


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
#### Premium Per Capacity (P Capacity)
- This is called **P capacity** (higher tier)
- Here you will have all the features of F
- You have to enable it at the **Tenant level**
- It is billed **per capacity**, not per user
- **Rule of thumb:** If you have 10 users, don't go with this approach — buy F SKU + club with PPU or Pro license

#### Premium Per User (PPU)
Use PPU if you need features like:
- Direct Query
- Data Mart
- 48 refreshes
- Data Flow
- XMLA Endpoint

> **Note:** In this case you need F SKU.  
> Only if you need **48 refreshes per day**, use PPU. Otherwise, switch to F SKU — these come as a part of it.

#### Pro License
- Needed when we **do not have 48 refreshes + F SKU**
- Other features like Data Flow will still be available
