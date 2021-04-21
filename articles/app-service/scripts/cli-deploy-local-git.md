---
title: 'CLI: Implementación desde un repositorio GIT local'
description: Aprenda a usar la CLI de Azure para automatizar la implementación y administración de la aplicación App Service. En este ejemplo se indica cómo implementar código desde un repositorio local de Git.
author: msangapu-msft
tags: azure-service-management
ms.assetid: 048f98aa-f708-44cb-9b9e-953f67dc6da8
ms.devlang: azurecli
ms.topic: sample
ms.date: 12/11/2017
ms.author: msangapu
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: d0be7b057a54c23b15fc4e898687d434c18a956e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787870"
---
# <a name="create-an-app-service-app-and-deploy-code-from-a-local-git-repository-using-azure-cli"></a>Creación de una aplicación de App Service e implementación de código desde un repositorio de Git local mediante la CLI de Azure

Este script de ejemplo crea una aplicación en App Service con sus recursos relacionados e implementa el código de la aplicación en un repositorio de Git local.


[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial requiere la versión 2.0 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/deploy-local-git/deploy-local-git.sh?highlight=3-5 "Create an app and deploy code from a local Git repository")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [`az group create`](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [`az appservice plan create`](/cli/azure/appservice/plan#az_appservice_plan_create) | Crea un plan de App Service, |
| [`az webapp create`](/cli/azure/webapp#az_webapp_create) | Crea una aplicación de App Service. |
| [`az webapp deployment user set`](/cli/azure/webapp/deployment/user#az_webapp_deployment_user_set) | Establece las credenciales de implementación a nivel de la cuenta para App Service. |
| [`az webapp deployment source config-local-git`](/cli/azure/webapp/deployment/source#az_webapp_deployment_source_config_local_git) | Crea una configuración de control de código fuente para un repositorio GIT local. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Puede encontrar ejemplos de script adicionales de la CLI de App Service en la [documentación de Azure App Service](../samples-cli.md).
