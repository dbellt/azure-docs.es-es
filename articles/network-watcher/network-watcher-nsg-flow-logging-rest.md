---
title: 'Administración de registros de flujos de NSG: API de REST de Azure'
titleSuffix: Azure Network Watcher
description: Esta página explica cómo administrar registros de flujo de grupos de seguridad de red en Azure Network Watcher con la API de REST
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: damendo
ms.openlocfilehash: 0c2725ff58e40bc56aac528a17fe4d66a128d475
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108146948"
---
# <a name="configuring-network-security-group-flow-logs-using-rest-api"></a>Configuración de registros de flujo de grupos de seguridad de red mediante la API de REST

> [!div class="op_single_selector"]
> - [Azure Portal](network-watcher-nsg-flow-logging-portal.md)
> - [PowerShell](network-watcher-nsg-flow-logging-powershell.md)
> - [CLI de Azure](network-watcher-nsg-flow-logging-cli.md)
> - [REST API](network-watcher-nsg-flow-logging-rest.md)

Los registros de flujo de grupos de seguridad de red son una característica de Network Watcher que permite ver información acerca del tráfico IP de entrada y de salida en un grupo de seguridad de red. Estos registros de flujo se escriben en formato JSON y muestran los flujos de entrada y salida en función de cada regla, la NIC a la que se aplica el flujo, información de 5-tupla sobre el flujo (IP de origen/destino, puerto de origen/destino, protocolo), y si se permitió o denegó el tráfico.

## <a name="before-you-begin"></a>Antes de empezar

ARMclient se usa para llamar a la API de REST con PowerShell. ARMClient se encuentra en Chocolatey en [ARMClient en Chocolatey](https://chocolatey.org/packages/ARMClient). Las especificaciones detalladas de la API REST de registros de flujo de grupo de seguridad de red se pueden encontrar [aquí](/rest/api/network-watcher/flowlogs). 

En este escenario, se da por hecho que ya ha seguido los pasos descritos en [Creación de una instancia de Network Watcher](network-watcher-create.md) para crear un monitor de red.

> [!Important]
> En las llamadas a la API de REST de Network Watcher, el nombre del grupo de recursos en el identificador URI de la solicitud es el grupo de recursos que contiene Network Watcher, y no los recursos sobre los que realiza las acciones de diagnóstico.

## <a name="scenario"></a>Escenario

El escenario descrito en este artículo muestra cómo habilitar, deshabilitar y realizar consultas de los registros de flujo con la API de REST. Para más información sobre registros de flujos de grupo de seguridad de red, consulte la [introducción al registro de flujos de grupo de seguridad de red](network-watcher-nsg-flow-logging-overview.md).

En este escenario:

* Habilitar registros de flujo (versión 2)
* Deshabilitar los registros de flujo
* Consultar el estado de los registros de flujo

## <a name="log-in-with-armclient"></a>Inicio de sesión con ARMClient

Inicie sesión en armclient con las credenciales de Azure.

```powershell
armclient login
```

## <a name="register-insights-provider"></a>Registro del proveedor de Insights

Para que el registro del flujo de trabajo funcione correctamente, es necesario registrar el proveedor **Microsoft.Insights**. Si no está seguro de si el proveedor **Microsoft.Insights** está registrado, ejecute el siguiente script.

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
armclient post "https://management.azure.com//subscriptions/${subscriptionId}/providers/Microsoft.Insights/register?api-version=2016-09-01"
```

## <a name="enable-network-security-group-flow-logs"></a>Habilitación de los registros de flujo de grupos de seguridad de red

En el ejemplo siguiente se muestra el comando para habilitar la versión 2 de los registros de flujo. Para la versión 1, reemplace el campo "version" con "1":

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
$targetUri = "" # example /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName/providers/Microsoft.Network/networkSecurityGroups/{nsgName}"
$storageId = "/subscriptions/00000000-0000-0000-0000-000000000000/{resourceGroupName/providers/Microsoft.Storage/storageAccounts/{saName}"
$resourceGroupName = "NetworkWatcherRG"
$networkWatcherName = "NetworkWatcher_westcentralus"
$requestBody = @"
{
    'targetResourceId': '${targetUri}',
    'properties': {
    'storageId': '${storageId}',
    'enabled': 'true',
    'retentionPolicy' : {
            days: 5,
            enabled: true
        },
    'format': {
        'type': 'JSON',
        'version': 2
    }
    }
}
"@

armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/configureFlowLog?api-version=2016-12-01" $requestBody
```

La respuesta devuelta por el ejemplo anterior es la siguiente:

```json
{
  "targetResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}",
  "properties": {
    "storageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{saName}",
    "enabled": true,
    "retentionPolicy": {
      "days": 5,
      "enabled": true
    },
    "format": {
    "type": "JSON",
    "version": 2
    }
  }
}
```

## <a name="disable-network-security-group-flow-logs"></a>Deshabilitación de los registros de flujo de grupos de seguridad de red

Use el ejemplo siguiente para deshabilitar los registros de flujo. La llamada es la misma que para habilitar los registros de flujo, excepto que la propiedad activada está establecida en **false**.

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
$targetUri = "" # example /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName/providers/Microsoft.Network/networkSecurityGroups/{nsgName}"
$storageId = "/subscriptions/00000000-0000-0000-0000-000000000000/{resourceGroupName/providers/Microsoft.Storage/storageAccounts/{saName}"
$resourceGroupName = "NetworkWatcherRG"
$networkWatcherName = "NetworkWatcher_westcentralus"
$requestBody = @"
{
    'targetResourceId': '${targetUri}',
    'properties': {
    'storageId': '${storageId}',
    'enabled': 'false',
    'retentionPolicy' : {
            days: 5,
            enabled: true
        },
    'format': {
        'type': 'JSON',
        'version': 2
    }
    }
}
"@

armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/configureFlowLog?api-version=2016-12-01" $requestBody
```

La respuesta devuelta por el ejemplo anterior es la siguiente:

```json
{
  "targetResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}",
  "properties": {
    "storageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{saName}",
    "enabled": false,
    "retentionPolicy": {
      "days": 5,
      "enabled": true
    },
    "format": {
    "type": "JSON",
    "version": 2
    }
  }
}
```

## <a name="query-flow-logs"></a>Consulta del estado de los registros de flujo

La siguiente llamada de REST consulta el estado del flujo de registros en un grupo de seguridad de red.

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000"
$targetUri = "" # example /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName/providers/Microsoft.Network/networkSecurityGroups/{nsgName}"
$resourceGroupName = "NetworkWatcherRG"
$networkWatcherName = "NetworkWatcher_westcentralus"
$requestBody = @"
{
    'targetResourceId': '${targetUri}',
}
"@

armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/queryFlowLogStatus?api-version=2016-12-01" $requestBody
```

El siguiente es un ejemplo de la respuesta devuelta:

```json
{
  "targetResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}",
  "properties": {
    "storageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{saName}",
    "enabled": true,
   "retentionPolicy": {
      "days": 5,
      "enabled": true
    },
    "format": {
    "type": "JSON",
    "version": 2
    }
  }
}
```

## <a name="download-a-flow-log"></a>Descarga de un registro de flujo

La ubicación de almacenamiento de un registro de flujo se define en el momento de la creación. Una herramienta práctica para acceder a estos registros de flujo guardados en una cuenta de almacenamiento es el Explorador de Microsoft Azure Storage, que puede descargarse aquí: https://storageexplorer.com/

Si se especifica una cuenta de almacenamiento, los archivos de captura de paquetes se guardan en una cuenta de almacenamiento en la siguiente ubicación:

```
https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json
```

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [visualizar los registros de flujo de NSG con Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md)

Aprenda a [visualizar los registros de flujo de NSG con herramientas de código abierto](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)