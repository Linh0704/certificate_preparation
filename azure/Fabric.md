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

In Microsoft Fabric, each workspace is assigned a Spark cluster. An administrator can manage settings for the Spark cluster in the Data Engineering/Science section of the workspace settings.\

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
```
# Load a file into a dataframe
df = spark.read.load('Files/mydata.csv', format='csv', header=True)

# Save the dataframe as a delta table
df.write.format("delta").saveAsTable("mytable")
```