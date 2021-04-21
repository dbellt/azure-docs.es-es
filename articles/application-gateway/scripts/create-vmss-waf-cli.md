---
title: 'Ejemplo de script de la CLI de Azure: restringir el tráfico web | Microsoft Docs'
description: 'Ejemplo de script de la CLI de Azure: cree una puerta de enlace de aplicaciones con un firewall de aplicaciones web y un conjunto de escalado de máquinas virtuales que use reglas OWASP para restringir el tráfico.'
services: application-gateway
documentationcenter: networking
author: vhorne
tags: azure-resource-manager
ms.service: application-gateway
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 01/29/2018
ms.author: victorh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 6e20a4324f46925b8f83d7519c481a1d5bfc06a9
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789310"
---
# <a name="restrict-web-traffic-using-the-azure-cli"></a>Restringir el tráfico web mediante la CLI de Azure

Este script crea una puerta de enlace de aplicaciones con un firewall de aplicaciones web que usa un conjunto de escalado de máquinas virtuales para los servidores back-end. El firewall de aplicaciones web restringe el tráfico web basado en reglas OWASP. Después de ejecutar el script, se puede probar la puerta de enlace de aplicaciones mediante la dirección IP pública.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/application-gateway/create-vmss/create-vmss-waf.sh "Create application gateway")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Ejecute el siguiente comando para quitar el grupo de recursos, la puerta de enlace de aplicaciones y todos los recursos relacionados.

```azurecli-interactive 
az group delete --name myResourceGroupAG --yes
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear la implementación. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) | Crea una red virtual. |
| [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) | Crea una subred en una red virtual. |
| [az network public-ip create](/cli/azure/network/public-ip) | Crea la dirección IP pública de la puerta de enlace de aplicaciones. |
| [az network application-gateway create](/cli/azure/network/application-gateway) | Crear una puerta de enlace de aplicaciones. |
| [az vmss create](/cli/azure/vmss#az_vmss_create) | Crea un conjunto de escalado de máquinas virtuales. |
| [az storage account create](/cli/azure/storage/account#az_storage_account_create) | Crea una cuenta de almacenamiento. |
| [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az_monitor_diagnostic_settings_create) | Crea una cuenta de almacenamiento. |
| [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show) | Obtiene la dirección IP pública de la puerta de enlace de aplicaciones. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure/overview).

Se pueden encontrar más ejemplos de script de la CLI de la puerta de enlace de aplicaciones en la [documentación de Azure Application Gateway](../cli-samples.md).
