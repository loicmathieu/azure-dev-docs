---
title: The Azure development flow
description: An overview of the Azure cloud development cycle, which involves provisioning (creating and configuring), coding, testing, deployment, and management of Azure resources.
ms.date: 11/29/2022
ms.topic: conceptual
ms.custom: devx-track-python, py-fresh-zinc
---

# The Azure development flow: provision, code, test, deploy, and manage

[Previous article: provisioning, accessing, and managing resources](cloud-development-provisioning.md)

Now that you understand Azure's model of services and resources, you can understand the overall flow of developing cloud applications with Azure: **provision**, **code**, **test**, **deploy**, and **manage**.

| Step | Primary tools | Activities |
| --- | --- | --- |
| Provision | Azure CLI, Azure portal, VS Code Azure Tools extensions, Cloud Shell, Python scripts using Azure SDK management libraries | Create resource groups and create resources in those groups; configure resources to be ready for use from app code and/or ready to receive Python code in deployments. |
| Code | Code editor (such as Visual Studio Code and PyCharm), Azure SDK client libraries, reference documentation | Write Python code using the Azure SDK client libraries to interact with provisioned resources. |
| Test | Python runtime, debugger | Run Python code locally against active cloud resources (typically dev or test resources rather than production resources). The code itself isn't yet hosted on Azure, which helps you debug and iterate quickly. |
| Deploy | VS Code, Azure CLI, GitHub Actions, Azure Pipelines | Once code has been tested locally, deploy it to an appropriate Azure hosting service where the code itself can run in the cloud. Deployed code typically runs against staging or production resources. |
| Manage | Azure CLI, Azure portal, VS Code, Python scripts, Azure Monitor | Monitor app performance and responsiveness, make adjustments in production environment, migrate improvements back to dev environment for the next round of provisioning and development. |

## Step 1: Provision and configure resources

As described in the [previous article of this series](cloud-development-provisioning.md), the first step in developing any application is to provision and configure the resources that make up the target environment for your application.

Provisioning begins by creating a resource group in a suitable Azure region. You can create a resource group through the Azure portal, VS Code with Azure Tools extensions, the Azure CLI, or with a custom script that uses the Azure SDK management libraries (or REST API).

Within that resource group, you then provision and configure the individual resources you need, again using the portal, VS Code, the CLI, or the Azure SDK. (Again, review the [Azure developer's guide](/azure/guides/developer/azure-developer-guide) for an overview of available resource types.)

Configuration includes setting access policies that control what identities (service principals and/or application IDs) are able to access those resources. Access policies are managed through Azure [Role-Based Access Control (RBAC)](/azure/role-based-access-control/overview); some services have more specific access controls as well. As a cloud developer working with Azure, make sure to familiarize yourself with Azure RBAC because you use it with just about any resource that has security concerns.

For most application scenarios, you typically create provisioning scripts with the Azure CLI and/or Python code using the Azure SDK management libraries. Such scripts describe the totality of your application's resource needs (essentially defining the custom cloud computer to which you're deploying the application). A script enables you to easily recreate the same set of resources within different environment like development, test, staging, and production. When you automate, you can avoid manually performing many repeated steps in Azure portal or VS Code. Such scripts also make it easy to provision an environment in a different region, or to use different resource groups. If you also maintain these scripts in source control repositories, you also have full auditing and change history.

## Step 2: Write your app code to use resources

Once you've provisioned the resources you need for your application, you write the application code to work with the run time aspects of those resources.

For example, in the provisioning step you might have created an Azure storage account, created a blob container within that account, and set access policies for the application on that container. This provisioning process is demonstrated in [Example - Provision Azure Storage](./sdk/examples/azure-sdk-example-storage.md). From your code, you can then authenticate with that storage account and then create, update, or delete blobs within that container. This run time process is demonstrated in [Example - Use Azure Storage](./sdk/examples/azure-sdk-example-storage.md). Similarly, you might have provisioned a database with a schema and appropriate permissions (as demonstrated in [Example - Provision a database](./sdk/examples/azure-sdk-example-database.md)), so that your application code can connect to the database and perform the usual create-read-update-delete queries.

App code typically uses environment variables to identify the names and URLs of the resources to use. Environment variables allow you to easily switch between cloud environments (dev, test, staging, and production) without any changes to the code. The various Azure services that host application code provide a means to define the necessary variables. For example, in Azure App Service (to host web apps) and Azure Functions (serverless compute for Azure), you define *application settings* through the Azure portal, VS Code, or Azure CLI, which then appear to your code as environment variables.

As a Python developer, you'll likely write your application code in Python using the Azure SDK client libraries for Python. That said, any independent part of a cloud application can be written in any supported language. If you're working on a team using multiple programming languages, it's possible that some parts of the application use Python, some JavaScript, some Java, and others C#.

Application code can use the Azure SDK management libraries to perform provisioning and management operations as needed. Provisioning scripts, similarly, can use the SDK client libraries to initialize resources with specific data, or perform housekeeping tasks on cloud resources even when those scripts are run locally.

## Step 3: Test and debug your app code locally

Developers typically like to test app code on their local workstations before deploying that code to the cloud. Testing app code locally means that you're typically accessing other resources that you've already provisioned in the cloud, such as storage, databases, and so forth. The difference is that you're not yet running the app code itself within a cloud service.

By running the code locally, you can also take full advantage of debugging features offered by tools such as Visual Studio Code and manage your code in a source control repository.

You don't need to modify your code at all for local testing: Azure fully supports local development and debugging using the same code you deploy to the cloud. Environment variables are again the key: in the cloud, your code can access the hosting resource's settings as environment variables. When you create those same environment variables locally, the same code runs without modification. This pattern works for authentication credentials, resource URLs, connection strings, and any number of other settings, making it easy to use resources in a development environment when running code locally and production resources once the code is deployed to the cloud.

## Step 4: Deploy your app code to Azure

Once you've tested your code locally, you're ready to deploy the code to the Azure resource that you've provisioned to host it. For example, if you're writing a Django web app, you either deploy that code to a virtual machine (where you provide your own web server) or to Azure App Service (which provides the web server for you). Once deployed, that code is running on the server rather than on your local machine, and can access all the Azure resources for which it's authorized.

As noted in the previous section, in typical development processes you first deploy your code to the resources you've provisioned in a development environment. After a round of testing, you deploy your code to resources in a staging environment, making the application available to your test team and perhaps preview customers. Once you're satisfied with the application's performance, you can deploy the code to your production environment. All of these deployments can also be automated through continuous integration and continuous deployment using Azure Pipelines and GitHub Actions.

However you do it, once the code is deployed to the cloud, it truly becomes a cloud application, running entirely on the server computers in Azure data centers.

## Step 5: Manage, monitor, and revise

After deployment, you want to make sure the application is performing as it should, responding to customer requests and using resources efficiently (and at the lowest cost). You can manage how Azure automatically scales your deployment as needed, and you can collect and monitor performance data with Azure portal, VS Code, the Azure CLI, or custom scripts written with the Azure SDK libraries. You can then make real-time adjustments to your provisioned resources to optimize performance, again using any of the same tools.

Monitoring gives you insight about how you might restructure your cloud application. For example, you may find that certain portions of a web app (such as a group of API endpoints) are used only occasionally in comparison to the primary parts. You could then choose to deploy those APIs separately as serverless Azure Functions. As functions, they have their own backing compute resources that don't compete with the main application but cost only pennies per month. Your main application then becomes more responsive to more customers without having to scale up to a higher-cost tier.

## Next steps

You're now familiar with the basic structure of Azure and the overall development flow: provision resources, write and test code, deploy the code to Azure, and then monitor and manage those resources.

The next step is to get familiar with the Azure SDK libraries for Python, which you'll be using in many parts of the flow.

> [!div class="nextstepaction"]
> [Configure your local Python dev environment for Azure >>>](./configure-local-development-environment.md)