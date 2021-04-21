---
title: 'Ejemplo de script de la CLI de Azure: exportación desde un almacén de Azure App Configuration'
titleSuffix: Azure App Configuration
description: Uso de un script de la CLI de Azure para exportar una configuración desde Azure App Configuration
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.devlang: azurecli
ms.topic: sample
ms.date: 02/19/2020
ms.author: alkemper
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1c4eb6e2aa150751dfbadc2307d64ab206b92b6d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107782232"
---
# <a name="export-from-an-azure-app-configuration-store"></a>Exportación desde un almacén de Azure App Configuration

Este script de ejemplo exporta pares clave-valor desde un almacén de Azure App Configuration.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial requiere la versión 2.0 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="sample-script"></a>Script de ejemplo

```azurecli-interactive
#!/bin/bash

# Export all key-values
az appconfig kv export --name myTestAppConfigStore --file ~/Export.json
```

[!INCLUDE [cli-script-cleanup](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para exportar desde un almacén de App Configuration. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az appconfig kv export](/cli/azure/appconfig/kv#az_appconfig_kv_export) | Exporta desde un recurso del almacén de App Configuration. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Puede encontrar ejemplos de script adicionales de la CLI y App Configuration en los [ejemplos de la CLI y Azure App Configuration](../cli-samples.md).
