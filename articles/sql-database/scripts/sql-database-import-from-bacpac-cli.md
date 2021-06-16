---
title: 'CLI de Azure: Importación de un archivo BACPAC en una base de datos de Azure SQL Database'
description: Script de ejemplo de la CLI de Azure para importar un archivo BACPAC en una base de datos en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: backup-restore
ms.custom: load & move data, devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: shkale-msft
ms.author: shkale
ms.reviewer: mathoma
ms.date: 05/24/2019
ms.openlocfilehash: d0c20ee8e64d5ed9887e04efeaa721e2ea8d08c9
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110688656"
---
# <a name="use-cli-to-import-a-bacpac-file-into-a-database-in-sql-database"></a>Uso de la CLI para importar un archivo BACPAC en una base de datos de SQL Database

Este ejemplo de script de la CLI de Azure importa una base de datos de un archivo *.bacpac* a una base de datos de SQL Database.  

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Script de ejemplo

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

### <a name="run-the-script"></a>Ejecute el script.

[!code-azurecli-interactive[main](../../../cli_scripts/sql-database/import-from-bacpac/import-from-bacpac.sh "Create SQL Database")]

### <a name="clean-up-deployment"></a>Limpieza de la implementación

Use el siguiente comando para quitar el grupo de recursos y todos los recursos que tenga asociados.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Referencia de ejemplo

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Descripción |
|---|---|
| [az sql server](/cli/azure/sql/server) | Comandos de servidor. |
| [az sql db import](/cli/azure/sql/db#az_sql_db_import) | Comando de importación de la base de datos. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de SQL Database en la [documentación de Azure SQL Database](../../azure-sql/database/az-cli-script-samples-content-guide.md).
