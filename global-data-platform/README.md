# Global Data Platform

## 4. Design Rules

### 

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

