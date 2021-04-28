---
title: Tipos de runbooks de Azure Automation
description: En este artículo se describen los tipos de runbooks que puede usar en Azure Automation y las consideraciones de determinar qué tipo usar.
services: automation
ms.subservice: process-automation
ms.date: 02/17/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3e4f90372c2da22e8df3430ce340477352e5033b
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107830455"
---
# <a name="azure-automation-runbook-types"></a>Tipos de runbooks de Azure Automation

La característica de automatización de procesos de Azure Automation admite varios tipos de runbooks, tal como se define en la tabla siguiente. Para más información sobre el entorno de automatización de procesos, consulte [Ejecución de un runbook en Azure Automation](automation-runbook-execution.md).

| Tipo | Descripción |
|:--- |:--- |
| [Gráfico](#graphical-runbooks)|Runbook gráfico basado en Windows PowerShell, y creado y editado completamente en el editor gráfico de Azure Portal. |
| [Flujo de trabajo gráfico de PowerShell](#graphical-runbooks)|Runbook gráfico basado en flujo de trabajo de Windows PowerShell, y creado y editado completamente en el editor gráfico de Azure Portal. |
| [PowerShell](#powershell-runbooks) |Runbook de texto basado en scripting de Windows PowerShell. |
| [Flujo de trabajo de PowerShell](#powershell-workflow-runbooks)|Runbook de texto basado en scripting de flujo de trabajo de Windows PowerShell. |
| [Python](#python-runbooks) |Runbook de texto basado en scripting de Python. |

Tenga en cuenta las siguientes consideraciones al determinar qué tipo usar para un runbook concreto.

* No es posible convertir runbooks de tipo gráfico a texto o viceversa.
* Existen limitaciones al utilizar runbooks de diferentes tipos como runbooks secundarios. Para más información, consulte [Child runbooks in Azure Automation](automation-child-runbooks.md) (Runbooks secundarios en Azure Automation).

## <a name="graphical-runbooks"></a>Runbooks gráficos

Puede crear y editar runbooks gráficos y runbooks gráficos de flujo de trabajo de PowerShell mediante el editor gráfico de Azure Portal. Sin embargo, no puede crear o editar este tipo de runbooks con otra herramienta. Principales características de los runbooks gráficos:

* Se exportan a archivos de la cuenta de Automation y se importan a otra cuenta de Automation.
* Generan código de PowerShell.
* Se pueden convertir a y desde runbooks gráficos de flujo de trabajo de PowerShell durante la importación.

### <a name="advantages"></a>Ventajas

* Utilice modelos de creación visual para insertar, vincular y configurar.
* Céntrese en cómo fluyen los datos por el proceso.
* Represente visualmente los procesos de administración.
* Incluya otros runbooks como runbooks secundarios para crear flujos de trabajo de nivel alto.
* Anime a usar la programación modular.

### <a name="limitations"></a>Limitaciones

* No puede crear o editar fuera de Azure Portal.
* Puede requerir una actividad de código que contenga código de PowerShell para ejecutar una lógica compleja.
* No puede convertir a uno de los [formatos de texto](automation-runbook-types.md), ni puede convertir un runbook de texto a un formato gráfico. 
* No puede ver ni modificar directamente el código de PowerShell que el flujo de trabajo gráfico crea. Puede ver el código que crea en las actividades de código.
* No puede ejecutar runbooks en Linux Runbook Worker. Consulte [Automatización de recursos en los centros de datos o nube con Hybrid Runbook Worker](automation-hybrid-runbook-worker.md).

## <a name="powershell-runbooks"></a>Runbooks de PowerShell

Los runbooks de PowerShell están basados en Windows PowerShell. Puede modificar directamente el código del runbook con el editor de texto en el Portal de Azure.  También puede usar cualquier editor de texto sin conexión e [importar el runbook](manage-runbooks.md) en Azure Automation.

### <a name="advantages"></a>Ventajas

* Implemente toda la lógica compleja con código de PowerShell sin otras complejidades del flujo de trabajo de PowerShell.
* Se inician con más rapidez que los runbooks de flujo de trabajo de PowerShell, ya que no necesitan compilarse antes de la ejecución.
* Ejecute en Azure y en Hybrid Runbook Worker para Windows y Linux.

### <a name="limitations"></a>Limitaciones

* Debe estar familiarizado con el scripting de PowerShell.
* Los runbooks no pueden usar el [procesamiento paralelo](automation-powershell-workflow.md#use-parallel-processing) para ejecutar varias acciones en paralelo.
* Los runbooks no pueden usar los [puntos de control](automation-powershell-workflow.md#use-checkpoints-in-a-workflow) para reanudar un runbook si se produce un error.
* Solo puede incluir los runbooks gráficos y de flujo de trabajo de PowerShell como runbooks secundarios mediante el cmdlet [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook), que crea un trabajo.

### <a name="known-issues"></a>Problemas conocidos

A continuación se describen los problemas conocidos actuales con los runbooks de PowerShell:

* Los runbooks de PowerShell no pueden recuperar un [recurso de variable](./shared-resources/variables.md) sin cifrar con un valor null.
* Los runbooks de PowerShell no pueden recuperar un recurso de variable con `*~*` en el nombre.
* Una operación [Get-Process](/powershell/module/microsoft.powershell.management/get-process) en un bucle de un runbook de PowerShell puede bloquearse después de más de 80 iteraciones.
* Un runbook de PowerShell puede producir un error si intenta escribir una gran cantidad de datos en el flujo de salida a la vez. Puede evitar este problema si hace que el runbook genere únicamente la información necesaria para trabajar con objetos grandes. Por ejemplo, en lugar de usar `Get-Process` sin limitaciones, puede hacer que el cmdlet genere solo los parámetros necesarios como en `Get-Process | Select ProcessName, CPU`.

## <a name="powershell-workflow-runbooks"></a>Runbooks del flujo de trabajo de PowerShell

Los runbooks del flujo de trabajo de PowerShell son runbooks de texto basados en el [flujo de trabajo de Windows PowerShell](automation-powershell-workflow.md). Puede modificar directamente el código del runbook con el editor de texto en el Portal de Azure. También puede usar cualquier editor de texto sin conexión e [importar el runbook](manage-runbooks.md) en Azure Automation.

### <a name="advantages"></a>Ventajas

* Implemente toda la lógica compleja con código del flujo de trabajo de PowerShell.
* Use los [puntos de control](automation-powershell-workflow.md#use-checkpoints-in-a-workflow) para reanudar la operación si se produce un error.
* Utilice el [procesamiento paralelo](automation-powershell-workflow.md#use-parallel-processing) para realizar varias acciones en paralelo.
* Se pueden incluir otros runbooks gráficos o de flujo de trabajo de PowerShell como runbooks secundarios para crear flujos de trabajo de alto nivel.

### <a name="limitations"></a>Limitaciones

* Debe estar familiarizado con el flujo de trabajo de PowerShell.
* Los runbooks deben tratar la complejidad adicional del flujo de trabajo de PowerShell como [objetos deserializados](automation-powershell-workflow.md#deserialized-objects).
* Los runbooks tardan más en iniciarse que los runbooks de PowerShell, ya que deben compilarse antes de su ejecución.
* Solo puede incluir runbooks de PowerShell como runbooks secundarios mediante el cmdlet `Start-AzAutomationRunbook`.
* Los runbooks no se pueden ejecutar en una instancia de Hybrid Runbook Worker de Linux.

## <a name="python-runbooks"></a>Runbooks de Python

Los runbooks de Python utilizan Python 2 y Python 3. Actualmente, los runbooks de Python 3 están en versión preliminar. Puede modificar directamente el código del runbook con el editor de texto en Azure Portal. También puede usar cualquier editor de texto sin conexión e [importar el runbook](manage-runbooks.md) en Azure Automation.

Los runbooks de Python 3 se admiten en las siguientes infraestructuras globales de Azure:

* Azure Global
* Azure Government

### <a name="advantages"></a>Ventajas

* Use las sólidas bibliotecas de Python.
* Se pueden ejecutar en Azure o en instancias de Hybrid Runbook Worker.
* En el caso de Python 2, las instancias de Hybrid Runbook Worker de Windows son compatibles si [Python2.7](https://www.python.org/downloads/release/latest/python2) está instalado.
* Para los trabajos en la nube de Python 3, se admite la versión 3.8 de Python. Los scripts y paquetes de cualquier versión 3.x podrían funcionar si el código es compatible con distintas versiones.  
* En el caso de los trabajos híbridos de Python 3 en máquinas Windows, puede instalar cualquier versión 3. x que desee usar.  
* Para los trabajos híbridos de Python 3 para Linux, depende de la versión de Python 3 instalada en el equipo para ejecutar DSC OMSConfig y Hybrid Worker para Linux. Se recomienda instalar la versión 3.6 en máquinas Linux. Sin embargo, otras versiones también deberían funcionar si no hay cambios importantes en los contratos o firmas de método entre versiones de Python 3.

### <a name="limitations"></a>Limitaciones

* Debe estar familiarizado con el scripting de Python.
* Para utilizar bibliotecas de terceros, debe [importar los paquetes](python-packages.md) a la cuenta de Automation.
* Usar el cmdlet **Start-AutomationRunbook** en PowerShell o en el flujo de trabajo de PowerShell para iniciar un runbook de Python 3 (versión preliminar) no funciona. Puede usar el cmdlet  **Start-AzAutomationRunbook** desde el nódulo Az.Automation o el cmdlet  **Start-AzureRmAutomationRunbook** del módulo AzureRm.Automation para solucionar esta limitación.  
* Los runbooks (versión preliminar) y los paquetes de Python 3 no funcionan con PowerShell.
* Azure Automation no admite  **sy.stderr**.

### <a name="known-issues"></a>Problemas conocidos

Los trabajos de Python 3 a veces generan un error con el mensaje de excepción para indicar que *la ruta de acceso ejecutable del intérprete no es válida*. Podría ver esta excepción si un trabajo se retrasa y empieza más de 10 minutos tarde o usa **Start-AutomationRunbook** para iniciar runbooks de Python 3. Si el trabajo se retrasa, reiniciar el runbook debería ser suficiente.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los runbooks de PowerShell, consulte [Tutorial: Creación de un runbook de PowerShell](learn/automation-tutorial-runbook-textual-powershell.md).
* Para mas información sobre los runbooks de flujo de trabajo de PowerShell, consulte [Tutorial: Creación de un runbook de flujo de trabajo de PowerShell](learn/automation-tutorial-runbook-textual.md).
* Para información sobre los runbooks gráficos, consulte [Tutorial: Creación de un runbook gráfico](learn/automation-tutorial-runbook-graphical.md).
* Para información sobre los runbooks de Python, consulte [Tutorial: Creación de un runbook de Python](learn/automation-tutorial-runbook-textual-python2.md).
