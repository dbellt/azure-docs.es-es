---
title: Integración con otros servicios
titleSuffix: Azure Digital Twins
description: Conozca los requisitos de entrada y salida al implementar Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 3/16/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 024d4db296aa663c74d1058335611b76741108e8
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108209938"
---
# <a name="integrate-azure-digital-twins-with-other-services"></a>Integre Azure Digital Twins con otros servicios

Azure Digital Twins se suele usar junto con otros servicios para crear soluciones flexibles y conectadas que usan los datos de diversas maneras.

Mediante [rutas de eventos](concepts-route-events.md), Azure Digital Twins puede recibir datos de servicios ascendentes, como [IoT Hub](../iot-hub/about-iot-hub.md) o [Logic Apps](../logic-apps/logic-apps-overview.md), que se usan para proporcionar telemetría y notificaciones. 

Azure Digital Twins también puede enrutar datos a servicios de bajada, como [Azure Maps](../azure-maps/about-azure-maps.md) y [Time Series Insights](../time-series-insights/overview-what-is-tsi.md) con fines de almacenamiento, integración del flujo de trabajo, análisis, etc. 

## <a name="data-ingress"></a>Entrada de datos

Azure Digital Twins se puede controlar con datos y eventos de cualquier servicio: [IoT Hub](../iot-hub/about-iot-hub.md), [Logic Apps](../logic-apps/logic-apps-overview.md), su propio servicio personalizado, etc. Esto le permite recopilar datos de telemetría de dispositivos físicos en el entorno y procesarlos mediante el grafo de Azure Digital Twins en la nube.

En lugar de tener una instancia de IoT Hub integrada en segundo plano, Azure Digital Twins le permite "traer su propia" instancia de IoT Hub para usarla con el servicio. Puede usar un centro de IoT Hub existente que tenga actualmente en producción o bien implementar uno nuevo para usarlo con este fin. Así, obtiene acceso completo a todas las funcionalidades de administración de los dispositivos de IoT Hub.

Para ingerir datos de cualquier origen en Azure Digital Twins, use una [función de Azure Functions](../azure-functions/functions-overview.md). Más información sobre este patrón en [Procedimiento: Ingesta de telemetría de IoT Hub](how-to-ingest-iot-hub-data.md) o pruébelo usted mismo en Azure Digital Twins: [Tutorial: Conexión de una solución de un extremo a otro](tutorial-end-to-end.md). 

También puede obtener información sobre cómo conectar Azure Digital Twins a un desencadenador de Logic Apps en [Procedimiento: integración con Logic Apps](how-to-integrate-logic-apps.md).

## <a name="data-egress-services"></a>Servicios de salida de datos

Azure Digital Twins puede enviar datos a los **puntos de conexión** conectados. Los puntos de conexión admitidos pueden ser:
* [Event Hubs](../event-hubs/event-hubs-about.md)
* [Event Grid](../event-grid/overview.md)
* [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)

Los puntos de conexión se asocian a Azure Digital Twins mediante las API de administración o Azure Portal. Obtenga más información sobre cómo adjuntar un punto de conexión a Azure Digital Twins en [Procedimiento: Administración de puntos de conexión y rutas](how-to-manage-routes-apis-cli.md).

Hay muchos otros servicios donde es posible que quiera dirigir los datos en última instancia, como [Azure Storage](../storage/common/storage-introduction.md), [Azure Maps](../azure-maps/about-azure-maps.md) o [Time Series Insights](../time-series-insights/overview-what-is-tsi.md). Para enviar los datos a servicios como estos, asocie el servicio de destino a un punto de conexión.

Por ejemplo, si también usa Azure Maps y desea poner en correlación la ubicación con el [grafo gemelo](concepts-twins-graph.md) de Azure Digital Twins, puede usar Azure Functions con Event Grid para establecer la comunicación entre todos los servicios de la implementación. Más información acerca de esto en [Procedimiento: uso de Azure Digital Twins para actualizar un plano interior de Azure Maps](how-to-integrate-maps.md)

También puede aprender a enrutar los datos de forma similar a Time Series Insights en [Procedimiento: Integración con Time Series Insights](how-to-integrate-time-series-insights.md).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre los puntos de conexión y el enrutamiento de eventos a servicios externos:
* [Conceptos: Enrutamiento de eventos de Azure Digital Twins](concepts-route-events.md)

Vea cómo configurar Azure Digital Twins para ingerir datos de IoT Hub:
* [Procedimiento: Ingesta de telemetría desde IoT Hub](how-to-ingest-iot-hub-data.md)