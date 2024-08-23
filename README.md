# Azure Data Factory-COVID-19 Data Pipeline
This project leverages Azure Data Factory to build a data pipeline for processing and analyzing COVID-19 data. It demonstrates how to configure Azure Data Factory to handle various data sources, perform data transformations, and generate insights from COVID-19 datasets.



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
![Screenshot 2024-07-20 234113](https://github.com/user-attachments/assets/69807e06-14e2-48f9-aea6-2c78a2fd111e)

#### Copy Activity 
![Screenshot 2024-07-20 234345](https://github.com/user-attachments/assets/44c551fa-81f0-48ce-a3fe-b9758d31d157)


- Create population container in the blob storage and put zipped TSV file in it.  
- Create the raw container in data lake storage Gen2
- Create linked service to blob storage and data lake storage Gen2
- Create Dataset for Blob storage and Data lake storage Gen2
- Create Pipeline of Copy Population data
- Check if the Column Count of the file matches then copy the file
- Create storage event trigger for pipeline
### Ingest ECDC data (HTTP Connector)
![Screenshot 2024-07-21 022704](https://github.com/user-attachments/assets/0540f5e1-3ce3-49a8-ae88-5e349e3e86d9)

##### Data Ingestion Requirements:
- Covid-19 new cases and deaths by Country
- Covid-19 Hospital admissions & ICU cases
- Country Response to Covid-19
![image](https://github.com/user-attachments/assets/4799d473-fac2-4d82-8709-66dc1ac23e19)

Create linked service to HTTP connector 
Create dataset from HTTP connector
Create dataset for ADLS Gen2
Create dataset for json file (in the configs directory) to make lookup with it to get the information
Create Pipeline that consists of:
- Lookup to get the information from json file
- ForEach to make for loop to the file to get the all http data
- Copy activity to copy data from http connector
  ![image](https://github.com/user-attachments/assets/4442a678-3685-456c-ab7b-0d0e36c6585e)

# Step 2:Transformation
![Screenshot 2024-07-21 172727](https://github.com/user-attachments/assets/a0680fe0-69ac-4bf9-bc76-28c28849c815)

### 1)Transform Cases & Deaths Data 
use **Data Flows** to:
1) Filter only Europe data.
2) lookup to lookup file to get 2_digit_Code and 3_digit_code
3) Pivot the indicator column to confirmed cases and deaths
4) change data to reported date
5) Drop rate_14_day column
![image](https://github.com/user-attachments/assets/cbadad51-7634-498d-80bb-1fcb736acedc)

This all the transformation that we need to do on cases and deaths data
![image](https://github.com/user-attachments/assets/6f8ed6ba-baa3-4d79-9060-b0701d7fada8)

The Pipeline of the Dataflow
![image](https://github.com/user-attachments/assets/24e1aa5a-299f-4b10-8a50-d19ca412462e)

### 2)Transform Hospital Admissions Data
1) Drop url column
2) Separate the file into 2 files one for daily and one for weakly
3) Pivot the indicator colum to Daily hospital occupancy, Daily ICU occupancy, and the same with weakly file
4) Rename date to reported date
5) Use lookup to get reported_weak_start_date and  reported_weak_end_date
6) Use another lookup to get some information
![image](https://github.com/user-attachments/assets/a82d10c7-09a9-428c-8268-81f4fac5d7cd)

The Data Flow
![image](https://github.com/user-attachments/assets/8e711d28-8bea-4d75-b411-b479f489b80d)


### 3)Transform Population Data
![image](https://github.com/user-attachments/assets/023b180a-b469-447a-b30b-b22872eb7dd9)


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
![image](https://github.com/user-attachments/assets/80c4def7-10c1-4814-8e1f-4d0770e3a51b)

1. Split the country code & age group
2. Exclude all data other than 2019
3. Remove non numeric data from percentage
4. Pivot the data by age group
5. Join to dim_country to get the country, 3 digit country code and the total population.
# step 3: Copy Data to Azure SQL
![image](https://github.com/user-attachments/assets/d75de8ea-9ed7-4911-a058-698ea2dea78e)

- Create tables
- Create pipeline to copy data
![image](https://github.com/user-attachments/assets/8a135edf-8bd5-497f-8ae3-531639412a63)

# 3)Data Orchestration
### Requirements
- Pipeline executions are full automated 
- Pipelines run at regular intervals or on an event occurring 
- Activities only run once the upstream dependency has been satisfied
- Easier to monitor for execution progress and issues
The Pipeline of execute population data

![image](https://github.com/user-attachments/assets/d3347c77-5091-4915-9f34-522296f4b718)


  



