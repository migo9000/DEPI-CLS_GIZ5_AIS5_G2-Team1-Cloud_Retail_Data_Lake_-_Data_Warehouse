# Cloud Retail Data Lake & Data Warehouse

## Problem Statement
A retail organization has multiple operational datasets but needs a scalable cloud architecture for storing raw data, processing it, and providing structured analytical data to business users.

## Project Objective
Build a small-scale Azure data platform using a **data lake → processing → warehouse** architecture.  
Azure's architecture guidance describes Data Lake Storage and Synapse as components that can work together for storing, processing, and querying data at scale. *(Microsoft Learn)*

## Key Features / Scope
- Upload raw CSV/JSON/Parquet data to Azure Data Lake Storage Gen2  
- Organize data into raw/processed/curated zones  
- Query lake files using Synapse serverless SQL  
- Transform data using Synapse Spark/PySpark  
- Convert appropriate datasets to Parquet  
- Create a dimensional model:
  - Fact Sales  
  - Dim Product  
  - Dim Customer  
  - Dim Store  
  - Dim Date  
- Create Synapse external tables/views  
- Load curated data into a warehouse  
- Implement basic data-quality checks  
- Apply appropriate access controls  
- Document the architecture  
- Monitor pipeline execution  

## Expected Deliverables
- Azure architecture diagram  
- ADLS folder structure  
- PySpark notebooks  
- Synapse SQL scripts  
- Dimensional model  
- Data-quality validation  
- Sample analytical queries  
- Pipeline screenshots  
- Technical README  

## Optional Extension
Add **Delta Lake** for transactional/ACID processing and optimize selected datasets using partitioning/Z-ordering.