---
title: Ejemplos de escenarios de implementación de la versión preliminar de Caché conectada de Microsoft | Microsoft Docs
titleSuffix: Device Update for Azure IoT Hub
description: Tutoriales de ejemplos de escenarios de implementación de la versión preliminar de Caché conectada de Microsoft
author: andyriv
ms.author: andyriv
ms.date: 2/16/2021
ms.topic: tutorial
ms.service: iot-hub-device-update
ms.openlocfilehash: c116bbf5ea9f5fc6e58962e02c93c630fc747d9e
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107811731"
---
# <a name="microsoft-connected-cache-preview-deployment-scenario-samples"></a>Ejemplos de escenarios de implementación de la versión preliminar de Caché conectada de Microsoft

## <a name="single-level-azure-iot-edge-gateway-no-proxy"></a>Puerta de enlace de Azure IoT Edge de nivel único sin proxy

En el diagrama siguiente se describe el escenario en el que una puerta de enlace de Azure IoT Edge que tiene acceso directo a los recursos de la red CDN y tiene un dispositivo hoja Azure IoT, como un Raspberry PI, que es un dispositivo secundario aislado de Internet de la puerta de enlace de Azure IoT Edge. 

  :::image type="content" source="media/connected-cache-overview/disconnected-device-update.png" alt-text="Actualización de un dispositivo sin conexión con Caché conectada de Microsoft" lightbox="media/connected-cache-overview/disconnected-device-update.png":::

1. Agregue el módulo de Caché conectada de Microsoft a la implementación del dispositivo de puerta de enlace de Azure IoT Edge en Azure IoT Hub (consulte [Soporte técnico para dispositivos desconectados](connected-cache-disconnected-device-update.md) para más información sobre cómo obtener el módulo).
2. Agregue las variables de entorno para la implementación. A continuación se muestra un ejemplo de las variables de entorno.

    **Variables de entorno**
    
    | Nombre                          | Value                                                                 |
    | ----------------------------- | ----------------------------------------------------------------------| 
    | CACHE_NODE_ID                 | Consulte las descripciones de las [variables de entorno](connected-cache-configure.md). |
    | CUSTOMER_ID                   | Consulte las descripciones de las [variables de entorno](connected-cache-configure.md). |
    | CUSTOMER_KEY                  | Consulte las descripciones de las [variables de entorno](connected-cache-configure.md). |
    | STORAGE_1_SIZE_GB             | 10                                                                    |

3. Agregue las opciones de creación del contenedor para la implementación. A continuación se muestra un ejemplo de las opciones de creación del contenedor.

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

Para comprobar que Caché conectada de Microsoft funciona correctamente, ejecute el siguiente comando en el terminal del dispositivo IoT Edge que hospeda el módulo o en cualquier dispositivo de la red. Reemplace \<Azure IoT Edge Gateway IP\> por la dirección IP o el nombre de host de la puerta de enlace de IoT Edge (consulte los detalles de la variable de entorno para obtener información sobre la visibilidad de este informe).

```bash
    wget http://<IoT Edge Gateway IP>/mscomtest/wuidt.gif?cacheHostOrigin=au.download.windowsupdate.com
```

## <a name="single-level-azure-iot-edge-gateway-with-outbound-unauthenticated-proxy"></a>Puerta de enlace de Azure IoT Edge de nivel único con proxy de salida no autenticado

En este escenario hay una puerta de enlace de Azure IoT Edge que tiene acceso a los recursos de la red CDN mediante un proxy de salida no autenticado. Se va a configurar Caché conectada de Microsoft para almacenar en caché el contenido de un repositorio personalizado y el informe de resumen se ha hecho visible para cualquier usuario de la red. A continuación se muestra un ejemplo de las variables de entorno de MCC que se van a establecer.

  :::image type="content" source="media/connected-cache-overview/single-level-proxy.png" alt-text="Proxy de nivel único de Caché conectada de Microsoft" lightbox="media/connected-cache-overview/single-level-proxy.png":::

1. Agregue el módulo de Caché conectada de Microsoft a la implementación del dispositivo de puerta de enlace de Azure IoT Edge en Azure IoT Hub.
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
    | UPSTREAM_PROXY                | Dirección IP o FQDN del servidor proxy                                          |

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

Para comprobar que Caché conectada de Microsoft funciona correctamente, ejecute el siguiente comando en el terminal del dispositivo Azure IoT Edge que hospeda el módulo o en cualquier dispositivo de la red. Reemplace \<Azure IoT Edge Gateway IP\> por la dirección IP o el nombre de host de la puerta de enlace de IoT Edge (consulte los detalles de la variable de entorno para obtener información sobre la visibilidad de este informe).

```bash
    wget http://<Azure IoT Edge Gateway IP>/mscomtest/wuidt.gif?cacheHostOrigin=au.download.windowsupdate.com 
```
