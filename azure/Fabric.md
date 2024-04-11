# Introduction 
Fabric includes the following services:

- Data engineering
- Data integration
- Data warehousing
- Real-time analytics
- Data science
- Business intelligence

## OneLake
OneDrive for data

![Alt text](/images/image.png)

One important feature of OneLake is the ability to **create shortcuts**

![Alt text](/images/image2.png)


Fabric: 

- **Synapse Data Engineering**: data engineering with a Spark platform for **data transformation at scale**.

- **Synapse Data Warehouse**: data warehousing with industry-leading SQL performance and scale to support data use.

- **Synapse Data Science**: data science with Azure Machine Learning and Spark for model training and execution tracking in a scalable environment.

- **Synapse Real-Time Analytics**: real-time analytics to query and analyze large volumes of data in real-time.

- **Data Factory**: data integration combining Power Query with the scale of Azure Data Factory to **move and transform data**.

- **Power BI**: business intelligence for translating data to decisions.

**Fabric Admin**

# Lakehouses 

## Create, explore lakehouse

- **Lakehouse** is the lakehouse storage and metadata, where you interact with files, folders, and table data.

- **Semantic model** (default) is an automatically created data model based on the tables in the lakehouse. Power BI reports can be built from the semantic model.

- **SQL Endpoint** is a read-only SQL endpoint through which you can connect and query data with Transact-SQL.

## Ingest data into a lakehouse
Upload

Dataflows

Notebooks

Data Factory

## Access data with shortcuts

Shortcuts enable you to integrate data into your lakehouse while keeping it stored in external storage.

Shortcuts are useful when you need to source data that's in a different storage account or even a different cloud provider.

## Explore and transform data

Apache Spark

SQL analytic endpoint

Dataflows Gen2

Data pipelines

### Analyze and visualize data in a lakehouse

# Apache Spark

Apache Spark is an open source parallel processing framework for large-scale data processing and analytics.

Spark uses a "divide and conquer" approach to processing large volumes of data quickly by distributing the work across multiple computers

In Microsoft Fabric, each **workspace** is assigned a **Spark cluster**. An administrator can manage settings for the Spark cluster in the Data Engineering/Science section of the workspace settings.

- Node Family: The type of virtual machines used for the Spark cluster nodes. Memory optimized nodes provide optimal performance.
- Runtime version: The version of Spark (and dependent subcomponents) to be run on the cluster.
- Spark Properties: Spark-specific settings that you want to enable or override in your cluster.

Spark Notebook

Spark job

> The Parquet format is typically preferred for data files that you will use for further analysis or ingestion into an analytical store. Parquet is a very efficient format that is supported by most large scale data analytics systems. In fact, sometimes your data transformation requirement may simply be to convert data from another format (such as CSV) to Parquet!


Partitioning is an optimization technique that enables Spark to maximize performance across the worker nodes.

```python
bikes_df.write.partitionBy("Category").mode("overwrite").parquet("Files/bike_data")
```

The Spark catalog supports tables based on files in various formats. The preferred format in Microsoft Fabric is delta

# Delta Lake
Tables in a Microsoft Fabric lakehouse are based on the Linux foundation **Delta Lake table format**

Benefits:

- Relational tables that support querying and data modification.
- Support for ACID transactions.
- Data versioning and **time travel**.
- Support for batch and streaming data.
- Standard formats and interoperability.

## Create delta table

From dataframe

```python
# Load a file into a dataframe
df = spark.read.load('Files/mydata.csv', format='csv', header=True)

# Save the dataframe as a delta table
df.write.format("delta").saveAsTable("mytable")
```

The data for the table is saved in **Parquet** files in the **Tables** storage area in the lakehouse, along with a **_delta_log** folder

*external* tables

```python
df.write.format("delta").saveAsTable("myexternaltable", path="Files/myexternaltable")

df.write.format("delta").saveAsTable("myexternaltable", path="abfss://my_store_url..../myexternaltable")
```

*DeltaTableBuilder* API

```python
from delta.tables import *

DeltaTable.create(spark) \
  .tableName("products") \
  .addColumn("Productid", "INT") \
  .addColumn("ProductName", "STRING") \
  .addColumn("Category", "STRING") \
  .addColumn("Price", "FLOAT") \
  .execute()
```

Spark SQL

```SQL
%%sql
CREATE TABLE salesorders
(
    Orderid INT NOT NULL,
    OrderDate TIMESTAMP NOT NULL,
    CustomerName STRING,
    SalesTotal FLOAT NOT NULL
)
USING DELTA
```

Save data in delta format

```python
delta_path = "Files/mydatatable"
df.write.format("delta").save(delta_path)

new_df.write.format("delta").mode("overwrite").save(delta_path)
```

## Work with delta tables

Spark SQL

```python
spark.sql("INSERT INTO products VALUES (1, 'Widget', 'Accessories', 2.99)")
```

or 

```sql
%%sql

UPDATE products
SET Price = 2.49 WHERE ProductId = 1;
```

Delta API

```python
from delta.tables import *
from pyspark.sql.functions import *

# Create a DeltaTable object
delta_path = "Files/mytable"
deltaTable = DeltaTable.forPath(spark, delta_path)

# Update the table (reduce price of accessories by 10%)
deltaTable.update(
    condition = "Category == 'Accessories'",
    set = { "Price": "Price * 0.9" })
```

*time travel*

```sql
%%sql

DESCRIBE HISTORY products

DESCRIBE HISTORY 'Files/mytable'
```

or

```python
df = spark.read.format("delta").option("versionAsOf", 0).load(delta_path)
```

## Streaming Data

Spark Structured 

Streaming with delta tables

streaming source: A stream is created that reads data from the table folder as new data is appended.

```python
from pyspark.sql.types import *
from pyspark.sql.functions import *

# Load a streaming dataframe from the Delta Table
stream_df = spark.readStream.format("delta") \
    .option("ignoreChanges", "true") \
    .load("Files/delta/internetorders")

# Now you can process the streaming data in the dataframe
# for example, show it:
stream_df.show()
```

streaming sink: New data is added to the stream whenever a file is added to the folder.

```python
from pyspark.sql.types import *
from pyspark.sql.functions import *

# Create a stream that reads JSON data from a folder
inputPath = 'Files/streamingdata/'
jsonSchema = StructType([
    StructField("device", StringType(), False),
    StructField("status", StringType(), False)
])
stream_df = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

# Write the stream to a delta table
table_path = 'Files/delta/devicetable'
checkpoint_path = 'Files/delta/checkpoint'
delta_stream = stream_df.writeStream.format("delta").option("checkpointLocation", checkpoint_path).start(table_path)
```

```sql
%%sql

CREATE TABLE DeviceTable
USING DELTA
LOCATION 'Files/delta/devicetable';

SELECT device, status
FROM DeviceTable;
```

# Data Factory

## Pipeline templates

![alt text](/images/image-3.png)

## Monitor Pipelines
**Validate**
![alt text](/images/image-4.png)

History
![alt text](/images/image-5.png)

# Dataflows Gen2

## Dataflow

Dataflows are a type of cloud-based ETL (Extract, Transform, Load) tool for building and executing scalable data transformation processes.

## Dataflow Gen2

ataflows Gen2 is to provide an easy, reusable way to perform ETL tasks using Power Query Online.

Benefits:

- **Extend data with consistent data**, such as a standard date dimension table.
- **Allow** self-service users **access** to a subset of data warehouse separately.
- **Optimize performance** with dataflows, which enable extracting data once for reuse, reducing data refresh time for slower sources.
- **Simplify data source** complexity by only exposing dataflows to larger analyst groups.
- Ensure **consistency** and **quality** of data by enabling users to clean and transform data before loading it to a destination.
- **Simplify data integration** by providing a low-code interface that ingests data from various sources.

Limitations:

- Not a replacement for a data warehouse.
- Row-level security isn't supported.
- Fabric capacity workspace is required.

![alt text](/images/image-6.png)

1. Power Query ribbon

- Filter and Sort rows
- Pivot and Unpivot
- Merge and Append queries
- Split and Conditional split
- Replace values and Remove duplicates
- Add, Rename, Reorder, or Delete columns
- Rank and Percentage calculator
- Top N and Bottom N

2. Queries pane

The Queries pane shows you the different data sources (**queries**). Rename, duplicate, reference, and enable staging are some of the options available.

1. Diagram view

The Diagram View allows you to visually see how the data sources are connected and the different applied transformations.

4. Data Preview pane

The Data Preview pane only shows a subset of data to allow you to see which transformations you should make and how they affect the data.

5. Query Settings pane

The Query Settings pane primarily includes Applied Steps. Each transformation you do is tied to a step, some of which are automatically applied when you connect the data source. Depending on the complexity of the transformations, you may have several applied steps for each query.

## Integrate Dataflows Gen2 and Pipelines

Use a dataflow for data ingestion and transformation, and landing into a Lakehouse using dataflows. Then incorporate the dataflow into a pipeline to orchestrate extra activities.

# Ingest data with Spark and Fabric notebooks

## Connect to data

By default, Fabric notebooks use PySpark

## Connect to external sources

```python
# Azure Blob Storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "nyctlc"
blob_relative_path = "yellow"
blob_sas_token = "sv=2022-11-02&ss=bfqt&srt=c&sp=rwdlacupiytfx&se=2023-09-08T23:50:02Z&st=2023-09-08T15:50:02Z&spr=https&sig=abcdefg123456" 

# Construct the path for connection
wasbs_path = f'wasbs://{blob_container_name}@{blob_account_name}.blob.core.windows.net/{blob_relative_path}?{blob_sas_token}'

# Read parquet data from Azure Blob Storage path
blob_df = spark.read.parquet(wasbs_path)

# Show the Azure Blob DataFrame
blob_df.show()
```

## Configure alternate authentication

```python
# Placeholders for Azure SQL Database connection info
server_name = "your_server_name.database.windows.net"
port_number = 1433  # Default port number for SQL Server
database_name = "your_database_name"
table_name = "YourTableName" # Database table
client_id = "YOUR_CLIENT_ID"  # Service principal client ID
client_secret = "YOUR_CLIENT_SECRET"  # Service principal client secret
tenant_id = "YOUR_TENANT_ID"  # Azure Active Directory tenant ID


# Build the Azure SQL Database JDBC URL with Service Principal (Active Directory Integrated)
jdbc_url = f"jdbc:sqlserver://{server_name}:{port_number};database={database_name};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;Authentication=ActiveDirectoryIntegrated"

# Properties for the JDBC connection
properties = {
    "user": client_id, 
    "password": client_secret,  
    "driver": "com.microsoft.sqlserver.jdbc.SQLServerDriver",
    "tenantId": tenant_id  
}

# Read entire table from Azure SQL Database using AAD Integrated authentication
sql_df = spark.read.jdbc(url=jdbc_url, table=table_name, properties=properties)

# Show the Azure SQL DataFrame
sql_df.show()
```

## Write data into a lakehouse

### Write to a file

```python
# Write DataFrame to Parquet file format
parquet_output_path = "your_folder/your_file_name"
df.write.mode("overwrite").parquet(parquet_output_path)
print(f"DataFrame has been written to Parquet file: {parquet_output_path}")

# Write DataFrame to Delta table
delta_table_name = "your_delta_table_name"
df.write.format("delta").mode("overwrite").saveAsTable(delta_table_name)
print(f"DataFrame has been written to Delta table: {delta_table_name}")
```

### Write to a Delta table 

```python
# Use format and save to load as a Delta table
table_name = "nyctaxi_raw"
filtered_df.write.mode("overwrite").format("delta").save(f"Tables/{table_name}")

# Confirm load as Delta table
print(f"Spark DataFrame saved to Delta table: {table_name}")
```

### Optimize Delta table writes

**V-Order** enables *faster* and *more efficient* reads by various compute engines, such as Power BI, SQL, and Spark.V-order applies special sorting, distribution, encoding, and compression on parquet files at write-time.

**Optimize write** improves the *performance* and *reliability* by **reducing** the number of files written and increasing their size. It's useful for scenarios where the Delta tables have suboptimal or nonstandard file sizes, or where the extra write latency is tolerable.



```python
**# Enable V-Order 
spark.conf.set("spark.sql.parquet.vorder.enabled", "true")

# Enable automatic Delta optimized write
spark.conf.set("spark.microsoft.delta.optimizeWrite.enabled", "true"*)*
``` 

## Consider uses for ingested data

When you load data, it's a good idea to do some **basic cleaning** like removing duplicates, handling errors, converting null values, and getting rid of empty entries to ensure data quality and consistency.

Data scientists usually prefer fewer changes so they can explore wide tables. They would likely want access to the raw ingested data. Fabric's Data Wrangler then let's them explore the data and generate transformation code for their specific needs.

Whereas Power BI data analysts may require more transformation and modeling before they can use the data. While Power BI can transform data, starting with well-prepared data allows analysts to develop reports and insights more efficiently.

# Medallion architecture design

## Introduction

The medallion architecture brings **structure** and **efficiency** to your lakehouse environment.

## Medallion architecture

![alt text](/images/image-7.png)

Bronze Layer: Raw Data

Silver Layer: Validate and refine data (remove nulls, deduplicate...)

Gold Layer: aggregate data, enrich with external information

Data transformation:

- Altering structure/content
- Use dataflow/notebook (Notebooks are a more suitable tool for data transformation with big data, Dataflow suitable for small semantic model)

Data orchestation:

- Coordination/management
- Pipelines: series of steps

## Implementation

- Set up the foundation
- Design architecture
![alt text](/images/image-8.png)
- Ingest data into bronze
- Transform data and load to silver
- Generate gold layer
  - using dimensional model
  - can have multiple gold layers for different audiences/domains
  - Data Warehouse as gold layer
- Enable downstream consumption

## Query and report data

The SQL analytics endpoint in Fabric enables you to write queries, manage the semantic model, and query data using the new visual query experience.

Tailoring medallion layers to different needs allows you to optimize data processing and access for specific use cases.

## Manage lakehouse

### Secure

Secure your lakehouse by ensuring that only authorized users can access data. In Fabric, you can do this by setting permissions at the **workspace** or **item** level.

- Security and Access Considerations: Define who needs access at each layer - interact with sensitive data.
- Gold Layer Access Control: emphasizing the importance of minimal permissions.
- Silver Layer Utilization: balancing flexibility and security.
- Bronze Layer Access Control: emphasizing the importance of minimal permissions.

### CI/CD

Git integration in Fabric.

CI/CD is crucial at the gold layer of a lakehouse because it ensures that high-quality, validated, and reliable data is available for consumption.

-> enhances data accuracy, accelerates decision-making, and supports data-driven initiatives effectively.

# Data warehouses in Fabric

## Introduction

Fabric's datawarehouse:

- centralizes/organize data to unified view
- provies full SQL semantics
- desgined to use by the whole team

## Fundamental

- Data ingestion
- Data storage
- Data processing
- Data analysis and delivery

Support T-SQL capabilities, fully managed, scalable

DE build relational layer top of data - DA use T-SQL and Power BI to explore the data.

### Design data warehouse

Tables organized in a schema that is optimized for a multidimensional modeling

#### Tables

Fact tables

- numerical data
- large number of rows
- primary source of analysis

Dimensions tables

- descriptive information
- small number of rows
- provide context for the data in fact table
- have *surrogate key* - number key and *alternate key* - specific instance of a entity (customer id)

Types:

- time dimensions: enables DA to aggregate data over temporal intervals
- slowly changing dimensions: enable users to analyze and understand changes to data overtime

#### Data warehouse schema designs

dimension data is *de-normalized* to **reduce duplication**

Normally: start schema

![alt text](/images/image-9.png)

Lots of levels/information shared by different things: snowflake schema

![alt text](/images/image-10.png)

## Understand data warehouse

Lakehouse: collection of files, folders, tables, shortcuts...

Fabric: lakeview -> Lakehouse (read tables, use sql analytics endpoint)

### Describe a data warehouse

Create your data warehouse directly in Fabric from the **create hub** or within a **workspace**.

### Ingest data

#### Create tables

```sql
COPY INTO dbo.Region 
FROM 'https://mystorageaccountxxx.blob.core.windows.net/private/Region.csv' WITH ( 
            FILE_TYPE = 'CSV'
            ,CREDENTIAL = ( 
                IDENTITY = 'Shared Access Signature'
                , SECRET = 'xxx'
                )
            ,FIRSTROW = 2
            )
GO
```

#### Table considerations

Staging tables: temporary tables used for data cleansing, data transformations, data validation

Data warehouse tasks in order:

- Ingest new data to data lake
- Load data from files to staging tables
- Load dimension tables from dimension data
- Load fact tables
- Perform post-load optimization

## Query and transform data

### SQL query editor

