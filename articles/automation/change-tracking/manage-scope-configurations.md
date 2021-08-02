---
title: Limitación del ámbito de implementación de Change Tracking e Inventario de Azure Automation
description: En este artículo se explica cómo trabajar con las configuraciones de ámbito para limitar el ámbito de una implementación de Change Tracking e Inventario.
services: automation
ms.subservice: change-inventory-management
ms.date: 05/27/2021
ms.topic: conceptual
ms.openlocfilehash: 6812a0b0688efdb75d847a36d661ba87017a8b9d
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110653201"
---
# <a name="limit-change-tracking-and-inventory-deployment-scope"></a>Limitación del ámbito de implementación de Change Tracking e Inventario

En este artículo se describe cómo trabajar con configuraciones de ámbito al usar la característica [Change Tracking e Inventario](overview.md) para implementar cambios en las VM. Para obtener más información, consulte [Soluciones de supervisión como destino en Azure Monitor (versión preliminar)](../../azure-monitor/insights/solution-targeting.md).

## <a name="about-scope-configurations"></a>Acerca de las configuraciones de ámbito

Una configuración de ámbito es un grupo de una o varias búsquedas (consultas) guardadas que se usa para limitar el ámbito de Change Tracking e Inventario a equipos concretos. La configuración de ámbito se usa dentro del área de trabajo de Log Analytics para seleccionar como destino los equipos que se van a habilitar. Cuando agrega un equipo a los cambios de la característica, el equipo también se agrega a una búsqueda guardada en el área de trabajo.

De forma predeterminada, Seguimiento de cambios e inventario crea un grupo de equipos llamado **ChangeTracking__MicrosoftDefaultComputerGroup** en función de cómo se habiliten las máquinas:

* En la cuenta de Automation, ha seleccionado **+ Agregar máquinas virtuales de Azure**.
* En la cuenta de Automation, ha seleccionado **Administrar máquinas** y, luego, ha elegido la opción **Habilitar en todas las máquinas disponibles** o ha seleccionado **Habilitar en las máquinas seleccionadas**.

Si se selecciona uno de los métodos anteriores, este grupo de equipos se agrega a la configuración del ámbito **MicrosoftDefaultScopeConfig-ChangeTracking**. También puede agregar uno o varios grupos de equipos personalizados a este ámbito para satisfacer sus necesidades de administración y controlar cómo se habilitan equipos específicos para la administración con Seguimiento de cambios e inventario.

Para quitar una o varias máquinas de **ChangeTracking__MicrosoftDefaultComputerGroup** y dejar de administrarlas con Seguimiento de cambios e inventario, consulte [Eliminación de máquinas virtuales de Seguimiento de cambios e inventario](remove-vms-from-change-tracking.md).

## <a name="set-the-scope-limit"></a>Definición del límite de ámbito

Para limitar el ámbito de la implementación de Change Tracking e Inventario:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. En Azure Portal, vaya a **Areas de trabajo de Log Analytics**. Seleccione el área de trabajo de la lista.

3. En el área de trabajo de Log Analytics, seleccione **Configuraciones de ámbito (versión preliminar)** en el menú izquierdo.

4. Seleccione los puntos suspensivos a la derecha de la configuración de ámbito **MicrosoftDefaultScopeConfig-ChangeTracking** y elija **Editar**.

5. En el panel de edición, expanda **Seleccionar grupos de equipos**. En el panel **Grupos de equipos** se muestran las búsquedas guardadas que se agregan a la configuración de ámbito. La búsqueda guardada que usa Update Management es la siguiente:

    |Nombre     |Category  |Alias  |
    |---------|---------|---------|
    |MicrosoftDefaultComputerGroup     | ChangeTracking        | ChangeTracking__MicrosoftDefaultComputerGroup         |

6. Si ha agregado un grupo personalizado, se muestra en la lista. Para anular la selección, desactive la casilla situada a la izquierda del elemento. Para agregar un grupo personalizado al ámbito, selecciónelo y, cuando haya terminado con los cambios, haga clic en **Seleccionar**.

7. En la página **Editar configuración de ámbito**, haga clic en **Aceptar** para guardar los cambios.

## <a name="next-steps"></a>Pasos siguientes

* Para trabajar con Change Tracking e Inventario, vea [Administración de Change Tracking e Inventario](manage-change-tracking.md).
* Para solucionar problemas generales de la característica, vea [Solución de problemas de Change Tracking e Inventario](../troubleshoot/change-tracking.md).
