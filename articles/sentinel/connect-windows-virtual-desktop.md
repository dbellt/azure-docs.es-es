---
title: Conexión de datos de Windows Virtual Desktop a Azure Sentinel | Microsoft Docs
description: Aprenda a conectar los datos de Windows Virtual Desktop a Azure Sentinel.
services: sentinel
documentationcenter: na
author: batamig
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/22/2021
ms.author: bagol
ms.openlocfilehash: de7126b7a095868680ff08fab6691782379bb742
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111952344"
---
# <a name="connect-windows-virtual-desktop-data-to-azure-sentinel"></a>Conexión de los datos de Windows Virtual Desktop a Azure Sentinel

En este artículo se describe cómo puede supervisar los entornos de Windows Virtual Desktop (WVD) mediante Azure Sentinel.

Por ejemplo, la supervisión de los entornos de Windows Virtual Desktop puede permitirle proporcionar un trabajo más remoto mediante escritorios virtualizados, a la vez que mantiene la posición de seguridad de la organización.

## <a name="windows-virtual-desktop-data-in-azure-sentinel"></a>Datos de Windows Virtual Desktop en Azure Sentinel

Los datos de Windows Virtual Desktop en Azure Sentinel incluye los siguientes tipos:


|data  |Descripción  |
|---------|---------|
|**Registros de eventos de Windows**     |  Los registros de eventos de Windows del entorno de WVD se transmiten a un área de trabajo de Log Analytics habilitada para Azure Sentinel, de la misma manera que los registros de eventos de Windows desde otras máquinas de Windows, fuera del entorno de WVD. <br><br>Instale el agente de Log Analytics en la máquina de Windows y configure los registros de eventos de Windows que se enviarán al área de trabajo de Log Analytics.<br><br>Para más información, consulte:<br>- [Instalación del agente de Log Analytics en equipos Windows](../azure-monitor/agents/agent-windows.md)<br>- [Recopilación de orígenes de datos del registro de eventos de Windows con el agente de Log Analytics](../azure-monitor/agents/data-sources-windows-events.md)<br>- [Conexión de eventos de seguridad de Windows](connect-windows-security-events.md)       |
|**Alertas de Microsoft Defender para punto de conexión**     |  Para configurar Microsoft Defender para punto de conexión para Windows Virtual Desktop, use el mismo procedimiento para cualquier otro punto de conexión de Windows. <br><br>Para más información, consulte: <br>- [Configuración de la implementación de Microsoft Defender para punto de conexión](/windows/security/threat-protection/microsoft-defender-atp/production-deployment)<br>- [Conexión de datos de Microsoft 365 Defender con Azure Sentinel](connect-microsoft-365-defender.md)       |
|**Diagnostico de Windows Virtual Desktop**     | Diagnóstico de Windows Virtual Desktop es una característica del servicio PaaS de Windows Virtual Desktop, que registra información cada vez que alguien asignado al rol de Windows Virtual Desktop usa el servicio. <br><br>Cada registro contiene información sobre el rol de Windows Virtual Desktop implicado en la actividad, los mensajes de error que se muestran durante la sesión, la información del inquilino y la información del usuario. <br><br>La característica de diagnóstico crea registros de actividad para las acciones de usuario y administrativas. <br><br>Para obtener más información, consulte [Uso de Log Analytics para la característica de diagnóstico en Windows Virtual Desktop](../virtual-desktop/virtual-desktop-fall-2019/diagnostics-log-analytics-2019.md).        |
|     |         |

## <a name="connect-windows-virtual-desktop-data"></a>Conexión con los datos de Windows Virtual Desktop

Para empezar la ingesta de datos de Windows Virtual Desktop en Azure Sentinel, siga las instrucciones de la documentación de los datos Windows Virtual Desktop.

Para obtener más información, consulte [Inserción de los datos de Windows Virtual Desktop en el área de trabajo de Log Analytics.](../virtual-desktop/diagnostics-log-analytics.md)

## <a name="find-your-data"></a>Búsqueda de sus datos

Una vez establecida una conexión correcta, ejecute las consultas en Azure Sentinel en función de los datos de Log Analytics.

Por ejemplo, puede acceder a las consultas de ejemplo que se encuentran en la [documentación de Windows Virtual Desktop](../virtual-desktop/diagnostics-log-analytics.md).


Azure Sentinel también proporciona consultas integradas en el área **General** > **Registros** > **WINDOWS VIRTUAL DESKTOP**:

[![Consultas integradas de Windows Virtual Desktop en Azure Sentinel.](media/connect-windows-virtual-desktop/windows-virtual-desktop-queries.png) ](media/connect-windows-virtual-desktop/windows-virtual-desktop-queries.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes


Para obtener más información, consulte el [glosario de Azure Monitor para Windows Virtual Desktop](../virtual-desktop/azure-monitor-glossary.md).