---
title: 'Ejemplo de script de la CLI de Azure: integración de App Service con Application Gateway | Microsoft Docs'
description: 'Ejemplo de script de la CLI de Azure: integración de App Service con Application Gateway'
services: appservice
documentationcenter: appservice
author: madsd
manager: ccompy
editor: ''
tags: azure-service-management
ms.assetid: 6c16d6f8-3c08-4a59-858e-684a2ceccb5f
ms.service: app-service
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: web
ms.date: 12/09/2019
ms.author: madsd
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: d30cc27fc3c546619e85bb9aabd0b31c10102e96
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787816"
---
# <a name="integrate-app-service-with-application-gateway-using-cli"></a>Integración de App Service con Application Gateway mediante la CLI

Este script de ejemplo crea una aplicación web de Azure App Service, una red virtual de Azure y una instancia de Application Gateway. A continuación, restringe el tráfico de la aplicación web para que solo se origine desde la subred de Application Gateway.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial requiere la versión 2.0.74 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/integrate-with-app-gateway/integrate-with-app-gateway.sh "Integrate with Application Gateway")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una aplicación de App Service, Cosmos DB y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [`az group create`](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [`az network vnet create`](/cli/azure/network/vnet#az_network_vnet_create) | Crea una red virtual. |
| [`az network public-ip create`](/cli/azure/network/public-ip#az_network_public_ip_create) | Crea una dirección IP pública. |
| [`az network public-ip show`](/cli/azure/network/public-ip#az_network_public_ip_show) | Muestra los detalles de una dirección IP pública. |
| [`az appservice plan create`](/cli/azure/appservice/plan#az_appservice_plan_create) | Crea un plan de App Service, |
| [`az webapp create`](/cli/azure/webapp#az_webapp_create) | Crea una aplicación web de App Service. |
| [`az webapp show`](/cli/azure/webapp#az_webapp_show) | Muestra los detalles de una aplicación web de App Service. |
| [`az webapp config access-restriction add`](/cli/azure/webapp/config/access-restriction#az_webapp_config_access_restriction_add) | Agrega una restricción de acceso a la aplicación web de App Service. |
| [`az network application-gateway create`](/cli/azure/network/application-gateway#az_network_application_gateway_create) | Crea una instancia de Application Gateway. |
| [`az network application-gateway http-settings update`](/cli/azure/network/application-gateway/http-settings#az_network-application-gateway-http_settings_update) | Actualiza la configuración HTTP de Application Gateway. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Puede encontrar ejemplos de script adicionales de la CLI de App Service en la [documentación de Azure App Service](../samples-cli.md).
