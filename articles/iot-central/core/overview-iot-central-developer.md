---
title: Desarrollo de dispositivos para Azure IoT Central | Microsoft Docs
description: Azure IoT Central es una plataforma de aplicaciones de IoT que simplifica la creación de soluciones de IoT. En este artículo se proporciona una introducción al desarrollo de dispositivos para conectarse a la aplicación IoT Central. Los dispositivos usan la telemetría para enviar los datos de streaming y las propiedades para informar sobre el estado de un dispositivo. IOT central puede establecer el estado de un dispositivo mediante las propiedades grabables y llamar a los comandos en un dispositivo.
author: dominicbetts
ms.author: dobett
ms.date: 05/05/2020
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom:
- mvc
- device-developer
ms.openlocfilehash: ebd2759d4dfb8ee79130f9b4876eba8d45226d04
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107718798"
---
# <a name="iot-central-device-development-guide"></a>Guía de desarrollo de dispositivos IoT Central

*Este artículo se aplica a los desarrolladores de dispositivos.*

Una aplicación IoT Central le permite supervisar y administrar millones de dispositivos a lo largo de su ciclo de vida. Esta guía está dirigida a los desarrolladores de dispositivos que implementan código para ejecutarlo en dispositivos que se conectan a IoT Central.

Los dispositivos interactúan con una aplicación IoT Central con los siguientes primitivos:

- _Telemetría_: son datos que un dispositivo envía a IoT Central. Por ejemplo, una secuencia de valores de temperatura de un sensor incorporado.
- _Propiedades_: son valores de estado que un dispositivo notifica a IoT Central. Por ejemplo, la versión actual del firmware del dispositivo. También puede tener propiedades de escritura que IoT Central puede actualizar en el dispositivo, como la temperatura de destino.
- _Comandos_: se llaman desde IoT Central para controlar el comportamiento de un dispositivo. Por ejemplo, la aplicación IoT Central podría llamar a un comando para reiniciar un dispositivo.

Un generador de soluciones es responsable de configurar paneles y vistas en la interfaz de usuario web de IoT Central para visualizar la telemetría, administrar las propiedades y llamar a los comandos.

## <a name="types-of-device"></a>Tipos de dispositivos

En las secciones siguientes se describen los principales tipos de dispositivos que se pueden conectar a una aplicación IoT Central:

### <a name="standalone-device"></a>Dispositivo independiente

Un dispositivo independiente se conecta directamente a IoT Central. Normalmente, un dispositivo independiente envía telemetría desde sus sensores incorporados o conectados a la aplicación IoT Central. Los dispositivos independientes también pueden notificar valores de propiedad, recibir valores de propiedad de escritura y responder a los comandos.

### <a name="gateway-device"></a>Dispositivo de puerta de enlace

Un dispositivo de puerta de enlace administra uno o varios dispositivos de nivel inferior que se conectan a la aplicación IoT Central. IoT Central se usa para configurar las relaciones entre los dispositivos de nivel inferior y el dispositivo de puerta de enlace. Para más información, consulte [Definición de un nuevo tipo de dispositivo de puerta de enlace de IoT en la aplicación de Azure IoT Central](./tutorial-define-gateway-device-type.md).

### <a name="edge-device"></a>Dispositivo perimetral

Un dispositivo perimetral se conecta directamente a IoT Central, pero actúa como intermediario para otros dispositivos conocidos como _dispositivos hoja_. Un dispositivo perimetral normalmente se encuentra cerca de los dispositivos hoja para los que actúa como intermediario. Algunos de los escenarios en los que se usan dispositivos perimetrales son:

- Habilitación de dispositivos que no se pueden conectar directamente a IoT Central y lo hacen a través del dispositivo perimetral. Por ejemplo, un dispositivo hoja podría usar Bluetooth para conectarse al dispositivo perimetral, que luego se conecta a IoT Central a través de Internet.
- Adición de telemetría antes de que se envíe a IoT Central. Este enfoque puede ayudar a reducir los costos de enviar datos a IoT Central.
- Control local de los dispositivos hoja para evitar la latencia asociada con la conexión a IoT Central a través de Internet.

Un dispositivo perimetral también puede enviar su propia telemetría, notificar sus propiedades y responder a los comandos y las actualizaciones de propiedades de escritura.

IoT Central solo ve el dispositivo perimetral, no los dispositivos hoja conectados a él.

Para más información, consulte [Incorporación de un dispositivo Azure IoT Edge a la aplicación Azure IoT Central](./tutorial-add-edge-as-leaf-device.md).

## <a name="connect-a-device"></a>Conexión de un dispositivo

Azure IoT Central usa Azure [IoT Device Provisioning Service (DPS)](../../iot-dps/about-iot-dps.md) para administrar el registro y la conexión de todos los dispositivos.

El uso de DPS permite:

- Que IoT Central admite la incorporación y la conexión de dispositivos a escala.
- Que usted pueda generar credenciales de dispositivo y configurar los dispositivos sin conexión sin tener que registrarlos primero en la interfaz de usuario de IoT Central.
- Que usted pueda usar identificadores de dispositivo propios para registrar estos en IoT Central. Esto simplifica la integración con sistemas del área de operaciones existentes.
- Una manera sencilla y coherente de conectar dispositivos a IoT Central.

Para más información, consulte [Conexión a Azure IoT Central](./concepts-get-connected.md) y [Procedimientos recomendados](concepts-best-practices.md).

### <a name="security"></a>Seguridad

La conexión entre un dispositivo y la aplicación IoT Central está protegida mediante [firmas de acceso compartido](./concepts-get-connected.md#sas-group-enrollment) o [certificados X.509](./concepts-get-connected.md#x509-group-enrollment) estándar del sector.

### <a name="communication-protocols"></a>Protocolos de comunicación

Los protocolos de comunicación que un dispositivo puede usar para conectarse a IoT Central incluyen MQTT, AMQP y HTTPS. A nivel interno, IoT Central usa una instancia de IoT Hub para permitir la conectividad del dispositivo. Para más información sobre los protocolos de comunicación que admite IoT Hub para la conectividad de los dispositivos, consulte [Selección de un protocolo de comunicación](../../iot-hub/iot-hub-devguide-protocols.md).

## <a name="implement-the-device"></a>Implementación del dispositivo

Una plantilla de dispositivo IoT Central incluye un _modelo_ que especifica los comportamientos que debe implementar un dispositivo de ese tipo. Estos comportamientos incluyen telemetría, propiedades y comandos.

> [!TIP]
> Puede exportar el modelo desde IoT Central como un archivo JSON de [lenguaje de definición de Digital Twins (DTDL) v2](https://github.com/Azure/opendigitaltwins-dtdl).

Cada modelo tiene un _identificador de modelo de dispositivo gemelo_ (DTMI) único, como `dtmi:com:example:Thermostat;1`. Cuando un dispositivo se conecta a IoT Central, envía el DTMI del modelo que implementa. A continuación, IoT Central puede asociar la plantilla de dispositivo correcta con el dispositivo.

[IoT Plug and Play](../../iot-pnp/overview-iot-plug-and-play.md) define un conjunto de convenciones que un dispositivo debe seguir cuando implementa un modelo DTDL.

Los [SDK de dispositivos IoT de Azure](#languages-and-sdks) incluyen compatibilidad con las convenciones de IoT Plug and Play.

### <a name="device-model"></a>Modelo de dispositivo

Un modelo de dispositivo se define mediante [DTDL](https://github.com/Azure/opendigitaltwins-dtdl). Este lenguaje le permite definir:

- La telemetría que envía el dispositivo. La definición incluye el nombre y el tipo de datos de la telemetría. Por ejemplo, un dispositivo envía telemetría de temperatura como un valor double.
- Las propiedades que el dispositivo notifica a IoT Central. Una definición de propiedad incluye su nombre y tipo de datos. Por ejemplo, un dispositivo informa del estado de una válvula como un valor booleano.
- Las propiedades que el dispositivo puede recibir desde IoT Central. Opcionalmente, puede marcar una propiedad como grabable. Por ejemplo, IoT Central envía una temperatura de destino como un valor double a un dispositivo.
- Los comandos a los que responde el dispositivo. La definición incluye el nombre del comando y los nombres y tipos de datos de todos los parámetros. Por ejemplo, un dispositivo responde a un comando de reinicio que especifica la cantidad de segundos que hay que esperar antes de reiniciar.

Un modelo de DTDL puede ser _sin componentes_ o de _varios componentes_:

- Modelo sin componentes: Un modelo sencillo no utiliza componentes insertados o en cascada. Toda la telemetría, las propiedades y los comandos se definen en un único _componente predeterminado_. Para ver un ejemplo, consulte el modelo de [termostato](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json).
- Modelo de varios componentes: Un modelo más complejo que incluye dos o más componentes. Estos componentes incluyen un único componente predeterminado y uno o más componentes anidados adicionales. Para ver un ejemplo, consulte el modelo de [controlador de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json).

Para más información, consulte la [Guía de modelado de IoT Plug and Play](../../iot-pnp/concepts-modeling-guide.md).

### <a name="conventions"></a>Convenciones

Un dispositivo debe seguir las convenciones de IoT Plug and Play cuando intercambia datos con IoT Central. Entre las convenciones se incluyen las siguientes:

- Enviar el DTMI cuando se conecte a IoT Central.
- Enviar cargas y metadatos JSON con formato correcto a IoT Central.
- Responder correctamente a comandos y propiedades grabables desde IoT Central.
- Seguir las convenciones de nomenclatura para los comandos de componentes.

> [!NOTE]
> En la actualidad, IoT Central no es completamente compatible con los tipos de datos **Array** y **Geospatial** de DTDL.

Para más información sobre el formato de los mensajes JSON que un dispositivo intercambia con IoT Central, consulte [Cargas de telemetría, propiedades y comandos](concepts-telemetry-properties-commands.md).

Para más información sobre las convenciones de IoT Plug and Play, consulte [Convenciones de IoT Plug and Play](../../iot-pnp/concepts-convention.md).

### <a name="device-sdks"></a>SDK de dispositivo

Use uno de los [SDK de dispositivo Azure IoT](../../iot-hub/iot-hub-devguide-sdks.md#azure-iot-hub-device-sdks) para implementar el comportamiento del dispositivo. El código debe:

- Registrar el dispositivo en DPS y usar la información de DPS para conectarse al centro de IoT Hub interno en la aplicación IoT Central.
- Anunciar el DTMI del modelo que el dispositivo implementa.
- Enviar telemetría en el formato que especifica el modelo de dispositivo. IoT Central usa el modelo de la plantilla de dispositivo para determinar cómo utilizar la telemetría para las visualizaciones y el análisis.
- Sincronizar los valores de propiedad entre el dispositivo e IoT Central. El modelo especifica los nombres de propiedad y los tipos de datos de modo que IoT Central pueda mostrar la información.
- Implementar controladores de comandos para los comandos especificados en el modelo. El modelo especifica los nombres de comandos y los parámetros que el dispositivo debe usar.

Para obtener más información sobre el rol de las plantillas de dispositivo, consulte [¿Qué son las plantillas de dispositivo?](./concepts-device-templates.md)

Para ver algún código de ejemplo, consulte [Creación y conexión de una aplicación cliente](./tutorial-connect-device.md).

### <a name="languages-and-sdks"></a>Lenguajes y SDK

Para más información sobre los lenguajes y SDK admitidos, consulte [Uso de los SDK de dispositivos de Azure IoT Hub](../../iot-hub/iot-hub-devguide-sdks.md#azure-iot-hub-device-sdks).

## <a name="next-steps"></a>Pasos siguientes

Si es un desarrollador de dispositivos y desea profundizar en algún código, el paso siguiente que se sugiere se indica en [Creación y conexión de un aplicación cliente a una aplicación de Azure IoT Central](./tutorial-connect-device.md).

Para más información sobre el uso de IoT Central, los siguientes pasos sugeridos son probar los inicios rápidos, empezando por [Creación de una aplicación de Azure IoT Central](./quick-deploy-iot-central.md).
