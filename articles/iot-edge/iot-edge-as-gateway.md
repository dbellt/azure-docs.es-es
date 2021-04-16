---
title: 'Puertas de enlace para dispositivos de bajada: Azure IoT Edge | Microsoft Docs'
description: Use Azure IoT Edge para crear un dispositivo de puerta de enlace transparente, opaco o proxy que envíe datos desde varios dispositivos de bajada a la nube y los procese localmente.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 03/23/2021
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: aa8b7372af91fc7cb194dfc3a6212cb4ce1fa0a2
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105027353"
---
# <a name="how-an-iot-edge-device-can-be-used-as-a-gateway"></a>Uso de un dispositivo IoT Edge como puerta de enlace

[!INCLUDE [iot-edge-version-201806-or-202011](../../includes/iot-edge-version-201806-or-202011.md)]

Los dispositivos IoT Edge pueden funcionar como puertas de enlace, lo que proporciona una conexión entre otros dispositivos de la red y su instancia de IoT Hub.

El módulo del Centro de IoT Edge actúa como IoT Hub, por lo que puede controlar conexiones desde cualquier dispositivo que tenga una identidad con la misma instancia de IoT Hub. Este tipo de patrón de puertas de enlace se denomina *transparente*, porque los mensajes pueden pasar desde dispositivos de nivel inferior hasta IoT Hub como si no hubiera una puerta de enlace entre ellos.

En el caso de los dispositivos que no pueden conectarse a IoT Hub por sí solos, las puertas de enlace de IoT Edge pueden ofrecer esa conexión. Este tipo de patrón de puerta de enlace se denomina *traducción*, porque el dispositivo IoT Edge tiene que realizar el procesamiento de los mensajes de dispositivo de nivel inferior entrantes antes de que se puedan reenviar a IoT Hub. Estos escenarios requieren de módulos adicionales en la puerta de enlace de IoT Edge para administrar los pasos de procesamiento.

Los patrones de puerta de enlace transparente y de traducción no se excluyen mutuamente. Un solo dispositivo IoT Edge puede funcionar como puerta de enlace transparente y como puerta de enlace de traducción.

Todos los patrones de puerta de enlace proporcionan las siguientes ventajas:

* **Análisis en el perimetral**: use los servicios de IA en el entorno local para procesar los datos que proceden de dispositivos de nivel inferior sin enviar datos de telemetría con total fidelidad a la nube. Busca y reacciona a la información localmente y solo envía un subconjunto de datos a IoT Hub.
* **Aislamiento de dispositivos de nivel inferior**: el dispositivo de puerta de enlace puede proteger todos los dispositivos de nivel inferior de la exposición a Internet. Se puede situar entre una red de tecnología operativa (OT) que no tiene conectividad y una red de tecnologías de la información (TI) que proporciona acceso a Internet. Del mismo modo, los dispositivos que no tienen la capacidad de conectarse a IoT Hub por su cuenta pueden conectarse a un dispositivo de puerta de enlace en su lugar.
* **Multiplexación de conexiones**: todos los dispositivos que se conectan a IoT Hub mediante una puerta de enlace IoT Edge pueden usar la misma conexión subyacente. Esta capacidad de multiplexación requiere que la puerta de enlace de IoT Edge use AMQP como su protocolo de nivel superior.
* **Suavizado de tráfico**: el dispositivo IoT Edge implementará automáticamente un retroceso exponencial si IoT Hub limita el tráfico, al tiempo que se conservan los mensajes localmente. Esta ventaja hace que la solución sea resistente a los picos de tráfico.
* **Compatibilidad sin conexión**: el dispositivo de puerta de enlace almacena los mensajes y las actualizaciones gemelas que no se puedan entregar a IoT Hub.

## <a name="transparent-gateways"></a>Puertas de enlace transparentes

En un patrón de puerta de enlace transparente, los dispositivos que teóricamente podrían conectarse a IoT Hub pueden conectarse en su lugar a un dispositivo de puerta de enlace. Los dispositivos de nivel inferior tienen sus propias identidades de IoT Hub y se conectan mediante protocolos MQTT o AMQP. La puerta de enlace simplemente pasa las comunicaciones entre los dispositivos e IoT Hub. Tanto los dispositivos como los usuarios que interactúan con ellos mediante IoT Hub no saben que una puerta de enlace está arbitrando sus comunicaciones. Esta falta de conocimiento significa que la puerta de enlace se considera *transparente*.

Para obtener más información sobre cómo el centro de IoT Edge administra la comunicación entre los dispositivos de nivel inferior y la nube, consulte [Información del entorno de ejecución de Azure IoT Edge y su arquitectura](iot-edge-runtime.md).

<!-- 1.1 -->
::: moniker range="iotedge-2018-06"
![Diagrama sobre el patrón de puerta de enlace transparente](./media/iot-edge-as-gateway/edge-as-gateway-transparent.png)

>[!NOTE]
>En las versiones 1.1 y anteriores de IoT Edge, un dispositivo de IoT Edge no puede ser inferior a una puerta de enlace de IoT Edge.
>
>A partir de la versión 1.2 de IoT Edge, las puertas de enlace transparentes pueden controlar conexiones desde otros dispositivos de nivel inferior de IoT Edge. Para obtener más información, cambie a la versión [1.2 de IoT Edge](?view=iotedge-2020-11&preserve-view=true) de este artículo.

::: moniker-end

<!-- 1.2 -->
::: moniker range=">=iotedge-2020-11"

A partir de la versión 1.2 de IoT Edge, las puertas de enlace transparentes pueden controlar conexiones desde otros dispositivos de nivel inferior de IoT Edge.

<!-- TODO add a downstream IoT Edge device to graphic -->

::: moniker-end

### <a name="parent-and-child-relationships"></a>Relaciones primarias y secundarias

Las relaciones de puertas de enlace transparentes se declaran en IoT Hub al establecer la puerta de enlace de IoT Edge como elemento *primario* de un dispositivo de nivel inferior *secundario* que se conecta a ella.

La relación de elementos primarios y secundarios se establece en tres puntos de la configuración de la puerta de enlace:

#### <a name="cloud-identities"></a>Identidades en la nube

Todos los dispositivos en un escenario de puerta de enlace transparente necesitan identidades de nube para poder autenticarse en IoT Hub. Al crear o actualizar una identidad del dispositivo, puede establecer los dispositivos primarios o secundarios del mismo. Esta configuración autoriza al dispositivo de puerta de enlace primario para que controle la autenticación de los dispositivos secundarios.

>[!NOTE]
>La configuración del dispositivo primario en IoT Hub solía ser un paso opcional para los dispositivos de nivel inferior que utilizan la autenticación de clave simétrica. Sin embargo, a partir de la versión 1.1.0, cada dispositivo de nivel inferior debe estar asignado a un dispositivo primario.
>
>Puede configurar el centro de IoT Edge para volver al comportamiento anterior estableciendo la variable de entorno **AuthenticationMode** en el valor **CloudAndScope**.

Los dispositivos secundarios solo pueden tener un dispositivo primario. Cada dispositivo primario puede tener hasta 100 dispositivos secundarios.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"
Los dispositivos IoT Edge pueden ser elementos tanto primarios como secundarios en relaciones de puerta de enlace transparentes. Se puede crear una jerarquía de varios dispositivos IoT Edge que se comunican entre sí. El nodo superior de una jerarquía de puerta de enlace puede tener hasta cinco generaciones de elementos secundarios. Por ejemplo, un dispositivo IoT Edge puede tener cinco niveles de dispositivos IoT Edge vinculados como dispositivos secundarios. No obstante, el dispositivo IoT Edge de la quinta generación no puede tener ningún dispositivo secundario, ya sea IoT Edge o de otro tipo.
::: moniker-end

#### <a name="gateway-discovery"></a>Detección de puerta de enlace

Un dispositivo secundario necesita la capacidad para encontrar a su dispositivo primario en la red local. Configure los dispositivos de puerta de enlace con un **nombre de host**, ya sea un nombre de dominio completo (FQDN) o una dirección IP, que los dispositivos secundarios usarán para localizarlo.

En los dispositivos IoT de nivel inferior, use el parámetro **gatewayHostname** en la cadena de conexión para apuntar al dispositivo primario.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"
En los dispositivos IoT Edge de nivel inferior, use el parámetro **parent_hostname** en el archivo de configuración para apuntar al dispositivo primario.
::: moniker-end

#### <a name="secure-connection"></a>Conexión segura

Los dispositivos primarios y secundarios también deben autenticar las conexiones entre sí. Cada dispositivo necesita una copia de un certificado de entidad de certificación raíz compartido que los dispositivos secundarios usan para comprobar que se están conectando a la puerta de enlace adecuada.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"
Cuando varias puertas de enlace de IoT Edge se conectan entre sí en una jerarquía de puertas de enlace, todos los dispositivos de la jerarquía deben usar una sola cadena de certificados.
::: moniker-end

### <a name="device-capabilities-behind-transparent-gateways"></a>Funcionalidades del dispositivo detrás de puertas de enlace transparentes

Todos los elementos primitivos de IoT Hub que funcionan con la canalización de mensajería de IoT Edge también admiten escenarios de puerta de enlace transparente. Cada puerta de enlace de IoT Edge tiene funcionalidades de almacenamiento y reenvío para los mensajes que pasan a través de ella.

Use la siguiente tabla para ver cómo se admiten las diferentes funcionalidades de IoT Hub en los dispositivos, en comparación con los dispositivos que se encuentran detrás de las puertas de enlace.

<!-- 1.1 -->
::: moniker range="iotedge-2018-06"

| Capacidad | Dispositivo IoT | IoT detrás de una puerta de enlace |
| ---------- | ---------- | -------------------- |
| [Mensajes del dispositivo a la nube (D2C)](../iot-hub/iot-hub-devguide-messages-d2c.md) |  ![Sí: D2C de IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: D2C de IoT secundario](./media/iot-edge-as-gateway/check-yes.png) |
| [Mensajes de la nube al dispositivo (C2D)](../iot-hub/iot-hub-devguide-messages-c2d.md) | ![Sí: C2D de IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: C2D de IoT secundario](./media/iot-edge-as-gateway/check-yes.png) |
| [Métodos directos](../iot-hub/iot-hub-devguide-direct-methods.md) | ![Sí: método directo de IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: método directo de IoT secundario](./media/iot-edge-as-gateway/check-yes.png) |
| [Dispositivos gemelos](../iot-hub/iot-hub-devguide-device-twins.md) y [módulos gemelos](../iot-hub/iot-hub-devguide-module-twins.md) | ![Sí: gemelos de IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: gemelos de IoT secundario](./media/iot-edge-as-gateway/check-yes.png) |
| [Carga de archivos](../iot-hub/iot-hub-devguide-file-upload.md) | ![Sí: carga de archivos de IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: carga de archivos de IoT secundario](./media/iot-edge-as-gateway/crossout-no.png) |

::: moniker-end

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"

| Capacidad | Dispositivo IoT | IoT detrás de una puerta de enlace | Dispositivo de IoT Edge | IoT Edge detrás de una puerta de enlace |
| ---------- | ---------- | --------------------------- | --------------- | -------------------------------- |
| [Mensajes del dispositivo a la nube (D2C)](../iot-hub/iot-hub-devguide-messages-d2c.md) |  ![Sí: D2C de IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: D2C de IoT secundario](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: D2C de IoT Edge](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: D2C de IoT Edge secundario](./media/iot-edge-as-gateway/check-yes.png) |
| [Mensajes de la nube al dispositivo (C2D)](../iot-hub/iot-hub-devguide-messages-c2d.md) | ![Sí: C2D de IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: C2D de IoT secundario](./media/iot-edge-as-gateway/check-yes.png) | ![No: C2D de IoT Edge](./media/iot-edge-as-gateway/crossout-no.png) | ![No: C2D de IoT Edge secundario](./media/iot-edge-as-gateway/crossout-no.png) |
| [Métodos directos](../iot-hub/iot-hub-devguide-direct-methods.md) | ![Sí: método directo de IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: método directo de IoT secundario](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: método directo de IoT Edge](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: método directo de IoT Edge secundario](./media/iot-edge-as-gateway/check-yes.png) |
| [Dispositivos gemelos](../iot-hub/iot-hub-devguide-device-twins.md) y [módulos gemelos](../iot-hub/iot-hub-devguide-module-twins.md) | ![Sí: gemelos de IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: gemelos de IoT secundario](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: gemelos de IoT Edge](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: gemelos de IoT Edge secundario](./media/iot-edge-as-gateway/check-yes.png) |
| [Carga de archivos](../iot-hub/iot-hub-devguide-file-upload.md) | ![Sí: carga de archivos de IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: carga de archivos de IoT secundario](./media/iot-edge-as-gateway/crossout-no.png) | ![Sí: carga de archivos de IoT Edge](./media/iot-edge-as-gateway/crossout-no.png) | ![Sí: carga de archivos de IoT Edge secundario](./media/iot-edge-as-gateway/crossout-no.png) |
| Extracciones de imágenes de contenedor |   |   | ![Sí: extracción de contenedor de IoT Edge](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: extracción de contenedor de IoT Edge secundario](./media/iot-edge-as-gateway/check-yes.png) |
| Carga de blobs |   |   | ![Sí: carga de blobs de IoT Edge](./media/iot-edge-as-gateway/check-yes.png) | ![Sí: carga de blobs de IoT Edge secundario](./media/iot-edge-as-gateway/check-yes.png) |

Las **imágenes de contenedor** se pueden descargar, almacenar y entregar de dispositivos primarios a dispositivos secundarios.

Los **blobs**, incluidos los paquetes de soporte y los registros, se pueden cargar de dispositivos secundarios a dispositivos primarios.

::: moniker-end

## <a name="translation-gateways"></a>Puertas de enlace de traducción

Si los dispositivos de nivel inferior no se pueden conectar a IoT Hub, la puerta de enlace de IoT Edge debe actuar como traductor. A menudo, este patrón es necesario para los dispositivos que no son compatibles con MQTT, AMQP o HTTP. Dado que estos dispositivos no se pueden conectar a IoT Hub, tampoco se pueden conectar al módulo del centro de IoT Edge sin procesamiento previo.

Los módulos personalizados o de terceros que suelen ser específicos para el hardware o el protocolo del dispositivo de nivel inferior deben implementarse en la puerta de enlace de IoT Edge. Estos módulos de traducción toman los mensajes entrantes y los convierten en un formato que IoT Hub puede entender.

Hay dos patrones para las puertas de enlace de traducción: *traducción del protocolo* y *traducción de identidades*.

![Diagrama: patrones de puerta de enlace de traducción](./media/iot-edge-as-gateway/edge-as-gateway-translation.png)

### <a name="protocol-translation"></a>Traducción del protocolo

En el patrón de puertas de enlace de traducción de protocolos, solo la puerta de enlace de IoT Edge tiene una identidad con IoT Hub. El módulo de traducción recibe los mensajes de los dispositivos de nivel inferior, los convierte en un protocolo admitido y, a continuación, el dispositivo IoT Edge envía los mensajes en nombre de los dispositivos de nivel inferior. Toda la información parece proceder de un dispositivo, la puerta de enlace. Los dispositivos de nivel inferior deben insertar información de identificación adicional en sus mensajes si las aplicaciones de nube quieren analizar los datos de cada dispositivo. Además, los elementos primitivos de IoT Hub, como los gemelos y los métodos directos, solo son compatibles con el dispositivo de puerta de enlace, no con los dispositivos de nivel inferior. Las puertas de enlace de este patrón se consideran *opacas*, a diferencia de las puertas de enlace transparentes, ya que ocultan las identidades de los dispositivos de nivel inferior.

La traducción de protocolos es compatible con los dispositivos que tienen restricciones de recursos. Muchos dispositivos existentes producen datos que pueden favorecer el conocimiento del negocio; sin embargo, no están diseñados teniendo en mente la conectividad a la nube. Las puertas de enlace opacas permiten que estos datos se desbloqueen y utilicen en una solución IoT.

### <a name="identity-translation"></a>Traducción de la identidad

El patrón de la puerta de enlace de traducción de identidades está basado en la traducción de protocolos, aunque la puerta de enlace de IoT Edge también proporciona una identidad del dispositivo de IoT Hub en nombre de los dispositivos de nivel inferior. El módulo de traducción se encarga de entender el protocolo que usan los dispositivos de nivel inferior, proporcionarles identidades y trasladar sus mensajes a elementos primitivos de IoT Hub. Los dispositivos de nivel inferior aparecen en IoT Hub como dispositivos de primera clase con gemelos y métodos. Un usuario puede interactuar con los dispositivos en IoT Hub y no ser consciente del dispositivo de puerta de enlace intermedio.

La traducción de identidades proporciona las ventajas de la traducción del protocolo y, además, permite una administración completa de los dispositivos de nivel inferior desde la nube. Todos los dispositivos de la solución IoT se muestran en IoT Hub con independencia del protocolo que usen.

### <a name="device-capabilities-behind-translation-gateways"></a>Funcionalidades del dispositivo detrás de puertas de enlace de traducción

En la siguiente tabla se explica cómo se extienden las características de IoT Hub a los dispositivos de nivel inferior en los dos patrones de puerta de enlace de traducción.

| Capacidad | Traducción del protocolo | Traducción de la identidad |
| ---------- | -------------------- | -------------------- |
| Identidades almacenadas en el registro de identidades de IoT Hub | Solo la identidad del dispositivo de puerta de enlace | Identidades de todos los dispositivos conectados |
| Dispositivo gemelo | Solo la puerta de enlace tiene dispositivo y un módulo gemelos | Cada dispositivo conectado tiene su propio dispositivo gemelo |
| Métodos directos y mensajes de nube a dispositivo | La nube solo puede tratar el dispositivo de puerta de enlace | La nube puede tratar individualmente cada dispositivo conectado |
| [Limitaciones y cuotas de IoT Hub](../iot-hub/iot-hub-devguide-quotas-throttling.md) | Aplicación al dispositivo de puerta de enlace | Aplicación a cada dispositivo |

Cuando se usa el patrón de traducción de protocolos, todos los dispositivos que se conectan a través de esa puerta de enlace comparten la misma cola de nube a dispositivo, que puede contener 50 mensajes como máximo. Use este patrón solo cuando pocos dispositivos están conectados a través de cada puerta de enlace de campo y su tráfico de la nube al dispositivo sea bajo.

El entorno de ejecución de Azure IoT Edge no incluye funcionalidades de traducción de protocolos o identidades. Estos patrones requiere módulos personalizados o de terceros que suelen ser específicos del hardware y el protocolo usados. [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules) contiene varios módulos de traducción de protocolo para elegir. Para obtener un ejemplo que usa el patrón de traducción de identidad, vea [Starter Kit de LoRaWAN para Azure IoT Edge](https://github.com/Azure/iotedge-lorawan-starterkit).

## <a name="next-steps"></a>Pasos siguientes

Aprenda los tres pasos para configurar una puerta de enlace transparente:

* [Configuración de un dispositivo IoT Edge para que actúe como puerta de enlace transparente](how-to-create-transparent-gateway.md)
* [Autenticación de un dispositivo de bajada en Azure IoT Hub](how-to-authenticate-downstream-device.md)
* [Conexión de un dispositivo de bajada a una puerta de enlace Azure IoT Edge](how-to-connect-downstream-device.md)
