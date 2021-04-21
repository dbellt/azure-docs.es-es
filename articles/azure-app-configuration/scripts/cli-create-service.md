---
title: 'Ejemplo de script de la CLI de Azure: creación de un almacén de Azure App Configuration'
titleSuffix: Azure App Configuration
description: Creación de un almacén de Azure App Configuration mediante un script de ejemplo de la CLI de Azure. Consulte los vínculos de los artículos de referencia a comandos que se utilizan en el script.
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.topic: sample
ms.date: 01/24/2020
ms.author: alkemper
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4a341e9772c67da7781a5cec92f286c037c55ce9
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768964"
---
# <a name="create-an-azure-app-configuration-store"></a>Creación de un almacén de Azure App Configuration

Este script de ejemplo crea una nueva instancia de Azure App Configuration en un nuevo grupo de recursos.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial requiere la versión 2.0 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="sample-script"></a>Script de ejemplo

```azurecli-interactive
#!/bin/bash

appConfigName=myTestAppConfigStore
#resource name must be lowercase
myAppConfigStoreName=${appConfigName,,}
myResourceGroupName=$appConfigName"Group"

# Create resource group 
az group create --name $myResourceGroupName --location eastus

# Create the Azure AppConfig Service resource and query the hostName
appConfigHostname=$(az appconfig create \
  --name $myAppConfigStoreName \
  --location eastus \
  --resource-group $myResourceGroupName \
  --query endpoint \
  --sku free \
  -o tsv
  )

# Get the AppConfig connection string 
appConfigConnectionString=$(az appconfig credential list \
--resource-group $myResourceGroupName \
--name $myAppConfigStoreName \
--query "[?name=='Secondary Read Only'] .connectionString" -o tsv)

# Echo the connection string for use in your application
echo "$appConfigConnectionString"
```

Anote el nombre real generado para el nuevo grupo de recursos. Usará ese nombre de grupo de recursos cuando quiera eliminar todos los recursos del grupo.

[!INCLUDE [cli-script-cleanup](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos y un almacén de App Configuration. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az appconfig create](/cli/azure/appconfig#az_appconfig_create) | Crea un recurso del almacén de App Configuration. |
| [az appconfig credential list](/cli/azure/appconfig/credential#az_appconfig_credential_list) | Enumera las claves de acceso de un almacén de App Configuration. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Puede encontrar ejemplos de script adicionales de la CLI y App Configuration en [Azure App Configuration: ejemplos de la CLI](../cli-samples.md).
