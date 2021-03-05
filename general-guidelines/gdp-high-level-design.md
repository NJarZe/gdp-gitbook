# GDP High-Level Design

The following image shows the high-level architecture of the Global Data Platform. Two options are shown for the ingestion of data into the Global Data Platform. Azure integration Runtime is used for data ingestion if Data Factory is used otherwise an ETL/ELT tool can be used directly to write data into ADLS Gen2. Some ETL/ELT tools can write the data directly to Snowflake but for the purpose of standardization, data will be copied to ADLS first. Using ADLS as a landing zone allows the use of other data analytics tools such as Databricks.

![High-Level Design](../.gitbook/assets/dp-high-level-design.png)

The general structure of the data processing pipelines running on the Global Data Platform \(GDP\) follow the structure shown below

![](https://github.com/NJarZe/gdp-gitbook/tree/4be67165d0fb6646303b084d135b48ef7065ae6d/media/image8.png) Figure 7 High-Level Pipeline Architecture

The blocks in green represent processing whereas the blocks in blue represent storage. In the following sub-sections, we will describe each of the components.

