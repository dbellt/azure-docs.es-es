---
title: archivo de inclusión
description: archivo de inclusión
services: iot-hub
author: chrissie926
ms.service: iot-hub
ms.topic: include
ms.date: 08/07/2019
ms.author: menchi
ms.custom: include file
ms.openlocfilehash: a5c1ddd085ae65b9920d73f50f993f4646785a69
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "68883828"
---
## <a name="create-a-module-identity"></a>Creación de una identidad de módulo

En esta sección se crea una aplicación de consola .NET que crea una identidad de dispositivo y otra de módulo en el registro de identidades del centro. No se puede conectar un dispositivo o un módulo al centro a menos que tenga una entrada en el registro de identidades. Para más información, consulte la sección sobre el [registro de la identidad de la guía para desarrolladores de IoT Hub](../articles/iot-hub/iot-hub-devguide-identity-registry.md).

Cuando se ejecuta esta aplicación de consola, genera una clave y un identificador únicos para el dispositivo y el módulo. El dispositivo y el módulo usan estos valores para identificarse al enviar mensajes del dispositivo a la nube a IoT Hub. Los identificadores distinguen mayúsculas de minúsculas.

1. Abra Visual Studio y seleccione **Crear un proyecto**.

1. En **Crear un proyecto**, seleccione **Aplicación de consola (.NET Framework)**.

1. Seleccione **Siguiente** para abrir **Configurar el nuevo proyecto**. Llame *CreateIdentities* al proyecto e *IoTHubGetStarted* a la solución. Asegúrese de que la versión de .NET Framework sea 4.6.1 o una posterior.

    ![Especificación del nombre y la plataforma de la solución de Visual Studio](./media/iot-hub-get-started-create-module-identity-csharp/configure-createidentities-project.png)

1. En Visual Studio, abra **Herramientas** > **Administrador de paquetes NuGet** > **Administrar paquetes NuGet para la solución**. Seleccione la pestaña **Examinar**.

1. Busque **Microsoft.Azure.Devices**. Selecciónelo y, a continuación, seleccione **Instalar**.

    ![Instalación de la versión actual del SDK del servicio .NET de Azure IoT Hub](./media/iot-hub-get-started-create-module-identity-csharp/install-service-sdk.png)

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs** :

   ```csharp
   using Microsoft.Azure.Devices;
   using Microsoft.Azure.Devices.Common.Exceptions;
   ```

1. Agregue los campos siguientes a la clase **Program** . Sustituya el valor de marcador de posición por la cadena de conexión de IoT Hub para el centro que creó en la sección anterior.

   ```csharp
   const string connectionString = "<replace_with_iothub_connection_string>";
   const string deviceID = "myFirstDevice";
   const string moduleID = "myFirstModule";
   ```

1. Agregue el siguiente código al final de la clase **Main**.

   ```csharp
   static void Main(string[] args)
   {
       AddDeviceAsync().Wait();
       AddModuleAsync().Wait();
   }
   ```

1. Agregue los métodos siguientes a la clase **Program**:

    ```csharp
    private static async Task AddDeviceAsync()
    {
       RegistryManager registryManager = 
         RegistryManager.CreateFromConnectionString(connectionString);
       Device device;

       try
       {
           device = await registryManager.AddDeviceAsync(new Device(deviceID));
       }
       catch (DeviceAlreadyExistsException)
        {
            device = await registryManager.GetDeviceAsync(deviceID);
        }

        Console.WriteLine("Generated device key: {0}", 
          device.Authentication.SymmetricKey.PrimaryKey);
    }

    private static async Task AddModuleAsync()
    {
        RegistryManager registryManager = 
          RegistryManager.CreateFromConnectionString(connectionString);
        Module module;

        try
        {
            module = 
              await registryManager.AddModuleAsync(new Module(deviceID, moduleID));
        }
        catch (ModuleAlreadyExistsException)
        {
            module = await registryManager.GetModuleAsync(deviceID, moduleID);
        }

        Console.WriteLine("Generated module key: {0}", module.Authentication.SymmetricKey.PrimaryKey);
    }
    ```

    El método `AddDeviceAsync` crea una identidad del dispositivo con el identificador **myFirstDevice**. Si el identificador de dispositivo ya existe en el registro de identidades, el código simplemente recupera la información existente del dispositivo. A continuación, la aplicación muestra la clave principal de esa identidad. Esta clave se usará en la aplicación de dispositivo simulado para conectarse al centro.

    El método `AddModuleAsync` crea una identidad de módulo con el identificador **myFirstModule** en el dispositivo **myFirstDevice**. Si el identificador de módulo ya existe en el registro de identidades, el código simplemente recupera la información existente del módulo. A continuación, la aplicación muestra la clave principal de esa identidad. Esta clave se usará en la aplicación de módulo simulado para conectarse al centro.

   [!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

1. Ejecute la aplicación y anote la clave del dispositivo y del módulo.

> [!NOTE]
> El registro de identidades de IoT Hub solo almacena identidades de dispositivos y módulos para permitir el acceso seguro al centro. El registro de identidades almacena los identificadores y las claves de dispositivo para usarlas como credenciales de seguridad. El registro de identidades también almacena una marca de habilitado o deshabilitado de cada dispositivo que se puede usar para deshabilitar el acceso a dicho dispositivo. Si la aplicación necesita almacenar otros metadatos específicos del dispositivo, debe usar un almacén específico de la aplicación. No hay marcas de habilitado/deshabilitado para las identidades de módulo. Consulte la [guía de desarrolladores de IoT Hub](../articles/iot-hub/iot-hub-devguide-identity-registry.md) para obtener más información.
