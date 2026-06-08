
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

