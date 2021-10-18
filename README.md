# Azure Data Factory security

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
