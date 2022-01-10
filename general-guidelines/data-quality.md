---
description: Aliaxis Data Quality Framework
---

# Data Quality Framework

## Business Definition

This document covers the details of data quality evaluation process for Aliaxis business processes and its corresponding DWH tables. This document will enable the end users to define data quality rules and constraints in an excel file which will then be uploaded on the SharePoint from where it will be picked up by the daily ingestion process. These new tables will then act as the driving force for the entire data quality and business rule evaluation pipelines.

![Architecture with Processing Technologies](../.gitbook/assets/adf-data-quality.png)

### **Data Quality Tables**

### Expectation
This table contains expectation of the business or process that needs to be evaluated based on and defined by the SQL queries in *Business Rule* table.
The structure of the metadata table is as follows

| Field Name | Description | Sample Value\(s\) |
| :--- | :--- | :--- |
| EXPECTATION_ID | Unique id assignment for the expectation, it could also be folder path for manual file ingestion | _Corporate_EMEA/SALES/sales-budget, sales-unique-id |
| DIVISION | Division name for which rule is defined | EMEA, AGRP |
| BUSINESS_UNIT | Business unit name for which the rule is defined | end-to-end, jimtem |
| BUSINESS_UNIT_COMPONENT |  Business unit component name for which the rule is defined | sales, crm, resinbi |
| EXPECTATION_DESCRIPTION | A meaningful statement explaning the expectation, this will be added to email notification | As business user, I would like to sure that commercial segaments are valid in manually uploaded file. |
| ENVIRONMENT | Name of the enviornment for which rule has beed defined | DEV, TST, PRD |
| DISTRIBUTION_LIST_TO | Comma(,) or semi-colon(;) separated email addresses | hhasan@aliaxis.com;cdelasmarinas@aliaxis.com |
| DISTRIBUTION_LIST_CC | Acronym for the division. Used to create the ADLS path where files are stored | flourenco@aliaxis.com |
| EMAIL_SUBJECT | Email subject for the expectation | EMEA Data Quality Check for Sales Budget |
| EMAIL_HTML_BODY | HTML email message for the expectation | \<p>Data quality rules are passed.\</p> |
| RAISE_EXCEPTION | Whether or not failed the pipeline on data quality compromise | True, False |
| SEND_ALERT | Whether or not send email alert on data quality compromise | True, False |
| IS_ACTIVE | Boolean attribute defines whether or not this expectation is active and should run or not. | True, False |


### Business Rule
This table contains SQL queries for a given *Expectation* defined in previous tab.

The structure of the metadata table is as follows
| Field Name | Description | Sample Value\(s\) |
| :--- | :--- | :--- |
| BUSINESS_RULE_ID | Unique id assignment for the business rule | 1234, unique-guid |
| SQL_STATEMENT | SQL query to be run against expectation, query `MUST` return boolean result i.e. `FALSE AS RESULT`. If `FALSE` then quality check will be considered as successful| ```SELECT IFNULL (SUM (CASE WHEN B.COMPANY_CD IS NULL THEN 1 ELSE 0 END), 0) > -1 AS RESULT FROM EMEA_DEV_ODS.MANUAL_UPLOADS.SALES_BUDGET A LEFT JOIN EMEA_DEV_DWH.SALES.DIM_COMPANY B ON A.COMPANY_CD = B.COMPANY_CD ;``` |
| RULE_DESCRIPTION | Description for the rule | Step 1: Check COMPANY_CD |
| HOW_TO_FIX | Details about how to fix this data quality issue | Make sure company codes are correct and matching with code defined in SALES.DIM_COMPANY table. |
| EXECUTION_ORDER | An order of execution for all SQL statements aka business rules against expectation. | 1,2,3,4,5,6... |
| IS_ACTIVE | Boolean attribute defines whether or not this expectation is active and should run or not. | True, False |
| EXPECTATION_ID | A foreign key reference to *Expectation* table | _Corporate_EMEA/SALES/sales-budget, sales-unique-id |

### Business Rule Result
This table contains SQL queries for a given *Expectation* defined in previous tab.

### Azure Data Factory Pipeline (ADF)

Azure data factory pipeline can be used to trigger the expectation with set of business rules after certain point in master piplein(s). For example, this pipeline can be called after data has been ingested via manual upload process as well as after the STG ingestion to evaluate the quality of that after transformation.
### Parameters

1. `EXPECTATION_ID` - optional if DIVISION,BUSINESS_UNIT and BUSINESS_UNIT_COMPONENT are supplied
2. `DIVISION` - optional if EXPECTATION_ID is supplied
3. `BUSINESS_UNIT` - optional if EXPECTATION_ID or DIVISION is supplied with *ALL* as value for this param
4. `BUSINESS_UNIT_COMPONENT` - optional if EXPECTATION_ID or DIVISION or BUSINESS_UNIT is supplied with *ALL* as value for this param
5. `FILE_NAME` - optional, can be sent from manaul uploads pipeline to add the filename in email body.

### ADF Pipeline Reference Query
```
SELECT * FROM EMEA_@{pipeline().globalParameters.ADLS_ENV}_ODS.MANUAL_UPLOADS.EXPECTATION_DQ
WHERE IS_ACTIVE = TO_BOOLEAN('TRUE')
AND  
(
    Upper('@{pipeline().parameters.EXPECTATION_ID}')  like Upper('%'||EXPECTATION_ID||'%')
    OR
    (
        UPPER("DIVISION") = UPPER('@{pipeline().parameters.DIVISION}')
		AND
		UPPER("BUSINESS_UNIT") = UPPER(CASE WHEN UPPER('@{pipeline().parameters.BUSINESS_UNIT}') = 'ALL' THEN "BUSINESS_UNIT" ELSE '@{pipeline().parameters.BUSINESS_UNIT}' END)
		AND
		UPPER("BUSINESS_UNIT_COMPONENT") = UPPER(CASE WHEN UPPER('@{pipeline().parameters.BUSINESS_UNIT_COMPONENT}') = 'ALL' THEN "BUSINESS_UNIT_COMPONENT" ELSE '@{pipeline().parameters.BUSINESS_UNIT_COMPONENT}' END)
    )
    
)
LIMIT 1;
```

### Sample Email Notification
Below is sample email notification generated by the process after evaluation of an expectation consists of multiple business rules.

![Sample email notification](../.gitbook/assets/sample-email-notification.png)

## How to add my expectations and business rules?
To make the ingestion process easy and accessible to technical teams, expectations and business rules can be uploaded to SharePoint location as an excel file.
#### SharePoint Location
``` /<DIVISION//GDP_Data_Quality_Framework/business-rule/<filename>.xlsx ```

File must contain above mentioned fields required for tables in separate tabs.
[download the sample excel file for EMEA](../.gitbook/assets/sample-excel-file.xlsx)