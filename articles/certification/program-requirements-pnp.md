---
title: Requisitos de certificación de IoT Plug and Play
description: Requisitos del programa de certificación de IoT Plug and Play
author: cbroad
ms.author: cbroad
ms.topic: conceptual
ms.date: 03/15/2021
ms.custom: IoT Plug and Play Certification Requirements
ms.service: certification
ms.openlocfilehash: eaf3ef6016388404761e913f878972df4a333061
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109712809"
---
# <a name="iot-plug-and-play-certification-requirements"></a>Requisitos de certificación de IoT Plug and Play

En este documento se describen las funcionalidades específicas del dispositivo que se representarán en el catálogo de dispositivos de Azure IoT. Una funcionalidad es un atributo de dispositivo singular que puede ser una implementación de software o una combinación de implementaciones de software y hardware.

## <a name="program-purpose"></a>Propósito del programa

La versión preliminar de IoT Plug and Play permite a los creadores de soluciones integrar dispositivos inteligentes en sus soluciones sin necesidad de configuración manual. IoT Plug and Play se basa en un modelo de dispositivos que un dispositivo usa para anunciar sus funcionalidades a una aplicación compatible con IoT Plug and Play. Este modelo está estructurado en un conjunto de elementos: telemetría, propiedades y comandos.

El compromiso de la certificación de IoT Plug and Play es:

1.  Las interfaces y los modelos de dispositivos definidos son compatibles con el [lenguaje de definición de gemelos digitales](https://github.com/Azure/opendigitaltwins-dtdl)  
1.  Fácil integración con las soluciones basadas en Azure IoT mediante las [API gemelas digitales](../iot-pnp/concepts-digital-twin.md): Azure IoT Hub y Azure IoT Central
1.  Veracidad del producto validada en dispositivos certificados
1.  Cumple todos los requisitos [de Azure Certified Device](./program-requirements-azure-certified-device.md)

## <a name="requirements"></a>Requisitos

**[Obligatorio] Dispositivo a nube: la finalidad de la prueba es asegurarse de que los dispositivos que envían telemetría funcionan con IoT Hub**

| **Nombre**                | IoTPnP.D2C                                               |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ahora disponible                                                |
| **Se aplica a**          | Dispositivo hoja o dispositivo perimetral                                      |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | El dispositivo debe enviar los esquemas de telemetría a IoT Hub. Microsoft proporciona el [flujo de trabajo del portal](https://certify.azure.com) para ejecutar las pruebas. Dispositivo a nube (obligatorio): **1**. Valida que el dispositivo puede enviar un mensaje a IoT Hub administrado de AICS **2**. El usuario debe especificar el número y la frecuencia de los mensajes. **3.** AICS valida la telemetría que la instancia de Hub recibe. |
| **Recursos**           | [Pasos de certificación](./overview.md) (tiene todos los recursos adicionales) |


**[Obligatorio] DTDL V2: la finalidad de la prueba para garantizar que los modelos de dispositivos y las interfaces definidos son compatibles con el lenguaje de definición de Digital Twins v2.**                                                              

| **Nombre**                | IoTPnP.DTDL                                                  |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ahora disponible                                                |
| **Se aplica a**          | Cualquier dispositivo                                                   |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | El [flujo de trabajo del portal](https://certify.azure.com) se valida: **1.** Anuncio del identificador de modelo y asegúrese de que el dispositivo está conectado mediante MQTT o MQTT a través del protocolo de WebSockets **2.** Los modelos son compatibles con DTDL v2 **3.** La telemetría, las propiedades y los comandos se implementan correctamente e interactúan entre el gemelo digital de IoT Hub y el dispositivo gemela en el dispositivo |
| **Recursos**           | [Actualizaciones de versión preliminar pública](../iot-pnp/overview-iot-plug-and-play.md) |

**[Obligatorio] Los modelos de dispositivos se publican en el repositorio de modelos público**

| **Nombre**                | IoTPnP.ModelRepo                                             |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ahora disponible                                                |
| **Se aplica a**          | Cualquier dispositivo                                                   |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | Todos los modelos de dispositivo deben publicarse en el repositorio público. Los modelos de dispositivo se resuelven a través de los modelos disponibles en el repositorio público **1.** El usuario debe publicar manualmente los modelos en el repositorio público antes de enviar para la certificación. **2.** Tenga en cuenta que una vez que se publican los modelos, es inmutable. Se recomienda encarecidamente publicar solamente cuando se finalizan los modelos y el código de dispositivo incorporado.*1  *1 El usuario debe ponerse en contacto con el soporte técnico de Microsoft para revocar los modelos una vez publicados en el repositorio de modelos **3.** El [flujo de trabajo del portal](https://certify.azure.com) comprueba la existencia de los modelos en el repositorio público cuando el dispositivo está conectado al servicio de certificación |
| **Recursos**           | [Repositorio de modelos](../iot-pnp/overview-iot-plug-and-play.md) |

**[Obligatorio] Validación del dispositivo físico mediante GSG**

| **Nombre**                                  | IoTPnP.Physicaldevice                                      |
| ----------------------------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino**                   | Ahora disponible                                                |
| **Se aplica a**                            | Cualquier dispositivo                                                   |
| **SISTEMA OPERATIVO**                                    | Independiente                                                     |
| **Tipo de validación**                       | Manual                                                       |
| **Validación**                            | Los asociados deben ponerse en contacto con Microsoft ([iotcert@microsoft.com](mailto:iotcert@microsoft.com)) para tomar medidas para realizar validaciones adicionales en el dispositivo físico. Debido a la situación de la COVID-19, estamos explorando varias maneras de realizar la validación de dispositivos físicos sin enviar el dispositivo a Microsoft. |
| **Recursos**                             | Más adelante encontrará detalles al respecto                                 |
| **Recomendado para Azure**       | N/D    |

**[Si se implementa] Interfaz de información del dispositivo: la finalidad de la prueba es validar que la interfaz de información del dispositivo se implementa correctamente en el código del dispositivo**

| **Nombre**                | IoTPnP.DeviceInfoInterface                                   |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ahora disponible                                                |
| **Se aplica a**          | Cualquier dispositivo                                                   |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | El [flujo de trabajo del portal](https://certify.azure.com) valida el código del dispositivo que implementa la interfaz de información del dispositivo **1.** Comprueba que el código del dispositivo emite los valores para IoT Hub **2.** Comprueba que la interfaz está implementada en DCM (esta implementación cambiará en DTDL v2) **3.** Comprueba que las propiedades no pueden escribirse (solo lectura) **4.** Comprueba que el tipo de esquema es String y/o Long, no null |
| **Recursos**           | [Interfaz definidas por Microsoft](../iot-pnp/overview-iot-plug-and-play.md) |
| **Recomendado para Azure**  | N/D                                                          |

**[Si se implementa] De la nube al dispositivo: el propósito de la prueba es asegurarse de que los mensajes se pueden enviar desde la nube a los dispositivos.**

| **Nombre**                | IoTPnP.C2D                                               |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ahora disponible                                                |
| **Se aplica a**          | Dispositivo hoja o dispositivo perimetral                                      |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | El dispositivo debe ser capaz de enviar mensajes de la nube al dispositivo desde IoT Hub. Microsoft proporciona el [flujo de trabajo del portal](https://certify.azure.com) para ejecutar estas pruebas. De la nube al dispositivo (si se implementa): **1.** Valida que el dispositivo puede recibir el mensaje de IoT Hub **2.** AICS envía un mensaje aleatorio y valida mediante la confirmación del mensaje desde el dispositivo |
| **Recursos**           | **1.** [Pasos de certificación](./overview.md) (tiene todos los recursos adicionales), **2.** [Envío de mensajes de la nube al dispositivo desde una instancia de IoT Hub](../iot-hub/iot-hub-devguide-messages-c2d.md) |

**[Si se implementa] Métodos directos: la finalidad de la prueba es asegurarse de que los dispositivos funcionen con IoT Hub y admita métodos directos**

| **Nombre**                | IoTPnP.DirectMethods                                     |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ahora disponible                                                |
| **Se aplica a**          | Dispositivo hoja o dispositivo perimetral                                      |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | El dispositivo debe ser capaz de recibir las solicitudes de comandos de IoT Hub y responder a ellas. Microsoft proporciona el [flujo de trabajo del portal](https://certify.azure.com) para ejecutar las pruebas. Métodos directos (si se implementan): **1.** El usuario tiene que especificar la carga del método de método directo. **2.** AICS valida que la solicitud de carga especificada se envía desde Hub y el mensaje de confirmación recibido por el dispositivo |
| **Recursos**           | **1.** [Pasos de certificación](./overview.md) (tiene todos los recursos adicionales), **2.** [Descripción de los métodos directos de IoT Hub](../iot-hub/iot-hub-devguide-direct-methods.md) |

**[Si se implementa] Propiedad gemela del dispositivo: la finalidad de la prueba es asegurarse de que los dispositivos que envían la telemetría funcionan con IoT Hub y admite algunas de las funcionalidades de IoT Hub, como métodos directos y la propiedad de dispositivo gemelo**

| **Nombre**                | IoTPnP.DeviceTwin                                        |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ahora disponible                                                |
| **Se aplica a**          | Dispositivo hoja o dispositivo perimetral                                      |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | El dispositivo debe enviar los esquemas de telemetría a IoT Hub. Microsoft proporciona el [flujo de trabajo del portal](https://certify.azure.com) para ejecutar las pruebas. Propiedad de dispositivo gemelo (si se implementa): **1.** AICS valida la propiedad de lectura y escritura en JSON del dispositivo gemelo **2.** El usuario tiene que especificar la carga de JSON que se va a cambiar **3**. AICS valida las propiedades deseadas especificadas enviadas desde IoT Hub y el mensaje de confirmación recibido por el dispositivo |
| **Recursos**           | **1.** [Pasos de certificación](./overview.md) (tiene todos los recursos adicionales), **2.** [Uso de dispositivos gemelos con IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md) |