---
title: 'Inicio rápido: Aprovisionamiento de un dispositivo X.509 simulado en Azure IoT Hub mediante C#'
description: 'Inicio rápido: Creación y aprovisionamiento de un dispositivo X.509 mediante el SDK de dispositivos para C# para Azure IoT Hub Device Provisioning Service (DPS). En esta guía de inicio rápido se utilizan inscripciones individuales.'
author: wesmc7777
ms.author: wesmc
ms.date: 02/01/2021
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.devlang: csharp
ms.custom: mvc
ms.openlocfilehash: a328115c606cb4bface2d3dc7b8f0c502d063e9f
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107987441"
---
# <a name="quickstart-create-and-provision-an-x509-device-using-c-device-sdk-for-iot-hub-device-provisioning-service"></a>Inicio rápido: Creación y aprovisionamiento de un dispositivo X.509 mediante el SDK de dispositivos para C# de IoT Hub Device Provisioning Service

[!INCLUDE [iot-dps-selector-quick-create-simulated-device-x509](../../includes/iot-dps-selector-quick-create-simulated-device-x509.md)]

En estos pasos se muestra cómo usar el código de dispositivo a partir de los [ejemplos de Azure IoT para C#](https://github.com/Azure-Samples/azure-iot-samples-csharp) para aprovisionar un dispositivo X.509. En este artículo, va a ejecutar código de ejemplo de dispositivo en la máquina de desarrollo para conectarse a una instancia de IoT Hub mediante el servicio Device Provisioning.

## <a name="prerequisites"></a>Requisitos previos

Si no está familiarizado con el proceso de aprovisionamiento automático, revise la información general sobre [Aprovisionamiento](about-iot-dps.md#provisioning-process). Asegúrese de completar los pasos descritos en [Configuración del servicio Azure IoT Hub Device Provisioning con Azure Portal](./quick-setup-auto-provision.md) antes de continuar.

Azure IoT Hub Device Provisioning Service admite dos tipos de inscripciones:
- [Grupos de inscripción](concepts-service.md#enrollment-group): usados para inscribir varios dispositivos relacionados.
- [Inscripciones individuales](concepts-service.md#individual-enrollment): usadas para inscribir un solo dispositivo.

En este artículo se mostrarán las inscripciones individuales.

[!INCLUDE [IoT Device Provisioning Service basic](../../includes/iot-dps-basic.md)]

<a id="setupdevbox"></a>
## <a name="prepare-the-development-environment"></a>Preparación del entorno de desarrollo 

1. Asegúrese de que `git` está instalado en su máquina y se agrega a las variables de entorno accesibles para la ventana de comandos. Consulte las [herramientas de cliente de Git de Software Freedom Conservancy](https://git-scm.com/download/) para instalar la versión más reciente de las herramientas `git`, lo que incluye **Git Bash**, la aplicación de línea de comandos que puede usar para interactuar con su repositorio de Git local. 

1. Abra un símbolo del sistema o Git Bash. Clone el repositorio de GitHub de los ejemplos de Azure IoT para C#:
    
    ```bash
    git clone https://github.com/Azure-Samples/azure-iot-samples-csharp.git
    ```

1. Asegúrese de tener instalado el [SDK de .NET Core 3.1 o posterior](https://dotnet.microsoft.com/download) en la máquina. Para comprobar la versión, use el siguiente comando.

    ```bash
    dotnet --info
    ```

## <a name="create-a-self-signed-x509-device-certificate"></a>Creación de un certificado de dispositivo X.509 autofirmado

En esta sección, creará un certificado de prueba X.509 autofirmado utilizando `iothubx509device1` como nombre común del firmante. Es importante tener en cuenta los siguientes puntos:

* Los certificados autofirmados son solo para la realización de pruebas, no se deben usar en producción.
* La fecha de expiración predeterminada de un certificado autofirmado es de un año.
* El identificador del dispositivo IoT será el nombre común del firmante en el certificado. Asegúrese de usar un nombre de firmante que cumpla con los [requisitos de cadena del identificador de dispositivo](../iot-hub/iot-hub-devguide-identity-registry.md#device-identity-properties).

Usará código de ejemplo de [X509Sample](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/provisioning/Samples/device/X509Sample) para crear el certificado que se empleará con la entrada de inscripción individual en el dispositivo.


1. En un símbolo del sistema de PowerShell, cambie los directorios al directorio de proyecto para el ejemplo de aprovisionamiento de dispositivos X.509.

    ```powershell
    cd .\azure-iot-samples-csharp\provisioning\Samples\device\X509Sample
    ```

2. El código de ejemplo está configurado para usar los certificados X.509 almacenados en un archivo con formato PKCS12 protegido con contraseña (certificate.pfx). Además, necesita un archivo de certificado de clave pública (certificate.cer) para crear una inscripción individual más adelante en este inicio rápido. Para generar el certificado autofirmado y los archivos .cer y .pfx asociados, ejecute el siguiente comando:

    ```powershell
    PS D:\azure-iot-samples-csharp\provisioning\Samples\device\X509Sample> .\GenerateTestCertificate.ps1 iothubx509device1
    ```

3. El script le pide una contraseña PFX. Recuerde esta contraseña, ya que tendrá que volver a usarla para ejecutar el ejemplo. Puede ejecutar `certutil` para volcar el certificado y comprobar el nombre del firmante.

    ```powershell
    PS D:\azure-iot-samples-csharp\provisioning\Samples\device\X509Sample> certutil .\certificate.pfx
    Enter PFX password:
    ================ Certificate 0 ================
    ================ Begin Nesting Level 1 ================
    Element 0:
    Serial Number: 7b4a0e2af6f40eae4d91b3b7ff05a4ce
    Issuer: CN=iothubx509device1, O=TEST, C=US
     NotBefore: 2/1/2021 6:18 PM
     NotAfter: 2/1/2022 6:28 PM
    Subject: CN=iothubx509device1, O=TEST, C=US
    Signature matches Public Key
    Root Certificate: Subject matches Issuer
    Cert Hash(sha1): e3eb7b7cc1e2b601486bf8a733887a54cdab8ed6
    ----------------  End Nesting Level 1  ----------------
      Provider = Microsoft Strong Cryptographic Provider
    Signature test passed
    CertUtil: -dump command completed successfully.    
    ```

 ## <a name="create-an-individual-enrollment-entry-for-the-device"></a>Creación de una entrada de inscripción individual para el dispositivo


1. Inicie sesión en Azure Portal, seleccione el botón **Todos los recursos** del menú de la izquierda y abra el servicio de aprovisionamiento.

2. En el menú de Device Provisioning Service, seleccione **Administrar inscripciones**. Seleccione la pestaña **Inscripciones individuales** y seleccione el botón **Agregar inscripción individual** de la parte superior. 

3. En el panel **Agregar inscripción**, escriba la siguiente información:
   - Seleccione **X.509** como *Mecanismo* de atestación de identidad.
   - En el *Archivo .pem o .cer de certificado principal*, elija *Seleccionar un archivo* para seleccionar el archivo de certificado **certificate.cer** creado en los pasos anteriores.
   - Deje el valor de **Id. de dispositivo** en blanco. El dispositivo se aprovisionará con su id. de dispositivo establecido en el nombre común en el certificado X.509, **iothubx509device1**. Este nombre común también será el nombre usado para el identificador de registro de la entrada de inscripción individual. 
   - De forma opcional, puede proporcionar la siguiente información:
       - Seleccione un centro de IoT vinculado con el servicio de aprovisionamiento.
       - Actualice el **Estado inicial del dispositivo gemelo** con la configuración inicial deseada para el dispositivo.
   - Una vez completado, presione el botón **Guardar**. 

     [![Agregar inscripción individual para la atestación X.509 en el portal](./media/quick-create-simulated-device-x509-csharp/device-enrollment.png)](./media/quick-create-simulated-device-x509-csharp/device-enrollment.png#lightbox)
    
   Al realizar la inscripción correctamente, la entrada de la inscripción X.509 aparece como **iothubx509device1** en la columna *Id. de registro* de la pestaña *Inscripciones individuales*. 



## <a name="provision-the-device"></a>Aprovisionamiento del dispositivo

1. En la hoja **Información general** de su servicio de aprovisionamiento, anote el valor de **_Ámbito de id._**

    ![Extracción de información del punto de conexión del servicio Device Provisioning desde la hoja del portal](./media/quick-create-simulated-device-x509-csharp/copy-scope.png) 


2. Escriba el siguiente comando para compilar y ejecutar el ejemplo de aprovisionamiento de dispositivo X.509. Reemplace el valor de `<IDScope>` por el ámbito de id. de su servicio de aprovisionamiento. 

    El archivo de certificado será de forma predeterminada *./certificate.pfx* y solicitará la contraseña .pfx.  

    ```powershell
    dotnet run -- -s <IDScope>
    ```

    Si desea pasar todo como un parámetro, puede usar el siguiente formato de ejemplo.

    ```powershell
    dotnet run -- -s 0ne00000A0A -c certificate.pfx -p 1234
    ```


3. El dispositivo se conectará a DPS y se asignará a un centro de IoT. Además, el dispositivo enviará un mensaje de telemetría al centro de IoT.

    ```output
    Loading the certificate...
    Found certificate: 10952E59D13A3E388F88E534444484F52CD3D9E4 CN=iothubx509device1, O=TEST, C=US; PrivateKey: True
    Using certificate 10952E59D13A3E388F88E534444484F52CD3D9E4 CN=iothubx509device1, O=TEST, C=US
    Initializing the device provisioning client...
    Initialized for registration Id iothubx509device1.
    Registering with the device provisioning service...
    Registration status: Assigned.
    Device iothubx509device2 registered to sample-iot-hub1.azure-devices.net.
    Creating X509 authentication for IoT Hub...
    Testing the provisioned device with IoT Hub...
    Sending a telemetry message...
    Finished.
    ```

4. Compruebe que el dispositivo se haya aprovisionado. Si el dispositivo se aprovisiona correctamente en el centro de IoT vinculado con su servicio de aprovisionamiento, el identificador del dispositivo aparecerá en la hoja **Dispositivos IoT** del centro de IoT. 

    ![El dispositivo se registra con el centro de IoT](./media/quick-create-simulated-device-x509-csharp/registration.png) 

    Si ha cambiado el valor predeterminado de *Estado inicial del dispositivo gemelo* en la entrada de inscripción para el dispositivo, el dispositivo puede extraer el estado gemelo deseado desde el centro y actuar en consecuencia. Para más información, consulte [Información y uso de dispositivos gemelos en IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md)


## <a name="clean-up-resources"></a>Limpieza de recursos

Si planea seguir trabajando con el ejemplo de cliente de dispositivo y explorándolo, no limpie los recursos que se crean en este inicio rápido. Si no va a continuar, use el siguiente comando para eliminar todos los recursos que se han creado en este inicio rápido.

1. Cierre la ventana de salida de ejemplo del cliente del dispositivo en su máquina.
1. En el menú de la izquierda de Azure Portal, seleccione **Todos los recursos** y seleccione Device Provisioning Service. En la parte superior de la hoja **Información general**, presione **Eliminar** en la parte superior del panel.  
1. En el menú de la izquierda de Azure Portal, seleccione **Todos los recursos** y después su centro de IoT. En la parte superior de la hoja **Información general**, presione **Eliminar** en la parte superior del panel.  

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha aprovisionado un dispositivo X.509 en su centro de IoT mediante Azure IoT Hub Device Provisioning Service. Para aprender a inscribir un dispositivo X.509 mediante programación, pase al inicio rápido para la inscripción de dispositivos X.509 mediante programación. 

> [!div class="nextstepaction"]
> [Inicio rápido de Azure: Inscripción de dispositivos X.509 en Azure IoT Hub Device Provisioning Service](quick-enroll-device-x509-csharp.md)
