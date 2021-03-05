# General Information

### 3.1. Storage Layer

The storage layer contains all the logical elements which store data. Each of the logical element is briefly explained below.

#### 3.1.1. Source Systems

A source system is any relational database or file which contains useful information for Aliaxis and needs to be loaded into the GDP. The source system can be any on-prem as well as cloud data store.

#### 3.1.2. Landing Zone

Landing zone is an area on Azure Data Lake Store Gen 2 \(ADLS\) responsible for receiving all data required to be ingested to the GDP. ADLS is a binary data store which can store all types of text or binary files. It is highly scalable and can store terabytes of data without performance degradation. Instead of ADLS, Azure Blob Storage can also be used but ADLS has the following most important advantages over the Blob Storage:

1. Hadoop Distributed File System \(HDFS\) API support which means that all big data processing tools like Apache Spark can connect to it and process data.
2. Hierarchical File Structure allowing faster file system operations. e.g., renaming or access control
3. Faster analytical processing performance

Support for ADLS V2 has recently been added by Snowflake and is in public preview at the time of writing of this document. More details can be found [here](https://www.snowflake.com/blog/snowflake-support-for-microsofts-adls-gen2-public-preview/).

The landing zone is intended as:

* The storage zone for the raw ingested data
* An archive allowing reprocessing if so required

Each data that gets ingested in GDP first lands in this zone. This zone has the following properties:

* Data is stored in its original format without modification.
* Data is arranged with respect to the date \(for daily ingests\), hour \(for hourly ingestion\), week or month of ingestion based on the ingestion frequency. The recommended date format is YYYY-MM-DD.
* There is no obligation to map metadata to the data but if required for technical debugging, metadata can be mapped.
* Such metadata is not be exposed to the business users. Metadata is attached to the ODS.
* Only technical users or data engineers have access to this zone.
* It acts as an archive of the data and if the downstream data products are lost, these can be recreated from the data in the landing zone.

#### 3.1.3. Operational Data Store \(ODS\)

An operational data store \(ODS\) is a central database that provides a snapshot of the latest data from multiple transactional systems for operational reporting. It enables organizations to combine data in its original format from various [sources](https://www.stitchdata.com/integrations/sources/) into a single [destination](https://www.stitchdata.com/integrations/destinations/) to make it available for business reporting.

ODS has its own staging area/raw zone which stores intermediate data arriving from the landing zone. For instance, when delta is received for a table from the landing zone, the delta needs to be applied to the table in the ODS to get its current state. If a snapshot is received, it can directly be loaded to ODS following the right naming conventions.

The reporting tools can be connected directly to ODS for operational reporting but not to the raw zone of the ODS. Transformation tools creating data marts can access both the raw zone as well as the ODS.

ODS has following properties:

* Metadata is mapped to the data available in ODS.
* ODS is exposed for querying to the business users as well as the use-case applications for ad hoc/batch queries.
* Quality checks are performed on this zone to make sure that the data is consistent with the source
* The data in this zone is supposed to be a replica of the data in the source systems or a view of the source data e.g., SCD2, SCD3 etc. and should not be a new dataset created based on multiple sources.

#### 3.1.4. Staging \(STG\)

The staging layer contains prepared data ready to be loaded into a data warehouse/data mart. The schema of a staging table is similar to its corresponding target table in the data warehouse.

#### 3.1.5. Data Warehouse \(DWH\)

The term DWH here represents an EDW or a data mart. As we follow Kimball’s approach, first data marts will be built. Reporting/BI tools connect to the DWH directly.

### 3.2. Processing Layer

In the following, different elements of the processing layer are briefly explained.

#### 3.2.1. Ingestion Tool

The ingestion tool \(also called the loading tool\) is responsible for extracting data from the source systems \(on-prem/cloud\) and loading/ingesting it to the GDP. Many tools exist in the market, e.g., Azure Data Factory, Talend, Rivery etc.

**3.2.1.1. Azure Data Factory**

As the choice of the tools \(ingestion/ETL\) is out of scope, we only provide high-level principles and recommendations regarding the choice. For the purpose of completeness of this document, we will take Azure Data Factory \(ADF\) as the ingestion tool. ADF can not only act as an ingestion tool but also as an orchestration tool connecting different pieces of a pipeline together.

Figure 2 shows the high-level architecture of the GDP data processing pipelines with technologies. ADF has been added in the orchestration layer. The ingestion is performed by ADF Copy activity which has several connectors for connecting with a diverse set of data stores. A list of supported data sources can be found [here](https://docs.microsoft.com/en-us/azure/data-factory/connector-overview). Note that ADF supports SAP Business Warehouse, SAP HANA and SAP Table as data sources. In Figure 8, ADF copy activity extracts the data from the source systems and copies it to the landing zone. Once the data is copied, Snowflake pull is triggered by ADF. The pull can be a copy command or a Snowpipe. Snowflake now supports auto-ingest Snowpipe for loading data based on file notifications via Azure Event Grid.

ADF acts as an orchestrator and can sequentially execute the copy activity followed by the Snowflake Pull.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image9.png) Figure 8 High Level Architecture with Processing Technologies

#### 3.2.2. Snowflake Pull

There are two ways how Snowflake can pull data into Snowflake. 1. Copy command for bulk loading of data 2. Snowpipe for continuous loading of data

Apparently Snowpipe is a more attractive option compared to the Copy command but a Snowpipe needs to be configured for each table. In the following, we briefly explain how the Snowflake pull can be configured with ADF

**3.2.2.1. Copy Command**

A stored procedure containing the copy command can be created. ADF can call the stored procedure using the .NET API running in an Azure function. As ADF already knows which files were copied to ADLS, the list of the files is passed to the stored procedure which ingests the data using the copy command. In case of failures, the error is passed back to ADF and can be viewed in ADF. This provides a single pane of glass for data loading.

Creation of file formats for new tables to be ingested can be automated before ingestion into Snowflake.

**3.2.2.2. Snowpipe Auto-Ingest**

A Snowpipe can be created with auto-ingest enabled. An integration with Azure event hub needs to be created for each snow-pipe and a Snowpipe is needed for each table. This requires some additional effort for the configuration of ADLS notifications push to Azure Event Hub and then consumption of those notifications by Snowflake. Also, the loading errors are visible in Snowflake \(not in ADF\). Thus, for complete visibility of data ingestion, both the platforms \(ADF and Snowflake\) need to logged into. ADF can be used for an automated creation of the Snowpipe per table. Figure 9 shows the process flow of an Auto-Ingest Snowpipe. More details about creation of an automated Snowpipe can be found [here](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-auto-azure.html).

![Snowpipe Auto-ingest Process Flow](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image10.png) Figure 9 Snowpipe Auto-Ingest Process Flow

**3.2.2.3. Snowpipe via REST API**

Snowpipes can be called via a REST api when auto-ingest option is disabled. In this case, ADF can call the Snowpipe right after copying the data into ADLS. Figure 10 show the process of calling the Snowpipe REST endpoint for ingestion of data loaded into a stage which is ADLS in our case. As shown, there are two steps for data ingestion using this approach. Step 1 involves loading data into ADLS. Step 2 involves calling the REST endpoint of the corresponding Snowpipe with the paths of files that need to be ingested. The paths of these files are inserted in the ingest queue and the files are ingested by the Snowpipe using the COPY command. Both the steps can be orchestrated using ADF.

![Snowpipe Process Flow](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image11.png) Figure 10 Calling a Snowpipe via its REST Endpoint

#### 3.2.3. Transformation Tool

The transformation tool reads the data from the ODS or the raw zone of the ODS and after transformation/integration of data, puts the result in the staging area of the DWH. The choice of the transformation tool is out of scope of this exercise but for the purpose of completeness, we take SnowSQL has the transformation tool.

#### 3.2.4. DWH Loading Framework

The process of loading the data from the staging Dimension/Fact tables to the target Dimension/Fact tables can be automated via a framework. Keyrus has such a framework for SQL Server Integration Service but can be modified to work with Snowflake. If the framework is not used, SnowSQL procedures can be used to do the same. The process of reading the data from ODS/Raw zone, writing the transformed/integrated data into staging tables and loading the data from the staging tables to the DWH can be orchestrated by ADF.

### 3.3. Spectrum of Responsibility

Figure 11 shows the spectrum of responsibility. Loading/Ingestion of the data into the GDP and making it available for the local teams to consume is recommended to be the responsibility of the local teams. Reading the data from ODS/Raw zone and delivering the Data Marts is the responsibility of the local teams. If the DWH Loading framework is used, the last step of loading the DWH can be automated and can also be centralized at BICC. In any case, BICC will help & support the local teams in the implementation.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image12.png) Figure 11 Spectrum of Responsibility

## 4. Design Rules

### 4.1. Databases & Zones

Figure 12 shows an overview of the database design rules in various storage layers of the GDP.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image13.png) Figure 12 Database Design Overview

#### 4.1.1. Landing Zone

The path layout is constructed as follows:

> /{DIV}/{ENV}/{type}/{zone\_code}/{source\_code}/{schema}/{table}\_{ingestion\_type}/{datetime}

The paths consist of the following parts:

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Item</b>
      </th>
      <th style="text-align:left"><b>Description</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">{DIV}</td>
      <td style="text-align:left">
        <p><b>Description:</b> Aliaxis divisions code. DIV represents scope in the
          GDP</p>
        <p>Aliaxis group will be considered as one division/business entity in the
          GDP and represents everything local to the group.</p>
        <p><b>Length:</b> 2-4 upper case letters</p>
        <p><b>Possible Values:</b> EMEA, APAC, AGRP, CMMN (Common)</p>
        <p><b>Values Description:</b>
        </p>
        <ul>
          <li>EMEA: Europe Middle East &amp; Africa</li>
          <li>APAC: Asia Pacific</li>
          <li>AGRP: Aliaxis Group</li>
          <li>CMMN: Common DIV/scope contains everything central to the GDP and Aliaxis
            containing common schemas/procedures/formats etc.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{ENV}</td>
      <td style="text-align:left">
        <p><b>Description:</b> Code of the environment to which the zone belongs</p>
        <p><b>Length:</b> 3 letters (upper case)</p>
        <p><b>Values:</b>
        </p>
        <ul>
          <li>DEV</li>
          <li>TST</li>
          <li>PRD</li>
        </ul>
        <p><b>Values Description:</b>
        </p>
        <ul>
          <li>DEV: Development Environment</li>
          <li>TST: Test Environment</li>
          <li>PRD: Production Environment</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{type}</td>
      <td style="text-align:left">
        <p><b>Description:</b> Type of data the directory stores.</p>
        <p><b>Length:</b> 4 letters lower case</p>
        <p><b>Values:</b>
        </p>
        <ul>
          <li>data</li>
          <li>exec</li>
        </ul>
        <p><b>Values Description:</b>
        </p>
        <ul>
          <li>data: the actual data ingested from source systems or newly created entities</li>
          <li>exec: for execution related data e.g., ETL pipelines metadata, logs etc.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{zone_code}</td>
      <td style="text-align:left">
        <p><b>Description:</b> A zone qualifies the layer a data entity belongs to
          and its properties. For example, data that is ingested in the data platform
          is stored in the landing zone.</p>
        <p><b>Length:</b> 3 letter code (lower case)</p>
        <p><b>Values:</b>
        </p>
        <ul>
          <li>lan</li>
          <li>ods</li>
          <li>sda</li>
          <li>cda</li>
          <li>scr</li>
          <li>exe</li>
          <li>adm</li>
        </ul>
        <p><b>Values Description</b>
        </p>
        <ul>
          <li>lan: stands for landing zone. Data ingested in the data platform is first
            stored in the landing zone in its original format.</li>
          <li>ods: stands for operational data store.</li>
          <li>sda: stands for Specific Data Area</li>
          <li>cda: stands for Global Data Area</li>
          <li>scr: stands for Scratch</li>
          <li>adm: stands for administration</li>
        </ul>
        <p>Zones other than landing are used for advance analytics which is out of
          scope of the current GDP. Zones mentioned here for the sake of completeness
          and further extension of this document in future.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{source_code}</td>
      <td style="text-align:left">
        <p><b>Description:</b> A code is allocated to each source system of data.</p>
        <p>For example, in an organization an instance of an oracle database could
          be commonly referred to as &#x201C;ORA&#x201D;. In some organizations,
          a database is categorized as an application. If this is the case, the code
          used in the Master Application List (global list of applications running
          in an organization). There can be dozens of types of sources for the data
          platform. When a source is onboarded, the BICC works on the naming convention
          to make sure that it is correct.</p>
        <p><b>Length:</b> No limitation. The exact name of the source system used
          by the local teams to refer to the source system should be used. The vaue
          should be in lower case.</p>
        <p><b>Values:</b> No fixed values. If Sharepoint is the source system and
          it is called sharepoint, the value should be &#x201C;sharepoint&#x201D;.</p>
        <p><b>Values Description:</b> NA</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{schema}</td>
      <td style="text-align:left">
        <p><b>Description:</b> When dealing with relational databases, schema is a
          logical collection of tables. For example, a DB2 or Oracle instance can
          have multiple schemas/databases which contain multiple tables. In the case
          of non-relational databases, it is a collection of files related to an
          application.</p>
        <p><b>Length:</b> Actual name length of the schema. Use small caps for database
          and table names</p>
        <p><b>Values:</b> The name of the schema in the source system.</p>
        <p><b>Values Description:</b> NA</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{table}</td>
      <td style="text-align:left">
        <p><b>Description:</b> The name of the table in the schema. For files, it
          is the logical name of the concept the file represents. If there is an
          excel file manually exported from a source system and contains sales figures,
          the corresponding table could be named as &#x201C;sales_figures&#x201D;.</p>
        <p><b>Length:</b> no limitation. Lower case should be used.</p>
        <p><b>Values:</b> NA</p>
        <p><b>Values Description:</b> NA</p>
        <p>Snapshot</p>
        <p>Delta</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{ingestion_type}</td>
      <td style="text-align:left">
        <p><b>Description:</b> The type of ingestion.</p>
        <p><b>Length:</b> 1 lower case letter</p>
        <p><b>Values:</b> d, s</p>
        <p><b>Values Description:</b>
        </p>
        <ul>
          <li>d: Delta ingestion (only changed records are ingested)</li>
          <li>s: Snapshot ingestion (each time, the whole snapshot of the table is ingested.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{datetime}</td>
      <td style="text-align:left">
        <p><b>Description:</b> The date when the data was ingested.</p>
        <p><b>Length:</b> 10. YYYY-MM-DD</p>
        <p><b>Values:</b> NA</p>
        <p><b>Values Description:</b> NA</p>
      </td>
    </tr>
  </tbody>
</table>

#### 4.1.2. ODS

An ODS is a database in Snowflake which contains the latest snapshot of the source systems. Following naming convention should be used for the ODS database in Snowflake:

> **{DIV}\_{ENV}\_ODS**

Inside the ODS database, each ingested schema from the source systems should be represented as:

> **{DIV}\_{ENV}\_ODS.{SOURCE\_CODE}\_{SCHEMA}**

Formatting considerations:

* Upper case should be used
* Inside each schema, original table names should be used.

#### 4.1.3. Landing Metadata Schema

A stage schema should be created in the ODS of the DIV with the following naming convention:

> **{DIV}\_{ENV}\_ODS.META\_LAN**

For example, for EMEA production environment, the name of the stage schema would be:

> **EMEA\_PRD\_ODS.META\_LAN**

This schema should contain all the file formats used for the loading of files from ADLS/Blob Storage. These file formats are used by the Snowpipes and the COPY commands to copy data to ODS.

#### 4.1.4. ODS

ODS contains the latest snapshot of the source systems but the data ingested might not be in the format to be loaded to ODS as-is. Two types of datasets land in ODS from the landing zone:

1. Deltas
2. Snapshots

**4.1.4.1. Snapshots**

Snapshots can be directly loaded into ODS in the following ways:

1. Replacing the existing table in ODS with the new snapshot from the landing zone.
2. Appending the new snapshot records with a new date column

Option 2 is recommended. All queries to the tables should contains the date column in the where clause.

**4.1.4.2. Deltas**

For deltas, changes need to be applied to the ODS in the overwrite mode. If a transformation tool is used other than SnowSQL, the data can be read by the tool and changes can be applied to the ODS. If SnowSQL is to be used, the changes can be applied in two ways:

1. Creation of external tables on the data in the landing zone and using SnowSQL to insert data into ODS. External tables allow SnowSQL to query data in external storage systems like ADLS/Blob Storage.
2. Loading data to a staging area in Snowflake and then using SnowSQL to read the staging tables and insert data in ODS. A {DIV}\_{ENV}\_ODS\_STG database with the same naming convention as ODS should be created to hold the staging schemas and tables.

Option 2 is recommended.

**4.1.4.3. Rules**

Following rules apply.

* One ODS DB per division and per environment \(DEV/TST/PRD\)
* The following name convention applies: {DIV}\_{ENV}\_ODS
  * Example: EMEA\_DEV\_ODS for ODS in DEV environment of EMEA
* Data Schemas structure:
  * One schema per source system schema:
  * Naming convention: {source code}\_{schema name}
* Tables structures:
  * All tables should be named with the same name as the original table with a suffix describing the type of load i.e., {table\_name}\_{load\_type}
    * load\_type can be SS for snapshots and DL for deltas
    * Example: SALES\_DL or SALES\_SS
  * Tables will have the following columns:
    * All the columns from the source file/source table
    * Name of the source file \(varchar\)
    * Column that keeps the insertion timestamp :sf\_timestamp \(TIMESTAMP\_NTZ\) in UTC time zone
    * Semi structured data loaded into ODS will have the following columns:
      * One column for the unstructured data \(variant type\)
      * Name of the source file \(varchar\)
      * Column that keeps the insertion timestamp :sf\_timestamp \(TIMESTAMP\_NTZ\) in UTC time zone

#### 4.1.5. STG

* Preparation of the incremental data to be loaded into the DWH
* Loading method is Overwrite
* The schema structure is identical as the DWH schema
* Target table of the staging area which should be in the same structure as the DWH-table
* In general, the incremental process of this layer will be based on the sf\_timestamp column, but in some cases additional logic needs to be defined.
* The schemas in the STG layer should be defined as transient schemas.

#### 4.1.6. DWH

* Each schema in this layer represents a business world \(e.g. Sales, Marketing etc.\)
* Loading method for this layer is merge
* Front end tools \(like Power BI\) should connect only to this layer or the ODS layer.
* The DWH DB also contains a “common” schema which contains the enterprise data model \(DIV scope\) and common entities reusable across different data marts. During the creation of a DataMart, the data architect should identify entities that can be reused in existing/future data marts. These entities should be moved to the common schema.
* If there are country-specific tables that belong in the common schema, for example master data that will be used across several projects for a given country, then this table must have a prefix with the company code. For example, DIM\_PRODUCT\_IT001. This table is the dimension table for products, which is specific for Italy. It is placed in the schema COMMON because it can be reused by other projects from the same country. 

#### 4.1.7. SANDBOX

At the moment no Sandbox is foreseen. The entry is present for the sake of completeness.

#### 4.1.8. META

The META database is used for storage of all non-data entities. The META database should follow the following naming convention:

> {DIV}\_{ENV}\_META

Following should be the schemas in the META database. 1. **USP**: This schema stores the User Stored Procedures written by local teams for transformation etc. E.g., EMEA\_PRD\_META.USP.USP\_STG\_DIM\_CUSTOMER for a USP loading data to the staging table of the customer dimension. 2. **CSP**: This schema stores the Common Stored Procedures \(CSPs\) common to all projects with in a division but not to other divisions. 3. **FF**: The File Format \(FF\) schema stores all file formats used within a DIV. For instance, a JSON file format can be stored as: EMEA\_PRD\_META.FF.FF\_JSON. It is recommended to use global file formats as a standard but for loading of files not coming through the ingestion system, local file formats can be created. 4. **MONITORING**: All monitoring information regarding the transformation pipelines is stored in this schema.

#### 4.1.9. COMMON

The common schema contains DIV wide models that can be used across projects. There can be two types of data that should be stored in the common schema.

1. Enterprise data model \(EDM\) within DIV scope
2. Division wide data entities useful across projects/business lines, e.g., Customer

### 4.2. Deployment Scripts/Stored Procedures

The deployment scripts as well as the stored procedures should use {ENV} as a variable. Use of {ENV} as a variable allows the promotion of the code to different environments via deployment scripts without any change to the code.

### 4.3. Naming conventions

#### 4.3.1. General

* By default, use English
* For data sources replicated to the GDP, the field/table names in local language will be replicated but effort should be made that the new entities’ names should be in English
* Never use SQL keywords as the name for objects
* Never use white space
* Names must be readable and meaningful
* Use only letters and underscores.
* No repeating of entity name unless reserved words are used:
  * Customer\_Name, Customer\_City =&gt; Name, City in the Customer dimension
  * Product\_Name, Product\_Category =&gt; Name, Category in the Product dimension
  * Date in the Hourly table =} hourly\_dt

#### 4.3.2. Databases

* All DB’s names must be in capital letters
* For lower environments add \_{ENV} to the DB name.
  * E.g. EMEA\_DEV\_ODS

#### 4.3.3. Tables

* All table names must be in capital letters
* No prefixes for landing, staging and data warehouse are needed because the tables are in different databases/schemas.
* Tables should be called:
  * Operational Data Store \(ODS\):
    * {source system name}
  * Staging \(STG\):
    * DIM\_{name} or FACT\_{name}:
      * {name} = name of dimension or fact in the DWH
      * Target table of the staging area which should be an almost exact copy of the DWH-table
    * When there are local \(country-specific\) projects that will load the same dimension or fact, but without names or language translations that will common for the division, the table must have the company code as a suffix. 

      For example, 

      EMEA as a division has created a customer dimension table DIM\_CUSTOMER. 

      A local project in Spain needs to build reports that will show Spain-specific customers, with a customer hierarchy that doesn’t match the hierarchy used at the divisional level. 

      In this case, Spain will create a customer dimension table DIM\_CUSTOMER\_ES001, that will hold the customer information that is specific for the Spanish local projects.

    * TMP\_{name} for intermediate staging tables if necessary
  * Data Warehouse \(DWH\):
    * DIM\_{dimension name}
    * FACT\_{fact name} for aggregate tables, add suffix ‘\_AGG’
    * BRIDGE\_{bridge name}
    * Proper use of prefixing in the data warehouse database should only include the above prefixes so that everyone can recognize the data.

#### 4.3.4. Fields

* Use whole words where possible \(abbreviations should be avoided, acronyms and common usage excepted\)
* Never use reserved keywords [https://docs.snowflake.com/en/sql-reference/reserved-keywords.html](https://docs.snowflake.com/en/sql-reference/reserved-keywords.html)
* List of accepted abbreviations. Other abbreviations should be registered in the knowledge base \(e.g., this document\) before usage.
  * ID = Identifier
  * CD = Code
  * DESCR = Description
  * IND = Indicator
  * NBR = Number
  * PCT = Percentage
  * QTY = Quantity
  * AMT = Amount
  * DT = Date
  * SK = Surrogate Key
  * TS = DateTime \(“TimeStamp”\)
  * FK = Foreign Key
  * FSK = Foreign Surrogate Key
* All fields names must be in capital letters
* The surrogate key field should be named {table\_name}\_SK \(only for dimensions\)
* The reference on a fact table to a dimension table should be called {referenced table name}\_FK
* When SCD2 is used on dimensions a first surrogate key must be created. It should be called {table\_name}\_FSK. This is for all versions of the dimensions the same value, it contains the first SK from for that dimension;
* The reference on a fact table to a dimension table should be called {referenced table name}\_FK and also {referenced table name}\_FFK. The \_FK point to the historical correct version of the dimension, the \_FFK points to all dimension records with the same business key but in combination with the T\_Is\_Current field \(on the dimension\) it always gives the current version of the dimension. \(Context: Slowly Changing Dimension\)

#### 4.3.5. Views

* VW\_{description of the view}

#### 4.3.6. Stored procedures

* “USP\_...”

#### 4.3.7. Users

* For users that were created in Snowflake \)in contrast to users that were created in AD\) add prefix of sf\_ before the user name

#### 4.3.8. Data Factory

There should be two data factory instances per DIV. 1. Local Data Factory: This factory is responsible for all orchestration activities within a DIV for local implementations. 2. BICC Data Factory: This factory contains the pipelines used for DIV’s BICC pipelines e.g., data ingestion & Lifecycle management.

The following naming convension should be followed:

* {DIV}\_{ENV}\_DF for local implementations
* {DIV}\_{ENV}\_DF\_BICC for BICC pipelines

### 4.4. Coding conventions

* Never use “select \* …”
* Use aliases for table names and always use these for each column \(in the select, join, where, …\)
* Use the ANSI join, not the old syntax with the join in the “where”-clause.
* Use a column list in an insert statement
* Use columns names in the order clause not the column numbers
* SQL statements should be readable:
  * Use indentation
  * Each statement will begin a new line and use at least on empty line to separate statements
  * Use spaces in expressions E.g. a = 1 not a=1
  * Use parentheses to group conditions in the WHERE clause when using “OR”
* Declare statements should be placed before any other code
* If temporary tables are used, always check their existence before using them. Those tables will not be dropped for debugging reasons.
* Stored procedures should have a coding header containing:
  * Name
  * Parameters \(if used\)
  * Return \(if used\)
  * Description

```text
-- =============================================
-- Name: USP\_test
-- Parameters:
-- @param1: first parameter
-- @param2: second parameter
-- Return: nothing
-- Description: This is an example of a description
-- =============================================
```

* Never use checksum\(\) and binary\_checksum\(\) for change detection because these can give wrong results and are not always 100% reliable.

### 4.5. Design conventions

#### 4.5.1. ODS Staging Tables

* Every table must have audit fields which must be set in the ETL, not with triggers:
  * FILE\_NAME = the file id in which the data originally was presented
  * SF\_TIMESTAMP = creation date and time: Default CURRENT\_TIMESTAMP\(\)

#### 4.5.2. Datawarehouse tables

* Every table in DWH must have audit fields which must be set in the ETL, not with triggers:
  * CREATE\_TIMESTAMP = creation date and time: Default GETDATE\(\)
  * UPDATE\_TIMESTAMP = date and time when the data has been modified: Default GETDATE\(\)
  * SF\_TIMESTAMP –min\(sf\_timestamp\) of all of the table source
* Every dimension table must have a surrogate key column \(more details [here](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dimension-surrogate-key/)\)
  * This can be done by creating an identity column. Identity starts at 1 and increments by 1
* All non-measure columns should be “not nullable”.
* For all foreign keys on a fact table the values of the corresponding business keys will be stored on the table as well.

#### 4.5.3. Staging Tables

* Is an exact copy of the DWH tables

#### 4.5.4. Data types

* Use INT for numbers without a fractional component.
* Use NUMBER\(30,X\) to define numbers with a fractional component. Details [here](https://support.snowflake.net/s/article/To-Float-or-Not-to-Float-Choosing-Correct-Numeric-Data-Type-in-Snowflake).
* TIMESTAMP\_LTZ: All time stamp should be in UTC \(Details [here](https://medium.com/hashmapinc/the-hitchhikers-guide-to-timestamps-in-snowflake-part-1-534e2373fb8e)\). You can define another column for the local timestamp \(e.g., local time of transaction\). TIMESTAMP\_LTZ internally stores UTC time with a specified precision. However, all operations are performed in the current session’s time zone, controlled by the TIMEZONE.
* Use DATE for date columns
* For all fields containing a string, e.g., Varchar or nvarchar, STRING datatype should be used
* Use integers for storing year/month/day values

#### 4.5.5. Data Models

In this section, we cover the data models of the different storage areas in the data platform. The landing zone contains data as files in directories partitioned by the date/time of ingestion. The ODS stores data in the model used by the source systems which mostly 3NF but referential integrities are not enforced. In DWH, for the storage of data marts, dimensional model is used. The choice of snowflake vs star schema depends on the usecase and is determined by the development team. DWH also contains the enterprise data model. When a project is initiated for the EDM, the data architect should determine the suitable storage model.

## 5. Security

### 5.1. General

* Every object has 1 and only 1 role owner
* Privileges on objects are granted to Roles. Roles are then granted to Users/Groups.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image14.png) Figure 13 Snowflake General Control Hierarchy

### 5.2. Aliaxis Roles Hierarchy

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image15.png) Figure 14 Aliaxis Roles Hierarchy

ROLE\_EMEA\_PRD\_{PROJECT-NAME}\_RW/RO =&gt; ODS, STG, DWH

Systems roles that are in IT level:

#### 5.2.1. ACCOUNTADMIN

* A role that encapsulates the SYSADMIN and SECURITYADMIN system-defined roles. It is the top-level role in the system and should be granted only to a limited/controlled number of users in your account.

#### 5.2.2. SECURITYADMIN

* A role that can monitor, and manage users and roles. More specifically, this role is used to: Modify and monitor any user, role, or session.
* Grant permissions for cross-database permissions \(e.g. division analyst would like to get read access to a table located in another division’s DB\)
* Modify any grant, including revoking it.

#### 5.2.3. SYSADMIN

* A role that has privileges to create warehouses and databases \(and other objects\) in an account.
* The only role that can create databases and warehouses 
* SYSADMIN role is the only role that can write into the ODS layer in addition to a technical user created specifically for this purpose.
* SYSADMIN role will grant manage permissions to the lower level roles

#### 5.2.4. DIV Specific Roles

For each division the SECURITYADMIN will create the following roles:

**5.2.4.1. {DIV}\_ADMIN**

* A role that has privileges to create schemas in the STG and DWH layers.
* This role will be responsible to grant permissions to roles inside the division’s STG and DWH DB.
* The role can change warehouse properties

**5.2.4.2. {DIV}\_{ENV}\_RW**

* A role with a writing permission inside the project DB.
* There can be many write roles:
  * {proj}\_etl\_prod
  * {proj}\_etl\_dev
  * {proj}\_dev\_developer

**5.2.4.3. {DIV}\_{ENV}\_RO**

* Read only role on the project data schemas for end users
* There can be many read-only roles.
* Main is to querying business data inside the production environment
* A good practise will be to create read only role per business world \(one for finance, of for marketing etc.\)

| Grant\Role | SYSADMIN | DIV ADMIN | {PRJ}\_RW | PRJ\_RO |
| :--- | :--- | :--- | :--- | :--- |
| Create warehouse | V |  |  |  |
| Change warehouse | V | V |  |  |
| Create Database | V |  |  |  |
| Create Schema \(ODS layer\) | V |  |  |  |
| Create Schema \(STG + DWH layers\) | V | V |  |  |
| Create Tables \(ODS layer\) | V |  |  |  |
| Create Tables \(STG + DWH layers\) | V | V | V |  |
| Use Tables | V | V | V | V |
| Grant to roles in Database level | V | V |  |  |
| Create CSP stored procedures | V |  |  |  |
| Create USP stored procedure | V | V | V |  |

Figure 15 Role Privileges Summary

### 5.3. Data Sharing Between Divisions

In a case that user or process from a specific division needs to select data from other division we recommend to create a new role. The SYSADMIN will be responsible to provide select permissions on the required object to the role.

E.g. A user from EMEA that needs to see data from ODS table which located in the ODS layer of APAC. Here are the required actions:

* Create new role: ROLE\_APAC\_TO\_EMEA\_{PROJECT\_NAME}
* Grant select permission to this role

User from EMEA can join data between EMEA and APAC by using the fully qualified call to the table db\_name.schema\_name.table\_name

### 5.4. Power BI Security

* Workspaces are the lowest level of access control in PBI. Big workspaces containing datasets/dashboards for multiple business lines should not be created.
* Separate workspaces should be created for DEV/TST
* Access controls within a workspace should be implemented through RLS \(Row Level Security\). RLS applies to the Viewer role only.
* Developers should be granted Member role to the DEV
* Deployment team should be granted the Member role to the TST and PRD workspaces. They can publish content to the workspaces.
* Admins can delete workspaces. They should be carefully chosen or the function should be centralized.
* Workspace contributors can share the content with other users and even enable them to share further. Therefore, a mistake can publicly expose business critical data. Proper governance model should be implemented for Power BI.
* Apps enable a distinct production layer for your visual content
* Another approach for DEV/TST/PRD segregation is to consider the app workspace as your DEV/TEST environment & consider Apps as your production environment. Visual content modifications in your app workspace will not immediately propagate to the App
* Use Apps to safely distribute visual content across the organization
* End users should not be given View role on the workspace as this exposes the dev content to the end users. Instead Apps should be exposed to the end users
* For UAT \(User Acceptance Testing\), the reports/dashboards available in the workspace and ready for UAT should be individually shared with the testers instead of giving them the Viewer role.
* Certify the datasets which fulfill the quality and consistency standards.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image16.png) Figure 16 Contributor Vs Member Role

* Apply data export regulations on individual reports
* Where possible, use security groups instead of granting access to individual users.
* Use direct query as much as possible to maintain a single version of truth in Snowflake.
* Use audit logs to monitor regularly:
  * Created Power BI dataset
  * Deleted Power BI report
  * Exported Power BI report visual data
  * Shared Power BI dashboard
* Turn off external data sharing
* Use security classification feature
* Turn-off data export unless absolutely required.

### 5.5. Firewall Rules

* Only account level network polices will be implemented for Snowflake.
* Each DIV will have its own network policy represented by the code of the DIV, i.e., {DIV}
* User level network policies will be allowed as an exception on case to case bases.
* For data transfer, public end points should not be exposed to the public internet.
* If Virtual Machines are required to be provisioned, RDP ports of the machines with direct access to data services will not be exposed to the internet.

### 5.6. User Access Request Process

1. User access to a dataset should always be approved by the corresponding data owner. There already exists a project on which other users are working on the GDP. This means that the sources of the project have been onboarded and corresponding security access groups in AAD and roles in Snowflake have been created. In this case, the following steps are followed in the process \(Figure 17\):
2. The user explores the datasets in the Azure Data Catalog \(ADC\) and finds the dataset he/she is interested in. He/she also finds the name and email address\(es\) of the corresponding data owner\(s\).
3. If ADC is not active yet, he/she looks in the Data Source Catalog to see if the source has already been onboarded and who is the owner.
4. The user fills in the User Access Form
5. The form is sent to the dataset owner for approval
6. The dataset owner either refuses the access request or asks the AD Administrator to add user X in security group Y.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image17.png) Figure 17 Access Request Process \(data exists in SF\)

### 5.7. Project Onboarding Process

When there does not exist any project in the SF, the onboarding process should be followed. Instead of granting individual users’ access to individual tables, it is recommended to group them in projects. The projects can range from adhoc data exploration to planned analytics projects. There are RO and RW roles per project which are linked to AD security groups. For new projects, the project owner takes the lead and initiates the request for granting access to project members. Figure 18 shows the process of onboarding a new project.

1. The project owner or his/her delegate\(s\) explore the metadata in the Azure Data Catalog to find the datasets required to be ingested for the project in scope. The data exploration can also be done at the source systems.
2. The project owner enters the details of the sources systems in the data source catalog if it does not exist.
3. The project onboarding form is filled in which contains the purpose of the project, the data sources required up to table level, different roles \(sponsors, owners, team\) and the access rights required.
4. The onboarding form is shared with the data owner\(s\) of the data sets required for the project.
5. The data owner approves the onboarding request and asks the AD administrators to either create new groups for the project or add the users to an existing group.
6. AD administrator creates the corresponding security groups and sends a confirmation to the project and data owner.
7. The project owner sends the filled-in and approved form to BICC for data ingestion and project onboarding
8. The BICC takes necessary actions for onboarding of the data, creation of necessary resources and access management.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image18.png) Figure 18 Project Onboarding Process

## 6. Data load strategy

### 6.1. Data ingestion from source files

* This process responsible for loading data from external sources.
* Our recommendation is to use Snowpipe with auto ingest for loading the data. 
* Snowpipe loads data within minutes after files are added to a stage and submitted for ingestion. This ensures users have the latest results, as soon as the raw data is available.
* Snowpipe doesn’t purge loaded files. For that is possible to use azure blob storage lifecycle management or to create an external job that deletes old files.
* Load data in small files\(~100MB\) instead of one large file. Load data in parallel using multiple nodes.

### 6.2. Data transformation \(logic steps\)

* This process is responsible to transform the raw data from many different systems into a unified analytics layer.
* This step is independent of the data ingestion step.

### 6.3. ETL tool Alternatives

* Data factory with Stored procedures
* SAAS ELT tool like Rivery
* On-prem ETL tool like Talend

### 6.4. Source Onboarding Process

Figure 19 shows the source onboarding request. Source onboarding is required when it is the first time GDP will be ingesting the data from the source. Sources from which data has been ingested by the GDP at least once do not need to be onboarded because the BICC has complete information about ingestion of data from that particular data source.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image19.png) Figure 19 Source Onboarding Process

The step by step onboarding process is defined below:

1. Dataset owner \(a person who is responsible for the data source\) fills in the source onboarding form with the help of the local IT team.
2. The source onboarding form requires the information about the security groups which will have RO/RW access to the data source. If the security groups are not present, the form is sent to the AAD administrators for creation of the groups.
3. Once the groups are created, the dataset owner is notified. He/she sends the form to the local SPOC.
4. Local SPOC sends the onboarding form to BICC.
5. BICC updates the data source catalog based on the information from the onboarding form.
6. BICC starts the process of onboarding the data source. The process includes:
7. Requesting the local SPOC for installation of Azure Integration Runtime \(AIR\) on the premises. BICC provides all the necessary information required for the installation as well as configuration of the AIR.
8. Local SPOC asks the local IT team for the installation of AIR if not already installed.
9. BICC validating the connection and loads a test table.

## 7. Virtual DWH Specifications

### 7.1. Topology

At the very least, define per region the following WH’s:

1. {DIV}\_{ENV}\_ELT\_WH – will be used for the ETL process.
2. {DIV}\_{ENV}\_REP\_WH – for used by analysis / reporting tool.

By default, only one {DIV}\_DEV\_WH will be created for ELT an reporting purposes. Separate DEV & TST Warehouses for the above shoul be created if required.

There are many Snowflake parameters in warehouse level that need to b reviewed. E.g., STATEMENT\_TIMEOUT\_IN\_SECONDS \(Amount of time, i seconds, after which a running SQL statement is canceled by the system\) For analytics WH is better to set it to 15 minutes.

### 7.2. Sizing

* It Is a best practice to experiment by running the same queries against warehouses of multiple sizes \(e.g. X-Large, Large, Medium\).
* From our experience, it is better to start with XS warehouse and then increase if needed.
* The multi cluster feature can help with high concurrency level. Instead if increasing the WH size it better to increase the maximum cluster size.

### 7.3. Life cycle management

### 7.4. Creation of DEV/TST/PRD Environment

* Warehouse
  * It is a good practice \(although is not mandatory\) to separate the computing warehouses of the Production environment with the warehouses of the Dev environment.
* Database
  * Snowflake cloning feature is a powerful tool that can be used to help simplify the life cycle management. At least 2 last versions of the clones should be kept.
  * Whenever is required to rebuild the dev environment clone the relevant PRD DB to new DEV DATABASE ![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image20.png)
  * The clone replicates all granted privileges on the corresponding child objects: schemas, tables, views, etc. besides the source object itself.
  * If the target table is fully qualified in the COPY statement in the pipe definition \(in the form of db\_name.schema\_name.table\_name or schema\_name.table\_name\), this can result in duplicate data getting loaded into the target table in the source database or schema by each pipe.
  * Cloned pipe objects that have the AUTO\_INGEST parameter set to TRUE will be in a new state, STOPPED\_CLONED, by default.
  * Our recommendation is to create a fully clone script that do following:
    * Clone DB to New DB and rename it to new name
    * Update the permissions for the relevant roles
    * Set pipes DB to the new DB \(if is not fully qualified\)

### 7.5. Naming convention

* Add ENV qualifier to all the cloned DBs \(DEV, TST\)
* Versioning: Add a numeric code that represent the version number. E.g.,: EMEA\_DEV\_ODS \_01
* If need to replace main DEV DB with other version you can use the command SWAP.

## 8. ETL design rules

### 8.1. General

The landing will be loaded with the data from the different sources. If for some reason the data in the file or from the source table does not meet our expectations, the loading of the destination table will stop and the next file or source table will be loaded. If USPs are used for ETL purpose, these should be called from one MasterUSP.

### 8.2. Naming conventions

* Always use English. If another language is required, approval from BICC should be requested.
* The naming of all the data packages should start with the target database. For example a package which load the staging should start with STG\_
* The naming of all the USPs should contain the target table. For example a staging USP which fills the DIM\_CUSTOMER table should be called USP\_STG\_DIM\_CUSTOMER.
* USPs to execute/control USPs: USP\_CTRL\_{Subject}.E.g. USP\_CTRL\_STG, USP\_CTRL\_DWH
* Only one entry point should be available. This is called the MasterUSP. MasterUSP should be called by the orchestration tool, e.g., Data Factory.
* All functional tables, stored procedures and views must be placed in the meta schema.
* Two types of stored procedures for Transformations:
  * USP \(User Stored Procedure\) written for local implementations/pipelines specific to the project
  * CSP \(Common Stored Procedure\) written by BICC to be shared by all countries for their implementation, e.g., procedures for loading the data from staging tables to dimension tables. CSPs can also have a DIV focus.

## 9. Glossary

| Abbreviation | Full Form |
| :--- | :--- |
| AAD | Azure Active Directory |
| AD | Active Directory |
| ADB | Azure Databricks |
| ADC | Azure Data Catalog |
| ADLS | Azure Data Lake Store |
| AIR | Azure Integration Runtime |
| BI | Business Intelligence |
| BICC | Business Intelligence Competence Center |
| CSP | Common Stored Procedure |
| DEV | Development |
| DF | Data Factory |
| DM | Data Mart |
| DWH | Data Warehouse |
| EDM | Enterprise Data Model |
| EDP | Enterprise Data Platform |
| ELT | Extract Load Transform |
| ETL | Extract Transform Load |
| FF | File Format |
| GDP | Global Data Platform |
| LT | Long Term |
| MT | Medium Term |
| MVP | Minimal Viable Product |
| ODS | Operational Data Store |
| PBI | Power BI |
| PRD | Production |
| QA | Quality Assurance |
| RLS | Row Level Security |
| RO | Read Only |
| RW | Read Write |
| SCD | Slowly Changing Dimension |
| SF | Snowflake |
| SPOC | Single Point of Contact |
| SS | Self Service |
| ST | Short Term |
| STG | Staging |
| TST | Test |
| USP | User Stored Procedure |

## 10. APPENDICES

### 10.1. APPENDIX A: General Principles for the Ingestion Tool

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

Delta based on a column with temporal information \(timestamp/incremental id\)

### 10.2. APPENDIX B: SnowSQL

The advantage of using SnowSQL is that it uses the compute power of the Snowflake platform for performing the transformations. The data does not need to exit the Snowflake platform and therefore the overhead of additional network latency is avoided. If an ETL tool is used, the data needs to exit the Snowflake platform to the ETL tool where the processing takes place. After processing, the result is pushed back to Snowflake. Some ETL tools support query pushdown which means that they can use the power of the underlying database for faster processing. This approach has its own pros and cons which are out of scope for this document.

The disadvantage of using only SnowSQL for transformation is the lack of orchestration capabilities. Here ADF can help as an orchestrator. It can call Snowflake stored procedures and report back the result as well as the errors. ADF also has strong scheduling capabilities and supports templating. It now comes with full support of Azure Devops.

