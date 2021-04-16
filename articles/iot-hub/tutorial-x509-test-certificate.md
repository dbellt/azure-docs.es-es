---
title: 'Tutorial: Prueba de la capacidad de los certificados X.509 para autenticar dispositivos en una instancia de Azure IoT Hub | Microsoft Docs'
description: 'Tutorial: Prueba de los certificados X.509 para autenticarse en Azure IoT Hub'
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
ms.openlocfilehash: 7d1900782fce6b84ed79014e985393f3626d171b
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107379441"
---
# <a name="tutorial-testing-certificate-authentication"></a>Tutorial: Prueba de la autenticación de certificados

Puede usar el siguiente ejemplo de código C# para comprobar si el certificado puede autenticar el dispositivo en la instancia de IoT Hub. Tenga en cuenta que debe hacer lo siguiente antes de ejecutar el código de prueba:

* Cree un certificado de entidad de certificación raíz o subordinada.
* Cargue el certificado de la entidad de certificación en la instancia de IoT Hub.
* Demuestre que posee el certificado de la entidad de certificación.
* Agregue un dispositivo a la instancia de IoT Hub.
* Cree un certificado de dispositivo con el mismo identificador de dispositivo que el dispositivo.

## <a name="code-example"></a>Ejemplo de código

En el siguiente ejemplo se muestra cómo crear una aplicación de C# para simular el dispositivo X.509 registrado en su instancia de IoT Hub. En el ejemplo, se envían los valores de temperatura y humedad del dispositivo simulado al centro de conectividad. En este tutorial, solo se va a crear la aplicación del dispositivo. La creación de la aplicación del servicio IoT Hub que enviará una respuesta a los eventos enviados por este dispositivo simulado queda como ejercicio para los lectores.

1. Abra Visual Studio, seleccione **Crear un proyecto nuevo** y luego seleccione la plantilla de proyecto **Aplicación de consola (.NET Framework)** . Seleccione **Next** (Siguiente).

1. En **Configure su nuevo proyecto**, asigne al proyecto el nombre *SimulateX509Device* y luego seleccione **Crear**.

   ![Creación del proyecto de dispositivo X.509 en Visual Studio](./media/iot-hub-security-x509-get-started/create-device-project-vs2019.png)

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **SimulateX509Device** y seleccione **Administrar paquetes NuGet**.

1. En **Administrador de paquetes NuGet**, seleccione **Examinar** y busque y elija **Microsoft.Azure.Devices.Client**. Seleccione **Instalar**.

   ![Adición del paquete NuGet del SDK de dispositivo en Visual Studio](./media/iot-hub-security-x509-get-started/device-sdk-nuget.png)

    Este paso permite descargar, instalar y agregar una referencia al paquete NuGet del SDK de dispositivo IoT de Azure y sus dependencias.

    Escriba y ejecute el código siguiente:

```csharp
using System;
using Microsoft.Azure.Devices.Client;
using System.Security.Cryptography.X509Certificates;
using System.Threading.Tasks;
using System.Text;

namespace SimulateX509Device
{
    class Program
    {
        private static int MESSAGE_COUNT = 5;

        // Temperature and humidity variables.
        private const int TEMPERATURE_THRESHOLD = 30;
        private static float temperature;
        private static float humidity;
        private static Random rnd = new Random();

        // Set the device ID to the name (device identifier) of your device.
        private static String deviceId = "{your-device-id}";

        static async Task SendEvent(DeviceClient deviceClient)
        {
            string dataBuffer;
            Console.WriteLine("Device sending {0} messages to IoTHub...\n", MESSAGE_COUNT);

            // Iterate MESSAGE_COUNT times to set randomm termperature and humidity values.
            for (int count = 0; count < MESSAGE_COUNT; count++)
            {
                // Set random values for temperature and humidity.
                temperature = rnd.Next(20, 35);
                humidity = rnd.Next(60, 80);
                dataBuffer = string.Format("{{\"deviceId\":\"{0}\",\"messageId\":{1},\"temperature\":{2},\"humidity\":{3}}}", deviceId, count, temperature, humidity);
                Message eventMessage = new Message(Encoding.UTF8.GetBytes(dataBuffer));
                eventMessage.Properties.Add("temperatureAlert", (temperature > TEMPERATURE_THRESHOLD) ? "true" : "false");
                Console.WriteLine("\t{0}> Sending message: {1}, Data: [{2}]", DateTime.Now.ToLocalTime(), count, dataBuffer);

                // Send to IoT Hub.
                await deviceClient.SendEventAsync(eventMessage);
            }
        }
        static void Main(string[] args)
        {
            try
            {
                // Create an X.509 certificate object.
                var cert = new X509Certificate2(@"{full path to pfx certificate.pfx}", "{your certificate password}");

                // Create an authentication object using your X.509 certificate. 
                var auth = new DeviceAuthenticationWithX509Certificate("{your-device-id}", cert);

                // Create the device client.
                var deviceClient = DeviceClient.Create("{your-IoT-Hub-name}.azure-devices.net", auth, TransportType.Mqtt);

                if (deviceClient == null)
                {
                    Console.WriteLine("Failed to create DeviceClient!");
                }
                else
                {
                    Console.WriteLine("Successfully created DeviceClient!");
                    SendEvent(deviceClient).Wait();
                }

                Console.WriteLine("Exiting...\n");
            }
            catch (Exception ex)
            {
                Console.WriteLine("Error in sample: {0}", ex.Message);
            }
         }
    }
}
```