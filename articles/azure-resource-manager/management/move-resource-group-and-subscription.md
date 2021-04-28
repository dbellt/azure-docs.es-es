---
title: Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción
description: Use Azure Resource Manager para trasladar recursos a un nuevo grupo de recursos o a una nueva suscripción.
ms.topic: conceptual
ms.date: 04/16/2021
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7a50ecc6081f8fa7c7600ddf1f98a95eceafa73b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784630"
---
# <a name="move-resources-to-a-new-resource-group-or-subscription"></a>Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción

En este artículo se explica cómo trasladar recursos de Azure a otra suscripción de Azure o a otro grupo de recursos en la misma suscripción. Puede usar Azure Portal, Azure PowerShell, la CLI de Azure o la API REST para trasladar recursos.

Tanto el grupo de origen como el grupo de destino se bloquean durante la operación de traslado. Las operaciones de escritura y eliminación están bloqueadas en los grupos de recursos hasta que se completa el movimiento. Este bloqueo significa que no puede agregar, actualizar ni eliminar recursos de los grupos de recursos. Pero no significa que los recursos estén inmovilizados. Por ejemplo, si mueve un servidor lógico de Azure SQL y sus bases de datos a un nuevo grupo de recursos o a una nueva suscripción, las aplicaciones que usan las bases de datos no experimentan tiempo de inactividad. Seguirá pudiendo leer y escribir en estas bases de datos. El bloqueo puede durar un máximo de cuatro horas, pero la mayoría de los movimientos se completan en mucho menos tiempo.

Si se mueve un recurso, solo se mueve a un nuevo grupo de recursos o suscripción. No cambia la ubicación del recurso.

## <a name="changed-resource-id"></a>Cambio en el identificador de recurso

Cuando se mueve un recurso, cambia su identificador de recurso. El formato estándar para un identificador de recurso es `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}`. Al trasladar un recurso a un nuevo grupo de recursos o a una nueva suscripción, cambia uno o varios valores de esa ruta de acceso.

Si usa el identificador de recurso en cualquier lugar, deberá cambiar dicho valor. Por ejemplo, si tiene un [panel personalizado](../../azure-portal/quickstart-portal-dashboard-azure-cli.md) en el portal que hace referencia a un identificador de recurso, deberá actualizar dicho valor. Busque scripts o plantillas que deban actualizarse para el nuevo identificador de recurso.

## <a name="checklist-before-moving-resources"></a>Lista de comprobación antes de mover recursos

Hay algunos pasos importantes que deben realizarse antes de mover un recurso. Puede evitar errores mediante la comprobación de estas condiciones.

1. Los recursos que desea trasladar deben ser compatibles con la operación de traslado. Para una lista de qué recursos son compatibles con el traslado, consulte [Compatibilidad con la operación de traslado para recursos](move-support-resources.md).

1. Algunos servicios tienen limitaciones o requisitos específicos al trasladar recursos. Si ha trasladado cualquiera de los siguientes servicios, compruebe esa guía antes de trasladarlo.

   * Si usa Azure Stack Hub, no puede mover recursos entre grupos.
   * [Guía de traslado de App Services](./move-limitations/app-service-move-limitations.md)
   * [Guía de traslado de Azure DevOps Services](/azure/devops/organizations/billing/change-azure-subscription?toc=/azure/azure-resource-manager/toc.json)
   * [Guía de traslado del modelo de implementación clásica](./move-limitations/classic-model-move-limitations.md): proceso clásico, almacenamiento clásico, redes virtuales clásicas y Cloud Services
   * [Guía de traslado de red](./move-limitations/networking-move-limitations.md)
   * [Guía de traslado de Recovery Services](../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json)
   * [Guía de movimiento de Virtual Machines](./move-limitations/virtual-machines-move-limitations.md)
   * Para trasladar una suscripción de Azure a un nuevo grupo de administración, consulte [Movimiento de suscripciones](../../governance/management-groups/manage.md#move-subscriptions).

1. Si mueve un recurso que tiene un rol de Azure asignado directamente al recurso (o a un recurso secundario), la asignación de roles no se mueve y queda huérfana. Después de moverlo, debe volver a crear asignaciones de roles. Finalmente, la asignación de roles huérfana se quita automáticamente, pero se recomienda quitar la asignación de roles antes del cambio.

    Para más información sobre cómo administrar las asignaciones de roles, consulte [Enumeración de asignaciones de roles de Azure](../../role-based-access-control/role-assignments-list-portal.md#list-role-assignments-at-a-scope) y [Asignación de roles de Azure](../../role-based-access-control/role-assignments-portal.md).

1. Las suscripciones de origen y de destino deben estar activas. Si tiene problemas para habilitar una cuenta que se ha deshabilitado, [cree una solicitud de soporte técnico de Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Seleccione **Administración de suscripciones** para el tipo de problema.

1. Las suscripciones de origen y destino deben existir en el mismo [inquilino de Azure Active Directory](../../active-directory/develop/quickstart-create-new-tenant.md). Para comprobar que ambas suscripciones tienen el mismo identificador de inquilino, utilice Azure PowerShell o la CLI de Azure.

   Para Azure PowerShell, use:

   ```azurepowershell-interactive
   (Get-AzSubscription -SubscriptionName <your-source-subscription>).TenantId
   (Get-AzSubscription -SubscriptionName <your-destination-subscription>).TenantId
   ```

   Para la CLI de Azure, utilice:

   ```azurecli-interactive
   az account show --subscription <your-source-subscription> --query tenantId
   az account show --subscription <your-destination-subscription> --query tenantId
   ```

   Si los identificadores de inquilino de las suscripciones de origen y destino no son los mismos, use los siguientes métodos para conciliarlos:

   * [Transferencia de la propiedad de una suscripción de Azure a otra cuenta](../../cost-management-billing/manage/billing-subscription-transfer.md)
   * [Asociación o adición de una suscripción de Azure a Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)

1. La suscripción de destino correspondiente al proveedor de recursos del recurso que se traslada debe estar registrada. Si no es así, recibirá un error en el que se indicará que la **suscripción no está registrada para un tipo de recurso**. Podría encontrar este error al mover un recurso a una nueva suscripción que nunca se ha utilizado con ese tipo de recurso.

   En PowerShell, use los siguientes comandos para obtener el estado de registro:

   ```azurepowershell-interactive
   Set-AzContext -Subscription <destination-subscription-name-or-id>
   Get-AzResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
   ```

   Para registrar un proveedor de recursos, use:

   ```azurepowershell-interactive
   Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
   ```

   En la CLI de Azure, use los siguientes comandos para obtener el estado de registro:

   ```azurecli-interactive
   az account set -s <destination-subscription-name-or-id>
   az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
   ```

   Para registrar un proveedor de recursos, use:

   ```azurecli-interactive
   az provider register --namespace Microsoft.Batch
   ```

1. La cuenta que mueve los recursos debe tener al menos los permisos siguientes:

   * **Microsoft.Resources/subscriptions/resourceGroups/moveResources/action** en el grupo de recursos de origen.
   * **Microsoft.Resources/subscriptions/resourceGroups/write** en el grupo de recursos de destino.

1. Antes de mover los recursos, compruebe las cuotas de la suscripción a la que está trasladando los recursos. Si trasladar los recursos significa que la suscripción excederá sus límites, debe revisar si puede solicitar un aumento de la cuota. Para ver una lista de estos límites y cómo solicitar un aumento, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).

1. **Para un traslado entre suscripciones, el recurso y sus recursos dependientes deben encontrarse en el mismo grupo de recursos y deben trasladarse juntos.** Por ejemplo, una máquina virtual con discos administrados requeriría que la máquina virtual y los discos administrados se trasladaran juntos, junto con otros recursos dependientes.

   Si va a trasladar un recurso a una nueva suscripción, compruebe si el recurso tiene recursos dependientes y si están ubicados en el mismo grupo de recursos. Si los recursos no están en el mismo grupo de recursos, compruebe si los recursos pueden combinarse en el mismo grupo de recursos. Si es así, lleve todos estos recursos al mismo grupo de recursos mediante una operación de traslado entre grupos de recursos.

   Para más información, consulte [Escenario para el traslado entre suscripciones](#scenario-for-move-across-subscriptions).

## <a name="scenario-for-move-across-subscriptions"></a>Escenario para el traslado entre suscripciones

El traslado de recursos de una suscripción a otra es un proceso de tres pasos:

![escenario de traslado entre suscripciones](./media/move-resource-group-and-subscription/cross-subscription-move-scenario.png)

Con fines ilustrativos, solo tenemos un recurso dependiente.

* Paso 1: Si los recursos dependientes se distribuyen entre diferentes grupos de recursos, trasládelos primero a un grupo de recursos.
* Paso 2: Traslade el recurso y los recursos dependientes juntos desde la suscripción de origen hasta la de destino.
* Paso 3: Opcionalmente, redistribuya los recursos dependientes a distintos grupos de recursos dentro de la suscripción de destino.

## <a name="validate-move"></a>Validar el movimiento

La [operación de validación del movimiento](/rest/api/resources/resources/moveresources) le permite probar el escenario de movimiento sin mover realmente los recursos. Use esta operación para comprobar si el movimiento se realizará correctamente. La validación se llama automáticamente cuando se envía una solicitud de traslado. Use esta operación solo cuando necesite determinar de antemano los resultados. Para ejecutar esta operación, necesita el:

* nombre del grupo de recursos de origen
* identificador de recurso del grupo de recursos de destino
* identificador de recurso de cada recurso que se va a mover
* [token de acceso](/rest/api/azure/#acquire-an-access-token) de la cuenta

Envíe la solicitud siguiente:

```HTTP
POST https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<source-group>/validateMoveResources?api-version=2019-05-10
Authorization: Bearer <access-token>
Content-type: application/json
```

Con un cuerpo de la solicitud:

```json
{
 "resources": ["<resource-id-1>", "<resource-id-2>"],
 "targetResourceGroup": "/subscriptions/<subscription-id>/resourceGroups/<target-group>"
}
```

Si la solicitud tiene un formato correcto, la operación devuelve:

```HTTP
Response Code: 202
cache-control: no-cache
pragma: no-cache
expires: -1
location: https://management.azure.com/subscriptions/<subscription-id>/operationresults/<operation-id>?api-version=2018-02-01
retry-after: 15
...
```

El código de estado 202 indica que se ha aceptado la solicitud de validación, pero aún no se ha determinado si la operación de movimiento se realizará correctamente. El valor `location` contiene una dirección URL que se usa para comprobar el estado de la operación de ejecución larga.

Para comprobar el estado, envíe la solicitud siguiente:

```HTTP
GET <location-url>
Authorization: Bearer <access-token>
```

Mientras todavía se esté ejecutando la operación, continuará recibiendo el código de estado 202. Espere el número de segundos que se indica en el valor `retry-after` antes de intentarlo de nuevo. Si la operación de movimiento se valida correctamente, recibirá el código de estado 204. Si falla la validación de movimiento, recibirá un mensaje de error, como:

```json
{"error":{"code":"ResourceMoveProviderValidationFailed","message":"<message>"...}}
```

## <a name="use-the-portal"></a>Uso del portal

Para mover recursos, seleccione el grupo de recursos que contiene dichos recursos.

Cuando se visualiza el grupo de recursos, la opción para mover está deshabilitada.

:::image type="content" source="./media/move-resource-group-and-subscription/move-first-view.png" alt-text="opción para mover deshabilitada":::

Para habilitar la opción para mover, seleccione los recursos que desea trasladar. Para seleccionar todos los recursos, active la casilla situada en la parte superior de la lista. O bien, seleccione cada recurso de forma individual. Después de seleccionar los recursos, se habilita la opción para moverlos.

:::image type="content" source="./media/move-resource-group-and-subscription/select-resources.png" alt-text="seleccionar recursos":::

Seleccione el botón **Mover**.

:::image type="content" source="./media/move-resource-group-and-subscription/move-options.png" alt-text="opciones de movimiento":::

Este botón ofrece tres opciones:

* Mover a un grupo de recursos nuevo
* Mover a una suscripción nueva
* Mover a una región nueva Para cambiar las regiones, vea [Mover recursos entre regiones (desde un grupo de recursos)](../../resource-mover/move-region-within-resource-group.md?toc=/azure/azure-resource-manager/management/toc.json).

Seleccione si va a mover los recursos a un nuevo grupo de recursos o a una nueva suscripción.

Seleccione el grupo de recursos de destino. Confirme que tiene que actualizar los scripts para estos recursos y seleccione **Aceptar**. Si ha seleccionado mover a una suscripción nueva, también debe seleccionar la suscripción de destino.

:::image type="content" source="./media/move-resource-group-and-subscription/move-destination.png" alt-text="seleccionar destino":::

Después de validar que los recursos se pueden mover, verá una notificación de que se está ejecutando la operación de movimiento.

:::image type="content" source="./media/move-resource-group-and-subscription/move-notification.png" alt-text="notificación":::

Cuando haya finalizado, se le notificará del resultado.

## <a name="use-azure-powershell"></a>Uso de Azure PowerShell

Para mover recursos existentes a otro grupo de recursos o a otra suscripción, use el comando [Move-AzResource](/powershell/module/az.resources/move-azresource). El siguiente ejemplo muestra cómo trasladar varios recursos a un nuevo grupo de recursos.

```azurepowershell-interactive
$webapp = Get-AzResource -ResourceGroupName OldRG -ResourceName ExampleSite
$plan = Get-AzResource -ResourceGroupName OldRG -ResourceName ExamplePlan
Move-AzResource -DestinationResourceGroupName NewRG -ResourceId $webapp.ResourceId, $plan.ResourceId
```

Para moverlos a una nueva suscripción, especifique un valor para el parámetro `DestinationSubscriptionId`.

## <a name="use-azure-cli"></a>Uso de CLI de Azure

Para trasladar recursos existentes a otro grupo de recursos o a otra suscripción, use el comando [az resource move](/cli/azure/resource#az_resource_move). Proporcione los identificadores de recursos de los recursos que se van a mover. El siguiente ejemplo muestra cómo trasladar varios recursos a un nuevo grupo de recursos. En el parámetro `--ids`, ofrezca una lista separada por espacios de los identificadores de recurso que se van a trasladar.

```azurecli
webapp=$(az resource show -g OldRG -n ExampleSite --resource-type "Microsoft.Web/sites" --query id --output tsv)
plan=$(az resource show -g OldRG -n ExamplePlan --resource-type "Microsoft.Web/serverfarms" --query id --output tsv)
az resource move --destination-group newgroup --ids $webapp $plan
```

Para mover a una nueva suscripción, proporcione el parámetro `--destination-subscription-id`.

## <a name="use-rest-api"></a>Use la API de REST

Para mover recursos existentes a otro grupo de recursos o a otra suscripción, use la operación [Mover recursos](/rest/api/resources/resources/moveresources).

```HTTP
POST https://management.azure.com/subscriptions/{source-subscription-id}/resourcegroups/{source-resource-group-name}/moveResources?api-version={api-version}
```

En el cuerpo de la solicitud, especifique el grupo de recursos de destino y los recursos a mover.

```json
{
 "resources": ["<resource-id-1>", "<resource-id-2>"],
 "targetResourceGroup": "/subscriptions/<subscription-id>/resourceGroups/<target-group>"
}
```

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**Pregunta: La operación de movimiento de recursos, que normalmente tarda unos minutos, se ha ejecutado durante casi una hora. ¿Hay algo incorrecto?**

Mover un recurso es una operación compleja que tiene distintas fases. Pueden existir más implicaciones, además del proveedor de recursos del recurso que intenta mover. Debido a las dependencias entre proveedores de recursos, Azure Resource Manager admite 4 horas para que se complete la operación. Este período de tiempo proporciona a los proveedores de recursos una oportunidad de recuperarse de problemas transitorios. Si la solicitud de movimiento se encuentra dentro del período de 4 horas, la operación continúa intentando completar la ejecución, que puede ser correcta. Los grupos de recursos de origen y de destino se bloquean durante este tiempo para evitar problemas de coherencia.

**Pregunta: ¿Por qué mi grupo de recursos se bloquea durante 4 horas mientras se realiza el movimiento de recursos?**

Se permite que una solicitud de movimiento tarde hasta 4 horas en completarse. Para evitar que se muevan las modificaciones de los recursos, los grupos de recursos de origen y de destino se bloquean durante el movimiento de recursos.

Hay dos fases en una solicitud de movimiento. En la primera fase, se mueve el recurso. En la segunda fase, se envían notificaciones a otros proveedores de recursos que dependen del recurso que se está moviendo. Un grupo de recursos puede bloquearse durante las 4 horas completas cuando se produce un error en un proveedor de recursos en cualquiera de las fases. Durante el tiempo permitido, Resource Manager reintenta el paso con el error.

Si un recurso no se puede mover durante las 4 horas, Resource Manager desbloquea ambos grupos de recursos. Los recursos que se movieron correctamente se encuentran en el grupo de recursos de destino. Los recursos que no se pudieron mover se dejan en el grupo de recursos de origen.

**Pregunta: ¿Cuáles son las implicaciones de que los grupos de recursos de origen y de destino se bloqueen durante el movimiento de recursos?**

El bloqueo le impide eliminar cualquier grupo de recursos, crear un nuevo recurso en cualquiera de los grupos de recursos o eliminar cualquiera de los recursos implicados en el movimiento.

En la siguiente imagen se muestra un mensaje de error de Azure Portal cuando un usuario intenta eliminar un grupo de recursos que forma parte de un movimiento en curso.

![Mensaje de error de movimiento de intento de eliminación](./media/move-resource-group-and-subscription/move-error-delete.png)

**Pregunta: ¿Qué significa el código de error "MissingMoveDependentResources"?**

Al mover un recurso, sus recursos dependientes deben existir en el grupo de recursos o la suscripción de destino, o bien deben incluirse en la solicitud de movimiento. Se recibe el código de error MissingMoveDependentResources cuando un recurso dependiente no cumple este requisito. El mensaje de error contiene detalles sobre el recurso dependiente que debe incluirse en la solicitud de movimiento.

Por ejemplo, mover una máquina virtual podría requerir mover siete tipos de recursos con tres proveedores de recursos diferentes. Estos proveedores y tipos de recursos son:

* Microsoft.Compute

  * virtualMachines
  * disks
* Microsoft.Network
  * networkInterfaces
  * publicIPAddresses
  * networkSecurityGroups
  * virtualNetworks
* Microsoft.Storage
  * storageAccounts

Otro ejemplo común implica mover una red virtual. Es posible que tenga que mover varios recursos asociados a esa red virtual. La solicitud de movimiento podría requerir mover las direcciones IP públicas, las tablas de rutas, las puertas de enlace de red virtual, los grupos de seguridad de red y más.

**Pregunta: ¿Por qué no puedo mover algunos recursos de Azure?**

Actualmente, no todos los recursos de Azure se pueden mover. Para una lista de qué recursos son compatibles con el movimiento, consulte [Compatibilidad con la operación de traslado para recursos](move-support-resources.md).

**Pregunta: ¿Cuántos recursos puedo mover en una sola operación?**

Cuando sea posible, divida las operaciones de movimiento grandes en varias operaciones de movimiento independientes. Resource Manager inmediatamente devolverá un error si hay más de 800 recursos en una sola operación. No obstante, también se puede producir un error por agotamiento del tiempo de espera al mover menos de 800 recursos.

**Pregunta: ¿Qué significa el error de que un recurso no está en el estado correcto?**

Si recibe un mensaje de error que indica que no se puede mover un recurso porque no se encuentra en un estado correcto, es posible que en realidad sea un recurso dependiente lo que esté bloqueando el movimiento. Normalmente, el código de error es **MoveCannotProceedWithResourcesNotInSucceededState**.

Si el grupo de recursos de origen o de destino contiene una red virtual, los estados de todos los recursos dependientes de la red virtual se comprueban durante el traslado. La comprobación incluye los recursos directa e indirectamente dependientes de la red virtual. Si alguno de esos recursos se encuentra en un estado de error, el traslado se bloqueará. Por ejemplo, si se produce un error en una máquina virtual que usa la red virtual, el traslado se bloqueará. El traslado se bloqueará incluso cuando la máquina virtual no sea uno de los recursos que se están trasladando y no esté en uno de los grupos de recursos que forma parte del traslado.

Cuando recibe este error, tiene dos opciones: trasladar los recursos a un grupo de recursos que no tenga una red virtual o [ponerse en contacto con el departamento de soporte técnico](../../azure-portal/supportability/how-to-create-azure-support-request.md).

## <a name="next-steps"></a>Pasos siguientes

Para una lista de qué recursos son compatibles con el traslado, consulte [Compatibilidad con la operación de traslado para recursos](move-support-resources.md).
