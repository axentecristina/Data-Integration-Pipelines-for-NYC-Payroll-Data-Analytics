# Data Integration Pipelines for NYC Payroll Data Analytics

## Project Introduction

The City of New York would like to develop a Data Analytics platform on Azure Synapse Analytics to accomplish two primary objectives:

1. Analyze how the City's financial resources are allocated and how much of the City's budget is being devoted to overtime.
2. Make the data available to the interested public to show how the City’s budget is being spent on salary and overtime pay for all municipal employees.

The source data resides in Azure Data Lake and needs to be processed in a NYC data warehouse. The source datasets consist of CSV files with Employee master data and monthly payroll data entered by various City agencies.

## Project Instructions

## Step 1: Prepare the Data Infrastructure

Azure Resources used for this project:
- Azure Data Lake Gen2
- Azure SQL DB
- Azure Data Factory
- Azure Synapse Analytics

1. Create the data lake and upload data

- Create an Azure Data Lake Storage Gen2 (storage account) and associated storage container resource. In the created container, create three directories in this storage container named
  - dirpayrollfiles,
  - dirhistoryfiles.
- Upload the below files in **dirpayrollfiles** folder:
  - EmpMaster.csv
  - AgencyMaster.csv
  - TitleMaster.csv
  - nycpayroll_2021.csv
    ![DIRPAYROLLFILES](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/0048a4a0-d1a1-4da6-8c47-a59cc5b131a6)

- Upload the nycpayroll_2020.csv file in **dirhistory**;
  ![DIRHISTORYFILES](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/62390ce5-0fbf-40e9-968a-bd06afa1884d)


2. Create an Azure Data Factory Resource
3. Create a SQL Database named **db__nycpayroll**
4. Create a Synapse Analytics workspace
5. Create the summary table(with dedicated pool) in Synapse Analytics workspace
   
![AZURE_SYNAPSE_TABLE_POOL](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/2cce2cf1-23ca-4197-9e6b-c3576fb20098)

6. Create master data tables and payroll transaction tables in SQL DB
   
![SQLDBRESOURCE_TABLES_PROOF](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/39b86a1b-aa9b-4224-854a-63c89f49452e)

## Step 2: Create Linked Services

1. Create Linked Service to the Azure Data Lake that contains the data files;
![LINKED_SERVICE_STORAGE_PROOF](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/8fa749cf-0fcb-4027-82f9-69cf1300469f)

2. Create a Linked Service to SQL Database;
![LINKED_SERVICE_SQLDATABASE_PROOF](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/f42dcb4f-9959-41c3-a84a-743d70c58e41)

3. Create a Linked Service to Azure Synapse Analytics.
![LINKED_SERVICE_SYNAPSE_WORKSPACE_PROOF](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/e8baf28c-73ca-45d0-839e-c1d7479e1da2)

## Step 3: Create Datasets in Azure Data Factory

1. Create the datasets for the files from Azure Data Lake Gen2
2. Create the datasets for the tables from SQL DB;
3. Create the datasets for destination table in Synapse Analytics
![NYC_PAYROLL_SUMMARY](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/6666448a-4b17-4aca-9e18-8d83ef849f71)

## Step 4: Create Data Flows

Create data flow to load 2020 Payroll data from Azure DataLake Gen2 storage to SQL db table created earlier. Repeat the same process to add data flow to load data for each file in Azure DataLake to the corresponding SQL DB tables.

![DATAFLOW_PAYROLL2021](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/e60fbefc-b82b-4d30-b4bc-c70228adbf06)

## Step 5: Data Aggregation and Parametrization

Extract the 2021 year data and historical data, merge, aggregate and store it in DataLake staging area which will be used by Synapse Analytics external table. The aggregation will be on Agency Name, Fiscal Year and TotalPaid.

- Create new data flow and name it Dataflow Summary
- Add source as payroll 2020 data from SQL DB
- Add another source as payroll 2021 data from SQL DB
- Create a new Union activity and select both payroll datasets as the source
- Make sure to do any source to target mappings if required. This can be done by adding a Select activity before Union
- After Union, add a Filter activity, go to Expression builder
  - Create a parameter named- dataflow_param_fiscalyear and give value 2020 or 2021
  - Include expression to be used for filtering: toInteger(FiscalYear) >=$dataflow_param_fiscalyear
- Now, choose Derived Column after filter
  - Name the column: TotalPaid
  - Add following expression: RegularGrossPaid + TotalOTPaid+TotalOtherPay
- Add an Aggregate activity to the data flow next to the TotalPaid activity
  - Under Group by, select AgencyName and FiscalYear
  - Set the expression to sum(TotalPaid)
- Add a Sink activity after the Aggregate
  - Select the sink as summary table created in SQL db
  - In Settings, tick Truncate table
- Add another Sink activity, this will create two sinks after Aggregate
  - Select the sink as summary table created in Azure Synapse Analytics
  - In Settings, tick Truncate table
 
![DATAFLOW_SUMMARY_PROOF](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/96254665-276a-4fe9-bcec-5a358deb32e0)

## Step 6: Pipeline Creation

- We will create a pipeline to load data from Azure DataLake Gen2 storage in SQL db for individual datasets, perform aggregations and store the summary results back into SQL db and Azure Synapse destination table;

![PIPELINE_PROOF](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/7098b1a9-d7f1-430e-bdf9-f5efb3a8161e)
  
- Trigger and Monitor Pipeline

![SCREENSHOT_PIPELINE_RUN_PROOF](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/291cd7f7-b964-427a-8866-b7256ec82d7a)

- Verify Pipeline
  
![QUERY_SUMMARY_SYNAPSE_PROOF](https://github.com/axentecristina/Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics/assets/48519726/6d49ffc0-763b-45e6-8fb9-ba3698e3b5b1)

