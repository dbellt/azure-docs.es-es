---
title: Incorporación de una identidad administrada a un tipo de nodo de clúster administrado de Service Fabric
description: En este artículo se muestra cómo agregar una identidad administrada a un tipo de nodo de clúster administrado de Service Fabric
ms.topic: how-to
ms.date: 5/10/2021
ms.openlocfilehash: 9b83d0a7e68e162eb3231d4605755e74dc7f9fe4
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685322"
---
# <a name="add-a-managed-identity-to-a-service-fabric-managed-cluster-node-type"></a>Incorporación de una identidad administrada a un tipo de nodo de clúster administrado de Service Fabric

Cada tipo de nodo de un clúster administrado de Service Fabric se encuentra respaldado por un conjunto de escalado de máquinas virtuales. Para permitir el uso de identidades administradas con un tipo de nodo de clúster administrado, se ha agregado una propiedad `vmManagedIdentity` a las definiciones de tipo de nodo que contienen una lista de identidades que se pueden usar, `userAssignedIdentities`. La funcionalidad refleja cómo se pueden usar las identidades administradas en los clústeres no administrados, como el uso de una identidad administrada con la [extensión del conjunto de escalado de máquinas virtuales de Azure Key Vault](../virtual-machines/extensions/key-vault-windows.md).

Para ver un ejemplo de una implementación de clústeres administrados de Service Fabric que usa la identidad administrada en un tipo de nodo, consulte [esta plantilla](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/SF-Managed-Standard-SKU-1-NT-MI).

> [!NOTE]
> Actualmente, solo se admiten identidades asignadas por el usuario para esta característica.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar:

* Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Si tiene previsto usar PowerShell, [instale](/cli/azure/install-azure-cli) la CLI de Azure para ejecutar los comandos de referencia de la CLI.

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario

Una identidad administrada asignada por el usuario se puede definir en la sección de recursos de una plantilla de Azure Resource Manager (ARM) para su creación tras la implementación:

```JSON
{ 
    "type": "Microsoft.ManagedIdentity/userAssignedIdentities", 
    "name": "[parameters('userAssignedIdentityName')]", 
    "apiVersion": "2018-11-30", 
    "location": "[resourceGroup().location]"  
},
```

O crearse a través de PowerShell:

```powershell
az group create --name <resourceGroupName> --location <location>
az identity create --name <userAssignedIdentityName> --resource-group <resourceGroupName>
```

## <a name="add-a-role-assignment-with-service-fabric-resource-provider"></a>Adición de una asignación de roles con el proveedor de recursos de Service Fabric

Agregue una asignación de roles a la identidad administrada con la aplicación de proveedor de recursos de Service Fabric. Esta asignación permite al proveedor de recursos de Service Fabric asignar la identidad al conjunto de escalado de máquinas virtuales del clúster administrado. 

Obtenga la entidad de servicio de la aplicación del proveedor de recursos de Service Fabric:

```powershell
Login-AzAccount
Select-AzSubscription -SubscriptionId <SubId>
Get-AzADServicePrincipal -DisplayName "Azure Service Fabric Resource Provider"
```

> [!NOTE]
> Asegúrese de que se encuentra en la suscripción correcta; el identificador de entidad de seguridad cambia si la suscripción está en otro inquilino.

```powershell
ServicePrincipalNames : {74cb6831-0dbb-4be1-8206-fd4df301cdc2}
ApplicationId         : 74cb6831-0dbb-4be1-8206-fd4df301cdc2
ObjectType            : ServicePrincipal
DisplayName           : Azure Service Fabric Resource Provider
Id                    : 00000000-0000-0000-0000-000000000000
Type                  :
```

Use el identificador de la salida anterior como **principalId** y el identificador de definición de roles de debajo como **roleDefinitionId** cuando corresponda en la plantilla o el comando de PowerShell:

|El nombre de la definición de roles|Id. de definición de roles|
|----|-------------------------------------|
|Operador de identidad administrada|f1a07417-d97a-45cb-824c-7a7467783830|


Esta asignación de roles se puede definir en la plantilla de la sección de recursos mediante el identificador de entidad de seguridad y el identificador de definición de roles:

```JSON
{
    "type": "Microsoft.Authorization/roleAssignments", 
    "apiVersion": "2020-04-01-preview",
    "name": "[parameters('vmIdentityRoleNameGuid')]",
    "scope": "[concat('Microsoft.ManagedIdentity/userAssignedIdentities', '/', parameters('userAssignedIdentityName'))]",
    "dependsOn": [ 
        "[concat('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName'))]"
    ], 
    "properties": {
        "roleDefinitionId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'f1a07417-d97a-45cb-824c-7a7467783830')]",
        "principalId": "00000000-0000-0000-0000-000000000000" 
    } 
}, 
```
> [!NOTE]
> vmIdentityRoleNameGuid debe ser un GUID válido. Si implementa de nuevo la misma plantilla, incluida esta asignación de roles, asegúrese de que el GUID sea el mismo que el usado originalmente o quite este recurso, ya que solo debe crearse una vez.

o crearse por medio de PowerShell con el identificador de entidad de seguridad y el nombre de definición de roles:

```powershell
New-AzRoleAssignment -PrincipalId 00000000-0000-0000-0000-000000000000 -RoleDefinitionName "Managed Identity Operator" -Scope "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<userAssignedIdentityName>"
```

## <a name="add-managed-identity-properties-to-node-type-definition"></a>Adición de las propiedades de identidad administrada a la definición de tipo de nodo

Por último, agregue las propiedades `vmManagedIdentity` y `userAssignedIdentities` a la definición de tipo de nodo del clúster administrado. Asegúrese de usar **2021-05-01** o posterior para `apiVersion`.

```json

 {
    "type": "Microsoft.ServiceFabric/managedclusters/nodetypes",
    "apiVersion": "2021-05-01",
    ...
    "properties": {
        "isPrimary" : true,
        "vmInstanceCount": 5,
        "dataDiskSizeGB": 100,
        "vmSize": "Standard_D2_v2",
        "vmImagePublisher" : "MicrosoftWindowsServer",
        "vmImageOffer" : "WindowsServer",
        "vmImageSku" : "2019-Datacenter",
        "vmImageVersion" : "latest",
        "vmManagedIdentity": {
            "userAssignedIdentities": [
                "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('userAssignedIdentityName'))]"
            ]
        }
    }
}
```

Después de la implementación, la identidad administrada creada se ha agregado al conjunto de escalado de máquinas virtuales del tipo de nodo designado y se puede usar como se espera, al igual que en cualquier clúster no administrado.

## <a name="troubleshooting"></a>Solución de problemas

No se puede agregar correctamente una asignación de roles con el siguiente error en la implementación:

:::image type="content" source="media/how-to-managed-identity-managed-cluster-vmss/role-assignment-error.png" alt-text="Error de implementación de Azure Portal que muestra el cliente con el identificador de objeto o aplicación de SFRP que no tiene permiso para realizar la actividad de administración de identidades":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Implementación de una aplicación en un clúster administrado de Service Fabric](./tutorial-managed-cluster-deploy-app.md)