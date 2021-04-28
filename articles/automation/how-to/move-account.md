---
title: Traslado de la cuenta de Azure Automation a otra suscripción
description: En este artículo se indica cómo mover la cuenta de Automation a otra suscripción.
services: automation
ms.subservice: process-automation
ms.date: 01/07/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1fd9350baf1805c0e6278b6210ad9818fa18c8d8
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107831499"
---
# <a name="move-your-azure-automation-account-to-another-subscription"></a>Traslado de la cuenta de Azure Automation a otra suscripción

Azure Automation permite mover algunos recursos a un nuevo grupo de recursos o a una nueva suscripción. Puede mover recursos mediante Azure Portal, PowerShell, la CLI de Azure o la API REST. Para más información sobre el proceso, consulte [Movimiento de recursos a un grupo de recursos o suscripción nuevos](../../azure-resource-manager/management/move-resource-group-and-subscription.md).

La cuenta de Automation es uno de los recursos que se pueden mover. En este artículo, conocerá el procedimiento para mover cuentas de Automation a otro recurso o suscripción. Los pasos generales para mover la cuenta de Automation son:

1. Deshabilite las características.
2. Desvincule el área de trabajo.
3. Mueva la cuenta de Automation.
4. Elimine las cuentas de ejecución y vuelva a crearlas.
5. Vuelva a habilitar las características.

## <a name="remove-features"></a>Eliminación de las características

Para desvincular el área de trabajo de la cuenta de Automation, se deben quitar los recursos de características del área de trabajo:

- Change Tracking e Inventario
- Administración de actualizaciones
- Inicio y detención de máquinas virtuales durante las horas de trabajo

1. Ubique el grupo de recursos en Azure Portal.
2. Busque cada característica y seleccione **Eliminar** en la página **Eliminar recursos**.

    ![Captura de pantalla de la eliminación de recursos de características en Azure Portal](../media/move-account/delete-solutions.png)

Si lo prefiere, puede eliminar los recursos mediante el cmdlet [Remove-AzResource](/powershell/module/Az.Resources/Remove-AzResource):

```azurepowershell-interactive
$workspaceName = <myWorkspaceName>
$resourceGroupName = <myResourceGroup>
Remove-AzResource -ResourceType 'Microsoft.OperationsManagement/solutions' -ResourceName "ChangeTracking($workspaceName)" -ResourceGroupName $resourceGroupName
Remove-AzResource -ResourceType 'Microsoft.OperationsManagement/solutions' -ResourceName "Updates($workspaceName)" -ResourceGroupName $resourceGroupName
Remove-AzResource -ResourceType 'Microsoft.OperationsManagement/solutions' -ResourceName "Start-Stop-VM($workspaceName)" -ResourceGroupName $resourceGroupName
```

### <a name="remove-alert-rules-for-startstop-vms-during-off-hours"></a>Eliminación de reglas de alerta de la característica Start/Stop VMs during off hours

En el caso de la característica Start/Stop VMs during off hours, también debe quitar las reglas de alerta creadas por ella.

1. En Azure Portal, vaya al grupo de recursos y seleccione **Supervisión** > **Alertas** > **Administrar reglas de alerta**.

   ![Captura de pantalla de la página Alertas, que muestra la selección de Administrar reglas de alerta](../media/move-account/alert-rules.png)

2. En la página Reglas, debería ver una lista de las alertas configuradas en ese grupo de recursos. La característica crea estas reglas:

    * AutoStop_VM_Child
    * ScheduledStartStop_Parent
    * SequencedStartStop_Parent

3. Seleccione las reglas de una en una y haga clic en **Eliminar** para quitarlas.

    ![Captura de pantalla de la página Reglas que solicita confirmación de eliminación de las reglas seleccionadas](../media/move-account/delete-rules.png)

    > [!NOTE]
    > Si no ve ninguna regla de alerta en la página Reglas, cambie el campo **Estado** a **Deshabilitado** para mostrar las alertas deshabilitadas. 

4. Cuando quite las reglas de alerta, debe quitar el grupo de acciones creado para las notificaciones de Start/Stop VMs during off hours. En Azure Portal, seleccione **Monitor** > **Alertas** > **Administrar los grupos de acciones**.

5. Seleccione **StartStop_VM_Notification**. 

6. En la página del grupo de acciones, seleccione **Eliminar**.

    ![Captura de pantalla de la página Grupo de acciones](../media/move-account/delete-action-group.png)

Si lo prefiere, puede eliminar el grupo de acciones mediante el cmdlet [Remove-AzActionGroup](/powershell/module/az.monitor/remove-azactiongroup):

```azurepowershell-interactive
Remove-AzActionGroup -ResourceGroupName <myResourceGroup> -Name StartStop_VM_Notification
```

## <a name="unlink-your-workspace"></a>Desvinculación del área de trabajo

Ahora puede desvincular el área de trabajo:

1. En Azure Portal, seleccione **Cuenta de Automation** > **Recursos relacionados** > **Área de trabajo vinculada**. 

2. Seleccione **Desvincular área de trabajo** para desvincular el área de trabajo de la cuenta de Automation.

    ![Captura de pantalla de la desvinculación de un área de trabajo de una cuenta de Automation](../media/move-account/unlink-workspace.png)

## <a name="move-your-automation-account"></a>Traslado de la cuenta de Automation

Ahora puede mover la cuenta de Automation y sus runbooks. 

1. En Azure Portal, vaya al grupo de recursos de la cuenta de Automation. Seleccione **Mover** > **Mover a otra suscripción**.

    ![Captura de pantalla de la página Grupo de recursos, traslado a otra suscripción](../media/move-account/move-resources.png)

2. Seleccione los recursos del grupo de recursos que quiere mover. Asegúrese de incluir los recursos de la cuenta de Automation, los runbooks y el área de trabajo de Log Analytics.

## <a name="re-create-run-as-accounts"></a>Volver a crear las cuentas de ejecución

Las [cuentas de ejecución](../automation-security-overview.md#run-as-accounts) crean una entidad de servicio en Azure Active Directory para autenticarse con recursos de Azure. Al cambiar las suscripciones, la cuenta de Automation ya no usa la cuenta de ejecución existente. Para volver a crear las cuentas de ejecución:

1. Vaya a la cuenta de Automation en la nueva suscripción y, en **Configuración de la cuenta**, seleccione **Cuentas de ejecución**. Verá que las cuentas de ejecución se muestran ahora como incompletas.

    ![Captura de pantalla de cuentas de ejecución, que se muestran incompletas](../media/move-account/run-as-accounts.png)

2. Elimine las cuentas de ejecución de una en una mediante el botón **Eliminar** de la página **Propiedades**. 

    > [!NOTE]
    > Si no tiene permisos para crear o ver las cuentas de ejecución, verá el siguiente mensaje: `You do not have permissions to create an Azure Run As account (service principal) and grant the Contributor role to the service principal.` Para más información, consulte [Permisos necesarios para configurar cuentas de ejecución](../automation-security-overview.md#permissions).

3. Después de eliminar las cuentas de ejecución, seleccione **Crear** en **Cuenta de ejecución de Azure**. 

4. En la página Agregar cuenta de ejecución de Azure, seleccione **Crear** para crear la cuenta de ejecución y la entidad de servicio. 

5. Repita los pasos anteriores con la cuenta de ejecución de Azure clásico.

## <a name="enable-features"></a>Habilitación de las características

Después de volver a crear las cuentas de ejecución, debe habilitar de nuevo las características que quitó antes de que trasladara la cuenta:

1. Para activar Change Tracking e Inventario, seleccione **Change Tracking e Inventario** en su cuenta de Automation. Elija el área de trabajo de Log Analytics que trasladó y seleccione **Habilitar**.

2. Repita el paso 1 con Update Management.

    ![Captura de pantalla de la nueva habilitación de las características en la cuenta de Automation trasladada](../media/move-account/reenable-solutions.png)

3. Las máquinas que se habilitan con sus características son visibles una vez que ha conectado el área de trabajo de Log Analytics existente. Para activar la característica Start/Stop VMs during off-hours, debe volver a habilitarla. En **Recursos relacionados**, seleccione **Start/Stop VMs** > **Más información acerca de la solución y cómo habilitarla** > **Crear** para iniciar la implementación.

4. En la página Agregar solución, elija el área de trabajo de Log Analytics y la cuenta de Automation.

    ![Captura de pantalla del menú Agregar solución](../media/move-account/add-solution-vm.png)

5. Configure la característica como se describe en [Introducción a Start/Stop VMs during off-hours](../automation-solution-vm-management.md).

## <a name="verify-the-move"></a>Comprobación del traslado

Una vez completado el traslado, compruebe las que funcionalidades de la lista siguiente estén habilitadas. 

|Capacidad|Pruebas|Solución de problemas|
|---|---|---|
|Runbooks|Un runbook puede ejecutar recursos de Azure y conectarse a ellos correctamente.|[Solución de problemas de runbooks](../troubleshoot/runbooks.md)
|Control de código fuente|Puede ejecutar una sincronización manual en el repositorio de control de código fuente.|[Integración de control de código fuente](../source-control-integration.md)|
|Change Tracking e Inventario|Compruebe que puede ver los datos del inventario actual en las máquinas.|[Solución de problemas de Change Tracking](../troubleshoot/change-tracking.md)|
|Administración de actualizaciones|Asegúrese de que ve las máquinas y están en buen estado.</br>Ejecute una implementación de actualizaciones de software de prueba.|[Solución de problemas de Update management](../troubleshoot/update-management.md)|
|Recursos compartidos|Compruebe que ve todos los recursos compartidos, como [credenciales](../shared-resources/credentials.md) y [variables](../shared-resources/variables.md).|

## <a name="next-steps"></a>Pasos siguientes

Para información sobre el traslado de recursos en Azure, consulte [Traslado de recursos de Azure](../../azure-resource-manager/management/move-support-resources.md).
