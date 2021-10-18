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

> You can set a certain location of an Azure IR, in which case the data movement or activity dispatch will happen in that specific region. If you choose to use the auto-resolve Azure IR which is the default, ADF will make a best effort to automatically detect your sink and source data store to choose the best location either in the same region if available or the closest one in the same geography for the Copy Activity. For anything else, it will use the IR in the Data Factory region. Azure Integration Runtime also has support for virtual networks.

