---
title: Ejemplo de CLI para restaurar una instancia de Azure SQL Database a partir de una copia de seguridad con replicación geográfica
description: Script de ejemplo de la CLI de Azure para restaurar una base de datos de Instancia administrada de Azure SQL Database a partir de una copia de seguridad con redundancia geográfica.
services: sql-database
ms.service: sql-database
ms.subservice: backup-restore
ms.custom: ''
ms.devlang: azurecli
ms.topic: sample
author: shkale-msft
ms.author: shkale
ms.reviewer: mathoma
ms.date: 07/03/2019
ms.openlocfilehash: c0655ae56422ce29d145814b2cf4686684ae95b1
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110698109"
---
# <a name="use-cli-to-restore-a-managed-instance-database-to-another-geo-region"></a>Uso de la CLI para restaurar una base de datos de Instancia administrada de SQL Database en otra región con replicación geográfica

Este ejemplo de script de la CLI de Azure restaura una base de datos de Instancia administrada de Azure SQL Database desde una región remota con replicación geográfica (georestauración).  

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Script de ejemplo

### <a name="prerequisites"></a>Requisitos previos

Un par de instancias administradas ya existente; consulte [Uso de la CLI de Azure para crear una instancia administrada de Azure SQL Database](sql-database-create-configure-managed-instance-cli.md).

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

### <a name="run-the-script"></a>Ejecute el script.

```azurepowershell-interactive
#!/bin/bash
$instance = "<instanceId>" # add instance here
$targetInstance = "<targetInstanceId>" # add target instance here
$resource = "<resourceId>" # add resource here

$randomIdentifier = $(Get-Random)
$managedDatabase = "managedDatabase-$randomIdentifier"

echo "Creating $($managedDatabase) on $($instance)..."
az sql midb create -g $resource --mi $instance -n $managedDatabase

echo "Restoring $($managedDatabase) to $($targetInstance)..."
az sql midb restore -g $resource --mi $instance -n $managedDatabase --dest-name $targetInstance --time "2018-05-20T05:34:22"
```

## <a name="sample-reference"></a>Referencia de ejemplo

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Script | Descripción |
|---|---|
| [az sql midb](/cli/azure/sql/midb) | Comandos de base de datos de Instancia administrada de Azure SQL Database. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de SQL Database en la [documentación de Azure SQL Database](../../azure-sql/database/az-cli-script-samples-content-guide.md).
