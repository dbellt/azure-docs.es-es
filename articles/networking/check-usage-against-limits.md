---
title: Comparación del uso de recursos de Azure con los límites | Microsoft Docs
description: Aprenda a comparar el uso de recursos de Azure con los límites de suscripción de Azure.
services: networking
documentationcenter: na
author: KumudD
ms.author: kumud
tags: azure-resource-manager
ms.service: azure
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/05/2018
ms.openlocfilehash: d629e65106145a4af364cd9dd489250c8910c25d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107778578"
---
# <a name="check-resource-usage-against-limits"></a>Comparación del uso de recursos con los límites

En este artículo, aprenderá a ver el número de cada tipo de recurso de red que ha implementado en su suscripción y cuáles son los [límites](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fnetworking%2ftoc.json#networking-limits) para ella. La posibilidad de ver el uso de recursos con respecto a los límites resulta de utilidad para realizar el seguimiento del uso actual y planear el uso futuro. El seguimiento de uso se puede realizar con [Azure Portal](#azure-portal), [PowerShell](#powershell) o la [CLI de Azure](#azure-cli).

## <a name="azure-portal"></a>Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En la esquina superior izquierda de Azure Portal, seleccione **Todos los servicios**.
3. Escriba *Suscripciones* en el cuadro **Filtro**. Cuando la opción **Suscripciones** aparezca en los resultados de búsqueda, selecciónela.
4. Seleccione el nombre de la suscripción de la que desea ver información de uso.
5. En **Configuración**, seleccione **Usage + quota** (Uso y cuota).
6. Puede seleccionar las opciones siguientes:
   - **Tipos de recursos**: puede seleccionar todos los tipos de recursos, o aquellos tipos específicos que quiera ver.
   - **Proveedores**: puede seleccionar todos los proveedores de recursos, o bien seleccionar **Compute**, **Network** o **Storage**.
   - **Ubicaciones**: puede seleccionar todas las ubicaciones de Azure, o ubicaciones específicas.
   - Puede seleccionar mostrar todos los recursos o solo aquellos donde al menos uno se haya implementado.

     En el ejemplo de la siguiente ilustración se muestran todos los recursos de red con al menos un recurso implementado en la región Este de EE. UU.:

       ![Vista de datos de uso](./media/check-usage-against-limits/view-usage.png)

     Si quiere ordenar las columnas, seleccione el encabezado de columna. Los límites mostrados son los límites de su suscripción. Si necesita aumentar el límite predeterminado, seleccione **Solicitar aumento** y luego complete y envíe la solicitud de soporte técnico. Todos los recursos tienen un límite máximo enumerado en los [límites](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fnetworking%2ftoc.json#networking-limits) de Azure. Si el límite actual ya es el número máximo, no se puede aumentar el límite.

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Puede ejecutar los comandos siguientes en [Azure Cloud Shell](https://shell.azure.com/powershell), o mediante la ejecución de PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Si ejecuta PowerShell desde el equipo, necesita el módulo Azure PowerShell versión 1.0.0 o posterior. Ejecute `Get-Module -ListAvailable Az` en el equipo para encontrar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si ejecuta PowerShell localmente, también debe ejecutar `Login-AzAccount` para iniciar sesión en Azure.

Para ver el uso con respecto a los límites, ejecute [Get-AzNetworkUsage](/powershell/module/az.network/get-aznetworkusage). En el ejemplo siguiente se obtiene el uso de recursos donde al menos un recurso está implementado en la ubicación Este de EE. UU.:

```azurepowershell-interactive
Get-AzNetworkUsage `
  -Location eastus `
  | Where-Object {$_.CurrentValue -gt 0} `
  | Format-Table ResourceType, CurrentValue, Limit
```

Recibirá una salida con el mismo formato que el del ejemplo siguiente:

```output
ResourceType            CurrentValue Limit
------------            ------------ -----
Virtual Networks                   1    50
Network Security Groups            2   100
Public IP Addresses                1    60
Network Interfaces                 1 24000
Network Watchers                   1     1
```

## <a name="azure-cli"></a>Azure CLI

Si usa la interfaz de la línea de comandos (CLI) de Azure para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/bash) o ejecute la CLI en el equipo. En este artículo se requiere la CLI de Azure versión 2.0.32 o posterior. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si ejecuta la CLI de Azure localmente, también debe ejecutar `az login` para iniciar sesión en Azure.

Para ver el uso con respecto a los límites, ejecute [az network list-usages](/cli/azure/network#az_network_list_usages). En el ejemplo siguiente se obtiene el uso de recursos en la ubicación Este de EE. UU.:

```azurecli-interactive
az network list-usages \
  --location eastus \
  --out table
```

Recibirá una salida con el mismo formato que el del ejemplo siguiente:

```output
Name                    CurrentValue Limit
------------            ------------ -----
Virtual Networks                   1    50
Network Security Groups            2   100
Public IP Addresses                1    60
Network Interfaces                 1 24000
Network Watchers                   1     1
Load Balancers                     0   100
Application Gateways               0    50
```