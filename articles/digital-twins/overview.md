---
title: ¿Qué es Azure Digital Twins?
titleSuffix: Azure Digital Twins
description: Información general sobre lo que se puede hacer con Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 3/12/2020
ms.topic: overview
ms.service: digital-twins
ms.openlocfilehash: a64564887fa2738fd3c0586d1040be3c95193927
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108205798"
---
# <a name="what-is-azure-digital-twins"></a>¿Qué es Azure Digital Twins?

**Azure Digital Twins** es una oferta de plataforma como servicio (PaaS) que permite la creación de grafos de conocimientos basados en modelos digitales de entornos completos. Estos entornos podrían ser edificios, fábricas, granjas, redes energéticas, ferrocarriles, estadios, etc., e incluso ciudades enteras. Estos modelos digitales se pueden usar para obtener información que impulse mejores productos, operaciones optimizadas, costos reducidos y experiencias de cliente innovadoras.

Aproveche su experiencia en este campo además de Azure Digital Twins para crear soluciones personalizadas y conectadas capaces de:
* Modelar cualquier entorno y hacer que las gemelos digitales cobren vida de una manera escalable y segura.
* Conectar recursos como dispositivos IoT y sistemas empresariales existentes.
* Usar un sólido sistema de eventos para crear lógica de negocios y procesamiento de datos dinámicos.
* Realizar la integración con los servicios de datos, análisis e inteligencia artificial de Azure para facilitar el seguimiento del pasado y predecir el futuro.

## <a name="azure-digital-twins-capabilities"></a>Funcionalidades de Azure Digital Twins

Este es un resumen de las características proporcionadas por Azure Digital Twins.

### <a name="open-modeling-language"></a>Lenguaje de modelado abierto

En Azure Digital Twins, se definen las entidades digitales que representan las personas, los lugares y las cosas del entorno físico mediante tipos de gemelos personalizados denominados [modelos](concepts-models.md). 

Puede pensar en estas definiciones de modelo como un vocabulario especializado para describir su negocio. Para una solución de administración de edificios, por ejemplo, puede definir modelos como "edificio", "planta" y "ascensor". Después, puede crear **gemelos digitales** basados en estos modelos para representar su entorno específico.

[!INCLUDE [digital-twins-versus-device-twins](../../includes/digital-twins-versus-device-twins.md)]

Los modelos se definen en un lenguaje similar a JSON denominado [lenguaje de definición de gemelos digitales (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md), y describen gemelos en términos de sus propiedades de estado, eventos de telemetría, comandos, componentes y relaciones.
* Los modelos definen **relaciones semánticas** entre las entidades para que pueda conectar su gemelos a un gráfico de información que refleje sus interacciones. Puede considerar los modelos como nombres en una descripción de su mundo y las relaciones como verbos.
* También puede especializar gemelos mediante la herencia de modelos. Un modelo puede heredar de otro.

DTDL se utiliza para los modelos de datos en otros servicios de Azure IoT, como [IoT Plug and Play (PnP)](../iot-pnp/overview-iot-plug-and-play.md) y [Time Series Insights (TSI)](../time-series-insights/overview-what-is-tsi.md). Esto le ayuda a mantener la solución Azure Digital Twins conectada y compatible con otras partes del ecosistema de Azure.

### <a name="live-execution-environment"></a>Entorno de ejecución dinámico

Los modelos digitales de Azure Digital Twins son representaciones dinámicas actualizadas del mundo real. Mediante el uso de las relaciones de los modelos de DTDL personalizados, conectará gemelos a un **gráfico dinámico** que representa el entorno.

Puede tener una visualización del gráfico de Azure Digital Twins con la ayuda de una aplicación de ejemplo, el [explorador de Azure Digital Twins](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/).

Así es la visualización de ejemplo:

:::image type="content" source="media/includes/azure-digital-twins-explorer.png" alt-text="Captura de pantalla de la aplicación de ejemplo Explorador de Azure Digital Twins, que muestra un gráfico de nodos que representan gemelos digitales" lightbox="media/includes/azure-digital-twins-explorer.png":::

Azure Digital Twins proporciona un **sistema de eventos** completo para mantener ese gráfico actualizado con la lógica de negocios y el procesamiento de datos. Puede conectar recursos de proceso externos, como [Azure Functions](../azure-functions/functions-overview.md), para impulsar este procesamiento de datos de maneras flexibles y personalizadas.

También puede extraer información del entorno de ejecución dinámico mediante la eficaz **API de consultas** de Azure Digital Twins. La API le permite realizar consultas con condiciones de búsqueda abundantes, que incluyen valores de propiedades, relaciones, propiedades de relaciones, información del modelo, etc. También puede combinar consultas mediante la recopilación de una gran variedad de datos sobre su entorno y la respuesta a preguntas personalizadas que son importantes para usted.

### <a name="input-from-iot-and-business-systems"></a>Entrada de sistemas empresariales y de IoT

Para mantener el entorno de ejecución dinámico de Azure Digital Twins actualizado con el mundo real, puede usar [IoT Hub](../iot-hub/about-iot-hub.md) para conectar la solución a los dispositivos IoT e IoT Edge. Estos dispositivos administrados por el centro de conectividad se representan como parte del grafo de gemelos y proporcionan los datos que impulsan el modelo.

Puede crear una nueva instancia de IoT Hub para este propósito con Azure Digital Twins, o bien conectar una instancia de IoT Hub existente junto con los dispositivos que ya administra.

También puede impulsar Azure Digital Twins desde otros orígenes de datos mediante las API de REST o conectores a otros servicios como [Logic Apps](../logic-apps/logic-apps-overview.md).

### <a name="output-to-tsi-storage-and-analytics"></a>Salida a TSI, almacenamiento y análisis

Los datos del modelo de Azure Digital Twins se pueden enrutar a servicios de Azure de nivel inferior para operaciones de análisis o almacenamiento adicionales. Esto se proporciona a través de **rutas de evento**, que usan el [centro de eventos](../event-hubs/event-hubs-about.md), [Event Grid](../event-grid/overview.md) o [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) para impulsar los flujos de datos deseados.

Algunas de las cosas que puede hacer con las rutas de eventos son:
* Almacenar datos de Azure Digital Twins en [Azure Data Lake](../storage/blobs/data-lake-storage-introduction.md).
* Analizar datos de Azure Digital Twins con [Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) u otras herramientas de análisis de datos de Microsoft.
* Integrar flujos de trabajo más grandes con Logic Apps.
* Conectar Azure Digital Twins a Time Series Insights para realizar un seguimiento del historial de series temporales de cada gemelo.
* Alinear un modelo de serie temporal en Time Series Insights con un origen en Azure Digital Twins.

Esta es otra manera de que Azure Digital Twins pueda conectarse a una solución más grande y respaldar sus necesidades personalizadas de continuar trabajando con esta información.

## <a name="azure-digital-twins-in-a-solution-context"></a>Azure Digital Twins en un contexto de la solución

Azure Digital Twins se usa normalmente en combinación con otros servicios de Azure como parte de una solución de IoT más grande. 

Una solución completa con Azure Digital Twins puede contener las siguientes partes:
* La instancia de servicio de Azure Digital Twins. Almacena los modelos gemelos y el grafo de gemelos con su estado, y dirige el procesamiento de eventos.
* Una o varias aplicaciones cliente que controlan la instancia de Azure Digital Twins mediante la configuración de modelos, la creación de topologías y la extracción de información del grafo de gemelos.
* Uno o más recursos de proceso externos para procesar eventos generados por Azure Digital Twins u orígenes de datos conectados, como dispositivos. Una forma común de proporcionar recursos de proceso es a través de [Azure Functions](../azure-functions/functions-overview.md).
* Una instancia de IoT Hub para proporcionar funcionalidades de administración de dispositivos y de flujo de datos de IoT.
* Servicios de nivel inferior para controlar tareas como la integración de flujos de trabajo (como [Logic Apps](../logic-apps/logic-apps-overview.md), almacenamiento en frío, integración de series temporales o análisis).

En el diagrama siguiente se muestra dónde se encuentra Azure Digital Twins en el contexto de una solución de Azure IoT más grande.

:::image type="content" source="media/overview/solution-context.png" alt-text="Diagrama que muestra los orígenes de entrada, los servicios de salida y la comunicación bidireccional con las aplicaciones cliente y los recursos de proceso externos." border="false" lightbox="media/overview/solution-context.png":::

## <a name="service-limits"></a>Límites de servicio

Puede leer sobre los **límites de servicio** de Azure Digital Twins aquí: [Límites del servicio Azure Digital Twins](reference-service-limits.md). Esta información puede ser útil al trabajar con el servicio a fin de comprender sus limitaciones funcionales y de velocidad, así como los límites que se pueden ajustar en caso necesario.

## <a name="terminology"></a>Terminología

Puede ver una lista de **términos de IoT comunes** y sus usos en los servicios de Azure IoT, incluido Azure Digital Twins, aquí: [Glosario de Azure IoT](../iot-fundamentals/iot-glossary.md?toc=/azure/digital-twins/toc.json&bc=/azure/digital-twins/breadcrumb/toc.json). Este recurso puede ser una referencia útil cuando empiece a trabajar con Azure Digital Twins y a crear una solución de IoT.

## <a name="next-steps"></a>Pasos siguientes

* Profundice en el uso de Azure Digital Twins en el inicio rápido: [Guía de inicio rápido: Exploración de un escenario de ejemplo](quickstart-azure-digital-twins-explorer.md).

* O bien, empiece a obtener información sobre los conceptos de Azure Digital Twins con [Conceptos: Modelos personalizados](concepts-models.md).