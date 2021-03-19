---
description: >-
  This section defines guidelines regarding the configuration of different
  Snowflake objects.
---

# Snowflake Configuration

## Virtual DWH Specifications

### Topology

At the very least, define per region the following WH’s:

1. {DIV}\_{ENV}\_ELT\_WH – will be used for the ETL process.
2. {DIV}\_{ENV}\_REP\_WH – for used by analysis / reporting tool.

By default, only one {DIV}\_DEV\_WH will be created for ELT an reporting purposes. Separate DEV & TST Warehouses for the above shoul be created if required.

There are many Snowflake parameters in warehouse level that need to b reviewed. E.g., STATEMENT\_TIMEOUT\_IN\_SECONDS \(Amount of time, i seconds, after which a running SQL statement is canceled by the system\) For analytics WH is better to set it to 15 minutes.

### Sizing

It Is a best practice to experiment by running the same queries against warehouses of multiple sizes \(e.g. X-Large, Large, Medium\). The general recommendation is to start with XS warehouse and then increase if needed.

The multi cluster feature can help with high concurrency level; instead if increasing the WH size it better to increase the maximum cluster size. This should always be tested along with BICC to ensure optimal results.

## Creation of DEV/TST/PRD Environment

### Warehouse

It is a good practice to separate the computing warehouses of the Production environment with the warehouses of the Dev environment. 

By default, each division will have separate warehouses for Production and Development. If this is not the case, contact BICC to ensure this recommendation is followed.

### Database

Snowflake cloning feature is a powerful tool that can be used to help simplify the life cycle management. At least 2 last versions of the clones should be kept.

Whenever it's required to rebuild the dev environment clone the relevant PRD DB to new DEV DATABASE ![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image20.png)

Cloning replicates all granted privileges on the corresponding child objects: schemas, tables, views, etc. besides the source object itself.

{% hint style="warning" %}
**Caution:** If the target table is fully qualified in the COPY statement in the pipe definition \(in the form of db\_name.schema\_name.table\_name or schema\_name.table\_name\), this can result in duplicate data getting loaded into the target table in the source database or schema by each pipe.
{% endhint %}

Cloned pipe objects that have the AUTO\_INGEST parameter set to TRUE will be in a new state, STOPPED\_CLONED, by default.

The recommendation is to create a fully clone script that do following:

* Clone DB to New DB and rename it to new name
* Update the permissions for the relevant roles
* Set pipes DB to the new DB \(if is not fully qualified\)

### Cloned objects naming convention

* Add ENV qualifier to all the cloned DBs \(DEV, TST\)
* Versioning: Add a numeric code that represent the version number. E.g.,: EMEA\_DEV\_ODS \_01
* If need to replace main DEV DB with other version you can use the command SWAP.

