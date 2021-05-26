---
title: Visualización de métricas con Azure Monitor
titleSuffix: Azure Digital Twins
description: Consulte cómo ver las métricas de Azure Digital Twins en Azure Monitor.
author: baanders
ms.author: baanders
ms.date: 8/4/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: ffae474f6de5de7715585f0606a9630b136a2c37
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110092141"
---
# <a name="troubleshooting-azure-digital-twins-metrics"></a>Solución de problemas de Azure Digital Twins: Métricas

Las métricas descritas en este artículo ofrecen información sobre el estado de los recursos de Azure Digital Twins en la suscripción de Azure. Las métricas de Azure Digital Twins le ayudan a evaluar el estado general del servicio Azure Digital Twins y los recursos conectados a él. Estas estadísticas orientadas al usuario ayudan a ver lo que está ocurriendo con su instancia de Azure Digital Twins y a realizar el análisis de la causa raíz de los problemas sin necesidad de ponerse en contacto con el soporte técnico de Azure.

Las métricas están habilitadas de forma predeterminada. Puede ver las métricas de Azure Digital Twins desde [Azure Portal](https://portal.azure.com).

## <a name="how-to-view-azure-digital-twins-metrics"></a>Procedimiento para ver las métricas de Azure Digital Twins

1. Cree una instancia de Azure Digital Twins. Puede encontrar instrucciones sobre cómo configurar una instancia de Azure Digital Twins en [Procedimiento: Configuración de una instancia y autenticación](how-to-set-up-instance-portal.md).

2. Busque la instancia de Azure Digital Twins en [Azure Portal](https://portal.azure.com) (puede abrir la página correspondiente escribiendo su nombre en la barra de búsqueda del portal). 

    En el menú de la instancia, seleccione **Métricas**.
   
    :::image type="content" source="media/troubleshoot-metrics/azure-digital-twins-metrics.png" alt-text="Captura de pantalla que muestra la página de métricas de Azure Digital Twins":::

    En esta página se muestran las métricas de la instancia de Azure Digital Twins. También puede crear vistas personalizadas de las métricas seleccionando las que quiera ver en la lista.
    
3. Para elegir entre enviar los datos de las métricas a un punto de conexión de Event Hubs o a una cuenta de Azure Storage, seleccione **Configuración de diagnóstico** en el menú y, luego, **Agregar configuración de diagnóstico**.

    :::image type="content" source="media/troubleshoot-diagnostics/diagnostic-settings.png" alt-text="Captura de pantalla que muestra la página de configuración de diagnóstico y el botón para agregarla":::

    Para más información sobre este proceso, consulte [Solución de problemas: Configuración de diagnósticos](troubleshoot-diagnostics.md).

4. Para optar por configurar alertas para los datos de métricas, seleccione **Alertas** en el menú y, a continuación, **+ Nueva regla de alerta**.
    :::image type="content" source="media/troubleshoot-alerts/alerts-pre.png" alt-text="Captura de pantalla que muestra la página Alertas y el botón para agregar":::

    Para más información sobre este proceso, consulte [Solución de problemas: Configuración de alertas](troubleshoot-alerts.md).

## <a name="azure-digital-twins-metrics-and-how-to-use-them"></a>Métricas de Azure Digital Twins y cómo usarlas

Azure Digital Twins proporciona varias métricas para ofrecerle una visión general del estado de la instancia y de sus recursos asociados. También puede combinar información de varias métricas para conseguir una imagen más amplia del estado de la instancia. 

En las tablas siguientes se describen las métricas de las que las instancias de Azure Digital Twins realizan un seguimiento y cómo se relaciona cada métrica con el estado general de la instancia.

#### <a name="metrics-for-tracking-service-limits"></a>Métricas para el seguimiento de los límites del servicio

Puede configurar estas métricas para que realicen un seguimiento cuando se acerque al [límite del servicio publicado](reference-service-limits.md#functional-limits) en algún aspecto de la solución. 

Para configurar esta opción, use la característica de [alertas](troubleshoot-alerts.md) de Azure Monitor. Asimismo, puede definir los umbrales de estas métricas para recibir una alerta cuando una métrica alcance un determinado porcentaje de su límite publicado.

| Métrica | Nombre para mostrar de la métrica | Unidad | Tipo de agregación| Descripción | Dimensions |
| --- | --- | --- | --- | --- | --- |
| TwinCount | Recuento de gemelos (versión preliminar) | Count | Total | Número total de gemelos de una instancia de Azure Digital Twins. Use esta métrica para determinar si se está aproximando al [límite del servicio](reference-service-limits.md#functional-limits) para el número máximo de gemelos permitido por instancia. |  None |
| ModelCount | Recuento de modelos (versión preliminar) | Count | Total | Número total de modelos de una instancia de Azure Digital Twins. Use esta métrica para determinar si se está aproximando al [límite del servicio](reference-service-limits.md#functional-limits) para el número máximo de modelos permitido por instancia. | None |

#### <a name="api-request-metrics"></a>Métricas de solicitud de API

Métricas relacionadas con las solicitudes de API:

| Métrica | Nombre para mostrar de la métrica | Unidad | Tipo de agregación| Descripción | Dimensions |
| --- | --- | --- | --- | --- | --- |
| ApiRequests | Solicitudes de API | Count | Total | Número de solicitudes de API realizadas para las operaciones de lectura, escritura, eliminación y consulta de Digital Twins. |  Autenticación, <br>operación, <br>protocolo, <br>código de estado, <br>clase de código de estado, <br>Texto de estado |
| ApiRequestsFailureRate | Tasa de errores de solicitudes de API | Percent | Average | Porcentaje de solicitudes de API que el servicio recibe para la instancia que proporciona un código de respuesta de error interno (500) para las operaciones de lectura, escritura, eliminación y consulta de Digital Twins. | Autenticación, <br>operación, <br>protocolo, <br>código de estado, <br>clase de código de estado, <br>Texto de estado
| ApiRequestsLatency | Latencia de solicitudes de API | Milisegundos | Average | Tiempo de respuesta de las solicitudes de API. Hace referencia a la hora desde la que Azure Digital Twins recibe la solicitud hasta que el servicio envía un resultado de éxito o error para las operaciones de lectura, escritura, eliminación y consulta Digital Twins. | Autenticación, <br>operación, <br>Protocolo |

#### <a name="billing-metrics"></a>Métricas de facturación

Métricas relacionadas con la facturación:

| Métrica | Nombre para mostrar de la métrica | Unidad | Tipo de agregación| Descripción | Dimensions |
| --- | --- | --- | --- | --- | --- |
| BillingApiOperations | Operaciones de la API de facturación | Count | Total | Métrica de facturación para el recuento de todas las solicitudes de API realizadas en el servicio Azure Digital Twins. | Meter ID |
| BillingMessagesProcessed | Mensajes de facturación procesados | Count | Total | Métrica de facturación para el número de mensajes enviados desde Azure Digital Twins a puntos de conexión externos.<br><br>Para que se considere un solo mensaje para su facturación, una carga no debe ser superior a 1 KB. Las cargas superiores a este se contarán como mensajes adicionales en incrementos de 1 KB (por lo que un mensaje de entre 1 y 2 KB se contará como dos mensajes, entre 2 y 3 KB serán tres mensajes, etc.).<br>Esta restricción también se aplica a las respuestas, por lo que una llamada que devuelve 1,5 KB en el cuerpo de la respuesta, por ejemplo, se facturará como dos operaciones. | Meter ID |
| BillingQueryUnits | Facturación de unidades de consulta | Count | Total | Número de unidades de consulta, una medida del uso de recursos de servicio calculada internamente, que se consume para ejecutar consultas. También hay una API auxiliar disponible para medir las unidades de consulta: [Clase de QueryChargeHelper](/dotnet/api/azure.digitaltwins.core.querychargehelper?view=azure-dotnet&preserve-view=true) | Meter ID |

Para obtener más información sobre la forma en que se factura Azure Digital Twins, consulte [Precios de Azure Digital Twins](https://azure.microsoft.com/pricing/details/digital-twins/).

#### <a name="ingress-metrics"></a>Métricas de entrada

Métricas relacionadas con la entrada de datos:

| Métrica | Nombre para mostrar de la métrica | Unidad | Tipo de agregación| Descripción | Dimensions |
| --- | --- | --- | --- | --- | --- |
| IngressEvents | Eventos de entrada | Count | Total | Número de eventos de telemetría entrantes en Azure Digital Twins. | Resultado |
| IngressEventsFailureRate | Tasa de errores de los eventos de entrada | Percent | Average | Porcentaje de eventos de telemetría entrantes para los que el servicio devuelve un código de respuesta de error interno (500). | Resultado |
| IngressEventsLatency | Latencia de eventos de entrada | Milisegundos | Average | Hora a partir de la cual llega un evento cuando está listo para que Azure Digital Twins lo haga salir, momento en el que el servicio envía un resultado correcto o erróneo. | Resultado |

#### <a name="routing-metrics"></a>Métricas de enrutamiento

Métricas relacionadas con el enrutamiento:

| Métrica | Nombre para mostrar de la métrica | Unidad | Tipo de agregación| Descripción | Dimensions |
| --- | --- | --- | --- | --- | --- |
| MessagesRouted | Mensajes enrutados | Count | Total | Número de mensajes que se enrutan a un servicio de Azure de punto de conexión, como Event Hubs, Service Bus o Event Grid. | Tipo de punto de conexión <br>Resultado |
| RoutingFailureRate | Tasa de errores de enrutamiento | Percent | Average | Porcentaje de eventos que producen un error a medida que se enrutan desde Azure Digital Twins a un servicio de Azure de punto de conexión, como Event Hubs, Service Bus o Event Grid. | Tipo de punto de conexión <br>Resultado |
| RoutingLatency | Latencia de enrutamiento | Milisegundos | Average | Tiempo transcurrido entre el enrutamiento de un evento desde Azure Digital Twins hasta su publicación en el servicio de Azure de punto de conexión, como Event Hubs, Service Bus o Event Grid. | Tipo de punto de conexión <br>Resultado |

## <a name="dimensions"></a>Dimensions

Las dimensiones ayudan a identificar más detalles sobre las métricas. Algunas de las métricas de enrutamiento proporcionan información por punto de conexión. En la tabla siguiente se enumeran los valores posibles para estas dimensiones.

| Dimensión | Valores |
| --- | --- |
| Autenticación | OAuth |
| Operación (para las solicitudes de API) | Microsoft.DigitalTwins/digitaltwins/delete, <br>Microsoft.DigitalTwins/digitaltwins/write, <br>Microsoft.DigitalTwins/digitaltwins/read, <br>Microsoft.DigitalTwins/eventroutes/read, <br>Microsoft.DigitalTwins/eventroutes/write, <br>Microsoft.DigitalTwins/eventroutes/delete, <br>Microsoft.DigitalTwins/models/read, <br>Microsoft.DigitalTwins/models/write, <br>Microsoft.DigitalTwins/models/delete, <br>Microsoft.DigitalTwins/query/action |
| Tipo de punto de conexión | Event Grid, <br>Event Hub, <br>Service Bus |
| Protocolo | HTTPS |
| Resultado | Correcto, <br>Error |
| Código de estado | 200, 404, 500, etc. |
| Clase de código de estado | 2xx, 4xx, 5xx, etc. |
| Texto de estado | Error interno del servidor, No encontrado, etc. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la administración de las métricas registradas para Digital Twins, consulte [Solución de problemas: Configuración de diagnósticos](troubleshoot-diagnostics.md).
