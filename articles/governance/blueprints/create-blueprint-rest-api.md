---
title: 'Inicio rápido: Creación de un plano técnico con la API de REST'
description: En este inicio rápido se usa Azure Blueprints para crear, definir e implementar artefactos mediante la API REST.
ms.date: 05/01/2021
ms.topic: quickstart
ms.openlocfilehash: 37f67e6db753969344b8c442de78b867dd41ecd6
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108323832"
---
# <a name="quickstart-define-and-assign-an-azure-blueprint-with-rest-api"></a>Inicio rápido: Definición y asignación de un plano técnico de Azure Blueprint con API REST

Aprender a crear y asignar planos técnicos permite definir patrones comunes para desarrollar configuraciones reutilizables y de implementación rápida basadas en plantillas de Azure Resource Manager, directivas, seguridad, etc. En este tutorial, aprenderá a usar planos técnicos de Azure Blueprint para realizar algunas de las tareas más comunes relacionadas con la creación, asignación y administración de directivas en toda la organización, como, por ejemplo:

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.
- Registre el proveedor de recursos `Microsoft.Blueprint`. Para instrucciones, consulte [Tipos y proveedores de recursos](../../azure-resource-manager/management/resource-providers-and-types.md).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="getting-started-with-rest-api"></a>Introducción a la API REST

Si no está familiarizado con la API REST, comience por revisar la [Referencia de la API REST de Azure](/rest/api/azure/) para obtener una descripción general de la API REST, específicamente el identificador URI de la solicitud y el cuerpo de la solicitud. En este artículo se utilizan estos conceptos para proporcionar instrucciones para trabajar con planos técnicos de Azure Blueprint y da por supuesto un conocimiento práctico de ellos. Herramientas como [ARMClient](https://github.com/projectkudu/ARMClient) y otras pueden controlar la autorización automáticamente y se recomiendan para principiantes.

Para información sobre las especificaciones de Azure Blueprints, consulte [API REST de Azure Blueprints](/rest/api/blueprints/).

### <a name="rest-api-and-powershell"></a>API REST y PowerShell

Si aún no tiene una herramienta para realizar llamadas de API REST, considere el uso de PowerShell para estas instrucciones. A continuación se muestra un encabezado de ejemplo para autenticar con Azure. Genere un encabezado de autenticación, a veces llamado **Token de portador**, y proporcione el identificador URI de la API REST al que conectarse con cualquier parámetro o un **cuerpo de la solicitud**:

```azurepowershell-interactive
# Log in first with Connect-AzAccount if not using Cloud Shell

$azContext = Get-AzContext
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azProfile)
$token = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)
$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $token.AccessToken
}

# Invoke the REST API
$restUri = 'https://management.azure.com/subscriptions/{subscriptionId}?api-version=2020-01-01'
$response = Invoke-RestMethod -Uri $restUri -Method Get -Headers $authHeader
```

Reemplace `{subscriptionId}` de la variable **$restUri** anterior para obtener información sobre la suscripción. La variable $response contiene el resultado del cmdlet `Invoke-RestMethod`, que puede analizarse con cmdlets como [ConvertFrom-Json](/powershell/module/microsoft.powershell.utility/convertfrom-json). Si el punto de conexión de servicio de la API REST espera un **cuerpo de la solicitud**, proporcione una variable con formato JSON al parámetro `-Body` de `Invoke-RestMethod`.

## <a name="create-a-blueprint"></a>Creación de un plano técnico

El primer paso para definir un patrón estándar de cumplimiento es elaborar un plano técnico a partir de los recursos disponibles. Vamos a crear un plano técnico llamado "MyBlueprint" para configurar las asignaciones de roles y de directivas de la suscripción. A continuación, vamos a agregar un grupo de recursos, una plantilla de Resource Manager y una asignación de roles en el grupo de recursos.

> [!NOTE]
> Cuando se usa la API REST, el objeto _blueprint_ se crea en primer lugar. Para cada  _artefacto_ que se agregue y que tenga parámetros, estos deben definirse de antemano en el _plano técnico_ inicial.

En cada identificador URI de la API REST, hay variables usadas que se deben reemplazar por sus propios valores:

- `{YourMG}`: reemplácelo por el identificador del grupo de administración
- `{subscriptionId}`: reemplácelo por el identificador de suscripción

> [!NOTE]
> Los planos técnicos también pueden crearse en el nivel de suscripción. Para ver un ejemplo, consulte el artículo de [creación de un plano técnico en el ejemplo de suscripción](/rest/api/blueprints/blueprints/createorupdate#subscriptionblueprint).

1. Cree el objeto _blueprint_ inicial. El **cuerpo de la solicitud** incluye propiedades sobre el plano técnico, cualquier grupo de recursos que se vaya a crear y todos los parámetros en el nivel del plano técnico. Los parámetros se establecen durante la asignación y son utilizados por los artefactos que se agregan en pasos posteriores.

   - URI DE LA API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint?api-version=2018-11-01-preview
     ```

   - Cuerpo de la solicitud

     ```json
     {
         "properties": {
             "description": "This blueprint sets tag policy and role assignment on the subscription, creates a ResourceGroup, and deploys a resource template and role assignment to that ResourceGroup.",
             "targetScope": "subscription",
             "parameters": {
                 "storageAccountType": {
                     "type": "string",
                     "metadata": {
                         "displayName": "storage account type.",
                         "description": null
                     }
                 },
                 "tagName": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The name of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "tagValue": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The value of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "contributors": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Contributor role at the subscription"
                     }
                 },
                 "owners": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Owner role at the resource group"
                     }
                 }
             },
             "resourceGroups": {
                 "storageRG": {
                     "description": "Contains the resource template deployment and a role assignment."
                 }
             }
         }
     }
     ```

1. Agregue una asignación de roles a la suscripción. El **cuerpo de la solicitud** define el _tipo_ de artefacto, las propiedades se alinean con el identificador de definición de rol y las identidades de la entidad de servicio se pasan como una matriz de valores. En el siguiente ejemplo, las identidades de la entidad de servicio a las que se ha asignado el rol especificado se configuran con un parámetro que se establece durante la asignación de planos técnicos. Este ejemplo se usa el rol integrado _Colaborador_ con el GUID `b24988ac-6180-42a0-ab88-20f7382dd24c`.

   - URI DE LA API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/roleContributor?api-version=2018-11-01-preview
     ```

   - Cuerpo de la solicitud

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
             "principalIds": "[parameters('contributors')]"
         }
     }
     ```

1. Agregue la asignación de directivas a la suscripción. El **cuerpo de la solicitud** define el _tipo_ de artefacto, las propiedades que se alinean con una definición de directiva o iniciativa y configura la asignación de directivas para que utilice los parámetros definidos del plano técnico que se van a configurar durante la asignación del plano técnico. En este ejemplo se usa la directiva integrada _Aplicar una etiqueta y su valor predeterminado a los grupos de recursos_ con el GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - URI DE LA API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/policyTags?api-version=2018-11-01-preview
     ```

   - Cuerpo de la solicitud

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "description": "Apply tag and its default value to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "[parameters('tagName')]"
                 },
                 "tagValue": {
                     "value": "[parameters('tagValue')]"
                 }
             }
         }
     }
     ```

1. Agregue otra asignación de directiva para la etiqueta Storage (reutilizando el parámetro _storageAccountType_) en la suscripción. Este artefacto de asignación de directivas adicional muestra que un parámetro definido en el plano técnico lo puede utilizar más de un artefacto. En el ejemplo, **storageAccountType** se usa para establecer una etiqueta en el grupo de recursos. Este valor proporciona información acerca de la cuenta de almacenamiento que se crea en el paso siguiente. En este ejemplo se usa la directiva integrada _Aplicar una etiqueta y su valor predeterminado a los grupos de recursos_ con el GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - URI DE LA API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/policyStorageTags?api-version=2018-11-01-preview
     ```

   - Cuerpo de la solicitud

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "description": "Apply storage tag and the parameter also used by the template to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "StorageType"
                 },
                 "tagValue": {
                     "value": "[parameters('storageAccountType')]"
                 }
             }
         }
     }
     ```

1. Agregue la plantilla en el grupo de recursos. El **cuerpo de la solicitud** de una plantilla de Resource Manager incluye el componente JSON normal de la plantilla y define el grupo de recursos de destino con **properties.resourceGroup**. La plantilla también reutiliza los parámetros del plano técnico **storageAccountType**, **tagName** y **tagValue**, cada uno de los cuales se pasa a la plantilla. Los parámetros del plano técnico se ponen a disposición de la plantilla mediante la definición de **properties.parameters** y dentro de la plantilla JSON el par clave/valor se utiliza para insertar el valor. Los nombres de los parámetros del plano técnico y de la plantilla podrían ser los mismos, pero se han hecho diferentes para ilustrar cómo se pasa cada uno de ellos del plano técnico al artefacto de la plantilla.

   - URI DE LA API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/templateStorage?api-version=2018-11-01-preview
     ```

   - Cuerpo de la solicitud

     ```json
     {
         "kind": "template",
         "properties": {
             "template": {
                 "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                 "contentVersion": "1.0.0.0",
                 "parameters": {
                     "storageAccountTypeFromBP": {
                         "type": "string",
                         "defaultValue": "Standard_LRS",
                         "allowedValues": [
                             "Standard_LRS",
                             "Standard_GRS",
                             "Standard_ZRS",
                             "Premium_LRS"
                         ],
                         "metadata": {
                             "description": "Storage Account type"
                         }
                     },
                     "tagNameFromBP": {
                         "type": "string",
                         "defaultValue": "NotSet",
                         "metadata": {
                             "description": "Tag name from blueprint"
                         }
                     },
                     "tagValueFromBP": {
                         "type": "string",
                         "defaultValue": "NotSet",
                         "metadata": {
                             "description": "Tag value from blueprint"
                         }
                     }
                 },
                 "variables": {
                     "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
                 },
                 "resources": [{
                     "type": "Microsoft.Storage/storageAccounts",
                     "name": "[variables('storageAccountName')]",
                     "apiVersion": "2016-01-01",
                     "tags": {
                        "[parameters('tagNameFromBP')]": "[parameters('tagValueFromBP')]"
                     },
                     "location": "[resourceGroups('storageRG').location]",
                     "sku": {
                         "name": "[parameters('storageAccountTypeFromBP')]"
                     },
                     "kind": "Storage",
                     "properties": {}
                 }],
                 "outputs": {
                     "storageAccountSku": {
                         "type": "string",
                         "value": "[variables('storageAccountName')]"
                     }
                 }
             },
             "resourceGroup": "storageRG",
             "parameters": {
                 "storageAccountTypeFromBP": {
                     "value": "[parameters('storageAccountType')]"
                 },
                 "tagNameFromBP": {
                     "value": "[parameters('tagName')]"
                 },
                 "tagValueFromBP": {
                     "value": "[parameters('tagValue')]"
                 }
             }
         }
     }
     ```

1. Agregue la asignación de roles en el grupo de recursos. Similar a la entrada anterior de asignación de roles, el ejemplo siguiente utiliza el identificador de definición para el rol **Propietario** y le proporciona un parámetro diferente del plano técnico. Este ejemplo se usa el rol integrado _Propietario_ con el GUID `8e3af657-a8ff-443c-a75c-2fe8c4bcb635`.

   - URI DE LA API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/roleOwner?api-version=2018-11-01-preview
     ```

   - Cuerpo de la solicitud

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "resourceGroup": "storageRG",
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
             "principalIds": "[parameters('owners')]"
         }
     }
     ```

## <a name="publish-a-blueprint"></a>Publicación de un plano técnico

Ahora que los artefactos se han agregado al plano técnico, es momento de publicarlo. La publicación lo hace disponible para ser asignado a una suscripción.

- URI DE LA API REST

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/versions/{BlueprintVersion}?api-version=2018-11-01-preview
  ```

El valor de `{BlueprintVersion}` es una cadena de letras, números y guiones (sin espacios ni otros caracteres especiales) con una longitud máxima de 20 caracteres. Utilice un nombre único e informativo, como **v20180622-135541**.

## <a name="assign-a-blueprint"></a>Asignación de un plano técnico

Cuando se ha publicado un plano técnico mediante la API REST, se puede asignar a una suscripción. Asigne el plano técnico creado a una de las suscripciones de la jerarquía del grupo de administración. Si el proyecto se guarda en una suscripción, solo se puede asignar a dicha suscripción. El **cuerpo de la solicitud** especifica el plano técnico que se va a asignar, proporciona el nombre y la ubicación de cualquier grupo de recursos de la definición del plano técnico y proporciona todos los parámetros que se han definido en el plano técnico y que han sido utilizados por uno o más artefactos asociados.

En cada identificador URI de la API REST, hay variables usadas que se deben reemplazar por sus propios valores:

- `{tenantId}`: reemplácelo por su identificador de inquilino
- `{YourMG}`: reemplácelo por el identificador del grupo de administración
- `{subscriptionId}`: reemplácelo por el identificador de suscripción

1. Proporcione a la entidad de servicio de Azure Blueprint el rol de **propietario** en la suscripción de destino. El valor de AppId es estático (`f71766dc-90d9-4b7d-bd9d-4499c4331c3f`), pero el identificador de la entidad de servicio varía según el inquilino. Los detalles se pueden solicitar para su inquilino mediante la API REST siguiente. Utiliza [Graph API de Azure Active Directory](/graph/migrate-azure-ad-graph-planning-checklist), que tiene una autorización diferente.

   - URI DE LA API REST

     ```http
     GET https://graph.windows.net/{tenantId}/servicePrincipals?api-version=1.6&$filter=appId eq 'f71766dc-90d9-4b7d-bd9d-4499c4331c3f'
     ```

1. Ejecute la implementación del plano técnico asignándolo a una suscripción. Como los parámetros **contributors** y **owners** requieren una serie de objectIds de las entidades de servicio para que se les conceda la asignación de roles, utilice [Graph API de Azure Active Directory](/graph/migrate-azure-ad-graph-planning-checklist) para recopilar los elementos objectId que se utilizarán en el **cuerpo de la solicitud** para sus propios usuarios, grupos o entidades de servicio.

   - URI DE LA API REST

     ```http
     PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Blueprint/blueprintAssignments/assignMyBlueprint?api-version=2018-11-01-preview
     ```

   - Cuerpo de la solicitud

     ```json
     {
         "properties": {
             "blueprintId": "/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint",
             "resourceGroups": {
                 "storageRG": {
                     "name": "StorageAccount",
                     "location": "eastus2"
                 }
             },
             "parameters": {
                 "storageAccountType": {
                     "value": "Standard_GRS"
                 },
                 "tagName": {
                     "value": "CostCenter"
                 },
                 "tagValue": {
                     "value": "ContosoIT"
                 },
                 "contributors": {
                     "value": [
                         "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
                         "38833b56-194d-420b-90ce-cff578296714"
                     ]
                 },
                 "owners": {
                     "value": [
                         "44254d2b-a0c7-405f-959c-f829ee31c2e7",
                         "316deb5f-7187-4512-9dd4-21e7798b0ef9"
                     ]
                 }
             }
         },
         "identity": {
             "type": "systemAssigned"
         },
         "location": "westus"
     }
     ```

   - Identidad administrada asignada por el usuario

     Una asignación de plano técnico también puede usar una [identidad administrada asignada por el usuario](../../active-directory/managed-identities-azure-resources/overview.md).
     En este caso, la parte de **identidad** del cuerpo de la solicitud cambia del modo siguiente. Reemplace `{yourRG}` y `{userIdentity}` por el nombre del grupo de recursos y el nombre de su identidad administrada asignada por el usuario, respectivamente.

     ```json
     "identity": {
         "type": "userAssigned",
         "tenantId": "{tenantId}",
         "userAssignedIdentities": {
             "/subscriptions/{subscriptionId}/resourceGroups/{yourRG}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{userIdentity}": {}
         }
     },
     ```

     La **identidad administrada asignada por el usuario** puede estar en cualquier suscripción y grupo de recursos para los que tenga permiso el usuario que asigna el plano técnico.

     > [!IMPORTANT]
     > Azure Blueprints no administra la identidad administrada asignada por el usuario. Los usuarios son responsables de asignar los roles y permisos necesarios o, de lo contrario, se producirá un error en la asignación del plano técnico.

## <a name="clean-up-resources"></a>Limpieza de recursos

### <a name="unassign-a-blueprint"></a>Cancelación de la asignación de un plano técnico

Puede eliminar un plano de una suscripción. A menudo, la eliminación se realiza cuando ya no son necesarios los recursos de artefacto. Cuando se quita un plano técnico, se omiten los artefactos que se asignaron como parte de ese plano técnico. Para quitar una asignación del plano técnico, use la siguiente operación de la API REST:

- URI DE LA API REST

  ```http
  DELETE https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Blueprint/blueprintAssignments/assignMyBlueprint?api-version=2018-11-01-preview
  ```

### <a name="delete-a-blueprint"></a>Eliminación de un plano técnico

Para quitar el mismo plano técnico, utilice la siguiente operación de la API REST:

- URI DE LA API REST

  ```http
  DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint?api-version=2018-11-01-preview
  ```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado, asignado y eliminado un plano técnico con la API REST. Para más información sobre Azure Blueprints, vaya el artículo sobre el ciclo de vida de los planos técnicos.

> [!div class="nextstepaction"]
> [Más información acerca del ciclo de vida del plano técnico](./concepts/lifecycle.md)