---
title: archivo de inclusión
description: archivo de inclusión
services: iot-fundamentals
author: robinsh
ms.service: iot-fundamentals
ms.topic: include
ms.date: 08/07/2018
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 08cca67455df4b2d28bba0a7410fccc11446fcdc
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96010712"
---
En este artículo se proporciona el siguiente nivel de detalle para proteger la infraestructura de Internet de las cosas (IoT) basada en IoT de Azure. Incluye vínculos a detalles de niveles de implementación para configurar e implementar cada componente. También se proporcionan comparaciones y opciones entre diversos métodos que compiten entre sí.

La protección de la implementación de IoT de Azure se puede dividir en las siguientes tres áreas de seguridad:

* **Seguridad del dispositivo**: la protección del dispositivo de IoT mientras se implementa en su entorno natural.

* **Seguridad de la conexión**: asegurar que todos los datos transmitidos entre el dispositivo IoT e IoT Hub son confidenciales y a prueba de manipulaciones.

* **Seguridad de la nube**: proporcionar un medio de proteger los datos mientras se trasladan a la nube y se almacenan allí.

![Tres áreas de seguridad](./media/iot-secure-your-deployment/overview.png)

## <a name="secure-device-provisioning-and-authentication"></a>Seguridad en el aprovisionamiento y la autenticación de dispositivos

Los aceleradores de soluciones de IoT protegen los dispositivos de IoT según los dos métodos siguientes:

* Proporciona una clave de identidad única (tokens de seguridad) para cada dispositivo, que puede usar el dispositivo para comunicarse con el centro de IoT.

* Usa un [certificado X.509](https://www.itu.int/rec/T-REC-X.509-201210-S) y una clave privada en el dispositivo como medio para autenticarlo en IoT Hub. Este método de autenticación garantiza que la clave privada que lleva el dispositivo no se conoce externamente en ningún momento, lo que proporciona un nivel de seguridad más alto.

El método de token de seguridad proporciona autenticación para cada llamada realizada por el dispositivo a IoT Hub mediante la asociación de la clave simétrica a cada llamada. La autenticación basada en X.509 permite la autenticación de un dispositivo IoT en la capa física como parte del establecimiento de la conexión TLS. El método basado en tokens de seguridad se puede usar sin la autenticación de X.509, que es un patrón menos seguro. La elección de un método u otro viene determinada principalmente por lo segura que deba ser la autenticación del dispositivo y la disponibilidad de almacenamiento seguro en este (para almacenar la clave privada de forma segura).

## <a name="iot-hub-security-tokens"></a>Tokens de seguridad de IoT Hub

IoT Hub usa tokens de seguridad para autenticar dispositivos y servicios con el fin de evitar el envío de claves en la red. Además, los tokens de seguridad están limitados en cuanto al ámbito y el período de validez. Los SDK de IoT de Azure generan automáticamente tokens sin necesidad de ninguna configuración especial. Algunos escenarios, sin embargo, requieren que el usuario genere y utilice directamente los tokens de seguridad. Estos escenarios incluyen el uso directo de las superficies MQTT, AMQP o HTTP, o bien la implementación del patrón de servicio de token.

Puede encontrar más detalles sobre la estructura del token de seguridad y su uso en los siguientes artículos:

* [Estructura del token de seguridad](../articles/iot-hub/iot-hub-devguide-security.md#security-token-structure)

* [Uso de tokens de SAS como dispositivo](../articles/iot-hub/iot-hub-devguide-security.md#use-sas-tokens-in-a-device-app)

Cada instancia de IoT Hub tiene un [Registro de identidad](../articles/iot-hub/iot-hub-devguide-identity-registry.md) que se usa para crear recursos en el servicio para cada dispositivo, como una cola que contiene los mensajes de nube a dispositivo en curso, y para permitir el acceso a los puntos de conexión accesibles desde el dispositivo. El Registro de identidades de IoT Hub proporciona un almacenamiento seguro de las identidades y las claves de seguridad de los dispositivos de una solución. Se pueden agregar identidades de dispositivo individuales o en grupo a una lista de elementos permitidos o bloqueados, de forma que se tiene un completo control sobre el acceso a los dispositivos. En los artículos siguientes se proporcionan más detalles sobre la estructura del Registro de identidad y las operaciones admitidas.

[IoT Hub admite protocolos como MQTT, AMQP y HTTP](../articles//iot-hub/iot-hub-devguide-security.md). Cada uno de estos protocolos usa tokens de seguridad entre el dispositivo IoT y IoT Hub de manera diferente:

* AMQP: seguridad basada en notificaciones SASL PLAIN y AMQP (`{policyName}@sas.root.{iothubName}` con tokens en el nivel de IoT Hub, `{deviceId}` con tokens con ámbito de dispositivo).

* MQTT: el paquete CONNECT usa `{deviceId}` como `{ClientId}`, `{IoThubhostname}/{deviceId}` en el campo de **Nombre de usuario** y un token SAS en el campo de **contraseña**.

* HTTP: el token válido está en el encabezado de la solicitud de autorización.

Puede usar el Registro de identidad de IoT Hub para configurar las credenciales de seguridad de cada dispositivo y el control de acceso. Sin embargo, si una solución de IoT ya realizó una inversión importante en un [Registro de identidad del dispositivo o un esquema de autenticación personalizados](../articles/iot-hub/iot-hub-devguide-security.md#custom-device-and-module-authentication), se puede integrar en una infraestructura existente con IoT Hub mediante la creación de un servicio de token.

### <a name="x509-certificate-based-device-authentication"></a>Autenticación de dispositivos basada en certificados X.509

El uso de un [certificado X.509 basado en un dispositivo](../articles/iot-hub/iot-hub-devguide-security.md) y su par de claves pública y privada asociado permite autenticación adicional en la capa física. La clave privada se almacena de forma segura en el dispositivo y es imposible detectarla fuera de él. El certificado X.509 contiene información sobre el dispositivo, como el identificador y otros detalles de la organización. Mediante la clave privada se genera una firma del certificado.

Flujo de aprovisionamiento de dispositivos de alto nivel:

* Asociar un identificador a un dispositivo físico: identidad del dispositivo o certificado X.509 asociado al dispositivo durante su fabricación o puesta en servicio.

* Crear una entrada de identidad correspondiente en IoT Hub: identidad del dispositivo e información del dispositivo asociada en el Registro de Identidad de IoT Hub.

* Almacenar la huella digital del certificado X.509 de forma segura en el Registro de identidad de IoT Hub.

### <a name="root-certificate-on-device"></a>Certificado raíz en el dispositivo

Al establecer una conexión TLS segura con Azure IoT Hub, el dispositivo IoT autentica IoT Hub con un certificado raíz que forma parte de su SDK. Para el SDK cliente de C, el certificado se encuentra en la carpeta "\\c\\certs" en la raíz del repositorio. A pesar de que estos certificados raíz son de larga duración, puede caducar o revocarse. Si no hay forma de actualizar el certificado en el dispositivo, es posible que no se pueda conectar posteriormente a IoT Hub (o a cualquier otro servicio en la nube). Contar con los medios para actualizar el certificado raíz una vez implementado el dispositivo IoT reduce eficazmente este riesgo.

## <a name="securing-the-connection"></a>Protección de la conexión

La conexión a Internet entre el dispositivo IoT y Azure IoT Hub está protegida mediante el estándar de Seguridad de capa de transporte (TLS). Azure IoT admite [TLS 1.2](https://tools.ietf.org/html/rfc5246), TLS 1.1 y TLS 1.0, en este orden. La compatibilidad con TLS 1.0 solo se proporciona para permitir versiones anteriores. Consulte [Compatibilidad de TLS en IoT Hub](../articles/iot-hub/iot-hub-tls-support.md) para ver cómo configurar el centro para usar TLS 1.2, ya que proporciona la máxima seguridad.

## <a name="securing-the-cloud"></a>Protección de la nube

Azure IoT Hub permite la definición de [directivas de control de acceso](../articles/iot-hub/iot-hub-devguide-security.md) para cada clave de seguridad. Para conceder acceso a cada uno de los puntos de conexión de IoT Hub, se utiliza el siguiente conjunto de permisos. Los permisos limitan el acceso a una instancia de IoT Hub según la funcionalidad.

* **RegistryRead**. Concede acceso de lectura al Registro de identidad. Para más información, consulte [Registro de identidades](../articles/iot-hub/iot-hub-devguide-identity-registry.md).

* **RegistryReadWrite**. Concede acceso de lectura y escritura al Registro de identidad. Para más información, consulte [Registro de identidades](../articles/iot-hub/iot-hub-devguide-identity-registry.md).

* **ServiceConnect**. Concede acceso a los puntos de conexión de comunicación y supervisión accesibles desde el servicio en la nube. Por ejemplo, concede permiso a los servicios en la nube de back-end para recibir mensajes de dispositivo a la nube, enviar mensajes de nube a dispositivo y recuperar las confirmaciones de entrega correspondientes.

* **DeviceConnect**. Concede acceso a los puntos de conexión accesibles desde los dispositivos. Por ejemplo, concede permiso para enviar mensajes de dispositivo a nube y recibir mensajes de nube a dispositivo. Este permiso lo usan los dispositivos.

Existen dos maneras de obtener permisos **DeviceConnect** con IoT Hub con [tokens de seguridad](../articles/iot-hub/iot-hub-devguide-security.md#use-sas-tokens-in-a-device-app): mediante una clave de identidad de dispositivo o una clave de acceso compartido. Además, es importante tener en cuenta que puede acceder a toda la funcionalidad desde los dispositivos expuestos por diseño en los puntos de conexión con el prefijo `/devices/{deviceId}`.

Los [componentes de servicio solo pueden generar tokens de seguridad](../articles/iot-hub/iot-hub-devguide-security.md#use-security-tokens-from-service-components) mediante directivas de acceso compartido que conceden los permisos apropiados.

Azure IoT Hub y otros servicios que pueden formar parte de la solución permiten la administración de usuarios mediante Azure Active Directory.

Los datos consumidos por Azure IoT Hub se pueden usar en una gran variedad de servicios, como Azure Stream Analytics, Azure Blob Storage, etc. Estos servicios permiten el acceso de administración. Lea más sobre estos servicios y las opciones disponibles:

* [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/): servicio de base de datos completamente indexado y escalable destinado a datos semiestructurados que administra los metadatos de los dispositivos aprovisionados, como los atributos, la configuración y las propiedades de seguridad. Azure Cosmos DB ofrece procesamiento de alto rendimiento, indexación de datos independiente del esquema y una completa interfaz de consultas SQL.

* [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/): procesamiento de transmisiones en tiempo real en la nube que permite desarrollar e implementar rápidamente una solución de análisis de bajo costo para desvelar datos detallados en tiempo real de dispositivos, sensores, infraestructura y aplicaciones. Los datos de este servicio completamente administrado se pueden escalar a cualquier volumen, sin dejar de obtener alto rendimiento, baja latencia y resistencia.

* [Azure App Services](https://azure.microsoft.com/services/app-service/): plataforma en la nube que sirve para crear sólidas aplicaciones web y móviles desde las que se puede obtener acceso a datos desde cualquier lugar, ya sea en la nube o desde la infraestructura local. Cree atractivas aplicaciones móviles para iOS, Android y Windows. Realice integraciones con aplicaciones de Software como servicio (SaaS) y empresariales gracias a la conectividad de serie a docenas de aplicaciones empresariales y servicios basados en la nube. Codifique en su lenguaje e IDE (.NET, Node.js, PHP, Python o Java) preferidos para crear aplicaciones web y API con más rapidez que nunca.

* [Logic Apps](https://azure.microsoft.com/services/app-service/logic/): la característica Logic Apps de Azure App Service sirve para integrar la solución de IoT con los sistemas de línea de negocio existentes y para automatizar los procesos de flujo de trabajo. Logic Apps permite a los desarrolladores diseñar flujos de trabajo que se inician desde un desencadenador y ejecutan luego una serie de pasos: reglas y acciones que usan eficaces conectores para la integración con los procesos de negocio. Logic Apps ofrece conectividad de serie a un gran ecosistema de aplicaciones SaaS, basadas en la nube y locales.

* [Azure Blob Storage](https://azure.microsoft.com/services/storage/): almacenamiento en la nube confiable y económico para los datos que los dispositivos envían a la nube.

## <a name="conclusion"></a>Conclusión

En este artículo se proporciona información general sobre los detalles de los niveles de implementación para diseñar e implementar una infraestructura de IoT mediante IoT de Azure. La configuración de la seguridad de cada componente es clave para proteger la infraestructura general de IoT. Las opciones de diseño disponibles en IoT de Azure proporcionan un cierto nivel de flexibilidad y elección; sin embargo, cada elección puede tener implicaciones para la seguridad. Se recomienda valorar cada una de estas opciones mediante una evaluación de riesgo y costo.
