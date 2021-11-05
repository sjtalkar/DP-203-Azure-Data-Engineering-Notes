# Azure Data Factory creation requirements

To create Data Factory instances, the user account that you use to sign in to Azure must be a member of the contributor or owner role, or an administrator of the Azure subscription.

To create and manage Data Factory objects including datasets, linked services, pipelines, triggers, and integration runtimes, the following requirements must be met:

To create and manage child resources in the Azure portal, you must belong to the Data Factory Contributor role at the resource group level or above.
To create and manage resources with PowerShell or the SDK, the contributor role at the resource level or above is sufficient.
Data Factory Contributor role
When you are added as a member of this role, you have the following permissions:

Create, edit, and delete data factories and child resources including datasets, linked services, pipelines, triggers, and integration runtimes.
Deploy Resource Manager templates. Resource Manager deployment is the deployment method used by Data Factory in the Azure portal.
Manage App Insights alerts for a data factory.
At the resource group level or above, lets users deploy Resource Manager template.
Create support tickets.
If the Data Factory Contributor role does not meet your requirement, you can create your own custom role.


# Create linked services

Before you create a dataset, you must create a linked service to link your data store to the data factory. Linked services are much like connection strings, which define the connection information needed for Data Factory to connect to external resources. There are over 100 connectors that can be used to define a linked service.

A linked service in Data Factory can be defined using the **Copy Data Activity** in the ADF designer, or you can **create them independently** to point to a data store or a compute resources.

Alternatively you can programmatically define a linked service in the JSON format to be used via REST APIs or the SDK, using the following notation:
```{
    "name": "<Name of the linked service>",
    "properties": {
        "type": "<Type of the linked service>",
        "typeProperties": {
              "<data store or compute-specific type properties>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

# Manage integration runtimes

In Data Factory, an activity defines the action to be performed. A linked service defines a target data store or a compute service. An integration runtime provides the infrastructure for the activity and linked services.

Integration Runtime is referenced by the linked service or activity, and provides the compute environment where the activity **either runs on or gets dispatched from.** This way, the activity can be performed in the region closest possible to the target data store or compute service in the most performant way while meeting security and compliance needs.

In short, the Integration Runtime (IR) is the compute infrastructure used by Azure Data Factory. It provides the following data integration capabilities across different network environments, including:

- Data Flow: Execute a Data Flow in managed Azure compute environment.
- Data movement: Copy data across data stores in public network and data stores in private network (on-premises or virtual private network). It provides support for built-in connectors, format conversion, column mapping, and performant and scalable data transfer.
- Activity dispatch: Dispatch and monitor transformation activities running on a variety of compute services such as Azure Databricks, Azure HDInsight, Azure Machine Learning, Azure SQL Database, SQL Server, and more.
- SSIS package execution: Natively execute SQL Server Integration Services (SSIS) packages in a managed Azure compute environment.


Whenever an Azure Data Factory instance is created, a default Integration Runtime environment is created that supports operations on cloud data stores and compute services in public network. This can be viewed when the integration runtime is set to Auto-Resolve

Integration runtime types
Data Factory offers three types of Integration Runtime, and you should choose the type that best serve the data integration capabilities and network environment needs you are looking for. These three types are:

Azure
Self-hosted
Azure-SSIS

The following table describes the capabilities and network support for each of the integration runtime types:

INTEGRATION RUNTIME TYPES

| IR type     | Public Network                            | Private Network                 |   |   |
|-------------|-------------------------------------------|---------------------------------|---|---|
| Azure       | Data Flow Data movement Activity Dispatch |                                 |   |   |
| Self-hosted | Data movement Activity dispatch           | Data movement Activity dispatch |   |   |
| Azure-SSIS  | SSIS package execution                    | SSIS package execution          |   |   |



Determining which integration runtime to use
There are a range of factors that affect the Integration Runtime that you will use. The following is a guide that will help you select the right IR

### Copy activity
For the Copy activity, it requires source and sink linked services to define the direction of data flow. The following logic is used to determine which integration runtime instance is used to perform the copy:

- Copying between two cloud data sources: when both source and sink linked services are using Azure IR, ADF will use the regional Azure IR if you specified, or auto determine a location of Azure IR if you choose the auto-resolve IR (default) as described in Integration runtime location section.

- Copying between a cloud data source and a data source in private network: if either source or sink linked service points to a self-hosted IR, the copy activity is executed on that self-hosted Integration Runtime.

- Copying between two data sources in private network: both the source and sink Linked Service must point to the same instance of integration runtime, and that integration runtime is used to execute the copy Activity.

### Lookup and GetMetadata activity
The Lookup and GetMetadata activity is executed on the integration runtime associated to the data store linked service.

### Transformation activity
Each transformation activity has a target compute Linked Service, which points to an integration runtime. This integration runtime instance is where the transformation activity is dispatched from.

### Data Flow activity
Data Flow activity is executed on the integration runtime associated to it.


## Azure integration runtime
An Azure integration runtime is capable of:

- Running Data Flows in Azure
- Running Copy Activity between cloud data stores
- Dispatching the following transform activities in public network: Databricks Notebook/ Jar/ Python activity, HDInsight Hive activity, HDInsight Pig activity, HDInsight MapReduce activity, HDInsight Spark activity, HDInsight Streaming activity, Machine Learning Batch Execution activity, Machine Learning Update Resource activities, Stored Procedure activity, Data Lake Analytics U-SQL activity, .NET custom activity, Web activity, Lookup activity, and Get Metadata activity.

> You can set a certain location of an Azure IR, in which case the data movement or activity dispatch will happen in that specific region. If you choose to use the auto-resolve Azure IR which is the default, ADF will make a best effort to automatically detect your sink and source data store to choose the best location either in the same region if available or the closest one in the same geography for the Copy Activity. For anything else, it will use the IR in the Data Factory region. Azure Integration Runtime also has support for virtual networks


## Use virtual networks to secure Azure resources
Virtual networks allow secure communication between Azure services, or with servers that exist in on-premises network. Virtual networks or VNets are the fundamental building block for configuring private networks. It enables the secure communication between Azure resources, services on the internet, and with server on on-premises networks. Azure Data Factory may ingest data from an on-premises server, or a virtual machine that is hosted within Azure. To achieve this, a self-hosted integration runtime can be deployed on a server inside a virtual network. To restrict access, you should configure a Network Security Group (NSG) to only allow administrative access. When using Azure-SSIS Integration runtime, you are given the option to join a virtual network. Accepting this option enables Azure Data Factory to create network resources, an example of which is that a Network Security Group is automatically created by Azure Data Factory, and port 3389 is open to all traffic by default. Lock this down to ensure that only your administrators have access.


### [Transformation Types](https://docs.microsoft.com/en-us/learn/modules/code-free-transformation-scale/3-describe-transformation-types)



With decoupled storage and compute, when using Synapse SQL one can benefit from independent sizing of compute power irrespective of your storage needs. For serverless SQL pool scaling is done automatically, while for dedicated SQL pool one can:

Grow or shrink compute power, within a dedicated SQL pool, without moving data.
Pause compute capacity while leaving data intact, so you only pay for storage.
Resume compute capacity during operational hours.


Serverless SQL pool allows you to query your data lake files, while dedicated SQL pool allows you to query and ingest data from your data lake files. When data is ingested into dedicated SQL pool, the data is sharded into distributions to optimize the performance of the system. You can choose which sharding pattern to use to distribute the data when you define the table. These sharding patterns are supported:

Hash
Round Robin
Replicate


The serverless SQL pool Control node utilizes Distributed Query Processing (DQP) engine to optimize and orchestrate distributed execution of user query by splitting it into smaller queries that will be executed on Compute nodes. Each small query is called task and represents distributed execution unit. It reads file(s) from storage, joins results from other tasks, groups, or orders data retrieved from other tasks.

The Compute nodes store all user data in Azure Storage and run the parallel queries. The Data Movement Service (DMS) is a system-level internal service that moves data across the nodes as necessary to run queries in parallel and return accurate results.

With decoupled storage and compute, when using Synapse SQL one can benefit from independent sizing of compute power irrespective of your storage needs. For serverless SQL pool scaling is done automatically, while for dedicated SQL pool one can:

- Grow or shrink compute power, within a dedicated SQL pool, without moving data.
- Pause compute capacity while leaving data intact, so you only pay for storage.
- Resume compute capacity during operational hours.


### Serverless SQL pool 

It is a distributed data processing system, built for large-scale data and computational functions. Serverless SQL pool enables you to analyze your Big Data in seconds to minutes, depending on the workload. Serverless SQL pool is serverless, hence there's no infrastructure to setup or clusters to maintain. A default endpoint for this service is provided within every Azure Synapse workspace, so you can start querying data as soon as the workspace is created.
There is no charge for resources reserved, you are only being charged for the data processed by queries you run, hence this model is a true pay-per-use model.

 It is suitable for the following scenarios:

- Basic discovery and exploration - Quickly reason about the data in various formats (Parquet, CSV, JSON) in your data lake, so you can plan how to extract insights from it.
- Logical data warehouse – Provide a relational abstraction on top of raw or disparate data without relocating and transforming data, allowing always up-to-date view of your data. Learn more about creating logical data warehouse.
- Data transformation - Simple, scalable, and performant way to transform data in the lake using T-SQL, so it can be fed to BI and other tools, or loaded into a relational data store (Synapse SQL databases, Azure SQL Database, etc.).

- Databases - serverless SQL pool endpoint can have multiple databases.
- Schemas - Within a database, there can be one or many object ownership groups called schemas.
- Views, stored procedures, inline table value functions
- External resources – data sources, file formats, and tables

Supported T-SQL:

- Full SELECT surface area is supported, including a majority of SQL functions
- CETAS - CREATE EXTERNAL TABLE AS SELECT
- DDL statements related to views and security only


Serverless SQL pool has no local storage, only metadata objects are stored in databases. Therefore, T-SQL related to the following concepts isn't supported:

- Tables
- Triggers
- Materialized views
- DDL statements other than ones related to views and security
- DML statements

Azure Active Directory integration and multi-factor authentication
Serverless SQL pool enables you to centrally manage identities of database user and other Microsoft services with Azure Active Directory integration. This capability simplifies permission management and enhances security. Azure Active Directory (Azure AD) supports multi-factor authentication (MFA) to increase data and application security while supporting a single sign-on process.

### DEDICATED SQL POOL

Azure Synapse Analytics is an analytics service that brings together enterprise data warehousing and Big Data analytics. Dedicated SQL pool (formerly SQL DW) refers to the enterprise data warehousing features that are available in Azure Synapse Analytics. 
Once your dedicated SQL pool is created, you can import big data with simple PolyBase T-SQL queries, and then use the power of the distributed query engine to run high-performance analytics.

Dedicated SQL pool (formerly SQL DW) stores data in relational tables with columnar storage. This format significantly reduces the data storage costs, and improves query performance. Once data is stored, you can run analytics at massive scale. Compared to traditional database systems, analysis queries finish in seconds instead of minutes, or hours instead of days.


| Dedicated                                                      | Serverless                                                                                                          | From                                                                               |
|----------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| Tables                                                         | Yes                                                                                                                 | No, serverless model can query only external data placed on Azure Storage          |
| Views                                                          | Yes. Views can use query language elements that are available in dedicated model.                                   | Yes. Views can use query language elements that are available in serverless model. |
| Schemas                                                        | Yes                                                                                                                 | Yes                                                                                |
| Temporary tables                                               | Yes                                                                                                                 | No                                                                                 |
| Procedures                                                     | Yes                                                                                                                 | Yes                                                                                |
| Functions                                                      | Yes                                                                                                                 | Yes, only inline table-valued functions.                                           |
| Triggers                                                       | No                                                                                                                  | No                                                                                 |
| External tables                                                | Yes. See supported data formats.                                                                                    | Yes. See supported data formats.                                                   |
| Caching queries                                                | Yes, multiple forms (SSD-based caching, in-memory, resultset caching). In addition, Materialized View are supported | No                                                                                 |
| Table variables                                                | No, use temporary tables                                                                                            | No                                                                                 |
| Table distribution                                             | Yes                                                                                                                 | No                                                                                 |
| Table indexes                                                  | Yes                                                                                                                 | No                                                                                 |
| Table partitions                                               | Yes                                                                                                                 | No                                                                                 |
| Statistics                                                     | Yes                                                                                                                 | Yes                                                                                |
| Workload management, resource classes, and concurrency control | Yes                                                                                                                 | No                                                                                 |
| Cost control                                                   | Yes, using scale-up and scale-down actions.                                                                         | Yes, using the Azure portal or T-SQL procedure.                                    |
