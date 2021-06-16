---
title: 'CLI de Azure: Copia de seguridad de una base de datos de Azure SQL Database'
description: Script de ejemplo de la CLI de Azure para realizar una copia de seguridad de una base de datos única de Azure SQL en un contenedor de Azure Storage
services: sql-database
ms.service: sql-database
ms.subservice: backup-restore
ms.custom: devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: shkale-msft
ms.author: shkale
ms.reviewer: mathoma
ms.date: 03/27/2019
ms.openlocfilehash: 0485600654fd7a041b62f07c38dfa200845cdae8
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110703153"
---
# <a name="use-cli-to-backup-an-azure-sql-single-database-to-an-azure-storage-container"></a>Uso de la CLI para realizar una copia de seguridad de una base de datos única de Azure SQL en un contenedor de Azure Storage

Este ejemplo de la CLI de Azure realiza una copia de seguridad de una base de datos de SQL Database en un contenedor de Azure Storage.  

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="sample-script"></a>Script de ejemplo

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

```azurecli-interactive
$subscription = "<subscriptionId>" # add subscription here

az account set -s $subscription # ...or use 'az login'
```

### <a name="run-the-script"></a>Ejecute el script.

[!code-azurecli-interactive[main](../../../cli_scripts/sql-database/backup-database/backup-database.sh "Restore SQL Database")]

### <a name="clean-up-deployment"></a>Limpieza de la implementación

Use el siguiente comando para quitar el grupo de recursos y todos los recursos que tenga asociados.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Referencia de ejemplo

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az sql server](/cli/azure/sql/server) | Comandos de servidor. |
| [az sql db](/cli/azure/sql/db) | Comandos de base de datos. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de SQL Database en la [documentación de Azure SQL Database](../../azure-sql/database/az-cli-script-samples-content-guide.md).
