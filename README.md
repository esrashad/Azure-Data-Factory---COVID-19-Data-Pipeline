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

4. **Data Management**:
   - **Azure SQL Database**: Used for structured data storage, enabling efficient querying and data management.

5. **Publishing**:
   - Transformed and analyzed data is published for visualization and insights using Power BI.
This architecture leverages Azure Data Factory for orchestrating data workflows, along with other Azure services, to provide a scalable and efficient solution for COVID-19 data analytics, ensuring accurate insights and robust data management.


