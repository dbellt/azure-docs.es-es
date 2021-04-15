---
title: Métricas de Azure Event Hubs en Azure Monitor | Microsoft Docs
description: En este artículo se proporciona información sobre cómo usar Azure Monitoring para supervisar Azure Event Hubs.
ms.topic: article
ms.date: 02/25/2021
ms.openlocfilehash: 9b31ed9e8bea67367533813f20265761b77ae371
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107307357"
---
# <a name="azure-event-hubs-metrics-in-azure-monitor"></a>Métricas de Azure Event Hubs en Azure Monitor

Las métricas de Event Hubs le permiten conocer el estado de los recursos de Event Hubs en su suscripción de Azure. Con un amplio conjunto de datos de métricas, puede evaluar el estado general de los centros de eventos, no solo en el nivel de espacio de nombres, sino también en el nivel de entidad. Estas estadísticas pueden ser importantes, ya que le ayudan a supervisar el estado de los centros de eventos. Las métricas también pueden ayudarle a solucionar problemas de causa principal sin necesidad de ponerse en contacto con el soporte técnico de Azure.

Azure Monitor proporciona interfaces de usuario unificadas para la supervisión de distintos servicios de Azure. Para obtener más información, vea [Supervisión de Microsoft Azure](../azure-monitor/overview.md) y el ejemplo [Retrieve Azure Monitor metrics with .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) (Recuperación de métricas de Azure Monitor con .NET) en GitHub.

## <a name="access-metrics"></a>Acceso a la métrica

Azure Monitor proporciona varias maneras de tener acceso a las métricas. Puede acceder a las métricas desde [Azure Portal](https://portal.azure.com), o usar las API de Azure Monitor (REST y .NET) y soluciones de análisis como Log Analytics y Event Hubs. Para más información, vea [Datos de supervisión recopilados por Azure Monitor](../azure-monitor/data-platform.md).

De forma predeterminada, las métricas están habilitadas y puede acceder a datos de los últimos 30 días. Si es necesario conservar los datos durante un período mayor, se pueden archivar los datos de las métricas en una cuenta de Azure Storage. Este valor se establece en la [configuración de diagnóstico](../azure-monitor/essentials/diagnostic-settings.md) de Azure Monitor.


## <a name="access-metrics-in-the-portal"></a>Acceso a métricas del portal

Una vez transcurrido un tiempo, las métricas se pueden supervisar en [Azure Portal](https://portal.azure.com). En el ejemplo siguiente se muestra cómo ver las solicitudes correctas y las solicitudes entrantes en el nivel de cuenta:

![Ver las métricas de éxito][1]

También puede tener acceso a las métricas directamente a través del espacio de nombres. Para ello, seleccione el espacio de nombres y, después, seleccione **Métricas**. Para mostrar las métricas filtradas al ámbito del centro de eventos, seleccione el centro de eventos y, después, seleccione **Métricas**.

Para ver métricas que admitan las dimensiones, debe filtrar por el valor de la dimensión que desee como se muestra en el siguiente ejemplo:

![Filtrar por valor de dimensión][2]

## <a name="billing"></a>Facturación

Actualmente, el uso de métricas en Azure Monitor es gratuito. Sin embargo, si usa otras soluciones que ingieren datos de métricas, puede que se le facturen dichas soluciones. Por ejemplo, se le facturará por Azure Storage si archiva datos de métricas en una cuenta de Azure Storage. También se le facturará por Azure si transmite datos de métricas a registros de Azure Monitor para realizar análisis avanzados.

Las siguientes métricas ofrecen una visión general del estado de su servicio. 

> [!NOTE]
> Se están dejando de usar algunas métricas a medida que se mueven a otro nombre. Esto podría requerir que actualice las referencias. Las métricas que se marquen con la palabra clave "en desuso" no se admitirán en el futuro.

Los valores de las métricas se envían a Azure Monitor cada minuto. La granularidad de tiempo define el intervalo de tiempo para el que se presentan los valores de las métricas. El intervalo de tiempo compatible para todas las métricas de Event Hubs es 1 minuto.

## <a name="azure-event-hubs-metrics"></a>Métricas de Azure Event Hubs
Para obtener una lista de las métricas admitidas por el servicio, consulte [Azure Event Hubs](../azure-monitor/essentials/metrics-supported.md#microsofteventhubnamespaces).

> [!NOTE]
> Cuando se produce un error de usuario, Azure Event Hubs actualiza la métrica **Errores de usuario**, pero no registra ninguna otra información de diagnóstico. Por lo tanto, debe capturar los detalles de los errores de usuario en las aplicaciones. También puede convertir los datos de telemetría generados cuando se envían o se reciben mensajes en Application Insights. Para obtener un ejemplo, consulte [Realizar un seguimiento con Application Insights](../service-bus-messaging/service-bus-end-to-end-tracing.md#tracking-with-azure-application-insights).

## <a name="azure-monitor-integration-with-siem-tools"></a>Integración de Azure Monitor con las herramientas SIEM
El enrutamiento de los datos de supervisión (registros de actividad, registros de diagnóstico, etc.) a un centro de eventos con Azure Monitor permite conseguir una integración sencilla con las herramientas de Administración de eventos e información de seguridad (SIEM). Para más información, consulte los siguientes artículos o entradas de blog:

- [Flujo de datos de supervisión de Azure a un centro de eventos para que lo consuma una herramienta externa](../azure-monitor/essentials/stream-monitoring-data-event-hubs.md)
- [Introducción a Azure Log Integration](/previous-versions/azure/security/fundamentals/azure-log-integration-overview)
- [Uso de Azure Monitor para su integración con herramientas SIEM](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

En el escenario en el que una herramienta SIEM consume datos de registro de un centro de eventos, si no ve ningún mensaje entrante o si ve mensajes entrantes pero no hay mensajes salientes en el gráfico de métricas, siga estos pasos:

- Si **no hay mensajes entrantes**, significa que el servicio Azure Monitor no está moviendo los registros de auditoría o diagnóstico al centro de eventos. En este caso, abra una incidencia de soporte técnico con el equipo de Azure Monitor. 
- Si hay mensajes entrantes, pero **no hay mensajes salientes**, significa que la aplicación SIEM no está leyendo los mensajes. Póngase en contacto con el proveedor de SIEM para determinar si la configuración del centro de eventos en esas aplicaciones es correcta.


## <a name="next-steps"></a>Pasos siguientes

* Vea [Información general sobre la supervisión en Microsoft Azure](../azure-monitor/overview.md).
* Ejemplo [Retrieve Azure Monitor metrics with .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) (Recuperación de métricas de Azure Monitor con .NET) en GitHub. 

Para obtener más información acerca de Event Hubs, visite los vínculos siguientes:

- Empiece a trabajar con un tutorial de Event Hubs
    - [.NET Core](event-hubs-dotnet-standard-getstarted-send.md)
    - [Java](event-hubs-java-get-started-send.md)
    - [Python](event-hubs-python-get-started-send.md)
    - [JavaScript](event-hubs-node-get-started-send.md)
* [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.yml)
* [Aplicaciones de ejemplo que usan Event Hubs](https://github.com/Azure/azure-event-hubs/tree/master/samples)

[1]: ./media/event-hubs-metrics-azure-monitor/event-hubs-monitor1.png
[2]: ./media/event-hubs-metrics-azure-monitor/event-hubs-monitor2.png
