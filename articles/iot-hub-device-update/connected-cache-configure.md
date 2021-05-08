---
title: Configuración de Caché conectada de Microsoft para Device Update para Azure IoT Hub | Microsoft Docs
titleSuffix: Device Update for Azure IoT Hub
description: Introducción a Caché conectada de Microsoft para Device Update para Azure IoT Hub
author: andyriv
ms.author: andyriv
ms.date: 2/16/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: 6e7b8d567034cc9557a2d9fcec4afbffa878cf75
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107811838"
---
# <a name="configure-microsoft-connected-cache-for-device-update-for-azure-iot-hub"></a>Configuración de Caché conectada de Microsoft para Device Update para Azure IoT Hub

Caché conectada de Microsoft se implementa en las puertas de enlace Azure IoT Edge como un módulo Azure IoT Edge. Al igual que otros módulos Azure IoT Edge, las variables de entorno de la implementación del módulo MCC y las opciones de creación del contenedor se usan para configurar los módulos de Caché conectada de Microsoft.  En esta sección se definen las variables de entorno y las opciones de creación del contenedor necesarias para que un cliente pueda implementar correctamente el módulo Caché conectada de Microsoft para su uso por parte de Device Update para Azure IoT Hub.

## <a name="microsoft-connected-cache-azure-iot-edge-module-deployment-details"></a>Detalles de implementación del módulo Azure IoT Edge Caché conectada de Microsoft

El nombre del módulo Caché conectada de Microsoft se asigna según el criterio del administrador. No hay otras interacciones de módulos o servicios que se basen en el nombre del módulo para la comunicación. Además, la relación entre elementos primarios y secundarios de los servidores de Caché conectada de Microsoft no depende de este nombre de módulo, sino del FQDN o la dirección IP de la puerta de enlace Azure IoT Edge que se ha configurado, como se explicó anteriormente.

Las variables de entorno del módulo Azure IoT Edge Caché conectada de Microsoft se usan para pasar la información de identidad del módulo básica y la configuración funcional del módulo al contenedor.

| Nombre de variable                 | Formato del valor                           | Obligatorio/opcional | Funcionalidad                                    |
| ----------------------------- | ---------------------------------------| ----------------- | ------------------------------------------------ |
| CUSTOMER_ID                   | GUID del identificador de suscripción de Azure             | Obligatorio          | Esta es la clave del cliente, que proporciona<br>autenticación segura del nodo de caché para Optimización de distribución<br>Servicios.<br>Se requiere para que el módulo funcione. |
| CACHE_NODE_ID                 | GUID del identificador del nodo de caché                     | Obligatorio          | Identifica de forma única el nodo de Caché conectada de Microsoft<br>para los servicios de Optimización de distribución.<br>Se requiere para que<br> el módulo funcione. |
| CUSTOMER_KEY                  | GUID de la clave de cliente                     | Obligatorio          | Esta es la clave del cliente, que proporciona<br>autenticación segura del nodo de caché para los servicios de Optimización de distribución.<br>Se requiere para que el módulo funcione.|
| STORAGE_ *N* _SIZE_GB           | Donde N es la unidad de caché   | Requerido          | Especifique hasta 9 unidades para almacenar en caché el contenido y especifique el espacio máximo en<br>gigabytes que se va a asignar para el contenido en cada unidad de caché. Ejemplos:<br>STORAGE_1_SIZE_GB = 150<br>STORAGE_2_SIZE_GB = 50<br>El número de la unidad debe coincidir con los valores del enlace de la unidad de caché especificados<br>en la valor de la opción de creación del contenedor MicrosoftConnectedCache *N*.<br>El tamaño mínimo de la caché es de 10 GB.|
| UPSTREAM_HOST                 | FQDN/IP                                | Opcional          | Este valor puede especificar un nodo de Caché conectada de Microsoft<br>que actúa como proxy si el nodo de Caché conectada<br> está desconectado de Internet. Esta configuración se usa para admitir<br> el escenario de IoT anidado.<br>**Nota:** Caché conectada de Microsoft escucha en el puerto http predeterminado 80.|
| UPSTREAM_PROXY                | FQDN/IP:PORT                           | Opcional          | Servidor proxy de salida de Internet.<br>También podría ser el proxy de la DMZ de OT si se trata de una red ISA 95. |
| CACHEABLE_CUSTOM_ *N* _HOST     | HOST/IP<br>FQDN                        | Opcional          | Necesario para admitir repositorios de paquetes personalizados.<br>Los repositorios se pueden hospedar localmente o en Internet.<br>No hay ningún límite en el número de hosts personalizados que se pueden configurar.<br><br>Ejemplos:<br>Nombre = CACHEABLE_CUSTOM_1_HOST valor = packages.foo.com<br> Nombre = CACHEABLE_CUSTOM_2_HOST valor = packages.bar.com    |
| CACHEABLE_CUSTOM_ *N* _CANONICAL| Alias                                  | Opcional          | Necesario para admitir repositorios de paquetes personalizados.<br>Este valor se puede usar como alias y el servidor de caché lo utilizará para hacer referencia<br>a nombres DNS diferentes. Por ejemplo, el nombre de host del contenido del repositorio puede ser packages.foo.com,<br>pero para distintas regiones podría haber un prefijo adicional que se agrega al nombre de host,<br>como westuscdn.packages.foo.com y eastuscdn.packages.foo.com.<br>Al establecer el alias canónico, se asegura de que el contenido no esté duplicado<br>para el contenido procedente del mismo host, pero diferentes orígenes de red CDN.<br>El formato del valor canónico no es importante, pero debe ser único para el host.<br>Puede ser más fácil establecer el valor para que coincida con el valor del host.<br><br>Ejemplos basados en los ejemplos de hosts personalizados anteriores:<br>Nombre = CACHEABLE_CUSTOM_1_CANONICAL valor = foopackages<br> Nombre = CACHEABLE_CUSTOM_2_CANONICAL valor = packages.bar.com  |
| IS_SUMMARY_PUBLIC             | Verdadero o falso                          | Opcional          | Permite ver el informe de resumen en la red local o en Internet.<br>Es necesario el uso de una clave de API (que se describe más adelante) para ver el informe de resumen si está establecido en true. |
| IS_SUMMARY_ACCESS_UNRESTRICTED| Verdadero o falso                          | Opcional          | Permite ver el informe de resumen en la red local o en Internet sin<br>usar la clave de API desde cualquier dispositivo de la red. Utilícelo si no desea bloquear el acceso<br>para ver los datos de resumen del servidor de caché mediante el explorador. |
            
## <a name="microsoft-connected-cache-azure-iot-edge-module-container-create-options"></a>Opciones de creación del contenedor del módulo Azure IoT Edge Caché conectada de Microsoft

Las opciones de creación del contenedor para la implementación del módulo de MCC proporcionan control de la configuración relacionada con el almacenamiento y los puertos utilizados por el módulo de MCC. Esta es la lista de variables creadas por el contenedor necesarias para implementar MCC.

### <a name="container-to-host-os-drive-mappings"></a>Asignaciones de unidad de SO del contenedor al host

Se requiere para asignar la ubicación de almacenamiento del contenedor a la ubicación de almacenamiento en el disco. Se pueden especificar hasta nueve ubicaciones.

>[!Note]
>El número de la unidad debe coincidir con los valores del enlace de la unidad de caché especificados en la variable de entorno STORAGE_ *N* _SIZE_GB valor, ```/MicrosoftConnectedCache*N*/:/nginx/cache*N*/```

### <a name="container-to-host-tcp-port-mappings"></a>Asignaciones de puertos TCP del contenedor al host

Esta opción especifica el puerto http de la máquina externa en el que MCC escucha las solicitudes de contenido. El valor predeterminado del puerto del host es el puerto 80 y no se admiten otros puertos en este momento, ya que actualmente el cliente de ADU realiza solicitudes en el puerto 80. El puerto TCP 8081 es el puerto interno del contenedor en el que escucha MCC y no se puede cambiar.

### <a name="container-service-tcp-port-mappings"></a>Asignaciones de puertos TCP del servicio de contenedor

El módulo de Caché conectada de Microsoft tiene un servicio de .NET Core que es utilizado por el motor de almacenamiento en caché para diversas funciones.

>[!Note]
>Para admitir el perímetro anidado de Azure IoT, el puerto del host no se debe establecer en 5000 porque el módulo de proxy del registro ya escucha en el puerto del host 5000.


Opciones de creación del contenedor de ejemplo

```json
{
    "HostConfig": {
        "Binds": [
            "/microsoftConnectedCache1/:/nginx/cache1/"
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

## <a name="microsoft-connected-cache-summary-report"></a>Informe de resumen de Caché conectada de Microsoft

El informe de resumen es actualmente la única manera de que un cliente vea los datos de almacenamiento en caché de las instancias de Caché conectada de Microsoft implementadas en puertas de enlace Azure IoT Edge. El informe se genera a intervalos de 15 segundos e incluye las estadísticas medias del período, así como estadísticas agregadas para la duración del módulo. Las estadísticas principales en las que los clientes estarán interesados son:

* **hitBytes**: es la suma de los bytes entregados que proceden directamente de la memoria caché.
* **missBytes**: es la suma de los bytes entregados que Caché conectada de Microsoft tuvo que descargar de la red CDN para ver la memoria caché.
* **eggressBytes**: es la suma de hitBytes y missBytes, y es el total de bytes entregados a los clientes.
* **hitRatioBytes**: es la relación entre hitBytes y egressBytes.  Por ejemplo, si el 100 % de eggressBytes entregado en un período fuese igual a hitBytes, esto sería 1.


El informe de resumen está disponible en `http://<FQDN/IP of Azure IoT Edge Gateway hosting MCC>:5001/summary`. Reemplace \<Azure IoT Edge Gateway IP\> por la dirección IP o el nombre de host de su puerta de enlace IoT Edge. (consulte los detalles de la variable de entorno para obtener información sobre la visibilidad de este informe).