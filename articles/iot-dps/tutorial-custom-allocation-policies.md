---
title: Tutorial de uso de directivas de asignación personalizadas con Azure IoT Hub Device Provisioning Service (DPS)
description: Tutorial de uso de directivas de asignación personalizadas con Azure IoT Hub Device Provisioning Service (DPS)
author: wesmc7777
ms.author: wesmc
ms.date: 09/23/2020
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.custom: mvc
ms.openlocfilehash: f19f43b89cd2527a67827d7434f2e054ee40001e
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107227388"
---
# <a name="tutorial-use-custom-allocation-policies-with-device-provisioning-service-dps"></a>Tutorial: Uso de directivas de asignación personalizadas con Device Provisioning Service (DPS)

Una directiva de asignación personalizada le ofrece más control sobre cómo se asignan los dispositivos a un centro de IoT. Esto se logra mediante el uso de código personalizado en una [función de Azure](../azure-functions/functions-overview.md) que se ejecuta durante el aprovisionamiento para asignar dispositivos a un centro de IoT. El servicio de aprovisionamiento de dispositivos llama al código de función de Azure y proporciona toda la información pertinente sobre el dispositivo y la inscripción. El código de la función se ejecuta y devuelve la información del centro de IoT para aprovisionar el dispositivo.

Mediante directivas de asignación personalizadas, puede definir sus propias directivas de asignación cuando las directivas proporcionadas por Device Provisioning Service no cumplan los requisitos de su escenario.

Por ejemplo, quizás le interese examinar el certificado que usa un dispositivo durante el aprovisionamiento y asignar el dispositivo a un centro de IoT según una propiedad del certificado. También, es posible que tenga la información almacenada en una base de datos para los dispositivos y necesite consultarla para determinar a qué centro de IoT se debe asignar el dispositivo.

En este artículo se muestra un grupo de inscripción con una directiva de asignación personalizada que usa una función de Azure escrita en C# para aprovisionar tostadoras con claves simétricas. Los dispositivos no reconocidos como tostadora no se aprovisionarán en un centro de IoT.

Estos dispositivos solicitarán el aprovisionamiento mediante un código de ejemplo de aprovisionamiento incluido en el [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c).


En este tutorial, llevará a cabo lo siguiente:

> [!div class="checklist"]
> * Creará una nueva instancia de Azure Function App para respaldar la función de asignación personalizada
> * Creará una nueva inscripción de grupo mediante una función de Azure para la directiva de asignación personalizada
> * Creará claves de dispositivo para dos dispositivos
> * Configurará el entorno de desarrollo para el código del dispositivo de ejemplo del [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c)
> * Ejecutará los dispositivos y comprobará que están aprovisionados según la directiva de asignación personalizada


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

* En este artículo se asume que ha completado los pasos descritos en [Configuración del servicio Azure IoT Hub Device Provisioning con Azure Portal](./quick-setup-auto-provision.md) para crear las instancias de IoT Hub y DPS.

* Tener instalada la versión más reciente de [Git](https://git-scm.com/download/).

* En el caso de un entorno de desarrollo de Windows, se requiere [Visual Studio](https://visualstudio.microsoft.com/vs/) 2019 con la carga de trabajo [Desarrollo para el escritorio con C++](/cpp/ide/using-the-visual-studio-ide-for-cpp-desktop-development) habilitada. También se admiten Visual Studio 2015 y Visual Studio 2017.

* En el caso de Linux o macOS, consulte la sección correspondiente en [Preparación del entorno de desarrollo](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) en la documentación del [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c).


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-custom-allocation-function"></a>Creación de la función de asignación personalizada

En esta sección, creará una función de Azure que implementa la directiva de asignación personalizada. Esta función decide si un dispositivo se debe registrar en la instancia de IoT Hub en función de si su identificador de registro contiene la cadena **contoso-toaster**.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). En la página principal, seleccione **+ Crear un recurso**.

2. En el cuadro de búsqueda *Buscar en Marketplace*, escriba "Function App". En la lista desplegable, seleccione **Aplicación de funciones** y, luego, seleccione **Crear**.

3. En la página de creación de la **aplicación de funciones**, en la pestaña **Datos básicos**, escriba los siguientes valores para la nueva aplicación de función y seleccione **Revisar y crear**:

    **Suscripción**: si tiene varias suscripciones y la suscripción deseada no está seleccionada, elija la que desee usar.

    **Grupo de recursos**: este campo le permite crear un nuevo grupo de recursos o elegir uno existente para que contenga la aplicación de función. Elija el mismo grupo de recursos que contenga el centro de IoT que creó anteriormente para las pruebas, por ejemplo, **TestResources**. Al colocar juntos todos los recursos relacionados en un grupo, puede administrarlos juntos también.

    **Nombre de la aplicación de funciones**: escriba un nombre único de aplicación de función. En este ejemplo se usa **contoso-function-app**.

    **Publicar**: compruebe que se ha seleccionado **Código**.

    **Pila del entorno en tiempo de ejecución**: seleccione **.NET Core** en la lista desplegable.

    **Región**: seleccione la misma región que la del grupo de recursos. En este ejemplo se usa **West US**.

    > [!NOTE]
    > De forma predeterminada, está habilitado Application Insights. Aunque en este artículo Application Insights no es necesario, puede ayudarle a entender y a investigar los problemas que encuentre con la asignación personalizada. Si lo prefiere, puede deshabilitar Application Insights; para ello, seleccione la pestaña **Supervisión** y, luego, seleccione **No** en **Habilitar Application Insights**.

    ![Creación de una aplicación de funciones de Azure para hospedar la función de asignación personalizada](./media/tutorial-custom-allocation-policies/create-function-app.png)

4. En la página **Resumen**, seleccione **Crear** para crear la aplicación de funciones. La implementación puede tardar varios minutos. Cuando finalice, seleccione **Ir al recurso**.

5. En el panel izquierdo debajo de **Funciones**, haga clic en **Funciones** y, a continuación, en **+ Agregar** para agregar una nueva función.

6. En la página de plantillas, seleccione el icono **Desencadenador HTTP** y, después, seleccione **Crear función**. Se creará una función denominada **HttpTrigger1** y el portal mostrará la página de información general de la función.

7. Haga clic en **Código y prueba** para la nueva función. El portal mostrará el contenido del archivo de código **run.csx**. 

8. Reemplace el código de la función **HttpTrigger1** por el código siguiente y seleccione **Guardar**. El código de la asignación personalizada está listo para usarse.

    ```csharp
    #r "Newtonsoft.Json"

    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;

    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        // Get request body
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);

        log.LogInformation("Request.Body:...");
        log.LogInformation(requestBody);

        // Get registration ID of the device
        string regId = data?.deviceRuntimeContext?.registrationId;

        string message = "Uncaught error";
        bool fail = false;
        ResponseObj obj = new ResponseObj();

        if (regId == null)
        {
            message = "Registration ID not provided for the device.";
            log.LogInformation("Registration ID : NULL");
            fail = true;
        }
        else
        {
            string[] hubs = data?.linkedHubs.ToObject<string[]>();

            // Must have hubs selected on the enrollment
            if (hubs == null)
            {
                message = "No hub group defined for the enrollment.";
                log.LogInformation("linkedHubs : NULL");
                fail = true;
            }
            else
            {
                // This is a Contoso Toaster 
                if (regId.Contains("contoso-toaster"))
                {
                    //Log IoT hubs configured for the enrollment
                    foreach(string hubString in hubs)
                    {
                        log.LogInformation("linkedHub : " + hubString);
                    }

                    obj.iotHubHostName = hubs[0];
                    log.LogInformation("Selected hub : " + obj.iotHubHostName);
                }
                else
                {
                    fail = true;
                    message = "Unrecognized device registration.";
                    log.LogInformation("Unknown device registration");
                }
            }
        }

        log.LogInformation("\nResponse");
        log.LogInformation((obj.iotHubHostName != null) ? JsonConvert.SerializeObject(obj) : message);

        return (fail)
            ? new BadRequestObjectResult(message) 
            : (ActionResult)new OkObjectResult(obj);
    }

    public class ResponseObj
    {
        public string iotHubHostName {get; set;}
    }
    ```

9. Justo debajo de la parte inferior del archivo de código **run.csx**, haga clic en **Registros** para supervisar el registro de la función de asignación personalizada. 


## <a name="create-the-enrollment"></a>Creación de la inscripción

En esta sección, creará un grupo de inscripciones que use la directiva de asignación personalizada. Para que sea más sencillo, en este artículo se usa la [atestación de clave simétrica](concepts-symmetric-key-attestation.md) con la inscripción. Con el fin de obtener una solución más segura, considere la posibilidad de usar la [atestación de certificado X.509](concepts-x509-attestation.md) con una cadena de confianza.

1. Aún en [Azure Portal](https://portal.azure.com), abra el servicio de aprovisionamiento.

2. Seleccione **Administrar inscripciones** en el panel izquierdo y, luego, seleccione el botón **Agregar grupo de inscripciones** en la parte superior de la página.

3. En **Agregar grupo de inscripciones**, escriba la información de la tabla siguiente y haga clic en el botón **Guardar**.

    | Campo | Descripción o valor sugerido |
    | :---- | :----------------------------- |
    | **Nombre de grupo** | Escriba **contoso-custom-allocated-devices** |
    | **Tipo de atestación** | Seleccione **Clave simétrica** |
    | **Generar claves automáticamente** | ya debe estar activada esta casilla. |
    | **Seleccione cómo desea asignar los dispositivos a los centros** | Seleccione **Personalizado (use la función de Azure)** |
    | **Seleccione los centros de IoT a los que se puede asignar este grupo** | Seleccione la instancia de IoT Hub que creó anteriormente al completar el inicio rápido. |
    | **Seleccionar la función de Azure** | Seleccione la suscripción que contiene la aplicación de funciones que ha creado. A continuación, seleccione **contoso-function-app** y **HttpTrigger1** para la función. |

    ![Adición de un grupo de inscripciones de asignación personalizado para la atestación de clave simétrica](./media/tutorial-custom-allocation-policies/create-custom-allocation-enrollment.png)

4. Después de guardar la inscripción, vuelva a abrirla y apunte la **clave principal**. Primero debe guardar la inscripción para que se generen las claves. Esta clave simétrica principal se usará para generar claves de dispositivo únicas para dispositivos que intenten aprovisionarse más adelante. 

## <a name="derive-unique-device-keys"></a>Derivación de las claves de dispositivo únicas

Los dispositivos no usan la clave simétrica principal directamente. En su lugar, use la clave principal para obtener una clave de dispositivo para cada dispositivo. En esta sección, creará dos claves de dispositivo únicas. Una de las claves se usará para la tostadora simulada. La otra, se usará para la bomba de calor simulada. Las claves generadas permitirán a ambos dispositivos intentar registrarse. Solo uno de los identificadores de registro de dispositivo tendrá un sufijo válido que el código de ejemplo de la directiva de asignación personalizada acepte. Como resultado, se aceptará un identificador y se rechazará el otro.

Para obtener la clave de dispositivo, usará la clave simétrica que apuntó anteriormente para calcular el [HMAC-SHA256](https://wikipedia.org/wiki/HMAC) del identificador de registro de cada dispositivo y convertirá el resultado en formato Base64. Para obtener más información sobre cómo crear claves de dispositivo derivadas con grupos de inscripción, consulte la sección de inscripciones de grupo de [Atestación de clave simétrica](concepts-symmetric-key-attestation.md).

Para el ejemplo de este artículo, use los dos identificadores de registro de dispositivo que se muestran a continuación con el código siguiente para calcular una clave de dispositivo para ambos:

* **contoso-toaster-007**
* **contoso-heatpump-088**

Reemplace el valor de la variable **KEY** por la **clave principal** que anotó antes de que se creara el grupo de inscripción. El valor de clave y la salida que se muestran con el código siguiente son solo un ejemplo.

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

Si utiliza una estación de trabajo basada en Windows, puede usar PowerShell para generar las claves de dispositivo derivadas tal y como se muestra en el ejemplo siguiente.

```powershell
$KEY='oiK77Oy7rBw8YB6IS6ukRChAw+Yq6GC61RMrPLSTiOOtdI+XDu0LmLuNm11p+qv2I+adqGUdZHm46zXAQdZoOA=='

$REG_ID1='contoso-toaster-007'
$REG_ID2='contoso-heatpump-088'

$hmacsha256 = New-Object System.Security.Cryptography.HMACSHA256
$hmacsha256.key = [Convert]::FromBase64String($key)
$sig1 = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID1))
$sig2 = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID2))
$derivedkey1 = [Convert]::ToBase64String($sig1)
$derivedkey2 = [Convert]::ToBase64String($sig2)

echo "`n`n$REG_ID1 : $derivedkey1`n$REG_ID2 : $derivedkey2`n`n"
```

```powershell
contoso-toaster-007 : JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=
contoso-heatpump-088 : 6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=
```

#### <a name="bash"></a>[Bash](#tab/bash)

Si usa una estación de trabajo de Linux, puede utilizar openssl para generar las claves de dispositivo derivadas tal y como se muestra en el ejemplo de Bash siguiente.

```bash
KEY=oiK77Oy7rBw8YB6IS6ukRChAw+Yq6GC61RMrPLSTiOOtdI+XDu0LmLuNm11p+qv2I+adqGUdZHm46zXAQdZoOA==

REG_ID1=contoso-toaster-007
REG_ID2=contoso-heatpump-088

keybytes=$(echo $KEY | base64 --decode | xxd -p -u -c 1000)
devkey1=$(echo -n $REG_ID1 | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64)
devkey2=$(echo -n $REG_ID2 | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64)

echo -e $"\n\n$REG_ID1 : $devkey1\n$REG_ID2 : $devkey2\n\n"
```

```bash
contoso-toaster-007 : JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=
contoso-heatpump-088 : 6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=
```

---


## <a name="prepare-an-azure-iot-c-sdk-development-environment"></a>Preparación de un entorno de desarrollo del SDK de Azure IoT para C

Estos dispositivos solicitarán el aprovisionamiento mediante un código de ejemplo de aprovisionamiento incluido en el [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c).

En esta sección, preparará un entorno de desarrollo para compilar el [SDK de Azure IoT](https://github.com/Azure/azure-iot-sdk-c). El SDK incluye el código de ejemplo para el dispositivo simulado. Este dispositivo simulado tratará de aprovisionarse durante la secuencia de arranque del dispositivo.

En esta sección, nos enfocamos en una estación de trabajo basada en Windows. Para obtener un ejemplo de Linux, consulte la configuración de máquinas virtuales en [How to provision for multitenancy](how-to-provision-multitenant.md) (Cómo aprovisionar para varios inquilinos).

1. Descargue el [sistema de compilación CMake](https://cmake.org/download/).

    **Antes** de comenzar la instalación de `CMake`, es importante que los requisitos previos de Visual Studio (Visual Studio y la carga de trabajo de desarrollo de escritorio con C++) estén instalados en la máquina. Una vez que los requisitos previos están en su lugar, y se ha comprobado la descarga, instale el sistema de compilación de CMake.

2. Busque el nombre de etiqueta de la [versión más reciente](https://github.com/Azure/azure-iot-sdk-c/releases/latest) del SDK.

3. Abra un símbolo del sistema o el shell de Bash de Git. Ejecute los siguientes comandos para clonar la versión más reciente del repositorio de GitHub del [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c). Use la etiqueta que encontró en el paso anterior como valor del parámetro `-b`:

    ```cmd/sh
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Esta operación puede tardar varios minutos en completarse.

4. Cree un subdirectorio `cmake` en el directorio raíz del repositorio de Git y vaya a esa carpeta. Ejecute los siguientes comandos desde el directorio `azure-iot-sdk-c`:

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

5. Ejecute el siguiente comando para compilar una versión del SDK específica para su plataforma de cliente de desarrollo. Se generará una solución de Visual Studio para el dispositivo simulado en el directorio `cmake`. 

    ```cmd
    cmake -Dhsm_type_symm_key:BOOL=ON -Duse_prov_client:BOOL=ON  ..
    ```

    Si `cmake` no puede encontrar el compilador de C++, es posible que obtenga errores de compilación durante la ejecución del comando. Si eso sucede, pruebe a ejecutar este comando en el [símbolo del sistema de Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs).

    Una vez realizada la compilación, las últimas líneas de salida tendrán un aspecto similar al siguiente:

    ```cmd/sh
    $ cmake -Dhsm_type_symm_key:BOOL=ON -Duse_prov_client:BOOL=ON  ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```

## <a name="simulate-the-devices"></a>Simulación de los dispositivos

En esta sección, actualizará el ejemplo de aprovisionamiento denominado **prov\_dev\_client\_sample** que está ubicado en el SDK de Azure IoT para C que ha configurado anteriormente.

El código de ejemplo simula una secuencia de arranque de dispositivo que envía la solicitud de aprovisionamiento a la instancia de Device Provisioning Service. La secuencia de arranque hará que se reconozca y se asigne la tostadora al centro de IoT según la directiva de asignación personalizada.

1. En Azure Portal, seleccione la pestaña **Información general** para su servicio Device Provisioning y anote el valor de **_Ámbito de id_**.

    ![Extracción de información del punto de conexión del servicio Device Provisioning desde la hoja del portal](./media/quick-create-simulated-device-x509/extract-dps-endpoints.png) 

2. En Visual Studio, abra el archivo de solución **azure_iot_sdks.sln** que se ha generado anteriormente al ejecutar CMake. El archivo de solución debe estar en la siguiente ubicación:

    ```
    azure-iot-sdk-c\cmake\azure_iot_sdks.sln
    ```

3. En la ventana del *Explorador de soluciones* de Visual Studio, desplácese hasta la carpeta **Aprovisionar\_Ejemplos**. Expanda el proyecto de ejemplo denominado **prov\_dev\_client\_sample**. Expanda **Archivos de código fuente** y abra **prov\_dev\_cliente\_sample.c**.

4. Busque la constante `id_scope` y reemplace el valor por el valor de **Ámbito de id.** que copió anteriormente. 

    ```c
    static const char* id_scope = "0ne00002193";
    ```

5. Busque la definición de la función `main()` en el mismo archivo. Asegúrese de que la variable `hsm_type` está establecida en `SECURE_DEVICE_TYPE_SYMMETRIC_KEY`, tal como se muestra a continuación:

    ```c
    SECURE_DEVICE_TYPE hsm_type;
    //hsm_type = SECURE_DEVICE_TYPE_TPM;
    //hsm_type = SECURE_DEVICE_TYPE_X509;
    hsm_type = SECURE_DEVICE_TYPE_SYMMETRIC_KEY;
    ```

6. En la función `main()`, busque la llamada a `Prov_Device_Register_Device()`. Justo antes de esa llamada, agregue las siguientes líneas de código que usan [`Prov_Device_Set_Provisioning_Payload()`](/azure/iot-hub/iot-c-sdk-ref/prov-device-client-h/prov-device-set-provisioning-payload) para pasar una carga de JSON personalizada durante el aprovisionamiento. Esto permite proporcionar más información a las funciones de asignación personalizadas. También se puede usar para pasar el tipo de dispositivo en lugar de examinar el identificador de registro. Para más información sobre el envío y la recepción de cargas de datos personalizadas con DPS, consulte [Transferencia de una carga útil entre un dispositivo y DPS](how-to-send-additional-data.md).

    ```c
    // An example custom payload
    const char* custom_json_payload = "{\"MyDeviceFirmwareVersion\":\"12.0.2.5\",\"MyDeviceProvisioningVersion\":\"1.0.0.0\"}";

    prov_device_result = Prov_Device_Set_Provisioning_Payload(prov_device_handle, custom_json_payload);
    if (prov_device_result != PROV_DEVICE_RESULT_OK)
    {
        (void)printf("\r\nFailure setting provisioning payload: %s\r\n", MU_ENUM_TO_STRING(PROV_DEVICE_RESULT, prov_device_result));
    }
    ```

7. Haga clic con el botón derecho en el proyecto **prov\_dev\_client\_sample** y seleccione **Set as Startup Project** (Establecer como proyecto de inicio).

### <a name="simulate-the-contoso-toaster-device"></a>Simulación de la tostadora de Contoso

1. Para simular el dispositivo tostadora, encuentre la llamada a `prov_dev_set_symmetric_key_info()` en **prov\_dev\_client\_sample.c** que se marcó como comentario.

    ```c
    // Set the symmetric key if using they auth type
    //prov_dev_set_symmetric_key_info("<symm_registration_id>", "<symmetric_Key>");
    ```

    Quite la marca del comentario de la llamada de función y reemplace los valores de marcador de posición (incluidos los corchetes angulares) por el identificador de registro de la tostadora y la clave de dispositivo derivada que ha generado antes. Solo se proporciona el valor de clave **JC8F96eayuQwwz + PkE7IzjH2lIAjCUnAa61tDigBnSs =** como ejemplo.

    ```c
    // Set the symmetric key if using they auth type
    prov_dev_set_symmetric_key_info("contoso-toaster-007", "JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=");
    ```

    Guarde el archivo.

2. En el menú de Visual Studio, seleccione **Depurar** > **Iniciar sin depurar** para ejecutar la solución. En la solicitud para volver a compilar el proyecto, seleccione **Sí** para recompilar el proyecto antes de ejecutarlo.

    El siguiente texto es un ejemplo de la salida del registro del código de la función de asignación personalizada que se ejecuta para la tostadora. Observe que hay un centro correctamente seleccionado para una tostadora. Observe también el miembro `payload` que contiene el contenido JSON personalizado que agregó al código. Este miembro está disponible para su código para usarlo en `deviceRuntimeContext`.

    Este registro está disponible al hacer clic en **Registros** en el código de la función en el portal:

    ```cmd
    2020-09-23T11:44:37.505 [Information] Executing 'Functions.HttpTrigger1' (Reason='This function was programmatically called via the host APIs.', Id=4596d45e-086f-4e86-929b-4a02814eee40)
    2020-09-23T11:44:41.380 [Information] C# HTTP trigger function processed a request.
    2020-09-23T11:44:41.381 [Information] Request.Body:...
    2020-09-23T11:44:41.381 [Information] {"enrollmentGroup":{"enrollmentGroupId":"contoso-custom-allocated-devices","attestation":{"type":"symmetricKey"},"capabilities":{"iotEdge":false},"etag":"\"e8002126-0000-0100-0000-5f6b2a570000\"","provisioningStatus":"enabled","reprovisionPolicy":{"updateHubAssignment":true,"migrateDeviceData":true},"createdDateTimeUtc":"2020-09-23T10:58:31.62286Z","lastUpdatedDateTimeUtc":"2020-09-23T10:58:31.62286Z","allocationPolicy":"custom","iotHubs":["contoso-toasters-hub-1098.azure-devices.net"],"customAllocationDefinition":{"webhookUrl":"https://contoso-function-app.azurewebsites.net/api/HttpTrigger1?****","apiVersion":"2019-03-31"}},"deviceRuntimeContext":{"registrationId":"contoso-toaster-007","symmetricKey":{},"payload":{"MyDeviceFirmwareVersion":"12.0.2.5","MyDeviceProvisioningVersion":"1.0.0.0"}},"linkedHubs":["contoso-toasters-hub-1098.azure-devices.net"]}
    2020-09-23T11:44:41.687 [Information] linkedHub : contoso-toasters-hub-1098.azure-devices.net
    2020-09-23T11:44:41.688 [Information] Selected hub : contoso-toasters-hub-1098.azure-devices.net
    2020-09-23T11:44:41.688 [Information] Response
    2020-09-23T11:44:41.688 [Information] {"iotHubHostName":"contoso-toasters-hub-1098.azure-devices.net"}
    2020-09-23T11:44:41.689 [Information] Executed 'Functions.HttpTrigger1' (Succeeded, Id=4596d45e-086f-4e86-929b-4a02814eee40, Duration=4347ms)    
    ```

    El ejemplo de salida del dispositivo siguiente muestra el arranque correcto de la tostadora simulada y su conexión a la instancia del servicio de aprovisionamiento para asignarse al centro de IoT de tostadoras según la directiva de asignación personalizada:

    ```cmd
    Provisioning API Version: 1.3.6

    Registering Device

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service: contoso-toasters-hub-1098.azure-devices.net, deviceId: contoso-toaster-007

    Press enter key to exit:
    ```

### <a name="simulate-the-contoso-heat-pump-device"></a>Simulación de la bomba de calor de Contoso

1. Para simular el dispositivo de la bomba de calor, actualice la llamada a `prov_dev_set_symmetric_key_info()` en **prov\_dev\_client\_sample.c** de nuevo con el identificador de registro de la bomba de calor y la clave de dispositivo derivada que ha generado antes. Solo se proporciona el valor de la clave **6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg =** como ejemplo.

    ```c
    // Set the symmetric key if using they auth type
    prov_dev_set_symmetric_key_info("contoso-heatpump-088", "6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=");
    ```

    Guarde el archivo.

2. En el menú de Visual Studio, seleccione **Depurar** > **Iniciar sin depurar** para ejecutar la solución. En la solicitud para volver a compilar el proyecto, haga clic en **Sí** para recompilar el proyecto antes de ejecutarlo.

    El siguiente texto es un ejemplo de la salida del registro del código de la función de asignación personalizada que se ejecuta para la bomba de calor. La directiva de asignación personalizada rechaza este registro con un error HTTP 400 Solicitud incorrecta. Observe el miembro `payload` que contiene el contenido JSON personalizado que agregó al código. Este miembro está disponible para su código para usarlo en `deviceRuntimeContext`.

    Este registro está disponible al hacer clic en **Registros** en el código de la función en el portal:

    ```cmd
    2020-09-23T11:50:23.652 [Information] Executing 'Functions.HttpTrigger1' (Reason='This function was programmatically called via the host APIs.', Id=2fa77f10-42f8-43fe-88d9-a8c01d4d3f68)
    2020-09-23T11:50:23.653 [Information] C# HTTP trigger function processed a request.
    2020-09-23T11:50:23.654 [Information] Request.Body:...
    2020-09-23T11:50:23.654 [Information] {"enrollmentGroup":{"enrollmentGroupId":"contoso-custom-allocated-devices","attestation":{"type":"symmetricKey"},"capabilities":{"iotEdge":false},"etag":"\"e8002126-0000-0100-0000-5f6b2a570000\"","provisioningStatus":"enabled","reprovisionPolicy":{"updateHubAssignment":true,"migrateDeviceData":true},"createdDateTimeUtc":"2020-09-23T10:58:31.62286Z","lastUpdatedDateTimeUtc":"2020-09-23T10:58:31.62286Z","allocationPolicy":"custom","iotHubs":["contoso-toasters-hub-1098.azure-devices.net"],"customAllocationDefinition":{"webhookUrl":"https://contoso-function-app.azurewebsites.net/api/HttpTrigger1?****","apiVersion":"2019-03-31"}},"deviceRuntimeContext":{"registrationId":"contoso-heatpump-088","symmetricKey":{},"payload":{"MyDeviceFirmwareVersion":"12.0.2.5","MyDeviceProvisioningVersion":"1.0.0.0"}},"linkedHubs":["contoso-toasters-hub-1098.azure-devices.net"]}
    2020-09-23T11:50:23.654 [Information] Unknown device registration
    2020-09-23T11:50:23.654 [Information] Response
    2020-09-23T11:50:23.654 [Information] Unrecognized device registration.
    2020-09-23T11:50:23.655 [Information] Executed 'Functions.HttpTrigger1' (Succeeded, Id=2fa77f10-42f8-43fe-88d9-a8c01d4d3f68, Duration=11ms)
    ```

    El ejemplo de salida del dispositivo siguiente muestra el arranque de la bomba de calor simulada y su conexión a la instancia del servicio de aprovisionamiento para intentar registrarse en un centro de IoT mediante la directiva de asignación personalizada. Esto produce un error (`Custom allocation failed with status code: 400`), ya que la directiva de asignación personalizada se diseñó para permitir solo tostadoras:


    ```cmd
    Provisioning API Version: 1.3.7
    
    Registering Device
    
    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Error: Time:Wed Sep 23 13:06:01 2020 File:d:\testing\azure-iot-sdk-c\provisioning_client\src\prov_device_ll_client.c Func:_prov_transport_process_json_reply Line:658 Provisioning Failure: OperationId: 4.eb89f3e8407a3711.2525bd34-02e9-4e91-a9c0-4dbc4ad5de66 - Date: 2020-09-23T17:05:58.2363145Z - Msg: Custom allocation failed with status code: 400
    Error: Time:Wed Sep 23 13:06:01 2020 File:d:\testing\azure-iot-sdk-c\provisioning_client\src\prov_transport_mqtt_common.c Func:_prov_transport_common_mqtt_dowork Line:1014 Unable to process registration reply.
    Error: Time:Wed Sep 23 13:06:01 2020 File:d:\testing\azure-iot-sdk-c\provisioning_client\src\prov_device_ll_client.c Func:_on_transport_registration_data Line:770 Failure retrieving data from the provisioning service
    
    Failure registering device: PROV_DEVICE_RESULT_DEV_AUTH_ERROR
    Press enter key to exit:    
    ```
    
## <a name="clean-up-resources"></a>Limpieza de recursos

Si tiene previsto seguir trabajando con los recursos creados en este artículo, puede dejarlos. Si no, siga estos pasos para eliminar todos los recursos creados en este artículo para evitar cargos innecesarios.

En estos pasos se supone que ha creado todos los recursos de este artículo como se ha indicado en el mismo grupo de recursos denominado **contoso-us-resource-group**.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible. El grupo de recursos y todos los recursos contenidos en él se eliminan permanentemente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados. Si ha creado una instancia de IoT Hub en un grupo de recursos ya existente que contiene recursos que desea conservar, puede eliminar solo esa instancia en lugar de eliminar todo el grupo de recursos.
>

Para eliminar el grupo de recursos por nombre:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y después seleccione **Grupos de recursos**.

2. En el cuadro de texto **Filtrar por nombre...**, escriba el nombre del grupo de recursos que contiene los recursos: **contoso-us-resource-group**. 

3. A la derecha del grupo de recursos de la lista de resultados, seleccione **...** y, a continuación, **Eliminar grupo de recursos**.

4. Se le pedirá que confirme la eliminación del grupo de recursos. Escriba de nuevo el nombre del grupo de recursos para confirmar y, después, seleccione **Eliminar**. Transcurridos unos instantes, el grupo de recursos y todos los recursos que contiene se eliminan.

## <a name="next-steps"></a>Pasos siguientes

Para ver un ejemplo más detallado de la directiva de asignación personalizada, consulte 

> [!div class="nextstepaction"]
> [Uso de directivas de asignación personalizadas](how-to-use-custom-allocation-policies.md)

* Para más información sobre el reaprovisionamiento, consulte

> [!div class="nextstepaction"]
> [Conceptos sobre el reaprovisionamiento de dispositivos de IoT Hub](concepts-device-reprovision.md)

* Para más información sobre el desaprovisionamiento, consulte

> [!div class="nextstepaction"]
> [Desaprovisionamiento de dispositivos aprovisionados automáticamente](how-to-unprovision-devices.md)