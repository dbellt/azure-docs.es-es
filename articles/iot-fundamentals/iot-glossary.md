---
title: Glosario de términos de Azure IoT | Microsoft Docs
description: 'Guía del desarrollador: glosario en el que se explican algunos de los términos comunes empleados en los artículos de Azure IoT.'
author: dominicbetts
ms.author: dobett
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/08/2021
ms.openlocfilehash: a45b8db1f17ead8e55bac2b2bf82aa44cb7f080e
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107304707"
---
# <a name="glossary-of-iot-terms"></a>Glosario de términos de IoT

En este artículo se enumeran varios de los términos que se usan de forma habitual en los artículos de IoT.

## <a name="a"></a>A

### <a name="advanced-message-queueing-protocol"></a>Advanced Message Queueing Protocol

[Advanced Message Queueing Protocol (AMQP)](https://www.amqp.org/) es uno de los protocolos de mensajería que [IoT Hub](#iot-hub) admite para la comunicación con dispositivos. Para obtener más información acerca de los protocolos de mensajería que admite IoT Hub, consulte [Enviar y recibir mensajes con IoT Hub](../iot-hub/iot-hub-devguide-messaging.md).

### <a name="allocation-policy"></a>Directiva de asignación

En [Device Provisioning Service](#device-provisioning-service), la directiva de asignación determina la forma en que el servicio asigna dispositivos a [instancias de IoT Hub vinculadas](#linked-iot-hub).

### <a name="attestation-mechanism"></a>Mecanismo de atestación

En [Device Provisioning Service](#device-provisioning-service), el mecanismo de atestación es el método que se usa para confirmar la identidad de cualquier dispositivo. El mecanismo de atestación se configura en una [inscripción](#enrollment).

Los mecanismos de atestación incluyen certificados X.509, módulos de plataforma segura y claves simétricas.

### <a name="automatic-deployment"></a>Implementación automática

En IoT Edge, una implementación automática configura un conjunto de destino de dispositivos de IoT Edge que ejecutan un conjunto de módulos de IoT Edge. Cada implementación garantiza continuamente que todos los dispositivos que coinciden con su condición de destino están ejecutando el conjunto especificado de módulos, incluso cuando se crean nuevos dispositivos o se modifican para que coincidan con la condición de destino. Cada dispositivo IoT Edge solo recibe la implementación de prioridad más alta cuya condición de destino cumple. Más información acerca de la [implementación automática de IoT Edge](../iot-edge/module-deployment-monitoring.md).

### <a name="automatic-device-configuration"></a>Configuración automática de dispositivos

El back-end de la solución puede usar las [configuraciones automáticas de dispositivos](../iot-hub/iot-hub-automatic-device-management.md) para asignar las propiedades deseadas a un conjunto de [dispositivos gemelos](#device-twin) e informar del estado mediante métricas del sistema y métricas personalizadas.

### <a name="automatic-device-management"></a>Administración automática de dispositivos

La administración automática de dispositivos de Azure IoT Hub automatiza muchas de las tareas repetitivas y complejas de administración de grandes flotas de dispositivos durante su ciclo de vida completo. Con la administración automática de dispositivos, puede tener como destino un conjunto de dispositivos según sus propiedades, definir una configuración deseada y permitir que IoT Hub actualice los dispositivos en cuanto estén dentro del alcance.  Consta de [configuraciones automáticas de dispositivos](../iot-hub/iot-hub-automatic-device-management.md) e [implementaciones automáticas de IoT Edge](../iot-edge/how-to-deploy-at-scale.md).

### <a name="azure-digital-twins"></a>Azure Digital Twins

Azure Digital Twins es una oferta de plataforma como servicio (PaaS) para crear representaciones digitales de cosas, lugares, procesos empresariales y personas reales. Cree gráficos de conocimientos que representen entornos completos y úselos para obtener información para impulsar mejores productos, optimizar operaciones y costos, y crear innovadoras experiencias para los clientes. Para más información, consulte [Azure Digital Twins](../digital-twins/index.yml).

### <a name="azure-digital-twins-instance"></a>Instancia de Azure Digital Twins

Una única instancia del servicio Azure Digital Twins en la suscripción de un cliente. Aunque [Azure Digital Twins](#azure-digital-twins) hace referencia al servicio de Azure en su totalidad, su **instancia** de Azure Digital Twins es el recurso individual de Azure Digital Twins.

### <a name="azure-iot-device-sdks"></a>SDK de dispositivos IoT de Azure

Hay _SDK de dispositivos_ disponibles para varios idiomas que permiten crear [aplicaciones para dispositivo](#device-app) que interactúan con un centro de IoT. Los tutoriales de IoT Hub muestran cómo se utilizan estos SDK de dispositivos. Puede encontrar el código fuente y más información sobre los SDK de dispositivos en este [repositorio](https://github.com/Azure/azure-iot-sdks) de GitHub.

### <a name="azure-iot-explorer"></a>Azure IoT Explorer

[Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer) se usa para ver la telemetría que el dispositivo envía, trabajar con propiedades de dispositivo y llamar a comandos. También puede usar el explorador para interactuar con los dispositivos y probarlos y para administrar [dispositivos IoT Plug and Play](#iot-plug-and-play-device).

### <a name="azure-iot-service-sdks"></a>SDK de servicios IoT de Azure

Hay _SDK de servicios_ disponibles para varios idiomas que permiten crear [aplicaciones de back-end](#back-end-app) que interactúan con un centro de IoT. Los tutoriales de IoT Hub muestran cómo se utilizan estos SDK de servicios. Puede encontrar el código fuente y más información sobre los SDK de servicios en este [repositorio](https://github.com/Azure/azure-iot-sdks) de GitHub.

### <a name="azure-iot-tools"></a>Herramientas de Azure IoT

Las [herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) son una extensión de Visual Studio Code multiplataforma y de código abierto que lo ayuda a administrar Azure IoT Hub y dispositivos en VS Code. Con las herramientas de Azure IoT, los desarrolladores de IoT podrían desarrollar proyectos de IoT en VS Code con facilidad.

## <a name="b"></a>B

### <a name="back-end-app"></a>Aplicación de back-end

En el contexto de [IoT Hub](#iot-hub), una aplicación de back-end es una aplicación que se conecta a uno de los puntos de conexión orientados al servicio en un centro de IoT. Por ejemplo, una aplicación de back-end podría recuperar mensajes [del dispositivo a la nube](#device-to-cloud) o administrar el [registro de identidades](#identity-registry). Normalmente, una aplicación de back-end se ejecuta en la nube pero, en muchos de los tutoriales, las aplicaciones de back-end son aplicaciones de consola que se ejecutan en el equipo de desarrollo local.

### <a name="built-in-endpoints"></a>Puntos de conexión integrados

Un tipo de [punto de conexión](#endpoint) que se crea en IoT Hub. Todas las instancias de IoT Hub incluyen un [punto de conexión](../iot-hub/iot-hub-devguide-endpoints.md) integrado que es compatible con Event Hubs. Puede utilizar cualquier mecanismo que funcione con Event Hubs para leer mensajes del dispositivo a la nube desde este punto de conexión.

## <a name="c"></a>C

### <a name="cloud-gateway"></a>Puerta de enlace en la nube

Una puerta de enlace en la nube habilita la conectividad para dispositivos que no se pueden conectar directamente a [IoT Hub](#iot-hub). Una puerta de enlace en la nube se hospeda en la nube a diferencia de una [puerta de enlace de campo](#field-gateway) que se ejecuta localmente en los dispositivos. Un caso de uso típico de una puerta de enlace en la nube es la implementación de la traducción de protocolos para los dispositivos.

### <a name="cloud-to-device"></a>De la nube al dispositivo

Hace referencia a los mensajes enviados desde un centro de IoT a un dispositivo conectado. A menudo, estos mensajes son comandos que indican al dispositivo que realice una acción. Para más información, vea [Enviar y recibir mensajes con IoT Hub](../iot-hub/iot-hub-devguide-messaging.md).

### <a name="commands"></a>Comandos:

En IoT Plug and Play, los comandos definidos en una [interfaz](#interface) representan los métodos que se pueden ejecutar en el [gemelo digital](#digital-twin). Por ejemplo, un comando para restablecer un dispositivo.

### <a name="component"></a>Componente

En IoT Plug and Play y Azure Digital Twins, los componentes le permiten crear una [interfaz](#interface) del modelo como ensamblado de otras interfaces. Un [modelo de dispositivo](#device-model) puede combinar varias interfaces como componentes. Por ejemplo, un modelo podría incluir un componente interruptor y un componente termostato. Varios componentes de un modelo pueden usar también el mismo tipo de interfaz. Por ejemplo, un modelo puede incluir dos componentes termostato.

### <a name="configuration"></a>Configuración

En el contexto de la [configuración automática de dispositivos](../iot-hub/iot-hub-automatic-device-management.md), una configuración en IoT Hub define la configuración deseada de un conjunto de dispositivos gemelos y ofrece una serie de métricas para informar del estado y el progreso.

### <a name="connection-string"></a>Cadena de conexión

Las cadenas de conexión se usan en el código de aplicación a fin de encapsular la información necesaria para conectarse a un punto de conexión. Normalmente, una cadena de conexión incluye la dirección de la información del punto de conexión y de seguridad, pero los formatos de la cadena de conexión varían en función de los servicios. Hay dos tipos de cadena de conexión asociadas con el servicio de IoT Hub:

- Las *cadenas de conexión de dispositivo* permiten que los dispositivos se conecten a puntos de conexión orientados a dispositivos de un centro de IoT Hub.

- Las *cadenas de conexión de dispositivo* permiten que las aplicaciones de back-end se conecten a puntos de conexión orientados a servicios de un centro de IoT Hub.

### <a name="custom-endpoints"></a>Puntos de conexión personalizados

Puede crear [puntos de conexión](../iot-hub/iot-hub-devguide-endpoints.md) personalizados en una instancia de IoT Hub para enviar mensajes que distribuye una [regla de enrutamiento](#routing-rules). Los puntos de conexión personalizados se conectan directamente con una instancia de Event Hubs, una cola de Service Bus o un tema de Service Bus.

### <a name="custom-gateway"></a>Puerta de enlace personalizada

Una puerta de enlace habilita la conectividad para dispositivos que no se pueden conectar directamente a [IoT Hub](#iot-hub). Puede usar Azure IoT Edge con el fin de crear puertas de enlace personalizadas que implementan lógica personalizada para controlar mensajes, conversiones de protocolo personalizadas y otros procesos del nodo perimetral.

## <a name="d"></a>D

### <a name="data-point-message"></a>Mensaje de punto de datos

Un mensaje de punto de datos es un mensaje [del dispositivo a la nube](#device-to-cloud) que contiene datos de [telemetría](#telemetry), como la velocidad del viento o la temperatura.

### <a name="default-component"></a>Componente predeterminado

En IoT Plug and Play, todos los [modelos de dispositivos](#device-model) tienen un componente predeterminado. Un modelo de dispositivo simple solo tiene un componente predeterminado; este tipo de modelo también se conoce como dispositivo sin componente. Un modelo más complejo tiene varios componentes anidados bajo el componente predeterminado.

### <a name="deployment-manifest"></a>Manifiesto de implementación

En [IoT Edge](#iot-edge), el manifiesto de implementación es un documento JSON que contiene la información que se va a copiar en los módulos gemelos de uno o varios dispositivos de IoT Edge para implementar un conjunto de módulos, rutas y propiedades deseadas el módulo asociado.

### <a name="desired-configuration"></a>Configuración deseada

En el contexto de un [dispositivo gemelo](../iot-hub/iot-hub-devguide-device-twins.md), la configuración deseada hace referencia al conjunto completo de propiedades y metadatos en el dispositivo gemelo que se debe sincronizar con el dispositivo.

### <a name="desired-properties"></a>Propiedades deseadas

En el contexto de un [dispositivo gemelo](../iot-hub/iot-hub-devguide-device-twins.md), las propiedades deseadas son una subsección del dispositivo gemelo que se usa con las [propiedades notificadas](#reported-properties) para sincronizar la configuración o la condición del dispositivo. Solo una [aplicación de back-end](#back-end-app) puede establecer las propiedades deseadas y solo la [aplicación para dispositivo](#device-app) las puede observar.

### <a name="device"></a>Dispositivo

En el contexto de IoT, un dispositivo suele ser un dispositivo de computación independiente a pequeña escala que puede recopilar datos o controlar otros dispositivos. Por ejemplo, un dispositivo podría ser un dispositivo de supervisión ambiental o un controlador para los sistemas de riego y ventilación en un invernadero. El [catálogo de dispositivos](https://catalog.azureiotsolutions.com/) proporciona una lista de dispositivos de hardware certificados que funcionan con [IoT Hub](#iot-hub).

### <a name="device-app"></a>Aplicación para dispositivo

Una aplicación para dispositivo se ejecuta en un [dispositivo](#device) y esta aplicación administra la comunicación con el [centro de IoT](#iot-hub). Por lo general, se suele utilizar uno de los [SDK de dispositivos IoT de Azure](#azure-iot-device-sdks) al implementar una aplicación para dispositivo. En muchos de los tutoriales de IoT, se usa un [dispositivo simulado](#simulated-device) por motivos de comodidad.

### <a name="device-builder"></a>Creador de dispositivos

Un creador de dispositivos usa un [modelo de dispositivo](#device-model) e [interfaces](#interface) al implementar código para que se ejecute en un [dispositivo IoT Plug and Play](#iot-plug-and-play-device). Los creadores de dispositivos usan normalmente uno de los [SDK de dispositivo IoT de Azure](#azure-iot-device-sdks) para implementar el cliente del dispositivo.

### <a name="device-certification"></a>Certificación de dispositivos

El programa de certificación de dispositivos IoT Plug and Play comprueba que un dispositivo cumple los requisitos de certificación de IoT Plug and Play. Puede agregar un dispositivo certificado al [catálogo de dispositivos certificados para Azure IoT](https://aka.ms/devicecatalog).

### <a name="device-condition"></a>Condición de dispositivo

Hace referencia a información de estado del dispositivo, como el método de conectividad actualmente en uso, según notifica una [aplicación para dispositivo](#device-app). Las [aplicaciones para dispositivo](#device-app) también pueden notificar sus funcionalidades. Se puede consultar la información de condición y funcionalidad mediante dispositivos gemelos.

### <a name="device-data"></a>Datos del dispositivo

Los datos del dispositivo hacen referencia a los datos por dispositivo almacenados en el [registro de identidades](#identity-registry) del centro de IoT. Es posible importar y exportar estos datos.

### <a name="device-identity"></a>Identidad del dispositivo

La identidad del dispositivo (o id. de dispositivo) es el identificador único asignado a cada dispositivo registrado del [registro de identidades](#identity-registry) de IoT Hub.

### <a name="device-management"></a>Administración de dispositivos

La administración de dispositivos abarca el ciclo de vida completo asociado a la administración de dispositivos de la solución de IoT, incluidos la planeación, el aprovisionamiento, la configuración, la supervisión y la retirada.

### <a name="device-management-patterns"></a>Patrones de administración de dispositivos

[IoT Hub](#iot-hub) permite patrones comunes de administración de dispositivos incluidos el reinicio, el restablecimiento de fábrica y las actualizaciones de firmware de los dispositivos.

### <a name="device-model"></a>Modelo de dispositivo

Un modelo de dispositivo es un tipo de [modelo](#model) que usa el [lenguaje de definición de Digital Twins](#digital-twins-definition-language-dtdl) para describir las funcionalidades de un dispositivo IoT Plug and Play. Un modelo de dispositivo simple usa una sola interfaz para describir las funcionalidades del dispositivo. Un modelo de dispositivo más complejo incluye varios componentes, cada uno de los cuales describe un conjunto de funcionalidades. Para obtener más información, consulte [Componentes de los modelos de IoT Plug and Play](../iot-pnp/concepts-components.md).

### <a name="device-modeling"></a>Modelado de dispositivos

Un [creador de dispositivos](#device-builder) o [creador de módulos](#module-builder) usa el [lenguaje de definición Digital Twins](#digital-twins-definition-language-dtdl) para modelar las funcionalidades de un [dispositivo IoT Plug and Play](#iot-plug-and-play-device). Un [creador de soluciones](#solution-builder) puede configurar una solución de IOT a partir del modelo.

### <a name="device-provisioning"></a>Aprovisionamiento de dispositivos

El aprovisionamiento de dispositivos es el proceso de agregar los [datos del dispositivo](#device-data) iniciales a los almacenes de la solución. Para habilitar un nuevo dispositivo y que se conecte al centro, debe agregar un nuevo identificador de dispositivo y claves al [registro de identidades](#identity-registry) del centro de IoT. Como parte del proceso de aprovisionamiento, es posible que tenga que inicializar datos específicos del dispositivo en otros almacenes de la solución.

### <a name="device-provisioning-service"></a>Servicio de aprovisionamiento de dispositivos

IoT Hub Device Provisioning Service (DPS) es un servicio auxiliar de [IoT Hub](#iot-hub) que se utiliza para configurar el aprovisionamiento de dispositivos sin interacción de un centro IoT especificado. Con DPS se pueden aprovisionar millones de dispositivos de forma segura y escalable.

### <a name="device-rest-api"></a>API REST de dispositivos

Puede usar la [API REST de dispositivos](/rest/api/iothub/device) desde un dispositivo para enviar mensajes del dispositivo a la nube a IoT Hub y recibir mensajes [de la nube al dispositivo](#cloud-to-device) desde una instancia de IoT Hub. Por lo general, debería usar uno de los [SDK de dispositivos](#azure-iot-device-sdks) de nivel superior, tal como se muestra en los tutoriales de IoT Hub.

### <a name="device-twin"></a>Dispositivo gemelo

Un dispositivo gemelo es un documento JSON que almacena información sobre el estado de los dispositivos, como metadatos, configuraciones y condiciones. IoT Hub conserva un dispositivo gemelo por cada dispositivo que se aprovisiona en el centro de IoT. Los dispositivos gemelos permiten sincronizar las condiciones de dispositivo y las configuraciones entre el dispositivo y el back-end de soluciones. Puede consultar los dispositivos gemelos para buscar dispositivos específicos y ver el estado de las operaciones de larga duración.

### <a name="device-to-cloud"></a>Del dispositivo a la nube

Hace referencia a los mensajes enviados desde un dispositivo conectado a [IoT Hub](#iot-hub). Estos mensajes pueden ser [puntos de datos](#data-point-message) o [mensajes](#interactive-message) interactivos. Para más información, vea [Enviar y recibir mensajes con IoT Hub](../iot-hub/iot-hub-devguide-messaging.md).

### <a name="digital-twin"></a>Gemelo digital

Un gemelo digital es una colección de datos digitales que representa un objeto físico. Los cambios en el objeto físico se reflejan en el gemelo digital. En algunos escenarios, puede usar el gemelo digital para manipular el objeto físico. El [servicio Azure Digital Twins](../digital-twins/index.yml) usa [modelos](#model) expresados en el [lenguaje de definición de Digital Twins (DTDL)](#digital-twins-definition-language-dtdl) para representar gemelos digitales de dispositivos físicos o conceptos empresariales abstractos de alto nivel, lo que permite una amplia gama de soluciones de gemelos digitales basadas en la nube. Un dispositivo [IoT Plug and Play](../iot-pnp/index.yml) tiene un gemelo digital, descrito por un [modelo de dispositivo](#device-model) DTDL.

### <a name="digital-twin-change-events"></a>Eventos de cambio de gemelo digital

Cuando un [dispositivo IoT Plug and Play](#iot-plug-and-play-device) se conecta a un centro de IoT, el centro puede usar su funcionalidad de enrutamiento para enviar notificaciones de cambios de gemelos digitales. Por ejemplo, cada vez que un valor de [propiedad](#properties) cambia en un dispositivo, IoT Hub puede enviar una notificación a un punto de conexión, como un centro de eventos.

### <a name="digital-twin-route"></a>Enrutamiento de gemelo digital

Una ruta configurada en un centro de IoT para ofrecer [eventos de cambio de gemelo digital](#digital-twin-change-events) a un punto de conexión, como un centro de eventos.

### <a name="digital-twins-definition-language-dtdl"></a>Lenguaje de definición de Digital Twins (DTDL)

Un lenguaje JSON-LD que sirve para describir los [modelos](#interface) y las [interfaces](#iot-plug-and-play-device) de los [dispositivos IoT Plug and Play](#model) y las entidades de [Azure Digital Twins](../digital-twins/index.yml). Use la [versión 2 del Lenguaje de definición de Digital Twins](https://github.com/Azure/opendigitaltwins-dtdl) para describir las funcionalidades de un [gemelo digital](#digital-twin) y habilitar la plataforma IoT y las soluciones IoT con el fin de usar la semántica de la entidad. El lenguaje de definición de Digital Twins se suele abreviar como DTDL.

### <a name="direct-method"></a>Método directo

Un [método directo](../iot-hub/iot-hub-devguide-direct-methods.md) es una manera de desencadenar un método para ejecutarse en un dispositivo mediante la invocación de una API en su IoT Hub.

### <a name="downstream-services"></a>Servicios descendentes

Un término relativo que describe los servicios que reciben datos del contexto actual. Por ejemplo, si piensa en el contexto de Azure Digital Twins, [Time series Insights](../time-series-insights/index.yml) se consideraría un servicio de flujo descendente si ha configurado los datos para que fluyan desde Azure Digital Twins a Time Series Insights.

## <a name="e"></a>E

### <a name="endpoint"></a>Punto de conexión

Representación con nombre de un servicio de enrutamiento de datos que puede recibir datos de otros servicios.

Un centro de IoT expone varios [puntos de conexión](../iot-hub/iot-hub-devguide-endpoints.md) que permiten a las aplicaciones conectarse con el centro de IoT. Hay puntos de conexión accesibles desde los dispositivos que permiten a los dispositivos realizar operaciones tales como envío de mensajes [del dispositivo a la nube](#device-to-cloud) y recepción de mensajes [de la nube al dispositivo](#cloud-to-device). Hay puntos de conexión de administración accesibles desde el servicio que permiten a las [aplicaciones de back-end](#back-end-app) realizar operaciones como la administración de [identidades de dispositivo](#device-identity) y la administración de dispositivos gemelos. Hay [puntos de conexión integrados](#built-in-endpoints) orientados al servicio para leer mensajes del dispositivo a la nube. Puede crear [puntos de conexión](#custom-endpoints) personalizados para recibir mensajes del dispositivo a la nube que distribuye una [regla de enrutamiento](#routing-rules).

### <a name="enrollment"></a>Inscripción

En [Device Provisioning Service](#device-provisioning-service), una inscripción es el registro de dispositivos individuales o grupos de dispositivos que se pueden registrar en una [instancia de IoT Hub vinculada](#linked-iot-hub) a través del aprovisionamiento automático.

### <a name="enrollment-group"></a>Grupo de inscripción

En [Service Device Provisioning](#device-provisioning-service), un grupo de inscripción identifica un grupo de dispositivos que comparten un [mecanismo de atestación](#attestation-mechanism) de clave simétrica o X.509.

### <a name="event-handlers"></a>Controladores de eventos

Esto puede hacer referencia a cualquier proceso desencadenado por la llegada de un evento y realiza una acción de procesamiento. Una forma de crear controladores de eventos es agregar código de procesamiento de eventos a una función de Azure y enviar datos a través de ellos mediante [puntos de conexión](#endpoint) y el [enrutamiento de eventos](#event-routing).

### <a name="event-hub-compatible-endpoint"></a>Punto de conexión compatible con Event Hub

Para leer mensajes [del dispositivo a la nube](#device-to-cloud) enviados a IoT Hub, puede conectarse a un punto de conexión en el centro y utilizar cualquier método compatible con Event Hub para leer esos mensajes. Los métodos compatibles con Event Hub incluyen el uso de los [SDK de Event Hubs](../event-hubs/event-hubs-programming-guide.md) y [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md).

### <a name="event-routing"></a>Enrutamiento de eventos

El proceso de envío de eventos y sus datos de un dispositivo o servicio al [punto de conexión](#endpoint) de otro. 

En IoT Hub, puede definir [reglas de enrutamiento](#routing-rules) para describir cómo se deben enviar los mensajes. En Azure Digital Twins, las rutas de eventos son entidades que se crean para este fin. Las rutas de eventos de Azure Digital Twins pueden contener filtros para limitar los tipos de eventos que se envían a cada punto de conexión.

## <a name="f"></a>F

### <a name="field-gateway"></a>Puerta de enlace de campo

Una puerta de enlace de campo habilita la conectividad de dispositivos que no se pueden conectar directamente a [IoT Hub](#iot-hub) y se suele implementar localmente con los dispositivos. Para obtener más información, vea [¿Qué es IoT Hub de Azure?](../iot-hub/about-iot-hub.md).

## <a name="g"></a>G

### <a name="gateway"></a>Puerta de enlace

Una puerta de enlace habilita la conectividad para dispositivos que no se pueden conectar directamente a [IoT Hub](#iot-hub). Consulte también [Puerta de enlace de campo](#field-gateway), [Puerta de enlace en la nube](#cloud-gateway) y [Puerta de enlace personalizada](#custom-gateway).

### <a name="gateway-device"></a>Dispositivo de puerta de enlace

Un dispositivo es un ejemplo de una [puerta de enlace de campo](#field-gateway). Un dispositivo de puerta de enlace puede ser un [dispositivo](#device) IoT estándar o un [dispositivo IoT Edge](#iot-edge-device).

Un dispositivo de puerta de enlace habilita la conectividad para dispositivos descendentes que no se pueden conectar directamente a [IoT Hub](#iot-hub).

## <a name="h"></a>H

### <a name="hardware-security-module"></a>Módulo de seguridad de hardware

Se usa un módulo de seguridad de hardware (HSM) para el almacenamiento seguro basado en hardware de secretos de dispositivos. Es el formato más seguro de almacenamiento secreto para un dispositivo. Tanto los certificados X.509 como las claves simétricas se pueden almacenar en un HSM. En [Device Provisioning Service](#device-provisioning-service), cualquier [mecanismo de atestación](#attestation-mechanism) puede usar un HSM.

## <a name="i"></a>I

### <a name="id-scope"></a>Ámbito de identificador

El ámbito de identificador es un valor único asignado a una instancia de [Device Provisioning Service (DPS)](#device-provisioning-service) cuando se crea.

Las aplicaciones de IoT Central hacen uso de las instancias de DPS y hacen que el ámbito de identificador esté disponible a través de la interfaz de usuario de IoT Central.

### <a name="identity-registry"></a>Registro de identidad

El [registro de identidades](../iot-hub/iot-hub-devguide-identity-registry.md) es el componente integrado de un centro de IoT que almacena información acerca de los dispositivos individuales permitidos para conectarse a un centro de IoT.

### <a name="individual-enrollment"></a>Inscripción individual

En [Device Provisioning Service](#device-provisioning-service), las inscripciones individuales identifican un dispositivo único que usa un certificado de hoja X.509 o una clave simétrica como [mecanismo de atestación](#attestation-mechanism).

### <a name="interactive-message"></a>Mensaje interactivo

Un mensaje interactivo es un mensaje [de nube a dispositivo](#cloud-to-device) que desencadena una acción inmediata en el back-end de solución. Por ejemplo, un dispositivo puede enviar una alarma sobre un error que se debe registrar automáticamente en un sistema CRM.

### <a name="interface"></a>Interfaz

En IoT Plug and Play, una interfaz describe las funcionalidades relacionadas que un [dispositivo IoT Plug and Play](#iot-plug-and-play-device) o un [gemelo digital](#digital-twin) implementa. Puede reutilizar las interfaces en distintos [modelos de dispositivo](#device-model). Cuando se usa una interfaz en un modelo de dispositivo, define un [componente](#component) del dispositivo. Un dispositivo simple solo contiene una interfaz predeterminada.

En Azure Digital Twins, la *interfaz* se puede usar para hacer referencia al elemento de código de nivel superior en una definición de modelo [DTDL](#digital-twins-definition-language-dtdl).

### <a name="iot-edge"></a>IoT Edge

Azure IoT Edge permite una implementación controlada por la nube de los servicios de Azure y código específico de solución para dispositivos locales. Los [dispositivos IoT Edge](#iot-edge-device) pueden agregar datos de otros dispositivos para realizar computación y análisis antes de que se envíen a la nube. Para obtener más información, consulte [Azure IoT Edge](../iot-edge/index.yml).

### <a name="iot-edge-agent"></a>Agente de IoT Edge

La parte del sistema de tiempo de ejecución de IoT Edge responsable de implementar y supervisar los módulos.

### <a name="iot-edge-device"></a>Dispositivo de IoT Edge

Los dispositivos IoT Edge usan [módulos](#modules) IoT Edge para ejecutar servicios de Azure, servicios de terceros o su propio código. En el dispositivo IoT Edge, el [entorno de ejecución de IoT Edge](#iot-edge-runtime) administra los módulos. Puede supervisar y administrar de forma remota un dispositivo IoT Edge desde la nube.

### <a name="iot-edge-hub"></a>Centro de IoT Edge

La parte del sistema de tiempo de ejecución de IoT Edge responsable de las comunicaciones de módulo a módulo, comunicaciones ascendentes (hacia IoT Hub) y descendentes (desde IoT Hub).

### <a name="iot-edge-runtime"></a>Entorno de tiempo de ejecución de IoT Edge

El sistema de tiempo de ejecución de IoT Edge incluye todo lo que Microsoft distribuye para instalarse en un dispositivo IoT Edge. Incluye el agente de Edge, el centro de Edge y el demonio de seguridad de IoT Edge.

### <a name="iot-extension-for-azure-cli"></a>Extensión de IoT para la CLI de Azure

[La extensión de IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension) es una herramienta de la línea de comandos multiplataforma. La herramienta permite administrar los dispositivos en el [registro de identidades](#identity-registry), enviar y recibir mensajes y archivos de los dispositivos, así como supervisar las operaciones del centro de IoT.

### <a name="iot-hub"></a>IoT Hub

IoT Hub es un servicio de Azure totalmente administrado que permite la comunicación bidireccional confiable y segura entre millones de dispositivos y un back-end de soluciones. Para obtener más información, vea [¿Qué es IoT Hub de Azure?](../iot-hub/about-iot-hub.md). Con su suscripción de Azure, puede crear centros de IoT para controlar las cargas de trabajo de mensajería de IoT.

### <a name="iot-hub-metrics"></a>Métricas de IoT Hub

Las métricas de IoT Hub ofrecen datos sobre el estado de los centros de IoT en la suscripción de Azure. Las métricas de IoT Hub permiten evaluar el estado general del servicio y de los dispositivos conectados a él. Las métricas de IoT Hub pueden ayudar a ver lo que está ocurriendo con el centro de IoT y a investigar la causa raíz de los problemas sin necesidad de ponerse en contacto con el soporte técnico de Azure. Para más información, consulte [Supervisión de IoT Hub](../iot-hub/monitor-iot-hub.md).

### <a name="iot-hub-query-language"></a>Lenguaje de consulta de IoT Hub

El [lenguaje de consulta de IoT Hub](../iot-hub/iot-hub-devguide-query-language.md) es un lenguaje de tipo SQL que permite consultar el [trabajo](#job), los gemelos digitales y los dispositivos gemelos.

### <a name="iot-hub-resource-rest-api"></a>API REST de recursos de IoT Hub

Puede usar la [API REST de recursos de IoT Hub](/rest/api/iothub/iothubresource) para administrar los centros de IoT Hub en la suscripción de Azure; por ejemplo, puede realizar operaciones como crear, actualizar y eliminar centros.

### <a name="iot-plug-and-play-bridge"></a>Puente de IoT Plug and Play

IoT Plug and Play Bridge es una aplicación de código abierto que permite a los sensores y periféricos conectados a las puertas de enlace de Windows o Linux conectarse como [dispositivos IoT Plug and Play](#iot-plug-and-play-device).

### <a name="iot-plug-and-play-conventions"></a>Convenciones de IoT Plug and Play

Se espera que los [dispositivos](#iot-plug-and-play-device) IoT Plug and Play sigan un conjunto de convenciones al intercambiar datos con una solución.

### <a name="iot-plug-and-play-device"></a>Dispositivo IoT Plug and Play

Un dispositivo IoT Plug and Play suele ser un dispositivo informático independiente y a pequeña escala que recopila datos o controla otros dispositivos, y que ejecuta software o firmware que implementa un [modelo de dispositivo](#device-model).  Por ejemplo, un dispositivo IoT Plug and Play podría ser un dispositivo de supervisión ambiental o un controlador de un sistema de riego de agricultura inteligente. Un dispositivo IoT Plug and Play podría implementarse directamente o como módulo IoT Edge. Puede escribir una solución de IoT hospedada en la nube para imponer, controlar y recibir datos de dispositivos IoT Plug and Play.

### <a name="iot-solution-accelerators"></a>Aceleradores de soluciones de IoT

Los aceleradores de soluciones de Azure IoT agrupan varios servicios de Azure en soluciones. Estas soluciones le permiten iniciar rápidamente implementaciones de un extremo a otro de escenarios comunes de IoT. Para más información, vea [¿Cuáles son los aceleradores de soluciones de Azure IoT?](../iot-accelerators/about-iot-accelerators.md)

## <a name="j"></a>J

### <a name="job"></a>Trabajo

El back-end de solución puede utilizar [trabajos](../iot-hub/iot-hub-devguide-jobs.md) para programar y realizar el seguimiento de actividades en un conjunto de dispositivos registrados con IoT Hub. Las actividades incluyen actualizar las [propiedades deseadas](#desired-properties) del dispositivo gemelo, actualizar las [etiquetas](#tags) del dispositivo gemelo e invocar [métodos directos](#direct-method). [IoT Hub](#iot-hub) también usa trabajos para [importar y exportar](../iot-hub/iot-hub-devguide-identity-registry.md#import-and-export-device-identities) elementos desde el [registro de identidades](#identity-registry).

## <a name="l"></a>L

### <a name="leaf-device"></a>Dispositivo hoja

En [IoT Edge](#iot-edge), un dispositivo hoja es aquel que no tiene dispositivo descendente.

### <a name="lifecycle-event"></a>Evento de ciclo de vida

En Azure Digital Twins, este tipo de evento se desencadena cuando un elemento de datos (como un gemelo digital, una relación o un controlador de eventos) se crea en una instancia de Azure Digital Twins, o se elimina de ella.

### <a name="linked-iot-hub"></a>Instancia de IoT Hub vinculada

[Device Provisioning Service (DPS)](#device-provisioning-service) solo puede aprovisionar dispositivos en centros de IoT que se hayan vinculado a él. La vinculación de un centro de IoT a una instancia de DPS permite al servicio registrar un id. de dispositivo y establecer la configuración inicial en el dispositivo gemelo.

## <a name="m"></a>M

### <a name="model"></a>Modelo

Un modelo define un tipo de entidad en el entorno físico, incluidas sus propiedades, datos de telemetría, componentes y, a veces, otra información. Los modelos se usan para crear [gemelos digitales](#digital-twin) que representan objetos físicos concretos de este tipo. Los modelos se escriben en el [lenguaje de definición de Digital Twins](#digital-twins-definition-language-dtdl).

En el [servicio Azure Digital Twins](../digital-twins/index.yml), los modelos pueden definir dispositivos o conceptos empresariales abstractos de nivel superior. En [IoT Plug and Play](../iot-pnp/index.yml), los [modelos de dispositivo](#device-model) se usan para describir dispositivos de forma específica.

### <a name="model-id"></a>Id. de modelo

Cuando un dispositivo IoT Plug and Play se conecta a IoT Hub, envía el **id. del modelo** de [DTDL](#digital-twins-definition-language-dtdl) que implementa. Este identificador permite que la solución encuentre el modelo de dispositivo.

### <a name="model-repository"></a>Repositorio de modelos

En el repositorio de modelos se almacenan los [modelos de dispositivo](#device-model) y las [interfaces](#interface).

### <a name="model-repository-rest-api"></a>API REST del repositorio de modelos

Una API para administrar e interactuar con el repositorio de modelos. Por ejemplo, puede usar la API para agregar y buscar [modelos de dispositivo](#device-model).

### <a name="module-builder"></a>Generador de módulos

Un generador de módulos usa un [modelo de dispositivo](#device-model) e [interfaces](#interface) al implementar código para que se ejecute en un [dispositivo IoT Plug and Play](#iot-plug-and-play-device). Los generadores de módulos implementan el código como módulo o módulo IoT Edge para implementar en el tiempo de ejecución de IoT Edge en un dispositivo.

### <a name="module-identity"></a>Identidad de módulo

La identidad de módulo es el identificador único asignado a cada módulo que pertenece a un dispositivo. La identidad de módulo también se registra en el [registro de identidades](#identity-registry).

La identidad del módulo detalla las credenciales de seguridad que el módulo usa para autenticarse con [IoT Hub](#iot-hub) o, en el caso de los módulos IoT Edge, con el [centro de IoT Edge](#iot-edge-hub).

### <a name="module-image"></a>Imagen de módulo

La imagen de Docker que el [entorno de ejecución de IoT Edge](#iot-edge-runtime) usa para crear instancias del módulo.

### <a name="module-twin"></a>Módulo gemelo

De forma similar al dispositivo gemelo, un módulo gemelo es un documento JSON que almacena información sobre el estado de los módulos, como metadatos, configuraciones y condiciones. IoT Hub persiste un módulo gemelo para cada identidad de módulo aprovisionado en una identidad del dispositivo del centro de IoT. Los módulos gemelos permiten sincronizar las condiciones de módulo y las configuraciones entre el módulo y el back-end de soluciones. Puede consultar los módulos gemelos para buscar módulos específicos y consultar el estado de las operaciones de larga duración.

### <a name="modules"></a>Módulos

En el lado del dispositivo, los SDK de dispositivo de IoT Hub le permiten crear [módulos](../iot-hub/iot-hub-devguide-module-twins.md), donde cada uno abre una conexión independiente a IoT Hub. Esta funcionalidad le permite usar espacios de nombres distintos para distintos componentes del dispositivo.

La identidad del módulo y el módulo gemelo proporcionan las mismas funcionalidades que la [identidad del dispositivo](#device-identity) y el [dispositivo gemelo](#device-twin), pero con una mayor granularidad. Esto permite a los dispositivos compatibles, como dispositivos basados en el sistema operativo o dispositivos de firmware que administran varios componentes, aislar la configuración y las condiciones de cada uno de esos componentes.

En [IoT Edge](#iot-edge), un módulo es un contenedor de Docker que se puede implementar en dispositivos de IoT Edge. Realiza una tarea específica,tal como ingerir un mensaje desde un dispositivo, transformar un mensaje o enviarlo a una instancia de IoT Hub. Se comunica con otros módulos y envía datos al [entorno de ejecución de IoT Edge](#iot-edge-runtime).

### <a name="mqtt"></a>MQTT

[MQTT](https://mqtt.org/) es uno de los protocolos de mensajería que admite [IoT Hub](#iot-hub) para la comunicación con dispositivos. Para obtener más información acerca de los protocolos de mensajería que admite IoT Hub, consulte [Enviar y recibir mensajes con IoT Hub](../iot-hub/iot-hub-devguide-messaging.md).

## <a name="o"></a>O

### <a name="ontology"></a>Ontología

Un conjunto de modelos para un dominio determinado, como el patrimonio, las ciudades inteligentes, los sistemas de IoT, las redes de energía, etc. Las ontologías se suelen usar como esquemas para gráficos de conocimiento como los de [Azure Digital Twins](#azure-digital-twins), ya que proporcionan un punto de partida basado en los procedimientos recomendados y estándares del sector.

Para más información sobre las ontologías, vea [¿Qué es una ontología?](../digital-twins/concepts-ontologies.md)

### <a name="operations-monitoring"></a>Supervisión de operaciones

La [supervisión de operaciones](../iot-hub/iot-hub-operations-monitoring.md) de IoT Hub permite supervisar el estado de las operaciones del centro de IoT en tiempo real. [IoT Hub](#iot-hub) realiza el seguimiento de eventos a través de varias categorías de operaciones. Se puede optar por que los eventos de una o varias categorías se envíen a un punto de conexión del IoT Hub para su procesamiento. Los usuarios pueden supervisar los datos en busca de errores o configurar un procesamiento más complejo basado en patrones de datos.

## <a name="p"></a>P

### <a name="physical-device"></a>Dispositivo físico

Un dispositivo físico es un dispositivo real, como un Raspberry Pi que se conecta a un centro de IoT. Para mayor comodidad, muchos de los tutoriales de IoT Hub usan [dispositivos simulados](#simulated-device) para que se puedan ejecutar los ejemplos en el equipo local.

### <a name="primary-and-secondary-keys"></a>Claves principales y secundarias

Al conectarse a un punto de conexión accesible desde el dispositivo o el servicio en un centro de IoT, la [cadena de conexión](#connection-string) incluye la clave para el acceso. Al agregar un dispositivo al [registro de identidad](#identity-registry) o una [directiva de acceso compartido](#shared-access-policy) al centro, el servicio genera una clave principal y una secundaria. Tener dos claves permite pasar de una clave a otra cuando se actualiza una clave, sin perder acceso al centro de IoT.

### <a name="properties"></a>Propiedades

Las propiedades son campos de datos definidos en una [interfaz](#interface) que representan un estado persistente de un [gemelo digital](#digital-twin). Las propiedades se pueden declarar como de solo lectura o de escritura. Las propiedades de solo lectura, como el número de serie, se establecen mediante código que se ejecuta en el propio [dispositivo IoT Plug and Play](#iot-plug-and-play-device). Las propiedades que se pueden escribir, como un umbral de alarma, se establecen normalmente a partir de la solución de IoT basada en la nube.

### <a name="property-change-event"></a>Evento de cambio de propiedad

Evento que es el resultado de un cambio de propiedad en un [gemela digital](#digital-twin).

### <a name="protocol-gateway"></a>Puerta de enlace de protocolo

Una puerta de enlace de protocolo se implementa normalmente en la nube y proporciona servicios de traducción de protocolo para dispositivos que se conectan a [IoT Hub](#iot-hub). Para obtener más información, vea [¿Qué es IoT Hub de Azure?](../iot-hub/about-iot-hub.md).

## <a name="r"></a>R

### <a name="registration"></a>Registro

Es el registro de un dispositivo que se encuentra en el [registro de identidades](#identity-registry) de IoT Hub. El dispositivo se puede registrar directamente, o bien usar [Device Provisioning Service](#device-provisioning-service) para automatizar el registro de dispositivos.

### <a name="registration-id"></a>Identificador de registro

Se usa para identificar de forma única el [registro](#registration) de un dispositivo con [Device Provisioning Service](#device-provisioning-service). El identificador de registro puede tener el mismo valor que la [identidad del dispositivo](#device-identity).

### <a name="relationship"></a>Relación

En el servicio [Azure Digital Twins](../digital-twins/index.yml), las relaciones se usan para conectar [gemelos digitales](#digital-twin) a los grafos de conocimiento que representan digitalmente todo el entorno físico. Los tipos de relaciones que puede tener sus gemelos se definen como parte de las definiciones de [modelo](#model) de estos (el modelo de [DTDL](#digital-twins-definition-language-dtdl) de un tipo determinado de gemelo incluye información sobre las relaciones que puede tener a otros gemelos).

### <a name="reported-configuration"></a>Configuración notificada

En el contexto de un [dispositivo gemelo](../iot-hub/iot-hub-devguide-device-twins.md), la configuración notificada hace referencia al conjunto completo de propiedades y metadatos en el dispositivo gemelo que deben notificarse al back-end de la solución.

### <a name="reported-properties"></a>Propiedades notificadas

En el contexto de un [dispositivo gemelo](../iot-hub/iot-hub-devguide-device-twins.md), las propiedades notificadas son una subsección del dispositivo gemelo que se usa con las [propiedades deseadas](#desired-properties) para sincronizar la configuración o la condición del dispositivo. Solo la [aplicación para dispositivo](#device-app) puede establecer las propiedades notificadas y solo la [aplicación de back-end](#back-end-app) las puede leer y consultar.

### <a name="retry-policy"></a>Directiva de reintentos

Una directiva de reintentos se utiliza para controlar los [errores transitorios](/azure/architecture/best-practices/transient-faults) al conectarse a un servicio en la nube.

### <a name="routing-rules"></a>Reglas de enrutamiento

Configure [reglas de enrutamiento](../iot-hub/iot-hub-devguide-messages-read-custom.md) en su instancia de IoT Hub para enrutar mensajes del dispositivo a la nube a un [punto de conexión integrado](#built-in-endpoints) o a [puntos de conexión personalizados](#custom-endpoints) para que los procese el back-end de su solución.

## <a name="s"></a>S

### <a name="sasl-plain"></a>SASL PLAIN

SASL PLAIN es un protocolo que el protocolo AMQP utiliza para transferir tokens de seguridad.

### <a name="service-operations-endpoint"></a>Punto de conexión de las operaciones del servicio

Un [punto de conexión](#endpoint) para administrar la configuración del servicio utilizada por un administrador de servicios. Por ejemplo, en [Device Provisioning Service](#device-provisioning-service), el punto de conexión de servicio se puede usar para administrar las inscripciones.

### <a name="service-rest-api"></a>API de REST de servicio

Puede usar la [API REST de servicio](/rest/api/iothub/service/configuration) en el back-end de la solución para administrar sus dispositivos. La API le permite recuperar y actualizar las propiedades del [dispositivo gemelo](#device-twin), invocar [métodos directos](#direct-method) y programar [trabajos](#job). Por lo general, debería usar uno de los [SDK de servicios](#azure-iot-service-sdks) de nivel superior, tal como se muestra en los tutoriales de IoT Hub.

### <a name="shared-access-policy"></a>Directiva de acceso compartido

Una directiva de acceso compartido define los permisos concedidos a cualquier persona que tenga una [clave principal o secundaria](#primary-and-secondary-keys) válida asociada a esa directiva. Se pueden administrar las directivas de acceso compartido y las claves para el centro en el portal.

### <a name="shared-access-signature"></a>Firma de acceso compartido

Las firmas de acceso compartido (SAS) son un mecanismo de autenticación basado en valores hash seguros SHA-256 o en URI. La autenticación de SAS tiene dos componentes: una _directiva de acceso compartido_ y una _firma de acceso compartido_ (a menudo denominada token). Un dispositivo utiliza SAS para autenticarse con un centro de IoT. Las [aplicaciones de back-end](#back-end-app) también utilizan SAS para autenticarse con los puntos de conexión accesibles desde servicios en un centro de IoT. Normalmente, se incluye el token de SAS en la [cadena de conexión](#connection-string) que una aplicación usa para establecer una conexión con un centro de IoT.

### <a name="simulated-device"></a>Dispositivo simulado

Para mayor comodidad, muchos de los tutoriales de IoT Hub usan dispositivos simulados para que se puedan ejecutar los ejemplos en el equipo local. En cambio, un [dispositivo físico](#physical-device) es un dispositivo real, como un Raspberry Pi que se conecta a un centro de IoT.

### <a name="solution"></a>Solución

Una _solución_ puede hacer referencia a una solución de Visual Studio que incluye uno o más proyectos. Una _solución_ también puede hacer referencia a una solución de IoT que incluye elementos como dispositivos, [aplicaciones para dispositivo](#device-app), un centro de IoT, otros servicios de Azure y [aplicaciones de back-end](#back-end-app).

### <a name="solution-builder"></a>Creador de soluciones

Un creador de soluciones crea el back-end de la solución. Normalmente, un creador de soluciones trabaja con recursos de Azure, como IoT Hub y [repositorios de modelos](#model-repository).

### <a name="system-properties"></a>Propiedades del sistema

En el contexto de un [dispositivo gemelo](../iot-hub/iot-hub-devguide-device-twins.md), las propiedades del sistema son de solo lectura e incluyen información sobre el uso de dispositivos, como el último estado de conexión y tiempo de actividad.

## <a name="t"></a>T

### <a name="tags"></a>Etiquetas

En el contexto de un [dispositivo gemelo](../iot-hub/iot-hub-devguide-device-twins.md), las etiquetas son metadatos de dispositivos almacenados y recuperados por el back-end de solución en forma de documento JSON. Las etiquetas no son visibles para las aplicaciones en un dispositivo.

### <a name="target-condition"></a>Condición de destino

En una implementación de IoT Edge, la condición de destino selecciona los dispositivos de destino de la implementación, como por ejemplo, **tag.environment = prod**. La condición de destino se evalúa continuamente para incluir todos los nuevos dispositivos que cumplen los requisitos o desconectar los dispositivos que ya no lo hacen.

### <a name="telemetry"></a>Telemetría

Los dispositivos recopilan datos de telemetría, como la velocidad del viento o la temperatura, y usan mensajes de punto de datos para enviar la telemetría a un centro de IoT.

En IoT Plug and Play y Azure Digital Twins, los campos de telemetría definidos en una [interfaz](#interface) representan medidas. Estas medidas suelen ser valores, como por ejemplo las lecturas de los sensores, que envían los dispositivos, como los [dispositivos IoT Plug and Play](#iot-plug-and-play-device) en forma de flujo de datos.

A diferencia de las [propiedades](#properties), la telemetría no se almacena en un [gemelo digital](#digital-twin); es una secuencia de eventos de datos con límites temporales que se deben controlar a medida que se producen.

### <a name="telemetry-event"></a>Evento de telemetría

Un evento que indica la llegada de datos de telemetría.

### <a name="token-service"></a>Servicio de token

Se puede utilizar un servicio de token a fin de implementar un mecanismo de autenticación para los dispositivos. Usa una [directiva de acceso compartido](#shared-access-policy) de IoT Hub con permisos **DeviceConnect** para crear *tokens centrados en el dispositivo*. Estos tokens permiten que un dispositivo se conecte al centro de IoT. Un dispositivo utiliza un mecanismo de autenticación personalizado para autenticarse con el servicio de token. Si el dispositivo se autentica correctamente, el servicio de token emite un token de SAS para que el dispositivo pueda tener acceso al centro de IoT.

### <a name="twin-graph-or-digital-twin-graph"></a>Grafo gemelo (o grafo de gemelo digital)

En el servicio [Azure Digital Twins](../digital-twins/index.yml), es posible conectar [gemelos digitales](#digital-twin) a [relaciones](#relationship) para crear grafos de conocimiento que representan digitalmente todo un entorno físico. Una sola [instancia de Azure Digital Twins](#azure-digital-twins-instance) puede hospedar muchos grafos desconectados o un solo grafo interconectado.

### <a name="twin-queries"></a>Consultas gemelas

Las [consultas de dispositivos y módulos gemelos](../iot-hub/iot-hub-devguide-query-language.md) usan el lenguaje de consulta de IoT Hub de tipo SQL para recuperar información de los dispositivos o módulos gemelos. Puede usar el mismo lenguaje de consulta de IoT Hub para recuperar información sobre un [trabajo](#job) que se ejecuta en IoT Hub.

### <a name="twin-synchronization"></a>Sincronización de gemelos

La sincronización de gemelos usa las [propiedades deseadas](#desired-properties) en los dispositivos o módulos gemelos para configurar los dispositivos o módulos y recuperar las [propiedades notificadas](#reported-properties) de los dispositivos para almacenarlas en el gemelo.

## <a name="u"></a>U

### <a name="upstream-services"></a>Servicios ascendentes

Un término relativo que describe los servicios que aportan datos al contexto actual. Por ejemplo, en el contexto de Azure Digital Twins, IoT Hub se considera un servicio ascendente porque los datos fluyen de IoT Hub a Azure Digital Twins.

## <a name="x"></a>X

### <a name="x509-client-certificate"></a>Certificado de cliente X.509

Un dispositivo puede usar un certificado X.509 para autenticarse con [IoT Hub](#iot-hub). Un certificado X.509 es una alternativa al uso de un [token de SAS](#shared-access-signature).
