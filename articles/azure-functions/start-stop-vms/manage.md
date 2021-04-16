---
title: Administración de Start/Stop VMs v2 (versión preliminar)
description: En este artículo se explica cómo supervisar el estado de las máquinas virtuales de Azure administradas por la característica Start/Stop VMs v2 (versión preliminar) y realizar otras tareas de administración.
services: azure-functions
ms.subservice: ''
ms.date: 03/16/2021
ms.topic: conceptual
ms.openlocfilehash: 477e3fe51f05bb4323642f56233f1995e6394eec
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106111451"
---
# <a name="how-to-manage-startstop-vms-v2-preview"></a>Administración de Start/Stop VMs v2 (versión preliminar)

## <a name="azure-dashboard"></a>Panel de Azure

Start/Stop VMs v2 (versión preliminar) incluye un [panel](../../azure-monitor/visualizations.md#azure-dashboards) que le ayudará a comprender el ámbito de administración y las operaciones recientes en las máquinas virtuales. Es una manera rápida y sencilla de comprobar el estado de cada operación que se realiza en las máquinas virtuales de Azure. La visualización de cada icono se basa en una consulta de registro y, para ver la consulta, seleccione la opción de la **hoja Abrir en Registros** en la esquina derecha del icono. Esto abre la herramienta de [Log Analytics](../../azure-monitor/logs/log-analytics-overview.md#starting-log-analytics) en Azure Portal y, desde aquí, puede evaluar la consulta y modificarla para satisfacer sus necesidades, como las [alertas de registro](../../azure-monitor/alerts/alerts-log.md) personalizadas, un [libro](../../azure-monitor/visualize/workbooks-overview.md) personalizado, etc.

Los datos de registro que muestra cada icono en el panel se actualiza cada hora, con una opción de actualización manual a petición mediante un clic en el icono **Actualizar** en una visualización determinada o mediante la actualización del panel completo.

Para más información sobre cómo trabajar con un panel basado en registros, consulte el siguiente [tutorial](../../azure-monitor/visualize/tutorial-logs-dashboards.md).

## <a name="configure-email-notifications"></a>Configuración de notificaciones de correo electrónico

Para cambiar las notificaciones de correo electrónico después de implementar la característica Start/Stop VMs v2 (versión preliminar), puede modificar el grupo de acciones creado durante la implementación.

1. En Azure Portal, vaya a **Supervisar** y, a continuación, a **Alertas**. Seleccione **Administrar acciones**.

1. En la página **Administrar acciones**, seleccione el grupo de acciones llamado **StartStopV2_VM_Notification**.

    :::image type="content" source="media/manage/alerts-action-groups.png" alt-text="Captura de pantalla de la página Grupos de acciones.":::

1. En la página **StartStopV2_VM_Notification**, puede modificar las opciones de notificación por correo electrónico, SMS, inserción y voz. Agregue otras acciones o actualice la configuración existente a este grupo de acciones y, después, haga clic en **Aceptar** para guardar los cambios.

    :::image type="content" source="media/manage/email-notification-type-example.png" alt-text="Captura de pantalla de la página Correo electrónico, SMS, Notificación push y Voz que muestra una dirección de correo electrónico de ejemplo actualizada.":::

    Para más información sobre los grupos de acciones, consulte la información sobre [grupos de acciones](../../azure-monitor/alerts/action-groups.md).

La siguiente captura de pantalla es un correo electrónico de ejemplo que se envía cuando la característica cierra las máquinas virtuales.

:::image type="content" source="media/manage/email-notification-example.png" alt-text="Captura de pantalla de un correo electrónico de ejemplo que se envía cuando la característica cierra las máquinas virtuales.":::

## <a name="next-steps"></a>Pasos siguientes

Para solucionar problemas durante la administración de máquinas virtuales, consulte la [solución de problemas de la característica Start/Stop VMs v2](troubleshoot.md) (versión preliminar).