---
title: Solución de problemas de LocationNotFoundForRoleSize al implementar un servicio en la nube (clásico) en Azure | Microsoft Docs
description: En este artículo se muestra cómo resolver una excepción LocationNotFoundForRoleSize al implementar un servicio en la nube (clásico) en Azure.
services: cloud-services
author: mamccrea
ms.author: mamccrea
ms.service: cloud-services
ms.topic: troubleshooting
ms.date: 02/22/2021
ms.openlocfilehash: 54af2387ec0ff6c8f86f96821baad17736e8d85b
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107877973"
---
# <a name="troubleshoot-locationnotfoundforrolesize-when-deploying-a-cloud-service-classic-to-azure"></a>Solución de problemas de LocationNotFoundForRoleSize al implementar un servicio en la nube (clásico) en Azure

En este artículo, solucionará los errores de asignación en los que un tamaño de máquina virtual no está disponible al implementar un servicio en la nube de Azure (clásico).

Al implementar instancias en Cloud Services (clásico) o agregar nuevas instancias de rol de trabajo o web, Microsoft Azure asigna recursos de proceso.

En ocasiones, es posible que reciba errores durante estas operaciones, incluso antes de alcanzar el límite de la suscripción de Azure.

> [!TIP]
> La información también puede ser útil si tiene pensado realizar la implementación de sus servicios.

## <a name="symptom"></a>Síntoma

En Azure Portal, vaya a su instancia de Cloud Services (clásico) y, en la barra lateral, seleccione *Registro de operaciones (clásico)* para ver los registros.

![En la imagen se muestra la hoja del registro de operaciones (clásico).](./media/cloud-services-troubleshoot-location-not-found-for-role-size/cloud-services-troubleshoot-allocation-logs.png)

Al inspeccionar los registros de la instancia de Cloud Services (clásico), verá la siguiente excepción:

|Tipo de excepción  |Mensaje de error  |
|---------|---------|
|LocationNotFoundForRoleSize     |Error en la operación "`{Operation ID}`": "El nivel de VM solicitado no está disponible actualmente en (`{Region ID}`) para esta suscripción. Pruebe con otro nivel o realice la implementación en una ubicación distinta".|

## <a name="cause"></a>Causa

Existe un problema de capacidad con la región o el clúster donde se va a realizar la implementación. La excepción *LocationNotFoundForRoleSize* se produce cuando la SKU de recurso que ha seleccionado (tamaño de máquina virtual) no está disponible para la región especificada.

## <a name="solution"></a>Solución

En este escenario, debe seleccionar una región o SKU diferente donde implementar el servicio en la nube (clásico). Antes de implementar o actualizar el servicio en la nube (clásico), puede determinar qué SKU están disponibles en una región o zona de disponibilidad. Siga los procesos de la [CLI de Azure](#list-skus-in-region-using-azure-cli), de [PowerShell](#list-skus-in-region-using-powershell) o de la [API de REST](#list-skus-in-region-using-rest-api) a continuación.

### <a name="list-skus-in-region-using-azure-cli"></a>Enumeración de SKU en la región con la CLI de Azure

Puede usar el comando [az vm list-skus](/cli/azure/vm?view=azure-cli-latest
#<a name="az_vm_list_skus-command"></a>az_vm_list_skus).

- Use el parámetro `--location` para filtrar la salida a la ubicación que esté usando.
- Use el parámetro `--size` para buscar un nombre de tamaño parcial.
- Para más información, vea la guía sobre [Resolución del error con la SKU no disponible](../azure-resource-manager/templates/error-sku-not-available.md#solution-2---azure-cli).

    **Por ejemplo:**

    ```azurecli
    az vm list-skus --location southcentralus --size Standard_F --output table
    ```

    **Resultados de ejemplo:** ![Salida de la CLI de Azure tras la ejecución del comando "az vm list-skus --location southcentralus --size Standard_F --output table", que muestra las SKU disponibles.](./media/cloud-services-troubleshoot-constrained-allocation-failed/cloud-services-troubleshoot-constrained-allocation-failed-1.png)

#### <a name="list-skus-in-region-using-powershell"></a>Enumeración de SKU en la región con PowerShell

Puede usar el comando [Get-AzComputeResourceSku](/powershell/module/az.compute/get-azcomputeresourcesku).

- Filtre los resultados por ubicación.
- Debe tener la versión más reciente de PowerShell para que funcione este comando.
- Para más información, vea la guía sobre [Resolución del error con la SKU no disponible](../azure-resource-manager/templates/error-sku-not-available.md#solution-1---powershell).

**Por ejemplo:**

```azurepowershell
Get-AzComputeResourceSku | where {$_.Locations -icontains "centralus"}
```

**Otros comandos útiles:**

Filtre las ubicaciones que contienen el tamaño (Standard_DS14_v2):

```azurepowershell
Get-AzComputeResourceSku | where {$_.Locations.Contains("centralus") -and $_.ResourceType.Contains("virtualMachines") -and $_.Name.Contains("Standard_DS14_v2")}
```

Filtre todas las ubicaciones que contengan el tamaño (V3):

```azurepowershell
Get-AzComputeResourceSku | where {$_.Locations.Contains("centralus") -and $_.ResourceType.Contains("virtualMachines") -and $_.Name.Contains("v3")} | fc
```

#### <a name="list-skus-in-region-using-rest-api"></a>Enumeración de SKU en la región con la API de REST

Puede usar la operación [Resource Skus - List](/rest/api/compute/resourceskus/list). Se devuelven las SKU y las regiones disponibles en el formato siguiente:

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
      <<The rest of your file is located here>>
  ]
}
    
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más soluciones de errores de asignación y comprender mejor cómo se generan:

> [!div class="nextstepaction"]
> [Errores de asignación: Cloud Services (clásico)](cloud-services-allocation-failures.md)

Si su problema con Azure no se trata en este artículo, visite los foros de Azure en [MSDN y Stack Overflow](https://azure.microsoft.com/support/forums/). Puede publicar su problema en ellos o [@AzureSupport en Twitter](https://twitter.com/AzureSupport). También puede enviar una solicitud de soporte técnico de Azure. Para enviar una solicitud de soporte técnico, en la página de [soporte técnico de Azure](https://azure.microsoft.com/support/options/), seleccione *Obtener soporte técnico*.
