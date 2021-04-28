---
title: Información general sobre las opciones del SDK de dispositivo IoT de Azure
description: Obtenga información sobre qué SDK de dispositivo IoT de Azure usar en función de su rol y tareas de desarrollo.
author: philmea
ms.author: philmea
ms.service: iot-develop
ms.topic: overview
ms.date: 02/11/2021
ms.openlocfilehash: c35a9045bf809c03630fbb7c57f9d31e7b143422
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107876464"
---
# <a name="overview-of-azure-iot-device-sdks"></a>Información general sobre los SDK de dispositivo IoT de Azure

Los SDK de dispositivo IoT de Azure son un conjunto de bibliotecas cliente de dispositivo, guías para desarrolladores, ejemplos y documentación. Los SDK de dispositivo le ayudan a conectar los dispositivos a los servicios de Azure IoT mediante programación.

:::image type="content" border="false" source="media/about-iot-sdks/iot-sdk-diagram.png" alt-text="Diagrama que muestra varios SDK de Azure IoT":::

Como se muestra en el diagrama, hay varios SDK de dispositivo disponibles para ajustarse a las necesidades del dispositivo y del lenguaje de programación. Puede encontrar instrucciones sobre cómo seleccionar el SDK de dispositivo adecuado en [¿Qué SDK debo usar?](#which-sdk-should-i-use) También hay disponibles SDK de servicios IoT de Azure para conectar la aplicación basada en la nube con los servicios de Azure IoT en el back-end. Este artículo se centra en los SDK de dispositivo, pero puede encontrar información sobre los SDK de servicios de Azure [aquí](#service-sdks).

## <a name="why-should-i-use-the-azure-iot-device-sdks"></a>¿Por qué debo usar los SDK de dispositivo IoT de Azure?

Para conectar dispositivos a Azure IoT, puede crear una capa de conexión personalizada o usar los SDK de dispositivo IoT de Azure. El uso de los SDK de dispositivo IoT de Azure ofrece varias ventajas:

| Costo de desarrollo &nbsp; &nbsp; &nbsp; &nbsp; | Capa de conexión personalizada | SDK de dispositivo IoT de Azure |
| :-- | :------------------------------------------------ | :---------------------------------------- |
| Soporte técnico | Necesidad de dar soporte técnico y documentar todo lo que cree | Acceso al servicio de soporte técnico de Microsoft (GitHub, Microsoft Q&A, Microsoft Docs, equipos de soporte al cliente) |
| Nuevas características | Necesidad de agregar las nuevas características de Azure al middleware personalizado | Puede aprovechar de inmediato las nuevas características que Microsoft agrega constantemente a los SDK de IoT. |
| Inversión | Inversión de cientos de horas de desarrollo insertado para diseñar, crear, probar y mantener una versión personalizada | Puede aprovechar las herramientas gratuitas de código abierto. El único costo asociado a los SDK es la curva de aprendizaje. |

## <a name="which-sdk-should-i-use"></a>¿Qué SDK debo usar?

Los SDK de dispositivo IoT de Azure están disponibles en lenguajes de programación populares, como C, C#, Java, Node.js y Python. Hay dos consideraciones principales a la hora de elegir un SDK: las funcionalidades del dispositivo y la familiaridad del equipo con el lenguaje de programación.

### <a name="device-capabilities"></a>Funcionalidades del dispositivo

Al elegir un SDK, debe tener en cuenta los límites de los dispositivos que utiliza. Un dispositivo restringido es aquel que tiene un solo microcontrolador (MCU) y una memoria limitada. Si usa un dispositivo restringido, se recomienda usar el [SDK para C insertado](#embedded-c-sdk). Este SDK está diseñado para proporcionar el conjunto mínimo de funcionalidades para conectarse a Azure IoT. También puede seleccionar los componentes (cliente MQTT, TLS y bibliotecas de sockets) que estén más optimizados para el dispositivo insertado. Si el dispositivo restringido también ejecuta Azure RTOS, puede usar el middleware de Azure RTOS para conectarse a Azure IoT. El middleware de Azure RTOS incluye el SDK para C insertado con funcionalidad adicional para simplificar la conexión del dispositivo Azure RTOS a la nube.

Un dispositivo sin restricciones es aquel que tiene una CPU más potente, que es capaz de ejecutar un sistema operativo que admita un entorno de ejecución de un lenguaje como .NET o Python. Si usa un dispositivo sin restricciones, la consideración principal es la familiaridad con el lenguaje.

### <a name="your-teams-familiarity-with-the-programming-language"></a>Familiaridad del equipo con el lenguaje de programación

Los SDK de dispositivo IoT de Azure se implementan en varios lenguajes, por lo que puede elegir el lenguaje que prefiera. Los SDK de dispositivo también se integran con otras herramientas conocidas específicas del lenguaje. La capacidad de trabajar con herramientas y lenguajes de desarrollo conocidos permite a su equipo optimizar el ciclo de desarrollo de investigación, creación de prototipos, desarrollo de productos y mantenimiento continuo.

Siempre que sea posible, seleccione un SDK que resulte familiar para el equipo de desarrollo. Todos los SDK de Azure IoT son de código abierto y tienen varios ejemplos disponibles para que su equipo los evalúe y pruebe antes de confirmar un SDK específico.

## <a name="how-can-i-get-started"></a>¿Cómo puedo comenzar?

El punto de partida es explorar los repositorios de GitHub de los SDK de dispositivo de Azure. También puede probar una guía de [Inicio rápido](quickstart-send-telemetry-python.md) que muestre cómo usar rápidamente un SDK para enviar datos de telemetría a Azure IoT.

Las opciones para empezar a trabajar dependen del tipo de dispositivo que tenga:
- Para dispositivos restringidos, use el [SDK para C insertado](#embedded-c-sdk). 
- Para dispositivos que se ejecutan en Azure RTOS, puede realizar el desarrollo con el [middleware de Azure RTOS](#azure-rtos-middleware). 
- Para dispositivos sin restricciones, puede [elegir un SDK](#unconstrained-device-sdks) en el lenguaje que prefiera. 

### <a name="constrained-device-sdks"></a>SDK de dispositivos restringidos
Estos SDK están especializados para ejecutarse en dispositivos con recursos de proceso o memoria limitados. Para más información sobre los tipos de dispositivo comunes, consulte [Información general sobre los tipos de dispositivo de Azure IoT](concepts-iot-device-types.md).

#### <a name="embedded-c-sdk"></a>SDK para C insertado
* [Repositorio de GitHub](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/docs/iot)
* [Muestras](https://github.com/Azure/azure-sdk-for-c/blob/master/sdk/samples/iot/README.md)
* [Documentación de referencia](https://azure.github.io/azure-sdk-for-c/)
* [Compilación del SDK para C insertado](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/docs/iot#build)
* [Gráfico de tamaños de dispositivos restringidos](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/docs/iot#size-chart)

#### <a name="azure-rtos-middleware"></a>Middleware de Azure RTOS

* [Repositorio de GitHub](https://github.com/azure-rtos/netxduo/tree/master/addons/azure_iot)
* [Guías de introducción](https://github.com/azure-rtos/getting-started) y [más ejemplos](https://github.com/azure-rtos/samples)
* [Documentación de referencia](/azure/rtos/threadx/)

### <a name="unconstrained-device-sdks"></a>SDK de dispositivos sin restricciones
Estos SDK se pueden ejecutar en cualquier dispositivo que admita un entorno de ejecución de un lenguaje de orden superior. Esto incluye dispositivos como PC, Raspberry Pi y smartphones. Se diferencian principalmente por el lenguaje, por lo que puede elegir la biblioteca que mejor se adapte a su equipo y escenario.

#### <a name="c-device-sdk"></a>SDK de dispositivo para C
* [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-c)
* [Muestras](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples)
* [Paquetes](https://github.com/Azure/azure-iot-sdk-c/blob/master/readme.md#packages-and-libraries)
* [Documentación de referencia](/azure/iot-hub/iot-c-sdk-ref/)
* [Documentación de referencia del módulo Edge](/azure/iot-hub/iot-c-sdk-ref/iothub-module-client-h)
* [Compilación del SDK de dispositivos para C](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/readme.md#compiling-the-c-device-sdk)
* [Migración del SDK de C a otras plataformas](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md)
* [Documentación para desarrolladores](https://github.com/Azure/azure-iot-sdk-c/tree/master/doc), con información sobre cómo compilar y empezar a trabajar con varias plataformas.
* [Información de consumo de recursos de SDK de C de Azure IoT Hub](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/c_sdk_resource_information.md)

#### <a name="c-device-sdk"></a>SDK de dispositivo para C#

* [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-csharp)
* [Muestras](https://github.com/Azure/azure-iot-sdk-csharp#samples)
* [Package](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/)
* [Documentación de referencia](/dotnet/api/microsoft.azure.devices)
* [Documentación de referencia del módulo Edge](/dotnet/api/microsoft.azure.devices.client.moduleclient)

#### <a name="java-device-sdk"></a>SDK de dispositivo para Java

* [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-java)
* [Muestras](https://github.com/Azure/azure-iot-sdk-java/tree/master/device/iot-device-samples)
* [Package](https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-device-sdk)
* [Documentación de referencia](/java/api/com.microsoft.azure.sdk.iot.device)
* [Documentación de referencia del módulo Edge](/java/api/com.microsoft.azure.sdk.iot.device.moduleclient)

#### <a name="nodejs-device-sdk"></a>SDK de dispositivo para Node.js

* [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-node)
* [Muestras](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples)
* [Package](https://www.npmjs.com/package/azure-iot-device)
* [Documentación de referencia](/javascript/api/azure-iot-device/)
* [Documentación de referencia del módulo Edge](/javascript/api/azure-iot-device/moduleclient)

#### <a name="python-device-sdk"></a>SDK de dispositivo para Python

* [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-python)
* [Muestras](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples)
* [Package](https://pypi.org/project/azure-iot-device/)
* [Documentación de referencia](/python/api/azure-iot-device)
* [Documentación de referencia del módulo Edge](/python/api/azure-iot-device/azure.iot.device.iothubmoduleclient)

### <a name="service-sdks"></a>SDK de servicio
Azure IoT también ofrece SDK de servicios que le permiten crear aplicaciones de la solución para administrar dispositivos, obtener información detallada, visualizar datos y mucho más. Estos SDK son específicos de cada servicio de Azure IoT y están disponibles en C#, Java, JavaScript y Python para simplificar la experiencia de desarrollo. 

#### <a name="iot-hub"></a>IoT Hub

Los SDK de servicios de IoT Hub permiten crear aplicaciones que interactúan fácilmente con los centros de IoT para administrar los dispositivos y la seguridad. Puede usar estos SDK para enviar mensajes de la nube al dispositivo, invocar métodos directos en los dispositivos, actualizar las propiedades del dispositivo, etc.

[**Más información sobre IoT Hub**](https://azure.microsoft.com/services/iot-hub/) | [**Pruebe a controlar un dispositivo**](../iot-hub/quickstart-control-device-python.md)

**SDK de servicios IoT Hub para C#** : [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/iothub/service) | [Paquete](https://www.nuget.org/packages/Microsoft.Azure.Devices/) | [Ejemplos](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/iothub/service/samples) | [Documentación de referencia](/dotnet/api/microsoft.azure.devices)

**SDK de servicios IoT Hub para Java**: [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-java/tree/master/service) | [Paquete](https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-service-sdk) | [Ejemplos](https://github.com/Azure/azure-iot-sdk-java/tree/master/service/iot-service-samples) | [Documentación de referencia](/java/api/com.microsoft.azure.sdk.iot.service)

**SDK de servicios IoT Hub para JavaScript**: [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-node/tree/master/service) | [Paquete](https://www.npmjs.com/package/azure-iothub) | [Ejemplos](https://github.com/Azure/azure-iot-sdk-node/tree/master/service/samples) | [Documentación de referencia](/javascript/api/azure-iothub/)

**SDK de servicios IoT Hub para Python**: [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-hub) | [Paquete](https://pypi.python.org/pypi/azure-iot-hub/) | [Ejemplos](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-hub/samples) | [Documentación de referencia](/python/api/azure-iot-hub)

#### <a name="azure-digital-twins"></a>Azure Digital Twins

Azure Digital Twins es una oferta de plataforma como servicio (PaaS) que permite la creación de grafos de conocimientos basados en modelos digitales de entornos completos. Estos entornos podrían ser edificios, fábricas, granjas, redes energéticas, ferrocarriles, estadios, etc., e incluso ciudades enteras. Estos modelos digitales se pueden usar para obtener información que impulse mejores productos, operaciones optimizadas, costos reducidos y experiencias de cliente innovadoras. Azure IoT ofrece SDK de servicios para facilitar la creación de aplicaciones que usan la eficacia de Azure Digital Twins.

[**Más información sobre Azure Digital Twins**](https://azure.microsoft.com/services/digital-twins/) | [**Programación con las API de Azure Digital Twins**](../digital-twins/tutorial-code.md)

**SDK de servicios de ADT para C#** : [Repositorio de GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core) | [Paquete](https://www.nuget.org/packages/Azure.DigitalTwins.Core) | [Ejemplos](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core/samples) | [Documentación de referencia](/dotnet/api/overview/azure/digitaltwins/client)

**SDK de servicios de ADT para Java**: [Repositorio de GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/digitaltwins/azure-digitaltwins-core) | [Paquete](https://search.maven.org/artifact/com.azure/azure-digitaltwins-core/1.0.0/jar) | [Ejemplos](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/digitaltwins/azure-digitaltwins-core/src/samples) | [Documentación de referencia](/java/api/overview/azure/digitaltwins/client)

**SDK de servicios de ADT para Node.js**: [Repositorio de GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/digitaltwins/digital-twins-core) | [Paquete](https://www.npmjs.com/package/@azure/digital-twins-core) | [Ejemplos](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/digitaltwins/digital-twins-core/samples) | [Documentación de referencia](/javascript/api/@azure/digital-twins-core/)

**SDK de servicios de ADT para Python**: [Repositorio de GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/digitaltwins/azure-digitaltwins-core) | [Paquete](https://pypi.org/project/azure-digitaltwins-core/) | [Ejemplos](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/digitaltwins/azure-digitaltwins-core/samples) | [Documentación de referencia](/python/api/azure-digitaltwins-core/azure.digitaltwins.core)

#### <a name="device-provisioning-service"></a>Servicio de aprovisionamiento de dispositivos

IoT Hub Device Provisioning Service (DPS) es un servicio auxiliar para IoT Hub que habilita el aprovisionamiento sin interacción Just-In-Time a la instancia correcta del centro de IoT sin necesidad de intervención humana. DPS habilita el aprovisionamiento de millones de dispositivos de forma segura y escalable. Los SDK de servicios de DPS permiten crear aplicaciones que pueden administrar los dispositivos de forma segura mediante la creación de grupos de inscripción y la realización de operaciones masivas.

[**Más información sobre Device provisioning Service**](../iot-dps/index.yml) | [**Pruebe a crear una inscripción de grupo para dispositivos X.509**](../iot-dps/quick-enroll-device-x509-csharp.md)

**SDK de servicios de Device Provisioning Service para C#** : [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/provisioning/service) | [Paquete](https://www.nuget.org/packages/Microsoft.Azure.Devices.Provisioning.Service/) | [Ejemplos](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/provisioning/service/samples) | [Documentación de referencia](/dotnet/api/microsoft.azure.devices.provisioning.service)

**SDK de servicios de Device Provisioning Service para Java**: [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-java/tree/master/provisioning/provisioning-service-client/src) | [Paquete](https://mvnrepository.com/artifact/com.microsoft.azure.sdk.iot.provisioning/provisioning-service-client) | [Ejemplos](https://github.com/Azure/azure-iot-sdk-java/tree/master/provisioning/provisioning-samples#provisioning-service-client) | [Documentación de referencia](/java/api/com.microsoft.azure.sdk.iot.provisioning.service)

**SDK de servicios de Device Provisioning Service para Node.js**: [Repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/service) | [Paquete](https://www.npmjs.com/package/azure-iot-provisioning-service) | [Ejemplos](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/service/samples) | [Documentación de referencia](/javascript/api/azure-iot-provisioning-service)

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Conexión de un dispositivo a IoT Central (Python)](quickstart-send-telemetry-python.md)
* [Inicio rápido: Conexión de un dispositivo a IoT Hub (Python)](quickstart-send-telemetry-cli-python.md)
* [Introducción al desarrollo insertado](quickstart-device-development.md)
* Más información sobre las [ventajas de desarrollar con los SDK de Azure IoT](https://azure.microsoft.com/blog/benefits-of-using-the-azure-iot-sdks-in-your-azure-iot-solution/).