---
title: what-if de implementación de Bicep
description: Determine qué cambios se realizarán en los recursos antes de implementar un archivo Bicep.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/01/2021
ms.author: tomfitz
ms.openlocfilehash: 157db0aa85fe086e656cfc98c7b94527416350d0
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112027756"
---
# <a name="bicep-deployment-what-if-operation"></a>Operación what-if de implementación de Bicep

Antes de implementar un archivo Bicep, puede obtener una vista previa de los cambios que se producirán. Azure Resource Manager proporciona la operación what-if que le permite ver cómo cambiarán los recursos si implementa el archivo Bicep. La operación hipotética no realiza ningún cambio en los recursos existentes. En su lugar, predice los cambios si se implementa el archivo Bicep.

Puede usar la operación what-if con Azure PowerShell, la CLI de Azure o las operaciones de la API REST. La operación what-if se admite en las implementaciones de grupos de recursos, nivel de suscripción, grupos de administración e inquilinos.

## <a name="install-azure-powershell-module"></a>Instalación del módulo de Azure PowerShell

Para usar operaciones what-if en PowerShell, debe tener la versión **4.2 o posterior del módulo Az**.

Para instalar el módulo, use:

```powershell
Install-Module -Name Az -Force
```

Para más información sobre cómo instalar los módulos, consulte [Instalación de Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="install-azure-cli-module"></a>Instalación del módulo de la CLI de Azure

Para usar what-if en la CLI de Azure, debe tener la CLI de Azure versión 2.14.0 o posterior. Si es necesario, [instale la versión más reciente de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="see-results"></a>Ver los resultados

Cuando se usa what-if en PowerShell o la CLI de Azure, la salida incluye resultados codificados por colores que le ayudarán a apreciar los distintos tipos de cambios.

![Operación what-if de implementación de Bicep al administrador de recursos fullresourcepayload y tipos de cambio](./media/deploy-what-if/resource-manager-deployment-whatif-change-types.png)

La salida de texto es:

```powershell
Resource and property changes are indicated with these symbols:
  - Delete
  + Create
  ~ Modify

The deployment will update the following scope:

Scope: /subscriptions/./resourceGroups/ExampleGroup

  ~ Microsoft.Network/virtualNetworks/vnet-001 [2018-10-01]
    - tags.Owner: "Team A"
    ~ properties.addressSpace.addressPrefixes: [
      - 0: "10.0.0.0/16"
      + 0: "10.0.0.0/15"
      ]
    ~ properties.subnets: [
      - 0:

          name:                     "subnet001"
          properties.addressPrefix: "10.0.0.0/24"

      ]

Resource changes: 1 to modify.
```

> [!NOTE]
> La operación what-if no puede resolver la [función de referencia](./bicep-functions-resource.md#reference). Cada vez que se establece una propiedad en una expresión de plantilla que incluye la función de referencia, la operación what-if informa que la propiedad cambiará. Este comportamiento se produce porque la operacióno what-if compara el valor actual de la propiedad (como `true` o `false` para un valor booleano) con la expresión de plantilla sin resolver. Obviamente, estos valores no coinciden. Cuando se implementa el archivo Bicep, la propiedad solo cambia cuando la expresión de plantilla se resuelve en un valor diferente.

## <a name="what-if-commands"></a>Comandos what-if

### <a name="azure-powershell"></a>Azure PowerShell

Para obtener una vista previa de los cambios antes de implementar un archivo Bicep, use [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) o [New-AzSubscriptionDeployment](/powershell/module/az.resources/new-azdeployment). Agregue el parámetro de modificador `-Whatif` al comando de implementación.

* `New-AzResourceGroupDeployment -Whatif` para implementaciones de grupos de recursos
* `New-AzSubscriptionDeployment -Whatif` y `New-AzDeployment -Whatif` para implementaciones de nivel de suscripción

Puede usar el parámetro de modificador `-Confirm` para obtener una vista previa de los cambios y recibir un aviso para continuar con la implementación.

* `New-AzResourceGroupDeployment -Confirm` para implementaciones de grupos de recursos
* `New-AzSubscriptionDeployment -Confirm` y `New-AzDeployment -Confirm` para implementaciones de nivel de suscripción

Los comandos anteriores devuelven un resumen de texto que puede inspeccionar manualmente. Para obtener un objeto que se pueda inspeccionar mediante programación en busca de cambios, use [Get-AzResourceGroupDeploymentWhatIfResult](/powershell/module/az.resources/get-azresourcegroupdeploymentwhatifresult) o [Get-AzSubscriptionDeploymentWhatIfResult](/powershell/module/az.resources/get-azdeploymentwhatifresult).

* `$results = Get-AzResourceGroupDeploymentWhatIfResult` para implementaciones de grupos de recursos
* `$results = Get-AzSubscriptionDeploymentWhatIfResult` o `$results = Get-AzDeploymentWhatIfResult` para implementaciones de nivel de suscripción

### <a name="azure-cli"></a>Azure CLI

Para obtener una vista previa de los cambios antes de implementar una plantilla, use:

* [az deployment group what-if](/cli/azure/deployment/group#az_deployment_group_what_if) para implementaciones de grupos de recursos
* [az deployment sub what-if](/cli/azure/deployment/sub#az_deployment_sub_what_if) para implementaciones de nivel de suscripción
* [az deployment mg what-if](/cli/azure/deployment/mg#az_deployment_mg_what_if) para implementaciones de grupos de administración
* [az deployment tenant what-if](/cli/azure/deployment/tenant#az_deployment_tenant_what_if) para implementaciones de inquilinos

Puede usar el modificador `--confirm-with-what-if` (o su forma abreviada `-c`) para obtener una vista previa de los cambios y recibir un aviso para continuar con la implementación. Agregue este modificador a:

* [az deployment group create](/cli/azure/deployment/group#az_deployment_group_create)
* [az deployment sub create](/cli/azure/deployment/sub#az_deployment_sub_create)
* [az deployment mg create](/cli/azure/deployment/mg#az_deployment_mg_create)
* [az deployment tenant create](/cli/azure/deployment/tenant#az_deployment_tenant_create)

Por ejemplo, use `az deployment group create --confirm-with-what-if` o `-c` para implementaciones de grupos de recursos.

Los comandos anteriores devuelven un resumen de texto que puede inspeccionar manualmente. Para obtener un objeto JSON en el que pueda inspeccionar los cambios mediante programación, use el modificador `--no-pretty-print`. Por ejemplo, use `az deployment group what-if --no-pretty-print` para implementaciones de grupos de recursos.

Si quiere devolver los resultados sin colores, abra el archivo de [configuración de la CLI de Azure](/cli/azure/azure-cli-configuration). Establezca **no_color** en **yes**.

### <a name="azure-rest-api"></a>API REST de Azure

Para la API REST, use:

* [Implementaciones: What If](/rest/api/resources/deployments/whatif) para implementaciones de grupos de recursos
* [Implementaciones: What If en el ámbito de suscripción](/rest/api/resources/deployments/whatifatsubscriptionscope) para implementaciones de nivel de suscripción.
* [Implementaciones: What If en el ámbito del grupo de administración](/rest/api/resources/deployments/whatifatmanagementgroupscope) para implementaciones de grupos de administración.
* [Implementaciones: What If en el ámbito de inquilino](/rest/api/resources/deployments/whatifattenantscope) para implementaciones de inquilinos.

## <a name="change-types"></a>Tipos de cambio

La operación hipotética muestra seis tipos diferentes de cambios:

* **Create**: el recurso no existe actualmente, pero se define en el archivo Bicep. Se creará el recurso.
* **Delete**: este tipo de cambio solo se aplica cuando se usa el [modo completo](../templates/deployment-modes.md) para la implementación de plantillas JSON. El recurso existe, pero no se define en el archivo Bicep. Con el modo completo, se eliminará el recurso. Solo los recursos que [admiten la eliminación de modo completo](../templates/deployment-complete-mode-deletion.md) se incluyen en este tipo de cambio.
* **Ignore**: el recurso existe, pero no está definido en el archivo Bicep. No se implementará ni modificará el recurso.
* **NoChang:** : el recurso existe y se define en el archivo Bicep. El recurso se volverá a implementar, pero las propiedades del recurso no cambiarán. Este tipo de cambio se devuelve cuando [ResultFormat](#result-format) se establece en `FullResourcePayloads`, que es el valor predeterminado.
* **Modify**: el recurso existe y se define en el archivo Bicep. El recurso se volverá a implementar y las propiedades del recurso cambiarán. Este tipo de cambio se devuelve cuando [ResultFormat](#result-format) se establece en `FullResourcePayloads`, que es el valor predeterminado.
* **Deploy**: el recurso existe y se define en el archivo Bicep. El recurso será reimplementado. Es posible que las propiedades del recurso cambien o no. La operación devuelve este tipo de cambio cuando no tiene información suficiente para determinar si cualquier propiedad cambiará. Solo verá esta condición cuando [ResultFormat](#result-format) se establece en `ResourceIdOnly`.

## <a name="result-format"></a>Formato de resultado

Puede controlar el nivel de detalle que se devuelve sobre la predicción de cambios. Tiene dos opciones:

* **FullResourcePayloads**: devuelve una lista de los recursos que van a cambiar y detalles sobre las propiedades que cambiarán.
* **ResourceIdOnly**: devuelve una lista de los recursos que cambiarán.

El valor predeterminado es **FullResourcePayloads**.

Para los comandos de implementación de PowerShell, use el parámetro `-WhatIfResultFormat`. En los comandos de objeto de programación, use el parámetro `ResultFormat`.

Para la CLI de Azure, use el parámetro `--result-format`.

Los resultados siguientes muestran los dos formatos de salida diferentes:

* Cargas de recursos completas

  ```powershell
  Resource and property changes are indicated with these symbols:
    - Delete
    + Create
    ~ Modify

  The deployment will update the following scope:

  Scope: /subscriptions/./resourceGroups/ExampleGroup

    ~ Microsoft.Network/virtualNetworks/vnet-001 [2018-10-01]
      - tags.Owner: "Team A"
      ~ properties.addressSpace.addressPrefixes: [
        - 0: "10.0.0.0/16"
        + 0: "10.0.0.0/15"
        ]
      ~ properties.subnets: [
        - 0:

          name:                     "subnet001"
          properties.addressPrefix: "10.0.0.0/24"

        ]

  Resource changes: 1 to modify.
  ```

* Solo identificador de recurso

  ```powershell
  Resource and property changes are indicated with this symbol:
    ! Deploy

  The deployment will update the following scope:

  Scope: /subscriptions/./resourceGroups/ExampleGroup

    ! Microsoft.Network/virtualNetworks/vnet-001

  Resource changes: 1 to deploy.
  ```

## <a name="run-what-if-operation"></a>Ejecutar la operación hipotética

### <a name="set-up-environment"></a>Configuración de entorno

Para ver cómo funciona, vamos a ejecutar algunas pruebas. En primer lugar, [implemente un archivo Bicep que cree una red virtual](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/what-if/what-if-before.bicep). Usará esta red virtual para probar cómo se registran los cambios con what-if. Descargue una copia del archivo Bicep.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroup `
  -Name ExampleGroup `
  -Location centralus
New-AzResourceGroupDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateFile "what-if-before.bicep"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az group create \
  --name ExampleGroup \
  --location "Central US"
az deployment group create \
  --resource-group ExampleGroup \
  --template-file "what-if-before.bicep"
```

---

### <a name="test-modification"></a>Modificación de prueba

Una vez finalizada la implementación, está listo para probar la operación hipotética. Esta vez se implementa un [archivo Bicep que cambia la red virtual](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/what-if/what-if-after.bicep). Falta una de las etiquetas originales, se ha quitado una subred y el prefijo de dirección ha cambiado. Descargue una copia del archivo Bicep.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Whatif `
  -ResourceGroupName ExampleGroup `
  -TemplateFile "what-if-after.bicep"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az deployment group what-if \
  --resource-group ExampleGroup \
  --template-file "what-if-after.bicep"
```

---

La salida de what-if es similar a esta:

![Salida de la operación what-if de implementación de Bicep](./media/deploy-what-if/resource-manager-deployment-whatif-change-types.png)

La salida de texto es:

```powershell
Resource and property changes are indicated with these symbols:
  - Delete
  + Create
  ~ Modify

The deployment will update the following scope:

Scope: /subscriptions/./resourceGroups/ExampleGroup

  ~ Microsoft.Network/virtualNetworks/vnet-001 [2018-10-01]
    - tags.Owner:                    "Team A"
    + properties.enableVmProtection: false
    ~ properties.addressSpace.addressPrefixes: [
      - 0: "10.0.0.0/16"
      + 0: "10.0.0.0/15"
      ]
    ~ properties.subnets: [
      - 0:

          name:                     "subnet001"
          properties.addressPrefix: "10.0.0.0/24"

      ]

Resource changes: 1 to modify.
```

Observe en la parte superior de la salida que los colores están definidos para indicar el tipo de cambios.

En la parte inferior de la salida, se muestra que se eliminó el propietario de la etiqueta. El prefijo de dirección cambió de 10.0.0.0/16 a 10.0.0.0/15. Se eliminó la subred denominada subnet001. Recuerde que estos cambios no se implementaron. Obtendrá una vista previa de los cambios que se producirán si implementa la plantilla.

Algunas de las propiedades que se enumeran como eliminadas no cambiarán realmente. Las propiedades se pueden notificar incorrectamente como eliminadas cuando no están en el archivo Bicep, pero se establecen de forma automática durante la implementación como valores predeterminados. Este resultado se considera "ruido" en la respuesta what-if. El recurso implementado final tendrá los valores establecidos para las propiedades. A medida que la operación what-if evolucione, estas propiedades se excluirán del resultado.

## <a name="programmatically-evaluate-what-if-results"></a>Evaluación de los resultados de what-if mediante programación

Ahora, vamos a establecer el comando en una variable para evaluar mediante programación los resultados de what-if.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$results = Get-AzResourceGroupDeploymentWhatIfResult `
  -ResourceGroupName ExampleGroup `
  --template-file "what-if-after.bicep"
```

Puede ver un resumen de cada cambio.

```azurepowershell
foreach ($change in $results.Changes)
{
  $change.Delta
}
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
results=$(az deployment group what-if --resource-group ExampleGroup --template-file "what-if-after.bicep" --no-pretty-print)
```

---

## <a name="confirm-deletion"></a>Confirmar eliminación

Para obtener una vista previa de los cambios antes de implementar un archivo Bicep, use el parámetro de modificador confirm con el comando de implementación. Si los cambios son los esperados, responda que desea que se complete la implementación.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -ResourceGroupName ExampleGroup `
  -Confirm `
  -TemplateFile "what-if-after.bicep"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az deployment group create \
  --resource-group ExampleGroup \
  --confirm-with-what-if \
  --template-file "what-if-after.bicep"
```

---

![Implementación de Bicep: modo de implementación de salida de operación what-if completado](./media/deploy-what-if/resource-manager-deployment-whatif-output-confirm.png)

La salida de texto es:

```powershell
Resource and property changes are indicated with these symbols:
  - Delete
  + Create
  ~ Modify

The deployment will update the following scope:

Scope: /subscriptions/./resourceGroups/ExampleGroup

  ~ Microsoft.Network/virtualNetworks/vnet-001 [2018-10-01]
    - tags.Owner:                    "Team A&quot;
    + properties.enableVmProtection: false
    ~ properties.addressSpace.addressPrefixes: [
      - 0: &quot;10.0.0.0/16&quot;
      + 0: &quot;10.0.0.0/15&quot;
      ]
    ~ properties.subnets: [
      - 0:

          name:                     &quot;subnet001&quot;
          properties.addressPrefix: &quot;10.0.0.0/24&quot;

      ]

Resource changes: 1 to modify.

Are you sure you want to execute the deployment?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is &quot;Y"):
```

Verá los cambios esperados y puede confirmar que desea que se ejecute la implementación.

## <a name="sdks"></a>SDK

Puede usar la operación "what-if" a través de los SDK de Azure.

* Para Python, use [what-if](/python/api/azure-mgmt-resource/azure.mgmt.resource.resources.v2019_10_01.operations.deploymentsoperations#what-if-resource-group-name--deployment-name--properties--location-none--custom-headers-none--raw-false--polling-true----operation-config-).

* Para Java, use la [clase DeploymentWhatIf](/java/api/com.microsoft.azure.management.resources.deploymentwhatif).

* Para .NET, use la [clase DeploymentWhatIf](/dotnet/api/microsoft.azure.management.resourcemanager.models.deploymentwhatif).

## <a name="next-steps"></a>Pasos siguientes

* Para usar la operación what-if en una canalización, consulte [Prueba de plantillas de Resource Manager con What-If en una canalización](https://4bes.nl/2021/03/06/test-arm-templates-with-what-if/).
* Si observa resultados incorrectos de la operación hipotética, informe de los problemas en [https://aka.ms/whatifissues](https://aka.ms/whatifissues).
* Para información sobre un módulo de Microsoft Learn que abarque el uso de hipótesis, consulte [Obtención de una vista previa de los cambios y validación de recursos de Azure mediante What If y el kit de herramientas de pruebas de plantillas de ARM](/learn/modules/arm-template-test/).
