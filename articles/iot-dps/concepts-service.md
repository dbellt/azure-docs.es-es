---
title: Terminología utilizada con Azure IoT Hub Device Provisioning Service | Microsoft Docs
description: Se describe la terminología común utilizada con Device Provisioning Service (DPS) e IoT Hub.
author: wesmc7777
ms.author: wesmc
ms.date: 09/18/2019
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: eliotga
ms.openlocfilehash: c9397351d1b6be5533f3a5eb9206dfd97311d9a1
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109483360"
---
# <a name="iot-hub-device-provisioning-service-dps-terminology"></a>Terminología de IoT Hub Device Provisioning Service (DPS)

IoT Hub Device Provisioning es un servicio auxiliar de IoT Hub que se utiliza para configurar el aprovisionamiento de dispositivos sin interacción de un centro IoT especificado. Con el servicio Device Provisioning puede [aprovisionar](about-iot-dps.md#provisioning-process) millones de dispositivos de forma segura y escalable.

El aprovisionamiento de dispositivos es un proceso de dos partes. La primera consiste en establecer la conexión inicial entre el dispositivo y la solución de IoT *registrando* el dispositivo. La segunda es aplicar la *configuración* apropiada en el dispositivo en función de los requisitos específicos de la solución. Una vez completados ambos pasos, el dispositivo se considera totalmente *aprovisionado*. El servicio Device Provisioning automatiza ambos pasos para proporcionar una experiencia perfecta de aprovisionamiento para el dispositivo.

Este artículo proporciona información general sobre los conceptos de aprovisionamiento más aplicables a la administración del *servicio*. Este artículo es más apropiado para las personas implicadas en [el paso de configuración en la nube](about-iot-dps.md#cloud-setup-step) para preparar un dispositivo para la implementación.

## <a name="service-operations-endpoint"></a>Punto de conexión de las operaciones del servicio

El punto de conexión de las operaciones del servicio es para administrar la configuración del servicio y el mantenimiento de la lista de inscripción. Este punto de conexión solo lo utiliza el administrador del servicio; no lo usan los dispositivos.

## <a name="device-provisioning-endpoint"></a>Punto de conexión de aprovisionamiento de dispositivos

El punto de conexión de aprovisionamiento de dispositivos es el único que usan todos los dispositivos para el aprovisionamiento automático. La dirección URL es la misma para todas las instancias de los servicios de aprovisionamiento a fin de evitar tener que volver a programar los dispositivos con nueva información de conexión en escenarios de la cadena de suministro. El ámbito de identificador garantiza el aislamiento de los inquilinos.

## <a name="linked-iot-hubs"></a>Centros de IoT vinculados

El servicio Device Provisioning solo puede aprovisionar los dispositivos para los centros de IoT que se hayan vinculado a él. Vincular un centro de IoT a una instancia del servicio Device Provisioning proporciona los permisos de lectura/escritura del servicio al registro de dispositivos del centro de IoT; con el vínculo, el servicio puede registrar un identificador de dispositivo y establecer la configuración inicial en el dispositivo gemelo. Los centros de IoT vinculados pueden estar en cualquier región de Azure. Puede vincular centros de otras suscripciones a su servicio de aprovisionamiento.


## <a name="allocation-policy"></a>Directiva de asignación

La configuración del nivel de servicio que determina la forma en que el servicio Device Provisioning asigna dispositivos a un centro de IoT. Hay tres directivas de asignación admitidas:

* **Distribución uniformemente ponderada**: los centros de IoT vinculados tienen la misma probabilidad de tener dispositivos aprovisionados para ellos. Es la configuración predeterminada. Si va a aprovisionar dispositivos para un único centro de IoT Hub, puede mantener esta configuración.

* **Latencia más baja**: los dispositivos se aprovisionan en un centro de IoT con la latencia más baja en el dispositivo. Si varios centros de IoT vinculados proporcionarían la misma latencia más baja, el servicio de aprovisionamiento aplica hash a los dispositivos a través de esos centros

* **Configuración estática a través de la lista de inscripción**: la especificación del centro de IoT deseado en la lista de inscripción tiene prioridad sobre la directiva de asignación del nivel de servicio.

* **Personalizado (use la función de Azure)** : Una directiva de asignación personalizada le ofrece más control sobre cómo se asignan los dispositivos a un centro de IoT. Esto se logra mediante el uso de código personalizado en una función de Azure para asignar dispositivos a un centro de IoT. El servicio de aprovisionamiento de dispositivos llama al código de función de Azure y proporciona toda la información pertinente sobre el dispositivo y la inscripción en el código. El código de la función se ejecuta y devuelve la información del centro de IoT para aprovisionar el dispositivo.

## <a name="enrollment"></a>Inscripción

Una inscripción es el registro de dispositivos o grupos de dispositivos que pueden registrarse con el aprovisionamiento automático. El registro de inscripción contiene información sobre el dispositivo o grupo de dispositivos, como:
- el [mecanismo de atestación](#attestation-mechanism) que emplea el dispositivo
- la configuración inicial deseada (opcional)
- el centro de IoT preferido
- el identificador de dispositivo elegido

Hay dos tipos de inscripciones que admite el servicio Device Provisioning:

### <a name="enrollment-group"></a>Grupo de inscripción

Un grupo de inscripción es un grupo de dispositivos que comparten un mecanismo de atestación específico. Los grupos de inscripción admiten X. 509 y claves simétricas. Todos los dispositivos del grupo de inscripción X.509 presentan certificados X.509 que ha firmado la misma entidad de certificación (CA) raíz o intermedia. Cada dispositivo del grupo de inscripción de clave simétrica presenta tokens de SAS derivados de la clave simétrica de grupo. Tanto el nombre del grupo de inscripción como el nombre del certificado deben ser alfanuméricos, estar en minúsculas y pueden contener guiones.

> [!TIP]
> Se recomienda usar un grupo de inscripción para un gran número de dispositivos que compartan la configuración inicial deseada o para dispositivos que vayan todos al mismo inquilino.

### <a name="individual-enrollment"></a>Inscripción individual

Una inscripción individual es una entrada para el registro de un único dispositivo. Las inscripciones individuales pueden usar certificados X.509 de hoja o tokens de SAS (de un módulo de plataforma segura físico o virtual) como mecanismos de atestación. El identificador de registro de una inscripción individual está formado por caracteres alfanuméricos y en minúscula, y puede contener guiones. En las inscripciones individuales se puede especificar el identificador de dispositivo del centro de IoT deseado.

> [!TIP]
> Se recomiendan las inscripciones individuales para los dispositivos que requieran configuraciones iniciales únicas o para los dispositivos que solo permitan la autenticación mediante tokens de SAS a través de TPM como mecanismo de atestación.


## <a name="attestation-mechanism"></a>Mecanismo de atestación

Un mecanismo de atestación es el método utilizado para confirmar la identidad de un dispositivo. El mecanismo de atestación se configura en una entrada de inscripción e indica al servicio de aprovisionamiento el método que debe usarse al comprobar la identidad de un dispositivo durante el registro.

> [!NOTE]
> IoT Hub utiliza un "esquema de autenticación" para un concepto similar en ese servicio.

El servicio de aprovisionamiento de dispositivos admite las siguientes formas de atestación:
* **Certificados X.509** basado en el flujo de autenticación de certificados X.509 estándar. Para más información, vea [Atestación de X.509](concepts-x509-attestation.md).
* **Módulo de plataforma segura (TPM)** basado en un desafío nonce, utilizando el estándar TPM para las claves para presentar un token de Firma de acceso compartido (SAS) firmado. Esto no requiere un módulo de plataforma segura (TPM) físico en el dispositivo, pero el servicio espera atestar usando la clave de aprobación para cada [especificación de TPM](https://trustedcomputinggroup.org/work-groups/trusted-platform-module/). Para más información, vea [Atestación de TPM](concepts-tpm-attestation.md).
* **Clave simétrica**, basada en los [tokens de seguridad](../iot-hub/iot-hub-dev-guide-sas.md#security-tokens) de las firmas de acceso compartido (SAS), que incluyen una firma hash y una fecha de expiración insertada. Para más información, consulte [Symmetric key attestation](concepts-symmetric-key-attestation.md) (Atestación de clave simétrica).


## <a name="hardware-security-module"></a>Módulo de seguridad de hardware

El módulo de seguridad de hardware, o HSM, se usa para el almacenamiento seguro y basado en hardware de secretos de dispositivo y es la forma más segura de almacenamiento secreto. Tanto los certificados X.509 como los tokens de SAS se pueden almacenar en el módulo de seguridad de hardware. Los módulos de seguridad de hardware se pueden usar con ambos mecanismos de atestación que admite el servicio de aprovisionamiento.

> [!TIP]
> Se recomienda usar un módulo con los dispositivos para almacenar los secretos de forma segura en estos.

Los secretos de dispositivo también pueden almacenarse con software (en memoria), pero es una forma menos segura de almacenamiento que un módulo.



## <a name="id-scope"></a>Ámbito de identificador

El ámbito de identificador se asigna a un servicio Device Provisioning cuando el usuario lo crea y se utiliza para identificar de forma única el servicio de aprovisionamiento específico a través del que se registrará el dispositivo . El ámbito de identificador los genera el servicio y es inmutable, lo que garantiza su exclusividad.

> [!NOTE]
> Dicha exclusividad es importante para las operaciones de implementación de ejecución prolongada y en escenarios de fusiones y adquisiciones.


## <a name="registration"></a>Registro

Se trata del registro de un dispositivo que se ha registrado o aprovisionado correctamente en un centro de IoT a través del servicio Device Provisioning. Los registros se crean automáticamente; se pueden eliminar, pero no se pueden actualizar.


## <a name="registration-id"></a>Identificador de registro

El identificador de registro se usa para identificar de forma única el registro de un dispositivo con el servicio Device Provisioning. El identificador de dispositivo debe ser único en el [ámbito del identificador](#id-scope) del servicio de aprovisionamiento. Cada dispositivo debe tener un identificador de registro. El identificador de registro es alfanumérico, no distingue mayúsculas de minúsculas y puede contener caracteres especiales, como dos puntos, punto final, guión bajo y guión.

* En el caso del módulo de plataforma segura, el propio módulo proporciona el identificador de registro.
* En el caso de la atestación basada en X.509, el identificador de registro se proporciona como el nombre del sujeto del certificado.

## <a name="device-id"></a>El identificador de dispositivo

El identificador de dispositivo es el mismo que aparece en IoT Hub. El identificador de dispositivo deseado puede establecerse en la entrada de inscripción, pero no es necesario establecerlo. Solo se admite el establecimiento del identificador de dispositivo elegido en inscripciones individuales. Si no se especifica ningún identificador de dispositivo deseado en la lista de inscripción, el identificador de registro se utiliza como identificador de dispositivo al registrar el dispositivo. Más información sobre [identificadores de dispositivo en IoT Hub](../iot-hub/iot-hub-devguide-identity-registry.md).



## <a name="operations"></a>Operations

Las operaciones son la unidad de facturación del servicio Device Provisioning. Una operación es la finalización correcta de una instrucción para el servicio. Las operaciones incluyen registros y repeticiones de registros de dispositivos, además de cambios del servicio, como agregar entradas en la lista de inscripción y actualizarlas.
