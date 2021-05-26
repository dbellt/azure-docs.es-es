---
title: Solución de problemas de Start/Stop VMs v2 (versión preliminar)
description: En este artículo se explica cómo solucionar los problemas encontrados con la función Start/Stop VMs (versión preliminar) para las máquinas virtuales de Azure.
services: azure-functions
ms.subservice: start-stop-vms
ms.date: 03/31/2021
ms.topic: conceptual
ms.openlocfilehash: 83633b6f107aee995c19e1cda94f4f93a96e84e2
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110076787"
---
# <a name="troubleshoot-common-issues-with-startstop-vms-preview"></a>Solución de problemas comunes con Start/Stop VMs (versión preliminar)

En este artículo se proporciona información sobre la solución de problemas que pueden producirse al intentar instalar y configurar Start/Stop VMs (versión preliminar). Para obtener información general, consulte [Introducción a Start/Stop VMs](overview.md).

## <a name="general-validation-and-troubleshooting"></a>Validación y solución de problemas general

En esta sección se explica cómo solucionar problemas generales de los escenarios de programación y cómo ayudar a identificar la causa principal.

### <a name="azure-dashboard"></a>Panel de Azure

Puede empezar por revisar el panel compartido de Azure. El panel compartido de Azure implementado como parte de Start/Stop VMs v2 (versión preliminar) es una forma rápida y sencilla de comprobar el estado de cada operación que se realiza en las máquinas virtuales. Consulte el icono de las **acciones intentadas recientemente en máquinas virtuales** para ver todas las operaciones recientes ejecutadas en las máquinas virtuales. Hay cierta latencia, en torno a cinco minutos, para que los datos se muestren en el informe mientras extraen los datos del recurso de Application Insights.

### <a name="logic-apps"></a>Logic Apps

En función de qué aplicación lógica haya habilitado para admitir el escenario de inicio o detención, puede revisar el historial de ejecución para ayudar a identificar por qué el escenario de inicio o apagado programado no se completó correctamente para una o varias máquinas virtuales de destino. Para más información sobre cómo revisar esto en detalle, consulte [Revisión del historial de ejecuciones](../../logic-apps/monitor-logic-apps.md#review-runs-history).

### <a name="azure-storage"></a>Azure Storage

Puede revisar los detalles de las operaciones realizadas en las máquinas virtuales que se escriben en la tabla **requestsstoretable** de la cuenta de Azure Storage que se usa para Start/Stop VMs v2 (versión preliminar). Realice los pasos siguientes para ver esos registros.

1. Vaya a la cuenta de almacenamiento en Azure Portal y, en la cuenta, seleccione **Explorador de Storage (versión preliminar) en el panel izquierdo.
1. Seleccione **TABLAS** y, después, **requeststoretable**.
1. Cada registro de la tabla representa la acción de inicio y detención realizada en una máquina virtual de Azure basada en el ámbito de destino definido en el escenario de la aplicación lógica. Puede filtrar los resultados por cualquiera de las propiedades del registro (por ejemplo, TIMESTAMP, ACTION o TARGETTOPLEVELRESOURCENAME).

### <a name="azure-functions"></a>Azure Functions

Puede revisar los detalles de la invocación más reciente para cualquiera de las instancias de Azure Functions responsables de la ejecución de la máquina virtual y de la detención de la ejecución. Primero, vamos a revisar el flujo de ejecución.

El flujo de ejecución para el escenario **programado** y **secuenciado** se controla mediante la misma función. El esquema de carga es lo que determina qué escenario se realiza. Para el escenario **Programado**, el flujo de ejecución es - HTTP **programada** > cola **VirtualMachineRequestOrchestrator** > cola **VirtualMachineRequestExecutor**.

Desde la aplicación lógica, la función HTTP **programada** se invoca con el esquema de carga. Cuando la función HTTP **programada** recibe la solicitud, envía la información a la función de cola **Orchestrator**, que a su vez crea varias colas para que cada máquina virtual realice la acción.

Lleve a cabo los pasos siguientes para ver los detalles de la invocación.

1. En Azure Portal, vaya a **Azure Functions**.
1. Seleccione la aplicación de funciones para Start/Stop VMs v2 (versión preliminar) en la lista.
1. Seleccione **Funciones** en el panel izquierdo.
1. En la lista, se ven varias funciones asociadas a cada escenario. Seleccione la función HTTP **programada**.
1. Seleccione **Supervisar** en el panel izquierdo.
1. Seleccione el seguimiento de ejecución más reciente para ver los detalles de la invocación y la sección del mensaje para el registro detallado.
1. Repita los mismos pasos para cada función descrita como parte de la revisión anterior del flujo de ejecución.

Para más información sobre la supervisión de Azure Functions, consulte [Análisis de la telemetría de Azure Functions en Application Insights](../../azure-functions/analyze-telemetry-data.md).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la supervisión de Azure Functions y aplicaciones lógicas:

* [Supervisión de Azure Functions](../../azure-functions/functions-monitoring.md)

* [Configuración de la supervisión para Azure Functions](../../azure-functions/configure-monitoring.md)

* [Supervisión del estado de ejecución, revisión del historial de los desencadenadores y configuración de alertas en Azure Logic Apps](../../logic-apps/monitor-logic-apps.md)