---
title: Limitación del ámbito de implementación de Update Management de Azure Automation
description: En este artículo se indica cómo usar las configuraciones de ámbito para limitar el ámbito de implementación de Update Management.
services: automation
ms.date: 06/03/2021
ms.topic: conceptual
ms.custom: mvc
ms.openlocfilehash: 230be9fcdb30e51802a2e288cde86fd8047f70f0
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111854961"
---
# <a name="limit-update-management-deployment-scope"></a>Limitación del ámbito de implementación de Update Management

En este artículo se describe cómo trabajar con configuraciones de ámbito cuando se utiliza la característica [Update Management](overview.md) para implementar actualizaciones y revisiones en las máquinas. Para obtener más información, consulte [Soluciones de supervisión como destino en Azure Monitor (versión preliminar)](../../azure-monitor/insights/solution-targeting.md).

## <a name="about-scope-configurations"></a>Acerca de las configuraciones de ámbito

Una configuración de ámbito es un grupo de una o varias búsquedas (consultas) utilizadas para limitar el ámbito de Update Management a equipos concretos. La configuración de ámbito se usa dentro del área de trabajo de Log Analytics para seleccionar como destino los equipos que se van a habilitar. Cuando agrega un equipo para recibir actualizaciones de Update Management, el equipo también se agrega a una búsqueda guardada en el área de trabajo. 

De forma predeterminada, Update Management crea un grupo de equipos llamado **Updates__MicrosoftDefaultComputerGroup** en función de cómo se habiliten las máquinas con Update Management:

* En la cuenta de Automation, ha seleccionado **+ Agregar máquinas virtuales de Azure**.
* En la cuenta de Automation, ha seleccionado **Administrar máquinas** y, luego, ha elegido la opción **Habilitar en todas las máquinas disponibles** o ha selecionado **Habilitar en las máquinas seleccionadas**.

Si se selecciona uno de los métodos anteriores, este grupo de equipos se agrega a la configuración del ámbito **MicrosoftDefaultScopeConfig-Updates**. También puede agregar uno o varios grupos de equipos personalizados a este ámbito para satisfacer sus necesidades de administración y controlar cómo se habilitan equipos específicos para la administración con Update Management.

Para quitar una o varias máquinas de **Updates__MicrosoftDefaultComputerGroup** para dejar de administrarlas con Update Management. Consulte [Eliminación de máquinas virtuales de Update Management](remove-vms.md).

## <a name="set-the-scope-limit"></a>Definición del límite de ámbito

Para limitar el ámbito de implementación de Update Management:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. En Azure Portal, vaya a **Areas de trabajo de Log Analytics**. Seleccione el área de trabajo de la lista.

3. En el área de trabajo de Log Analytics, seleccione **Configuraciones de ámbito (versión preliminar)** en el menú izquierdo.

4. Seleccione los puntos suspensivos a la derecha de la configuración de ámbito **MicrosoftDefaultScopeConfig-Updates** y elija **Editar**.

5. En el panel de edición, expanda **Seleccionar grupos de equipos**. En el panel **Grupos de equipos** se muestran las búsquedas guardadas que se agregan a la configuración de ámbito. La búsqueda guardada que usa Update Management es la siguiente:

    |Nombre     |Category  |Alias  |
    |---------|---------|---------|
    |MicrosoftDefaultComputerGroup     | Actualizaciones        | Updates__MicrosoftDefaultComputerGroup         |

6. Si ha agregado un grupo personalizado, se muestra en la lista. Para anular la selección, desactive la casilla situada a la izquierda del elemento. Para agregar un grupo personalizado al ámbito, selecciónelo y, cuando haya terminado con los cambios, haga clic en **Seleccionar**.

7. En la página **Editar configuración de ámbito**, haga clic en **Aceptar** para guardar los cambios.

## <a name="next-steps"></a>Pasos siguientes

Puede [consultar los registros de Azure Monitor](query-logs.md) para analizar las evaluaciones de las actualizaciones, las implementaciones y otras tareas de administración relacionadas. Incluye consultas predefinidas para ayudarle a empezar.
