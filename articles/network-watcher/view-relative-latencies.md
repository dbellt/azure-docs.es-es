---
title: Vista de latencias relativas a las regiones de Azure desde ubicaciones específicas
description: Obtenga información sobre cómo ver las latencias relativas en distintos proveedores de Internet a regiones de Azure desde ubicaciones específicas.
services: network-watcher
documentationcenter: ''
author: damendo
ms.service: virtual-network
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/14/2017
ms.author: damendo
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 77433ff3a3bb74d3c976fb0a65e668e8e4d8f452
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110676099"
---
# <a name="view-relative-latency-to-azure-regions-from-specific-locations"></a>Vista de latencias relativas a las regiones de Azure desde ubicaciones específicas

> [!WARNING]
> Esta característica se encuentra actualmente en versión preliminar y aún se está probando su estabilidad.

En este tutorial, obtenga información sobre cómo usar el servicio Azure [Network Watcher](network-watcher-monitoring-overview.md) para poder decidir en qué región de Azure implementar su aplicación o servicio, en función de la información demográfica de los usuarios. Además, puede usarlo para evaluar las conexiones a Azure de los proveedores de servicios.  
        

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="create-a-network-watcher"></a>Creación de una instancia de Network Watcher

Si ya cuenta con una instancia de Network Watcher al menos en una [región](https://azure.microsoft.com/regions) de Azure, puede omitir las tareas que aparecen en esta sección. Cree un grupo de recursos para la instancia de Network Watcher. En este ejemplo, el grupo de recursos se crea en la región Este de EE. UU., pero puede crear el grupo de recursos en cualquier región de Azure.

```powershell
New-AzResourceGroup -Name NetworkWatcherRG -Location eastus
```

Cree una instancia de Network Watcher. Debe tener creada una instancia de Network Watcher al menos en una región de Azure. En este ejemplo, se crea una instancia de Network Watcher en la región de Azure Este de EE. UU.

```powershell
New-AzNetworkWatcher -Name NetworkWatcher_eastus -ResourceGroupName NetworkWatcherRG -Location eastus
```

## <a name="compare-relative-network-latencies-to-a-single-azure-region-from-a-specific-location"></a>Comparación de las latencias relativas de red a una sola región de Azure desde una ubicación específica

Evalúe los proveedores de servicios o solucione los problemas que notifican los usuarios, como "El sitio funcionaba lento", desde una ubicación específica hasta la región de Azure donde está implementado un servicio. Por ejemplo, el comando siguiente devuelve las latencias relativas promedio del proveedor de servicios de Internet entre el estado de Washington en Estados Unidos y la región de Azure Oeste de EE. UU. 2 entre el 13 y el 15 de diciembre de 2017:

```powershell
Get-AzNetworkWatcherReachabilityReport `
  -NetworkWatcherName NetworkWatcher_eastus `
  -ResourceGroupName NetworkWatcherRG `
  -Location "West US 2" `
  -Country "United States" `
  -State "washington" `
  -StartTime "2017-12-13" `
  -EndTime "2017-12-15"
```

> [!NOTE]
> No es necesario que la región que se especifique en el comando anterior sea la misma que la región que especificó cuando recuperó la instancia de Network Watcher. El comando anterior simplemente requiere que se especifique una instancia existente de Network Watcher. La instancia de Network Watcher puede estar en cualquier región. Si especifica valores para `-Country` y `-State`, deben ser valores válidos. Los valores distinguen mayúsculas de minúsculas. Los datos están disponibles para un número limitado de países, regiones, estados y ciudades. Ejecute los comandos que aparecen en la sección [Vista de los países, las regiones, los estados, las ciudades y los proveedores disponibles](#view-available) para ver una lista de los países, las regiones, las ciudades y los estados disponibles para usarlos con el comando anterior. 

> [!WARNING]
> Debe especificar una fecha dentro de los últimos 30 días para `-StartTime` y `-EndTime`. Especificar una fecha anterior dará como resultado que no se devuelva ningún dato.

La siguiente es la salida del comando anterior:

```powershell
AggregationLevel   : State
ProviderLocation   : {
                       "Country": "United States",
                       "State": "washington"
                     }
ReachabilityReport : [
                       {
                         "Provider": "Qwest Communications Company, LLC - ASN 209",
                         "AzureLocation": "West US 2",
                         "Latencies": [
                           {
                             "TimeStamp": "2017-12-14T00:00:00Z",
                             "Score": 92
                           },
                           {
                             "TimeStamp": "2017-12-13T00:00:00Z",
                             "Score": 92
                           }
                         ]
                       },
                       {
                         "Provider": "Comcast Cable Communications, LLC - ASN 7922",
                         "AzureLocation": "West US 2",
                         "Latencies": [
                           {
                             "TimeStamp": "2017-12-14T00:00:00Z",
                             "Score": 96
                           },
                           {
                             "TimeStamp": "2017-12-13T00:00:00Z",
                             "Score": 96
                           }
                         ]
                       }
                     ]
```

En la salida devuelta, el valor de **Score** es la latencia relativa entre las regiones y los proveedores. Una valor Score de 1 es la peor latencia (la más alta), mientras que un valor de 100 se refiere a la latencia más baja. Las latencias relativas se promedian para el día. En el ejemplo anterior, si bien queda claro que las latencias fueron iguales ambos días y que hay una pequeña diferencia entre la latencia de ambos proveedores, también se ve claramente que las latencias de ambos proveedores son bajas en la escala de 1 a 100. Si bien este es un resultado esperado, debido a que el estado de Washington en Estados Unidos está físicamente cerca de la región de Azure Oeste de EE. UU. 2, los resultados a veces no son los que se esperan. Cuanto mayor sea el intervalo de fechas que especifique, puede calcular el promedio de la latencia durante un período más prolongado.

## <a name="compare-relative-network-latencies-across-azure-regions-from-a-specific-location"></a>Comparación de las latencias relativas de red entre varias regiones de Azure desde una ubicación específica

Si en lugar de especificar las latencias relativas entre una ubicación específica y una región de Azure específica mediante `-Location`, desea determinar las latencias relativas a todas las regiones de Azure desde una ubicación física específica, también puede hacerlo. Por ejemplo, el comando siguiente permite evaluar en qué región de Azure implementar un servicio si los usuarios primarios son usuarios de Comcast ubicados en el estado de Washington:

```powershell
Get-AzNetworkWatcherReachabilityReport `
  -NetworkWatcherName NetworkWatcher_eastus `
  -ResourceGroupName NetworkWatcherRG `
  -Provider "Comcast Cable Communications, LLC - ASN 7922" `
  -Country "United States" `
  -State "washington" `
  -StartTime "2017-12-13" `
  -EndTime "2017-12-15"
```

> [!NOTE]
> A diferencia de cuando se especifica una sola ubicación, si no especifica una o especifica varias, como "Oeste de EE. UU.2", "Oeste de EE. UU.", debe especificar un proveedor de servicios de Internet cuando se ejecuta el comando. 

## <a name="view-available-countriesregions-states-cities-and-providers"></a><a name="view-available"></a>Vista de los países, las regiones, los estados, las ciudades y los proveedores disponibles

Hay datos disponibles para proveedores de servicios de Internet, países, regiones, estados y ciudades específicos. Para ver una lista de todos los proveedores de servicios de Internet, los países, las regiones, los estados y las ciudades disponibles para los cuales puede ver datos, escriba el comando siguiente:

```powershell
Get-AzNetworkWatcherReachabilityProvidersList -NetworkWatcherName NetworkWatcher_eastus -ResourceGroupName NetworkWatcherRG
```

Solo hay datos disponibles para los países, las regiones, los estados y las ciudades que devuelve el comando anterior. El comando siguiente requiere que se especifique una instancia existente de Network Watcher. El ejemplo especificó la instancia de Network Watcher *NetworkWatcher_eastus* en un grupo de recursos denominado *NetworkWatcherRG*, pero puede especificar cualquier instancia existente de Network Watcher. Si no tiene una instancia de Network Watcher, complete las tareas que aparecen en [Creación de una instancia de Network Watcher](#create-a-network-watcher) para crear una. 

Después de ejecutar el comando anterior, puede filtrar la salida devuelta si especifica los valores válidos para **Country**, **State** y **City**, si así lo desea.  Por ejemplo, para ver la lista de proveedores de servicios de Internet disponibles en Seattle, Washington, en Estados Unidos, escriba el comando siguiente:

```powershell
Get-AzNetworkWatcherReachabilityProvidersList `
  -NetworkWatcherName NetworkWatcher_eastus `
  -ResourceGroupName NetworkWatcherRG `
  -City Seattle `
  -Country "United States" `
  -State washington
```

> [!WARNING]
> El valor especificado para **Country** debe incluir mayúsculas y minúsculas. Los valores especificados para **State** y **City** deben estar en minúsculas. Los valores deben aparecer en la salida devuelta después de ejecutar el comando sin valores para **Country**, **State** y **City**. Si especifica un uso incorrecto de mayúsculas y minúsculas, o bien especifica un valor para **Country**, **State** o **City** que no esté en la salida devuelta después de ejecutar el comando sin valores para estas propiedades, la salida se devuelve vacía.
