---
title: Errores de recurso no encontrado
description: Describe cómo resolver errores cuando no se encuentra un recurso. El error se puede producir al implementar una plantilla de Azure Resource Manager o al realizar acciones de administración.
ms.topic: troubleshooting
ms.date: 03/23/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ffd1e5b2bd23c9a3f7ede4c3ee0067e12891d214
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108318844"
---
# <a name="resolve-resource-not-found-errors"></a>Solución de errores de recurso no encontrado

En este artículo se describen los errores que pueden aparecer cuando no se encuentra un recurso durante la implementación. Normalmente, este error se muestra al implementar recursos. También puede ver este error cuando realice tareas de administración y Azure Resource Manager no encuentre el recurso necesario. Por ejemplo, si intenta agregar etiquetas a un recurso que no existe, recibirá este error.

## <a name="symptom"></a>Síntoma

Hay dos códigos de error que indican que no se puede encontrar el recurso. El error **NotFound** devuelve un resultado similar al siguiente:

```
Code=NotFound;
Message=Cannot find ServerFarm with name exampleplan.
```

El error **ResourceNotFound** devuelve un resultado similar al siguiente:

```
Code=ResourceNotFound;
Message=The Resource 'Microsoft.Storage/storageAccounts/{storage name}' under resource
group {resource group name} was not found.
```

## <a name="cause"></a>Causa

Resource Manager necesita recuperar las propiedades de un recurso, pero no puede encontrar el recurso en las suscripciones.

## <a name="solution-1---check-resource-properties"></a>Solución 1: comprobación de las propiedades del recurso

Cuando reciba este error mientras realiza una tarea de administración, compruebe los valores que ha proporcionado para el recurso. Los tres valores que se deben comprobar son:

* Nombre del recurso
* Definición de un nombre de grupo de recursos
* Suscripción

Si usa PowerShell o la CLI de Azure, compruebe si el comando se ejecuta en la suscripción que contiene el recurso. Puede cambiar la suscripción con [Set-AzContext](/powershell/module/Az.Accounts/Set-AzContext) o [az account set](/cli/azure/account#az_account_set). Muchos comandos también proporcionan un parámetro de suscripción que le permite especificar una suscripción diferente a la del contexto actual.

Si tiene problemas para comprobar las propiedades, inicie sesión en el [portal](https://portal.azure.com). Busque el recurso que intenta usar y examine el nombre del recurso, el grupo de recursos y la suscripción.

## <a name="solution-2---set-dependencies"></a>Solución 2: establecimiento de dependencias

Si recibe este error al implementar una plantilla, es posible que tenga que agregar una dependencia. Cuando es posible, Resource Manager optimiza la implementación mediante la creación de recursos en paralelo. Si un recurso se debe implementar después de otro, debe usar el elemento **dependsOn** de la plantilla. Por ejemplo, al implementar una aplicación web, debe existir el plan de App Service. Si no ha especificado que la aplicación web dependa del plan de App Service, Resource Manager crea ambos recursos al mismo tiempo. Recibirá un error que indica que no se encuentra el recurso del plan de App Service, porque aún no existe cuando se intenta establecer una propiedad en la aplicación web. Este error se puede evitar estableciendo la dependencia en la aplicación web.

```json
{
  "type": "Microsoft.Web/sites",
  "apiVersion": "2015-08-01",
  "dependsOn": [
    "[variables('hostingPlanName')]"
  ],
  ...
}
```

No obstante, desea evitar configurar dependencias que no sean necesarias. Cuando hay dependencias innecesarias, prolonga la duración de la implementación al impedir que los recursos que no son dependientes entre sí se implementen en paralelo. Además, puede crear dependencias circulares que bloqueen la implementación. La función [reference](template-functions-resource.md#reference) y [list*](template-functions-resource.md#list) crea una dependencia implícita del recurso al que hace referencia, cuando este se implementa en la misma plantilla y se hace referencia a él con su nombre (no el identificador del recurso). Por lo tanto, puede tener más dependencias que las especificadas en la propiedad **dependsOn**. La función [resourceId](template-functions-resource.md#resourceid) no crea una dependencia implícita ni valida que el recurso existe. Las funciones [reference](template-functions-resource.md#reference) y [list*](template-functions-resource.md#list) no crean una dependencia implícita cuando el recurso se conoce por su identificador de recurso. Para crear una dependencia implícita, pase el nombre del recurso que se implementa en la misma plantilla.

Cuando se encuentre con problemas de dependencia, debe comprender mejor el orden de la implementación de recursos. Para ver el orden de las operaciones de implementación:

1. Seleccione el historial de implementaciones para el grupo de recursos.

   ![seleccionar el historial de implementaciones](./media/error-not-found/select-deployment.png)

2. Seleccione una implementación del historial y seleccione **Eventos**.

   ![seleccionar eventos de implementación](./media/error-not-found/select-deployment-events.png)

3. Examine la secuencia de eventos para cada recurso. Preste atención para el estado de cada operación. Por ejemplo, la siguiente imagen muestra tres cuentas de almacenamiento que se implementan en paralelo. Tenga en cuenta que las tres cuentas de almacenamiento se inician al mismo tiempo.

   ![implementación paralela](./media/error-not-found/deployment-events-parallel.png)

   La siguiente imagen muestra tres cuentas de almacenamiento que no se implementan en paralelo. La segunda cuenta de almacenamiento depende de la primera cuenta de almacenamiento y la tercera cuenta de almacenamiento depende de la segunda cuenta de almacenamiento. La primera cuenta de almacenamiento se inicia, se acepta y se completa antes de que se inicie la siguiente.

   ![implementación secuencial](./media/error-not-found/deployment-events-sequence.png)

## <a name="solution-3---get-external-resource"></a>Solución 3: obtención de recursos externos

Si necesita obtener un recurso que existe en otro grupo de recursos u otra suscripción al implementar una plantilla, utilice la [función resourceId](template-functions-resource.md#resourceid). Esta función devuelve el nombre completo del recurso.

Los parámetros de grupo de recursos y suscripción de la función resourceId son opcionales. Si no los proporciona, el valor predeterminado será la suscripción y el grupo de recursos actuales. Cuando trabaje con un recurso de otro grupo de recursos o suscripción, asegúrese de que proporciona dichos valores.

En el ejemplo siguiente se obtiene el identificador de recurso de un recurso que existe en otro grupo de recursos.

```json
"properties": {
  "name": "[parameters('siteName')]",
  "serverFarmId": "[resourceId('plangroup', 'Microsoft.Web/serverfarms', parameters('hostingPlanName'))]"
}
```

## <a name="solution-4---get-managed-identity-from-resource"></a>Solución 4: Obtención de una identidad administrada del recurso

Si va a implementar un recurso que crea implícitamente una [identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md), debe esperar hasta que ese recurso se implemente antes de recuperar los valores de la identidad administrada. Si pasa el nombre de la identidad administrada a la función [reference](template-functions-resource.md#reference), Resource Manager intenta resolver la referencia antes de implementar el recurso y la identidad. En su lugar, pase el nombre del recurso al que se aplica la identidad. Este enfoque garantiza que el recurso y la identidad administrada se implementan antes de que Resource Manager resuelva la función reference.

En la función reference, use `Full` para obtener todas las propiedades, incluida la identidad administrada.

El patrón es:

`"[reference(resourceId(<resource-provider-namespace>, <resource-name>), <API-version>, 'Full').Identity.propertyName]"`

> [!IMPORTANT]
> No use el patrón:
>
> `"[reference(concat(resourceId(<resource-provider-namespace>, <resource-name>),'/providers/Microsoft.ManagedIdentity/Identities/default'),<API-version>).principalId]"`
>
> Se producirá un error en la plantilla.

Por ejemplo, para obtener el identificador de la entidad de seguridad de una identidad administrada que se aplica a una máquina virtual, use:

```json
"[reference(resourceId('Microsoft.Compute/virtualMachines', variables('vmName')),'2019-12-01', 'Full').identity.principalId]",
```

O bien, para obtener el identificador de inquilino de una identidad administrada que se aplica a un conjunto de escalado de máquinas virtuales, use:

```json
"[reference(resourceId('Microsoft.Compute/virtualMachineScaleSets',  variables('vmNodeType0Name')), 2019-12-01, 'Full').Identity.tenantId]"
```

## <a name="solution-5---check-functions"></a>Solución 5: comprobación de las funciones

Al implementar una plantilla, busque expresiones que usen las funciones [reference](template-functions-resource.md#reference) o [listKeys](template-functions-resource.md#listkeys). Los valores que proporcione varían en función de si el recurso se está en la misma plantilla, grupo de recursos y suscripción. Compruebe que proporciona los valores de parámetro necesarios para su escenario. Si el recurso está en otro grupo de recursos, proporcione el identificador de recurso completo. Por ejemplo, para hacer referencia a una cuenta de almacenamiento en otro grupo de recursos, use:

```json
"[reference(resourceId('exampleResourceGroup', 'Microsoft.Storage/storageAccounts', 'myStorage'), '2017-06-01')]"
```

## <a name="solution-6---after-deleting-resource"></a>Solución 6: después de eliminar el recurso

Cuando se elimina un recurso, puede haber un breve período de tiempo en el que el recurso todavía aparece en el portal, pero no está disponible en realidad. Si selecciona el recurso, recibirá un error que indica que no se encuentra el recurso. Actualice el portal para obtener la vista más reciente.

Si el problema continúa después de una breve espera, [póngase en contacto con el soporte técnico](https://azure.microsoft.com/support/options/).
