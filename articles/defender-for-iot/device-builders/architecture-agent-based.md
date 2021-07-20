---
title: Qué es una arquitectura de solución basada en agente
description: Obtenga información sobre el flujo de información y la arquitectura con agente de Azure Defender para IoT.
ms.topic: overview
ms.date: 1/25/2021
ms.openlocfilehash: e21dc7c7b0c82378d90335e4d4ceb71373a56267
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/29/2021
ms.locfileid: "113011475"
---
# <a name="what-is-agent-based-solution-for-device-builders"></a>Qué es una solución basada en agente para los generadores de dispositivos

En este artículo se describe la arquitectura del sistema funcional de la solución con agente Azure Defender para IoT. Azure Defender para IoT ofrece dos conjuntos de funcionalidades que se adaptan a las necesidades de su entorno: la solución sin agente para organizaciones y la solución basada en agente para los generadores de dispositivos.

## <a name="iot-hub-built-in-security"></a>Seguridad integrada de IoT Hub

Defender para IoT está habilitado de forma predeterminada en todas las instancias de IoT Hub que se crean. Defender para IoT proporciona supervisión, recomendaciones y alertas en tiempo real, y todo ello sin que sea necesario instalar el agente en ningún dispositivo y usa análisis avanzado en los metadatos de IoT Hub registrados para analizar y proteger los dispositivos de campo y los centros de IoT. 

## <a name="defender-for-iot-micro-agent"></a>Microagente de Defender para IoT 

El microagente de Defender para IoT proporciona protección de la seguridad en profundidad y visibilidad del comportamiento de los dispositivos. Recopila, agrega y analiza eventos de seguridad sin procesar de los dispositivos. Los eventos de seguridad sin procesar pueden incluir las conexiones de IP, la creación de procesos, los inicios de sesión de los usuarios y otra información relacionada con la seguridad. Además, los agentes de dispositivo de Azure Defender para IoT controlan la agregación de eventos para ayudar a evitar un alta tasa de rendimiento de la red. Los agentes son altamente personalizables, lo que le permite usarlos para tareas específicas, como enviar solo información importante a los SLA más rápidos, o para agregar información de seguridad exhaustiva y contexto en segmentos más grandes, lo que evita mayores costos de servicio.

Los agentes de dispositivos y otra aplicaciones usan el **SDK de Send Security Message de Azure** para enviar información de seguridad a Azure IoT Hub. IoT Hub obtiene esta información y la reenvía al servicio de Defender para IoT.

Una vez que el servicio de Defender para IoT está habilitado, además de los datos reenviados, IoT Hub envía también todos los datos internos para el análisis por parte de Defender para IoT. Estos datos incluyen los registros de operaciones de dispositivos a la nube, las identidades de los dispositivos y la configuración del Hub. Toda esta información ayuda a crear la canalización de análisis de Defender para IoT.

La canalización del análisis de Defender para IoT también recibe otros flujos de inteligencia sobre amenazas de varios orígenes tanto desde dentro de Microsoft como desde los asociados de Microsoft. La canalización de análisis completa de Defender para IoT funciona con cada configuración del cliente realizada en el servicio (como alertas personalizadas y uso del SDK de Send Security Message).

Con la canalización de análisis, Defender para IoT combina todos los flujos de información para generar alertas y recomendaciones viables. La canalización contiene reglas personalizadas creadas tanto por investigadores de seguridad como por expertos, así como modelos de Machine Learning en busca de desviaciones del comportamiento estándar del dispositivo y análisis de riesgos.

Las recomendaciones y alertas de Defender para IoT (salida de la canalización de análisis) se escriben en el área de trabajo de Log Analytics de cada cliente. La inclusión de los eventos sin procesar en el área de trabajo, así como las alertas y recomendaciones permite realizar investigaciones y consultas en profundidad utilizando los detalles exactos de las actividades sospechosas detectadas.

:::image type="content" source="media/architecture/micro-agent-architecture.png" alt-text="La arquitectura de micro agente.":::

## <a name="next-steps"></a>Pasos siguientes

Consulte [Preguntas más frecuentes sobre el agente de Azure Defender para IoT](resources-agent-frequently-asked-questions.md).
