---
title: Uso de identidades administradas en Azure API Management | Microsoft Docs
description: Obtenga información sobre cómo crear identidades asignadas por el sistema y asignadas por el usuario en API Management mediante Azure Portal, PowerShell y una plantilla de Resource Manager.
services: api-management
documentationcenter: ''
author: miaojiang
manager: anneta
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: article
ms.date: 03/09/2021
ms.author: apimpm
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1dc0c67a173333f2d2bcabd71d92ec88fa5a467d
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110071783"
---
# <a name="use-managed-identities-in-azure-api-management"></a>Uso de identidades administradas en Azure API Management

En este artículo se muestra cómo crear una identidad administrada para una instancia de Azure API Management y cómo acceder a otros recursos. Una identidad administrada generada por Azure Active Directory (Azure AD) permite a la instancia de API Management acceder de forma fácil y segura a otros recursos protegidos de Azure AD, como Azure Key Vault. Azure administra esta identidad, por lo que usted no tiene que aprovisionar ni rotar ningún secreto. Para obtener más información sobre las identidades administradas, consulte el artículo sobre [Qué son las identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md).

Puede conceder dos tipos de identidades a una instancia de API Management:

- Una *identidad asignada por el sistema* está asociada al servicio y se elimina si se elimina el servicio. El servicio solo puede tener una identidad asignada por el sistema.
- Una *identidad asignada por el usuario* es un recurso de Azure independiente que puede asignarse al servicio. El servicio puede tener varias identidades asignadas por el usuario.

## <a name="create-a-system-assigned-managed-identity"></a>Creación de una identidad administrada asignada por el sistema

### <a name="azure-portal"></a>Azure portal

Para configurar una identidad administrada en Azure Portal, primero tendrá que crear una instancia de API Management y, a continuación, habilitar la característica.

1. Cree una instancia de API Management en el portal como lo haría normalmente. Búsquela en el portal.
2. Seleccione **Identidades administradas**.
3. En la pestaña **Asignado por el sistema**, cambie **Estado** a **Activado**. Seleccione **Guardar**.

    :::image type="content" source="./media/api-management-msi/enable-system-msi.png" alt-text="Selecciones para habilitar una identidad administrada asignada por el sistema" border="true":::

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Los siguientes pasos le guiarán por la creación de una instancia de API Management y la asignación de una identidad a esta mediante Azure PowerShell.

1. Si es necesario, instale Azure PowerShell con las instrucciones que se encuentra en la [Guía de Azure PowerShell](/powershell/azure/install-az-ps). Luego, ejecute `Connect-AzAccount` para crear una conexión con Azure.

2. Use el código siguiente para crear la instancia. Para ver más ejemplos de cómo utilizar Azure PowerShell con la instancia de API Management, consulte los [ejemplos de PowerShell de API Management](powershell-samples.md).

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create an API Management Consumption Sku service.
    New-AzApiManagement -ResourceGroupName $resourceGroupName -Name consumptionskuservice -Location $location -Sku Consumption -Organization contoso -AdminEmail contoso@contoso.com -SystemAssignedIdentity
    ```

3. Actualice una instancia existente para crear la identidad:

    ```azurepowershell-interactive
    # Get an API Management instance
    $apimService = Get-AzApiManagement -ResourceGroupName $resourceGroupName -Name $apiManagementName

    # Update an API Management instance
    Set-AzApiManagement -InputObject $apimService -SystemAssignedIdentity
    ```

### <a name="azure-resource-manager-template"></a>Plantilla del Administrador de recursos de Azure

Puede crear una instancia de API Management con una identidad mediante la inclusión de la siguiente propiedad en la definición de recursos:

```json
"identity" : {
    "type" : "SystemAssigned"
}
```

Esta propiedad indica a Azure que debe crear y administrar la identidad para la instancia de API Management.

Por ejemplo, una plantilla de Azure Resource Manager completa podría tener el aspecto siguiente:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "0.9.0.0",
    "resources": [{
        "apiVersion": "2019-01-01",
        "name": "contoso",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {},
        "sku": {
            "name": "Developer",
            "capacity": "1"
        },
        "properties": {
            "publisherEmail": "admin@contoso.com",
            "publisherName": "Contoso"
        },
        "identity": {
            "type": "systemAssigned"
        }
    }]
}
```

Cuando se crea la instancia, tiene las siguientes propiedades adicionales:

```json
"identity": {
    "type": "SystemAssigned",
    "tenantId": "<TENANTID>",
    "principalId": "<PRINCIPALID>"
}
```

La propiedad `tenantId` identifica a qué inquilino de Azure AD pertenece la identidad. La propiedad `principalId` es un identificador único para la nueva identidad de la instancia. En Azure AD, la entidad de servicio tiene el mismo nombre que asignó a la instancia de API Management.

> [!NOTE]
> Una instancia de API Management puede tener identidades asignadas por el sistema y asignadas por el usuario al mismo tiempo. En este caso, la propiedad `type` sería `SystemAssigned,UserAssigned`.

## <a name="supported-scenarios-using-system-assigned-identity"></a>Escenarios admitidos que usan la identidad asignada por el sistema

### <a name="obtain-a-custom-tlsssl-certificate-for-the-api-management-instance-from-azure-key-vault"></a><a name="use-ssl-tls-certificate-from-azure-key-vault"></a>Obtención de un certificado TLS/SSL personalizado para la instancia de API Management desde Azure Key Vault
Puede usar la identidad asignada por el sistema de una instancia de API Management para recuperar los certificados de TLS/SSL personalizados almacenados en Azure Key Vault. Después, puede asignar estos certificados a dominios personalizados en la instancia de API Management. Tenga en cuenta las consideraciones siguientes:

- El tipo de contenido del secreto debe ser *application/x-pkcs12*.
- Use el punto de conexión del secreto de certificado de Key Vault, que contiene el secreto.

> [!Important]
> Si no se proporciona la versión del objeto del certificado, API Management obtendrá automáticamente la versión más reciente del certificado dentro del plazo de cuatro horas a partir de que se cargue en Key Vault.

En el ejemplo siguiente se muestra una plantilla de Azure Resource Manager que contiene los siguientes pasos:

1. Creación de una instancia de API Management con una identidad administrada.
2. Actualización de las directivas de acceso de una instancia de Azure Key Vault y permiso para que la instancia de API Management obtenga secretos de ella.
3. Actualización de la instancia de API Management estableciendo un nombre de dominio personalizado mediante un certificado de la instancia de Key Vault.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publisherEmail": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "The email address of the owner of the service"
            }
        },
        "publisherName": {
            "type": "string",
            "defaultValue": "Contoso",
            "minLength": 1,
            "metadata": {
                "description": "The name of the owner of the service"
            }
        },
        "sku": {
            "type": "string",
            "allowedValues": ["Developer",
            "Standard",
            "Premium"],
            "defaultValue": "Developer",
            "metadata": {
                "description": "The pricing tier of this API Management instance"
            }
        },
        "skuCount": {
            "type": "int",
            "defaultValue": 1,
            "metadata": {
                "description": "The instance size of this API Management instance."
            }
        },
        "keyVaultName": {
            "type": "string",
            "metadata": {
                "description": "Name of the vault"
            }
        },
        "proxyCustomHostname1": {
            "type": "string",
            "metadata": {
                "description": "Proxy Custom hostname."
            }
        },
        "keyVaultIdToCertificate": {
            "type": "string",
            "metadata": {
                "description": "Reference to the Key Vault certificate. https://contoso.vault.azure.net/secrets/contosogatewaycertificate."
            }
        }
    },
    "variables": {
        "apiManagementServiceName": "[concat('apiservice', uniqueString(resourceGroup().id))]",
        "apimServiceIdentityResourceId": "[concat(resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName')),'/providers/Microsoft.ManagedIdentity/Identities/default')]"
    },
    "resources": [{
        "apiVersion": "2019-01-01",
        "name": "[variables('apiManagementServiceName')]",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {
        },
        "sku": {
            "name": "[parameters('sku')]",
            "capacity": "[parameters('skuCount')]"
        },
        "properties": {
            "publisherEmail": "[parameters('publisherEmail')]",
            "publisherName": "[parameters('publisherName')]"
        },
        "identity": {
            "type": "systemAssigned"
        }
    },
    {
        "type": "Microsoft.KeyVault/vaults/accessPolicies",
        "name": "[concat(parameters('keyVaultName'), '/add')]",
        "apiVersion": "2015-06-01",
        "dependsOn": [
            "[resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName'))]"
        ],
        "properties": {
            "accessPolicies": [{
                "tenantId": "[reference(variables('apimServiceIdentityResourceId'), '2015-08-31-PREVIEW').tenantId]",
                "objectId": "[reference(variables('apimServiceIdentityResourceId'), '2015-08-31-PREVIEW').principalId]",
                "permissions": {
                    "secrets": ["get"]
                }
            }]
        }
    },
    {
        "apiVersion": "2017-05-10",
        "name": "apimWithKeyVault",
        "type": "Microsoft.Resources/deployments",
        "dependsOn": [
        "[resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName'))]"
        ],
        "properties": {
            "mode": "incremental",
            "templateLink": {
                "uri": "https://raw.githubusercontent.com/solankisamir/arm-templates/master/basicapim.keyvault.json",
                "contentVersion": "1.0.0.0"
            },
            "parameters": {
                "publisherEmail": { "value": "[parameters('publisherEmail')]"},
                "publisherName": { "value": "[parameters('publisherName')]"},
                "sku": { "value": "[parameters('sku')]"},
                "skuCount": { "value": "[parameters('skuCount')]"},
                "proxyCustomHostname1": {"value" : "[parameters('proxyCustomHostname1')]"},
                "keyVaultIdToCertificate": {"value" : "[parameters('keyVaultIdToCertificate')]"}
            }
        }
    }]
}
```

### <a name="authenticate-to-the-back-end-by-using-an-api-management-identity"></a>Autenticación en el back-end mediante una identidad de API Management

Puede usar la identidad asignada por el sistema para autenticarse en el back-end a través de la directiva [authentication-managed-identity](api-management-authentication-policies.md#ManagedIdentity).

### <a name="connect-to-azure-resources-behind-ip-firewall-using-system-assigned-managed-identity"></a><a name="apim-as-trusted-service"></a>Conexión a recursos de Azure detrás del firewall de IP mediante una identidad administrada asignada por el sistema


API Management es un servicio de confianza de Microsoft para los siguientes recursos. Esto permite que el servicio se conecte a los siguientes recursos detrás de un firewall. Después de asignar explícitamente el rol de Azure adecuado a la [identidad administrada asignada por el sistema](../active-directory/managed-identities-azure-resources/overview.md) para esa instancia de recurso, el ámbito de acceso para la instancia corresponde al rol de Azure asignado a la identidad administrada.


|Servicio de Azure | Vínculo|
|---|---|
|Azure Storage | [Acceso de confianza a Azure Storage](../storage/common/storage-network-security.md?tabs=azure-portal#trusted-access-based-on-system-assigned-managed-identity)|
|Azure Service Bus | [Acceso de confianza a Azure Service Bus](../service-bus-messaging/service-bus-ip-filtering.md#trusted-microsoft-services)|
|Centro de eventos de Azure | [Acceso de confianza a Azure Event Hub](../event-hubs/event-hubs-ip-filtering.md#trusted-microsoft-services)|


## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario

> [!NOTE]
> Puede asociar una instancia de API Management con hasta 10 identidades administradas asignadas por el usuario.

### <a name="azure-portal"></a>Azure portal

Para configurar una identidad administrada en el portal, primero tendrá que crear una instancia de API Management y, a continuación, habilitar la característica.

1. Cree una instancia de API Management en el portal como lo haría normalmente. Búsquela en el portal.
2. Seleccione **Identidades administradas**.
3. En la pestaña **Usuario asignado**, seleccione **Agregar**.
4. Busque la identidad que creó anteriormente y selecciónela. Seleccione **Agregar**.

   :::image type="content" source="./media/api-management-msi/enable-user-assigned-msi.png" alt-text="Selecciones para habilitar una identidad administrada asignada por el usuario" border="true":::

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Los siguientes pasos le guiarán por la creación de una instancia de API Management y la asignación de una identidad a esta mediante Azure PowerShell.

1. Si es necesario, instale Azure PowerShell con las instrucciones que se encuentra en la [Guía de Azure PowerShell](/powershell/azure/install-az-ps). Luego, ejecute `Connect-AzAccount` para crear una conexión con Azure.

2. Use el código siguiente para crear la instancia. Para ver más ejemplos de cómo utilizar Azure PowerShell con la instancia de API Management, consulte los [ejemplos de PowerShell de API Management](powershell-samples.md).

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create a user-assigned identity. This requires installation of the "Az.ManagedServiceIdentity" module.
    $userAssignedIdentity = New-AzUserAssignedIdentity -Name $userAssignedIdentityName -ResourceGroupName $resourceGroupName

    # Create an API Management Consumption Sku service.
    $userIdentities = @($userAssignedIdentity.Id)

    New-AzApiManagement -ResourceGroupName $resourceGroupName -Location $location -Name $apiManagementName -Organization contoso -AdminEmail admin@contoso.com -Sku Consumption -UserAssignedIdentity $userIdentities
    ```

3. Actualice un servicio existente para asignar una identidad al servicio:

    ```azurepowershell-interactive
    # Get an API Management instance
    $apimService = Get-AzApiManagement -ResourceGroupName $resourceGroupName -Name $apiManagementName

    # Create a user-assigned identity. This requires installation of the "Az.ManagedServiceIdentity" module.
    $userAssignedIdentity = New-AzUserAssignedIdentity -Name $userAssignedIdentityName -ResourceGroupName $resourceGroupName

    # Update an API Management instance
    $userIdentities = @($userAssignedIdentity.Id)
    Set-AzApiManagement -InputObject $apimService -UserAssignedIdentity $userIdentities
    ```

### <a name="azure-resource-manager-template"></a>Plantilla del Administrador de recursos de Azure

Puede crear una instancia de API Management con una identidad mediante la inclusión de la siguiente propiedad en la definición de recursos:

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {}
    }
}
```

Al agregar el tipo asignado por el usuario se indica a Azure que use la identidad asignada por el usuario especificada para la instancia.

Por ejemplo, una plantilla de Azure Resource Manager completa podría tener el aspecto siguiente:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "0.9.0.0",
    "resources": [{
        "apiVersion": "2019-12-01",
        "name": "contoso",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {},
        "sku": {
            "name": "Developer",
            "capacity": "1"
        },
        "properties": {
            "publisherEmail": "admin@contoso.com",
            "publisherName": "Contoso"
        },
        "identity": {
            "type": "UserAssigned",
             "userAssignedIdentities": {
                "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]": {}
             }
        },
         "dependsOn": [
          "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]"
        ]
    }]
}
```

Cuando se crea el servicio, tiene las siguientes propiedades adicionales:

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {
            "principalId": "<PRINCIPALID>",
            "clientId": "<CLIENTID>"
        }
    }
}
```

El valor `principalId` es un identificador único de la identidad que se usa para la administración de Azure AD. El la propiedad `clientId` es un identificador único de la nueva identidad de la aplicación que se usa para especificar qué identidad utilizar durante las llamadas de runtime.

> [!NOTE]
> Una instancia de API Management puede tener identidades asignadas por el sistema y asignadas por el usuario al mismo tiempo. En este caso, la propiedad `type` sería `SystemAssigned,UserAssigned`.

## <a name="supported-scenarios-using-user-assigned-managed-identity"></a>Escenarios admitidos que usan la identidad administrada asignada por el usuario

### <a name="obtain-a-custom-tlsssl-certificate-for-the-api-management-instance-from-azure-key-vault"></a><a name="use-ssl-tls-certificate-from-azure-key-vault-ua"></a>Obtención de un certificado TLS/SSL personalizado para la instancia de API Management desde Azure Key Vault
Puede usar cualquier identidad asignada por el usuario para establecer la confianza entre una instancia de API Management y Key Vault. Esta confianza se puede usar para recuperar los certificados TLS/SSL personalizados almacenados en Azure Key Vault. Después, puede asignar estos certificados a dominios personalizados en la instancia de API Management.

Tenga en cuenta las consideraciones siguientes:

- El tipo de contenido del secreto debe ser *application/x-pkcs12*.
- Use el punto de conexión del secreto de certificado de Key Vault, que contiene el secreto.

> [!Important]
> Si no se proporciona la versión del objeto del certificado, API Management obtendrá automáticamente la versión más reciente del certificado dentro del plazo de cuatro horas a partir de que se cargue en Key Vault.

Para ver la plantilla completa, consulte [API Management con SSL basado en KeyVault mediante identidades asignadas por el usuario](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.apimanagement/api-management-key-vault-create/azuredeploy.json).

En esta plantilla, implementará lo siguiente:

* Azure API Management
* Identidad asignada por el usuario administrada por Azure
* Azure Key Vault para almacenar el certificado SSL/TLS

Para ejecutar automáticamente la implementación, haga clic en el botón siguiente:

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.apimanagement%2Fapi-management-key-vault-create%2Fazuredeploy.json)

### <a name="authenticate-to-the-back-end-by-using-a-user-assigned-identity"></a>Autenticación en el back-end mediante una identidad de asignada por el usuario

Puede usar la identidad asignada por el usuario para autenticarse en el back-end a través de la directiva [authentication-managed-identity](api-management-authentication-policies.md#ManagedIdentity).

## <a name="remove-an-identity"></a><a name="remove"></a>Eliminación de una identidad

Para quitar una identidad asignada por el sistema, puede deshabilitar la característica a través del portal o de la plantilla de Azure Resource Manager de la misma manera en que la creó. Las identidades asignadas por el usuario se pueden quitar individualmente. Para quitar todas las identidades, establezca el tipo de identidad en `"None"`.

Al quitar una identidad asignada por el sistema de esta manera también se eliminará de Azure AD. Las identidades asignadas por el sistema también se quitan automáticamente de Azure AD cuando se elimina la instancia de API Management.

Para quitar todas las identidades mediante la plantilla de Azure Resource Manager, actualice esta sección:

```json
"identity": {
    "type": "None"
}
```

> [!Important]
> Si una instancia de API Management se configura con un certificado SSL personalizado de Key Vault e intenta deshabilitar una identidad administrada, se producirá un error en la solicitud.
>
> Para desbloquearse, puede cambiar de un certificado de Azure Key Vault a un certificado codificado insertado y, a continuación, deshabilitar la identidad administrada. Para obtener más información, consulte [Configuración de un nombre de dominio personalizado](configure-custom-domain.md).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las identidades administradas para recursos de Azure:

* [¿Qué son las identidades administradas de recursos de Azure?](../active-directory/managed-identities-azure-resources/overview.md)
* [Plantillas del Administrador de recursos de Azure](https://github.com/Azure/azure-quickstart-templates)
* [Autenticación con una identidad administrada en una directiva](./api-management-authentication-policies.md#ManagedIdentity)
