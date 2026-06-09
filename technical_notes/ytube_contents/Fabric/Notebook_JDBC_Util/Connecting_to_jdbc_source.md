
# Connecting to SQL Server and Microsoft Fabric Warehouse from Fabric Notebooks

## Overview

When working with Microsoft Fabric, there are multiple approaches available for connecting to SQL Server, Fabric Warehouse, and other JDBC-compatible data sources. Each approach has different capabilities, authentication mechanisms, and operational considerations.

This document summarizes the commonly used connectivity options and their limitations.

---

# Option 1: JayDeBeApi (JDBC from Python)

## Overview

JayDeBeApi allows Python code to connect directly to JDBC-compliant databases using Java JDBC drivers.

### Key Capabilities

* Supports reading from SQL Server and Fabric Warehouse.
* Supports writing to SQL Server and Fabric Warehouse.
* Works from standard Python notebooks.

### Advantages

* Supports both read and write operations.
* Flexible JDBC connectivity.
* Can be used with any JDBC-compatible source.

### Limitations

* Required Python packages and JDBC drivers must be installed on every Spark cluster/session.
* Additional setup and dependency management are required.

### Reference

https://milescole.dev/data-engineering/2024/09/27/Another-Way-to-Connect-to-the-SQL-Endpoint.html

---

# Option 2: T-SQL Notebook Support (%%tsql)

## Overview

Fabric notebooks support direct T-SQL execution using the `%%tsql` magic command.

### Example

```sql
%%tsql -artifact <warehouse_name> -type Warehouse -bind df

SELECT *
FROM dbo.MyTable
```

### How It Works

The notebook cell starts with %%tsql.

Fabric interprets the entire cell as T-SQL instead of Python.

Results can be bound to a DataFrame using the -bind parameter.


### Advantages

Simple and native Fabric experience.

No additional libraries required.

Ideal for querying Fabric Warehouse objects.

### Limitations

Read-only scenario.

Cannot write data back to the Fabric Warehouse.

### Best Use Cases

Ad-hoc querying.

Data exploration.

Validation and troubleshooting.

## Option 3: Spark Data Warehouse Connector

### Overview

The Spark Data Warehouse Connector is the recommended approach for reading and writing data between Spark and Microsoft Fabric Warehouse.

### Reading Data

Use the three-part namespace:
``` df = spark.read.synapsesql("<warehouse/lakehouse name>.<schema name>.<table or view name>")
```

### Writing Data

The connector supports writing Spark DataFrames directly to Fabric Warehouse tables.

### Internal Write Process

The connector performs a two-step write operation:

    Stage Spark DataFrame data into intermediate storage.

    Execute a COPY INTO operation to load data into the Warehouse.

### Authentication

Authentication is automatically handled by Fabric:

    Users authenticate to the Fabric workspace.

    Credentials are automatically passed to the SQL engine.

    No additional authentication configuration is required.

### Advantages

    Native Fabric integration.

    Supports both read and write operations.

    Simplified authentication.

    Optimized for large-scale data movement.

### Best Use Cases

    ETL workloads.

    Data ingestion pipelines.

    Large-scale Spark transformations.

### References

Microsoft Documentation:

https://learn.microsoft.com/en-us/fabric/data-engineering/spark-data-warehouse-connector?tabs=pyspark

Additional Reading:

https://medium.com/@karlovskyl/can-you-work-with-a-data-warehouse-from-a-fabric-notebook-a16f62e5c891

## Option 4: Fabric Connections

### Overview

Fabric Connections provide a centralized and secure mechanism for managing credentials and accessing external data sources.

### Key Highlights

#### Enhanced Security

    Credentials are stored securely.

    Connections can be shared across users.

    Sensitive information is not exposed in notebook code.

#### Supported Data Sources

    Azure Blob Storage

    Azure SQL Database

    PostgreSQL

    Azure Key Vault

    Amazon S3

    Other supported cloud services

#### Supported Authentication Methods

    Basic Authentication

    Account Key

    Access Token

    Workspace Identity

    Service Principal

#### Code Generation

Fabric can automatically generate Python code snippets that:

    Use the configured connection.

    Handle authentication securely.

    Simplify data access.

#### Benefits

    Centralized credential management.

    Improved governance.

    Reduced secret management effort.

#### Reference

YouTube Demo:

https://www.youtube.com/watch?v=qjUa_gyBAeM

## Option 5: Leveraging Spark JVM (spark._jvm)
- Spark exposes access to the underlying JVM through `spark._jvm`.
- Using this approach, Python code can directly invoke Java JDBC APIs running inside the Spark driver process through Py4J.
- Approach is tested for basic and token based authentication mode. Token based approach is available in the gist

### Key Characteristics

    Direct access to Java JDBC APIs.

    Greater flexibility compared to higher-level connectors.

    Fine-grained control over JDBC operations.

### Authentication Options

Supports all authentication methods supported by the JDBC driver, including:

    Username/Password

    Access Tokens

    Service Principals

    Other JDBC-supported authentication mechanisms

### Real-World Use Case

This approach has been used to:

    Update ETL metadata tables in SQL Server.

    Execute JDBC operations directly from Fabric notebooks.

The same pattern can also be used for:

    Microsoft Fabric Warehouse

    Azure SQL Database

    SQL Server

    Other JDBC-compatible platforms

### Example Implementation

GitHub Gist:



### Limitations

#### Driver-Only Execution

All JDBC operations execute on the Spark Driver.

#### No Executor Parallelism

Operations do not run in parallel across Spark Executors.

#### Scalability Considerations

Suitable for:

    Metadata operations

    Control tables

    Small-volume updates

Not recommended for:

    Large-scale data ingestion

    High-volume data movement


# Various ways to connect to Sql Server or JDBC source
https://milescole.dev/data-engineering/2024/09/27/Another-Way-to-Connect-to-the-SQL-Endpoint.html
Read/Write to Fabric Warehouse JayDeBeApi:
  - Supports read and write to Fabric Warehouse
  - Onnly thing,  this package needs to be innstalled everytime in the cluster
If you want to connect to Fabric Warehouse, read and write to Fabric Warehouse
There are 2 ways 
    if you want to  Read:
               Python Notebook: To enable it, the notebook cell must start with %%tsql. This tells Fabric to treat the entire cell as T-SQL instead of Python code.
               %%tsql -artifact <warehouse_name> -type Warehouse -bind df
                SELECT * FROM dbo.MyTable
              - Using this, we cannot write to Warehouse.
              Pyspark Notebook: Using spark-data-warehouse-connector. This can read and write to fabric warehouse.
              Authentication supportedd is  Users sign in to the Microsoft Fabric workspace, and their credentials are automatically passed to the SQL engine for authentication and authorization. The credentials are automatically mapped, and users aren't required to provide specific configuration options
              use the 3 nnamespace to access the table df = spark.read.synapsesql("<warehouse/lakehouse name>.<schema name>.<table or view name>")
              Write a Spark dataframe data to warehouse table: connector employs a two-phase write process to a Fabric DW table. Initially, it stages the Spark dataframe data into an intermediate storage, followed by using the COPY INTO command to ingest the data into the Fabric DW table.
              
              

     
                
https://medium.com/@karlovskyl/can-you-work-with-a-data-warehouse-from-a-fabric-notebook-a16f62e5c891

https://learn.microsoft.com/en-us/fabric/data-engineering/spark-data-warehouse-connector?tabs=pyspark

Using Fabric Connection: 
Key Highlights:

    Enhanced Security: You can now manage credentials securely and share connections with others without exposing sensitive information 
    Supported Sources: The feature supports various cloud data sources, including Azure Blob Storage, PostgreSQL, SQL, Azure Key Vault, and Amazon S3 .
    Flexible Authentication: It supports multiple authentication types, such as Basic, Account Key, Token, Workspace Identity, and Service Principal .
    Code Integration: Users can automatically generate Python code snippets from a configured connection, which handle the underlying authentication and data fetching securely .


Ref:
https://www.youtube.com/watch?v=qjUa_gyBAeM
Approach of leveraging the Spark JVM
- Using Spark's JVM bridge (spark._jvm) allows direct access to Java JDBC APIs and provides greater flexibility.The Python code invokes Java classes running inside the Spark driver process through Py4J.
- Authenntication cacn bee Basic, Token based like mentioned in gist
- this approach used to update the metadata in SQL sevver. Similar can be used for connecting to Fabric Warehouse
https://gist.github.com/ZosBHAI/f79ff727292a84dbdb8b56d05d39bfd2
- Limitations : All JDBC operations execute on the Spark driver.They do not run in parallel across executors.

