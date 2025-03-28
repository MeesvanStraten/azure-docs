---
title: Create your first containerized Azure Functions on Azure Container Apps
description: Get started with Azure Functions on Azure Container Apps by deploying your first function app from a Linux image in a container registry.
ms.date: 03/07/2024
ms.topic: quickstart
ms.custom: build-2023, devx-track-azurecli, devx-track-extended-java, devx-track-js, devx-track-python, linux-related-content
zone_pivot_groups: programming-languages-set-functions
---

# Create your first containerized functions on Azure Container Apps 

In this article, you create a function app running in a Linux container and deploy it to an Azure Container Apps environment from a container registry. By deploying to Container Apps, you are able to integrate your function apps into cloud-native microservices. For more information, see [Azure Container Apps hosting of Azure Functions](functions-container-apps-hosting.md).

This article shows you how to use Functions tools to create your first function running in a Linux container, verify the functions locally, and then deploy the container to a Container Apps environment. 

Completing this quickstart incurs a small cost of a few USD cents or less in your Azure account, which you can minimize by [cleaning-up resources](#clean-up-resources) when you're done.

[!INCLUDE [functions-container-apps-preview](../../includes/functions-container-apps-preview.md)]

[!INCLUDE [functions-create-container-registry](../../includes/functions-create-container-registry.md)]

## Create supporting Azure resources for your function

Before you can deploy your container to Azure, you need to create three resources:

* A [resource group](../azure-resource-manager/management/overview.md), which is a logical container for related resources.
* A [Storage account](../storage/common/storage-account-create.md), which is used to maintain state and other information about your functions.
* An Azure Container Apps environment with a Log Analytics workspace.

Use the following commands to create these items. 

1. If you haven't done already, sign in to Azure.

    The [`az login`](/cli/azure/reference-index#az-login) command signs you into your Azure account. Use `az account set` when you have more than one subscription associated with your account.

1. Run the following command to update the Azure CLI to the latest version:
     
    ```azurecli
    az upgrade
    ```
   
    If your version of Azure CLI isn't the latest version, an installation begins. The manner of upgrade depends on your operating system. You can proceed after the upgrade is complete.

1. Run the following commands that upgrade the Azure Container Apps extension and register namespaces required by Container Apps:

    ```azurecli
    az extension add --name containerapp --upgrade -y
    az provider register --namespace Microsoft.Web 
    az provider register --namespace Microsoft.App 
    az provider register --namespace Microsoft.OperationalInsights 
    ```

1. Create a resource group named `AzureFunctionsContainers-rg`.

    
    ```azurecli
    az group create --name AzureFunctionsContainers-rg --location eastus
    ```
 
    This [`az group create`](/cli/azure/group#az-group-create) command creates a resource group in the East US region. If you instead want to use a region near you, using an available region code returned from the [az account list-locations](/cli/azure/account#az-account-list-locations) command. You must modify subsequent commands to use your custom region instead of `eastus`. You can only use regions [currently supported in the preview](./functions-container-apps-hosting.md#considerations-for-container-apps-hosting).

1. Create Azure Container App environment with workload profiles enabled.

    ```azurecli
    az containerapp env create --name MyContainerappEnvironment --enable-workload-profiles --resource-group AzureFunctionsContainers-rg --location eastus
    ```
    This command can take up to 10 minutes to complete.

1. Create a general-purpose storage account in your resource group and region.

    ```azurecli
    az storage account create --name <STORAGE_NAME> --location eastus --resource-group AzureFunctionsContainers-rg --sku Standard_LRS
    ```

    The [`az storage account create`](/cli/azure/storage/account#az-storage-account-create) command creates the storage account. 

    In the previous example, replace `<STORAGE_NAME>` with a name that is appropriate to you and unique in Azure Storage. Storage names must contain 3 to 24 characters numbers and lowercase letters only. `Standard_LRS` specifies a general-purpose account [supported by Functions](storage-considerations.md#storage-account-requirements).

1. Before you proceed, use this command to verify that your environment is ready.

   ```azurecli
   az containerapp env show -n MyContainerappEnvironment -g AzureFunctionsContainers-rg
   ```

## Create and configure a function app on Azure with the image

A function app on Azure manages the execution of your functions in your Azure Container Apps environment. In this section, you use the Azure resources from the previous section to create a function app from an image in a container registry in a Container Apps environment. You also configure the new environment with a connection string to the required Azure Storage account.

Use the [`az functionapp create`](/cli/azure/functionapp#az-functionapp-create) command to create a function app in the new managed environment backed by Azure Container Apps:

# [Azure Container Registry](#tab/acr)
::: zone pivot="programming-language-csharp"  
```azurecli
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --workload-profile-name "Consumption" --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime dotnet-isolated --image <LOGIN_SERVER>/azurefunctionsimage:v1.0.0 --registry-server <LOGIN_SERVER> --registry-username <REGISTRY_NAME> --registry-password <ADMIN_PASSWORD> 
```
::: zone-end  
::: zone pivot="programming-language-javascript" 
```console
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --workload-profile-name "Consumption" --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime node --image <LOGIN_SERVER>/azurefunctionsimage:v1.0.0 --registry-server <LOGIN_SERVER> --registry-username <REGISTRY_NAME> --registry-password <ADMIN_PASSWORD> 
```
::: zone-end  
::: zone pivot="programming-language-java" 
```console
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --workload-profile-name "Consumption" --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime java --image <LOGIN_SERVER>/azurefunctionsimage:v1.0.0 --registry-server <LOGIN_SERVER> --registry-username <REGISTRY_NAME> --registry-password <ADMIN_PASSWORD> 
```
::: zone-end 
::: zone pivot="programming-language-powershell"  
```console
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --workload-profile-name "Consumption" --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime powershell --image <LOGIN_SERVER>/azurefunctionsimage:v1.0.0 --registry-server <LOGIN_SERVER> --registry-username <REGISTRY_NAME> --registry-password <ADMIN_PASSWORD> 
```
::: zone-end  
::: zone pivot="programming-language-python"  
```console
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --workload-profile-name "Consumption" --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime python --image <LOGIN_SERVER>/azurefunctionsimage:v1.0.0 --registry-server <LOGIN_SERVER> --registry-username <REGISTRY_NAME> --registry-password <ADMIN_PASSWORD> 
```
::: zone-end  
::: zone pivot="programming-language-typescript"  
```console
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --workload-profile-name "Consumption" --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime node --image <LOGIN_SERVER>/azurefunctionsimage:v1.0.0 --registry-server <LOGIN_SERVER> --registry-username <REGISTRY_NAME> --registry-password <ADMIN_PASSWORD> 
```
::: zone-end

In the [`az functionapp create`](/cli/azure/functionapp#az-functionapp-create) command, the `--environment` parameter specifies the Container Apps environment and the `--image` parameter specifies the image to use for the function app. In this example, replace `<STORAGE_NAME>` with the name you used in the previous section for the storage account. Also, replace `<APP_NAME>` with a globally unique name appropriate to you, `<LOGIN_SERVER>` with your fully qualified Container Registry server, `<REGISTRY_NAME>` with your registry name for the login, and `<ADMIN_PASSWORD>` with the password to your admin account. 

> [!IMPORTANT]
> The admin account username and password are important credentials. Make sure to store them securely and never in an accessible location like a public repository.

# [Docker Hub](#tab/docker)
::: zone pivot="programming-language-csharp"  
```azurecli
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --workload-profile-name "Consumption" --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime dotnet-isolated --image <DOCKER_ID>/azurefunctionsimage:v1.0.0 
```
::: zone-end  
::: zone pivot="programming-language-javascript" 
```azurecli
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --workload-profile-name "Consumption" --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime node --image <DOCKER_ID>/azurefunctionsimage:v1.0.0
```
::: zone-end  
::: zone pivot="programming-language-java" 
```azurecli
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --workload-profile-name "Consumption" --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime java --image <DOCKER_ID>/azurefunctionsimage:v1.0.0
```
::: zone-end 
::: zone pivot="programming-language-powershell"  
```azurecli
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --workload-profile-name "Consumption" --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime powershell --image <DOCKER_ID>/azurefunctionsimage:v1.0.0
```
::: zone-end  
::: zone pivot="programming-language-python"  
```azurecli
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --workload-profile-name "Consumption" --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime python --image <DOCKER_ID>/azurefunctionsimage:v1.0.0
```
::: zone-end  
::: zone pivot="programming-language-typescript"  
```azurecli
az functionapp create --name <APP_NAME> --storage-account <STORAGE_NAME> --environment MyContainerappEnvironment --resource-group AzureFunctionsContainers-rg --functions-version 4 --runtime node --image <DOCKER_ID>/azurefunctionsimage:v1.0.0
```
::: zone-end

In the [`az functionapp create`](/cli/azure/functionapp#az-functionapp-create) command, the `--environment` parameter specifies the Container Apps environment and the `--image` parameter specifies the image to use for the function app. In this example, replace `<STORAGE_NAME>` with the name you used in the previous section for the storage account. Also, replace `<APP_NAME>` with a globally unique name appropriate to you and `<DOCKER_ID>` with your Docker Hub account ID. If you are using a private registry, you also need to supply  `--registry-username`, `--registry-password`, and `--registry-server`. 

---

Specifying `--workload-profile-name "Consumption"` creates your app in an environment using the default `Consumption` workload profile, which costs the same as running in a Container Apps Consumption plan. When you first create the function app, it pulls the initial image from your registry. 
  
<!--- CI/CD isn't yet supported: 
You can also [Enable continuous deployment](./functions-how-to-custom-container.md#enable-continuous-deployment-to-azure) to Azure from Docker Hub.-->

At this point, your functions are running in a Container Apps environment, with the required application settings already added. When you need to add other settings in your functions app, you can do this in the standard way for Functions. For more information, see [Use application settings](functions-how-to-use-azure-function-app-settings.md#settings).

>[!TIP] 
> When you make subsequent changes to your function code, you need to rebuild the container, republish the image to the registry, and update the function app with the new image version. For more information, see [Update an image in the registry](functions-how-to-custom-container.md#update-an-image-in-the-registry)

[!INCLUDE [functions-container-verify-azure](../../includes/functions-container-verify-azure.md)]

The request URL should look something like this:

::: zone pivot="programming-language-java,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python"  
`http://myacafunctionapp.kindtree-796af82b.eastus.azurecontainerapps.io/api/httpexample?name=functions`
::: zone-end  
::: zone pivot="programming-language-csharp"  
`http://myacafunctionapp.kindtree-796af82b.eastus.azurecontainerapps.io/api/httpexample`
::: zone-end

## Clean up resources

If you want to continue working with Azure Function using the resources you created in this article, you can leave all those resources in place. 

When you're done working with this function app deployment, delete the `AzureFunctionsContainers-rg` resource group to clean up all the resources in that group:

```azurecli
az group delete --name AzureFunctionsContainers-rg
```

## Next steps

> [!div class="nextstepaction"]  
> [Azure Container Apps hosting of Azure Functions](./functions-container-apps-hosting.md)   
> [!div class="nextstepaction"]  
> [Working with containers and Azure Functions](./functions-how-to-custom-container.md)  
> [!div class="nextstepaction"]  
> [Help make the experience better](https://microsoft.qualtrics.com/jfe/form/SV_byFGULLJlKPh9Xw)  
