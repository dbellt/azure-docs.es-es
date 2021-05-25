---
title: Obtenga información sobre la integración entre el aprovisionamiento y los registros de Azure Monitor en Azure Active Directory.
description: Obtenga información sobre la integración entre el aprovisionamiento y los registros de Azure Monitor en Azure Active Directory.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: conceptual
ms.workload: identity
ms.date: 05/11/2021
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: 8ad80b7fb8c148ae42d52b15719ddadf4b5302e2
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109783286"
---
# <a name="understand-how-provisioning-integrates-with-azure-monitor-logs"></a>Entienda cómo se integra el aprovisionamiento con los registros de Azure Monitor

El aprovisionamiento se integra con registros de Azure Monitor y Log Analytics. Con la supervisión de Azure puede hacer cosas como crear libros, también conocidos como paneles, almacenar registros de aprovisionamiento durante más de 30 días y crear consultas y alertas personalizadas. En este artículo se describe cómo se integran los registros de aprovisionamiento con los registros de Azure Monitor. Para obtener más información sobre cómo funcionan los registros de aprovisionamiento en general, vea [Registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md).

## <a name="enabling-provisioning-logs"></a>Habilitación de los registros de aprovisionamiento

Ya debería estar familiarizado con la supervisión de Azure y Log Analytics. Si no es así, vaya a obtener más información sobre ellos y luego vuelva aquí para conocer los registros de aprovisionamiento de aplicaciones. Para obtener más información sobre la supervisión de Azure, vea [Introducción a Azure Monitor](../../azure-monitor/overview.md). Para obtener más información sobre los registros de Azure Monitor y Log Analytics, consulte [Introducción a las consultas de registro en Azure Monitor](../../azure-monitor/logs/log-query-overview.md).

Una vez configurada la supervisión de Azure, es posible habilitar registros para el aprovisionamiento de aplicaciones. La opción se encuentra en la página **Configuración de diagnóstico**.

:::image type="content" source="media/application-provisioning-log-analytics/diagnostic-settings.png" alt-text="Acceso a la configuración de diagnóstico" lightbox="media/application-provisioning-log-analytics/diagnostic-settings.png":::

:::image type="content" source="media/application-provisioning-log-analytics/enable-log-analytics.png" alt-text="Habilitación de registros de aprovisionamiento de aplicaciones" lightbox="media/application-provisioning-log-analytics/enable-log-analytics.png":::

> [!NOTE]
> Si acaba de aprovisionar un área de trabajo, puede pasar algún tiempo antes de que pueda enviarle registros. Si recibe un error que le indica que la suscripción no está registrada para usar *microsoft.insights*, vuelva a realizar la comprobación después de unos minutos.
 
## <a name="understanding-the-data"></a>Descripción de los datos
El flujo de datos subyacente que el aprovisionamiento envía a los visores de registro es casi idéntico. Los registros de Azure Monitor obtienen casi el mismo flujo que la interfaz de usuario de Azure Portal y la API de Azure. Solo hay algunas **diferencias** en los campos de registro, tal y como se describe en la tabla siguiente. Para obtener más información sobre estos campos, consulte el artículo sobre la [lista provisioningObjectSummary](/graph/api/provisioningobjectsummary-list?preserve-view=true&tabs=http&view=graph-rest-beta).

|Registros de Azure Monitor   |Interfaz de usuario de Azure Portal   |API de Azure |
|----------|-----------|------------|
|errorDescription |reason |resultDescription |
|status |resultType |resultType |
|activityDateTime |TimeGenerated |TimeGenerated |


## <a name="azure-monitor-workbooks"></a>Libros de Azure Monitor

Los libros de Azure Monitor facilitan un lienzo flexible para el análisis de datos. También permiten la creación de informes visuales completos en Azure Portal. Para obtener más información, consulte [Introducción a los libros de Azure Monitor](../../azure-monitor/visualize/workbooks-overview.md).

El aprovisionamiento de aplicaciones incluye un conjunto de libros precompilados. Puede encontrarlos en la página Libros. Para ver los datos, tiene que asegurarse de que se rellenen todos los filtros (timeRange, jobID y appName). También tendrá que asegurarse de haber aprovisionado una aplicación; de lo contrario, no habrá ningún dato en los registros.

:::image type="content" source="media/application-provisioning-log-analytics/workbooks.png" alt-text="Libros de aprovisionamiento de aplicaciones" lightbox="media/application-provisioning-log-analytics/workbooks.png":::

:::image type="content" source="media/application-provisioning-log-analytics/report.png" alt-text="Panel de aprovisionamiento de aplicaciones" lightbox="media/application-provisioning-log-analytics/report.png":::

## <a name="custom-queries"></a>Consultas personalizadas

Puede crear consultas personalizadas y mostrar los datos en los paneles de Azure. Para conocer cómo hacerlo, consulte [Creación y uso compartido de paneles de datos de Log Analytics](../../azure-monitor/logs/get-started-queries.md). Además, asegúrese de consultar [Introducción a las consultas de registro en Azure Monitor](../../azure-monitor/logs/log-query-overview.md).

Estos son algunos ejemplos para empezar a trabajar con el aprovisionamiento de aplicaciones.

Consulte los registros de un usuario en función de su identificador en el sistema de origen:
```kusto
AADProvisioningLogs
| extend SourceIdentity = parse_json(SourceIdentity)
| where tostring(SourceIdentity.Id) == "49a4974bb-5011-415d-b9b8-78caa7024f9a"
```

Resumen del recuento por ErrorCode:
```kusto
AADProvisioningLogs
| summarize count() by ErrorCode = ResultSignature
```

Resumen del recuento de eventos al día por acción:
```kusto
AADProvisioningLogs
| where TimeGenerated > ago(7d)
| summarize count() by Action, bin(TimeGenerated, 1d)
```

Tome 100 eventos y las propiedades clave del proyecto:
```kusto
AADProvisioningLogs
| extend SourceIdentity = parse_json(SourceIdentity)
| extend TargetIdentity = parse_json(TargetIdentity)
| extend ServicePrincipal = parse_json(ServicePrincipal)
| where tostring(SourceIdentity.identityType) == "Group"
| project tostring(ServicePrincipal.Id), tostring(ServicePrincipal.Name), ModifiedProperties, JobId, Id, CycleId, ChangeId, Action, SourceIdentity.identityType, SourceIdentity.details, TargetIdentity.identityType, TargetIdentity.details, ProvisioningSteps
|take 100
```

## <a name="custom-alerts"></a>Alertas personalizadas

Azure Monitor le permite configurar alertas personalizadas para que pueda recibir notificaciones sobre eventos clave relacionados con el aprovisionamiento. Por ejemplo, es posible que quiera recibir una alerta en caso de picos de errores. O quizás en caso de picos de deshabilitaciones o eliminaciones. Otro ejemplo de cuándo puede querer recibir alertas es la falta de aprovisionamiento, lo que indica que algo no va bien.

Para más información sobre las alertas, consulte [Alertas de registro de Azure Monitor](../../azure-monitor/alerts/alerts-log.md).

Alerta cuando hay un pico de errores. Reemplace jobID por el valor jobID de la aplicación.

:::image type="content" source="media/application-provisioning-log-analytics/alert1.png" alt-text="Alerta cuando hay un pico de errores." lightbox="media/application-provisioning-log-analytics/alert1.png":::

Puede que un problema haya provocado que el servicio de aprovisionamiento haya dejado de ejecutarse. Use la siguiente alerta para detectar cuándo no hay ningún evento de aprovisionamiento durante un intervalo determinado.

:::image type="content" source="media/application-provisioning-log-analytics/alert2.png" alt-text="Puede que un problema haya provocado que el servicio de aprovisionamiento haya dejado de ejecutarse." lightbox="media/application-provisioning-log-analytics/alert2.png":::

Alerta cuando hay un pico de deshabilitaciones o eliminaciones.

:::image type="content" source="media/application-provisioning-log-analytics/alert3.png" alt-text="Alerta cuando hay un pico de deshabilitaciones o eliminaciones." lightbox="media/application-provisioning-log-analytics/alert3.png":::


## <a name="community-contributions"></a>Contribuciones de la comunidad

Se está adoptando un enfoque de código abierto basado en la comunidad para las consultas y los paneles de aprovisionamiento de aplicaciones. Si ha creado una consulta, una alerta o un libro que cree que puede resultar útil a otros usuarios, asegúrese de publicarlo en el [repositorio AzureMonitorCommunity de GitHub](https://github.com/microsoft/AzureMonitorCommunity). Luego, envíenos un correo electrónico con un vínculo. Lo revisaremos y publicaremos en el servicio para que otros usuarios también puedan beneficiarse. Puede ponerse en contacto con nosotros en provisioningfeedback@microsoft.com.

## <a name="next-steps"></a>Pasos siguientes

- [Análisis de registros de actividad de Azure AD con registros de Azure Monitor](../reports-monitoring/howto-analyze-activity-logs-log-analytics.md)
- [Introducción a las consultas de registros de Azure Monitor](../../azure-monitor/logs/get-started-queries.md)
- [Creación y administración de grupos de alertas en Azure Portal](../../azure-monitor/alerts/action-groups.md)
- [Instalación y uso de las vistas de Log Analytics para Azure Active Directory](../reports-monitoring/howto-install-use-log-analytics-views.md)
- [API de registros de aprovisionamiento](/graph/api/resources/provisioningobjectsummary?preserve-view=true&view=graph-rest-beta)