---
title: Traslado de recursos asociados a una configuración de mantenimiento a otra región
description: Obtenga información acerca de cómo mover recursos asociados a una configuración de mantenimiento de VM a otra región de Azure.
author: shants123
ms.service: virtual-machines
ms.topic: how-to
ms.date: 03/04/2020
ms.author: shants
ms.openlocfilehash: 4427071edf237d82e8a99d44678d77d23e180fff
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107865250"
---
# <a name="move-resources-in-a-maintenance-control-configuration-to-another-region"></a>Traslado de recursos de configuración de control de mantenimiento a otra región

Siga este artículo para mover recursos asociados a una configuración de control de mantenimiento a otra región de Azure. Puede que quiera mover una configuración por una serie de motivos. Por ejemplo, para aprovechar una nueva región, para implementar características o servicios que solo están disponibles en una región concreta, para cumplir los requisitos de gobernanza y las directivas internas o en respuesta al planeamiento de capacidad.

El [control de mantenimiento](maintenance-control.md), con configuraciones de mantenimiento personalizadas, le permite controlar cómo se aplican las actualizaciones de la plataforma a las máquinas virtuales Windows y Linux y a instancias de Azure Dedicated Host. Hay un par de escenarios para mover el control de mantenimiento entre regiones:

- Para mover los recursos asociados a una configuración de mantenimiento, pero no la propia configuración, siga este artículo.
- Para mover la configuración de control de mantenimiento, pero no los recursos asociados a la configuración, siga [estas instrucciones](move-region-maintenance-configuration.md).
- Para mover tanto la configuración de mantenimiento como los recursos asociados a ella, siga [estas instrucciones](move-region-maintenance-configuration.md). A continuación, siga las instrucciones de este artículo.

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar a mover los recursos asociados a una configuración de control de mantenimiento:

- Antes de empezar, asegúrese de que los recursos que va a mover existen en la nueva región.
- Compruebe las configuraciones de control de mantenimiento asociadas a las VM de Azure y a las instancias de Azure Dedicated Host que quiere mover. Compruebe cada recurso individualmente. Actualmente no hay ninguna manera de recuperar configuraciones de varios recursos.
- Al recuperar configuraciones de un recurso:
    - Asegúrese de usar el identificador de suscripción para la cuenta, no un identificador de Azure Dedicated Host.
    - CLI: El parámetro --output table solo se usa para facilitar la lectura y se puede eliminar o cambiar.
    - PowerShell: El parámetro Format-Table Name solo se usa para facilitar la lectura y se puede eliminar o cambiar.
    - Si usa PowerShell, obtendrá un error si intenta mostrar las configuraciones de un recurso que no tiene ninguna configuración asociada. El error será similar al siguiente: "Error en la operación con el estado: 'No se encuentra'. Detalles: error de cliente 404: no se encuentra la URL".

    
## <a name="prepare-to-move"></a>Preparación para la migración

1. Antes de empezar, defina estas variables. Hemos proporcionado un ejemplo para cada una.

    **Variable** | **Detalles** | **Ejemplo**
    --- | ---
    $subId | Id. de la suscripción que contiene las configuraciones de mantenimiento | "our-subscription-ID"
    $rsrcGroupName | Nombre del grupo de recursos (VM de Azure) | "VMResourceGroup"
    $vmName | Nombre del recurso de la VM |  "myVM"
    $adhRsrcGroupName |  Grupo de recursos (hosts dedicados) | "HostResourceGroup"
    $adh | Nombre del host dedicado | "myHost"
    $adhParentName | Nombre del recursos primario | "HostGroup"
    
2. Para recuperar las configuraciones de mantenimiento mediante el comando [Get-AZConfigurationAssignment](/powershell/module/az.maintenance/get-azconfigurationassignment) de PowerShell:

    - Para las instancias de Azure Dedicated Host, ejecute:
        ```
        Get-AzConfigurationAssignment -ResourceGroupName $adhRsrcGroupName -ResourceName $adh -ResourceType hosts -ProviderName Microsoft.Compute -ResourceParentName $adhParentName -ResourceParentType hostGroups | Format-Table Name
        ```

    - Para las VM de Azure, ejecute:

        ```
        Get-AzConfigurationAssignment -ResourceGroupName $rgName -ResourceName $vmName -ProviderName Microsoft.Compute -ResourceType virtualMachines | Format-Table Name
        ```
3. Para recuperar las configuraciones de mantenimiento mediante el comando [az maintenance assignment](/cli/azure/maintenance/assignment) de la CLI:

    - Para las instancias de Azure Dedicated Host:

        ```
        az maintenance assignment list --subscription $subId --resource-group $adhRsrcGroupName --resource-name $adh --resource-type hosts --provider-name Microsoft.Compute --resource-parent-name $adhParentName --resource-parent-type hostGroups --query "[].{HostResourceGroup:resourceGroup,ConfigName:name}" --output table
        ```

    - Para las VM de Azure:

        ```
        az maintenance assignment list --subscription $subId --provider-name Microsoft.Compute --resource-group $rsrcGroupName --resource-name $vmName --resource-type virtualMachines --query "[].{HostResourceGroup:resourceGroup, ConfigName:name}" --output table
        ```


## <a name="move"></a>Move 

1. [Siga estas instrucciones](../site-recovery/azure-to-azure-tutorial-migrate.md?toc=/azure/virtual-machines/windows/toc.json&bc=/azure/virtual-machines/windows/breadcrumb/toc.json) para mover las VM de Azure a la nueva región.
2. Una vez que se muevan los recursos, vuelva a aplicar las configuraciones de mantenimiento a los recursos de la nueva región según corresponda, en función de si ha movido las configuraciones de mantenimiento. Puede aplicar una configuración de mantenimiento a un recurso mediante [PowerShell](../virtual-machines/maintenance-control-powershell.md) o la [CLI](../virtual-machines/maintenance-control-cli.md).


## <a name="verify-the-move"></a>Comprobación del traslado

Compruebe los recursos de la nueva región y las configuraciones asociadas para los recursos en dicha región. 

## <a name="clean-up-source-resources"></a>Limpieza de los recursos de origen

Después del traslado, considere la posibilidad de eliminar los recursos trasladados en la región de origen.


## <a name="next-steps"></a>Pasos siguientes

Siga [estas instrucciones](move-region-maintenance-configuration.md) si necesita trasladar configuraciones de mantenimiento. 
