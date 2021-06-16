---
title: Aplicación administrada con identidad administrada
description: Configure la aplicación administrada con identidad administrada para vincularla a recursos existentes, administrar recursos de Azure y proporcionar identidad operativa para el registro de actividad.
ms.topic: conceptual
ms.author: jobreen
author: jjbfour
ms.date: 05/13/2019
ms.openlocfilehash: 018995ed973679a617b409b82e076198a558eef7
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111968774"
---
# <a name="azure-managed-application-with-managed-identity"></a>Aplicación administrada de Azure con identidad administrada

> [!NOTE]
> La compatibilidad de identidad administrada para aplicaciones administradas está actualmente en versión preliminar. Use la versión de api 2018-09-01-preview para usar la identidad administrada.

Obtenga información sobre cómo configurar una aplicación administrada para contener una identidad administrada. La identidad administrada puede usarse para permitir que el cliente conceda acceso a las aplicaciones administradas a recursos existentes adicionales. La identidad está administrada por la plataforma Azure y no requiere que aprovisione o rote los secretos. Para más información sobre las identidades administradas en Azure Active Directory (AAD), consulte [Identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md).

La aplicación puede tener dos tipos de identidades:

- Una **identidad asignada por el sistema** está asociada a la aplicación y se elimina si se elimina la aplicación. Una aplicación solo puede tener una identidad asignada por el sistema.
- Una **identidad asignada por el usuario** es un recurso de Azure independiente que puede asignarse a la aplicación. Una aplicación puede tener varias identidades asignadas por el usuario.

## <a name="how-to-use-managed-identity"></a>Cómo usar la identidad administrada

Identidad administrada permite muchos escenarios para aplicaciones administradas. Algunos escenarios comunes que pueden resolverse son:

- Implementar una aplicación administrada vinculada a recursos de Azure existentes. Un ejemplo es implementar una máquina virtual (VM) de Azure dentro de la aplicación administrada que se conecta a una [interfaz de red existente](../../virtual-network/virtual-network-network-interface-vm.md).
- Conceder acceso a la aplicación administrada y al editor a los recursos de Azure fuera del **grupo de recursos administrado**.
- Proporcionar una identidad operativa de aplicaciones administradas para el registro de actividad y otros servicios dentro de Azure.

## <a name="adding-managed-identity"></a>Agregar la identidad administrada

Para crear una aplicación administrada con una identidad administrada se requiere la configuración de una propiedad adicional en el recurso de Azure. A continuación se muestra una propiedad **identity** de ejemplo:

```json
{
"identity": {
    "type": "SystemAssigned, UserAssigned",
    "userAssignedIdentities": {
        "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.ManagedIdentity/userassignedidentites/myuserassignedidentity": {}
    }
}
```

Hay dos maneras comunes de crear una aplicación administrada con **identity**: [CreateUIDefinition.json](./create-uidefinition-overview.md) y [plantillas de Azure Resource Manager](../templates/syntax.md). Para escenarios simples de creación única, debe usarse CreateUIDefinition para habilitar la identidad administrada, ya que proporciona una experiencia enriquecida. Sin embargo, cuando se trabaja con sistemas avanzados o complejos que requieren implementaciones automatizadas o múltiples de aplicación administrada, pueden usarse plantillas.

### <a name="using-createuidefinition"></a>Uso de CreateUIDefinition

Se puede configurar una aplicación administrada con identidad administrada a través de [CreateUIDefinition.json](./create-uidefinition-overview.md). En la [sección Salidas](./create-uidefinition-overview.md#outputs), la clave `managedIdentity` puede usarse para reemplazar la propiedad identity de la plantilla de aplicación administrada. En el siguiente ejemplo se permite la identidad **asignada por el sistema** en la aplicación administrada. Para formar objetos de identidad más complejos, use elementos CreateUIDefinition para pedir entradas al consumidor. Estas entradas se pueden usar para construir aplicaciones administradas con una **identidad asignada por el usuario**.

```json
"outputs": {
    "managedIdentity": { "Type": "SystemAssigned" }
}
```

#### <a name="when-to-use-createuidefinition-for-managed-identity"></a>Cuándo usar CreateUIDefinition para identidad administrada

A continuación se muestran algunas recomendaciones sobre cuándo usar CreateUIDefinition para habilitar la identidad administrada en aplicaciones administradas.

- La creación de aplicaciones administradas se hace a través de Azure Portal o Marketplace.
- La identidad administrada requiere una intervención compleja del consumidor.
- La identidad administrada es necesaria en la creación de la aplicación administrada.

#### <a name="managed-identity-createuidefinition-control"></a>Control CreateUIDefinition de la identidad administrada

CreateUIDefinition admite un [control de identidad administrada](./microsoft-managedidentity-identityselector.md) integrado.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.0.1-preview",
  "parameters": {
    "basics": [],
    "steps": [
      {
        "name": "applicationSettings",
        "label": "Application Settings",
        "subLabel": {
          "preValidation": "Configure your application settings",
          "postValidation": "Done"
        },
        "bladeTitle": "Application Settings",
        "elements": [
          {
            "name": "appName",
            "type": "Microsoft.Common.TextBox",
            "label": "Managed application Name",
            "toolTip": "Managed application instance name",
            "visible": true
          },
          {
            "name": "appIdentity",
            "type": "Microsoft.ManagedIdentity.IdentitySelector",
            "label": "Managed Identity Configuration",
            "toolTip": {
              "systemAssignedIdentity": "Enable system assigned identity to grant the managed application access to additional existing resources.",
              "userAssignedIdentity": "Add user assigned identities to grant the managed application access to additional existing resources."
            },
            "defaultValue": {
              "systemAssignedIdentity": "Off"
            },
            "options": {
              "hideSystemAssignedIdentity": false,
              "hideUserAssignedIdentity": false,
              "readOnlySystemAssignedIdentity": false
            },
            "visible": true
          }
        ]
      }
    ],
    "outputs": {
      "applicationResourceName": "[steps('applicationSettings').appName]",
      "location": "[location()]",
      "managedIdentity": "[steps('applicationSettings').appIdentity]"
    }
  }
}
```

![Control CreateUIDefinition de la identidad administrada](./media/publish-managed-identity/msi-cuid.png)

### <a name="using-azure-resource-manager-templates"></a>Uso de plantillas del Administrador de recursos de Azure

> [!NOTE]
> Las plantillas de aplicación administrada de Marketplace se generan automáticamente para los clientes que siguen la experiencia de creación de Azure Portal.
> Para estos escenarios, la clave de salida `managedIdentity` en la instancia de CreateUIDefinition debe usarse para la identidad habilitada.

Además, la identidad administrada puede habilitarse con plantillas de Azure Resource Manager. En el siguiente ejemplo se permite la identidad **asignada por el sistema** en la aplicación administrada. Los objetos de identidad más complejos se pueden formar mediante los parámetros de plantillas de Azure Resource Manager para proporcionar entradas. Estas entradas se pueden usar para construir aplicaciones administradas con una **identidad asignada por el usuario**.

#### <a name="when-to-use-azure-resource-manager-templates-for-managed-identity"></a>Cuándo usar plantillas de Azure Resource Manager para la identidad administrada

A continuación se muestran algunas recomendaciones sobre cuándo usar plantillas de Azure Resource Manager para habilitar la identidad administrada en aplicaciones administradas.

- Las aplicaciones administradas se pueden implementar mediante programación basada en una plantilla.
- Se necesitan asignaciones de roles personalizadas para la identidad administrada a fin de aprovisionar la aplicación administrada.
- La aplicación administrada no necesita el flujo de creación de Azure Portal y Marketplace.

#### <a name="systemassigned-template"></a>Plantilla SystemAssigned

Una plantilla básica de Azure Resource Manager que implementa una aplicación administrada con la identidad **asignada por el sistema**.

```json
"resources": [
    {
        "type": "Microsoft.Solutions/applications",
        "name": "[parameters('applicationName')]",
        "apiVersion": "2018-09-01-preview",
        "location": "[parameters('location')]",
        "identity": {
            "type": "SystemAssigned"
        },
        "properties": {
            "ManagedResourceGroupId": "[parameters('managedByResourceGroupId')]",
            "parameters": { }
        }
    }
]
```

### <a name="userassigned-template"></a>Plantilla UserAssigned

Una plantilla básica de Azure Resource Manager que implementa una aplicación administrada con la identidad **asignada por el usuario**.

```json
"resources": [
    {
      "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
      "name": "[parameters('managedIdentityName')]",
      "apiVersion": "2018-11-30",
      "location": "[parameters('location')]"
    },
    {
        "type": "Microsoft.Solutions/applications",
        "name": "[parameters('applicationName')]",
        "apiVersion": "2018-09-01-preview",
        "location": "[parameters('location')]",
        "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
                "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',parameters('managedIdentityName'))]": {}
            }
        },
        "properties": {
            "ManagedResourceGroupId": "[parameters('managedByResourceGroupId')]",
            "parameters": { }
        }
    }
]
```

## <a name="granting-access-to-azure-resources"></a>Conceder acceso a los recursos de Azure

Una vez que se concede una identidad a una aplicación administrada, se le puede conceder acceso a recursos de Azure existentes. Este proceso puede realizarse a través de la interfaz de control de acceso (IAM) en Azure Portal. El nombre de la aplicación administrada o la **identidad asignada por el usuario** se pueden buscar para agregar una asignación de roles.

![Agregar una asignación de roles para una aplicación administrada](./media/publish-managed-identity/identity-role-assignment.png)

## <a name="linking-existing-azure-resources"></a>Vincular recursos de Azure existentes

> [!NOTE]
> Un **identidad asignada por el usuario** debe [configurarse](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) antes de implementar la aplicación administrada. Además, la implementación de recursos vinculados de las aplicaciones administradas solo se admite para el tipo **marketplace**.

La identidad administrada también puede usarse para implementar una aplicación administrada que requiera acceso a los recursos existentes durante su implementación. Cuando el cliente aprovisiona la aplicación administrada, las **identidades asignadas por el usuario** pueden agregarse para ofrecer autorizaciones adicionales a la implementación **mainTemplate**.

### <a name="authoring-the-createuidefinition-with-a-linked-resource"></a>Creación de CreateUIDefinition con un recurso vinculado

Al vincular la implementación de la aplicación administrada a los recursos existentes, se deben proporcionar tanto el recurso de Azure existente como una **identidad asignada por el usuario** con la asignación de roles aplicable en dicho recurso.

 Ejemplo de CreateUIDefinition que requiere dos entradas: un identificador de recurso de interfaz de red y un identificador de recurso de identidad asignada por el usuario.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
    "handler": "Microsoft.Compute.MultiVm",
    "version": "0.1.2-preview",
    "parameters": {
        "basics": [
            {}
        ],
        "steps": [
            {
                "name": "managedApplicationSetting",
                "label": "Managed Application Settings",
                "subLabel": {
                    "preValidation": "Managed Application Settings",
                    "postValidation": "Done"
                },
                "bladeTitle": "Managed Application Settings",
                "elements": [
                    {
                        "name": "networkInterfaceId",
                        "type": "Microsoft.Common.TextBox",
                        "label": "network interface resource id",
                        "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Network/networkInterfaces/existingnetworkinterface",
                        "toolTip": "Must represent the identity as an Azure Resource Manager resource identifer format ex. /subscriptions/sub1/resourcegroups/myGroup/providers/Microsoft.Network/networkInterfaces/networkinterface1",
                        "visible": true
                    },
                    {
                        "name": "userAssignedId",
                        "type": "Microsoft.Common.TextBox",
                        "label": "user assigned identity resource id",
                        "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.ManagedIdentity/userassignedidentites/myuserassignedidentity",
                        "toolTip": "Must represent the identity as an Azure Resource Manager resource identifer format ex. /subscriptions/sub1/resourcegroups/myGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/identity1",
                        "visible": true
                    }
                ]
            }
        ],
        "outputs": {
            "existingNetworkInterfaceId": "[steps('managedApplicationSetting').networkInterfaceId]",
            "managedIdentity": "[parse(concat('{\"Type\":\"UserAssigned\",\"UserAssignedIdentities\":{',string(steps('managedApplicationSetting').userAssignedId),':{}}}'))]"
        }
    }
}
```

Este archivo CreateUIDefinition.json genera una experiencia de creación de usuario que tiene dos campos. El primer campo permite al usuario escribir en el identificador de recurso de Azure para el recurso que se está vinculando con la implementación de la aplicación administrada. El segundo es para que un consumidor escriba el identificador de recursos de Azure de la **identidad asignada por el usuario**, que tiene acceso al recurso de Azure vinculado. La experiencia generada tendría el aspecto siguiente:

![Ejemplo de CreateUIDefinition con dos entradas: un identificador de recurso de interfaz de red y un identificador de recurso de identidad asignada por el usuario](./media/publish-managed-identity/network-interface-cuid.png)

### <a name="authoring-the-maintemplate-with-a-linked-resource"></a>Creación de mainTemplate con un recurso vinculado

Además de actualizar la instancia de CreateUIDefinition, la plantilla principal también debe actualizarse para aceptar el identificador de recurso vinculado que se ha pasado. La plantilla principal puede actualizarse para aceptar la nueva salida si se agrega un nuevo parámetro. Puesto que la salida `managedIdentity` invalida el valor en la plantilla de la aplicación administrada generada, no se pasa a la plantilla principal y no deben incluirse en la sección de parámetros.

Plantilla principal de ejemplo que establece el perfil de red a una interfaz de red existente proporcionada por CreateUIDefinition.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "existingNetworkInterfaceId": { "type": "string" }
    },
    "variables": {
    },
    "resources": [
        {
            "apiVersion": "2016-04-30-preview",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "myLinkedResourceVM",
            "location": "[resourceGroup().location]",
            "properties": {
                …,
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[parameters('existingNetworkInterfaceId')]"
                        }
                    ]
                }
            }
        }
    ]
}
```

### <a name="consuming-the-managed-application-with-a-linked-resource"></a>Consumir la aplicación administrada con un recurso vinculado

Una vez creado el paquete de la aplicación administrada, esta se puede consumir a través de Azure Portal. Antes de que pueda consumirse, hay varios requisitos previos.

- Debe crearse una instancia del recurso de Azure vinculado necesario.
- La **identidad asignada por el usuario** debe [crearse y recibir las asignaciones de roles](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) al recurso vinculado.
- El identificador del recurso vinculado existente y el identificador de la **identidad asignada por el usuario** se proporcionan a CreateUIDefinition.

## <a name="accessing-the-managed-identity-token"></a>Acceder al token de identidad administrada

Ahora se puede acceder el token de la aplicación administrada mediante la API `listTokens` desde el inquilino del editor. Un ejemplo de solicitud puede tener el siguiente aspecto:

``` HTTP
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Solutions/applications/{applicationName}/listTokens?api-version=2018-09-01-preview HTTP/1.1

{
    "authorizationAudience": "https://management.azure.com/",
    "userAssignedIdentities": [
        "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{userAssignedIdentityName}"
    ]
}
```

Parámetros del cuerpo de la solicitud:

Parámetro | Obligatorio | Descripción
---|---|---
authorizationAudience | *no* | El URI del id. de la aplicación del recurso de destino. También es la notificación `aud` (audiencia) del token emitido. El valor predeterminado es "https://management.azure.com/".
userAssignedIdentities | *no* | La lista de identidades administradas asignadas por el usuario para las cuales recuperar un token. Si no se especifica, `listTokens` devolverá el token para la identidad administrada asignada por el sistema.


Una respuesta de ejemplo puede tener el siguiente aspecto:

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json

{
    "value": [
        {
            "access_token": "eyJ0eXAi…",
            "expires_in": "2…",
            "expires_on": "1557…",
            "not_before": "1557…",
            "authorizationAudience": "https://management.azure.com/",
            "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Solutions/applications/{applicationName}",
            "token_type": "Bearer"
        }
    ]
}
```

La respuesta contendrá una matriz de tokens en la propiedad `value`:

Parámetro | Descripción
---|---
access_token | El token de acceso solicitado.
expires_in | El número de segundos que será válido el token de acceso.
expires_on | La hora a la que expira el token de acceso. Se representa como el número de segundos desde la época.
not_before | El intervalo de tiempo en que el token de acceso surte efecto. Se representa como el número de segundos desde la época.
authorizationAudience | La `aud` (audiencia) para la cual se solicitó el token de acceso. Es igual a la que se proporcionó en la solicitud `listTokens`.
resourceId | El identificador de recursos de Azure para el token emitido. Es el identificador de la aplicación administrada o el identificador de la identidad asignada por el usuario.
token_type | Tipo de token.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cómo configurar una aplicación administrada con un proveedor personalizado](../custom-providers/overview.md)