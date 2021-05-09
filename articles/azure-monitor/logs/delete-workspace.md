---
title: Eliminación y recuperación de un área de trabajo de Azure Log Analytics | Microsoft Docs
description: Obtenga información para eliminar el área de trabajo de Log Analytics si creó uno en una suscripción personal o reestructure el modelo de área de trabajo.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 12/20/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9c34dc77a7fbee34b454cdc6ded24a5a17e890f5
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108321184"
---
# <a name="delete-and-recover-azure-log-analytics-workspace"></a>Eliminación y recuperación de un área de trabajo de Azure Log Analytics

En este artículo se explica el concepto de eliminación temporal de áreas de trabajo de Azure Log Analytics y cómo recuperarlas.

## <a name="considerations-when-deleting-a-workspace"></a>Consideraciones al eliminar áreas de trabajo

Al eliminar un área de trabajo de Log Analytics, se realiza una operación de eliminación temporal para que se pueda recuperar el área de trabajo, incluidos los datos y los agentes conectados en un plazo de 14 días, tanto si la eliminación fue accidental como intencionada. Después del período de eliminación temporal, el recurso del área de trabajo y sus datos no podrán recuperarse y se pondrán en cola para purgarse completamente en un plazo de 30 días. El nombre del área de trabajo es "released" y puede usarlo para crear una nueva área de trabajo.

> [!NOTE]
> Si desea invalidar el comportamiento de eliminación temporal y eliminar el área de trabajo de forma permanente, siga los pasos descritos en [Eliminación permanente del área de trabajo](#permanent-workspace-delete).

Debe tener cuidado al eliminar un área de trabajo porque puede haber datos importantes y una configuración que puede afectar negativamente a las operaciones de servicio. Revise qué agentes, soluciones y otros servicios de Azure almacenan sus datos en Log Analytics. Por ejemplo:

* Soluciones de administración
* Azure Automation
* Agentes que se ejecutan en máquinas virtuales Windows y Linux
* Agentes que se ejecutan en equipos Windows y Linux en su entorno
* System Center Operations Manager

La operación de eliminación temporal quita el recurso del área de trabajo y los permisos de los usuarios asociados. Si los usuarios están asociados a otras áreas de trabajo, podrán seguir utilizando Log Analytics con esas otras áreas de trabajo.

## <a name="soft-delete-behavior"></a>Comportamiento de eliminación temporal

La operación de eliminación del área de trabajo quita el recurso de Resource Manager del área de trabajo, pero su configuración y sus datos se conservan durante 14 días (y da la sensación de que el área de trabajo se ha eliminado). Los agentes y grupos de administración de System Center Operations Manager configurados para enviar notificaciones al área de trabajo continúan en un estado huérfano durante el período de eliminación temporal. El servicio proporciona además un mecanismo para recuperar el área de trabajo eliminada (incluidos sus datos y recursos conectados), básicamente, deshaciendo la eliminación.

> [!NOTE] 
> Tanto las soluciones instaladas como los servicios vinculados, como la cuenta de Azure Automation, se quitan permanentemente del área de trabajo en el momento de la eliminación y no se pueden recuperar. Deben volver a configurarse después de la operación de recuperación para que el área de trabajo vuelva al estado en que estaba configurado anteriormente.

Las áreas de trabajo se pueden eliminar un mediante [PowerShell](/powershell/module/azurerm.operationalinsights/remove-azurermoperationalinsightsworkspace), [API REST](/rest/api/loganalytics/workspaces/delete) o en [Azure Portal](https://portal.azure.com).

### <a name="azure-portal"></a>Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
2. En Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba **Log Analytics**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Áreas de trabajo de Log Analytics**.
3. En la lista de áreas de trabajo de Log Analytics, seleccione un área de trabajo y, a continuación, haga clic en **Eliminar** en la parte superior del panel intermedio.
4. Aparece una página de confirmación que muestra la ingesta de datos en el área de trabajo durante la semana pasada. 
5. Si quiere eliminar permanentemente el área de trabajo al quitar la opción para recuperarla más tarde, active la casilla **Delete the workspace permanently** (Eliminar el área de trabajo de forma permanente).
6. Escriba el nombre del área de trabajo que desea confirmar y, a continuación, haga clic en **Eliminar**.

   ![Confirmación de la eliminación del área de trabajo](media/delete-workspace/workspace-delete.png)

### <a name="powershell"></a>PowerShell
```PowerShell
PS C:\>Remove-AzOperationalInsightsWorkspace -ResourceGroupName "resource-group-name" -Name "workspace-name"
```

## <a name="permanent-workspace-delete"></a>Eliminación permanente del área de trabajo
Es posible que el método de eliminación temporal no encaje en algunos escenarios como el desarrollo y las pruebas, donde es necesario repetir una implementación con la misma configuración y el mismo nombre de área de trabajo. En tales casos, puede eliminar el área de trabajo de forma permanente e "invalidar" el período de eliminación temporal. La operación de eliminación permanente del área de trabajo libera el nombre del área de trabajo y puede crear una nueva área de trabajo con el mismo nombre.

> [!IMPORTANT]
> Use la operación de eliminación permanente de áreas de trabajo con precaución porque es irreversible y no podrá recuperar el área de trabajo ni sus datos.

Para eliminar de manera permanente un área de trabajo mediante Azure Portal, active la casilla **Delete the workspace permanently** (Eliminar el área de trabajo de manera permanente) antes de hacer clic en el botón **Eliminar**.

Para eliminar de manera permanente un área de trabajo con PowerShell, agregue la etiqueta "-ForceDelete" para eliminar el área de trabajo de manera permanente. La opción "-ForceDelete" está disponible actualmente con Az.OperationalInsights 2.3.0 o versiones posteriores. 

```powershell
PS C:\>Remove-AzOperationalInsightsWorkspace -ResourceGroupName "resource-group-name" -Name "workspace-name" -ForceDelete
```

## <a name="recover-workspace"></a>Recuperación de áreas de trabajo
Al eliminar un área de trabajo de Log Analytics accidental o intencionadamente, el servicio coloca el área de trabajo en un estado de eliminación temporal, lo que hace que sea inaccesible para cualquier operación. El nombre del área de trabajo eliminada se conserva durante el periodo de eliminación temporal y no se puede para crear un área de trabajo nueva. Después del período de eliminación temporal, el área de trabajo no es recuperable, está programada para su eliminación permanente y su nombre se ha liberado y se puede usar para crear una nueva área de trabajo.

Puede recuperar el área de trabajo durante el período de eliminación temporal, incluidos los datos, la configuración y los agentes conectados. Necesita tener permisos de colaborador en la suscripción y el grupo de recursos donde se localizó el área de trabajo antes de la operación de eliminación temporal. La recuperación del área de trabajo se realiza al volver a crear el área de trabajo de Log Analytics con los detalles del área de trabajo eliminada, incluidos los siguientes:

- Id. de suscripción
- Nombre del grupo de recursos
- Nombre del área de trabajo
- Region

> [!IMPORTANT]
> Si el área de trabajo se eliminó como parte de la operación de eliminación del grupo de recursos, primero debe volver a crear el grupo de recursos.

### <a name="azure-portal"></a>Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
2. En Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba **Log Analytics**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Áreas de trabajo de Log Analytics**. Verá la lista de áreas de trabajo que tiene en el ámbito seleccionado.
3. Haga clic en **Recuperar** en el menú superior izquierdo para abrir una página con áreas de trabajo en estado de eliminación temporal que se puedan recuperar.

   ![Captura de la pantalla de áreas de trabajo de Log Analytics en Azure Portal con la opción Recuperar resaltada en la barra de menús.](media/delete-workspace/recover-menu.png)

4. Seleccione el área de trabajo y haga clic en **Recuperar** para recuperar dicha área de trabajo.

   ![Captura de pantalla del cuadro de diálogo Recuperar áreas de trabajo de Log Analytics eliminadas en Azure Portal con un área de trabajo resaltada y el botón Recuperar seleccionado.](media/delete-workspace/recover-workspace.png)


### <a name="powershell"></a>PowerShell
```PowerShell
PS C:\>Select-AzSubscription "subscription-name-the-workspace-was-in"
PS C:\>New-AzOperationalInsightsWorkspace -ResourceGroupName "resource-group-name-the-workspace-was-in" -Name "deleted-workspace-name" -Location "region-name-the-workspace-was-in"
```

El área de trabajo y todos sus datos se restauran después de la operación de recuperación. Las soluciones y los servicios vinculados se quitaron permanentemente del área de trabajo cuando se eliminó y se deben volver a configurar para restaurarla a su estado configurado anterior. Puede que algunos de los datos no estén disponibles para la consulta después de la recuperación del área de trabajo hasta que se vuelvan a instalar las soluciones asociadas y se agreguen sus esquemas al área de trabajo.

## <a name="troubleshooting"></a>Solución de problemas

Debe tener al menos permisos de *Colaborador de Log Analytics* para eliminar un área de trabajo.

* Si no está seguro de si el área de trabajo eliminada está en estado de eliminación temporal y se puede recuperar, haga clic en [Abrir papelera de reciclaje](#recover-workspace) en la página *Áreas de trabajo de Log Analytics* para ver una lista de áreas de trabajo eliminadas temporalmente por suscripción. Las áreas de trabajo eliminadas permanentemente no se incluyen en la lista.
* Si recibe el mensaje de error *El nombre del área de trabajo ya está en uso* o se produce un *conflicto* al crear un área de trabajo, puede deberse a:
  * el nombre del área de trabajo no está disponible y lo está usando alguien de su organización u otro cliente.
  * El área de trabajo se ha eliminado en los últimos 14 días y su nombre se mantiene reservado durante el período de eliminación temporal. Para invalidar la eliminación temporal y eliminar inmediatamente el área de trabajo y crear una con el mismo nombre, siga estos pasos para recuperar el área de trabajo primero y realizar una eliminación permanente:<br>
    1. [Recuperar](#recover-workspace) el área de trabajo.
    2. [Eliminar permanentemente](#permanent-workspace-delete) el área de trabajo.
    3. Crear una nueva área de trabajo con el mismo nombre.
 
      Después de que la llamada de eliminación se haya completado correctamente en el back-end, puede restaurar el área de trabajo y completar la operación de eliminación permanente en uno de los métodos sugeridos anteriormente.

* Si recibe el código de respuesta 204 de *Recurso no encontrado* al eliminar un área de trabajo, es posible que se hayan producido operaciones de reintentos consecutivos. 204 es una respuesta vacía, lo que normalmente significa que el recurso no existe y que la eliminación se completó sin hacer nada.
* Si elimina el grupo de recursos y el área de trabajo incluida, puede ver el área de trabajo eliminada en [Abrir papelera de reciclaje](#recover-workspace), sin embargo, se producirá un error con el código de error 404 en la operación de recuperación dado que el grupo de recursos no existe. Vuelva a crear el grupo de recursos e intente la recuperación de nuevo.
