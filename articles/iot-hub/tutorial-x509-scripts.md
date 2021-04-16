---
title: 'Tutorial: Uso de scripts de Microsoft para crear certificados de prueba X.509 para Azure IoT Hub | Microsoft Docs'
description: 'Tutorial: Uso de scripts personalizados para crear certificados de entidad de certificación y de dispositivo para Azure IoT Hub'
author: v-gpettibone
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 02/26/2021
ms.author: robinsh
ms.custom:
- mvc
- 'Role: Cloud Development'
- 'Role: Data Analytics'
ms.openlocfilehash: ff4b63f49a87dd9ca6b0ef458bdcf1c285a34a18
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378217"
---
# <a name="tutorial-using-microsoft-supplied-scripts-to-create-test-certificates"></a>Tutorial: Uso de scripts proporcionados por Microsoft para crear certificados de prueba

Microsoft proporciona scripts de PowerShell y Bash para ayudarle a entender cómo crear sus propios certificados X.509 y autenticarlos en una instancia de IoT Hub. Los scripts se encuentran en un [repositorio](https://github.com/Azure/azure-iot-sdk-c/tree/master/tools/CACertificates) de GitHub. Se proporcionan únicamente con fines de demostración. Los certificados creados por ellos no deben usarse para producción. Los certificados contienen contraseñas codificadas de forma rígida ("1234") y expiran después de 30 días. En el caso de un entorno de producción, deberá usar sus propios procedimientos recomendados para la creación de certificados y la administración de la vigencia.

## <a name="powershell-scripts"></a>Scripts de PowerShell

### <a name="step-1---setup"></a>Paso 1: Configuración

Obtenga OpenSSL para Windows. Consulte <https://www.openssl.org/docs/faq.html#MISC4> para ver dónde descargarlo o <https://www.openssl.org/source/> para crearlo desde el origen. Después, ejecute los scripts preliminares:

1. Copie los scripts de este [repositorio](https://github.com/Azure/azure-iot-sdk-c/tree/master/tools/CACertificates) de GitHub en el directorio local en el que desea trabajar. Todos los archivos se crearán como elementos secundarios de este directorio.

1. Inicie PowerShell como administrador.

1. Cambie al directorio en el que cargó los scripts.

1. En la línea de comandos, establezca la variable de entorno `$ENV:OPENSSL_CONF` en el directorio en el que se encuentra el archivo de configuración de OpenSSL (openssl.cnf).

1. Ejecute `Set-ExecutionPolicy -ExecutionPolicy Unrestricted` para que PowerShell pueda ejecutar los scripts.

1. Ejecute `. .\ca-certs.ps1`. Esto trae las funciones del script al espacio de nombres global de PowerShell.

1. Ejecute `Test-CACertsPrerequisites`. PowerShell usa el almacén de certificados de Windows para administrar certificados. Este comando comprueba que no habrá conflictos de nombres más adelante con los certificados existentes y que OpenSSL está configurado correctamente.

### <a name="step-2---create-certificates"></a>Paso 2: Creación de certificados

Ejecute `New-CACertsCertChain [ecc|rsa]`. Se recomienda ECC para los certificados de entidad de certificación, pero no es obligatorio. Este script actualiza el directorio y el almacén de certificados de Windows con los certificados intermedios y de entidad de certificación siguientes:

* intermediate1.pem
* intermediate2.pem
* intermediate3.pem
* RootCA.cer
* RootCA.pem

Después de ejecutar el script, agregue el nuevo certificado de entidad de certificación (RootCA.pem) a la instancia de IoT Hub:

1. Vaya a la instancia de IoT Hub y, después, a Certificados.

1. Seleccione **Agregar**.

1. Escriba un nombre para mostrar para el certificado de la entidad de certificación.

1. Cargue el certificado de la entidad de certificación.

1. Seleccione **Guardar**.

### <a name="step-3---prove-possession"></a>Paso 3: Demostrar la posesión

Ahora que ha cargado el certificado de la entidad de certificación en la instancia de IoT Hub, deberá demostrar que realmente es de su propiedad:

1. Seleccione el nuevo certificado de la entidad de certificación.

1. En el cuadro de diálogo **Detalles del certificado**, haga clic en **Generar código de verificación**. Para más información, consulte [Demostración de la posesión de un certificado de entidad de certificación](tutorial-x509-prove-possession.md).

1. Cree un certificado que contenga el código de verificación. Por ejemplo, si el código de verificación es `"106A5SD242AF512B3498BD6098C4941E66R34H268DDB3288"`, ejecute lo siguiente para crear un nuevo certificado en el directorio de trabajo que contenga el firmante `CN = 106A5SD242AF512B3498BD6098C4941E66R34H268DDB3288`. El script crea un certificado denominado `VerifyCert4.cer`.

    `New-CACertsVerificationCert "106A5SD242AF512B3498BD609C4941E66R34H268DDB3288"`

1. Cargue `VerifyCert4.cer` en la instancia de IoT Hub en el cuadro de diálogo **Detalles del certificado**.

1. Seleccione **Comprobar**.

### <a name="step-4---create-a-new-device"></a>Paso 4: Creación de un nuevo dispositivo

Cree un dispositivo para la instancia de IoT Hub:

1. En IoT Hub, vaya a la sección **Dispositivos IoT**.

1. Agregue un nuevo dispositivo con el identificador `mydevice`.

1. Para la autenticación, elija **X.509 firmado por CA**.

1. Ejecute `New-CACertsDevice mydevice` para crear un nuevo certificado de dispositivo. Esto crea los siguientes archivos en el directorio de trabajo:

   * `mydevice.pfx`
   * `mydevice-all.pem`
   * `mydevice-private.pem`
   * `mydevice-public.pem`

### <a name="step-5---test-your-device-certificate"></a>Paso 5: Prueba del certificado de dispositivo

Vaya a [Prueba de la autenticación de certificados](tutorial-x509-test-certificate.md) para determinar si el certificado de dispositivo puede autenticarse en la instancia de IoT Hub. Necesitará la versión PFX del certificado, `mydevice.pfx`.

### <a name="step-6---cleanup"></a>Paso 6: Limpieza

En el menú Inicio, Abra **Administrar certificados de equipo** y vaya a **Certificados - Equipo local > Personal**. Elimine los certificados emitidos por "Azure IoT CA TestOnly*". Igualmente, elimine los certificados adecuados de **>Entidad de certificación raíz de confianza > Certificados > Entidades emisoras de certificados intermedios > Certificados**.

## <a name="bash-scripts"></a>Scripts de Bash

### <a name="step-1---setup"></a>Paso 1: Configuración

1. Inicie Bash.

1. Cambie al directorio en el que quiere trabajar. Todos los archivos se crearán en este directorio.

1. Copie `*.cnf` y `*.sh` en el directorio de trabajo.

### <a name="step-2---create-certificates"></a>Paso 2: Creación de certificados

1. Ejecute `./certGen.sh create_root_and_intermediate`. Esto crea los siguientes archivos en el directorio **certs**:

    * azure-iot-test-only.chain.ca.cert.pem
    * azure-iot-test-only.intermediate.cert.pem
    * azure-iot-test-only.root.ca.cert.pem

1. Vaya a la instancia de IoT Hub y, después, a **Certificados**.

1. Seleccione **Agregar**.

1. Escriba un nombre para mostrar para el certificado de la entidad de certificación.

1. Cargue solo el certificado de la entidad de certificación en la instancia de IoT Hub. El nombre del certificado es `./certs/azure-iot-test-only.root.ca.cert.pem.`.

1. Seleccione **Guardar**.

### <a name="step-3---prove-possession"></a>Paso 3: Demostrar la posesión

1. Seleccione el nuevo certificado de la entidad de certificación creado en el paso anterior.

1. En el cuadro de diálogo **Detalles del certificado**, haga clic en **Generar código de verificación**. Para más información, consulte [Demostración de la posesión de un certificado de entidad de certificación](tutorial-x509-prove-possession.md).

1. Cree un certificado que contenga el código de verificación. Por ejemplo, si el código de verificación es `"106A5SD242AF512B3498BD6098C4941E66R34H268DDB3288"`, ejecute lo siguiente para crear un nuevo certificado en el directorio de trabajo denominado `verification-code.cert.pem` que contenga el firmante `CN = 106A5SD242AF512B3498BD6098C4941E66R34H268DDB3288`.

    `./certGen.sh create_verification_certificate "106A5SD242AF512B3498BD6098C4941E66R34H268DDB3288"`

1. Cargue el certificado en la instancia de IoT Hub en el cuadro de diálogo **Detalles del certificado**.

1. Seleccione **Comprobar**.

### <a name="step-4---create-a-new-device"></a>Paso 4: Creación de un nuevo dispositivo

Cree un dispositivo para la instancia de IoT Hub:

1. En IoT Hub, vaya a la sección Dispositivos IoT.

1. Agregue un nuevo dispositivo con el identificador `mydevice`.

1. Para la autenticación, elija **X.509 firmado por CA**.

1. Ejecute `./certGen.sh create_device_certificate mydevice` para crear un nuevo certificado de dispositivo. Esto crea dos archivos denominados `new-device.cert.pem` y `new-device.cert.pfx` en el directorio de trabajo.

### <a name="step-5---test-your-device-certificate"></a>Paso 5: Prueba del certificado de dispositivo

Vaya a [Prueba de la autenticación de certificados](tutorial-x509-test-certificate.md) para determinar si el certificado de dispositivo puede autenticarse en la instancia de IoT Hub. Necesitará la versión PFX del certificado, `new-device.cert.pfx`.

### <a name="step-6---cleanup"></a>Paso 6: Limpieza

Como el script de Bash solo crea certificados en el directorio de trabajo, simplemente elimínelos cuando haya terminado las pruebas.

## <a name="next-steps"></a>Pasos siguientes

Para probar el certificado, vaya a la [prueba de la autenticación de certificados](tutorial-x509-test-certificate.md) para determinar si el certificado puede autenticar el dispositivo en la instancia de IoT Hub.
