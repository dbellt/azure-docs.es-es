---
title: 'Tutorial: Aprovisionamiento de un dispositivo mediante el servicio Azure IoT Hub Device Provisioning'
description: En este tutorial se indica cómo aprovisionar el dispositivo en un único centro de IoT mediante Azure IoT Hub Device Provisioning Service (DPS)
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.custom: mvc
ms.openlocfilehash: 94568eb5d64f07133ebb9992cdae59400830dd5e
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107226895"
---
# <a name="tutorial-provision-the-device-to-an-iot-hub-using-the-azure-iot-hub-device-provisioning-service"></a>Tutorial: Aprovisionamiento del dispositivo en una instancia de IoT Hub mediante el servicio Azure IoT Hub Device Provisioning

En el tutorial anterior, aprendió a configurar un dispositivo para conectarse al servicio Device Provisioning. En este tutorial, aprenderá a usar este servicio para aprovisionar el dispositivo en una única instancia de IoT Hub mediante el aprovisionamiento automático y las **_listas de inscripción_** . En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Inscribir el dispositivo
> * Iniciar el dispositivo
> * Comprobar que el dispositivo está registrado

## <a name="prerequisites"></a>Prerrequisitos

Antes de continuar, asegúrese de configurar el dispositivo como se describe en el tutorial [Configuración del dispositivo para el aprovisionamiento mediante el servicio Azure IoT Hub Device Provisioning](./tutorial-set-up-device.md).

Si no está familiarizado con el proceso de aprovisionamiento automático, revise la información general sobre el [aprovisionamiento](about-iot-dps.md#provisioning-process) antes de continuar.

<a id="enrolldevice"></a>
## <a name="enroll-the-device"></a>Inscribir el dispositivo

Este paso incluye la incorporación de artefactos de seguridad únicos del dispositivo al servicio Device Provisioning. Estos artefactos de seguridad se basan en el [mecanismo de atestación](concepts-service.md#attestation-mechanism) del dispositivo como se indica a continuación:

- Para dispositivos basados en TPM necesita:
    - La *clave de aprobación* que es exclusiva de cada chip TPM o simulación y que se obtiene del fabricante.  Consulte [Descripción de la clave de aprobación de TPM](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc770443(v=ws.11)) para más información.
    - El *identificador de registro* que se usa para identificar de forma única un dispositivo en el espacio de nombres o el ámbito. Este identificador puede ser igual que el identificador de dispositivo o no. El identificador es obligatorio para todos los dispositivos. En el caso de los dispositivos basados en TPM, el identificador de registro puede derivarse del propio TPM, por ejemplo, un hash SHA-256 de la clave de aprobación del TPM.

      [![Información de inscripción del TPM en el portal](./media/tutorial-provision-device-to-hub/tpm-device-enrollment.png)](./media/tutorial-provision-device-to-hub/tpm-device-enrollment.png#lightbox)  

- Para dispositivos basados en X.509 necesita:
    - El [certificado emitido para el chip o simulación X.509](/windows/win32/seccertenroll/about-x-509-public-key-certificates), en un formato de archivo *.pem* o *.cer*. Para la inscripción individual, debe usar el *certificado del firmante* de cada dispositivo para su sistema X.509, mientras que para la inscripción de grupos, debe usar el *certificado raíz*. 

      [![Agregar inscripción individual para la atestación X.509 en el portal](./media/tutorial-provision-device-to-hub/individual-enrollment.png)](./media/tutorial-provision-device-to-hub/individual-enrollment.png#lightbox)

Hay dos maneras de inscribir el dispositivo en el servicio Device Provisioning:

- **Grupos de inscripción**: esta representa un grupo de dispositivos que comparten un mecanismo de atestación específico. Se recomienda usar un grupo de inscripción para un gran número de dispositivos que compartan la configuración inicial deseada o para dispositivos que vayan todos al mismo inquilino. Para más información sobre la atestación de identidad para grupos de inscripción, consulte [Seguridad](concepts-x509-attestation.md#controlling-device-access-to-the-provisioning-service-with-x509-certificates).

    [![Agregar inscripción de grupo para la atestación X.509 en el portal](./media/tutorial-provision-device-to-hub/group-enrollment.png)](./media/tutorial-provision-device-to-hub/group-enrollment.png#lightbox)

- **Inscripciones individuales**: esta representa una entrada de un único dispositivo que se puede registrar con el servicio Device Provisioning. Las inscripciones individuales pueden usar certificados X.509 o tokens de SAS (en un módulo de plataforma segura real o virtual) como mecanismos de atestación. Se recomiendan las inscripciones individuales para los dispositivos que requieran configuraciones iniciales únicas y los dispositivos que solo puedan utilizar tokens de SAS a través de módulos de plataforma segura reales o virtuales como mecanismo de atestación. En las inscripciones individuales se puede especificar el identificador de dispositivo del centro de IoT deseado.

Ahora puede inscribir el dispositivo con la instancia del servicio Device Provisioning, mediante los artefactos de seguridad basados en el mecanismo de atestación del dispositivo: 

1. Inicie sesión en Azure Portal, haga clic en el botón **Todos los recursos** situado en el menú izquierdo y abra el servicio Device Provisioning.

2. En la hoja de resumen del servicio Device Provisioning, seleccione **Manage enrollments** (Administrar inscripciones). Seleccione la pestaña **Inscripciones individuales** o **Grupos de inscripción** según la configuración del dispositivo. Haga clic en el botón **Agregar** de la parte superior. Seleccione **TPM** o **X.509** como *mecanismo* de atestación de identidad y escriba los artefactos de seguridad adecuados, según lo descrito anteriormente. Puede escribir un nuevo **identificador de dispositivo de IoT Hub**. Una vez completado, haga clic en el botón **Guardar**. 

3. Cuando el dispositivo se haya inscrito satisfactoriamente, verá que aparece en el portal como sigue:

    ![Inscripción de TPM satisfactoria en el portal](./media/tutorial-provision-device-to-hub/tpm-enrollment-success.png)

Después de la inscripción, el servicio de aprovisionamiento espera a que cualquiera de los dispositivos arranque y se conecte con él en cualquier momento posterior en el tiempo. Cuando el dispositivo arranca por primera vez, la biblioteca de cliente SDK interactúa con el chip para extraer los artefactos de seguridad del dispositivo y comprueba el registro con el servicio Device Provisioning. 

## <a name="start-the-iot-device"></a>Inicio del dispositivo IoT

El dispositivo IoT puede ser un dispositivo real o un dispositivo simulado. Como el dispositivo IoT se ha inscrito con una instancia de Device Provisioning Service, este puede iniciarse ahora y llamar al servicio de aprovisionamiento para que lo reconozca mediante el mecanismo de atestación. Una vez que el servicio de aprovisionamiento ha reconocido el dispositivo, este se asignará a un centro de IoT. 

Se incluyen ejemplos de dispositivos simulados, mediante atestación TPM y X.509, para C, Java, C#, Node.js y Python. Por ejemplo, un dispositivo simulado con TPM y el [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c) seguiría el proceso descrito en la sección [Simulate first boot sequence for a device](quick-create-simulated-device.md#simulate-first-boot-sequence-for-the-device) (Simulación de la primera secuencia de inicio de un dispositivo). El mismo dispositivo con la atestación de certificado X.509 haría referencia a esta sección de la [secuencia de inicio](quick-create-simulated-device-x509.md#simulate-first-boot-sequence-for-the-device).

Consulte la [Guía de procedimientos de MXChip Iot DevKit](how-to-connect-mxchip-iot-devkit.md) como ejemplo para un dispositivo real.

Inicie el dispositivo para permitir que la aplicación cliente de este inicie el registro con el servicio Device Provisioning.  

## <a name="verify-the-device-is-registered"></a>Comprobar que el dispositivo está registrado

Cuando arranca el dispositivo, se deben realizar las siguientes acciones:

1. El dispositivo envía una solicitud de registro al servicio Device Provisioning.
2. En el caso de los dispositivos TPM, el servicio Device Provisioning devuelve un desafío de registro al que responde el dispositivo. 
3. Una vez que el registro se ha realizado correctamente, el servicio Device Provisioning envía el identificador URI del centro de IoT, el del dispositivo y la clave cifrada nuevamente al dispositivo. 
4. Luego, la aplicación cliente de IoT Hub del dispositivo se conecta al centro. 
5. Después de una conexión correcta, el dispositivo aparecerá en el explorador **Dispositivos IoT** de IoT Hub. 

    ![Conexión correcta al centro en el portal](./media/tutorial-provision-device-to-hub/hub-connect-success.png)

Para obtener más información, consulte el ejemplo de cliente de dispositivo de aprovisionamiento, [prov_dev_client_sample.c](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/samples/prov_dev_client_sample/prov_dev_client_sample.c). En el ejemplo se muestra el aprovisionamiento de un dispositivo simulado mediante TPM, los certificados X.509 y las claves simétricas. Remítase a las guías de inicio rápido de atestación de [TPM](./quick-create-simulated-device.md), [X.509](./quick-create-simulated-device-x509.md) y [Clave simétrica](./quick-create-simulated-device-symm-key.md) para obtener instrucciones paso a paso sobre el uso del ejemplo.

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Inscribir el dispositivo
> * Iniciar el dispositivo
> * Comprobar que el dispositivo está registrado

Pase al siguiente tutorial para aprender a aprovisionar varios dispositivos en centros con equilibrio de carga.

> [!div class="nextstepaction"]
> [Aprovisionamiento de dispositivos en instancias de IoT Hub con equilibrio de carga](./tutorial-provision-multiple-hubs.md)