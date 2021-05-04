---
title: Puerta de enlace Azure IoT Edge anidada de dos niveles de Caché conectada de Microsoft con proxy no autenticado saliente | Microsoft Docs
titleSuffix: Device Update for Azure IoT Hub
description: Tutorial de la puerta de enlace Azure IoT Edge anidada de dos niveles de Caché conectada de Microsoft con proxy no autenticado saliente
author: andyriv
ms.author: andyriv
ms.date: 2/16/2021
ms.topic: tutorial
ms.service: iot-hub-device-update
ms.openlocfilehash: 623ce808423f76ae1be079e0424fe3ddf27d1d58
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107811893"
---
# <a name="microsoft-connected-cache-preview-deployment-scenario-sample-two-level-nested-azure-iot-edge-gateway-with-outbound-unauthenticated-proxy"></a>Ejemplo de escenario de implementación de la versión preliminar de Caché conectada de Microsoft: puerta de enlace Azure IoT Edge anidada de dos niveles con proxy no autenticado saliente

En el diagrama siguiente se describe el escenario en el que una puerta de enlace de Azure IoT Edge, que es un acceso directo a los recursos de la red CDN, funciona como elemento principal de otra puerta de enlace de Azure IoT Edge. La puerta de enlace de IoT Edge secundaria funciona como elemento principal de un dispositivo hoja Azure IoT, como un dispositivo Raspberry Pi. Tanto el elemento secundario de Azure IoT Edge como el dispositivo de Azure IoT están aislados de Internet. En el ejemplo siguiente se muestra la configuración para puertas de enlace de Azure IoT Edge de dos niveles, pero no hay límite en cuanto a la profundidad de hosts ascendentes que admitirá Caché conectada de Microsoft. No hay ninguna diferencia en las opciones de creación del contenedor de Caché conectada de Microsoft de los ejemplos anteriores.

Consulte la documentación sobre la [conexión de dispositivos IoT Edge de bajada: Azure IoT Edge](../iot-edge/how-to-connect-downstream-iot-edge-device.md?preserve-view=true&tabs=azure-portal&view=iotedge-2020-11) para obtener más información sobre la configuración de implementaciones por capas de las puertas de enlace Azure IoT Edge. Tenga en cuenta también que al implementar Azure IoT Edge, la caché conectada de Microsoft y los módulos personalizados, es decir todos los módulos, deben residir en el mismo registro de contenedor.

>[!Note]
>Al implementar Azure IoT Edge, la caché conectada de Microsoft y los módulos personalizados, es decir todos los módulos, deben residir en el mismo registro de contenedor.

  :::image type="content" source="media/connected-cache-overview/nested-level-proxy.png" alt-text="Caché conectada de Microsoft anidada" lightbox="media/connected-cache-overview/nested-level-proxy.png":::

## <a name="parent-gateway-configuration"></a>Configuración de puerta de enlace primaria
1. Agregue el módulo de Caché conectada de Microsoft a la implementación del dispositivo de puerta de enlace de Azure IoT Edge en Azure IoT Hub (consulte [Soporte técnico para dispositivos desconectados](connected-cache-disconnected-device-update.md) para más información sobre cómo obtener el módulo).
2. Agregue las variables de entorno para la implementación. A continuación se muestra un ejemplo de las variables de entorno.

    **Variables de entorno**

    | Nombre                          | Value                                                                 |
    | ----------------------------- | ----------------------------------------------------------------------| 
    | CACHE_NODE_ID                 | Consulte las descripciones de las [variables de entorno](connected-cache-configure.md). |
    | CUSTOMER_ID                   | Consulte las descripciones de las [variables de entorno](connected-cache-configure.md). |
    | CUSTOMER_KEY                  | Consulte las descripciones de las [variables de entorno](connected-cache-configure.md). |
    | STORAGE_1_SIZE_GB             | 10                                                                    |
    | CACHEABLE_CUSTOM_1_HOST       | Packagerepo.com:80                                                    |
    | CACHEABLE_CUSTOM_1_CANONICAL  | Packagerepo.com                                                       |
    | IS_SUMMARY_ACCESS_UNRESTRICTED| true                                                                  |

3. Agregue las opciones de creación del contenedor para la implementación. No hay ninguna diferencia en las opciones de creación del contenedor de MCC del ejemplo anterior. A continuación se muestra un ejemplo de las opciones de creación del contenedor.

### <a name="container-create-options"></a>Opciones de creación del contenedor

```json
{
    "HostConfig": {
        "Binds": [
            "/MicrosoftConnectedCache1/:/nginx/cache1/"
        ],
        "PortBindings": {
            "8081/tcp": [
                {
                    "HostPort": "80"
                }
            ],
            "5000/tcp": [
                {
                    "HostPort": "5100"
                }
            ]
        }
    }
}
```

## <a name="child-gateway-configuration"></a>Configuración de puerta de enlace secundaria

>[!Note]
>Si ha replicado contenedores en su configuración en su propio registro privado, será necesario modifica la configuración de config.toml y del entorno de ejecución en la implementación del módulo. Para más información, consulte [Conexión de dispositivos IoT Edge de nivel inferior: Azure IoT Edge](../iot-edge/how-to-connect-downstream-iot-edge-device.md?preserve-view=true&tabs=azure-portal&view=iotedge-2020-11#deploy-modules-to-lower-layer-devices).


1. Modifique la ruta de acceso de la imagen para el agente de Edge, como se muestra en el ejemplo siguiente:

    ```markdown
    [agent]
    name = "edgeAgent"
    type = "docker"
    env = {}
    [agent.config]
    image = "<parent_device_fqdn_or_ip>:8000/iotedge/azureiotedge-agent:1.2.0-rc2"
    auth = {}
    ```
2. Modifique la configuración del entorno de ejecución del centro de Microsoft Edge y el agente de Edge en la implementación de Azure IoT Edge, como se muestra en este ejemplo:
    
    * En el centro de Microsoft Edge, en el campo de imagen, escriba ```$upstream:8000/iotedge/azureiotedge-hub:1.2.0-rc2```.
    * En el agente de Edge, en el campo de imagen, escriba ```$upstream:8000/iotedge/azureiotedge-agent:1.2.0-rc2```.

3. Agregue el módulo Caché conectada de Microsoft a la implementación del dispositivo de puerta de enlace Azure IoT Edge en Azure IoT Hub.

   * Elija un nombre para el módulo: ```ConnectedCache```
   * Modifique el URI de la imagen: ```$upstream:8000/mcc/linux/iot/mcc-ubuntu-iot-amd64:latest```

4. Agregue el mismo conjunto de variables de entorno y opciones de creación de contenedor que se utilizó en la implementación primaria.
>[!Note]
>CACHE_NODE_ID debe ser único.  Los valores CUSTOMER_ID y CUSTOMER_KEY serán idénticos al elemento principal (consulte [Configuración de Caché conectada de Microsoft](connected-cache-configure.md)).

Para comprobar que la caché conectada de Microsoft funciona correctamente, ejecute el siguiente comando en el terminal del dispositivo IoT Edge que hospeda el módulo o en cualquier dispositivo de la red. Reemplace \<Azure IoT Edge Gateway IP\> por la dirección IP o el nombre de host de la puerta de enlace de IoT Edge (consulte los detalles de la variable de entorno para obtener información sobre la visibilidad de este informe).

```bash
    wget http://<CHILD Azure IoT Edge Gateway IP>/mscomtest/wuidt.gif?cacheHostOrigin=au.download.windowsupdate.com
```