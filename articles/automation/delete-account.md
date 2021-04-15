---
title: Eliminación de la cuenta de Azure Automation
description: En este artículo se explica cómo eliminar la cuenta de Automation en diferentes escenarios de configuración.
services: automation
ms.service: automation
ms.subservice: process-automation
ms.date: 03/18/2021
ms.topic: conceptual
ms.openlocfilehash: c3a514aa507fcf069671f987e175b7ae5be59d10
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105734920"
---
# <a name="how-to-delete-your-azure-automation-account"></a>Eliminación de la cuenta de Azure Automation

Después de habilitar una cuenta de Azure Automation para ayudarle a automatizar el proceso de TI o de negocio, o habilitar sus otras características para que admitan la administración de operaciones de las máquinas de Azure y que no son de Azure, como Update Management, puede decidir dejar de usar la cuenta de Automation. Si ha habilitado características que dependen de la integración con un área de trabajo de Log Analytics de Azure Monitor, es necesario realizar más pasos para completar esta acción.

La eliminación de la cuenta de Automation puede realizarse mediante uno de los métodos siguientes, en función de los modelos de implementación admitidos:

* Elimine el grupo de recursos que contiene la cuenta de Automation.
* Elimine el grupo de recursos que contiene la cuenta de Automation y el área de trabajo de Log Analytics de Azure Monitor vinculada, en los siguientes casos:

    * La cuenta y el área de trabajo están dedicadas a las características Update Management, Seguimiento de cambios e inventario o Start/Stop VMs during off-hours.
    * La cuenta está dedicada a la automatización de los procesos y se integra con un área de trabajo para enviar el estado y las transmisiones de trabajos del runbook.

* Desvincule el área de trabajo de Log Analytics de la cuenta de Automation y elimine la cuenta de Automation.
* Elimine la característica del área de trabajo vinculada, desvincule la cuenta del área de trabajo y, luego, elimine la cuenta de Automation.

En este artículo se explica cómo quitar completamente la cuenta de Automation mediante Azure Portal, PowerShell, la CLI de Azure o la API REST.

## <a name="delete-the-dedicated-resource-group"></a>Eliminación del grupo de recursos dedicado

Para eliminar la cuenta de Automation (y también el área de trabajo de Log Analytics si está vinculada a ella) creada en el mismo grupo de recursos dedicado a la cuenta, siga los pasos descritos en el artículo [Eliminación de grupos de recursos y recursos en Azure Resource Manager](../azure-resource-manager/management/delete-resource-group.md).

## <a name="delete-a-standalone-automation-account"></a>Eliminación de una cuenta de Automation independiente

Si la cuenta de Automation no está vinculada a un área de trabajo de Log Analytics, siga estos pasos para eliminarla.

# <a name="azure-portal"></a>[Azure Portal](#tab/azure-portal)

1. Inicie sesión en Azure en [https://portal.azure.com](https://portal.azure.com).

2. En Azure Portal, vaya a las **cuentas de Automation**.

3. Abra su cuenta de Automation y seleccione **Eliminar** en el menú.

Mientras se comprueba la información y se elimina la cuenta, puede hacer un seguimiento del progreso en **Notificaciones**, una opción del menú.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Este comando quita la cuenta de Automation sin solicitar validación.

```powershell
Remove-AzAutomationAccount -Name "automationAccountName" -Force -ResourceGroupName "resourceGroupName"
```

---

## <a name="delete-a-standalone-automation-account-linked-to-workspace"></a>Eliminación de una cuenta de Automation independiente vinculada al área de trabajo

Si la cuenta de Automation está vinculada a un área de trabajo de Log Analytics para recopilar transmisiones y registros de trabajos, realice los pasos siguientes para eliminarla.

Hay dos opciones para desvincular el área de trabajo de Log Analytics de la cuenta de Automation. Puede realizar este proceso desde la cuenta de Automation o desde el área de trabajo vinculada.

Para desvincularla desde la cuenta de Automation, realice los pasos siguientes.

1. En Azure Portal, vaya a las **cuentas de Automation**.

2. Abra la cuenta de Automation y seleccione **Área de trabajo vinculada** en **Recursos relacionados** a la izquierda.

3. En la página **Desvincular área de trabajo**, seleccione **Desvincular área de trabajo** y responda a los avisos.

   ![Página Desvincular área de trabajo](media/automation-solution-vm-management-remove/automation-unlink-workspace-blade.png)

    Mientras intenta desvincular el área de trabajo de Log Analytics, puede realizar el seguimiento del progreso en la opción **Notificaciones** del menú.

Para desvincularla desde el área de trabajo, realice los pasos siguientes.

1. En Azure Portal, vaya a **Areas de trabajo de Log Analytics**.

2. En el área de trabajo, seleccione **Cuenta de Automation** en **Recursos relacionados**.

3. En la página Cuenta de Automation, seleccione **Desvincular cuenta** y responda a los avisos.

Mientras intenta desvincular la cuenta de Automation, puede seguir el progreso en **Notificaciones** desde el menú.

Después de que la cuenta de Automation se haya desvinculado correctamente del área de trabajo, siga los pasos de la sección [Cuenta de Automation independiente](#delete-a-standalone-automation-account) para eliminar la cuenta.

## <a name="delete-a-shared-capability-automation-account"></a>Eliminación de una cuenta de Automation de capacidad compartida

Para eliminar la cuenta de Automation vinculada a un área de trabajo de Log Analytics para Update Management, Seguimiento de cambios e inventario o Start/Stop VMs during off-hours, realice los pasos siguientes.

### <a name="step-1-delete-the-solution-from-the-linked-workspace"></a>Paso 1. Eliminación de la solución del área de trabajo vinculada

# <a name="azure-portal"></a>[Azure Portal](#tab/azure-portal)

1. Inicie sesión en Azure en [https://portal.azure.com](https://portal.azure.com).

2. Vaya a la cuenta de Automation y seleccione **Área de trabajo vinculada** en **Recursos relacionados**.

3. Seleccione **Ir al área de trabajo**.

4. Haga clic en **Soluciones** en **General**.

5. En la página Soluciones, seleccione una de las siguientes opciones en función de las características implementadas en la cuenta:

    * En Start/Stop VMs during off-hours, seleccione **Start-Stop-VM [nombre del área de trabajo]** .
    * En Update Management, seleccione **Actualizaciones (nombre del área de trabajo)** .
    * En Seguimiento de cambios e inventario, seleccione **ChangeTracking (nombre del área de trabajo)** .

6. En la página **Solución**, seleccione **Eliminar** en el menú. Si hay implementada más de una de las características enumeradas anteriormente en la cuenta de Automation y el área de trabajo vinculada, deberá seleccionar y eliminar cada una de ellas antes de continuar.

7. Mientras se comprueba la información y se elimina la característica, puede hacer un seguimiento del progreso en **Notificaciones**, una opción del menú. Después del proceso de eliminación, se le devuelve a la página Soluciones.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para quitar una solución instalada con Azure PowerShell, use el cmdlet [Remove-AzMonitorLogAnalyticsSolution](/powershell/module/az.monitoringsolutions/remove-azmonitorloganalyticssolution).

```powershell
Remove-AzMonitorLogAnalyticsSolution -ResourceGroupName "resourceGroupName" -Name "solutionName"
```

---

### <a name="step-2-unlink-workspace-from-automation-account"></a>Paso 2. Desvinculación de un área de trabajo de una cuenta de Automation

Hay dos opciones para desvincular el área de trabajo de Log Analytics de la cuenta de Automation. Puede realizar este proceso desde la cuenta de Automation o desde el área de trabajo vinculada.

Para desvincularla desde la cuenta de Automation, realice los pasos siguientes.

1. En Azure Portal, vaya a las **cuentas de Automation**.

2. Abra la cuenta de Automation y seleccione **Área de trabajo vinculada** en **Recursos relacionados** a la izquierda.

3. En la página **Desvincular área de trabajo**, seleccione **Desvincular área de trabajo** y responda a los avisos.

   ![Página Desvincular área de trabajo](media/automation-solution-vm-management-remove/automation-unlink-workspace-blade.png)

    Mientras intenta desvincular el área de trabajo de Log Analytics, puede realizar el seguimiento del progreso en la opción **Notificaciones** del menú.

Para desvincularla desde el área de trabajo, realice los pasos siguientes.

1. En Azure Portal, vaya a **Areas de trabajo de Log Analytics**.

2. En el área de trabajo, seleccione **Cuenta de Automation** en **Recursos relacionados**.

3. En la página Cuenta de Automation, seleccione **Desvincular cuenta** y responda a los avisos.

Mientras intenta desvincular la cuenta de Automation, puede seguir el progreso en **Notificaciones** desde el menú.

### <a name="step-3-delete-automation-account"></a>Paso 3. Eliminación de una cuenta de Automation

Después de que la cuenta de Automation se haya desvinculado correctamente del área de trabajo, siga los pasos de la sección [Cuenta de Automation independiente](#delete-a-standalone-automation-account) para eliminar la cuenta.

## <a name="next-steps"></a>Pasos siguientes

Para crear una cuenta de Automation desde Azure Portal, consulte [Creación de una cuenta independiente de Azure Automation](automation-create-standalone-account.md). Si prefiere crear su cuenta mediante una plantilla, consulte [Creación de una cuenta de Automation con una plantilla de Azure Resource Manager](quickstart-create-automation-account-template.md).