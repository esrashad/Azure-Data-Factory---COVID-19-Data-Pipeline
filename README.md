# Azure-Data-Factory---COVID-19-Data-Pipeline
This repository features a project that leverages Azure Data Factory to build a data pipeline for processing and analyzing COVID-19 data. It demonstrates how to configure Azure Data Factory to handle various data sources, perform data transformations, and generate insights from COVID-19 datasets.



# Solution Architecture Overview
<img width="899" alt="architecture_diagram-2" src="https://github.com/user-attachments/assets/c9096c9d-b317-477c-b3f7-fa9fe83dcd0b">

The diagram represents the architecture of our COVID-19 data analytics solution:
1. **Data Ingestion**:
   - **ECDC COVID-19 Data**: Collected via an HTTP connector.
   - **Population Data**: Retrieved from Azure Blob Storage.

2. **Data Storage**:
   - All ingested data is initially stored in Azure Data Lake Storage Gen2.

3. **Data Transformation and Analysis**:
   - Data is processed and analyzed through various transformation steps orchestrated by Azure Data Factory.

4. **Data Warehousing*:
   - **Azure SQL Database**: Used for structured data storage, enabling efficient querying and data management.

5. **Publishing**:
   - Transformed and analyzed data is published for visualization and insights using Power BI.

### Environment setup
- Azure Subscription
- Data Factory
![Screenshot 2024-07-20 224923](https://github.com/user-attachments/assets/189c7ae8-661c-4714-9501-d82ed51310fe)

- Azure Blob Storage Account
  ![Screenshot 2024-07-20 230404](https://github.com/user-attachments/assets/a91f566f-d5d8-49cb-87fc-0d046e794812)

- Data Lake Storage Gen2
  ![Screenshot 2024-07-20 231327](https://github.com/user-attachments/assets/1b0adb73-5d68-4345-a2d6-0e310970bfa7)

- Azure SQL Database
  ![Screenshot 2024-07-20 232537](https://github.com/user-attachments/assets/81c3315f-7892-4678-a51f-dbf0e60fade8)

- Azure Databricks Cluster
![image](https://github.com/user-attachments/assets/67b358c3-ca56-4266-ba48-1ad8afbb8df0)



### Let's start step by step
# Step 1: Data Ingestion
### Ingest population data
we ingested our data from blob storage to azure data lake Gen2 by copy activity
![[Pasted image 20240720234123.png]]
#### Copy Activity 
![[Pasted image 20240720234400.png]]

- Create population container in the blob storage and put zipped TSV file in it.  
- Create the raw container in data lake storage Gen2
- Create linked service to blob storage and data lake storage Gen2
- Create Dataset for Blob storage and Data lake storage Gen2
- Create Pipeline of Copy Population data
- Check if the Column Count of the file matches then copy the file
- Create storage event trigger for pipeline
### Ingest ECDC data (HTTP Connector)
![[Pasted image 20240721022714.png]]
##### Data Ingestion Requirements:
- Covid-19 new cases and deaths by Country
- Covid-19 Hospital admissions & ICU cases
- Country Response to Covid-19
![[Pasted image 20240721142104.png]]
Create linked service to HTTP connector 
Create dataset from HTTP connector
Create dataset for ADLS Gen2
Create dataset for json file (in the configs directory) to make lookup with it to get the information
Create Pipeline that consists of:
- Lookup to get the information from json file
- ForEach to make for loop to the file to get the all http data
- Copy activity to copy data from http connector
![[Pasted image 20240721165053.png]]
# Step 2:Transformation
![[Pasted image 20240721172756.png]]
### 1)Transform Cases & Deaths Data 
use **Data Flows** to:
1) Filter only Europe data.
2) lookup to lookup file to get 2_digit_Code and 3_digit_code
3) Pivot the indicator column to confirmed cases and deaths
4) change data to reported date
5) Drop rate_14_day column
![[Pasted image 20240721174257.png]]
This all the transformation that we need to do on cases and deaths data
![[Pasted image 20240721214223.png]]
The Pipeline of the Dataflow
![[Pasted image 20240722104748.png]]
### 2)Transform Hospital Admissions Data
1) Drop url column
2) Separate the file into 2 files one for daily and one for weakly
3) Pivot the indicator colum to Daily hospital occupancy, Daily ICU occupancy, and the same with weakly file
4) Rename date to reported date
5) Use lookup to get reported_weak_start_date and  reported_weak_end_date
6) Use another lookup to get some information
![[Pasted image 20240722130108.png]]
The Data Flow
![[Pasted image 20240722145532.png]]

### 3)Transform Population Data
![[Pasted image 20240722175518.png]]

**We use Databricks and spark to transform population data**
- Create Databricks Service
- Create Databricks Cluster
- Mount Storage Accounts 
- Transformation Requirements
- Creating Pipeline
##### To Mount data lake storage:
- Create Azure Service Principal 
- Grant access for data lake to Azure Service Principal
- Create the mount in databricks using Service Principal
### Population data transformation requirements
![[Pasted image 20240722192048.png]]
1. Split the country code & age group
2. Exclude all data other than 2019
3. Remove non numeric data from percentage
4. Pivot the data by age group
5. Join to dim_country to get the country, 3 digit country code and the total population.
# step 3: Copy Data to Azure SQL
![[Pasted image 20240722203639.png]]
- Create tables
- Create pipeline to copy data
![[Pasted image 20240722210958.png]]
# 3)Data Orchestration
### Requirements
- Pipeline executions are full automated 
- Pipelines run at regular intervals or on an event occurring 
- Activities only run once the upstream dependency has been satisfied
- Easier to monitor for execution progress and issues
The Pipeline of execute population data

![[Pasted image 20240722222158.png]]

  



