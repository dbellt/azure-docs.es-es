---
title: 'Azure IoT Hub Device Provisioning Service: atestación del certificado X.509'
description: Describe conceptos específicos del uso de la atestación del certificado X.509 con Device Provisioning Service (DPS) e IoT Hub
author: wesmc7777
ms.author: wesmc
ms.date: 09/14/2020
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.openlocfilehash: d3847ec5ce253d2d5f2ad18ce9a0cc912335e2f4
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108142004"
---
# <a name="x509-certificate-attestation"></a>Atestación de certificados de cliente X.509

En este artículo se proporciona información general sobre conceptos relativos al servicio Device Provisioning durante el aprovisionamiento de dispositivos mediante la atestación de certificados X.509. Este artículo es apropiado para todas las personas implicadas en la preparación de un dispositivo para la implementación.

Los certificados X.509 se pueden almacenar en un HSM de módulo de seguridad de hardware.

> [!TIP]
> Se recomienda usar un HSM con los dispositivos para almacenar de forma segura los secretos, como el certificado X.509, en los dispositivos de producción.


## <a name="x509-certificates"></a>Certificados X.509

El uso de certificados X.509 como un mecanismo de atestación es una manera excelente para escalar la producción y simplificar el aprovisionamiento de dispositivos. Los certificados X.509 normalmente están organizados en una cadena de certificados de confianza en la que cada certificado de la cadena está firmado por la clave privada del certificado superior siguiente y así sucesivamente, terminando en un certificado raíz autofirmado. Esta opción establece una cadena de confianza delegada a partir del certificado raíz que generó una entidad de certificación (CA) raíz de confianza, a través de cada entidad de certificación intermedia y hasta el certificado "de hoja" de entidad final instalado en un dispositivo. Para más información consulte [Autenticación de dispositivos mediante certificados de entidades de certificación X.509](../iot-hub/iot-hub-x509ca-overview.md). 

A menudo la cadena de certificados representa alguna jerarquía lógica o física asociada con los dispositivos. Por ejemplo, un fabricante puede:
- emitir un certificado raíz de entidad de certificación autofirmado
- usar el certificado raíz para generar un certificado de entidad de certificación intermedia único para cada fábrica
- usar el certificado de cada fábrica para generar un certificado de entidad de certificación intermedia único para cada línea de producción en la planta
- y, por último, utilizar el certificado de la línea de producción para generar un certificado de dispositivo único (entidades finales) para cada dispositivo que se fabrica en la línea. 

Para más información consulte [Explicación de los conceptos de certificados de entidad de certificación X.509 en el sector de IoT](../iot-hub/iot-hub-x509ca-concept.md). 

### <a name="root-certificate"></a>Certificado raíz

Un certificado raíz es un certificado X.509 autofirmado que representa una entidad de certificación (CA). Es el término o anclaje de veracidad de la cadena de certificados. Una organización puede emitir sus propios certificados raíz o puede adquirirlos a través de una entidad de certificación raíz. Para más información, consulte [Obtención de certificados de entidad de certificación X.509](../iot-hub/tutorial-x509-scripts.md). También se puede hacer referencia al certificado raíz como un certificado de CA raíz.

### <a name="intermediate-certificate"></a>Certificado intermedio

Un certificado intermedio es un certificado X.509 que ha sido firmado por el certificado raíz (o por otro certificado que actúa como certificado intermedio en la cadena hasta el certificado raíz). El último certificado intermedio en una cadena se usa para firmar el certificado de hoja. También se puede hacer referencia a un certificado intermedio como un certificado de CA intermedio.

##### <a name="why-are-intermediate-certs-useful"></a>¿Por qué son útiles los certificados intermedios?
Los certificados intermedios se utilizan de varias maneras. Por ejemplo, se pueden usar para agrupar dispositivos en función de líneas de producto, clientes que adquieran dispositivos, divisiones de la empresa o fábricas. 

Supongamos que Contoso es una gran corporación con su propia infraestructura de clave pública (PKI) que usa el certificado raíz denominado *ContosoRootCert*. Cada subsidiaria de Contoso tiene su propio certificado intermedio que está firmado por *ContosoRootCert*. Posteriormente, cada subsidiaria usará su certificado intermedio para firmar sus certificados de hoja para cada dispositivo. En este escenario, Contoso puede usar una única instancia de DPS en la que *ContosoRootCert* se ha comprobado mediante una [prueba de posesión](./how-to-verify-certificates.md). Puede haber un grupo de inscripción para cada subsidiaria. De esta forma, cada subsidiaria individual no tendrá que ocuparse de comprobar los certificados.


### <a name="end-entity-leaf-certificate"></a>Certificado de entidad final "de hoja"

El certificado de hoja, o certificado de entidad final, identifica al titular del certificado. Tiene el certificado raíz en su cadena de certificados, así como cero o más certificados intermedios. El certificado de hoja no se usa para firmar otros certificados. Identifica de forma exclusiva al dispositivo ante el servicio de aprovisionamiento, y a veces se conoce como certificado del dispositivo. Durante la autenticación, el dispositivo usa la clave privada asociada con este certificado para responder a una prueba de desafío de posesión del servicio.

Los certificados de hoja usados con una entrada de [inscripción Individual](./concepts-service.md#individual-enrollment) requieren que el **nombre del sujeto** se establezca en el identificador de registro de la entrada de la inscripción individual. Los certificados de hoja utilizados con una entrada de [grupo de inscripción](./concepts-service.md#enrollment-group) deben tener configurado como **nombre del firmante** el identificador del dispositivo deseado que se mostrará en **Entradas de registro** para el dispositivo autenticado en el grupo de inscripción.

Para más información consulte [Autenticación de dispositivos firmados con certificados de entidad de certificación X.509](../iot-hub/iot-hub-x509ca-overview.md#authenticating-devices-signed-with-x509-ca-certificates).

## <a name="controlling-device-access-to-the-provisioning-service-with-x509-certificates"></a>Control del acceso de dispositivo al servicio de aprovisionamiento con certificados X.509

El servicio de aprovisionamiento expone dos tipos de inscripción que se puede usar para controlar el acceso a dispositivos mediante el mecanismo de atestación X.509:  

- Las entradas de [inscripción individual](./concepts-service.md#individual-enrollment) se configuran con el certificado de dispositivo asociado a un dispositivo específico. Estas entradas controlan las inscripciones para dispositivos específicos.
- Las entradas de [grupo de inscripción](./concepts-service.md#enrollment-group) están asociadas con un certificado de CA raíz o intermedio específico. Estas entradas controlan las inscripciones para todos los dispositivos que tienen ese certificado intermedio o raíz en su cadena de certificados. 

#### <a name="dps-device-chain-requirements"></a>Requisitos de la cadena de dispositivo de DPS

Cuando un dispositivo está intentando registrarse a través de DPS mediante un grupo de inscripción, debe enviar la cadena de certificados desde el certificado de hoja a un certificado comprobado mediante una [prueba de posesión](how-to-verify-certificates.md). En caso contrario, se producirá un error de autenticación.

Por ejemplo, si únicamente se comprueba el certificado raíz y se carga un certificado intermedio en el grupo de inscripción, el dispositivo debe presentar la cadena de certificados, desde el certificado de hoja al certificado raíz comprobado. Esta cadena de certificados incluiría cualquier certificado intermedio. Si el servicio DPS no puede atravesar la cadena de certificados hasta llegar a un certificado comprobado, se producirá un error de autenticación.

Por ejemplo, imagine que una corporación usa la siguiente cadena para un dispositivo.

![Ejemplo de cadena de certificados de dispositivo](./media/concepts-x509-attestation/example-device-cert-chain.png) 

Solo se comprueba el certificado raíz, y el certificado *intermediate2* se carga en el grupo de inscripción.

![Ejemplo de certificado raíz comprobado](./media/concepts-x509-attestation/example-root-verified.png) 

Si el dispositivo solo envía la siguiente cadena de dispositivo durante el aprovisionamiento, se producirá un error de autenticación. Esto se debe a que el servicio DPS no puede intentar la autenticación basándose en la supuesta validez del certificado *intermediate1*.

![Ejemplo de cadena de certificados que genera error](./media/concepts-x509-attestation/example-fail-cert-chain.png) 

Si el dispositivo envía la cadena de dispositivo completa durante el aprovisionamiento, como se indica a continuación, DPS puede intentar la autenticación del dispositivo.

![Ejemplo de cadena de certificados de dispositivo](./media/concepts-x509-attestation/example-device-cert-chain.png) 




> [!NOTE]
> Los certificados intermedios también se pueden comprobar mediante una [prueba de posesión](how-to-verify-certificates.md).


#### <a name="dps-order-of-operations-with-certificates"></a>Orden de las operaciones de DPS con certificados
Cuando un dispositivo se conecta al servicio de aprovisionamiento, el servicio da prioridad a las entradas de inscripción más específicas sobre las entradas de inscripción menos específicas. Es decir, si existe una inscripción individual para el dispositivo, el servicio de aprovisionamiento aplica a esa entrada. Si no hay ninguna inscripción individual para el dispositivo y existe un grupo de inscripción para el primer certificado intermedio en la cadena de certificados del dispositivo, el servicio aplica esa entrada, y así sucesivamente a lo largo de la cadena hasta la raíz. El servicio aplica la primera entrada aplicable que encuentra de forma que:

- Si la primera entrada de inscripción que se encuentra está habilitada, el servicio aprovisiona el dispositivo.
- Si la primera entrada de inscripción que se encuentra está deshabilitada, el servicio no aprovisiona el dispositivo.  
- Si no se encuentra ninguna entrada de inscripción para cualquiera de los certificados de la cadena de certificados del dispositivo, el servicio no aprovisiona el dispositivo. 

Este mecanismo y la estructura jerárquica de las cadenas de certificados proporciona una flexibilidad eficaz en la forma de controlar el acceso de dispositivos individuales, así como el de grupos de dispositivos. Por ejemplo, imagine cinco dispositivos con las siguientes cadenas de certificado: 

- *Dispositivo 1*: certificado raíz -> certificado A -> certificado de dispositivo 1
- *Dispositivo 2*: certificado raíz -> certificado A -> certificado de dispositivo 2
- *Dispositivo 3*: certificado raíz -> certificado A -> certificado de dispositivo 3
- *Dispositivo 4*: certificado raíz -> certificado B -> certificado de dispositivo 4
- *Dispositivo 5*: certificado raíz -> certificado B -> certificado de dispositivo 5

En principio, puede crear una única entrada de inscripción de grupo habilitada para el certificado raíz para habilitar el acceso para los cinco dispositivos. Si más adelante el certificado B se pone en peligro, puede crear una entrada de grupo de inscripción deshabilitada para el certificado B para evitar que el *Dispositivo 4* y el *Dispositivo 5* se inscriban. Si después el *Dispositivo 3* se pone en peligro, puede crear una entrada de inscripción individual deshabilitada para su certificado. Esto revoca el acceso del *Dispositivo 3*, pero sigue permitiendo que el *Dispositivo 1* y el *Dispositivo 2* se inscriban.