---
title: Introducción a la característica Start/Stop VMs during off-hours de Azure Automation
description: En este artículo se describe cómo quitar la característica Start/Stop VMs during off-hours y desvincular una cuenta de Automation desde el área de trabajo de Log Analytics.
services: automation
ms.subservice: process-automation
ms.date: 04/15/2021
ms.topic: conceptual
ms.openlocfilehash: 9ec76197bfde6bb679f70c44ab01712f9f52bfd2
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533957"
---
# <a name="remove-startstop-vms-during-off-hours-from-automation-account"></a>Eliminación de la característica Start/Stop VMs during off-hours de la cuenta de Automation

Después de habilitar la característica Start/Stop VMs during off-hours para administrar el estado de ejecución de las máquinas virtuales de Azure, puede decidir dejar de usarla. La eliminación de esta característica puede realizarse mediante uno de los métodos siguientes en función de los modelos de implementación admitidos:

* Elimine el grupo de recursos que contiene la cuenta de Automation y el área de trabajo de Log Analytics vinculada de Azure Monitor, ambos dedicados a admitir esta característica.
* Desvincule el área de trabajo de Log Analytics de la cuenta de Automation y elimine la cuenta de Automation dedicada para esta característica.
* Elimine la característica de una cuenta de Automation y un área de trabajo vinculada que admitan otros objetivos de administración y supervisión.

Al eliminar esta característica, solo se quitan los runbooks asociados, pero no se eliminan las programaciones ni las variables que se crearon durante la implementación o las definidas de manera personalizada que se crearon posteriormente.

> [!NOTE]
> Antes de continuar, compruebe que no hay ningún [bloqueo de Resource Manager](../azure-resource-manager/management/lock-resources.md) aplicado a la suscripción, el grupo de recursos o el recurso, que impida la eliminación o la modificación accidentales de recursos críticos. Al implementar la solución Start/Stop VMs during off-hours, se establece el nivel de bloqueo en **CanNotDelete** en varios recursos dependientes de la cuenta de Automation (específicamente sus runbooks y variables). Los bloqueos deben quitarse para poder eliminar la cuenta de Automation.

## <a name="delete-the-dedicated-resource-group"></a>Eliminación del grupo de recursos dedicado

Para eliminar el grupo de recursos creado, siga los pasos descritos en el artículo [Eliminación de grupos de recursos y recursos en Azure Resource Manager](../azure-resource-manager/management/delete-resource-group.md).

## <a name="delete-the-automation-account"></a>Eliminación de la cuenta de Automation

Para eliminar la cuenta de Automation dedicada a Start/Stop VMs during off-hours, realice los pasos siguientes.

1. Inicie sesión en Azure en [https://portal.azure.com](https://portal.azure.com).

2. Vaya a la cuenta de Automation y seleccione **Área de trabajo vinculada** en **Recursos relacionados**.

3. Seleccione **Ir al área de trabajo**.

4. Haga clic en **Soluciones** en **General**.

5. En la página Soluciones, seleccione **Start-Stop-VM[workspace]** .

6. En la página **VMManagementSolution[Workspace]** , en el menú, seleccione **Eliminar**.

7. Mientras se comprueba la información y se elimina la característica, puede hacer un seguimiento del progreso en **Notificaciones**, una opción del menú. Después del proceso de eliminación, se le devuelve a la página Soluciones.

### <a name="unlink-workspace-from-automation-account"></a>Desvinculación de un área de trabajo de una cuenta de Automation

Hay dos opciones para desvincular el área de trabajo de Log Analytics de la cuenta de Automation. Puede realizar este proceso desde la cuenta de Automation o desde el área de trabajo vinculada.

Para desvincularla desde la cuenta de Automation, realice los pasos siguientes.

1. En Azure Portal, seleccione **Cuentas de Automation**.

2. Abra la cuenta de Automation y seleccione **Área de trabajo vinculada** en **Recursos relacionados** a la izquierda.

3. En la página **Desvincular área de trabajo**, seleccione **Desvincular área de trabajo** y responda a los mensajes.

   ![Página Desvincular área de trabajo](media/automation-solution-vm-management-remove/automation-unlink-workspace-blade.png)

    Mientras intenta desvincular el área de trabajo de Log Analytics, puede realizar el seguimiento del progreso en la opción **Notificaciones** del menú.

Para desvincularla desde el área de trabajo, realice los pasos siguientes.

1. En Azure Portal, seleccione **Áreas de trabajo de Log Analytics**.

2. En el área de trabajo, seleccione **Cuenta de Automation** en **Recursos relacionados**.

3. En la página Cuenta de Automation, seleccione **Desvincular cuenta** y responda los mensajes.

Mientras intenta desvincular la cuenta de Automation, puede seguir el progreso en **Notificaciones** desde el menú.

### <a name="delete-automation-account"></a>Eliminación de una cuenta de Automation

1. En Azure Portal, seleccione **Cuentas de Automation**.

2. Abra su cuenta de Automation y seleccione **Eliminar** en el menú.

Mientras se comprueba la información y se elimina la cuenta, puede hacer un seguimiento del progreso en **Notificaciones**, una opción del menú.

## <a name="delete-the-feature"></a>Eliminación de la característica

Para eliminar la característica Start/Stop VMs during off-hours de la cuenta de Automation, realice los pasos siguientes. En este proceso no se elimina la cuenta de Automation ni el área de trabajo de Log Analytics. Si no desea conservar el área de trabajo Log Analytics, debe eliminarla manualmente. Para obtener más información sobre cómo eliminar el área de trabajo, consulte [Eliminación y recuperación de un área de trabajo de Azure Log Analytics](../azure-monitor/logs/delete-workspace.md).

1. Vaya a la cuenta de Automation y seleccione **Área de trabajo vinculada** en **Recursos relacionados**.

2. Seleccione **Ir al área de trabajo**.

3. Haga clic en **Soluciones** en **General**.

4. En la página Soluciones, seleccione **Start-Stop-VM[workspace]** .

5. En la página **VMManagementSolution[Workspace]** , en el menú, seleccione **Eliminar**.

    ![Eliminación de la característica de administración de máquinas virtuales](media/automation-solution-vm-management/vm-management-solution-delete.png)

6. En la ventana Eliminar solución, confirme que quiere eliminar la característica.

7. Mientras se comprueba la información y se elimina la característica, puede hacer un seguimiento del progreso en **Notificaciones**, una opción del menú. Después del proceso de eliminación, se le devuelve a la página Soluciones.

8. Si no desea mantener los [recursos](automation-solution-vm-management.md#components) creados por la característica o que ha creado posteriormente (por ejemplo, variables, programaciones, etc.), tendrá que eliminarlos manualmente de la cuenta.

## <a name="next-steps"></a>Pasos siguientes

Para volver a habilitar esta característica, consulte [Habilitación de Start/Stop VMs during off-hours](automation-solution-vm-management-enable.md).