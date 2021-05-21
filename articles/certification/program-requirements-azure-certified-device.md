---
title: Requisitos de Azure Certified Device
description: Requisitos del programa Azure Certified Device
author: cbroad
ms.author: cbroad
ms.topic: conceptual
ms.date: 03/15/2021
ms.custom: Azure Certified Device Certification Requirements
ms.service: certification
ms.openlocfilehash: 5acbd4a9cc48841999716cfd9c52d72ccad733b4
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109712827"
---
# <a name="azure-certified-device-requirements"></a>Requisitos de Azure Certified Device 
(conocido anteriormente como IoT Hub)

En este documento se describen las funcionalidades específicas del dispositivo que se representarán en el catálogo de Azure Certified Device. Una funcionalidad es un atributo de dispositivo singular que puede ser una implementación de software o una combinación de implementaciones de software y hardware. 

## <a name="program-purpose"></a>Propósito del programa

Microsoft va a simplificar IoT, y la certificación de Azure Certified Device es un programa de certificación de línea de base que pretende garantizar el aprovisionamiento seguro de todos los tipos de dispositivos en Azure IoT Hub.

Las garantías de la certificación de Azure Certified Device son las siguientes:

1. Telemetría de compatibilidad de dispositivos que funciona con IoT Hub.
2.  Compatibilidad de los dispositivos con IoT Hub Device Provisioning Service (DPS) para un aprovisionamiento seguro en Azure IoT Hub.
3.  El dispositivo admite la entrada sencilla de la transferencia del ámbito de identificador de DPS de destino sin necesidad de que el usuario vuelva a compilar el código insertado.
4.  Valida opcionalmente otros elementos, como los mensajes de la nube al dispositivo, los métodos directos y el dispositivo gemelo. 

## <a name="requirements"></a>Requisitos

**[Obligatorio] Dispositivo a nube: la finalidad de la prueba es asegurarse de que los dispositivos que envían telemetría funcionan con IoT Hub**

| **Nombre**                | AzureCertified.D2C                                               |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ahora disponible                                                |
| **Se aplica a**          | Dispositivo hoja o dispositivo perimetral                                      |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | El dispositivo debe enviar los esquemas de telemetría a IoT Hub. Microsoft proporciona el [flujo de trabajo del portal](https://certify.azure.com/) para ejecutar las pruebas. Dispositivo a nube (obligatorio): **1**. Valida que el dispositivo puede enviar un mensaje a IoT Hub administrado de AICS **2**. El usuario debe especificar el número y la frecuencia de los mensajes. **3.** AICS valida la telemetría que la instancia de Hub recibe. |
| **Recursos**           | [Pasos de certificación](./overview.md) (tiene todos los recursos adicionales) |

**[Obligatorio] DPS: la finalidad de la prueba es comprobar que el dispositivo implementa y admite IoT Hub Device Provisioning Service con uno de los tres métodos de atestación**

| **Nombre**                | AzureCertified.DPS                                               |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Nuevo                                                          |
| **Se aplica a**          | Cualquier dispositivo                                                   |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | El dispositivo admite la entrada sencilla de la propiedad del ámbito del identificador de DPS de destino. Microsoft proporciona el [flujo de trabajo del portal](https://certify.azure.com) para ejecutar las pruebas para validar que el dispositivo admite DPS **1.** El usuario debe seleccionar uno de los métodos de atestación (X.509, TPM y clave SAS) **2**. En función del método de atestación, el usuario debe realizar la acción correspondiente, por ejemplo, **a)** Cargar el certificado X.509 en el ámbito de DPS administrado de AICS **b)** Implementar la clave SAS o la clave de aprobación en el dispositivo |
| **Recursos**           | [Información general sobre Device Provisioning Service](../iot-dps/about-iot-dps.md) |

**[Si se implementa] De la nube al dispositivo: el propósito de la prueba es asegurarse de que los mensajes se pueden enviar desde la nube a los dispositivos**                                                              

| **Nombre**                | AzureCertified.C2D                                                  |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ahora disponible                                            |
| **Se aplica a**          | Dispositivo hoja o dispositivo perimetral                                                   |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | El dispositivo debe ser capaz de enviar mensajes de la nube al dispositivo desde IoT Hub. Microsoft proporciona el [flujo de trabajo del portal](https://certify.azure.com) para ejecutar estas pruebas. De la nube al dispositivo (si se implementa): **1**. Valida que el dispositivo puede recibir el mensaje de IoT Hub **2**. AICS envía un mensaje aleatorio y valida mediante la confirmación del mensaje desde el dispositivo  |
| **Recursos**           | **a)** [Pasos de certificación](./overview.md) (tiene todos los recursos adicionales) **b)** [Envío de mensajes de la nube al dispositivo desde una instancia de IoT Hub](../iot-hub/iot-hub-devguide-messages-c2d.md) |

**[Si se implementa] Métodos directos: la finalidad de la prueba es asegurarse de que los dispositivos funcionen con IoT Hub y admitan métodos directos**

| **Nombre**                | AzureCertified.DirectMethods                                             |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ahora disponible                                            |
| **Se aplica a**          | Dispositivo hoja o dispositivo perimetral                                                   |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | El dispositivo debe ser capaz de recibir las solicitudes de comandos de IoT Hub y responder a ellas. Microsoft proporciona el [flujo de trabajo del portal](https://certify.azure.com) para ejecutar las pruebas. Métodos directos (si se implementan) **1**. El usuario tiene que especificar la carga del método de método directo. **2.** AICS valida que la solicitud de carga especificada se envía desde Hub y el mensaje de confirmación recibido por el dispositivo |
| **Recursos**           | **a)** [Pasos de certificación](./overview.md) (tiene todos los recursos adicionales) **b)** [Descripción de los métodos directos de IoT Hub](../iot-hub/iot-hub-devguide-direct-methods.md) |

**[Si se implementa] Propiedad del dispositivo gemelo: la finalidad de la prueba es asegurarse de que los dispositivos que envían la telemetría funcionan con IoT Hub y admiten algunas de las funcionalidades de IoT Hub, como métodos directos y la propiedad de dispositivo gemelo**

| **Nombre**                                  | AzureCertified.DeviceTwin                                      |
| ----------------------------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino**                   | Ahora disponible                                            |
| **Se aplica a**                            | Dispositivo hoja o dispositivo perimetral                                                   |
| **SISTEMA OPERATIVO**                                    | Independiente                                                     |
| **Tipo de validación**                       | Automatizado                                                       |
| **Validación**                            | El dispositivo debe enviar los esquemas de telemetría a IoT Hub. Microsoft proporciona el [flujo de trabajo del portal](https://certify.azure.com) para ejecutar las pruebas. Propiedad del dispositivo gemelo (si se implementa) **1**. AICS valida la propiedad de lectura y escritura en JSON del dispositivo gemelo **2**. El usuario tiene que especificar la carga de JSON que se va a cambiar **3**. AICS valida las propiedades deseadas especificadas enviadas desde IoT Hub y el mensaje de confirmación recibido por el dispositivo |
| **Recursos**                             | **a)** [Pasos de certificación](./overview.md) (tiene todos los recursos adicionales) **b)** [Uso de dispositivos gemelos con IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md) |

**[Obligatorio] Limitar recompilación: el propósito de esta directiva garantiza que los dispositivos no deben necesitar de forma predeterminada que los usuarios vuelvan a compilar el código para implementar el dispositivo.**

| **Nombre**                                  | AzureCertified.Policy.LimitRecompile                                      |
| ----------------------------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino**                   | Directiva                                            |
| **Se aplica a**                            | Cualquier dispositivo                                                   |
| **SISTEMA OPERATIVO**                                    | Independiente                                                     |
| **Tipo de validación**                       | Directiva                                                       |
| **Validación**                            | Para simplificar la configuración de dispositivos para los usuarios, es necesario que todos los dispositivos se puedan configurar para conectarse a Azure sin necesidad de volver a compilar e implementar el código fuente del dispositivo. Esto incluye la información de DPS, como el identificador de ámbito, que debe establecerse como valor de configuración y no compilarse. Sin embargo, si el dispositivo contiene cierto hardware seguro o si hay circunstancias atenuantes en las que el usuario espera compilar e implementar código, póngase en contacto con el equipo de certificación para solicitar una revisión de excepciones. |
| **Recursos**                             | **a)** [Información general del servicio de aprovisionamiento de dispositivos](../iot-dps/about-iot-dps.md), **b)** [Archivo de configuración de ejemplo para la transferencia del ámbito de identificador de DPS](https://github.com/Azure/azure-iot-sdk-c/tree/public-preview-pnp/serializer/samples/devicetwin_simplesample) |
