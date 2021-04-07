---
description: >-
  These are the general guidelines on how to design the different layers of the
  Global Data Platform. These guidelines must be followed to guarantee a
  consistent across projects.
---

# Design Guidelines

## Databases & Zones

Figure 12 shows an overview of the database design rules in various storage layers of the GDP.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image13.png) Figure 12 Database Design Overview

### Landing Zone

The path layout is constructed as follows:

**/{DIV}/{ENV}/{type}/{zone\_code}/{source\_code}/{schema}/{table}\_{ingestion\_type}/{datetime}**

The paths consist of the following parts:

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Part</b>
      </th>
      <th style="text-align:left"><b>Description</b>
      </th>
      <th style="text-align:left">Format</th>
      <th style="text-align:left">Example(s)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">{DIV}</td>
      <td style="text-align:left">Aliaxis division code</td>
      <td style="text-align:left">2-4 upper-case letters</td>
      <td style="text-align:left">
        <p>EMEA</p>
        <p>APAC</p>
        <p>AGRP</p>
        <p>NA</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{ENV}</td>
      <td style="text-align:left">Code of the environment to which the zone belongs</td>
      <td style="text-align:left">3 upper-case letters</td>
      <td style="text-align:left">
        <p>DEV: Development environment</p>
        <p>TST: Test environment</p>
        <p>PRD: Production environment</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{type}</td>
      <td style="text-align:left">
        <p>Type of data the directory stores</p>
        <p>&lt;b&gt;&lt;/b&gt;</p>
      </td>
      <td style="text-align:left">4 lower-case letters</td>
      <td style="text-align:left">
        <p>data: data ingested from source systems</p>
        <p>exec: execution-related data, like pipeline metadata or logs</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{zone_code}</td>
      <td style="text-align:left">
        <p>A zone qualifies the layer a data entity belongs to and its properties.
          For example, data that is ingested in the data platform is stored in the
          landing zone.</p>
        <p>&lt;b&gt;&lt;/b&gt;</p>
      </td>
      <td style="text-align:left">3 lower-case letters</td>
      <td style="text-align:left">
        <p>lan: landing zone. This is where ingested data resides</p>
        <p>ods: operational data store</p>
        <p>sda: specific data area</p>
        <p>cda: global (common) data area</p>
        <p>scr: scratch</p>
        <p>adm: administration</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{source_code}</td>
      <td style="text-align:left">
        <p>A code is allocated to each source system of data.</p>
        <p>When a source is onboarded, the BICC works on the naming convention to
          make sure that it is correct.</p>
        <p>&lt;b&gt;&lt;/b&gt;</p>
      </td>
      <td style="text-align:left">Lower-case letters</td>
      <td style="text-align:left">
        <p>sharepoint</p>
        <p>ax-uk</p>
        <p>germany-sql-2012</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{schema}</td>
      <td style="text-align:left">Indicates the database schema from where the data is being ingested.
        <br
        />When dealing with non-relational databases, the schema can be generic
        (for example, all).</td>
      <td style="text-align:left">Lower-case letters</td>
      <td style="text-align:left">
        <p>dbo</p>
        <p>sales</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{table}</td>
      <td style="text-align:left">The name of the table from which data is being ingested</td>
      <td style="text-align:left">Lower-case letters</td>
      <td style="text-align:left">
        <p>sales_invoice</p>
        <p>accounts</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{ingestion_type}</td>
      <td style="text-align:left">
        <p>The type of ingestion.</p>
        <p>&lt;b&gt;&lt;/b&gt;</p>
      </td>
      <td style="text-align:left">1 lower-case letter</td>
      <td style="text-align:left">
        <p>s: snapshot ingestion. The entire table is ingested at every run</p>
        <p>d: delta ingestion. Only new and updated records are ingested.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">{datetime}</td>
      <td style="text-align:left">The date when the data was ingested.</td>
      <td style="text-align:left">10 characters in the format: YYYY-MM-DD</td>
      <td style="text-align:left">2020-01-01</td>
    </tr>
  </tbody>
</table>

### ODS \(Operational Data Store\)

ODS is a database in Snowflake which contains the latest snapshot \(or delta load\) of the source systems. Following naming convention should be used for the ODS database in Snowflake:

**{DIV}\_{ENV}\_ODS**

Inside the ODS database, each ingested schema from the source systems should be represented as:

**{DIV}\_{ENV}\_ODS.{SOURCE\_CODE}\_{SCHEMA}**

Formatting considerations:

* Upper case should be used
* Inside each schema, original table names should be used.

There are two types of datasets that are ingested into the ODS from the landing zone \(ADLS\). 

1. Deltas
2. Snapshots

#### Deltas

For deltas, changes need to be applied to the ODS in the overwrite mode. If a transformation tool is used other than SnowSQL, the data can be read by the tool and changes can be applied to the ODS. If SnowSQL is to be used, the changes can be applied in two ways:

1. Creation of external tables on the data in the landing zone and using SnowSQL to insert data into ODS. External tables allow SnowSQL to query data in external storage systems like ADLS/Blob Storage.
2. Loading data to a staging area in Snowflake and then using SnowSQL to read the staging tables and insert data in ODS. A {DIV}\_{ENV}\_ODS\_STG database with the same naming convention as ODS should be created to hold the staging schemas and tables.

Option 2 is recommended.

#### Snapshots

Snapshots can be directly loaded into ODS in the following ways:

1. Replacing the existing table in ODS with the new snapshot from the landing zone.
2. Appending the new snapshot records with a new date column

Option 2 is recommended. In this case, all queries to the tables should contain the date column in the where clause.

#### ODS Guidelines

* One ODS database per division and per environment \(DEV, TST, PRD\)
* The naming convention for the ODS database is {DIV}\_{ENV}\_ODS. For example EMEA\_DEV\_ODS for the ODS database of the development environment for EMEA.
* One schema for each source system
* Table structure
  * Tables should be named with the same name as the original table.
  * If the table is imported using deltas, a suffix of \_dl should be used.
  * Table columns should be all columns from the source system, plus 2 additional columns:
    * file\_name: will have the name of the file used to load this data
    * sf\_timestamp \(timestamp\_ntz\): has the timestamp of when the file is loaded to ODS
  * For tables holding semi structured data, the semi structured data should all be loaded into one column of type VARIANT, and the 2 above-mentioned columns should be added.

#### Landing Metadata Schema

A stage schema should be created in the ODS of the DIV with the following naming convention: **{DIV}\_{ENV}\_ODS.META\_LAN**

For example, for EMEA production environment, the name of the stage schema would be: **EMEA\_PRD**\_**ODS.META\_LAN**

This schema should contain all the file formats used for the loading of files from ADLS/Blob Storage, and any configuration tables that manage the ingestion of data into ODS.

### STG \(Staging\)

The staging layer is where data is prepared prior to making an incremental load into the data warehoue zone. 

The following guidelines should be followed in the staging zone

* Loading method is Overwrite
* The schema structure is identical as the DWH schema
* Target table of the staging area which should be in the same structure as the DWH-table
* The schemas in the STG layer should be defined as transient schemas.

### DWH \(Data Warehouse\)

The data warehouse layer \(DWH\) is where data in its final state is stored. Front end tools \(like, Power BI\) should connect only to this layer. 

Each data warehouse can contain a schema named "common". This schema will contain divisional-level entities that are reusable accross different datamarts in the same division. During the creation of a data mart, the data architect should identify entities that can be reused in existing/future data marts, and these entities should be moved to the common schema.

Each schema in the data warehouse represents a business function \(for example, Sales, or Marketing\).

Additional guidelines: 

* Loading method for this layer is merge
* If there are country-specific tables that belong in the common schema, for example master data that will be used across several projects for a given country, then this table must have a suffix with the company code. For example, DIM\_PRODUCT\_IT001. This table is the dimension table for products, which is specific for Italy. It is placed in the schema COMMON because it can be reused by other projects from the same country. 

### SANDBOX

There is a sandbox database, which can be used for testing any scenario or code. The name of the databsae is GLOBAL\_SANDBOX, and should be accessible to all developers in Snowflake.

### META

The META database is used for storage of all non-data entities. The META database should follow the following naming convention {DIV}\_{ENV}\_META

Following should be the schemas in the META database. 

1. **USP**: This schema stores the User Stored Procedures written by local teams for transformation etc. E.g., EMEA\_PRD\_META.USP.USP\_STG\_DIM\_CUSTOMER for a USP loading data to the staging table of the customer dimension. 
2. **CSP**: This schema stores the Common Stored Procedures \(CSPs\) common to all projects with in a division but not to other divisions. 
3. **FF**: The File Format \(FF\) schema stores all file formats used within a DIV. For instance, a JSON file format can be stored as: EMEA\_PRD\_META.FF.FF\_JSON. It is recommended to use global file formats as a standard but for loading of files not coming through the ingestion system, local file formats can be created. 
4. **MONITORING**: All monitoring information regarding the transformation pipelines is stored in this schema.

## Deployment Scripts/Stored Procedures

The deployment scripts as well as the stored procedures should use {ENV} as a variable. Use of {ENV} as a variable allows the promotion of the code to different environments via deployment scripts without any change to the code.

### Coding conventions

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
* For example,

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

## Naming conventions

### General

* By default, use English
* For data sources replicated to the GDP, the field/table names in local language will be replicated but effort should be made so new entity names should be in English
* Never use SQL keywords as the name for objects
* Never use white space
* Names must be readable and meaningful
* Use only letters and underscores.
* No repeating of entity name unless reserved words are used:
  * Customer\_Name, Customer\_City =&gt; Name, City in the Customer dimension
  * Product\_Name, Product\_Category =&gt; Name, Category in the Product dimension
  * Date in the Hourly table =} hourly\_dt

### Databases

* All DB’s names must be in capital letters
* For lower environments add \_{ENV} to the DB name. E.g. EMEA\_DEV\_ODS

### Tables

* All table names must be in capital letters
* No prefixes for landing, staging and data warehouse are needed because the tables are in different databases/schemas.
* Tables should be called:
  * In Operational Data Store \(ODS\) layer: {source system name}
  * In Staging \(STG\) layer: DIM\_{name} or FACT\_{name}
  * In Data Warehouse \(DWH\) layer: DIM\_{name}, FACT\_{name}, BRIDGE\_{name}
  * Proper use of prefixing in the data warehouse database should only include the above prefixes so that everyone can recognize the data.
* Target table of the staging area should be an almost exact copy of the DWH table
* When there are local \(country-specific\) projects that will load the same dimension or fact, but without names or language translations that will be common for the division, the table must have the company code as a suffix. For example, 
  * EMEA as a division has created a customer dimension table DIM\_CUSTOMER. 
  * A local project in Spain needs to build reports that will show Spain-specific customers, with a customer hierarchy that doesn’t match the hierarchy used at the divisional level. 
  * In this case, Spain will create a customer dimension table DIM\_CUSTOMER\_ES001, that will hold the customer information that is specific for the Spanish local projects.
* TMP\_{name} for intermediate staging tables, if necessary

### Fields

* Use whole words where possible \(abbreviations should be avoided, acronyms and common usage are accepted\)
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

### Views

* VW\_{description of the view}

### Stored procedures

* “USP\_{description of stored procedure}”

### Users

For users that were created in Snowflake \(in contrast to users that were created in AD\) add prefix of sf\_ before the user name. For example, sf\_new\_snowflake\_user

### Data Factory

There should be two data factory instances per DIV. 1. Local Data Factory: This factory is responsible for all orchestration activities within a DIV for local implementations. 2. BICC Data Factory: This factory contains the pipelines used for DIV’s BICC pipelines e.g., data ingestion & Lifecycle management.

The following naming convension should be followed:

* {DIV}\_{ENV}\_DF for local implementations
* {DIV}\_{ENV}\_DF\_BICC for BICC pipelines

## Additional conventions

### ODS Staging Tables

Every table in ODS must have audit fields which must be set in the ETL, not with triggers:

* FILE\_NAME = the file id in which the data originally was presented
* SF\_TIMESTAMP = creation date and time: Default CURRENT\_TIMESTAMP\(\)

### Datawarehouse tables

* Every table in DWH must have audit fields which must be set in the ETL, not with triggers:
  * CREATE\_TIMESTAMP = creation date and time: Default GETDATE\(\)
  * UPDATE\_TIMESTAMP = date and time when the data has been modified: Default GETDATE\(\)
  * SF\_TIMESTAMP –min\(sf\_timestamp\) of all of the table source
* Every dimension table must have a surrogate key column \(more details [here](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dimension-surrogate-key/)\)
  * This can be done by creating an identity column. Identity starts at 1 and increments by 1
* All non-measure columns should be “not nullable”.
* For all foreign keys on a fact table the values of the corresponding business keys will be stored on the table as well.

### Data types

* Use INT for numbers without a fractional component.
* Use NUMBER\(30,X\) to define numbers with a fractional component. Details [here](https://support.snowflake.net/s/article/To-Float-or-Not-to-Float-Choosing-Correct-Numeric-Data-Type-in-Snowflake).
* TIMESTAMP\_LTZ: All time stamp should be in UTC \(Details [here](https://medium.com/hashmapinc/the-hitchhikers-guide-to-timestamps-in-snowflake-part-1-534e2373fb8e)\). You can define another column for the local timestamp \(e.g., local time of transaction\). TIMESTAMP\_LTZ internally stores UTC time with a specified precision. However, all operations are performed in the current session’s time zone, controlled by the TIMEZONE.
* Use DATE for date columns
* For all fields containing a string, e.g., Varchar or nvarchar, STRING datatype should be used
* Use integers for storing year/month/day values

### Data Models

The landing zone contains data as files in directories partitioned by the date/time of ingestion. 

The ODS stores data in the model used by the source systems which mostly 3NF but referential integrities are not enforced. 

In DWH, for the storage of data marts, dimensional model is used. The choice of snowflake vs star schema depends on the usecase and is determined by the development team. DWH also contains the enterprise data model. When a project is initiated for the EDM, the data architect should determine the suitable storage model.

