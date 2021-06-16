---
title: Creación de una cuenta de Azure Video Analyzer
description: En este tema se explica cómo crear una cuenta para Azure Video Analyzer.
ms.service: azure-video-analyzer
ms.topic: how-to
ms.date: 05/01/2021
ms.openlocfilehash: c1c9b7a01fa23b3c5b450fdfa5d5024169b4453e
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112007560"
---
# <a name="create-a-video-analyzer-account"></a>Creación de una cuenta de Video Analyzer

Para empezar a usar Azure Video Analyzer, deberá crear una cuenta de dicho servicio. La cuenta deberá estar asociada a una cuenta de almacenamiento y a una [identidad administrada asignada por el usuario][docs-uami]. La identidad administrada deberá tener los permisos de los roles [Colaborador de datos de Storage Blob][docs-storage-access] y [Lector][docs-role-reader] para la cuenta de almacenamiento. En este artículo se describen los pasos para crear una cuenta de Video Analyzer.

 Puede usar Azure Portal o una [plantilla de Azure Resource Manager (ARM)][docs-arm-template] para crear una cuenta de Video Analyzer. Elija la pestaña para el método que quiere usar.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="portal"></a>[Portal](#tab/portal/)

[!INCLUDE [the video analyzer account and storage account must be in the same subscription and region](./includes/note-account-storage-same-subscription.md)]

### <a name="create-a-video-analyzer-account-in-the-azure-portal"></a>Creación de una cuenta de Video Analyzer en Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Con la barra de búsqueda de la parte superior, escriba **Video Analyzer**.
1. Haga clic en *Instancias de Video Analyzer* en la sección *Servicios*.
1. Haga clic en **Agregar**.
1. En la sección **Creación de una cuenta de Video Analyzer** escriba los valores necesarios.

    | Nombre | Descripción |
    | ---|---|
    |**Suscripción**|Si tiene más de una suscripción, en la lista de suscripciones de Azure seleccione una de entre aquellas a las que tiene acceso.|
    |**Grupo de recursos**|Seleccione un recurso existente, o bien cree uno. Un grupo de recursos es una colección de recursos que comparten ciclos de vida, permisos y directivas. Obtenga más información [aquí](../../azure-resource-manager/management/overview.md#resource-groups).|
    |**Nombre de la cuenta de Video Analyzer**|Escriba el nombre de la nueva cuenta de Video Analyzer. El nombre de la cuenta de Video Analyzer debe estar compuesto totalmente de minúsculas o números, sin espacios y con una longitud de entre 3 y 24 caracteres.|
    |**Ubicación**|Seleccione la región geográfica que se usará para almacenar los registros de vídeo y de metadatos para la cuenta de Video Analyzer. En la lista desplegable solo aparecen las regiones de Video Analyzer disponibles. |
    |**Cuenta de almacenamiento**|Seleccione una cuenta de almacenamiento para proporcionar almacenamiento de blobs para el contenido de vídeo en la cuenta de Video Analyzer. Puede seleccionar una cuenta de almacenamiento existente en la misma región geográfica que la cuenta de Video Analyzer, o bien crear una nueva cuenta de almacenamiento. Se crea una nueva cuenta de almacenamiento en la misma región. Las reglas para los nombres de las cuentas de almacenamiento son las mismas que para las cuentas de Video Analyzer.<br/>|
    |**Identidad del usuario**|Seleccione una identidad administrada asignada por el usuario que la nueva cuenta de Video Analyzer usará para acceder a la cuenta de almacenamiento. Puede seleccionar una identidad administrada asignada por el usuario existente, o bien puede crear una nueva. A la identidad administrada asignada por el usuario se le asignarán los roles [Colaborador de datos de Storage Blob][docs-storage-access] y [Lector][docs-role-reader] para la cuenta de almacenamiento.

1. En la parte inferior del formulario, haga clic en **Revisar y crear**.

## <a name="template"></a>[Plantilla](#tab/template/)

[!INCLUDE [the video analyzer account and storage account must be in the same subscription and region](./includes/note-account-storage-same-subscription.md)]

### <a name="create-a-video-analyzer-account-using-a-template"></a>Creación de una cuenta de Video Analyzer mediante una plantilla

Los recursos siguientes se definen en la plantilla:

- [**Microsoft.Media/videoAnalyzers**](/azure/templates/Microsoft.Media/videoAnalyzers): el recurso de cuenta de Video Analyzer.
- [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts): la cuenta de almacenamiento que usará Video Analyzer para almacenar vídeos y metadatos.
- [**Microsoft.ManagedIdentity/userAssignedIdentities**](/azure/templates/Microsoft.ManagedIdentity/userAssignedIdentities): la identidad administrada asignada por el usuario que Video Analyzer usará para acceder al almacenamiento.
- [**Microsoft.Storage/storageAccounts/providers/roleAssignments**](/azure/templates/microsoft.authorization/roleassignments): la asignación de roles que permite a Video Analyzer acceder a la cuenta de almacenamiento.

<!-- TODO replace with a reference like this:
:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.compute/vm-simple-linux/azuredeploy.json":::
-->

```json

{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "namePrefix": {
            "metadata": {
                "description": "Used to qualify the names of all of the resources created in this template."
            },
            "defaultValue": "avasample",
            "type": "string",
            "minLength": 3,
            "maxLength": 13
        }
    },
    "variables": {
        "storageAccountName": "[concat(parameters('namePrefix'),uniqueString(resourceGroup().id))]",
        "accountName": "[concat(parameters('namePrefix'),uniqueString(resourceGroup().id))]",
        "managedIdentityName": "[concat(parameters('namePrefix'),'-',resourceGroup().name,'-storage-access-identity')]"
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2020-10-01",
            "name": "deploy-storage-and-identity",
            "properties": {
                "mode": "Incremental",
                "expressionEvaluationOptions": {
                    "scope": "Inner"
                },
                "parameters": {
                    "namePrefix": {
                        "value": "[parameters('namePrefix')]"
                    },
                    "managedIdentityName": {
                        "value": "[variables('managedIdentityName')]"
                    }
                },
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {
                        "namePrefix": {
                            "type": "string"
                        },
                        "managedIdentityName": {
                            "type": "string"
                        }
                    },
                    "variables": {
                        "storageAccountName": "[concat(parameters('namePrefix'),uniqueString(resourceGroup().id))]",
                        "managedIdentityName": "[parameters('managedIdentityName')]",
                        "storageBlobDataContributorAssignment": "[guid('Storage Blob Data Contributor',variables('managedIdentityName'))]",
                        "storageBlobDataContributorDefinitionId": "[concat(resourceGroup().id, '/providers/Microsoft.Authorization/roleDefinitions/', 'ba92f5b4-2d11-453d-a403-e96b0029c9fe')]",
                        "readerAssignment": "[guid('Reader',variables('managedIdentityName'))]",
                        "readerDefinitionId": "[concat(resourceGroup().id, '/providers/Microsoft.Authorization/roleDefinitions/', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')]"
                    },
                    "resources": [
                        {
                            "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
                            "name": "[variables('managedIdentityName')]",
                            "apiVersion": "2015-08-31-preview",
                            "location": "[resourceGroup().location]"
                        },
                        {
                            "type": "Microsoft.Storage/storageAccounts",
                            "apiVersion": "2019-04-01",
                            "name": "[variables('storageAccountName')]",
                            "location": "[resourceGroup().location]",
                            "sku": {
                                "name": "Standard_LRS"
                            },
                            "kind": "StorageV2",
                            "properties": {
                                "accessTier": "Hot"
                            }
                        },
                        {
                            "name": "[concat(variables('storageAccountName'), '/Microsoft.Authorization/', variables('storageBlobDataContributorAssignment'))]",
                            "type": "Microsoft.Storage/storageAccounts/providers/roleAssignments",
                            "apiVersion": "2021-04-01-preview",
                            "dependsOn": [
                                "[variables('managedIdentityName')]",
                                "[variables('storageAccountName')]"
                            ],
                            "properties": {
                                "roleDefinitionId": "[variables('storageBlobDataContributorDefinitionId')]",
                                "principalId": "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities',variables('managedIdentityName')), '2018-11-30').principalId]",
                                "principalType": "ServicePrincipal"
                            }
                        },
                        {
                            "name": "[concat(variables('storageAccountName'), '/Microsoft.Authorization/', variables('readerAssignment'))]",
                            "type": "Microsoft.Storage/storageAccounts/providers/roleAssignments",
                            "apiVersion": "2021-04-01-preview",
                            "dependsOn": [
                                "[variables('managedIdentityName')]",
                                "[variables('storageAccountName')]"
                            ],
                            "properties": {
                                "roleDefinitionId": "[variables('readerDefinitionId')]",
                                "principalId": "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities',variables('managedIdentityName')), '2018-11-30').principalId]",
                                "principalType": "ServicePrincipal"
                            }
                        }
                    ],
                    "outputs": {}
                }
            }
        },
        {
            "type": "Microsoft.Media/videoAnalyzers",
            "comments": "The Azure Video Analyzer account",
            "apiVersion": "2021-05-01-preview",
            "name": "[variables('accountName')]",
            "location": "[resourceGroup().location]",
            "dependsOn": [
                "deploy-storage-and-identity"
            ],
            "properties": {
                "storageAccounts": [
                    {
                        "id": "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
                        "identity": {
                            "userAssignedIdentity": "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities',variables('managedIdentityName'))]"
                        }
                    }
                ]
            },
            "identity": {
                "type": "UserAssigned",
                "userAssignedIdentities": {
                    "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities',variables('managedIdentityName'))]": {}
                }
            }
        }
    ],
    "outputs": { }
}
```

> [!NOTE]
> La plantilla usa una implementación anidada para la asignación de roles a fin de garantizar que está disponible antes de implementar el recurso de cuenta de Video Analyzer.

### <a name="deploy-the-template"></a>Implementación de la plantilla

1. Para iniciar sesión en Azure y abrir una plantilla, haga clic en el botón *Implementar en Azure*.

    [![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)

1. Seleccione o escriba los siguientes valores. Utilice los valores predeterminados, si están disponibles.

    - **Suscripción**: seleccione una suscripción de Azure.
    - **Grupo de recursos**: seleccione un grupo de recursos existente del menú desplegable, o bien seleccione **Crear nuevo**, escriba un nombre único para el grupo de recursos y, a continuación, haga clic en **Aceptar**.
    - **Ubicación**: seleccione una ubicación.  Por ejemplo, **Oeste de EE. UU. 2**.
    - **Prefijo de nombre**: especifique una cadena que se usará como prefijo para el nombre de los recursos (se recomiendan los valores predeterminados).

1. Seleccione **Revisar + crear**. Una vez finalizada la validación, seleccione **Crear** para crear e implementar la VM.

Azure Portal se usa para implementar la plantilla. Además de Azure Portal, también puede usar la CLI de Azure, Azure PowerShell y la API de REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../../azure-resource-manager/templates/deploy-cli.md).

### <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Puede usar Azure Portal para comprobar la cuenta y los otros recursos que se crearon. Una vez finalizada la implementación, seleccione **Ir al grupo de recursos** para ver la cuenta y los otros recursos.

### <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el grupo de recursos, lo que eliminará la cuenta y todos los recursos del grupo de recursos.

1. Seleccione el **Grupo de recursos**.
1. En la página del grupo de recursos, seleccione **Eliminar**.
1. Cuando se le solicite, escriba el nombre del grupo de recursos y, después, seleccione **Eliminar**.

---

### <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre [cómo implementar Video Analyzer en un dispositivo IoT Edge][docs-deploy-on-edge].

<!-- links -->
[docs-uami]: ../../active-directory/managed-identities-azure-resources/overview.md
[docs-storage-access]: ../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor
[docs-role-reader]: ../../role-based-access-control/built-in-roles.md#reader
[docs-arm-template]: ../../azure-resource-manager/templates/overview.md
[docs-deploy-on-edge]: deploy-iot-edge-device.md
[click-to-deploy]: https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fgist.githubusercontent.com%2Fbennage%2F58523b2e6a4d3bf213f16893d894dcaf%2Fraw%2Fazuredeploy.json
<!-- TODO update the link above! -->