---
title: Bloqueo de recursos para impedir cambios
description: Impida que los usuarios actualicen o eliminen recursos de Azure aplicando un bloqueo para todos los usuarios y roles.
ms.topic: conceptual
ms.date: 05/07/2021
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 5d8af2529039aa6e9435243249d7724d996b119d
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109634804"
---
# <a name="lock-resources-to-prevent-unexpected-changes"></a>Bloqueo de recursos para impedir cambios inesperados

Como administrador, puede bloquear una suscripción, un grupo de recursos o un recurso para impedir que otros usuarios de su organización eliminen o modifiquen accidentalmente recursos críticos. El bloqueo invalida los permisos que el usuario pueda tener.

Puede establecer el bloqueo de nivel en **CanNotDelete** o **ReadOnly**. En el portal, los bloqueos se denominan **Delete** y **Read-only** respectivamente.

- **CanNotDelete** significa que los usuarios autorizados pueden leer y modificar recursos, pero no eliminarlos.
- **ReadOnly** significa que los usuarios autorizados solo pueden leer recursos, pero no actualizarlos ni eliminarlos. Aplicar este bloqueo es similar a restringir todos los usuarios autorizados a los permisos concedidos por el rol **Lector**.

Al diferencia del control de acceso basado en rol, los bloqueos de administración se usan para aplicar una restricción a todos los usuarios y roles. Para información sobre cómo establecer permisos para usuarios y roles, consulte [Control de acceso basado en rol de Azure (RBAC de Azure)](../../role-based-access-control/role-assignments-portal.md).

## <a name="lock-inheritance"></a>Bloqueo de la herencia

Cuando se aplica un bloqueo en un ámbito primario, todos los recursos heredan el mismo bloqueo. Incluso los recursos que agregue posteriormente heredan el bloqueo del elemento primario. El bloqueo más restrictivo de toda la herencia tiene prioridad.

## <a name="understand-scope-of-locks"></a>Descripción del ámbito de los bloqueos

> [!NOTE]
> Es importante comprender que los bloqueos no se aplican a todos los tipos de operaciones. Las operaciones de Azure se pueden dividir en dos categorías: plano de control y plano de datos. **Los bloqueos solo se aplican a las operaciones del plano de control**.

Las operaciones del plano de control son operaciones que se envían a `https://management.azure.com`. Las operaciones del plano de datos son operaciones que se envían a la instancia de un servicio, como `https://myaccount.blob.core.windows.net/`. Para más información, consulte [Plano de control y plano de datos de Azure](control-plane-and-data-plane.md). Para saber qué operaciones usan la dirección URL del plano de control, consulte [API REST de Azure](/rest/api/azure/).

Esta distinción significa que los bloqueos impiden cambios en un recurso, pero no restringen el modo en que los recursos cumplen sus propias funciones.  Por ejemplo, un bloqueo ReadOnly en un servidor lógico de SQL Database impide que elimine o modifique el servidor. Sin embargo, no le impide crear, actualizar o eliminar datos de la base de datos en ese servidor. Se permiten las transacciones de datos porque esas operaciones no se envían a `https://management.azure.com`.

En la sección siguiente se describen más ejemplos de las diferencias entre las operaciones del plano de control y del plano de datos.

## <a name="considerations-before-applying-locks"></a>Consideraciones antes de aplicar bloqueos

Aplicar bloqueos puede provocar resultados inesperados, ya que algunas operaciones que no parecen modificar el recurso realmente requieren acciones que el bloqueo ha bloqueado. Los bloqueos impedirán todas las operaciones que requieran una solicitud POST a la API de Azure Resource Manager. Algunos ejemplos comunes de las operaciones los bloqueos bloquean son:

- Un bloqueo de solo lectura en una **cuenta de almacenamiento** impide que todos los usuarios muestren las claves de cuenta. La operación [Crear lista de claves](/rest/api/storagerp/storageaccounts/listkeys) de Azure Storage se controla a través de una solicitud POST para proteger el acceso a las claves de cuenta, que proporcionan acceso completo a los datos de la cuenta de almacenamiento. Cuando se configura un bloqueo de solo lectura para una cuenta de almacenamiento, los usuarios que no poseen las claves de cuenta deben usar las credenciales de Azure AD para acceder a los datos de blobs o colas. Un bloqueo de solo lectura también impide la asignación de roles de RBAC de Azure del ámbito a la cuenta de almacenamiento o a un contenedor de datos (contenedor de blobs o colas).

- Un bloqueo de "no se puede eliminar" en una **cuenta de almacenamiento** no impide que se eliminen o modifiquen datos dentro de esa cuenta. Este tipo de bloqueo solo evita que se elimine la cuenta de almacenamiento, pero no protege los datos de blobs, colas, tablas o archivos que se encuentran dentro de esa cuenta.

- Un bloqueo de solo lectura en una **cuenta de almacenamiento** no impide que se eliminen o modifiquen datos dentro de esa cuenta. Este tipo de bloqueo solo evita que se elimine o modifique la cuenta de almacenamiento, pero no protege los datos de blobs, colas, tablas o archivos que se encuentran dentro de esa cuenta.

- Un bloqueo de solo lectura en un recurso de **App Service** evita que el Explorador de servidores de Visual Studio muestre los archivos del recurso, ya que esa interacción requiere acceso de escritura.

- Un bloqueo de solo lectura en un **grupo de recursos** que contiene un **plan de App Service** le impide [escalar horizontal o veritcalmente el plan](../../app-service/manage-scale-up.md).

- Un bloqueo de solo lectura en un **grupo de recursos** que contiene una **máquina virtual impide** que todos los usuarios inicien o reinicien la máquina virtual. Estas operaciones requieren una solicitud POST.

- Un bloqueo que no se puede eliminar en un **grupo de recursos** impide que Azure Resource Manager [elimine de forma automática las implementaciones](../templates/deployment-history-deletions.md) en el historial. Si alcanza 800 implementaciones en el historial, se producirá un error en las implementaciones.

- Un bloqueo de no se puede eliminar en el **grupo de recursos** creado por el **Servicio Azure Backup** genera un error en las copias de seguridad. El servicio admite un máximo de 18 puntos de restauración. Cuando está bloqueado, el servicio de copia de seguridad no puede limpiar los puntos de restauración. Para más información, consulte [Preguntas más frecuentes sobre la copia de seguridad de máquinas virtuales de Azure](../../backup/backup-azure-vm-backup-faq.yml).

- Un bloqueo de "no se puede eliminar" en un **grupo de recursos** impide que **Azure Machine Learning** escale automáticamente los [clústeres de proceso de Azure Machine Learning](../../machine-learning/concept-compute-target.md#azure-machine-learning-compute-managed) para quitar nodos no usados.

- Un bloqueo de solo lectura en una **suscripción** impide que **Azure Advisor** funcione correctamente. Advisor no puede almacenar los resultados de sus consultas.

## <a name="who-can-create-or-delete-locks"></a>Quién puede crear o eliminar bloqueos

Para crear o eliminar bloqueos de administración, debe tener acceso a las acciones `Microsoft.Authorization/*` o `Microsoft.Authorization/locks/*`. Entre los roles integrados, solamente se conceden esas acciones al **propietario** y al **administrador de acceso de usuarios**.

## <a name="managed-applications-and-locks"></a>Bloqueos y aplicaciones administradas

Algunos servicios de Azure, como Azure Databricks, utilizan [aplicaciones administradas](../managed-applications/overview.md) para implementar el servicio. En ese caso, el servicio crea dos grupos de recursos. Un grupo de recursos contiene información general sobre el servicio y no está bloqueado. El otro grupo de recursos contiene la infraestructura para el servicio y está bloqueado.

Si intenta eliminar el grupo de recursos de infraestructura, obtendrá un error que indica que el grupo de recursos está bloqueado. Si intenta eliminar el bloqueo para el grupo de recursos de infraestructura, obtendrá un error que indica que el bloqueo no puede eliminarse porque pertenece a una aplicación del sistema.

En su lugar, elimine el servicio, lo que también elimina el grupo de recursos de infraestructura.

Para las aplicaciones administradas, seleccione el servicio implementado.

![Seleccionar servicio](./media/lock-resources/select-service.png)

Tenga en cuenta que el servicio incluye un vínculo para un **Grupo de recursos administrado**. Ese grupo de recursos contiene la infraestructura y está bloqueado. No se puede eliminar directamente.

![Mostrar grupo administrado](./media/lock-resources/show-managed-group.png)

Para eliminar todo el contenido para el servicio, incluido el grupo de recursos de infraestructura bloqueado, seleccione **Eliminar** para el servicio.

![Eliminar servicio](./media/lock-resources/delete-service.png)

## <a name="configure-locks"></a>Configuración de bloqueos

### <a name="portal"></a>Portal

[!INCLUDE [resource-manager-lock-resources](../../../includes/resource-manager-lock-resources.md)]

### <a name="arm-template"></a>Plantilla ARM

Al usar una plantilla de Azure Resource Manager (plantilla de ARM) para implementar un bloqueo, debe tener en cuenta el ámbito del bloqueo y el ámbito de la implementación. Para aplicar un bloqueo en el ámbito de implementación, como el bloqueo de un grupo de recursos o una suscripción, no configure la propiedad de ámbito. Al bloquear un recurso dentro del ámbito de implementación, configure la propiedad de ámbito.

La plantilla siguiente aplica un bloqueo al grupo de recursos en el que está implementada. Observe que no hay una propiedad de ámbito en el recurso de bloqueo porque el ámbito del bloqueo coincide con el ámbito de implementación. Esta plantilla se implementa en el nivel de grupo de recursos.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
  },
  "resources": [
    {
      "type": "Microsoft.Authorization/locks",
      "apiVersion": "2016-09-01",
      "name": "rgLock",
      "properties": {
        "level": "CanNotDelete",
        "notes": "Resource group should not be deleted."
      }
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
resource createRgLock 'Microsoft.Authorization/locks@2016-09-01' = {
  name: 'rgLock'
  properties: {
    level: 'CanNotDelete'
    notes: 'Resource group should not be deleted.'
  }
}
```

---

Para crear un grupo de recursos y bloquearlo, implemente la plantilla siguiente en el nivel de suscripción.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgName": {
      "type": "string"
    },
    "rgLocation": {
      "type": "string"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "name": "[parameters('rgName')]",
      "location": "[parameters('rgLocation')]",
      "properties": {}
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-10-01",
      "name": "lockDeployment",
      "resourceGroup": "[parameters('rgName')]",
      "dependsOn": [
        "[resourceId('Microsoft.Resources/resourceGroups/', parameters('rgName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
            {
              "type": "Microsoft.Authorization/locks",
              "apiVersion": "2016-09-01",
              "name": "rgLock",
              "properties": {
                "level": "CanNotDelete",
                "notes": "Resource group and its resources should not be deleted."
              }
            }
          ],
          "outputs": {}
        }
      }
    }
  ],
  "outputs": {}
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

El archivo Bicep principal crea un grupo de recursos y usa un [módulo](../templates/bicep-modules.md) para crear el bloqueo.

```Bicep
targetScope = 'subscription'

param rgName string
param rgLocation string

resource createRg 'Microsoft.Resources/resourceGroups@2020-10-01' = {
  name: rgName
  location: rgLocation
}

module deployRgLock './lockRg.bicep' = {
  name: 'lockDeployment'
  scope: resourceGroup(createRg.name)
}
```

El módulo usa un archivo Bicep denominado _lockRg.bicep_ que agrega el bloqueo del grupo de recursos.

```bicep
resource createRgLock 'Microsoft.Authorization/locks@2016-09-01' = {
  name: 'rgLock'
  properties: {
    level: 'CanNotDelete'
    notes: 'Resource group and its resources should not be deleted.'
  }
}
```

---

Al aplicar un bloqueo a un **recurso** dentro del grupo de recursos, agregue la propiedad de ámbito. Configure el ámbito en el nombre del recurso que va a bloquear.

En el ejemplo siguiente se muestra una plantilla que crea un plan de App Service, un sitio web y un bloqueo en el sitio web. El ámbito del bloqueo se establece en el sitio web.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "hostingPlanName": {
      "type": "string"
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {
    "siteName": "[concat('ExampleSite', uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2020-12-01",
      "name": "[parameters('hostingPlanName')]",
      "location": "[parameters('location')]",
      "sku": {
        "tier": "Free",
        "name": "f1",
        "capacity": 0
      },
      "properties": {
        "targetWorkerCount": 1
      }
    },
    {
      "type": "Microsoft.Web/sites",
      "apiVersion": "2020-12-01",
      "name": "[variables('siteName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', parameters('hostingPlanName'))]"
      ],
      "properties": {
        "serverFarmId": "[parameters('hostingPlanName')]"
      }
    },
    {
      "type": "Microsoft.Authorization/locks",
      "apiVersion": "2016-09-01",
      "name": "siteLock",
      "scope": "[concat('Microsoft.Web/sites/', variables('siteName'))]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', variables('siteName'))]"
      ],
      "properties": {
        "level": "CanNotDelete",
        "notes": "Site should not be deleted."
      }
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```Bicep
param hostingPlanName string
param location string = resourceGroup().location

var siteName = concat('ExampleSite', uniqueString(resourceGroup().id))

resource serverFarm 'Microsoft.Web/serverfarms@2020-12-01' = {
  name: hostingPlanName
  location: location
  sku: {
    tier: 'Free'
    name: 'f1'
    capacity: 0
  }
  properties: {
    targetWorkerCount: 1
  }
}

resource webSite 'Microsoft.Web/sites@2020-12-01' = {
  name: siteName
  location: location
  properties: {
    serverFarmId: serverFarm.name
  }
}

resource siteLock 'Microsoft.Authorization/locks@2016-09-01' = {
  name: 'siteLock'
  scope: webSite
  properties:{
    level: 'CanNotDelete'
    notes: 'Site should not be deleted.'
  }
}
```

---

### <a name="azure-powershell"></a>Azure PowerShell

Bloquee recursos implementados con Azure PowerShell mediante el comando [New-AzResourceLock](/powershell/module/az.resources/new-azresourcelock).

Para bloquear un recurso, proporcione el nombre del recurso, su tipo y el nombre del grupo de recursos.

```azurepowershell-interactive
New-AzResourceLock -LockLevel CanNotDelete -LockName LockSite -ResourceName examplesite -ResourceType Microsoft.Web/sites -ResourceGroupName exampleresourcegroup
```

Para bloquear un grupo de recursos, proporcione el nombre del grupo de recursos.

```azurepowershell-interactive
New-AzResourceLock -LockName LockGroup -LockLevel CanNotDelete -ResourceGroupName exampleresourcegroup
```

Para obtener información sobre un bloqueo, use [Get-AzResourceLock](/powershell/module/az.resources/get-azresourcelock). Para obtener todos los bloqueos en su suscripción, use:

```azurepowershell-interactive
Get-AzResourceLock
```

Para obtener todos los bloqueos para un recurso, use:

```azurepowershell-interactive
Get-AzResourceLock -ResourceName examplesite -ResourceType Microsoft.Web/sites -ResourceGroupName exampleresourcegroup
```

Para obtener todos los bloqueos para un grupo de recursos, use:

```azurepowershell-interactive
Get-AzResourceLock -ResourceGroupName exampleresourcegroup
```

Para eliminar un bloqueo de un recurso, use:

```azurepowershell-interactive
$lockId = (Get-AzResourceLock -ResourceGroupName exampleresourcegroup -ResourceName examplesite -ResourceType Microsoft.Web/sites).LockId
Remove-AzResourceLock -LockId $lockId
```

Para eliminar un bloqueo de un grupo de recursos, use:

```azurepowershell-interactive
$lockId = (Get-AzResourceLock -ResourceGroupName exampleresourcegroup).LockId
Remove-AzResourceLock -LockId $lockId
```

### <a name="azure-cli"></a>Azure CLI

Bloquee recursos implementados con la CLI de Azure mediante el comando [az lock create](/cli/azure/lock#az_lock_create).

Para bloquear un recurso, proporcione el nombre del recurso, su tipo y el nombre del grupo de recursos.

```azurecli
az lock create --name LockSite --lock-type CanNotDelete --resource-group exampleresourcegroup --resource-name examplesite --resource-type Microsoft.Web/sites
```

Para bloquear un grupo de recursos, proporcione el nombre del grupo de recursos.

```azurecli
az lock create --name LockGroup --lock-type CanNotDelete --resource-group exampleresourcegroup
```

Para obtener información sobre un bloqueo, use [az lock list](/cli/azure/lock#az_lock_list). Para obtener todos los bloqueos en su suscripción, use:

```azurecli
az lock list
```

Para obtener todos los bloqueos para un recurso, use:

```azurecli
az lock list --resource-group exampleresourcegroup --resource-name examplesite --namespace Microsoft.Web --resource-type sites --parent ""
```

Para obtener todos los bloqueos para un grupo de recursos, use:

```azurecli
az lock list --resource-group exampleresourcegroup
```

Para eliminar un bloqueo de un recurso, use:

```azurecli
lockid=$(az lock show --name LockSite --resource-group exampleresourcegroup --resource-type Microsoft.Web/sites --resource-name examplesite --output tsv --query id)
az lock delete --ids $lockid
```

Para eliminar un bloqueo de un grupo de recursos, use:

```azurecli
lockid=$(az lock show --name LockSite --resource-group exampleresourcegroup  --output tsv --query id)
az lock delete --ids $lockid
```

### <a name="rest-api"></a>API DE REST

Puede bloquear los recursos implementados con la [API de REST para bloqueos de administración](/rest/api/resources/managementlocks). La API de REST le permite crear y eliminar bloqueos, y recuperar información acerca de los bloqueos existentes.

Para crear un bloqueo, ejecute:

```http
PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/locks/{lock-name}?api-version={api-version}
```

El ámbito puede ser una suscripción, un grupo de recursos o un recurso. El nombre del bloqueo es el nombre con el que desee llamar al bloqueo. Como versión de la API, use **2016-09-01**.

En la solicitud, incluya un objeto JSON que especifique las propiedades para el bloqueo.

```json
{
  "properties": {
  "level": "CanNotDelete",
  "notes": "Optional text notes."
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

- Para aprender a organizar de manera lógica los recursos, consulte [Uso de etiquetas para organizar sus recursos](tag-resources.md).
- Puede aplicar restricciones y convenciones a través de su suscripción con directivas personalizadas. Para obtener más información, consulte [¿Qué es Azure Policy?](../../governance/policy/overview.md)
- Para obtener instrucciones sobre cómo las empresas pueden utilizar Resource Manager para administrar eficazmente las suscripciones, vea [Scaffold empresarial de Azure: Gobernanza de suscripción prescriptiva](/azure/architecture/cloud-adoption-guide/subscription-governance).
