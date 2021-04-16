---
title: Solución de problemas de Azure Monitor para Windows Virtual Desktop - Azure
description: Solución de problemas con Azure Monitor para Windows Virtual Desktop.
author: Heidilohr
ms.topic: troubleshooting
ms.date: 03/29/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: a104f4d33e1bd38e130101b34d3fd2021de27cd2
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106445489"
---
# <a name="troubleshoot-azure-monitor-for-windows-virtual-desktop"></a>Solución de problemas de Azure Monitor para Windows Virtual Desktop

En este artículo se presentan problemas conocidos y soluciones para problemas comunes en Azure Monitor para Windows Virtual Desktop.

## <a name="issues-with-configuration-and-setup"></a>Problemas con la configuración y la instalación

Si el libro de configuración no funciona correctamente para automatizar la instalación, puede usar estos recursos para configurar el entorno manualmente:

- Para habilitar manualmente los diagnósticos o acceder al área de trabajo de Log Analytics, consulte [Envío de diagnósticos de Windows Virtual Desktop a Log Analytics](diagnostics-log-analytics.md).
- Para instalar manualmente la extensión de Log Analytics en un host de sesión, consulte [Extensión de máquina virtual de Log Analytics para Windows](../virtual-machines/extensions/oms-windows.md).
- Para configurar una nueva área de trabajo de Log Analytics, consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](../azure-monitor/logs/quick-create-workspace.md).
- Para agregar, quitar o editar los contadores de rendimiento, consulte [Configuración de contadores de rendimiento](../azure-monitor/agents/data-sources-performance-counters.md).
- Para configurar registros de eventos de Windows para un área de trabajo de Log Analytics, consulte [Recopilación de orígenes de datos del registro de eventos de Windows con el agente de Log Analytics](../azure-monitor/agents/data-sources-windows-events.md).

## <a name="my-data-isnt-displaying-properly"></a>Los datos no se muestran correctamente

Si los datos no se muestran correctamente, compruebe las siguientes soluciones comunes:

- En primer lugar, asegúrese de haber configurado correctamente el libro de configuración, tal como se describe en [Uso de Azure Monitor para Windows Virtual Desktop para la supervisión de implementaciones](azure-monitor.md). Si faltan contadores o eventos, los datos asociados a ellos no se mostrarán en Azure Portal.
- Compruebe los permisos de acceso y póngase en contacto con los propietarios de los recursos para solicitar los permisos que faltan; cualquier usuario que supervise Windows Virtual Desktop necesita los siguientes permisos:
    - Acceso de lectura a las suscripciones de Azure que contienen los recursos de Windows Virtual Desktop
    - Acceso de lectura a los grupos de recursos de la suscripción que contienen los hosts de la sesión de Windows Virtual Desktop 
    - Acceso de lectura a cualquier área de trabajo de Log Analytics que esté usando
- Es posible que tenga que abrir los puertos de salida en el firewall del servidor para permitir que Azure Monitor y Log Analytics envíen datos al portal. Para obtener información sobre cómo hacerlo, consulte los siguientes artículos:
      - [Puertos de salida de Azure Monitor](../azure-monitor/app/ip-addresses.md)
      - [Requisitos del firewall de Log Analytics](../azure-monitor/agents/log-analytics-agent.md#firewall-requirements). 
- ¿No ve los datos de la actividad reciente? Puede esperar 15 minutos y actualizar la fuente. Azure Monitor tiene un período de latencia de 15 minutos para rellenar los datos de registro. Para más información, consulte [Tiempo de la ingesta de datos de registro en Azure Monitor](../azure-monitor/logs/data-ingestion-time.md).

Si no le falta información, pero los datos todavía no se muestran correctamente, puede haber un problema en la consulta o en los orígenes de datos. Consulte [Limitaciones y problemas conocidos](#known-issues-and-limitations). 

## <a name="i-want-to-customize-azure-monitor-for-windows-virtual-desktop"></a>Quiero personalizar Azure Monitor para Windows Virtual Desktop

Azure Monitor para Windows Virtual Desktop usa los libros de Azure Monitor. Los libros permiten guardar una copia de la plantilla de libro de Windows Virtual Desktop y realizar sus propias personalizaciones.

Por diseño, las plantillas de libro personalizadas no adoptarán automáticamente las actualizaciones del grupo de productos. Para obtener más información, consulte [Solución de problemas de conclusiones basadas en libros](../azure-monitor/insights/troubleshoot-workbooks.md) y la [información general sobre libros](../azure-monitor/visualize/workbooks-overview.md).

## <a name="i-cant-interpret-the-data"></a>No puedo interpretar los datos

Obtenga más información sobre los términos de datos en el [glosario de Azure Monitor para Windows Virtual Desktop](azure-monitor-glossary.md).

## <a name="the-data-i-need-isnt-available"></a>Los datos que necesito no están disponibles

Si quiere supervisar más contadores de rendimiento o registros de eventos de Windows, puede habilitarlos para que envíen la información de diagnóstico al área de trabajo de Log Analytics y supervisarlos en **Diagnósticos de host: explorador del host**. 

- Para agregar contador de rendimiento, consulte [Configuración de contadores de rendimiento](../azure-monitor/agents/data-sources-performance-counters.md#configuring-performance-counters).
- Para agregar eventos de Windows, consulte [Configuración de registros de eventos de Windows](../azure-monitor/agents/data-sources-windows-events.md#configuring-windows-event-logs).

¿No encuentra un punto de datos para diagnosticar un problema? ¡Envíenos sus comentarios!

- Para obtener información sobre cómo dejar comentarios, consulte [Información general, comentarios y soporte técnico para la solución de problemas de Windows Virtual Desktop](troubleshoot-set-up-overview.md).
- También puede dejar comentarios sobre Windows Virtual Desktop en el [Concentrador de comentarios de Windows Virtual Desktop](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app).

## <a name="known-issues-and-limitations"></a>Limitaciones y problemas conocidos

Los siguientes son los problemas y las limitaciones que conocemos y en los que estamos trabajando para solucionar:

- Solo puede supervisar un grupo de hosts a la vez. 
- Para guardar los parámetros de configuración favoritos, debe guardar una plantilla personalizada del libro. Las plantillas personalizadas no adoptarán automáticamente las actualizaciones del grupo de productos.
- En ocasiones, el libro de configuración mostrará errores de "error de consulta" al cargar las selecciones. Actualice la consulta, vuelva a escribir la selección si es necesario, y el error se debería resolver por sí solo. 
- Algunos mensajes de error no están escritos de manera descriptiva, y no todos los mensajes de error se describen en la documentación.
- El contador de rendimiento de sesiones totales puede contar sesiones en exceso en un número reducido, y puede parecer que las sesiones totales superan el límite máximo de sesiones.
- El número de sesiones disponibles no refleja las directivas de escalado en el grupo de hosts.   
- ¿Observa tiempos de conexión contradictorios o inesperados? Aunque es poco frecuente, es posible que falte un evento de finalización de una conexión, lo que puede afectar a algunos objetos visuales y métricas.
- El tiempo de conexión incluye el tiempo que tardan los usuarios en escribir sus credenciales. Esto se correlaciona con la experiencia, pero en algunos casos puede mostrar picos falsos. 
    

## <a name="next-steps"></a>Pasos siguientes

- Para empezar, consulte [Uso de Azure Monitor para Windows Virtual Desktop para supervisar implementaciones](azure-monitor.md).
- Para estimar, medir y administrar los costos de almacenamiento de datos, consulte [Cálculo de costos de Azure Monitor](azure-monitor-costs.md).
- Consulte nuestro [glosario](azure-monitor-glossary.md) para obtener más información sobre los términos y conceptos relacionados con Azure Monitor para Windows Virtual Desktop.
