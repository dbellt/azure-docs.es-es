---
title: Introducción a Azure IoT Hub | Microsoft Docs
description: En este artículo se explican los usos de Azure IoT Hub. Le permite leer datos de forma escalable y administrar dispositivos de forma segura.
author: robinsh
ms.author: robinsh
ms.date: 05/03/2021
ms.topic: overview
ms.custom:
- mvc
- amqp
- mqtt
- 'role: Direction'
- 'role: System Architecture'
- contentperf:fy21q3
ms.service: iot-hub
services: iot-hub
ms.openlocfilehash: 95247a1485bac7282ffe95ddcc2ddd37883738dd
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109783827"
---
# <a name="what-is-azure-iot-hub"></a>¿Qué es Azure IoT Hub?

IoT Hub es un servicio administrado, hospedado en la nube, que actúa como centro de mensajes para comunicaciones bidireccionales entre una aplicación de IoT y los dispositivos conectados. Puede conectar millones de dispositivos y sus soluciones de back-end con confianza y de forma segura. La mayoría de dispositivos se pueden conectar a un centro de IoT. 

Se admiten varios patrones de mensajería, como telemetría del dispositivo a la nube, carga de archivos desde dispositivos y métodos de solicitud-respuesta para controlar los dispositivos desde la nube. IoT Hub también admite la supervisión para ayudarle a realizar un seguimiento de la creación de dispositivos, la conexión de dispositivos y los errores de los dispositivos.

Las funcionalidades de IoT Hub le ayudan a compilar soluciones de IoT escalables y completas, como la administración de equipos industriales que se usan en fabricación, el seguimiento de activos valiosos en el sector sanitario y la supervisión de edificios de oficinas.

## <a name="scale-your-solution"></a>Escalado de la solución

IoT Hub escala a millones de dispositivos conectados de manera simultánea y a millones de eventos por segundo para admitir las cargas de trabajo de IoT. Para obtener más información sobre cómo escalar el IoT Hub, vea [Escalado de IOT Hub](iot-hub-scaling.md). Para más información sobre los distintos niveles de servicio ofrecidos por IoT Hub y cómo ajustarlos mejor a sus necesidades de escalabilidad, consulte la [página de precios](https://azure.microsoft.com/pricing/details/iot-hub/).

## <a name="secure-your-communications"></a>Proteger las comunicaciones

Puede enviar datos de forma segura mediante IoT Hub.

* La autenticación por dispositivo permite que cada dispositivo se conecte a IoT Hub y se administre de manera segura.

* Tiene todo el control sobre el acceso de los dispositivos y puede controlar las conexiones por dispositivo.

* [IoT Hub Device Provisioning](../iot-dps/index.yml) aprovisiona automáticamente dispositivos en IoT Hub de manera correcta cuando el dispositivo se inicia por primera vez.

* Varios tipos de autenticación son compatibles con varias funcionalidades de dispositivos:

  * La autenticación basada en token de SAS permite empezar rápidamente con la solución de IoT.

  * La autenticación de certificado X.509 individual está disponible para una autenticación segura basada en estándares.

  * La autenticación CA X.509 se puede utilizar para una inscripción básica basada en estándares.

## <a name="route-device-data"></a>Enrutar datos del dispositivo

La funcionalidad integrada de redirección de mensajes le ofrece flexibilidad para configurar distribuciones ramificadas de mensajes basadas en reglas de manera automática:

* El [enrutamiento de mensajes](iot-hub-devguide-messages-d2c.md) se utiliza para controlar dónde envía el centro los datos de telemetría del dispositivo.

* Enrutar mensajes a varios puntos de conexión no tiene ningún costo adicional.

* Las reglas de enrutamiento se pueden configurar para dirigir automáticamente los mensajes en función de su contenido sin tener que escribir código.

## <a name="integrate-with-other-services"></a>Integración con otros servicios

Puede integrar IoT Hub con otros servicios de Azure para compilar soluciones completas de un extremo a otro. Por ejemplo, use:

* [Azure Event Grid](../event-grid/index.yml) para permitir que la empresa responda rápidamente a eventos críticos de forma confiable, escalable y segura.

* [Azure Logic Apps](../logic-apps/index.yml) para automatizar procesos de negocio.

* [Azure Machine Learning](iot-hub-weather-forecast-machine-learning.md) para agregar aprendizaje automático y modelos de AI a la solución.

* [Azure Stream Analytics](../stream-analytics/index.yml) para ejecutar cálculos de análisis en tiempo real en los flujos de datos de los dispositivos.

## <a name="configure-and-control-your-devices"></a>Configurar y controlar dispositivos

Puede administrar los dispositivos conectados a IoT Hub con una matriz de funcionalidades integradas.

* Almacene, sincronice y consulte metadatos e información de estado de todos los dispositivos.

* Establezca el estado de los dispositivos por dispositivo o en función de características comunes de los dispositivos.

* Responda automáticamente a un cambio de estado notificado por el dispositivo con la integración de enrutamiento de mensajes.

## <a name="make-your-solution-highly-available"></a>Hacer que la solución tenga alta disponibilidad

El [Acuerdo de Nivel de Servicio de IoT Hub](https://azure.microsoft.com/support/legal/sla/iot-hub/) es del 99,9 %. En el [SLA de Azure](https://azure.microsoft.com/support/legal/sla/) completo se explica la disponibilidad garantizada de Azure como un conjunto.

## <a name="connect-your-devices"></a>Conectar dispositivos

Use las bibliotecas de [SDK de dispositivo IoT de Azure](./iot-hub-devguide-sdks.md) bibliotecas para compilar aplicaciones que se ejecutan en los dispositivos e interactúan con IoT Hub. Las plataformas compatibles incluyen varias distribuciones de Linux, Windows y sistemas operativos en tiempo real. Los lenguajes compatibles incluyen:

* C
* C insertado
* C#
* Java
* Python
* Node.js.

IoT Hub y los SDK de dispositivo admiten los siguientes protocolos de conexión de dispositivos:

* HTTPS
* AMQP
* AMQP sobre WebSockets
* MQTT
* MQTT sobre WebSockets

IoT Hub y los SDK de dispositivo admiten las convenciones de [Azure IoT Plug and Play](../iot-pnp/overview-iot-plug-and-play.md) para conectar dispositivos. Los dispositivos IoT Plug and Play usan un modelo de dispositivo para anunciar su funcionalidad a las aplicaciones habilitadas para IoT Plug and Play. El modelo del dispositivo permite a los creadores de soluciones integrar dispositivos inteligentes con sus soluciones sin necesidad de configuración manual.

Si la solución no puede usar las bibliotecas de dispositivos, los dispositivos pueden usar los protocolos MQTT v3.1.1, HTTPS 1.1 o AMQP 1.0 para conectarse de manera nativa al centro.

Si la solución no puede usar ninguno de los protocolos compatibles, puede ampliar IoT Hub para admitir protocolos personalizados:

* Use [Azure IoT Edge](../iot-edge/index.yml) para crear una puerta de enlace de campo para realizar la traslación de protocolos en dispositivos perimetrales.

* Personalice la [puerta de enlace de protocolo de Azure IoT](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md) para realizar la traslación de protocolos en la nube.

## <a name="quotas-and-limits"></a>Cuotas y límites

Las suscripciones de Azure disponen de límites de cuota predeterminados para evitar el uso abusivo del servicio. Estos límites podrían afectar el ámbito de su solución IoT. El límite actual es de 50 de centros de IoT Hub por suscripción. Puede solicitar un aumento de la cuota si se pone en contacto con el soporte técnico. Para obtener más información, consulte las [cuotas y limitaciones de IoT Hub](iot-hub-devguide-quotas-throttling.md). Para obtener más información sobre los límites de cuota, vea alguno de los siguientes artículos:

* [Límites de servicio de suscripción de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md)

* [IoT Hub throttling and you](https://azure.microsoft.com/blog/iot-hub-throttling-and-you/) (Limitación de IoT Hub)

## <a name="iot-hub-on-azure-stack-hub-preview"></a>IoT Hub en Azure Stack Hub (versión preliminar)

IoT Hub en Azure Stack Hub (versión preliminar) permite crear soluciones de IoT híbridas. IoT Hub es un servicio administrado que actúa como centro de conectividad de mensajes para las comunicaciones bidireccionales entre la aplicación de IoT y los dispositivos que administra. Puede usar IoT Hub en Azure Stack Hub para compilar soluciones de IoT con comunicaciones confiables y seguras entre los dispositivos IoT y las soluciones del entorno local.

IoT Hub en Azure Stack Hub es gratuito durante la versión preliminar pública. Para más información, consulte [Introducción a IoT Hub en Azure Stack Hub](/azure-stack/operator/iot-hub-rp-overview).

## <a name="next-steps"></a>Pasos siguientes

Para probar una solución de IoT de un extremo a otro, consulte los siguientes inicios rápidos de IoT Hub:

* [Quickstart: Send telemetry from a device to an IoT hub](quickstart-send-telemetry-node.md) (Inicio rápido: enviar telemetría desde un dispositivo a IoT Hub)

Para más información sobre las distintas formas en que puede compilar e implementar soluciones de IoT con IoT de Azure, visite:

* [Aspectos básicos: Soluciones y tecnologías de IoT de Azure](../iot-fundamentals/iot-services-and-technologies.md).