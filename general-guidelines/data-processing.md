---
description: Different elements of data processing are explained below
---

# Data Processing

## Ingestion Tool

The ingestion tool \(also called the loading tool\) is responsible for extracting data from the source systems \(on-prem/cloud\) and loading/ingesting it to the GDP. Many tools exist in the market, e.g., Azure Data Factory, Talend, Rivery, etc.

### **Azure Data Factory**

Azure Data Factory \(ADF\) is the recommended ingestion tool. ADF can not only act as an ingestion tool but also as an orchestration tool connecting different pieces of a pipeline together.

The image below shows the high-level architecture of the GDP data processing pipelines with technologies, with ADF in the orchestration layer. ADF acts as an orchestrator and can sequentially execute the copy activity followed by the Snowflake Pull.

Ingestion will be performed using the Copy activity of ADF, which has several connectors for connecting with a diverse set of data stores. 

In the image below, ADF copy activity extracts the data from the source systems and copies it to the landing zone. Once the data is copied, Snowflake pull is triggered by ADF. The pull can be a copy command or a Snowpipe.

![Architecture with Processing Technologies](../.gitbook/assets/data-ingestion-process.png)

### Metadata Table

All Data Factory pipelines that ingest data from source to the ODS layer will use a metadata table that exists in the ODS database, under the schema meta\_lan. 

The structure of the metadata table is as follows

| Field Name | Description | Sample Value\(s\) |
| :--- | :--- | :--- |
| ETL\_SOURCE\_NAME | Coded name for the ETL source | crm, ax\_germany |
| SOURCE\_SCHEMA\_NAME | Schema where source table is stored | dbo |
| SOURCE\_TABLE\_NAME | Name of the table to be ingested | alaccount, salestable |
| SOURCE\_COLUMNS | List of column names \(separated by comma\), or \* \(for selecting all columns | code, name, description |
| SNOWFLAKE\_DATABASE\_NAME | Destination database \(in Snowflake\) | emea\_dev\_ods |
| SNOWFLAKE\_SCHEMA\_NAME | Destination schema \(in Snowflake\). Usually it is similar to the source name | ax\_uk, crm |
| SNOWFLAKE\_TABLE\_NAME | Destination table \(in Snowflake\). Usually it is the same table name as the source table | alaccount, salestable |
| DIVISION\_NAME | Acronym for the division. Used to create the ADLS path where files are stored | EMEA, APAC |
| ENVIRONMENT\_NAME | Acronym for the environment \(DEV, TST, PRD\). Used to create the ADLS path where files are stored | DEV |
| ADLS\_CONTAINER\_NAME | Name of the ADLS container | gdp-dev, gdp-prd |
| ADLS\_DATA\_AREA\_NAME | Area in ADLS where the data is located | lan, sda |
| ADLS\_SOURCE\_NAME | Name of the source in ADLS | crm, ax\_uk |
| ADLS\_SCHEMA\_NAME | Name of the schema folder in ADLS | dbo |
| ADLS\_TABLE\_NAME | Name of the table folder in ADLS | alaccount |
| SNOWFLAKE\_FILE\_FORMAT | File format name used to load the files into Snowflake | sql\_ax2012 |
| MAX\_ROWS\_PER\_FILE | Maximum number of rows one single file can contain before creating a new file | 1000000 |
| TRUNCATE\_TARGET\_TABLE | Boolean \(true, false\) indicating if the target table should be truncated or not | true |
| IS\_ACTIVE | Boolean \(true, false\) indicating if this configuration row should be imported or not | true |

### Snowflake Pull

There are two ways how Snowflake can pull data into Snowflake. 

1. Copy command for bulk loading of data
2. Snowpipe for continuous loading of data

Snowpipe is an automated way of ingesting data into ODS, but a Snowpipe needs to be configured for each table that will be ingested. 

#### **Copy Command**

A generic \(and already existing\) stored procedure containing the copy command will be used to ingest data into Snowflake; ADF will call the stored procedure using a Lookup activity. As ADF already knows which files were copied to ADLS \(via the metadata table\), the list of the files is passed to the stored procedure which ingests the data using the copy command. 

In case of failures, the error is passed back to ADF and can be viewed in ADF. This provides a single pane of glass for data loading.

#### **Snowpipe Auto-Ingest**

A Snowpipe can be created with auto-ingest enabled. An integration with Azure event hub needs to be created for each snow-pipe and a Snowpipe is needed for each table. This requires some additional effort for the configuration of ADLS notifications push to Azure Event Hub and then consumption of those notifications by Snowflake. 

In addition, the loading errors are visible in Snowflake \(not in ADF\). Thus, for complete visibility of data ingestion, both components \(ADF and Snowflake\) need to be logged into. ADF can be used for an automated creation of the Snowpipe per table. The following image shows the process flow of an Auto-Ingest Snowpipe. 

![Snowpipe Auto-ingest Process Flow](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image10.png) Figure 9 Snowpipe Auto-Ingest Process Flow

#### **Snowpipe via REST API**

Snowpipes can be called via a REST APIwhen auto-ingest option is disabled. In this case, ADF can call the Snowpipe right after copying the data into ADLS. 

The image below shows the process of calling the Snowpipe REST endpoint for ingestion of data loaded into ADLS. There are two steps for data ingestion using this approach. 

* Step 1: Loading data into ADLS. 
* Step 2: Calling the REST endpoint of the corresponding Snowpipe with the paths of files that need to be ingested. The paths of these files are inserted in the ingest queue and the files are ingested by the Snowpipe using the COPY command. 

Both the steps can be orchestrated using ADF.

![Snowpipe Process Flow](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image11.png) Figure 10 Calling a Snowpipe via its REST Endpoint

## General Principles for Data Ingestion

Loading of data from on-prem source systems to Snowflake should follow the following principles:

1. Data ingestion function should be centrally managed in BICC:
   1. To ensure standardization of data loading process.
   2. For cost reduction as tool related skills not required to developed locally.
   3. For central monitoring
   4. For handling runs, reruns, failures and error resolution
   5. To ensure data consistency
2. Data ingestion system should be cloud based
   1. For central management & monitoring
   2. Standardization across countries
   3. No Capex
   4. Scalability
   5. Easier management
3. Data ingestion system should be automated. Once a source system is connected, it should be able to ingest data with minimal effort.
4. Data ingestion system should be context aware or based on templates supporting environment variables. The environment variables should be adjusted per the country of source.
5. The system should automatically create technical metadata of the tables ingested.
6. The system should be able to map types between the source systems and snowflake.
7. The system should be able to support the following types of ingestion:
   1. Snapshot in time via bulk loading
   2. Delta based on a column with temporal information \(timestamp/incremental id\)

## Transformation Tool

The transformation tool reads data from the ODS, applies any transformations, cleanups and aggregations, and puts the result in the staging layer \(STG\). 

As of now, all transformations are done using Snowflake stored procedures. 

