---
title: Errores de SKU no disponible
description: Describe cómo solucionar problemas relacionados con el error de SKU no disponible al implementar recursos con Azure Resource Manager.
ms.topic: troubleshooting
ms.date: 04/14/2021
ms.openlocfilehash: 3baedf6a5c9f2dbfd3ddf666b458fac649fce2ac
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107503905"
---
# <a name="resolve-errors-for-sku-not-available"></a>Resolución de los errores de no disponibilidad de la SKU

Este artículo describe cómo resolver el error **SkuNotAvailable**. Si no puede encontrar ninguna SKU adecuada en esa región o zona ni en ninguna otra región o zona alternativas que satisfagan las necesidades del negocio, envíe una [solicitud de SKU](/troubleshoot/azure/general/region-access-request-process) al servicio de soporte técnico de Azure.

## <a name="symptom"></a>Síntoma

Al implementar un recurso (normalmente una máquina virtual), puede recibir el siguiente código y mensaje de error:

```
Code: SkuNotAvailable
Message: The requested tier for resource '<resource>' is currently not available in location '<location>'
for subscription '<subscriptionID>'. Please try another tier or deploy to a different location.
```

## <a name="cause"></a>Causa

Recibe este error si la SKU del recurso que ha seleccionado (como el tamaño de máquina virtual) no está disponible para la ubicación seleccionada.

Si va a implementar una máquina virtual de Azure Spot o una instancia del conjunto de escalado de Spot, no habrá capacidad para Azure Spot en esta ubicación. Para más información, consulte [Mensajes de error de Spot](../../virtual-machines/error-codes-spot.md).

## <a name="solution-1---powershell"></a>Solución 1: PowerShell

Para determinar qué SKU están disponibles en una región o una zona, use el comando [Get-AzComputeResourceSku](/powershell/module/az.compute/get-azcomputeresourcesku). Filtre los resultados por ubicación. Debe tener la versión más reciente de PowerShell para que funcione este comando.

```azurepowershell-interactive
Get-AzComputeResourceSku | where {$_.Locations -icontains "centralus"}
```

Los resultados incluyen una lista de SKU para la ubicación y las restricciones de esas SKU. Tenga en cuenta que una SKU debe aparecer como `NotAvailableForSubscription`.

```output
ResourceType          Name           Locations   Zone      Restriction                      Capability           Value
------------          ----           ---------   ----      -----------                      ----------           -----
virtualMachines       Standard_A0    centralus             NotAvailableForSubscription      MaxResourceVolumeMB   20480
virtualMachines       Standard_A1    centralus             NotAvailableForSubscription      MaxResourceVolumeMB   71680
virtualMachines       Standard_A2    centralus             NotAvailableForSubscription      MaxResourceVolumeMB  138240
virtualMachines       Standard_D1_v2 centralus   {2, 1, 3}                                  MaxResourceVolumeMB
```

Para filtrar por ubicación y SKU, utilice:

```azurepowershell-interactive
$SubId = (Get-AzContext).Subscription.Id

$Region = "centralus" # change region here
$VMSku = "Standard_M" # change VM SKU here

$VMSKUs = Get-AzComputeResourceSku | where {$_.Locations.Contains($Region) -and $_.ResourceType.Contains("virtualMachines") -and $_.Name.Contains($VMSku)}

$OutTable = @()

foreach ($SkuName in $VMSKUs.Name)
        {
            $LocRestriction = if ((($VMSKUs | where Name -EQ $SkuName).Restrictions.Type | Out-String).Contains("Location")){"NotAvavalableInRegion"}else{"Available - No region restrictions applied" }
            $ZoneRestriction = if ((($VMSKUs | where Name -EQ $SkuName).Restrictions.Type | Out-String).Contains("Zone")){"NotAvavalableInZone: "+(((($VMSKUs | where Name -EQ $SkuName).Restrictions.RestrictionInfo.Zones)| Where-Object {$_}) -join ",")}else{"Available - No zone restrictions applied"}
            
            
            $OutTable += New-Object PSObject -Property @{
                                                         "Name" = $SkuName
                                                         "Location" = $Region
                                                         "Applies to SubscriptionID" = $SubId
                                                         "Subscription Restriction" = $LocRestriction
                                                         "Zone Restriction" = $ZoneRestriction
                                                         }
         }

$OutTable | select Name, Location, "Applies to SubscriptionID", "Region Restriction", "Zone Restriction" | Sort-Object -Property Name | FT
```

El comando devuelve resultados como:

```output
Name                   Location  Applies to SubscriptionID            Region Restriction                         Zone Restriction                        
----                   --------  -------------------------            ------------------------                   ----------------     
Standard_M128          centralus xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Available - No region restrictions applied Available - No zone restrictions applied
Standard_M128-32ms     centralus xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Available - No region restrictions applied Available - No zone restrictions applied
Standard_M128-64ms     centralus xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Available - No region restrictions applied Available - No zone restrictions applied
Standard_M128dms_v2    centralus xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx NotAvavalableInRegion                      NotAvavalableInZone: 1,2,3
```

## <a name="solution-2---azure-cli"></a>Solución 2: CLI de Azure

Para determinar las SKU que están disponibles en una región, utilice el comando [az vm list-skus](/cli/azure/vm#az_vm_list_skus). Utilice el parámetro `--location` para filtrar la salida en función de la ubicación. Use el parámetro `--size` para buscar un nombre de tamaño parcial. Use el parámetro `--all` para mostrar toda la información, incluidos los tamaños que no están disponibles para la suscripción actual.

Debe disponer de la CLI de Azure 2.15.0 o una versión posterior. Para comprobar la versión, utilice `az --version`. Si es necesario, [actualice la instalación](/cli/azure/update-azure-cli).

```azurecli-interactive
az vm list-skus --location southcentralus --size Standard_F --all --output table
```

El comando devuelve resultados como:

```output
ResourceType     Locations       Name              Zones    Restrictions
---------------  --------------  ----------------  -------  --------------
virtualMachines  southcentralus  Standard_F1       1,2,3    None
virtualMachines  southcentralus  Standard_F2       1,2,3    None
virtualMachines  southcentralus  Standard_F4       1,2,3    None
...
virtualMachines  southcentralus  Standard_F72s_v2  1,2,3    NotAvailableForSubscription, type: Zone, locations: southcentralus, zones: 1,2,3
...
```

## <a name="solution-3---azure-portal"></a>Solución 3: Azure Portal

Para determinar qué SKU están disponibles en una región, use el [portal](https://portal.azure.com). Inicie sesión en el portal y agregue un recurso mediante la interfaz. A medida que establezca los valores, verá las SKU disponibles para ese recurso. No es necesario completar la implementación.

Por ejemplo, inicie el proceso de creación de una máquina virtual. Para ver otro tamaño disponible, seleccione **Cambiar tamaño**.

![Creación de una máquina virtual](./media/error-sku-not-available/create-vm.png)

Puede filtrar y desplazarse por los tamaños disponibles.

![SKU disponibles](./media/error-sku-not-available/available-sizes.png)

## <a name="solution-4---rest"></a>Solución 4: REST

Para determinar qué SKU están disponibles en una región, use la operación [Resource Skus - List](/rest/api/compute/resourceskus/list).

Se devuelven las SKU y las regiones disponibles en el formato siguiente:

```json
{
  "value": [
    {
      "resourceType": "virtualMachines",
      "name": "Standard_A0",
      "tier": "Standard",
      "size": "A0",
      "locations": [
        "eastus"
      ],
      "restrictions": []
    },
    {
      "resourceType": "virtualMachines",
      "name": "Standard_A1",
      "tier": "Standard",
      "size": "A1",
      "locations": [
        "eastus"
      ],
      "restrictions": []
    },
    ...
  ]
}
```