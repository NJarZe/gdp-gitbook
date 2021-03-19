---
description: >-
  The storage layer contains all the logical elements which store data. Each of
  the logical element is briefly explained below.
---

# Data Storage Layers

## Source Systems

A source system is any relational database or file which contains useful information for Aliaxis and needs to be loaded into the GDP. The source system can be any on-prem as well as cloud data store.

## Landing Zone

Landing zone is an area on Azure Data Lake Store Gen 2 \(ADLS\) responsible for receiving all data required to be ingested to the GDP. ADLS is a data store which can store all types of text or binary files. It is highly scalable and can store terabytes of data without performance degradation. 

Some features that ADLS has over regular Azure Blob Storage are:

1. Hadoop Distributed File System \(HDFS\) API support which means that all big data processing tools like Apache Spark can connect to it and process data.
2. Hierarchical File Structure allowing faster file system operations. e.g., renaming or access control
3. Faster analytical processing performance

The landing zone is intended as:

* The storage zone for the raw ingested data
* An archive allowing reprocessing if so required

{% hint style="info" %}
**Note:** Each data that gets ingested in GDP needs to land in this zone first.
{% endhint %}

This zone has the following properties:

* Data is stored in its original format without modification.
* Data is arranged with respect to the date \(for daily ingests\), hour \(for hourly ingestion\), week or month of ingestion based on the ingestion frequency. The date format to use is **YYYY-MM-DD**.
* All data files must include a header in their first row. This header must contain the field names.
* There is no obligation to map metadata to the data but if required for technical debugging, metadata can be mapped.
* Such metadata is not be exposed to the business users. Metadata is attached to the ODS.
* Only technical users or data engineers have access to this zone.
* It acts as an archive of the data and if the downstream data products are lost, these can be recreated from the data in the landing zone.

## Operational Data Store \(ODS\)

An operational data store \(ODS\) is a central database that provides a snapshot of the latest data from multiple transactional systems for operational reporting. It enables organizations to combine data in its original format from various sources into a single destination to make it available for business reporting.

ODS has following properties:

* Metadata is mapped to the data available in ODS.
* Quality checks are performed on this zone to make sure that the data is consistent with the source
* The data in this zone is supposed to be a replica of the data in the source systems or a view of the source data e.g., SCD2, SCD3 etc. and should not be a new dataset created based on multiple sources.

## Staging \(STG\)

The staging layer contains prepared data ready to be loaded into a data warehouse/data mart. The schema of a staging table is similar to its corresponding target table in the data warehouse.

## Data Warehouse \(DWH\)

The term DWH here represents an EDW or a data mart. As we follow Kimball’s approach, first data marts will be built. Reporting/BI tools connect to the DWH directly.

