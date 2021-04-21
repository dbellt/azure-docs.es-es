---
title: Desarrollo de módulos para Azure IoT Edge | Microsoft Docs
description: Desarrollo de módulos personalizados para Azure IoT Edge que pueden comunicarse con el tiempo de ejecución y IoT Hub
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 11/10/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 6274d0f0265220877938775ab6efd72ff22b84e4
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107031138"
---
# <a name="develop-your-own-iot-edge-modules"></a>Desarrollar sus propios módulos de IoT Edge

[!INCLUDE [iot-edge-version-201806-or-202011](../../includes/iot-edge-version-201806-or-202011.md)]

Los módulos de Azure IoT Edge pueden conectarse con otros servicios de Azure y contribuir a la mayor canalización de datos en la nube. En este artículo se describe cómo puede desarrollar módulos para comunicarse con el entorno de ejecución de IoT Edge e IoT Hub y, por lo tanto, el resto de la nube de Azure.

## <a name="iot-edge-runtime-environment"></a>Entorno de tiempo de ejecución de IoT Edge

El entorno de tiempo de ejecución de IoT Edge proporciona la infraestructura para integrar la funcionalidad de varios módulos de IoT Edge e implementarlos en dispositivos IoT Edge. Todos los programas se pueden empaquetar como un módulo de IoT Edge. Para aprovechar al máximo las funcionalidades de comunicación y administración de IoT Edge, un programa que se ejecute en un módulo puede usar el SDK de dispositivos IoT de Azure para conectarse al centro de IoT Edge local.
::: moniker range=">=iotedge-2020-11"
Los módulos también pueden usar cualquier cliente MQTT para conectarse al agente MQTT del centro de IoT Edge local.
::: moniker-end

### <a name="packaging-your-program-as-an-iot-edge-module"></a>Empaquetado del programa como módulo de IoT Edge

Para implementar el programa en un dispositivo IoT Edge, primero debe incluirse en contenedores y ejecutarse con un motor compatible con Docker. IoT Edge usa [Moby](https://github.com/moby/moby), el proyecto de código abierto que subyace a Docker, como motor compatible con Docker. Los mismos parámetros que se usan con Docker se pueden pasar a los módulos de IoT Edge. Para más información, consulte [Configuración de las opciones de creación de contenedores para módulos de IoT Edge](how-to-use-create-options.md).

## <a name="using-the-iot-edge-hub"></a>Uso del centro de IoT Edge

El centro de IoT Edge proporciona dos funcionalidades principales: proxy a IoT Hub y comunicaciones locales.

### <a name="connecting-to-iot-edge-hub-from-a-module"></a>Conexión al centro de IoT Edge desde un módulo

La conexión al centro de IoT Edge local desde un módulo implica los mismos pasos de conexión que para los clientes. Para obtener más información, consulte [Conexión al centro de IoT Edge](iot-edge-runtime.md#connecting-to-the-iot-edge-hub).

Para usar el enrutamiento de IoT Edge a través de AMQP o MQTT, puede usar ModuleClient desde el SDK de Azure IoT. Crear una instancia de ModuleClient para conectar el módulo al centro de IoT Edge que se ejecuta en el dispositivo, de forma similar a cómo las instancias de DeviceClient conectan dispositivos IoT a IoT Hub. Para más información sobre la clase ModuleClient y sus métodos de comunicación, consulte la referencia de API de su lenguaje preferido para el SDK: [C#](/dotnet/api/microsoft.azure.devices.client.moduleclient), [C](/azure/iot-hub/iot-c-sdk-ref/iothub-module-client-h), [Python](/python/api/azure-iot-device/azure.iot.device.iothubmoduleclient), [Java](/java/api/com.microsoft.azure.sdk.iot.device.moduleclient) o [Node.js](/javascript/api/azure-iot-device/moduleclient).

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

Para usar el agente MQTT de IoT Edge, debe traer su propio cliente MQTT e iniciar la conexión usted mismo con la información que recupere de la API de carga de trabajo del demonio de IoT Edge. <!--Need to add details here-->

Para obtener más información sobre cómo elegir entre el enrutamiento o la publicación/suscripción con el agente MQTT, consulte [Comunicación local](iot-edge-runtime.md#local-communication).

### <a name="mqtt-broker-primitives"></a>Primitivos del agente MQTT

#### <a name="send-a-message-on-a-user-defined-topic"></a>Envío de un mensaje en un tema definido por el usuario

Con el agente MQTT de IoT Edge, puede publicar mensajes en cualquier tema definido por el usuario. Para ello, autorice el módulo para que publique en temas concretos y, a continuación, obtenga un token de la API de carga de trabajo para usarlo como contraseña al conectarse al agente MQTT y, por último, publique mensajes en los temas autorizados con el cliente MQTT de su elección.

#### <a name="receive-messages-on-a-user-defined-topic"></a>Recepción de mensajes en un tema definido por el usuario

Con el agente MQTT de IoT Edge, la recepción de mensajes es similar. En primer lugar, asegúrese de que el módulo esté autorizado a suscribirse a temas específicos, a continuación, obtenga un token de la API de cargas de trabajo para usarlo como contraseña al conectarse al agente MQTT y, por último, suscríbase a los mensajes de los temas autorizados con el cliente MQTT de su elección.

::: moniker-end

### <a name="iot-hub-primitives"></a>Primitivos de IoT Hub

IoT Hub considera una instancia de módulo de forma análoga a un dispositivo, en el sentido de que:

* puede enviar [mensajes del dispositivo a la nube](../iot-hub/iot-hub-devguide-messaging.md);
* puede recibir [métodos directos](../iot-hub/iot-hub-devguide-direct-methods.md) dirigidos específicamente a su identidad.
* tiene un módulo gemelo, que es distinto y está aislado del [dispositivo gemelo](../iot-hub/iot-hub-devguide-device-twins.md) y de los otros módulos gemelos de ese dispositivo;

Actualmente, los módulos no pueden recibir mensajes de nube a dispositivo ni usar la característica de carga de archivos.

Al escribir un módulo, puede conectarse al centro de IoT Edge y usar los primitivos de IoT Hub como lo haría al usar IoT Hub con una aplicación de dispositivo. La única diferencia entre los módulos de IoT Edge y las aplicaciones de dispositivos IoT es que tiene que hacer referencia a la identidad de módulo en lugar de a la identidad del dispositivo.

#### <a name="device-to-cloud-messages"></a>Mensajes de dispositivo a nube

Un módulo de IoT Edge puede enviar mensajes a la nube a través del centro de IoT Edge que actúa como agente local y propaga los mensajes a la nube. Para habilitar el procesamiento complejo de mensajes del dispositivo a la nube, un módulo de IoT Edge también puede interceptar y procesar los mensajes enviados por otros módulos o dispositivos a su centro de IoT Edge local y enviar mensajes nuevos con datos procesados. Por tanto, se pueden crear cadenas de módulos de IoT Edge para compilar canalizaciones de procesamiento local.

Para enviar mensajes de telemetría del dispositivo a la nube mediante el enrutamiento, use ModuleClient del SDK de Azure IoT. Con el SDK de Azure IoT, cada módulo tiene el concepto de puntos de conexión de *entrada* y *salida* del módulo, que se asignan a temas de MQTT especiales. Use el método `ModuleClient.sendMessageAsync`, y este enviará mensajes en el punto de conexión de salida del módulo. A continuación, configure una ruta en edgeHub para enviar este punto de conexión de salida a IoT Hub.

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

El envío de mensajes de telemetría del dispositivo a la nube con el agente MQTT es similar a la publicación de mensajes en temas definidos por el usuario, pero con el siguiente tema especial de IoT Hub para el módulo: `devices/<device_name>/<module_name>/messages/events`. Las autorizaciones deben configurarse correctamente. El puente MQTT también debe estar configurado para reenviar los mensajes de este tema a la nube.

::: moniker-end

Para procesar mensajes mediante el enrutamiento, configure primero una ruta para enviar mensajes procedentes de otro punto de conexión (módulo o dispositivo) al punto de conexión de entrada del módulo y, a continuación, escuche los mensajes en el punto de conexión de entrada del módulo. Cada vez que vuelve un mensaje nuevo, el SDK de Azure IoT desencadena una función de devolución de llamada. Procese el mensaje con esta función de devolución de llamada y, opcionalmente, envíe mensajes nuevos en la cola de puntos de conexión del módulo.

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

El procesamiento de mensajes con el agente MQTT es similar a suscribirse a mensajes en temas definidos por el usuario, pero mediante los temas especiales de IoT Edge de la cola de salida del módulo: `devices/<device_name>/<module_name>/messages/events`. Las autorizaciones deben configurarse correctamente. De manera opcional, puede enviar mensajes nuevos en los temas de su elección.

::: moniker-end

#### <a name="twins"></a>Gemelos

Los gemelos son uno de los primitivos proporcionados por IoT Hub. Son documentos JSON que almacenan información acerca del estado, incluidos metadatos, configuraciones y condiciones. Cada módulo o dispositivo tiene su propio gemelo.

Para obtener un módulo gemelo con el SDK de Azure IoT, llame al método `ModuleClient.getTwin`.

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

Para obtener un módulo gemelo con cualquier cliente MQTT, se supone un poco más de trabajo, ya que obtener un gemelo no es un patrón MQTT típico. El módulo primero debe suscribirse al tema especial de IoT Hub `$iothub/twin/res/#`. El nombre de este tema se hereda de IoT Hub, y todos los dispositivos o módulos deben suscribirse al mismo tema. No significa que los dispositivos reciban los gemelos unos de otros. IoT Hub y edgeHub saben qué gemelo se debe entregar a dónde, incluso si todos los dispositivos escuchan el mismo nombre de tema. Una vez que se realice la suscripción, el módulo debe solicitar el gemelo mediante la publicación de un mensaje en el siguiente tema especial de IoT Hub con un identificador de solicitud `$iothub/twin/GET/?$rid=1234`. Este identificador de solicitud es un identificador arbitrario (es decir, un GUID), que IoT Hub devolverá junto con los datos solicitados. Así es como un cliente puede emparejar sus solicitudes con las respuestas. El código de resultado es un código de estado de tipo HTTP, donde se codifica correctamente como 200.

::: moniker-end

Para recibir una revisión de módulo gemelo con el SDK de Azure IoT, implemente una función de devolución de llamada y regístrela con el método `ModuleClient.moduleTwinCallback` desde el SDK de Azure IoT para que la función de devolución de llamada se desencadene cada vez que ingrese la revisión de un gemelo.

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

Para recibir una revisión de módulo gemelo con cualquier cliente MQTT, el proceso es muy similar al de recibir gemelos completo: un cliente tiene que suscribirse a un tema especial de IoT Hub `$iothub/twin/PATCH/properties/desired/#`. Una vez realizada la suscripción, cuando IoT Hub envía un cambio de la sección deseada del gemelo, el cliente la recibe.

::: moniker-end

#### <a name="receive-direct-methods"></a>Recepción de métodos directos

Para recibir un método directo con el SDK de Azure IoT, implemente una función de devolución de llamada y regístrela con el método `ModuleClient.methodCallback` desde el SDK de Azure IoT para que la función de devolución de llamada se desencadene cada vez que ingrese un método directo.

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

Para recibir un método directo con cualquier cliente MQTT, el proceso es muy similar a recibir revisiones de gemelos. El cliente debe confirmar que ha recibido la llamada y puede devolver cierta información al mismo tiempo. El tema especial de IoT Hub al que hay que suscribirse es `$iothub/methods/POST/#`.

::: moniker-end

## <a name="language-and-architecture-support"></a>Compatibilidad de idiomas y arquitecturas

IoT Edge admite varios sistemas operativos, arquitecturas de dispositivos y lenguajes de desarrollo para que pueda crear el escenario que más se ajuste a sus necesidades. Use esta sección para conocer las distintas opciones para desarrollar módulos de IoT Edge personalizados. Puede obtener más información acerca la compatibilidad de las herramientas y de los requisitos de cada lenguaje en [Preparación del entorno de desarrollo y prueba para IoT Edge](development-environment.md).

### <a name="linux"></a>Linux

En todos los idiomas de la tabla siguiente, IoT Edge admite el desarrollo para contenedores de Linux AMD64 y ARM32.

| Lenguaje de desarrollo | Herramientas de desarrollo |
| -------------------- | ----------------- |
| C | Visual Studio Code<br>Visual Studio 2017/2019 |
| C# | Visual Studio Code<br>Visual Studio 2017/2019 |
| Java | Visual Studio Code |
| Node.js | Visual Studio Code |
| Python | Visual Studio Code |

>[!NOTE]
>El soporte técnico del desarrollo y depuración para contenedores de Linux ARM64 está en [versión preliminar pública](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Para más información, consulte [Desarrollo y depuración de módulos ARM64 IoT Edge en Visual Studio Code (versión preliminar)](https://devblogs.microsoft.com/iotdev/develop-and-debug-arm64-iot-edge-modules-in-visual-studio-code-preview).

### <a name="windows"></a>Windows

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"
En todos los idiomas de la tabla siguiente, IoT Edge admite el desarrollo para contenedores de Windows AMD64.

| Lenguaje de desarrollo | Herramientas de desarrollo |
| -------------------- | ----------------- |
| C | Visual Studio 2017/2019 |
| C# | Visual Studio Code (sin funcionalidades de depuración)<br>Visual Studio 2017/2019 |
:::moniker-end
<!-- end 1.1 -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

IoT Edge 1.1 LTS es el último canal de versión compatible con los contenedores de Windows. A partir de la versión 1.2, no se admiten los contenedores de Windows.

Para obtener información sobre el desarrollo con contenedores de Windows, consulte la [versión 1.1 de IoT Edge](?view=iotedge-2018-06&preserve-view=true) de este artículo.

:::moniker-end
<!-- end 1.2 -->

## <a name="next-steps"></a>Pasos siguientes

[Preparación del entorno de desarrollo y prueba para IoT Edge](development-environment.md)

[Uso de Visual Studio para desarrollar módulos de C# para IoT Edge](how-to-visual-studio-develop-module.md)

[Uso de Visual Studio Code para desarrollar módulos para IoT Edge](how-to-vs-code-develop-module.md)

[Información y uso de los SDK de Azure IoT Hub](../iot-hub/iot-hub-devguide-sdks.md)
