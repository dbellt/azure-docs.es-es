---
title: 'CLI: Implementación en el espacio de ensayo'
description: Aprenda a usar la CLI de Azure para automatizar la implementación y administración de la aplicación App Service. En este ejemplo se muestra cómo implementar el código en un espacio de ensayo.
author: msangapu-msft
tags: azure-service-management
ms.assetid: 2b995dcd-e471-4355-9fda-00babcdb156e
ms.devlang: azurecli
ms.topic: sample
ms.date: 12/11/2017
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: bba2c3d6faf99962ce4e687832c4d26dcd3dc671
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787924"
---
# <a name="create-an-app-service-app-and-deploy-code-to-a-staging-environment-using-azure-cli"></a>Creación de una aplicación de App Service e implementación de código en un entorno de ensayo mediante la CLI de Azure

Este script de ejemplo crea una aplicación en App Service con una ranura de implementación adicional denominada "ensayo" y, a continuación, implementa una aplicación de ejemplo en la ranura "ensayo".

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]


[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial requiere la versión 2.0 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/deploy-deployment-slot/deploy-deployment-slot.sh "Create an app and deploy code to a staging environment")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [`az group create`](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [`az appservice plan create`](/cli/azure/appservice/plan#az_appservice_plan_create) | Crea un plan de App Service, |
| [`az webapp create`](/cli/azure/webapp#az_webapp_create) | Crea una aplicación de App Service. |
| [`az webapp deployment slot create`](/cli/azure/webapp/deployment/slot#az_webapp_deployment_slot_create) | Crea una ranura de implementación. |
| [`az webapp deployment source config`](/cli/azure/webapp/deployment/source#az_webapp_deployment_source_config) | Asocia una aplicación de App Service con un repositorio de Git o Mercurial. |
| [`az webapp deployment slot swap`](/cli/azure/webapp/deployment/slot#az_webapp_deployment_slot_swap) | Cambia la ranura de implementación especificada a producción. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Puede encontrar ejemplos de script adicionales de la CLI de App Service en la [documentación de Azure App Service](../samples-cli.md).
