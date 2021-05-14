---
title: Supervisión en la versión preliminar de Azure Static Web Apps
description: Supervise solicitudes, errores e información de seguimiento en la versión preliminar de Azure Static Web Apps
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 4/23/2021
ms.author: cshoe
ms.openlocfilehash: 474b612d835ab415f9607f737ef219acc2e99152
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108077702"
---
# <a name="monitor-azure-static-web-apps"></a>Supervisión en Azure Static Web Apps

Habilite [Application Insights](../azure-monitor/app/app-insights-overview.md) para supervisar las solicitudes de API, los errores y la información de seguimiento.

> [!NOTE]
> Para usar Application Insights con Azure Static Web Apps, se requiere una aplicación con una [API](./add-api.md).

## <a name="add-monitoring"></a>Adición de supervisión

Siga estos pasos para agregar la funcionalidad de supervisión de Application Insights a la aplicación web estática.

1. Abra la instancia de Static Web Apps en Azure Portal.

1. Seleccione **Application Insights** en el menú.

1. Seleccione **Sí** junto a _Habilitar Application Insights_.

1. Seleccione **Guardar**.

:::image type="content" source="media/monitoring/azure-static-web-apps-application-insights-add.png" alt-text="Incorporación de Application Insights a Azure Static Web Apps":::

> [!NOTE]
> Una vez creada la instancia de Application Insights, se crea una configuración de aplicación asociada en la instancia de Azure Static Web Apps que se usa para vincular los servicios.

## <a name="access-data"></a>Acceso a los datos

1. En la ventana _Información general_ de la aplicación web estática, seleccione el vínculo situado junto al _grupo de recursos_.

1. En la lista, seleccione la instancia de Application Insights con el mismo nombre que la aplicación web estática.

A continuación se resaltan algunas ubicaciones del portal que se usan para inspeccionar aspectos de los puntos de conexión de API de la aplicación.

> [!NOTE]
> Para obtener más información sobre el uso de Application Insights, consulte [¿Dónde veo la telemetría?](../azure-monitor/app/app-insights-overview.md#where-do-i-see-my-telemetry).

| Tipo | Ubicación en el menú | Descripción |
|--- | --- | --- |
| [Errores](../azure-monitor/app/asp-net-exceptions.md) | _Investigar > Errores_ | Revise las solicitudes con error. |
| [Solicitudes de servidor](../azure-monitor/app/tutorial-performance.md) | _Investigar > Rendimiento_ | Revise las solicitudes de API individuales.  |
| [Registros](../azure-monitor/app/diagnostic-search.md) | _Supervisión > Registros_ | Interactúe con un editor para consultar los registros de transacciones. |
| [Métricas](../azure-monitor/essentials/app-insights-metrics.md) | _Supervisión > Métricas_ | Interactúe con un diseñador para crear gráficos personalizados mediante diferentes métricas. |

### <a name="traces"></a>Traces

Siga estos pasos para ver los seguimientos en la aplicación.

1. En **Supervisión**, seleccione _Registros_.

1. Mantenga el mouse sobre cualquier tarjeta de la ventana _Consultas_.

1. Seleccione el botón **Cargar editor**.

1. Reemplace la consulta generada por la palabra `traces`.

1. Haga clic en el botón **Ejecutar**.

:::image type="content" source="media/monitoring/azure-static-web-apps-application-insights-traces.png" alt-text="Visualización de seguimientos de Application Insights":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de la autenticación y autorización](authentication-authorization.md)
