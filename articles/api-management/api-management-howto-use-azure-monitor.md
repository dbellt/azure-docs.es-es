---
title: 'Tutorial: Supervisión de las API publicadas en Azure API Management | Microsoft Docs'
description: Siga los pasos de este tutorial para aprender a usar métricas, alertas, registros de actividad y registros de recursos para supervisar las API en Azure API Management.
services: api-management
author: vladvino
ms.service: api-management
ms.custom: mvc
ms.topic: tutorial
ms.date: 10/14/2020
ms.author: apimpm
ms.openlocfilehash: f70a43efd0b56f76ee4b68e8fede3ce4e255063d
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110466031"
---
# <a name="tutorial-monitor-published-apis"></a>Tutorial: Supervisión de las API publicadas

Con Azure Monitor, puede visualizar, consultar, enrutar, archivar y realizar acciones sobre las métricas o los registros procedentes del servicio Azure API Management.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Visualización de las métricas de las API 
> * Configuración de una regla de alerta 
> * Visualización de registros de actividad
> * Habilitamiento y visualización de registros de recursos

También puede usar el [análisis](howto-use-analytics.md) integrado de API Management para supervisar el uso y el rendimiento de las API.

## <a name="prerequisites"></a>Requisitos previos

+ Conocer la [terminología de API Management de Azure](api-management-terminology.md).
+ Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
+ Además, realice el siguiente tutorial: [Importación y publicación de la primera API](import-and-publish.md).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="view-metrics-of-your-apis"></a>Visualización de las métricas de las API

API Management emite [métricas](../azure-monitor/essentials/data-platform-metrics.md) cada minuto, lo que le ofrece visibilidad casi en tiempo real sobre el estado y el mantenimiento de las API. Las siguientes son las dos métricas que se usan con más frecuencia. Para obtener una lista de todas las métricas disponibles, consulte las [métricas compatibles](../azure-monitor/essentials/metrics-supported.md#microsoftapimanagementservice).

* **Capacity** (Capacidad): ayuda a tomar decisiones acerca de cómo actualizar o degradar los servicios de APIM. La métrica se emite por minuto y refleja la capacidad de la puerta de enlace en el momento del informe. La métrica oscila entre 0 y 100, y se calcula en función de los recursos de la puerta de enlace, como el uso de la CPU y de la memoria.
* **Requests** (Solicitudes): ayuda a analizar el tráfico de API que pasa por los servicios API Management. La métrica se emite por minuto y notifica el número de solicitudes de puerta de enlace con dimensiones que incluyen los códigos de respuesta, la ubicación, el nombre de host y los errores. 

> [!IMPORTANT]
> Las siguientes métricas han quedado en desuso a partir de mayo de 2019 y se retirarán en agosto de 2023: Total Gateway Requests (Solicitudes totales de puerta de enlace), Successful Gateway Requests (Solicitudes de puerta de enlace correctas), Unauthorized Gateway Requests (Solicitudes de puerta de enlace no autorizadas), Failed Gateway Requests (Solicitudes de puerta de enlace con error), Other Gateway Requests (Otras solicitudes de puerta de enlace). Migre a la métrica Requests (Solicitudes), que proporciona una funcionalidad equivalente.

:::image type="content" source="media/api-management-howto-use-azure-monitor/apim-monitor-metrics.png" alt-text="Captura de pantalla de Métricas en la información general de API Management":::

Para acceder a la métrica:

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com). En la página **Información general**, revise las métricas clave de las API.
1. Para investigar las métricas con más detenimiento, seleccione **Métricas** en el menú situado cerca de la parte inferior de la página.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-metrics-blade.png" alt-text="Captura de pantalla del elemento Métricas en el menú de supervisión":::

1. En la lista desplegable, seleccione las métricas que le interesen. Por ejemplo, **Requests**. 
1. El gráfico muestra el número total de llamadas API.
1. El gráfico se puede filtrar mediante las dimensiones de la métrica de **Solicitudes**. Por ejemplo, seleccione **Agregar filtro**, **Backend Response Code Category** (Categoría del código de respuesta de back-end) y escriba 500 como valor. Ahora el gráfico muestra el número de solicitudes erróneas en el back-end de la API.   

## <a name="set-up-an-alert-rule"></a>Configuración de una regla de alerta 

Puede recibir [alertas](../azure-monitor/alerts/alerts-metric-overview.md) basadas en métricas y registros de actividad. Azure Monitor permite [configurar una alerta](../azure-monitor/alerts/alerts-metric.md) que haga lo siguiente cuando se desencadena:

* Enviar una notificación por correo electrónico
* Llamar a un webhook
* Invocar una aplicación lógica de Azure

Para configurar una regla de alerta de ejemplo basada en una métrica de solicitud:

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
1. Seleccione **Alertas** en la barra de menús cerca de la parte inferior de la página.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/alert-menu-item.png" alt-text="Captura de pantalla de la opción Alertas en el menú de supervisión":::

1. Seleccione **+ Nueva regla de alertas**.
1. En la ventana **Crear regla de alerta**, haga clic en **Seleccionar condición**.
1. En la ventana **Configurar lógica de señal**:
    1. En **Tipo de señal**, seleccione **Métricas**.
    1. En **Nombre de señal**, seleccione **Métricas**.
    1. En **Split by dimensions** (Dividir por dimensiones), en **Nombre de dimensión**, seleccione **Gateway Response Code Category** (Categoría de código de respuesta de puerta de enlace).
    1. En **Valores de dimensión**, seleccione **4xx**, para errores de cliente como solicitudes no autorizadas o no válidas.
    1. En **Lógica de alerta**, especifique un umbral después del cual se debe activar la alerta y seleccione **Hecho**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/threshold.png" alt-text="Captura de pantalla de las ventanas Configurar lógica de señal":::

1. Seleccione un grupo de acciones existente o cree uno nuevo. En el siguiente ejemplo, se crea un nuevo grupo de acciones. Se enviará un correo electrónico de notificación a admin@contoso.com. 

    :::image type="content" source="media/api-management-howto-use-azure-monitor/action-details.png" alt-text="Captura de pantalla de las notificaciones para el nuevo grupo de acciones":::

1. Escriba un nombre y la descripción de la regla de alertas y seleccione el nivel de gravedad. 
1. Seleccione **Crear regla de alertas**.
1. Ahora, pruebe la regla de alertas mediante una llamada a Conference API sin una clave de API. Por ejemplo:

    ```bash
    curl GET https://apim-hello-world.azure-api.net/conference/speakers HTTP/1.1 
    ```

    Se desencadenará una alerta basada en el período de evaluación y se enviará el correo electrónico a admin@contoso.com. 

    Las alertas también aparecen en la página **Alertas** de la instancia de API Management.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/portal-alerts.png" alt-text="Captura de pantalla de Alertas en el portal":::

## <a name="activity-logs"></a>Registros de actividad

Los registros de actividad proporcionan información sobre las operaciones llevadas a cabo en los servicios API Management. Con los registros de actividades, puede determinar los interrogantes “qué, quién y cuándo” de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en los servicios API Management.

> [!NOTE]
> Los registros de actividad no incluyen las operaciones de lectura (GET) ni las realizadas en Azure Portal o mediante las API de administración originales.

Puede acceder a registros de actividad en el servicio API Management o a los registros de todos los recursos de Azure en Azure Monitor. 

:::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-activity-logs.png" alt-text="Captura de pantalla del registro de actividad en el portal":::

Para ver el registro de actividad:

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).

1. Seleccione **Registro de actividad**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-activity-logs-blade.png" alt-text="Captura de pantalla del elemento Registro de actividad en el menú de supervisión":::
1. Seleccione el ámbito de filtrado que desee y seleccione **Aplicar**.

## <a name="resource-logs"></a>Registros del recurso

Los registros de recurso proporcionan información valiosa acerca de las operaciones y los errores que son importantes para la auditoría, así como para solucionar problemas. Los registros de recurso son diferentes de los registros de actividad. El registro de actividad proporciona información detallada sobre las operaciones llevadas a cabo en los recursos de Azure. Los registros de recurso proporcionan información detallada sobre las operaciones que realiza el recurso.

Para configurar los registros de recurso:

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
2. Seleccione **Configuración de diagnóstico**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-diagnostic-logs-blade.png" alt-text="Captura de pantalla del elemento Configuración de diagnóstico en el menú de supervisión":::

1. Seleccione **+ Agregar configuración de diagnóstico**.
1. Seleccione los registros o métricas que desea recopilar.

   Los registros de recurso se pueden archivar junto con las métricas en una cuenta de almacenamiento, transmitirlos en secuencias a un centro de eventos o enviarlos a un área de trabajo de Log Analytics. 

Para más información, consulte [Creación de una configuración de diagnóstico para enviar registros de plataforma y métricas a diferentes destinos](../azure-monitor/essentials/diagnostic-settings.md).

## <a name="view-diagnostic-data-in-azure-monitor"></a>Visualización de datos de diagnóstico en Azure Monitor

Si habilita la recopilación de registros de GatewayLogs o métricas en un área de trabajo de Log Analytics, los datos pueden tardar unos minutos en aparecer en Azure Monitor. Para ver los datos:

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
1. Seleccione **Registros** en el menú situado cerca de la parte inferior de la página.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/logs-menu-item.png" alt-text="Captura de pantalla del elemento Registros en el menú de supervisión":::

Ejecute consultas para ver los datos. Se proporcionan varias [consultas de ejemplo](../azure-monitor/logs/queries.md), pero también puede ejecutar la suya propia. Por ejemplo, la consulta siguiente recupera las 24 horas de datos más recientes de la tabla GatewayLogs:

```kusto
ApiManagementGatewayLogs
| where TimeGenerated > ago(1d) 
```

Para más información acerca del uso de registros de recursos para API Management, consulte:

* [Introducción a los análisis de registros de Azure Monitor](../azure-monitor/logs/log-analytics-tutorial.md), o bien pruebe el [entorno de demostración de Log Analytics](https://portal.loganalytics.io/demo).

* [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/logs/log-query-overview.md).

El siguiente JSON indica una entrada de ejemplo en GatewayLogs para una solicitud de API correcta. Para más información, consulte la [referencia del esquema](gateway-log-schema-reference.md). 

```json
{
    "Level": 4,
    "isRequestSuccess": true,
    "time": "2020-10-14T17:xx:xx.xx",
    "operationName": "Microsoft.ApiManagement/GatewayLogs",
    "category": "GatewayLogs",
    "durationMs": 152,
    "callerIpAddress": "xx.xx.xxx.xx",
    "correlationId": "3f06647e-xxxx-xxxx-xxxx-530eb9f15261",
    "location": "East US",
    "properties": {
        "method": "GET",
        "url": "https://apim-hello-world.azure-api.net/conference/speakers",
        "backendResponseCode": 200,
        "responseCode": 200,
        "responseSize": 41583,
        "cache": "none",
        "backendTime": 87,
        "requestSize": 526,
        "apiId": "demo-conference-api",
        "operationId": "GetSpeakers",
        "apimSubscriptionId": "master",
        "clientTime": 65,
        "clientProtocol": "HTTP/1.1",
        "backendProtocol": "HTTP/1.1",
        "apiRevision": "1",
        "clientTlsVersion": "1.2",
        "backendMethod": "GET",
        "backendUrl": "https://conferenceapi.azurewebsites.net/speakers"
    },
    "resourceId": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group>/PROVIDERS/MICROSOFT.APIMANAGEMENT/SERVICE/APIM-HELLO-WORLD"
}
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Visualización de las métricas de las API
> * Configuración de una regla de alerta 
> * Visualización de registros de actividad
> * Habilitamiento y visualización de registros de recursos


Avance hasta el siguiente tutorial:

> [!div class="nextstepaction"]
> [Seguimiento de llamadas](api-management-howto-api-inspector.md)