---
title: 'Tutorial: Desarrollo de módulos con C para Windows mediante Azure IoT Edge'
description: En este tutorial se muestra cómo crear módulos de IoT Edge con código de C e implementarlos en dispositivos Windows que ejecutan IoT Edge.
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 05/28/2019
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
monikerRange: =iotedge-2018-06
ms.openlocfilehash: a67bf92dbee9bf3d444add3be32544e4c3eba4f3
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106221609"
---
# <a name="tutorial-develop-c-iot-edge-modules-using-windows-containers"></a>Tutorial: Desarrollo de módulos de IoT Edge con C con contenedores de Windows

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

En este artículo se muestra cómo usar Visual Studio para desarrollar código de C e implementarlo en un dispositivo Windows que ejecuta Azure IoT Edge.

>[!NOTE]
>IoT Edge 1.1 LTS es el último canal de versión compatible con los contenedores de Windows. A partir de la versión 1.2, no se admiten los contenedores de Windows. Considere la posibilidad de usar o pasarse a [IoT Edge para Linux en Windows](iot-edge-for-linux-on-windows.md) para ejecutar IoT Edge en dispositivos Windows.

Los módulos de Azure IoT Edge se pueden usar para implementar código que implementa la lógica de negocios directamente en los dispositivos IoT Edge. En este tutorial, se detallan los pasos para crear e implementar un módulo de IoT Edge que filtra los datos de sensor.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Usar Visual Studio Code para crear un módulo de IoT Edge basado en el SDK de C.
> * Usar Visual Studio y Docker para crear una imagen de Docker y publicarla en el Registro.
> * Implementar el módulo en el dispositivo IoT Edge.
> * Ver datos generados.

El módulo IoT Edge que creó en este tutorial filtra lo datos sobre la temperatura generados por el dispositivo. El módulo envía mensajes ascendentes solo si la temperatura supera un umbral especificado. Este tipo de análisis perimetral resulta útil para reducir la cantidad de datos que se comunican a la nube y se almacenan en ella.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

En este tutorial se muestra cómo desarrollar un módulo en C mediante Visual Studio 2019 y, luego, implementarlo en un dispositivo Windows. Si va a desarrollar módulos para usar contenedores de Linux, vaya a [Desarrollo de un módulo IoT Edge con C con contenedores de Linux](tutorial-csharp-module.md) en su lugar.

Para conocer las opciones para desarrollar e implementar módulos de C con contenedores de Windows, consulte la siguiente tabla:

| C | Visual&nbsp;Studio&nbsp;Code | Visual Studio 2017&nbsp;y&nbsp;2019 |
| -- | ------------------ | :------------------: |
| Windows AMD64 |  | ![Desarrollo de módulos C para WinAMD64 en Visual Studio](./media/tutorial-c-module/green-check.png) |

Antes de comenzar este tutorial, configure el entorno de desarrollo siguiendo las instrucciones del tutorial [Desarrollo de módulos IoT Edge con contenedores de Windows](tutorial-develop-for-windows.md). Cuando haya terminado, el entorno contendrá los siguientes requisitos previos:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.
* Un [dispositivo Windows que ejecute Azure IoT Edge](how-to-install-iot-edge-windows-on-windows.md).
* Un registro de contenedor, como [Azure Container Registry](../container-registry/index.yml).
* [Visual Studio 2019](/visualstudio/install/install-visual-studio) configurado con la extensión [Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools).
* [Docker Desktop](https://docs.docker.com/docker-for-windows/install/) configurado para ejecutar contenedores de Windows.

Instale el SDK de C de Azure IoT para Windows x64 mediante vcpkg ejecutando los siguientes comandos:

   ```powershell
   git clone https://github.com/Microsoft/vcpkg
   cd vcpkg
   .\bootstrap-vcpkg.bat
   .\vcpkg install azure-iot-sdk-c:x64-windows
   .\vcpkg --triplet x64-windows integrate install
   ```

> [!TIP]
> Si usa Visual Studio 2017 (versión 15.7 o posterior), descargue e instale Azure IoT Edge Tools para Visual Studio 2017 desde [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools).

## <a name="create-a-module-project"></a>Creación de un proyecto de módulo

En esta sección, creará un proyecto de módulo de IoT Edge basado en el SDK de C mediante Visual Studio y la extensión Azure IoT Edge Tools. Después de crear una plantilla de proyecto, agrega código nuevo para que el módulo filtre los mensajes en función de sus propiedades notificadas.

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

Cree una plantilla de solución de C que pueda personalizar con su propio código.

1. Abra Visual Studio 2019 y seleccione **Crear nuevo proyecto**.

1. En el panel **Crear un nuevo proyecto**, busque **IoT Edge** y, luego, en la lista de resultados, seleccione el proyecto **Azure IoT Edge (Windows amd64)** .

   ![Captura de pantalla del panel "Crear un nuevo proyecto" de IoT Edge.](./media/tutorial-c-module-windows/new-project.png)

1. Seleccione **Next** (Siguiente).

    Se abre el panel **Configure su nuevo proyecto**.

   ![Captura de pantalla del panel "Configure su nuevo proyecto".](./media/tutorial-c-module-windows/configure-project.png)

1. En el panel **Configure su nuevo proyecto**, cambie el nombre del proyecto y la solución por algo más descriptivo, como **CTutorialApp**. 

1. Seleccione **Crear** para crear el proyecto.

   Se abre el panel **Agregar módulo**.

   ![Captura de pantalla del panel "Agregar módulo" para configurar el proyecto.](./media/tutorial-c-module-windows/add-application-and-module.png)

1. En la página **Configure su nuevo proyecto**, haga lo siguiente:

   a. En el panel izquierdo, seleccione la plantilla **C Module**.  
   b. En el cuadro **Nombre del módulo**, escriba **CModule**.  
   c. En el cuadro **URL de repositorio**, reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure, en el siguiente formato: `<registry name>.azurecr.io/cmodule`.

    > [!NOTE]
    > Un repositorio de imágenes incluye el nombre del registro de contenedor y el nombre de la imagen de contenedor. La imagen de contenedor se rellena previamente con el valor del nombre del proyecto del módulo.  Puede recuperar el servidor de inicio de sesión de la página de información general del registro de contenedor en Azure Portal.

1. Haga clic en **Aceptar** para crear el proyecto.

### <a name="add-your-registry-credentials"></a>Adición de las credenciales del Registro

El manifiesto de implementación comparte las credenciales del registro de contenedor con el entorno de ejecución de Azure IoT Edge. El entorno de ejecución necesita estas credenciales para extraer las imágenes privadas e insertarlas en el dispositivo IoT Edge. Use las credenciales de la sección **Claves de acceso** de Azure Container Registry.

1. En el Explorador de soluciones de Visual Studio, abra el archivo *deployment.template.json*.

1. Busque la propiedad **registryCredentials** en las propiedades $edgeAgent que desee. La dirección del registro de la propiedad se rellenará automáticamente con la información proporcionada al crear el proyecto. Los campos de nombre de usuario y contraseña deben contener nombres de variable. Por ejemplo:

   ```json
   "registryCredentials": {
     "<registry name>": {
       "username": "$CONTAINER_REGISTRY_USERNAME_<registry name>",
       "password": "$CONTAINER_REGISTRY_PASSWORD_<registry name>",
       "address": "<registry name>.azurecr.io"
     }
   }
   ```

1. Abra el archivo de entorno (ENV) en la solución del módulo. De forma predeterminada, el archivo está oculto en el Explorador de soluciones, por lo que es posible que tenga que seleccionar el botón **Mostrar todos los archivos** para verlo. El archivo ENV debe contener las mismas variables de nombre de usuario y contraseña que vio en el archivo *deployment.template.json*.

1. Agregue los valores de **Nombre de usuario** y **Contraseña** del registro de contenedor de Azure.

1. Guarde los cambios en el archivo ENV.

### <a name="update-the-module-with-custom-code"></a>Actualización del módulo con código personalizado

El código del módulo predeterminado recibe mensajes en una cola de entrada y los pasa por la cola de salida. Vamos a agregar código adicional para que el módulo procese los mensajes en el perímetro antes de reenviarlos al centro de IoT. Actualice el módulo para que analice los datos de temperatura de cada mensaje y solo envíe el mensaje al centro de IoT si la temperatura supera un umbral determinado.

1. En este escenario, los datos del sensor están en formato JSON. Para filtrar los mensajes en formato JSON, importe una biblioteca JSON para C. En este tutorial se usa Parson.

   a. Descargue el [repositorio de Parson GitHub](https://github.com/kgabis/parson).  
   b. Copie los archivos *parson.c* y *parson.h* en el proyecto CModule.  
   c. En Visual Studio, abra el archivo *CMakeLists.txt* desde la carpeta del proyecto CModule.  
   d. En la parte superior del archivo, importe los archivos de Parson como una biblioteca denominada **my_parson**.

      ```txt
      add_library(my_parson
          parson.c
          parson.h
      )
      ```  
   e. Agregue `my_parson` a la lista de bibliotecas de la sección "target_link_libraries" del archivo *CMakeLists.txt*.  
   f. Guarde el archivo *CMakeLists.txt*.  
   g. Seleccione **CModule** > **main.c**. Al final de la lista de instrucciones de inclusión, agregue una nueva para incluir `parson.h` a fin de obtener compatibilidad con JSON:

      ```c
      #include "parson.h"
      ```

1. En el archivo *main.c*, agregue una variable global denominada `temperatureThreshold` junto a la variable `messagesReceivedByInput1Queue`. Esta variable establece el valor que debe superar la temperatura medida para que los datos se envíen al centro de IoT.

    ```c
    static double temperatureThreshold = 25;
    ```

1. Busque la función `CreateMessageInstance` en *main.c*. Reemplace la instrucción interna *if-else* por el código siguiente, que agrega unas líneas de funcionalidad:

   ```c
   if ((messageInstance->messageHandle = IoTHubMessage_Clone(message)) == NULL)
   {
       free(messageInstance);
       messageInstance = NULL;
   }
   else
   {
       messageInstance->messageTrackingId = messagesReceivedByInput1Queue;
       MAP_HANDLE propMap = IoTHubMessage_Properties(messageInstance->messageHandle);
       if (Map_AddOrUpdate(propMap, "MessageType", "Alert") != MAP_OK)
       {
          printf("ERROR: Map_AddOrUpdate Failed!\r\n");
       }
   }
   ```

   Las nuevas líneas de código de la instrucción *else* agregan una nueva propiedad al mensaje, que lo etiqueta como una alerta. Este código etiqueta todos los mensajes como alertas, dado que se agregará una funcionalidad que envía mensajes al centro de IoT si notifican altas temperaturas.

1. Busque la función `InputQueue1Callback` y reemplace toda la función por el código siguiente. Esta función implementa el filtro de mensajería real. Cuando se recibe un mensaje, el filtro comprueba si la temperatura notificada supera el umbral. Si la temperatura supera el umbral, la función reenvía el mensaje a través de su cola de salida. Si no lo supera, la función omite el mensaje.

    ```c
    static unsigned char *bytearray_to_str(const unsigned char *buffer, size_t len)
    {
        unsigned char *ret = (unsigned char *)malloc(len + 1);
        memcpy(ret, buffer, len);
        ret[len] = '\0';
        return ret;
    }

    static IOTHUBMESSAGE_DISPOSITION_RESULT InputQueue1Callback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
    {
        IOTHUBMESSAGE_DISPOSITION_RESULT result;
        IOTHUB_CLIENT_RESULT clientResult;
        IOTHUB_MODULE_CLIENT_LL_HANDLE iotHubModuleClientHandle = (IOTHUB_MODULE_CLIENT_LL_HANDLE)userContextCallback;

        unsigned const char* messageBody;
        size_t contentSize;

        if (IoTHubMessage_GetByteArray(message, &messageBody, &contentSize) == IOTHUB_MESSAGE_OK)
        {
            messageBody = bytearray_to_str(messageBody, contentSize);
        } else
        {
            messageBody = "<null>";
        }

        printf("Received Message [%zu]\r\n Data: [%s]\r\n",
                messagesReceivedByInput1Queue, messageBody);

        // Check whether the message reports temperatures that exceed the threshold
        JSON_Value *root_value = json_parse_string(messageBody);
        JSON_Object *root_object = json_value_get_object(root_value);
        double temperature;

        // If temperature exceeds the threshold, send to output1
        if (json_object_dotget_value(root_object, "machine.temperature") != NULL && (temperature = json_object_dotget_number(root_object, "machine.temperature")) > temperatureThreshold)
        {
            printf("Machine temperature %f exceeds threshold %f\r\n", temperature, temperatureThreshold);
            // This message should be sent to next stop in the pipeline, namely "output1".  What happens at "outpu1" is determined
            // by the configuration of the Edge routing table setup.
            MESSAGE_INSTANCE *messageInstance = CreateMessageInstance(message);
            if (NULL == messageInstance)
            {
                result = IOTHUBMESSAGE_ABANDONED;
            }
            else
            {
                printf("Sending message (%zu) to the next stage in pipeline\n", messagesReceivedByInput1Queue);

                clientResult = IoTHubModuleClient_LL_SendEventToOutputAsync(iotHubModuleClientHandle, messageInstance->messageHandle, "output1", SendConfirmationCallback, (void *)messageInstance);
                if (clientResult != IOTHUB_CLIENT_OK)
                {
                    IoTHubMessage_Destroy(messageInstance->messageHandle);
                    free(messageInstance);
                    printf("IoTHubModuleClient_LL_SendEventToOutputAsync failed on sending msg#=%zu, err=%d\n", messagesReceivedByInput1Queue, clientResult);
                    result = IOTHUBMESSAGE_ABANDONED;
                }
                else
                {
                    result = IOTHUBMESSAGE_ACCEPTED;
                }
            }
        }
        // If message does not exceed the threshold, do not forward
        else
        {
            printf("Not sending message (%zu) to the next stage in pipeline.\r\n", messagesReceivedByInput1Queue);
            result = IOTHUBMESSAGE_ACCEPTED;
        }

        messagesReceivedByInput1Queue++;
        return result;
    }
    ```

1. Agregue una función `moduleTwinCallback`. Este método recibe las actualizaciones sobre las propiedades que se quieren del módulo gemelo y actualiza la variable **temperatureThreshold** para que coincida. Todos los módulos tienen su propio módulo gemelo, que le permite configurar el código que se ejecuta dentro del módulo directamente desde la nube.

    ```c
    static void moduleTwinCallback(DEVICE_TWIN_UPDATE_STATE update_state, const unsigned char* payLoad, size_t size, void* userContextCallback)
    {
        printf("\r\nTwin callback called with (state=%s, size=%zu):\r\n%s\r\n",
            MU_ENUM_TO_STRING(DEVICE_TWIN_UPDATE_STATE, update_state), size, payLoad);
        JSON_Value *root_value = json_parse_string(payLoad);
        JSON_Object *root_object = json_value_get_object(root_value);
        if (json_object_dotget_value(root_object, "desired.TemperatureThreshold") != NULL) {
            temperatureThreshold = json_object_dotget_number(root_object, "desired.TemperatureThreshold");
        }
        if (json_object_get_value(root_object, "TemperatureThreshold") != NULL) {
            temperatureThreshold = json_object_get_number(root_object, "TemperatureThreshold");
        }
    }
    ```

1. Busque la función `SetupCallbacksForModule`. Reemplace la función por el siguiente código, que agrega una instrucción *else if* para comprobar si se ha actualizado el módulo gemelo.

   ```c
   static int SetupCallbacksForModule(IOTHUB_MODULE_CLIENT_LL_HANDLE iotHubModuleClientHandle)
   {
       int ret;

       if (IoTHubModuleClient_LL_SetInputMessageCallback(iotHubModuleClientHandle, "input1", InputQueue1Callback, (void*)iotHubModuleClientHandle) != IOTHUB_CLIENT_OK)
       {
           printf("ERROR: IoTHubModuleClient_LL_SetInputMessageCallback(\"input1\")..........FAILED!\r\n");
           ret = MU_FAILURE;
       }
       else if (IoTHubModuleClient_LL_SetModuleTwinCallback(iotHubModuleClientHandle, moduleTwinCallback, (void*)iotHubModuleClientHandle) != IOTHUB_CLIENT_OK)
       {
           printf("ERROR: IoTHubModuleClient_LL_SetModuleTwinCallback(default)..........FAILED!\r\n");
           ret = MU_FAILURE;
       }
       else
       {
           ret = 0;
       }

       return ret;
   }
   ```

1. Guarde el archivo *main.c*.

1. Abra el archivo *deployment.template.json*.

1. Agregue el módulo gemelo de **CModule** al manifiesto de implementación. Inserte el siguiente contenido JSON en la parte inferior de la sección `moduleContent`, después del módulo gemelo `$edgeHub`:

   ```json
   "CModule": {
       "properties.desired":{
           "TemperatureThreshold":25
       }
   }
   ```

   ![Captura de pantalla que muestra la adición del módulo gemelo a la plantilla de implementación.](./media/tutorial-c-module-windows/module-twin.png)

1. Guarde el archivo *deployment.template.json*.

## <a name="build-and-push-your-module"></a>Compilación e inserción del módulo

En la sección anterior, creó una solución de IoT Edge y agregó código a **CModule** para filtrar los mensajes en los que la temperatura registrada por la máquina estaba por debajo del umbral aceptable. Ahora, tiene que compilar la solución como una imagen de contenedor e insertarla en el registro de contenedor.

### <a name="sign-in-to-docker"></a>Inicio de sesión en Docker

Especifique las credenciales del registro de contenedor para Docker en el equipo de desarrollo, con el fin de que pueda insertar la imagen de contenedor para que se almacene en el registro.

1. Abra PowerShell o una ventana de símbolo del sistema.

1. Inicie sesión en Docker con las credenciales del registro de contenedor de Azure que guardó después de crear el registro.

   ```cmd
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Es posible que reciba una advertencia de seguridad que recomienda el uso de `--password-stdin`. Aunque este es el procedimiento recomendado para escenarios de producción, está fuera del ámbito de este tutorial. Para más información, consulte la [referencia de docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

### <a name="build-and-push"></a>Compilación e inserción

La máquina de desarrollo ahora tiene acceso al registro de contenedores y también los dispositivos IoT Edge. Es hora de convertir el código del proyecto en una imagen de contenedor.

1. En el Explorador de soluciones de Visual Studio, haga clic con el botón derecho en el nombre del proyecto que quiere compilar. El nombre predeterminado es **AzureIotEdgeApp1**. En este tutorial, elegimos el nombre **CTutorialApp** y, dado que está compilando un módulo de Windows, la extensión debe ser **Windows.Amd64**.

1. Seleccione **Build and Push IoT Edge Modules** (Compilación e inserción de los módulos de IoT Edge).

   El comando de compilación e inserción inicia tres operaciones:
   * En primer lugar, crea una carpeta en la solución denominada *config*, que contiene el manifiesto completo de implementación. Se crea con la información de la plantilla de implementación y otros archivos de solución. 
   * En segundo lugar, ejecuta `docker build` para compilar la imagen de contenedor, en función del Dockerfile adecuado para la arquitectura de destino. 
   * Finalmente, ejecuta `docker push` para insertar el repositorio de imágenes en el registro de contenedor.

   Este proceso puede tardar varios minutos la primera vez, pero será más rápido la próxima vez que ejecute los comandos.

## <a name="deploy-modules-to-the-device"></a>Implementación de módulos en el dispositivo

Utilice Cloud Explorer de Visual Studio y la extensión Azure IoT Edge Tools para implementar el proyecto de módulo en el dispositivo IoT Edge. Ya ha preparado un manifiesto de implementación para su escenario, el archivo *deployment.windows-amd64.json* de la carpeta *config*. Ahora todo lo que necesita hacer es seleccionar un dispositivo que reciba la implementación.

Asegúrese de que el dispositivo IoT Edge está en funcionamiento.

1. En Cloud Explorer de Visual Studio, expanda los recursos para ver la lista de dispositivos IoT.

1. Haga clic con el botón derecho en el nombre del dispositivo IoT Edge que quiere que reciba la implementación.

1. Seleccione **Create Deployment** (Crear implementación).

1. En el Explorador de archivos de Visual Studio, seleccione el archivo *deployment.windows-amd64.json* de la carpeta *config* de la solución.

1. Actualice Cloud Explorer para ver los módulos implementados que aparecen en el dispositivo.

## <a name="view-generated-data"></a>Visualización de datos generados

Una vez aplicado el manifiesto de implementación al dispositivo de IoT Edge, el entorno de ejecución de IoT Edge del dispositivo recopila la nueva información de implementación y comienza a ejecutarse con ella. Los módulos que se ejecutan en el dispositivo, pero que no están incluidos en el manifiesto de implementación, se detienen. Los módulos que faltan en el dispositivo se inician.

Puede usar la extensión IoT Edge Tools para ver los mensajes a medida que llegan al centro de IoT.

1. En Cloud Explorer de Visual Studio, seleccione el nombre del dispositivo IoT Edge.

1. En la lista **Acciones**, seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión del evento integrado).

1. Vea los mensajes que llegan a su centro de IoT. Los mensajes pueden tardar un rato en llegar, ya que el dispositivo de IoT Edge tiene que recibir su nueva implementación e iniciar todos los módulos. Para que se puedan enviar los mensajes, los cambios en el código de CModule deben esperar hasta que la temperatura de la máquina alcance los 25 grados. El código también agrega el tipo de mensaje **Alerta** a los mensajes que alcanzan ese umbral de temperatura.

   ![Captura de pantalla de la ventana de salida que muestra los mensajes que llegan al centro de IoT.](./media/tutorial-c-module-windows/view-d2c-message.png)

## <a name="edit-the-module-twin"></a>Edición del módulo gemelo

El módulo gemelo de CModule se usa para establecer el umbral de temperatura en 25 grados. Puede usar al módulo gemelo para cambiar la funcionalidad sin tener que actualizar el código del módulo.

1. En Visual Studio, abra el archivo *deployment.windows amd64.json*. 

   *No* abra el archivo *deployment.template*. Si no ve el manifiesto de implementación en el archivo *config* en el Explorador de soluciones, seleccione el icono **Mostrar todos los archivos** en la barra de herramientas de dicho explorador.

1. Busque el gemelo de CModule y cambie el valor del parámetro **temperatureThreshold** por una nueva temperatura que esté entre 5 y 10 grados por encima de la última notificada.

1. Guarde el archivo *deployment.windows amd64.json*.

1. Siga de nuevo las instrucciones de implementación para aplicar el manifiesto de implementación actualizado al dispositivo.

1. Supervise los mensajes entrantes del dispositivo a la nube. Los mensajes deberían detenerse hasta que se alcance el nuevo umbral de temperatura.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede conservar los recursos y las configuraciones que ya ha creado en este tutorial y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba.

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que uso aquí.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un módulo IoT Edge que contiene código para filtrar datos sin procesar generados por el dispositivo IoT Edge.

Para saber cómo Azure IoT Edge puede ayudarle a implementar servicios en la nube de Azure para procesar y analizar datos en el perímetro, continúe con los siguientes tutoriales.

> [!div class="nextstepaction"]
> [Azure Functions](tutorial-deploy-function.md)
> [Azure Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Azure Machine Learning](tutorial-deploy-machine-learning.md)
> [Custom Vision Service](tutorial-deploy-custom-vision.md)
