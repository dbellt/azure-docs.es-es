---
title: 'Tutorial: Desarrollo de módulos con C# para Windows mediante Azure IoT Edge'
description: En este tutorial se muestra cómo crear módulos de IoT Edge con código de C# e implementarlos en dispositivos Windows que ejecutan IoT Edge.
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 08/03/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc, amqp, devx-track-csharp
monikerRange: =iotedge-2018-06
ms.openlocfilehash: 84e5bbff11d0a5ff5d47ca43a3da54d1f1fb5555
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106218862"
---
# <a name="tutorial-develop-c-iot-edge-modules-using-windows-containers"></a>Tutorial: Desarrollo de módulos de IoT Edge con C# con contenedores de Windows

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

En este artículo se muestra cómo usar Visual Studio para desarrollar código de C# e implementarlo en un dispositivo Windows que ejecuta Azure IoT Edge.

>[!NOTE]
>IoT Edge 1.1 LTS es el último canal de versión compatible con los contenedores de Windows. A partir de la versión 1.2, no se admiten los contenedores de Windows. Considere la posibilidad de usar o pasarse a [IoT Edge para Linux en Windows](iot-edge-for-linux-on-windows.md) para ejecutar IoT Edge en dispositivos Windows.

Los módulos de Azure IoT Edge se pueden usar para implementar código que implementa la lógica de negocios directamente en los dispositivos IoT Edge. En este tutorial, se detallan los pasos para crear e implementar un módulo de IoT Edge que filtra los datos de sensor.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Usar Visual Studio Code para crear un módulo de IoT Edge basado en el SDK de C#.
> * Usar Visual Studio y Docker para crear una imagen de Docker y publicarla en el Registro.
> * Implementar el módulo en el dispositivo IoT Edge.
> * Ver datos generados.

El módulo IoT Edge que creó en este tutorial filtra lo datos sobre la temperatura generados por el dispositivo. El módulo envía mensajes ascendentes solo si la temperatura supera un umbral especificado. Este tipo de análisis perimetral resulta útil para reducir la cantidad de datos que se comunican a la nube y se almacenan en ella.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

En este tutorial se muestra cómo desarrollar un módulo en C# mediante Visual Studio 2019 y, luego, implementarlo en un dispositivo Windows. Si va a desarrollar módulos para usar contenedores de Linux, vaya a [Desarrollo de un módulo IoT Edge con C# con contenedores de Linux](tutorial-csharp-module.md) en su lugar.

Para conocer las opciones para desarrollar e implementar módulos de C# con contenedores de Windows, consulte la siguiente tabla:

| C# | Visual&nbsp;Studio&nbsp;Code | Visual Studio 2017&nbsp;y&nbsp;2019 |
| -- | :------------------: | :------------------: |
| Desarrollo de Windows AMD64 | ![Desarrollo de módulos de C# para WinAMD64 en Visual Studio Code](./media/tutorial-c-module/green-check.png) | ![Desarrollo de módulos C# para WinAMD64 en Visual Studio](./media/tutorial-c-module/green-check.png) |
| Depuración de Windows AMD64 |   | ![Depuración de módulos C# para WinAMD64 en Visual Studio](./media/tutorial-c-module/green-check.png) |

Antes de comenzar este tutorial, configure el entorno de desarrollo siguiendo las instrucciones del tutorial [Desarrollo de módulos IoT Edge con contenedores de Windows](tutorial-develop-for-windows.md). Cuando haya terminado, el entorno contendrá los siguientes requisitos previos:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.
* Un [dispositivo Windows que ejecute Azure IoT Edge](how-to-install-iot-edge-windows-on-windows.md).
* Un registro de contenedor, como [Azure Container Registry](../container-registry/index.yml).
* [Visual Studio 2019](/visualstudio/install/install-visual-studio) configurado con la extensión [Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools).
* [Docker Desktop](https://docs.docker.com/docker-for-windows/install/) configurado para ejecutar contenedores de Windows.

> [!TIP]
> Si usa Visual Studio 2017 (versión 15.7 o posterior), descargue e instale Azure IoT Edge Tools para Visual Studio 2017 desde [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools).

## <a name="create-a-module-project"></a>Creación de un proyecto de módulo

En esta sección, creará un proyecto de módulo de IoT Edge mediante Visual Studio y la extensión Azure IoT Edge Tools. Después de crear una plantilla de proyecto, agrega código nuevo para que el módulo filtre los mensajes en función de sus propiedades notificadas.

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

Azure IoT Edge Tools proporciona plantillas de proyecto para todos los lenguajes del módulo IoT Edge admitidos en Visual Studio. Estas plantillas tienen todos los archivos y el código que necesita para implementar un módulo de trabajo con el fin de probar IoT Edge. También pueden proporcionarle un punto de partida para personalizarlas con su propia lógica de negocios.

1. Abra Visual Studio 2019 y seleccione **Crear nuevo proyecto**.

1. En el panel **Crear un nuevo proyecto**, busque **IoT Edge** y, luego, en la lista de resultados, seleccione el proyecto **Azure IoT Edge (Windows amd64)** .

   ![Captura de pantalla del panel "Crear un nuevo proyecto" de IoT Edge.](./media/tutorial-csharp-module-windows/new-project.png)

1. Seleccione **Next** (Siguiente).

    Se abre el panel **Configure su nuevo proyecto**.

   ![Captura de pantalla del panel "Configure su nuevo proyecto".](./media/tutorial-csharp-module-windows/configure-project.png)

1. En el panel **Configure su nuevo proyecto**, cambie el nombre del proyecto y la solución por algo más descriptivo, como **CSharpTutorialApp**. 

1. Seleccione **Crear** para crear el proyecto.

   Se abre el panel **Agregar módulo**.

   ![Captura de pantalla del panel "Agregar módulo" para configurar el proyecto.](./media/tutorial-csharp-module-windows/add-application-and-module.png)

1. En la página **Configure su nuevo proyecto**, haga lo siguiente:

   a. En el panel izquierdo, seleccione la plantilla **C# Module**.  
   b. En el cuadro **Nombre del módulo**, escriba **CSharpModule**.  
   c. En el cuadro **URL de repositorio**, reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure, en el siguiente formato: `<registry name>.azurecr.io/csharpmodule`.

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

1. En Visual Studio, seleccione **CSharpModule** > **Program.cs**.

1. En la parte superior del espacio de nombres **CSharpModule**, agregue tres instrucciones **using** para los tipos que se usarán más adelante en:

    ```csharp
    using System.Collections.Generic;     // For KeyValuePair<>
    using Microsoft.Azure.Devices.Shared; // For TwinCollection
    using Newtonsoft.Json;                // For JsonConvert
    ```

1. Agregue la variable **temperatureThreshold** a la clase **Program** tras la variable del contador. La variable temperatureThreshold establece el valor que debe superar la temperatura medida para que los datos se envíen al centro de IoT.

    ```csharp
    static int temperatureThreshold { get; set; } = 25;
    ```

1. Agregue las clases **MessageBody**, **Machine** y **Ambient** a la clase **Program** tras las declaraciones de variables. Estas clases definen el esquema esperado para el cuerpo de los mensajes entrantes.

    ```csharp
    class MessageBody
    {
        public Machine machine {get;set;}
        public Ambient ambient {get; set;}
        public string timeCreated {get; set;}
    }
    class Machine
    {
        public double temperature {get; set;}
        public double pressure {get; set;}
    }
    class Ambient
    {
        public double temperature {get; set;}
        public int humidity {get; set;}
    }
    ```

1. Busque el método **Init**. Este método crea y configura un objeto **ModuleClient**, que permite al módulo conectarse al entorno de ejecución de Azure IoT Edge local para enviar y recibir mensajes. El código también registra una devolución de llamada para recibir mensajes desde el centro de IoT Edge a través del punto de conexión **input1**.

   Reemplace el método Init por el código siguiente:

   ```csharp
   static async Task Init()
   {
       AmqpTransportSettings amqpSetting = new AmqpTransportSettings(TransportType.Amqp_Tcp_Only);
       ITransportSettings[] settings = { amqpSetting };

       // Open a connection to the Edge runtime.
       ModuleClient ioTHubModuleClient = await ModuleClient.CreateFromEnvironmentAsync(settings);
       await ioTHubModuleClient.OpenAsync();
       Console.WriteLine("IoT Hub module client initialized.");

       // Read the TemperatureThreshold value from the module twin's desired properties.
       var moduleTwin = await ioTHubModuleClient.GetTwinAsync();
       await OnDesiredPropertiesUpdate(moduleTwin.Properties.Desired, ioTHubModuleClient);

       // Attach a callback for updates to the module twin's desired properties.
       await ioTHubModuleClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertiesUpdate, null);

       // Register a callback for messages that are received by the module.
       await ioTHubModuleClient.SetInputMessageHandlerAsync("input1", FilterMessages, ioTHubModuleClient);
   }
   ```

   Este método Init actualizado todavía configura la conexión al entorno de ejecución de IoT Edge con ModuleClient, pero también agrega nueva funcionalidad. Lee las propiedades deseadas del gemelo del módulo para recuperar el valor **temperatureThreshold**. Luego, crea una devolución de llamada que escucha las actualizaciones futuras a las propiedades deseadas del módulo gemelo. Con esta devolución de llamada, puede actualizar el umbral de temperatura en el módulo gemelo de forma remota y los cambios se incorporarán al módulo.

   El método Init actualizado también cambia el método existente **SetInputMessageHandlerAsync**. En el código de ejemplo, los mensajes entrantes en *input1* se procesan con la función *PipeMessage*, pero deseamos cambiar este valor para utilizar la función *FilterMessages* que crearemos en los pasos siguientes.

1. Agregue el nuevo método **onDesiredPropertiesUpdate** a la clase **Program**. Este método recibe actualizaciones sobre las propiedades deseadas del módulo gemelo y actualiza la variable **temperatureThreshold** para que coincida. Todos los módulos tienen su propio módulo gemelo, que le permite configurar el código que se ejecuta dentro de un módulo directamente desde la nube.

    ```csharp
    static Task OnDesiredPropertiesUpdate(TwinCollection desiredProperties, object userContext)
    {
        try
        {
            Console.WriteLine("Desired property change:");
            Console.WriteLine(JsonConvert.SerializeObject(desiredProperties));

            if (desiredProperties["TemperatureThreshold"]!=null)
                temperatureThreshold = desiredProperties["TemperatureThreshold"];

        }
        catch (AggregateException ex)
        {
            foreach (Exception exception in ex.InnerExceptions)
            {
                Console.WriteLine();
                Console.WriteLine("Error when receiving desired property: {0}", exception);
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error when receiving desired property: {0}", ex.Message);
        }
        return Task.CompletedTask;
    }
    ```

1. Quite el método de ejemplo **PipeMessage** y reemplácelo por un nuevo método **FilterMessages**. Se llama a este método cada vez que el módulo recibe un mensaje del centro de IoT Edge. Este método filtra los mensajes que informan de temperaturas por debajo del umbral de temperatura que se establece mediante el módulo gemelo. También agrega la propiedad **MessageType** al mensaje con el valor establecido en **Alerta**.

    ```csharp
    static async Task<MessageResponse> FilterMessages(Message message, object userContext)
    {
        var counterValue = Interlocked.Increment(ref counter);
        try
        {
            ModuleClient moduleClient = (ModuleClient)userContext;
            var messageBytes = message.GetBytes();
            var messageString = Encoding.UTF8.GetString(messageBytes);
            Console.WriteLine($"Received message {counterValue}: [{messageString}]");

            // Get the message body.
            var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

            if (messageBody != null && messageBody.machine.temperature > temperatureThreshold)
            {
                Console.WriteLine($"Machine temperature {messageBody.machine.temperature} " +
                    $"exceeds threshold {temperatureThreshold}");
                using(var filteredMessage = new Message(messageBytes))
                {
                    foreach (KeyValuePair<string, string> prop in message.Properties)
                    {
                        filteredMessage.Properties.Add(prop.Key, prop.Value);
                    }

                    filteredMessage.Properties.Add("MessageType", "Alert");
                    await moduleClient.SendEventAsync("output1", filteredMessage);
                }
            }

            // Indicate that the message treatment is completed.
            return MessageResponse.Completed;
        }
        catch (AggregateException ex)
        {
            foreach (Exception exception in ex.InnerExceptions)
            {
                Console.WriteLine();
                Console.WriteLine("Error in sample: {0}", exception);
            }
            // Indicate that the message treatment is not completed.
            var moduleClient = (ModuleClient)userContext;
            return MessageResponse.Abandoned;
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
            // Indicate that the message treatment is not completed.
            ModuleClient moduleClient = (ModuleClient)userContext;
            return MessageResponse.Abandoned;
        }
    }
    ```

1. Guarde el archivo *Program.cs*.

1. Abra el archivo *deployment.template.json* en la solución IoT Edge. Este archivo le indica al agente de IoT Edge qué módulos implementar y le dice al centro de IoT Edge cómo enrutar los mensajes entre ellos. Aquí, los módulos que se van a implementar son **SimulatedTemperatureSensor** y **CSharpModule**.

1. Agregue el módulo gemelo **CSharpModule** al manifiesto de implementación. Inserte el siguiente contenido JSON en la parte inferior de la sección `modulesContent`, después del módulo gemelo **$edgeHub**:

    ```json
       "CSharpModule": {
           "properties.desired":{
               "TemperatureThreshold":25
           }
       }
    ```

    ![Captura de pantalla que muestra la adición del módulo gemelo a la plantilla de implementación.](./media/tutorial-csharp-module-windows/module-twin.png)

1. Guarde el archivo *deployment.template.json*.

## <a name="build-and-push-your-module"></a>Compilación e inserción del módulo

En la sección anterior se ha creado una solución de IoT Edge y se ha agregado código a **CSharpModule** para filtrar los mensajes en los que la temperatura registrada de la máquina está por debajo del umbral aceptable. Ahora, tiene que compilar la solución como una imagen de contenedor e insertarla en el registro de contenedor.

### <a name="sign-in-to-docker"></a>Inicio de sesión en Docker

1. Utilice el siguiente comando para iniciar sesión en Docker en la máquina de desarrollo. Use el nombre de usuario, la contraseña y el servidor de inicio de sesión de Azure Container Registry. Puede recuperar estos valores en la sección **Claves de acceso** del Registro en Azure Portal.

   ```cmd
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Es posible que reciba una advertencia de seguridad que recomienda el uso de `--password-stdin`. Aunque este es el procedimiento recomendado para escenarios de producción, está fuera del ámbito de este tutorial. Para más información, consulte la [referencia de docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

### <a name="build-and-push"></a>Compilación e inserción

1. En el Explorador de soluciones de Visual Studio, haga clic con el botón derecho en el nombre del proyecto que quiere compilar. El nombre predeterminado es **AzureIotEdgeApp1** y, puesto que va a compilar un módulo de Windows, la extensión debe ser **Windows.Amd64**.

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

1. Vea los mensajes que llegan a su centro de IoT. Los mensajes pueden tardar un rato en llegar, ya que el dispositivo de IoT Edge tiene que recibir su nueva implementación e iniciar todos los módulos. Para que se puedan enviar los mensajes, los cambios en el código de CSharpModule deben esperar hasta que la temperatura de la máquina alcance los 25 grados. El código también agrega el tipo de mensaje **Alerta** a los mensajes que alcanzan ese umbral de temperatura.

   ![Captura de pantalla de la ventana de salida que muestra los mensajes que llegan al centro de IoT.](./media/tutorial-csharp-module-windows/view-d2c-message.png)

## <a name="edit-the-module-twin"></a>Edición del módulo gemelo

El módulo gemelo de CSharpModule se usa para establecer el umbral de temperatura en 25 grados. Puede usar al módulo gemelo para cambiar la funcionalidad sin tener que actualizar el código del módulo.

1. En Visual Studio, abra el archivo *deployment.windows amd64.json*. 

   *No* abra el archivo *deployment.template*. Si no ve el manifiesto de implementación en el archivo *config* en el Explorador de soluciones, seleccione el icono **Mostrar todos los archivos** en la barra de herramientas de dicho explorador.

1. Busque el gemelo de CSharpModule y cambie el valor del parámetro **temperatureThreshold** por una nueva temperatura que esté entre 5 y 10 grados por encima de la última notificada.

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
