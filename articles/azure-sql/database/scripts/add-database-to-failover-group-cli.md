---
title: 'La CLI de Azure: Adición de una base de datos única a un grupo de conmutación por error'
description: Uso del script de ejemplo de la CLI de Azure para crear una base de datos de Azure SQL Database, agregarla a un grupo de conmutación por error y probar la conmutación por error.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: BustosMSFT
ms.author: robustos
ms.reviewer: mathoma
ms.date: 07/16/2019
ms.openlocfilehash: fe14d8845b743fae801b8a6d6e0cb2e4a44be7d6
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110694055"
---
# <a name="use-the-azure-cli-to-add-a-database-to-a-failover-group"></a>Uso de la CLI de Azure para agregar una base de datos a un grupo de conmutación por error

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

En este script de ejemplo de la CLI de Azure se crea una base de datos en Azure SQL Database y un grupo de conmutación por error, se le agrega la base de datos y se prueba la conmutación por error.

Si decide instalar y usar la CLI localmente, para este tema es preciso que ejecute la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="sample-script"></a>Script de ejemplo

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

```azurecli-interactive
$subscription = "<subscriptionId>" # add subscription here

az account set -s $subscription # ...or use 'az login'
```

### <a name="run-the-script"></a>Ejecute el script.

[!code-azurecli-interactive[main](../../../../cli_scripts/sql-database/failover-groups/add-single-db-to-failover-group-az-cli.sh "Add Azure SQL Database to failover group")]

### <a name="clean-up-deployment"></a>Limpieza de la implementación

Use el siguiente comando para quitar el grupo de recursos y todos los recursos que tenga asociados.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Referencia de ejemplo

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Descripción |
|---|---|
| [az sql db](/cli/azure/sql/db) | Comandos de base de datos. |
| [az sql failover-group](/cli/azure/sql/failover-group) | Comandos de grupo de conmutación por error. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de SQL Database en la [documentación de Azure SQL Database](../az-cli-script-samples-content-guide.md).
