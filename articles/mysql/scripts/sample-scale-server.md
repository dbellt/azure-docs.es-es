---
title: 'Script de la CLI: Escalado de un servidor de Azure Database for MySQL'
description: Este script de la CLI de ejemplo escala el servidor de Azure Database for MySQL (Base de datos de Azure para MySQL) a un nivel de rendimiento diferente después de consultar las métricas.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 12/02/2019
ms.openlocfilehash: e7dea1cb99ea8ce941b16739bdbd258843877e89
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107791665"
---
# <a name="monitor-and-scale-an-azure-database-for-mysql-server-using-azure-cli"></a>Supervisión y escalado de un servidor de Azure Database for MySQL (Base de datos de Azure para MySQL) mediante la CLI de Azure
Este script de la CLI de ejemplo escala el proceso y el almacenamiento de un servidor individual de Azure Database for MySQL después de consultar las métricas. La capacidad de procesamiento se puede ampliar o reducir. El almacenamiento solo se puede ampliar.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.0 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente. 

## <a name="sample-script"></a>Script de ejemplo
Actualice el script con el identificador de su suscripción.
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/scale-mysql-server/scale-mysql-server.sh "Create and scale Azure Database for MySQL.")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Use el comando siguiente para quitar el grupo de recursos y todos los recursos asociados a él después de ejecutarse el script de ejemplo. 
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/scale-mysql-server/delete-mysql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Explicación del script
Este script usa los comandos que se describen en la tabla siguiente:

| **Comando** | **Notas** |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az mysql server create](/cli/azure/mysql/server#az_mysql_server_create) | Crea un servidor MySQL que hospeda las bases de datos. |
| [az mysql server update](/cli/azure/mysql/server#az_mysql_server_update) | Actualiza las propiedades del servidor MySQL. |
| [az monitor metrics list](/cli/azure/monitor/metrics#az_monitor_metrics_list) | Especifica el valor de métrica de los recursos. |
| [az group delete](/cli/azure/group#az_group_delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre el [proceso y el almacenamiento de Azure Database for MySQL](../concepts-pricing-tiers.md)
- Pruebe otros scripts adicionales: [Ejemplos de la CLI de Azure para Azure Database for MySQL](../sample-scripts-azure-cli.md)
- Más información sobre la [CLI de Azure](/cli/azure)
