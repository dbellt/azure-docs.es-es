---
title: Archivo de novedades en Azure Sentinel
description: Una descripción de las novedades y los cambios en Azure Sentinel en los últimos seis meses, e incluso antes.
services: sentinel
author: batamig
ms.author: bagol
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: conceptual
ms.date: 05/13/2021
ms.openlocfilehash: 1e629b52b7904d2f671c9d472916ba0a2185a986
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110105393"
---
# <a name="archive-for-whats-new-in-azure-sentinel"></a>Archivo de novedades en Azure Sentinel

La página principal de notas de la versión de [Novedades de Azure Sentinel](whats-new.md) contiene actualizaciones de los últimos seis meses, mientras que esta página contiene elementos más antiguos.

Para más información sobre las características anteriores, consulte los [blogs de Tech Community](https://techcommunity.microsoft.com/t5/azure-sentinel/bg-p/AzureSentinelBlog/label-name/What's%20New).

Actualmente, las características indicadas están en VERSIÓN PRELIMINAR. En la página [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) se incluyen términos legales adicionales que se aplican a las características de Azure que se encuentran en versión beta, versión preliminar o que todavía no se han publicado para su disponibilidad general.


> [!TIP]
> Nuestros equipos de búsqueda de amenazas en Microsoft aportan consultas, cuadernos de estrategias, libros y cuadernos a la [comunidad de Azure Sentinel](https://github.com/Azure/Azure-Sentinel), lo que incluye [consultas de búsqueda](https://github.com/Azure/Azure-Sentinel) concretas que sus equipos pueden adaptar y usar.
>
> Usted también puede contribuir. Únase a nosotros en la [comunidad de GitHub de cazadores de amenazas de Azure Sentinel](https://github.com/Azure/Azure-Sentinel/wiki).
>

## <a name="november-2020"></a>Noviembre de 2020

- [Supervise el estado de los cuadernos de estrategias en Azure Sentinel](#monitor-your-playbooks-health-in-azure-sentinel)
- [Conector de Microsoft 365 Defender (versión preliminar pública)](#microsoft-365-defender-connector-public-preview)

### <a name="monitor-your-playbooks-health-in-azure-sentinel"></a>Supervise el estado de los cuadernos de estrategias en Azure Sentinel

Los cuadernos de estrategias de Azure Sentinel se basan en flujos de trabajo integrados en [Azure Log Apps](../logic-apps/index.yml), un servicio en la nube que le ayuda a programar, automatizar y organizar tareas, procesos empresariales y flujos de trabajo. Los cuadernos de estrategias se pueden invocar automáticamente al crear un incidente o al evaluar y usar incidentes. 

Para proporcionar información sobre el estado, el rendimiento y el uso de los cuadernos de estrategias, se ha agregado un [libro](../azure-monitor/visualize/workbooks-overview.md) denominado **Supervisión del estado de los cuadernos de estrategias**. 

El libro **Playbooks health monitoring** (Supervisión de estado de cuadernos de estrategias) se usa para supervisar el estado de los cuadernos de estrategias o buscar anomalías en la cantidad de ejecuciones correctas o con errores. 

El libro **Playbooks health monitoring** (Supervisión de estado de cuadernos de estrategias) ya está disponible en la galería de plantillas de Azure Sentinel:

:::image type="content" source="media/whats-new/playbook-monitoring-workbook.gif" alt-text="Libro Playbooks health monitoring (Supervisión de estado de cuadernos de estrategias) de ejemplo":::

Para más información, consulte:

- [Documentación de Logic Apps](../logic-apps/monitor-logic-apps-log-analytics.md#set-up-azure-monitor-logs)

- [Documentación sobre Azure Monitor](../azure-monitor/essentials/activity-log.md#send-to-log-analytics-workspace)

### <a name="microsoft-365-defender-connector-public-preview"></a>Conector de Microsoft 365 Defender (versión preliminar pública)
 
El conector de Microsoft 365 Defender para Azure Sentinel permite transmitir en secuencias registros de búsqueda avanzada (un tipo de datos de eventos sin procesar) desde Microsoft 365 Defender hasta Azure Sentinel. 

Con la integración de [Microsoft Defender para punto de conexión (MDATP)](/windows/security/threat-protection/) en el paraguas de seguridad de [Microsoft 365 Defender](/microsoft-365/security/mtp/microsoft-threat-protection), ahora puede recopilar los eventos de búsqueda avanzada de Microsoft Defender para punto de conexión mediante el conector Microsoft 365 Defender y transmitirlos en secuencia directamente a nuevas tablas creadas específicamente en el área de trabajo de Azure Sentinel. 

Las tablas de Azure Sentinel se basan en el mismo esquema que se usa en el portal de Microsoft 365 Defender y proporcionan acceso pleno al conjunto completo de registros de búsqueda avanzada. 

Para más información, consulte [Conexión de datos de Microsoft 365 Defender con Azure Sentinel](connect-microsoft-365-defender.md).

> [!NOTE]
> Microsoft 365 Defender se conocía antes como Microsoft Threat Protection o MTP. Microsoft Defender para Endpoint se conocía anteriormente como Protección contra amenazas avanzada de Microsoft Defender o MDATP.
> 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
>[Incorporación de Azure Sentinel](quickstart-onboard.md)

> [!div class="nextstepaction"]
>[Obtención de visibilidad sobre las alertas](quickstart-get-visibility.md)
