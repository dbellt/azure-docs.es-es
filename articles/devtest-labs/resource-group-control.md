---
title: Especificación del grupo de recursos para VM en Azure DevTest Labs | Microsoft Docs
description: Obtenga información sobre cómo especificar un grupo de recursos para VM en un laboratorio en Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: c6f576a20fc8fada9dd515e8ba2a266761a3e586
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107377495"
---
# <a name="specify-a-resource-group-for-lab-virtual-machines-in-azure-devtest-labs"></a>Especificación de un grupo de recursos para máquinas virtuales de laboratorio en Azure DevTest Labs

Como propietario de un laboratorio, puede configurar las máquinas virtuales del laboratorio para que se creen en un grupo de recursos específico. Esta característica sirve de ayuda en los siguientes escenarios:

- Hay menos grupos de recursos creados por los laboratorios en la suscripción.
- Los laboratorios se ejecutan dentro de un conjunto fijo de grupos de recursos que se configura.
- Se evitan las restricciones y las aprobaciones necesarias para crear grupos de recursos dentro de la suscripción de Azure.
- Se consolidan todos los recursos de laboratorio dentro de un grupo de recursos único para simplificar el seguimiento de esos recursos y aplicar [directivas](../governance/policy/overview.md) a fin de administrarlos en el grupo de recursos.

Con esta característica, puede usar un script para especificar un grupo de recursos nuevo o uno existente dentro de su suscripción de Azure para todas las máquinas virtuales del laboratorio. Actualmente, Azure DevTest Labs admite esta característica con una API.

> [!NOTE]
> Todos los límites de suscripción se aplican al crear laboratorios de DevTest Labs. Piense en un laboratorio como cualquier otro recurso en su suscripción. En el caso de los grupos de recursos, el límite es de [980 grupos de recursos por suscripción](../azure-resource-manager/management/azure-subscription-service-limits.md#subscription-limits). 

## <a name="use-azure-portal"></a>Usar Azure Portal
Siga estos pasos para especificar un grupo de recursos para todas las máquinas virtuales creadas en el laboratorio. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** en el menú de navegación izquierdo. 
3. Seleccione **DevTest Labs** en la lista.
4. En la lista de laboratorios, seleccione el **suyo**.  
5. Seleccione **Configuración y directivas** en la sección **Configuración** en el menú de la izquierda. 
6. Seleccione **Configuración de laboratorio** en el menú de la izquierda. 
7. Seleccione **Todas las máquinas virtuales en un grupo de recursos**. 
8. Seleccione un grupo de recursos existente en la lista desplegable o bien seleccione **Crear nuevo**, escriba un **nombre** para él y seleccione **Aceptar**. 

    ![Seleccione el grupo de recursos para todas las máquinas virtuales del laboratorio.](./media/resource-group-control/select-resource-group.png)

## <a name="use-powershell"></a>Uso de PowerShell 
El ejemplo siguiente muestra cómo usar el script de PowerShell para crear todas las máquinas virtuales de laboratorio en un nuevo grupo de recursos.

```powershell
[CmdletBinding()]
Param(
    $subId,
    $labRg,
    $labName,
    $vmRg
)

az login | out-null

az account set --subscription $subId | out-null

$rgId = "/subscriptions/"+$subId+"/resourceGroups/"+$vmRg

"Updating lab '$labName' with vm rg '$rgId'..."

az resource update -g $labRg -n $labName --resource-type "Microsoft.DevTestLab/labs" --api-version 2018-10-15-preview --set properties.vmCreationResourceGroupId=$rgId

"Done. New virtual machines will now be created in the resource group '$vmRg'."
```

Invoque el script con el siguiente comando. ResourceGroup.ps1 es el archivo que contiene el script anterior:

```powershell
.\ResourceGroup.ps1 -subId <subscriptionID> -labRg <labRGNAme> -labName <LanName> -vmRg <RGName> 
```

## <a name="use-an-azure-resource-manager-template"></a>Uso de una plantilla de Azure Resource Manager
Si usa la plantilla de Azure Resource Manager para crear un laboratorio, use la propiedad **vmCreationResourceGroupId** en la sección de propiedades del laboratorio de la plantilla, como se muestra en el ejemplo siguiente:

```json
        {
            "type": "microsoft.devtestlab/labs",
            "name": "[parameters('lab_name')]",
            "apiVersion": "2018-10-15-preview",
            "location": "eastus",
            "tags": {},
            "scale": null,
            "properties": {
                "vmCreationResourceGroupId": "/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>",
                "labStorageType": "Premium",
                "premiumDataDisks": "Disabled",
                "provisioningState": "Succeeded",
                "uniqueIdentifier": "000000000f-0000-0000-0000-00000000000000"
            },
            "dependsOn": []
        },
```


## <a name="api-to-configure-a-resource-group-for-lab-vms"></a>API para configurar un grupo de recursos para las máquinas virtuales del laboratorio
Tiene las siguientes opciones como propietario de un laboratorio cuando se usa esta API:

- Elija el **grupo de recursos del laboratorio** para todas las máquinas virtuales.
- Elija un **grupo de recursos existente** distinto del grupo de recursos del laboratorio para todas las máquinas virtuales.
- Escriba un nombre del **nuevo grupo de recursos** para todas las máquinas virtuales.
- Continúe con el comportamiento actual, es decir, se crea un grupo de recursos para cada VM en el laboratorio.
 
Esta configuración se aplica a nuevas máquinas virtuales creadas en el laboratorio. Las máquinas virtuales antiguas en el laboratorio creadas en sus propios grupos de recursos seguirán sin verse afectadas. Los entornos creados en su laboratorio se mantienen en sus propios grupos de recursos.

Cómo usar esta API:
- Use la versión de API **2018-10-15-preview**.
- Si especifica un nuevo grupo de recursos, asegúrese de tener **permisos de escritura en grupos de recursos** en su suscripción. Si carece de permisos de escritura, se genera un error al crear nuevas máquinas virtuales en el grupo de recursos especificado.
- Al usar la API, pase el **id. del grupo de recursos completo**. Por ejemplo: `/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroupName>`. Asegúrese de que el grupo de recursos esté en la misma suscripción que el laboratorio. 


## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes: 

- [Set policies for a lab](devtest-lab-set-lab-policy.md) (Definición de directivas para un laboratorio)
- [Preguntas más frecuentes](devtest-lab-faq.md)
