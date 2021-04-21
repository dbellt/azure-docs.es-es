---
title: Creación de una aplicación web con autenticación de SignalR Service y GitHub
description: 'Ejemplo de script de la CLI de Azure: creación de una aplicación web que usa autenticación de SignalR Service y GitHub'
author: sffamily
ms.service: signalr
ms.devlang: azurecli
ms.topic: sample
ms.date: 04/22/2018
ms.author: zhshang
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: c46ac350c1bf7f7593f334a9a7fa5df1af7b19d8
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787330"
---
# <a name="create-a-web-app-that-uses-signalr-service-and-github-authentication"></a>Creación de una aplicación web que usa autenticación de SignalR Service y GitHub

Este script de ejemplo crea un nuevo recurso de Azure SignalR Service, que se utiliza para insertar las actualizaciones de contenido en tiempo real en los clientes. Este script también agrega una nueva aplicación web y un nuevo plan de App Service para hospedar la aplicación web de ASP.NET que usa la instancia de SignalR Service. La aplicación web está configurada con la configuración de aplicación para conectarse al nuevo recurso de SignalR Service y se autentica con [autenticación de GitHub](https://developer.github.com/v3/guides/basics-of-authentication/). La aplicación web también está configurada para usar un origen de implementación de repositorio de Git local.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 

## <a name="sample-script"></a>Script de ejemplo

Este script usa la extensión *signalr* con la CLI de Azure. Ejecute el siguiente comando para instalar la extensión *signalr* para la CLI de Azure antes de usar este script de ejemplo:

```azurecli-interactive
#!/bin/bash

#========================================================================
#=== Update these values based on your desired deployment username    ===
#=== and password.                                                    ===
#========================================================================
deploymentUser=<Replace with your desired username>
deploymentUserPassword=<Replace with your desired password>

#========================================================================
#=== Update these values based on your GitHub OAuth App registration. ===
#========================================================================
GitHubClientId=<Replace with your GitHub OAuth app Client ID>
GitHubClientSecret=<Replace with your GitHub OAuth app Client Secret>


# Generate a unique suffix for the service name
let randomNum=$RANDOM*$RANDOM

# Generate unique names for the SignalR service, resource group, 
# app service, and app service plan
SignalRName=SignalRTestSvc$randomNum
#resource name must be lowercase
mySignalRSvcName=${SignalRName,,}
myResourceGroupName=$SignalRName"Group"
myWebAppName=SignalRTestWebApp$randomNum
myAppSvcPlanName=$myAppSvcName"Plan"

# Create resource group 
az group create --name $myResourceGroupName --location eastus

# Create the Azure SignalR Service resource
az signalr create \
  --name $mySignalRSvcName \
  --resource-group $myResourceGroupName \
  --sku Standard_S1 \
  --unit-count 1 \
  --service-mode Default

# Create an App Service plan.
az appservice plan create --name $myAppSvcPlanName --resource-group $myResourceGroupName --sku FREE

# Create the Web App
az webapp create --name $myWebAppName --resource-group $myResourceGroupName --plan $myAppSvcPlanName  

# Get the SignalR primary connection string
primaryConnectionString=$(az signalr key list --name $mySignalRSvcName \
  --resource-group $myResourceGroupName --query primaryConnectionString -o tsv)

#Add an app setting to the web app for the SignalR connection
az webapp config appsettings set --name $myWebAppName --resource-group $myResourceGroupName \
  --settings "Azure:SignalR:ConnectionString=$primaryConnectionString" 

#Add app settings to use with GitHub authentication
az webapp config appsettings set --name $myWebAppName --resource-group $myResourceGroupName \
  --settings "GitHubClientId=$GitHubClientId" 
az webapp config appsettings set --name $myWebAppName --resource-group $myResourceGroupName \
  --settings "GitHubClientSecret=$GitHubClientSecret" 

# Add the desired deployment user name and password
az webapp deployment user set --user-name $deploymentUser --password $deploymentUserPassword

# Configure Git deployment and note the deployment URL in the output
az webapp deployment source config-local-git --name $myWebAppName --resource-group $myResourceGroupName \
  --query [url] -o tsv
```

Anote el nombre real generado para el nuevo grupo de recursos. Se mostrará en la salida. Usará ese nombre de grupo de recursos cuando quiera eliminar todos los recursos del grupo.

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Cada comando de la tabla crea un vínculo a documentación específica del comando. Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az signalr create](/cli/azure/signalr#az_signalr_create) | Crea un recurso de Azure SignalR Service. |
| [az signalr key list](/cli/azure/signalr/key#az_signalr_key_list) | Muestra las claves, que usará la aplicación al insertar actualizaciones de contenido en tiempo real con SignalR. |
| [az appservice plan create](/cli/azure/appservice/plan#az_appservice_plan_create) | Crea un plan de Azure App Service para hospedar aplicaciones web. |
| [az webapp create](/cli/azure/webapp#az_webapp_create) | Crea una aplicación web de Azure mediante el plan de hospedaje de App Service. |
| [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) | Agrega la nueva configuración de aplicación de la aplicación web. Esta configuración de aplicación se usa para almacenar la cadena de conexión de SignalR y los secretos de aplicación de GitHub OAuth. |
| [az webapp deployment user set](/cli/azure/webapp/deployment/user#az_webapp_deployment_user_set) | Actualiza las credenciales de implementación. |
| [az webapp deployment source config-local-git](/cli/azure/webapp/deployment/source#az_webapp_deployment_source_config_local_git) | Obtiene una dirección URL de un punto de conexión de repositorio de Git para clonarla e insertarla en una implementación de aplicación web. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Se pueden encontrar ejemplos de scripts adicionales de la CLI de Azure SignalR Service en la [documentación de Azure SignalR Service](../signalr-reference-cli.md).
