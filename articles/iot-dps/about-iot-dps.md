---
title: Introducción a Azure IoT Hub Device Provisioning Service | Microsoft Docs
description: Describe el aprovisionamiento de dispositivos en Azure con servicio Device Provisioning Service (DPS) e IoT Hub
author: wesmc7777
ms.author: wesmc
ms.date: 04/04/2019
ms.topic: overview
ms.service: iot-dps
services: iot-dps
manager: eliotgra
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: e58028495da409df00db8e25adf68945a008bc9e
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107228306"
---
# <a name="what-is-azure-iot-hub-device-provisioning-service"></a>¿Qué es Azure IoT Hub Device Provisioning Service?
Microsoft Azure proporciona un amplio conjunto de servicios públicos en la nube integrados para todas las necesidades de su solución de IoT. IoT Hub Device Provisioning Service (DPS) es un servicio auxiliar para IoT Hub que habilita el aprovisionamiento sin interacción Just-In-Time a la instancia correcta del centro de IoT sin necesidad de intervención humana. DPS habilita el aprovisionamiento de millones de dispositivos de forma segura y escalable.

## <a name="when-to-use-device-provisioning-service"></a>Cuándo se debe usar el servicio Device Provisioning
Hay muchos escenarios de aprovisionamiento en los que DPS es una excelente opción para conectar dispositivos y configurarlos para IoT Hub como, por ejemplo:

* Aprovisionamiento sin intervención del usuario en una única solución de IoT sin codificar la información de conexión de IoT Hub instalada de fábrica (configuración inicial)
* Equilibrado de carga de dispositivos en varios centros
* Conexión de dispositivos a la solución de IoT de su propietario según los datos de transacción de ventas (multiinquilino)
* Conexión de dispositivos a una solución de IoT concreta en función del caso de uso (aislamiento de la solución)
* Conexión de un dispositivo a la instancia de IoT Hub con la latencia más baja (particionamiento geográfico)
* Reaprovisionamiento basado en un cambio del dispositivo
* Reversión de las claves utilizadas por el dispositivo para conectarse a IoT Hub (si no utiliza certificados X.509 para conectarse)

## <a name="behind-the-scenes"></a>Entre bambalinas
Todos los escenarios mencionados en la sección anterior se pueden realizar mediante DPS para el aprovisionamiento sin intervención del usuario con el mismo flujo. Muchos de los pasos manuales normalmente incluidos en el aprovisionamiento se automatizan con DPS para reducir el tiempo de implementación de los dispositivos de IoT y reducir el riesgo de error manual. La siguiente sección describe lo que ocurre en segundo plano a la hora de aprovisionar un dispositivo. El primer paso es manual pero todos los demás pasos se han automatizado.

![Flujo básico de aprovisionamiento](./media/about-iot-dps/dps-provisioning-flow.png)

1. El fabricante del dispositivo agrega la información de registro del dispositivo a la lista de inscripción en Azure Portal.
2. El dispositivo contacta con el punto de conexión de DPS configurado de fábrica. El dispositivo pasa la información de identificación a DPS para demostrar su identidad.
3. DPS valida la identidad del dispositivo mediante la validación del identificador del registro y la clave en la entrada de la lista de inscripción mediante el uso de un desafío nonce ([Módulo de plataforma segura](https://trustedcomputinggroup.org/work-groups/trusted-platform-module/)) o una comprobación estándar X.509 (X.509).
4. DPS registra el dispositivo con un IoT Hub y rellena el [estado deseado de los dispositivos gemelos](../iot-hub/iot-hub-devguide-device-twins.md).
5. El centro de IoT devuelve la información del identificador de dispositivo a DPS.
6. DPS devuelve la información de conexión del centro de IoT al dispositivo. El dispositivo ya puede empezar a enviar datos directamente a la instancia de IoT Hub.
7. El dispositivo se conecta a IoT Hub.
8. El dispositivo obtiene el estado deseado del dispositivo gemelo en la instancia de IoT Hub.

## <a name="provisioning-process"></a>Proceso de aprovisionamiento
Hay dos pasos diferentes en el proceso de implementación de un dispositivo, en los que DPS toma parte, que se pueden llevar a cabo de forma independiente:

* El **paso de fabricación** en el que se crea el dispositivo y se prepara la configuración de fábrica, y
* el **paso de configuración en la nube** en el que el servicio Device Provisioning se configura para el aprovisionamiento automatizado.

Ambos pasos se adaptan perfectamente con los procesos de fabricación e implementación existentes. DPS simplifica incluso algunos procesos de implementación que implican una gran cantidad de trabajo manual para obtener la información de conexión del dispositivo.

### <a name="manufacturing-step"></a>Paso de fabricación
Este paso es todo lo que sucede en la línea de fabricación. Los roles implicados en este paso incluyen diseñador de Silicon, fabricante de Silicon, integrador y/o el fabricante final del dispositivo. En este paso se crea el propio hardware.

DPS no introduce un nuevo paso en el proceso de fabricación; más bien, se vincula al paso existente que instala el software inicial e (idealmente) el HSM en el dispositivo. En lugar de crear un identificador de dispositivo en este paso, el dispositivo se programa con la información del servicio de aprovisionamiento, quedando así habilitado para llamar al servicio de aprovisionamiento para obtener la información de conexión y la asignación de solución de IoT al activarse.

También en este paso, el fabricante proporciona al operador o implementador del dispositivo la información clave de identificación. Proporcionar esa información puede ser tan simple como confirmar que todos los dispositivos tienen un certificado X.509 generado a partir de un certificado de firma proporcionado por el operador o el implementador del dispositivo o tan complicado como extraer la parte pública de una clave de aprobación de TPM de cada dispositivo TPM. Muchos fabricantes de Silicon ofrecen estos servicios hoy en día.

### <a name="cloud-setup-step"></a>Paso de configuración de la nube
Este paso trata sobre la configuración de la nube para un aprovisionamiento automático adecuado. Por lo general hay dos tipos de usuarios implicados en el paso de configuración de la nube: alguien que sabe cómo se deben configurar los dispositivos inicialmente (un operador del dispositivo) y otra persona que sabe cómo se van a dividir los dispositivos entre las instancias de IoT Hub (un operador de la solución).

Hay una configuración inicial única del aprovisionamiento que es obligatoria, algo que normalmente realiza el operador de la solución. Una vez configurado el servicio de aprovisionamiento, este no tiene que modificarse a menos que cambie el caso de uso.

Una vez configurado el servicio del aprovisionamiento automático, este debe estar preparado para inscribir dispositivos. Este lo realiza el operador del dispositivo, que conoce la configuración deseada de los dispositivos y es el responsable de garantizar que el servicio de aprovisionamiento pueda correctamente dar fe de la identidad del dispositivo cuando se trate de buscar su instancia de IoT Hub. El operador del dispositivo toma la información clave de identificación del fabricante y lo agrega a la lista de inscripción. Puede haber actualizaciones posteriores de la lista de inscripción a medida que se agregan nuevas entradas o se actualizan las entradas existentes con la información más reciente sobre los dispositivos.

## <a name="registration-and-provisioning"></a>Registro y aprovisionamiento
*Aprovisionamiento* tiene varios significados según el sector en el que se use el término. En el contexto del aprovisionamiento de dispositivos de IoT para su solución en la nube, el aprovisionamiento es un proceso de dos partes:

1. La primera consiste en establecer la conexión inicial entre el dispositivo y la solución de IoT registrando el dispositivo.
2. La segunda es aplicar la configuración apropiada en el dispositivo en función de los requisitos específicos de la solución en la que se registró.

Una vez que se han completado los dos pasos, podemos decir que el dispositivo ha sido totalmente aprovisionado. Algunos servicios en la nube solo proporcionan el primer paso del proceso de aprovisionamiento, registrando los dispositivos en el punto de conexión de la solución de IoT, pero sin proporcionar la configuración inicial. DPS automatiza ambos pasos para proporcionar una experiencia perfecta de aprovisionamiento para el dispositivo.

## <a name="features-of-the-device-provisioning-service"></a>Características del servicio Device Provisioning
DPS tiene muchas características que hacen que resulte idóneo para el aprovisionamiento de dispositivos.

* **Atestación segura**: compatible con las identidades X.509 y con las basadas en TPM.
* **Lista de inscripción** que contiene el registro completo de dispositivos o grupos de dispositivos que pueden registrarse en algún momento. La lista de inscripción contiene información sobre la configuración deseada del dispositivo una vez que se registra, y se puede actualizar en cualquier momento.
* **Varias directivas de asignación** para controlar la forma en que DPS asigna dispositivos a los centros de IoT para dar soporte a los escenarios: Menos latencia, distribución ponderada uniformemente (valor predeterminado) y configuración estática a través de la lista de inscripción. Para determinar la latencia, se usa el mismo método que [Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md#performance).
* **Registros de supervisión y diagnóstico** para asegurarse de que todo funciona correctamente.
* La **compatibilidad con varios centros** permite que Data Protection Manager asigne dispositivos a más de un centro de IoT. DPS puede comunicarse con los centros de varias suscripciones de Azure.
* La **compatibilidad entre regiones** permite que DPS asigne dispositivos a los centros de IoT en otras regiones.
* El **cifrado de datos en reposo** permite que los datos de Data Protection Manager se cifren y descifren de forma transparente mediante el cifrado AES de 256 bits, uno de los cifrados de bloques más sólidos disponibles, y que es compatible con FIPS 140-2.


Para más información sobre los conceptos y las características implicadas en el aprovisionamiento de dispositivos, consulte el tema [Terminología de DPS](concepts-service.md), así como los restantes temas conceptuales de la misma sección.

## <a name="cross-platform-support"></a>Compatibilidad multiplataforma
DPS, al igual que todos los servicios de Azure IoT, funciona entre plataformas con diversos sistemas operativos. Azure ofrece distintos SDK de código abierto en una variedad de [lenguajes](https://github.com/Azure/azure-iot-sdks) para facilitar la conexión de dispositivos y la administración del servicio. DPS admite los siguientes protocolos de conexión de dispositivos:

* HTTPS
* AMQP
* AMQP sobre Web Sockets
* MQTT
* MQTT sobre Web Sockets

DPS solo admite conexiones HTTPS para las operaciones de servicio.

## <a name="regions"></a>Regions
DPS está disponible en muchas regiones. Hay una lista actualizada de las regiones existentes y de las recién anunciadas para todos los servicios en [Regiones de Azure](https://azure.microsoft.com/regions/). Puede consultar la disponibilidad del servicio Device Provisioning en la página [Estado de Azure](https://azure.microsoft.com/status/).

> [!NOTE]
> DPS es global y no está enlazado a una ubicación. Sin embargo, debe especificar una región en la que van a residir los metadatos asociados con el perfil de DPS.

## <a name="availability"></a>Disponibilidad
Hay un Acuerdo de Nivel de Servicio del 99,9 % para DPS y también puede [leer el SLA](https://azure.microsoft.com/support/legal/sla/iot-hub/). En el [SLA de Azure](https://azure.microsoft.com/support/legal/sla/) completo se explica la disponibilidad garantizada de Azure como un conjunto.

## <a name="quotas"></a>Cuotas
Cada suscripción de Azure tiene límites de cuota predeterminados que pueden afectar al ámbito de su solución de IoT. El límite actual por suscripción es de 10 servicios Device Provisioning por suscripción.

[!INCLUDE [azure-iotdps-limits](../../includes/iot-dps-limits.md)]

Más información sobre los límites de cuota:
* [Límites de servicio de suscripción de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md)

## <a name="related-azure-components"></a>Componentes de Azure relacionados
DPS permite automatizar el aprovisionamiento de dispositivos con Azure IoT Hub. Más información sobre [IoT Hub](../iot-hub/index.yml).

## <a name="next-steps"></a>Pasos siguientes
Ahora tiene una visión general sobre el aprovisionamiento de dispositivos de IoT en Azure. El paso siguiente es probar un escenario global de IoT.

[Configuración de IoT Hub Device Provisioning Service con Azure Portal](quick-setup-auto-provision.md)

[Creación y aprovisionamiento de un dispositivo simulado](quick-create-simulated-device.md)

[Configurar el dispositivo para el aprovisionamiento](tutorial-set-up-device.md)