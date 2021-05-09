---
title: 'Inicio rápido: Uso de una clave simétrica para aprovisionar un dispositivo en Azure IoT Hub mediante C#'
description: En este inicio rápido, usará el SDK de dispositivo de C# para el servicio Device Provisioning (DPS) para aprovisionar un dispositivo de clave simétrica en un centro de IoT
author: wesmc7777
ms.author: wesmc
ms.date: 04/23/2021
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
manager: eliotgra
ms.custom: mvc
ms.openlocfilehash: e67616c2c92676c3af79e3040bc09d3b1b87a11b
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107988305"
---
# <a name="quickstart-provision-a-symmetric-key-device-using-c"></a>Inicio rápido: Aprovisionamiento de un dispositivo con claves simétricas mediante C#

En este inicio rápido, aprenderá a aprovisionar una máquina de desarrollo Windows como un dispositivo en un centro de IoT mediante C#. Este dispositivo utilizará una clave simétrica y una inscripción individual para autenticarse con una instancia del servicio Device Provisioning (DPS) para ser asignado a un centro de IoT. Se usará código de ejemplo del [Ejemplos de Azure IoT para C#](https://github.com/Azure-Samples/azure-iot-samples-csharp) para aprovisionar el dispositivo. 

Aunque en este artículo se muestra el aprovisionamiento con una inscripción individual, también puede usar grupos de inscripción. Hay algunas diferencias al usar los grupos de inscripción. Por ejemplo, debe usar una clave de dispositivo derivada con un identificador de registro único para el dispositivo. En [Aprovisionamiento de dispositivos con claves simétrica](how-to-legacy-device-symm-key.md) encontrará un ejemplo de grupo de inscripción. Para más información sobre los grupos de inscripción, consulte la sección sobre las [inscripciones de grupo para la atestación de clave simétrica](concepts-symmetric-key-attestation.md#group-enrollments).

Si no está familiarizado con el proceso de aprovisionamiento automático, revise la información general sobre [Aprovisionamiento](about-iot-dps.md#provisioning-process). 

Además, asegúrese de completar los pasos descritos en [Configuración del servicio Azure IoT Hub Device Provisioning con Azure Portal](./quick-setup-auto-provision.md) antes de continuar con este inicio rápido. Para seguir esta guía de inicio rápido, ya debe haber creado la instancia del Device Provisioning Service.

Este artículo está orientado a una estación de trabajo basada en Windows. No obstante, también puede realizar los procedimientos en Linux. Para ver un ejemplo en Linux, consulte [Aprovisionamiento para varios inquilinos](how-to-provision-multitenant.md).


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]


## <a name="prerequisites"></a>Prerrequisitos

* Asegúrese de que tiene el [SDK de .NET Core 2.1](https://dotnet.microsoft.com/download) o una versión posterior instalada en la máquina basada en Windows.

* Tener instalada la versión más reciente de [Git](https://git-scm.com/download/).

<a id="setupdevbox"></a>


## <a name="create-a-device-enrollment"></a>Creación de una inscripción de dispositivos

1. Inicie sesión en [Azure Portal](https://portal.azure.com), seleccione el botón **Todos los recursos** situado en el menú izquierdo y abra la instancia de Device Provisioning Service (DPS).

2. Seleccione la pestaña **Administrar inscripciones** y, después, seleccione el botón **Agregar inscripción individual** de la parte superior. 

3. En el panel **Agregar inscripción**, escriba la siguiente información y presione el botón **Guardar**.

   - **Mecanismo:** seleccione **Clave simétrica** como *mecanismo* de atestación de identidad.

   - **Generar claves automáticamente**: marque esta casilla.

   - **Identificador de registro**: escriba un identificador de registro para identificar la inscripción. Use únicamente caracteres alfanuméricos en minúsculas y guiones (“-”). Por ejemplo, **symm-key-csharp-device-01**.

   - **Id. de dispositivo de IoT Hub:** escriba un identificador de dispositivo. Por ejemplo, **csharp-device-01**.

     ![Agregar una inscripción individual para la atestación de clave simétrica en el portal](./media/quick-create-device-symmetric-key-csharp/create-individual-enrollment-csharp.png)

4. Cuando haya guardado la inscripción, se generarán la **clave principal** y la **clave secundaria**, y se agregarán a la entrada de la inscripción. La inscripción del dispositivo con clave simétrica se muestra como **symm-key-csharp-device-01** en la columna *Id. de registro* de la pestaña *Inscripciones individuales*. 

5. Abra la inscripción y copie el valor de su **clave principal** generada. Usará este valor de clave y el **identificador de registro** más adelante cuando ejecute el código de ejemplo de aprovisionamiento de dispositivos.



## <a name="prepare-the-c-environment"></a>Preparación del entorno de C# 

1. Abra un entorno de línea de comandos de Git CMD o Git Bash. Utilice el siguiente comando para clonar el repositorio de GitHub de los [ejemplos de Azure IoT para C#](https://github.com/Azure-Samples/azure-iot-samples-csharp):

    ```cmd
    git clone https://github.com/Azure-Samples/azure-iot-samples-csharp.git
    ```



<a id="firstbootsequence"></a>

## <a name="run-the-device-provisioning-code"></a>Ejecución del código de aprovisionamiento de dispositivos

En esta sección, ejecutará el ejemplo de aprovisionamiento de dispositivos con tres parámetros que autenticarán el código de ejemplo de aprovisionamiento de dispositivos como el dispositivo de clave simétrica para la inscripción en el recurso de DPS. Estos tres parámetros son:

* Ámbito de id.
* Identificador de registro de una inscripción individual.
* Clave simétrica principal de una inscripción individual.

El código de aprovisionamiento se pondrá en contacto con el recurso de DPS con estos parámetros para autenticar el dispositivo. Después, el dispositivo se asignará a un centro de IoT que ya está vinculado a la instancia de DPS en función de la configuración de inscripción individual. Una vez aprovisionado, el código de ejemplo enviará un mensaje de telemetría de prueba al centro de IoT.

1. En [Azure Portal](https://portal.azure.com), en el menú Device Provisioning Service, seleccione **Información general** y copie el valor de **Ámbito del identificador**. Usará este valor para el parámetro `IdScope` al ejecutar el código de ejemplo.

2. Abra un símbolo del sistema y vaya a *SymmetricKeySample* en el repositorio de ejemplos clonados:

    ```cmd
    cd azure-iot-samples-csharp\provisioning\Samples\device\SymmetricKeySample
    ```

3. En la carpeta *SymmetricKeySample*, abra el archivo *Parameters.cs* en un editor de texto. Este archivo muestra los parámetros admitidos por el ejemplo. En este artículo, solo se usarán los tres primeros parámetros necesarios al ejecutar el ejemplo. Revise el código de este archivo. No es necesario realizar ningún cambio.
 
    | Parámetro                         | Obligatorio | Descripción     |
    | :-------------------------------- | :------- | :-------------- |
    | `--s` o `--IdScope`              | True     | Ámbito del identificador de la instancia de DPS. |
    | `--i` o `--Id`                   | True     | Identificador de registro cuando se usa la inscripción individual o identificador del dispositivo deseado cuando se usa la inscripción de grupo. |
    | `--p` o `--PrimaryKey`           | True     | Clave principal de la inscripción individual o de grupo. |
    | `--e` o `--EnrollmentType`       | False    | Tipo de inscripción: `Individual` o `Group`. De manera predeterminada, su valor es `Individual`. |
    | `--g` o `--GlobalDeviceEndpoint` | False    | Punto de conexión global al que se conectarán los dispositivos. De manera predeterminada, su valor es `global.azure-devices-provisioning.net`. |
    | `--t` o `--TransportType`        | False    | Transporte que se va a utilizar para comunicarse con la instancia de Device Provisioning. Su valor predeterminado es `Mqtt`. Los valores posibles incluyen `Mqtt`, `Mqtt_WebSocket_Only`, `Mqtt_Tcp_Only`, `Amqp`, `Amqp_WebSocket_Only`, `Amqp_Tcp_only` y `Http1`.|
     
4. En la carpeta *SymmetricKeySample*, abra el archivo *ProvisioningDeviceClientSample.cs* en un editor de texto. Este archivo muestra cómo se usa la clase [SecurityProviderSymmetricKey](/dotnet/api/microsoft.azure.devices.shared.securityprovidersymmetrickey?view=azure-dotnet&preserve-view=true) junto con la clase [ProvisioningDeviceClient](/dotnet/api/microsoft.azure.devices.provisioning.client.provisioningdeviceclient?view=azure-dotnet&preserve-view=true) para aprovisionar el dispositivo de clave simétrica. Revise el código de este archivo.  No es necesario realizar ningún cambio.
 
5. Compile y ejecute el código de ejemplo con el siguiente comando después de reemplazar los tres parámetros de ejemplo. Use los valores correctos para el ámbito del identificador, el identificador del registro de inscripción y la clave principal de inscripción.
    
    ```console
    dotnet run --s 0ne00000A0A --i symm-key-csharp-device-01 --p sbDDeEzRuEuGKag+kQKV+T1QGakRtHpsERLP0yPjwR93TrpEgEh/Y07CXstfha6dhIPWvdD1nRxK5T0KGKA+nQ==
    ```    


6. La salida esperada debe ser similar a la siguiente salida, que muestra el centro de IoT vinculado al que se asignó el dispositivo en función de la configuración de inscripción individual. Se envía una cadena "TestMessage" de ejemplo al centro como prueba:

    ```output
    D:\azure-iot-samples-csharp\provisioning\Samples\device\SymmetricKeySample>dotnet run --s 0ne00000A0A --i symm-key-csharp-device-01 --p sbDDeEzRuEuGKag+kQKV+T1QGakRtHpsERLP0yPjwR93TrpEgEh/Y07CXstfha6dhIPWvdD1nRxK5T0KGKA+nQ==

    Initializing the device provisioning client...
    Initialized for registration Id symm-key-csharp-device-01.
    Registering with the device provisioning service...
    Registration status: Assigned.
    Device csharp-device-01 registered to ExampleIoTHub.azure-devices.net.
    Creating symmetric key authentication for IoT Hub...
    Testing the provisioned device with IoT Hub...
    Sending a telemetry message...
    Finished.
    Enter any key to exit.
    ```
    
7. En Azure Portal, vaya al centro de IoT vinculado a su servicio de aprovisionamiento y abra la hoja **Dispositivos de IoT**. Después de aprovisionar correctamente el dispositivo de clave simétrica en el centro de conectividad, el identificador del dispositivo se muestra con el valor de *ESTADO* **habilitado**. Es posible que deba presionar el botón **Actualizar** en la parte superior si tenía abierta la hoja antes de ejecutar el código de ejemplo del dispositivo. 

    ![El dispositivo se registra con el centro de IoT](./media/quick-create-device-symmetric-key-csharp/hub-registration-csharp.png) 

> [!NOTE]
> Si ha cambiado el valor predeterminado de *Estado inicial del dispositivo gemelo* en la entrada de inscripción para el dispositivo, el dispositivo puede extraer el estado gemelo deseado desde el centro y actuar en consecuencia. Para más información, consulte [Información y uso de dispositivos gemelos en IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md).
>


## <a name="clean-up-resources"></a>Limpieza de recursos

Si planea seguir trabajando con el ejemplo de cliente de dispositivo y explorándolo, no limpie los recursos que se crean en este inicio rápido. Si no va a continuar, use el siguiente comando para eliminar todos los recursos que se han creado en este inicio rápido.

1. En el menú de la izquierda de Azure Portal, seleccione **Todos los recursos** y seleccione Device Provisioning Service. Abra **Administrar inscripciones** en servicio y, después, seleccione la pestaña **Inscripciones individuales**. Active la casilla situada junto al campo *ID. DE REGISTRO* del dispositivo que ha inscrito en este inicio rápido y presione el botón **Eliminar** situado en la parte superior del panel. 
1. En el menú de la izquierda de Azure Portal, seleccione **Todos los recursos** y después su centro de IoT. Abra **Dispositivos IoT** en su centro, active la casilla que hay junto al campo *ID DE DISPOSITIVO* del dispositivo que registró en este inicio rápido y, luego, presione el botón **Eliminar** situado en la parte superior del panel.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprovisionado un dispositivo de clave simétrica basado en Windows en el centro de IoT mediante IoT Hub Device Provisioning Service. Para aprender a aprovisionar dispositivos con certificado X.509 mediante C#, continúe con el inicio rápido siguiente para dispositivos X.509. 

> [!div class="nextstepaction"]
> [Plantillas de inicio rápido de Azure: Aprovisionamiento de dispositivos X.509 mediante DPS y C#](quick-create-simulated-device-x509-csharp.md)
