---
title: Detención, reanudación y escalado con las API REST para un grupo de SQL dedicado (anteriormente SQL DW)
description: Administre la potencia de proceso de un grupo de SQL dedicado (anteriormente SQL DW) en Azure Synapse Analytics a través de las API REST.
services: synapse-analytics
author: antvgski
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/29/2019
ms.author: anvang
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: aab2897b4042657492d04494b589fbaa2605cc6d
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107886791"
---
# <a name="rest-apis-for-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>API REST para el grupo de SQL dedicado (anteriormente SQL DW) en Azure Synapse Analytics

API REST para administrar la potencia de proceso de un grupo de SQL dedicado (anteriormente SQL DW) en Azure Synapse Analytics.

> [!NOTE]
> Las API REST que se describen en este artículo no son aplicables a un grupo de SQL dedicado creado en un área de trabajo de Azure Synapse Analytics. Para información sobre las API REST que se usan específicamente en un área de trabajo de Azure Synapse Analytics, consulte [API REST de área de trabajo de Azure Synapse Analytics](/rest/api/synapse/).

## <a name="scale-compute"></a>Escalado de proceso

Para cambiar las unidades de almacenamiento de datos, use la API REST para [crear o actualizar la base de datos](/rest/api/sql/databases/createorupdate). En el ejemplo siguiente se establecen las unidades de almacenamiento de datos en DW1000 para la base de datos MySQLDW, que está hospedada en el servidor MyServer. El servidor está en un grupo de recursos de Azure denominado ResourceGroup1.

```
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Sql/servers/{server-name}/databases/{database-name}?api-version=2020-08-01-preview HTTP/1.1
Content-Type: application/json; charset=UTF-8

{
    location: "West Central US",
    "sku": {
    "name": "DW200c"
    }
}
```

## <a name="pause-compute"></a>Pausa del proceso

Para pausar una base de datos, use la API REST para [pausar la base de datos](/rest/api/sql/databases/pause). El siguiente ejemplo pausa una base de datos denominada Database02 que está hospedada en un servidor llamado Server01. El servidor está en un grupo de recursos de Azure denominado ResourceGroup1.

```
POST https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Sql/servers/{server-name}/databases/{database-name}/pause?api-version=2020-08-01-preview HTTP/1.1
```

## <a name="resume-compute"></a>Reanudación del proceso

Para iniciar una base de datos, use la API REST para [reanudar la base de datos](/rest/api/sql/databases/resume). El siguiente ejemplo inicia una base de datos denominada Database02 que está hospedada en un servidor llamado Server01. El servidor está en un grupo de recursos de Azure denominado ResourceGroup1.

```
POST https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Sql/servers/{server-name}/databases/{database-name}/resume?api-version=2020-08-01-preview HTTP/1.1
```

## <a name="check-database-state"></a>Comprobar el estado de la base de datos

> [!NOTE]
> Actualmente, la comprobación del estado de la base de datos podría devolver ONLINE mientras se está completando el flujo de trabajo en línea, lo que produce errores de conexión. Es posible que deba agregar un retraso de 2 a 3 minutos en el código de aplicación si utiliza esta llamada de API para desencadenar los intentos de conexión.

```
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Sql/servers/{serverName}/databases/{databaseName}?api-version=2020-08-01-preview
```

## <a name="get-maintenance-schedule"></a>Obtención de la programación de mantenimiento

Compruebe la programación de mantenimiento que se ha establecido para un grupo de SQL dedicado (anteriormente SQL DW).

```
GET https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Sql/servers/{server-name}/databases/{database-name}/maintenanceWindows/current?maintenanceWindowName=current&api-version=2017-10-01-preview HTTP/1.1

```

## <a name="set-maintenance-schedule"></a>Establecimiento de la programación de mantenimiento

Para establecer y actualizar una programación de mantenimiento en un grupo de SQL dedicado (anteriormente SQL DW).

```
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Sql/servers/{server-name}/databases/{database-name}/maintenanceWindows/current?maintenanceWindowName=current&api-version=2017-10-01-preview HTTP/1.1

{
    "properties": {
        "timeRanges": [
                {
                                "dayOfWeek": "Saturday",
                                "startTime": "00:00",
                                "duration": "08:00",
                },
                {
                                "dayOfWeek": "Wednesday",
                                "startTime": "00:00",
                                "duration": "08:00",
                }
                ]
    }
}

```

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte [Administración de proceso en Azure SQL Data Warehouse](sql-data-warehouse-manage-compute-overview.md).
