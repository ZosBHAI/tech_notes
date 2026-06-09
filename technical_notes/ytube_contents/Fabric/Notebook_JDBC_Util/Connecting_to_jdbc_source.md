
# Connecting to SQL Server and Microsoft Fabric Warehouse from Fabric Notebooks

## Overview

Following are approaches available for connecting to SQL Server, Fabric Warehouse, and other JDBC-compatible data sources. 

This document summarizes the commonly used connectivity options.

---

# Option 1: JayDeBeApi (JDBC from Python)

- JayDeBeApi allows Python code to connect directly to JDBC-compliant databases using Java JDBC drivers.

### Key Capabilities
* Supports both read and write operations.Can be used with any JDBC-compatible source. 

### Limitations

* In Fabric notebook, this Python packages must be installed on every Spark cluster/session.
* Additional setup and dependency management are required.

### Reference

https://milescole.dev/data-engineering/2024/09/27/Another-Way-to-Connect-to-the-SQL-Endpoint.html

---

# Option 2: T-SQL Notebook Support (%%tsql)

Fabric notebooks support direct T-SQL execution using the `%%tsql` magic command.

### Example

```sql
%%tsql -artifact <warehouse_name> -type Warehouse -bind df

SELECT *
FROM dbo.MyTable
```
Results can be bound to a DataFrame using the -bind parameter.

### How It Works

The notebook cell starts with %%tsql.

Fabric interprets the entire cell as T-SQL instead of Python.

### Limitations
Cannot write data back to the Fabric Warehouse.

## Option 3: Spark Data Warehouse Connector
Recommended approach for reading and writing data between Spark and Microsoft Fabric Warehouse.
### Reading Data

Use the three-part namespace:
```
df = spark.read.synapsesql("<warehouse/lakehouse name>.<schema name>.<table or view name>")
```

### Writing Data

The connector supports writing Spark DataFrames directly to Fabric Warehouse tables.

### Internal Write Process

The connector performs a **two-step** write operation:
    - Stage Spark DataFrame data into intermediate storage.
    - Execute a **COPY INTO** operation to load data into the Warehouse.

### Authentication

Authentication is automatically handled by Fabric:
    Users authenticate to the Fabric workspace.
    Credentials are automatically passed to the SQL engine.
    No additional authentication configuration is required.
### References
Microsoft Documentation:
https://learn.microsoft.com/en-us/fabric/data-engineering/spark-data-warehouse-connector?tabs=pyspark
Additional Reading:
https://medium.com/@karlovskyl/can-you-work-with-a-data-warehouse-from-a-fabric-notebook-a16f62e5c891

## Option 4: Fabric Connections
- Fabric Connections provide a centralized and secure mechanism for managing credentials and accessing external data sources.
- Sensitive information is not exposed in notebook code.
- Support only **cloud sources** not On-Premise.
- Fabric can automatically generate Python code snippets based on the configured credentials.
#### Reference

YouTube Demo: https://www.youtube.com/watch?v=qjUa_gyBAeM

## Option 5: Leveraging Spark JVM (spark._jvm)
- Spark exposes access to the underlying JVM through `spark._jvm`.
- Using this approach, Python code can directly invoke Java JDBC APIs running inside the Spark driver process through Py4J.
- Approach is tested for basic and token based authentication mode. Token based approach is available in the ![gist](https://gist.github.com/ZosBHAI/f79ff727292a84dbdb8b56d05d39bfd2).

- This approach has been used to:
    - Update ETL metadata tables in SQL Server/Fabric Warehouse.
    - Execute JDBC operations directly from Fabric notebooks on Microsoft Fabric Warehouse,Azure SQL Database or Other JDBC-compatible platforms.
- **Note:** All JDBC operations execute on the Spark Driver.Should not be used for large volume, for large volume of data Spark JDBC options should be considered.
  # Decision tree
``` mermaid
flowchart TD
    Start[("📓 Fabric Notebook<br>Python / Spark / T-SQL")]
    
    Start --> Decision{🎯 Destination &<br>Workload Type}
    
    %% On-Prem / Classic SQL Branch
    Decision -->|On-Prem SQL Server<br>or Generic JDBC| OnPremGroup[🏢 SQL Server /<br>JDBC Sources]
    
    OnPremGroup --> JayDeBeApi["🐍 Option 1: JayDeBeApi<br><br>✅ Read + Write<br>✅ Any JDBC source<br>⚠️ Per-session package install"]
    OnPremGroup --> SparkJVM["⚙️ Option 5: spark._jvm<br><br>✅ Metadata updates<br>✅ Token auth<br>⚠️ Driver only (not for bulk)"]
    
    %% Cloud / Fabric Warehouse Branch
    Decision -->|Fabric Warehouse<br>Azure SQL / Lakehouse| CloudGroup[☁️ Cloud Sources<br>Fabric WH / Azure SQL]
    
    CloudGroup --> SparkConnector["⭐ Option 3: Spark DW Connector<br>(Recommended)<br><br>✅ Native read/write<br>✅ Auto authentication<br>✅ COPY INTO for bulk"]
    CloudGroup --> TSQLMagic["📜 Option 2: %%tsql Magic<br><br>✅ Direct T-SQL<br>✅ Bind to DataFrame<br>❌ Write not supported"]
    CloudGroup --> FabricConn["🔐 Option 4: Fabric Connections<br><br>✅ Centralized secrets<br>✅ Auto code snippets<br>⚠️ Cloud sources only"]
    
    
    
    %% Recommendation summary
    subgraph Legend["📌 Decision Guide"]
        L1["🔹 Large volume read/write (Fabric WH) → <b>Spark DW Connector</b>"]
        L2["🔹 On‑prem / light metadata → <b>JayDeBeApi</b> or <b>spark._jvm</b>"]
        L3["🔹 Read‑only T‑SQL exploration → <b>%%tsql</b>"]
        L4["🔹 Secure cloud credential mgmt → <b>Fabric Connections</b>"]
    end
    
    %% Clickable links (GitHub Markdown compatible - renders in Mermaid Live / supported viewers)
    click JayDeBeApi "https://milescole.dev/data-engineering/2024/09/27/Another-Way-to-Connect-to-the-SQL-Endpoint.html" "JayDeBeApi: JDBC from Python"
    click SparkJVM "https://gist.github.com/ZosBHAI/f79ff727292a84dbdb8b56d05d39bfd2" "spark._jvm token auth example"
    click SparkConnector "https://learn.microsoft.com/en-us/fabric/data-engineering/spark-data-warehouse-connector?tabs=pyspark" "Spark Data Warehouse Connector docs"
    click TSQLMagic "https://medium.com/@karlovskyl/can-you-work-with-a-data-warehouse-from-a-fabric-notebook-a16f62e5c891" "%%tsql magic command"
    click FabricConn "https://www.youtube.com/watch?v=qjUa_gyBAeM" "Fabric Connections demo"
    
    style Start fill:#1e293b,stroke:#0f172a,color:#fff
    style Decision fill:#fefce8,stroke:#ca8a04,stroke-width:2px
    style OnPremGroup fill:#e2e3f5,stroke:#5b6e8c
    style CloudGroup fill:#d4f1f9,stroke:#2c6e9e
    style SparkConnector fill:#d1fae5,stroke:#10b981,stroke-width:2px
    style JayDeBeApi fill:#fff7ed,stroke:#f59e0b
    style SparkJVM fill:#f3e8ff,stroke:#a855f7
    style TSQLMagic fill:#ffedd5,stroke:#ea580c
    style FabricConn fill:#e0f2fe,stroke:#0284c7
    style Legend fill:#f8fafc,stroke:#cbd5e1,stroke-dasharray: 5 5
```


