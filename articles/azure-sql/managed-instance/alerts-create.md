---
title: Configuración de alertas y notificaciones para Instancia administrada (Azure Portal)
description: Use Azure Portal para crear alertas de Instancia administrada de SQL, que pueden desencadenar notificaciones o automatización cuando se cumplen las condiciones que ha especificado.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: urosmil
ms.author: urmilano
ms.reviewer: mathoma, wiassaf
ms.date: 05/04/2020
ms.openlocfilehash: 7b596b6e9702d0518bb1103088e718529ad7929e
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111593065"
---
# <a name="create-alerts-for-azure-sql-managed-instance-using-the-azure-portal"></a>Creación de alertas para Instancia administrada de Azure SQL mediante Azure Portal
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este artículo se muestra cómo configurar alertas para bases de datos en Instancia administrada de Azure SQL Database mediante Azure Portal. Alertas puede enviarle un correo electrónico, llamar a un webhook, ejecutar una función de Azure, un runbook, llamar a un sistema externo de vales compatible con ITSM, llamarle al teléfono o enviarle un mensaje de texto si una métrica como, por ejemplo, el tamaño de almacenamiento de la instancia o el uso de la CPU, alcanza un umbral predefinido. En este artículo también se indican procedimientos recomendados para establecer periodos de alerta.


## <a name="overview"></a>Información general

Puede recibir una alerta basada en las métricas de supervisión para los servicios de Azure o los eventos sobre ellos.

* **Valores de métrica** : la alerta se desencadena cuando el valor de una métrica específica cruza un umbral asignado en cualquier dirección. Es decir, se desencadena tanto la primera vez que se cumple la condición como después, cuando dicha condición ya deja de cumplirse.

Puede configurar una alerta para hacer lo siguiente cuando se desencadena:

* Enviar notificaciones por correo electrónico al administrador del servicio y a los coadministradores.
* Enviar un correo electrónico a direcciones de correo electrónico adicionales que especifique.
* Llamar a un número de teléfono con un mensaje de voz
* Enviar un mensaje de texto a un número de teléfono
* Llamar a un webhook
* Llamar a una función de Azure
* Llamar a un runbook de Azure
* Llamar a un sistema externo de vales compatible con ITSM

Puede configurar y obtener información sobre las reglas de alerta mediante [Azure Portal, PowerShell o la CLI de Azure](../../azure-monitor/alerts/alerts-classic-portal.md) o la [API de REST de Azure Monitor](/rest/api/monitor/alertrules). 

## <a name="alerting-metrics-available-for-managed-instance"></a>Métricas de alertas disponibles para la instancia administrada

> [!IMPORTANT]
> Las métricas de alertas están disponibles solo para la instancia administrada. Las métricas de alertas para bases de datos individuales de la instancia administrada no están disponibles. Por otra parte, la telemetría de diagnóstico de la base de datos está disponible en forma de [registros de diagnóstico](../database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md#diagnostic-telemetry-for-export). Las alertas de los registros de diagnóstico se pueden configurar desde el producto [SQL Analytics](../../azure-monitor/insights/azure-sql.md) mediante [scripts de alertas de registro](../../azure-monitor/insights/azure-sql.md#creating-alerts-for-sql-managed-instance) para la instancia administrada.

Las siguientes métricas de la instancia administrada están disponibles para la configuración de alertas:

| Métrica | Descripción | Unidad de medida \ valores posibles |
| :--------- | --------------------- | ----------- |
| Porcentaje de CPU medio | Porcentaje medio de uso de la CPU en el período de tiempo seleccionado. | 0-100 (porcentaje) |
| Bytes de E/S leídos | Bytes de E/S leídos en el período de tiempo seleccionado. | Bytes |
| Bytes de E/S escritos | Bytes de E/S escritos en el período de tiempo seleccionado. | Bytes |
| Recuento de solicitudes de E/S | Recuento de solicitudes de E/S en el período de tiempo seleccionado. | Numérico |
| Espacio de almacenamiento reservado | Espacio máximo de almacenamiento reservado actual para la instancia administrada. Cambios con la operación de escalado de recursos. | MB (megabytes) |
| Espacio de almacenamiento usado | Espacio de almacenamiento usado en el período seleccionado. Cambios con el consumo de almacenamiento de las bases de datos y la instancia. | MB (megabytes) |
| Recuento de núcleos virtuales | Núcleos virtuales aprovisionados para la instancia administrada. Cambios con la operación de escalado de recursos. | 4-80 (núcleos virtuales) |

## <a name="create-an-alert-rule-on-a-metric-with-the-azure-portal"></a>Creación de una regla de alerta de una métrica con Azure Portal

1. En [Azure Portal](https://portal.azure.com/), busque la instancia administrada que desea supervisar y selecciónela.

2. Seleccione el elemento de menú **Métricas** en la sección Supervisión.

   ![Supervisión](./media/alerts-create/mi-alerting-menu-annotated.png)
  
3. En el menú desplegable, seleccione una de las métricas en las que quiere configurar la alerta (el espacio de almacenamiento usado es la que se utiliza en el ejemplo).

4. Seleccione el período de agregación: promedio, mínimo o máximo alcanzado en el período de tiempo especificado (Promedio, Mínimo o Máximo). 

5. Seleccione **Nueva regla de alertas**.

6. En el panel Crear regla de alertas, haga clic en **Nombre de la condición** (en el ejemplo se utiliza el espacio de almacenamiento usado)

   ![Definir condición](./media/alerts-create/mi-create-metrics-alert-smaller-annotated.png)

7. En el panel Configurar lógica de señal, defina el operador, el tipo de agregación y el valor de umbral.

   * Las opciones del tipo de operador son: mayor que, igual que y menor que (el valor de umbral).
   * Las opciones del tipo de agregación son mínimo, máximo o promedio (en el período de granularidad de la agregación)
   * El valor de umbral es el valor de la alerta que se evaluará en función de los criterios del operador y la agregación.
   
   ![Configure_signal_logic](./media/alerts-create/mi-configure-signal-logic-annotated.png)
   
   En el ejemplo que se muestra en la captura de pantalla, se usa el valor 1 840 876 MB que representa un valor de umbral de 1,8 TB. Como el operador del ejemplo se ha establecido en "mayor que", se creará la alerta si el consumo de espacio de almacenamiento en la instancia administrada supera los 1,8 TB. Tenga en cuenta que el valor de umbral para las métricas de espacio de almacenamiento debe expresarse en MB.

8. Establezca el período de evaluación o granularidad de la agregación en minutos y la frecuencia de evaluación. La frecuencia de evaluación indica el tiempo que el sistema de alertas comprobará periódicamente si se ha cumplido la condición de umbral.

9. Seleccione el grupo de acciones. Este aparecerá y podrá seleccionar una acción existente o crear una nueva. Esta acción define qué ocurrirá cuando se desencadene una alerta (por ejemplo, enviar un correo electrónico, llamar a un teléfono, ejecutar un webhook, una función de Azure o un runbook).

   ![Select_action_group](./media/alerts-create/mi-select-action-group-smaller-annotated.png)

   * Para crear un nuevo grupo de acciones, seleccione **+Crear grupo de acciones**.

      ![Create_action_group_alerts](./media/alerts-create/mi-create-alert-action-group-smaller-annotated.png)
   
   * Defina cómo desea recibir las alertas: Especifique el nombre del grupo de acciones, un nombre corto, el nombre de la acción y seleccione el tipo de acción. El tipo de acción define si se le notificará por correo electrónico, mensaje de texto, llamada de voz o, tal vez, se ejecutará un webhook, una función de Azure, un runbook o se creará el vale de ITSM en el sistema compatible.

      ![Define_how_to_be_alerted](./media/alerts-create/mi-add-alerts-action-group-annotated.png)

10. Rellene los detalles de la regla de alertas para los registros y seleccione el tipo de gravedad.

      ![Rule_description](./media/alerts-create/mi-rule-details-complete-smaller-annotated.png)

   * Para completar la creación de la regla de alertas, haga clic en el botón **Crear regla de alertas**.

La nueva regla de alertas se activará en unos minutos y se desencadenará según lo estipulado en la configuración.

## <a name="verifying-alerts"></a>Comprobación de las alertas

> [!NOTE]
> Para suprimir alertas falsas, consulte [Supresión de alertas mediante reglas de acción](../../azure-monitor/alerts/alerts-action-rules.md#suppression-of-alerts).

Después de configurar una regla de alertas, compruebe que está satisfecho con el desencadenador de alertas y su frecuencia. En el ejemplo que aparece en esta página de configuración de una alerta en el espacio de almacenamiento usado, si la opción de alerta es correo electrónico, es posible que reciba un correo electrónico como el que se muestra a continuación.

   ![alert_example](./media/alerts-create/mi-email-alert-example-smaller-annotated.png)

El correo electrónico muestra el nombre de la alerta, los detalles del umbral y el motivo por el que se desencadenó la alerta, lo que le ayudará a comprobar y solucionar los problemas de la alerta. Puede usar el botón **Ver en Azure Portal** para ver la alerta recibida por correo electrónico en Azure Portal. 

## <a name="view-suspend-activate-modify-and-delete-existing-alert-rules"></a>Visualización, suspensión, activación, modificación y eliminación de las reglas de alertas existentes

> [!NOTE]
> Las alertas existentes se deben administrar desde el menú Alertas del panel de Azure Portal. Las alertas existentes no se pueden modificar desde la hoja de recursos Instancia administrada.

Para ver, suspender, activar, modificar y eliminar las reglas de alertas existentes:

1. Busque Alertas mediante la función de búsqueda de Azure Portal. Haga clic en Alertas.

   ![find_alerts](./media/alerts-create/mi-manage-alerts-browse-smaller-annotated.png)

   O bien, también puede hacer clic en Alertas en la barra de navegación de Azure, si la tiene configurada.

2. En el panel Alertas, seleccione Administrar reglas de alerta.

   ![modify_alerts](./media/alerts-create/mi-manage-alert-rules-smaller-annotated.png)

   Aparecerá la lista de alertas existentes. Seleccione una regla de alertas individual existente para administrarla. Las reglas activas existentes se pueden modificar y ajustar a sus preferencias. Las reglas activas también se pueden suspender sin necesidad de eliminarse. 

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre el sistema de alertas de Azure Monitor, consulte [Información general sobre las alertas en Microsoft Azure](../../azure-monitor/alerts/alerts-overview.md).
* Para más información acerca de las alertas de métrica, consulte [Comprender cómo funcionan las alertas de métricas en Azure Monitor](../../azure-monitor/alerts/alerts-metric-overview.md).
* Para más información sobre cómo configurar un webhook en alertas, consulte [Llamada a un webhook con una alerta de métrica clásica](../../azure-monitor/alerts/alerts-webhooks.md).
* Para más información sobre la configuración y administración de alertas con PowerShell, consulte las [Reglas de acción](/powershell/module/az.monitor/add-azmetricalertrulev2).
* Para más información sobre la configuración y administración de alertas mediante la API, consulte la [referencia de la API REST de Azure Monitor](/rest/api/monitor/).