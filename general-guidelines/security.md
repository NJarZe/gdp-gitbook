# Security

## Snowflake security

This section covers the security of Snowflake objects \(tables, views, stored procedures, etc.\).

### General Guideline

* Every object has only 1 role owner
* Privileges on objects are granted to Roles, and roles are granted to Users or to other Roles.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image14.png) Figure 13 Snowflake General Control Hierarchy

### Aliaxis Roles Hierarchy

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image15.png) Figure 14 Aliaxis Roles Hierarchy

ROLE\_EMEA\_PRD\_{PROJECT-NAME}\_RW/RO =&gt; ODS, STG, DWH

Systems roles that are in IT level:

#### ACCOUNTADMIN

A role that encapsulates the SYSADMIN and SECURITYADMIN system-defined roles. It is the top-level role in the system and should be granted only to a limited/controlled number of users in your account.

#### SECURITYADMIN

A role that can monitor, and manage users and roles. More specifically, this role is used to: Modify and monitor any user, role, or session; grant permissions for cross-database permissions \(e.g. division analyst would like to get read access to a table located in another division’s DB\), or modify any grant, including revoking it.

#### SYSADMIN

A role that has privileges to create warehouses and databases \(and other objects\) in an account.

This is the only role that should create databases and warehouses 

SYSADMIN role is the only role that can write into the ODS layer in addition to a technical user created specifically for this purpose.

SYSADMIN role will grant manage permissions to the lower level roles

#### DIV Specific Roles

For each division the SECURITYADMIN will create the following roles:

* **{DIV}\_ADMIN:** A role that has privileges to create schemas in the STG and DWH layers. This role will be responsible to grant permissions to roles inside the division’s STG and DWH DB. The role can change warehouse properties
* **{DIV}\_{ENV}\_RW:** A role with a writing permission inside the project DB. There can be many write roles: {proj}\_etl\_prod, {proj}\_etl\_dev, {proj}\_dev\_developer
* **{DIV}\_{ENV}\_RO:** Read-only role on the project data schemas for end users. There can be many read-only roles. This role's main purpose is to query business data inside the production environment
* A good practise will be to create read-only role per business world \(one for finance, one for marketing etc.\)

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

### Data Sharing Between Divisions

In a case that user or process from a specific division needs to select data from other division we recommend to create a new role. The SYSADMIN will be responsible to provide select permissions on the required object to the role.

For example, a user from EMEA that needs to see data from ODS table which located in the ODS layer of APAC. Here are the required actions:

* Create new role: ROLE\_APAC\_TO\_EMEA\_{PROJECT\_NAME}
* Grant select permission to this role

User from EMEA can join data between EMEA and APAC by using the fully qualified call to the table db\_name.schema\_name.table\_name

## Power BI Security

The general guidelines for Power BI security are the following. For information about Power BI governance and organization, see [Power BI](../product-specific-guidelines/power-bi-guidelines.md) section.

* Workspaces are the lowest level of access control in PBI. Big workspaces containing datasets/dashboards for multiple business lines should not be created.
* Separate workspaces should be created for DEV/PRD
* Developers should be granted Member role to the DEV
* Deployment team should be granted the Member role to PRD workspaces. They can publish content to the workspaces.
* Certify the datasets which fulfill the quality and consistency standards.
* Apply data export regulations on individual reports
* Where possible, use security groups instead of granting access to individual users.
* Use audit logs to monitor regularly:
* * Created Power BI dataset
  * Deleted Power BI report
  * Exported Power BI report visual data
  * Shared Power BI dashboard
* Turn off external data sharing
* Use security classification feature
* Turn-off data export unless absolutely required.

## User Access Request Process

1. User access to a dataset should always be approved by the corresponding data owner. If there already exists a project on which other users are working on the GDP. This means that the sources of the project have been onboarded and corresponding security access groups in AAD and roles in Snowflake have been created. In this case, the following steps are followed in the process \(Figure 17\):
2. The user explores the datasets in the Data Catalog and finds the dataset he/she is interested in. He/she also finds the name and email address\(es\) of the corresponding data owner\(s\).
3. The user fills in the User Access Form
4. The form is sent to the dataset owner for approval
5. The dataset owner either refuses the access request or asks the AD Administrator to add user X in security group Y.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image17.png) Figure 17 Access Request Process \(data exists in SF\)

## Project Onboarding Process

When there does not exist any project in Snowflake, the onboarding process should be followed. Instead of granting individual users access to individual tables, it is recommended to group them in projects. The projects can range from ad-hoc data exploration to planned analytics projects. There are RO and RW roles per project which are linked to AD security groups. For new projects, the project owner takes the lead and initiates the request for granting access to project members. Figure 18 shows the process of onboarding a new project.

1. The project owner or his/her delegate\(s\) explore the metadata in the Azure Data Catalog to find the datasets required to be ingested for the project in scope. The data exploration can also be done at the source systems.
2. The project owner enters the details of the sources systems in the data source catalog if it does not exist.
3. The project onboarding form is filled in which contains the purpose of the project, the data sources required up to table level, different roles \(sponsors, owners, team\) and the access rights required.
4. The onboarding form is shared with the data owner\(s\) of the data sets required for the project.
5. The data owner approves the onboarding request and asks the AD administrators to either create new groups for the project or add the users to an existing group.
6. AD administrator creates the corresponding security groups and sends a confirmation to the project and data owner.
7. The project owner sends the filled-in and approved form to BICC for data ingestion and project onboarding
8. The BICC takes necessary actions for onboarding of the data, creation of necessary resources and access management.

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image18.png) Figure 18 Project Onboarding Process

