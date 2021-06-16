---
title: Configuración de Microsoft OPC Publisher
description: En este tutorial aprenderá a configurar OPC Publisher en modo independiente.
author: jehona-m
ms.author: jemorina
ms.service: industrial-iot
ms.topic: tutorial
ms.date: 3/22/2021
ms.openlocfilehash: a889f0b740d506bd546f1fb67f050317146c31e8
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110677898"
---
# <a name="tutorial-configure-the-opc-publisher"></a>Tutorial: Configuración de OPC Publisher

Este tutorial contiene información sobre la configuración de OPC Publisher. Se pueden usar varias interfaces para configurarlo.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configuración de OPC Publisher mediante un archivo de configuración
> * Configuración de OPC Publisher mediante argumentos de la línea de comandos
> * Configuración de OPC Publisher mediante métodos directos de IoT Hub
> * Configuración de OPC Publisher mediante el microservicio REST complementario basado en la nube

## <a name="configuring-security"></a>Configuración de seguridad

IoT Edge proporciona a OPC Publisher su configuración de seguridad para acceder a IoT Hub automáticamente. OPC Publisher también se puede ejecutar como un contenedor de Docker independiente mediante la especificación de una cadena de conexión de dispositivo para acceder a IoT Hub mediante el parámetro de la línea de comandos `dc`. Se puede crear un dispositivo para IoT Hub y recuperar su cadena de conexión mediante Azure Portal.

Para acceder a los recursos habilitados para OPC UA, OPC UA usa los certificados X.509 y sus claves privadas asociadas. Esto se llama autenticación de aplicaciones de OPC UA, además de la autenticación de usuario de OPC UA. OPC Publisher usa un almacén de certificados basado en el sistema de archivos para administrar todos los certificados de aplicación. Durante el inicio, OPC Publisher comprueba si hay un certificado que pueda usar en este almacén de certificados y crea un nuevo certificado autofirmado y una nueva clave privada asociada si no hay ninguno. Los certificados autofirmados proporcionan una autenticación débil, puesto que no están firmados por una entidad de certificación de confianza, pero al menos se puede cifrar de esta manera la comunicación con el recurso habilitado para OPC UA.

La seguridad se habilita en el archivo de configuración mediante la marca `"UseSecurity": true,`. Se selecciona automáticamente el punto de conexión más seguro disponible en los servidores de OPC UA a los que se supone que se va a conectar OPC Publisher.
De manera predeterminada, OPC Publisher usa autenticación anónima de usuarios (además de la autenticación de aplicación descrita anteriormente). Sin embargo, OPC Publisher también admite la autenticación de usuarios con el nombre de usuario y la contraseña. Esto se puede especificar mediante la interfaz de configuración de la API REST (descrita a continuación) o mediante el archivo de configuración como se indica a continuación:
```
"OpcAuthenticationMode": "UsernamePassword",
"OpcAuthenticationUsername": "usr",
"OpcAuthenticationPassword": "pwd",
```
Además, la versión 2.5 y anteriores de OPC Publisher cifran el nombre de usuario y la contraseña en el archivo de configuración. La versión 2.6 y versiones posteriores solo admiten el nombre de usuario y la contraseña en texto sin formato. Esto se mejorará en la próxima versión de OPC Publisher.

Para conservar la configuración de seguridad de OPC Publisher en los reinicios, el certificado y la clave privada que se encuentran en el directorio del almacén de certificados deben estar asignados al sistema de archivos del sistema operativo del host de IoT Edge. Consulte la sección anterior "Especificación de las opciones de creación de contenedor en Azure Portal".

## <a name="configuration-via-configuration-file"></a>Configuración mediante un archivo de configuración

La manera más sencilla de configurar OPC Publisher es mediante un archivo de configuración. En el archivo [`publishednodes.json`](https://raw.githubusercontent.com/Azure/iot-edge-opc-publisher/master/opcpublisher/publishednodes.json) de este repositorio se proporciona un archivo de configuración de ejemplo, así como documentación sobre su formato.
La sintaxis del archivo de configuración ha cambiado con el tiempo y OPC Publisher todavía puede leer los formatos antiguos, pero los convierte al formato más reciente al guardar la configuración, lo que se realiza periódicamente de forma automática.

Este es el aspecto de un archivo de configuración básico:
```
[
  {
    "EndpointUrl": "opc.tcp://testserver:62541/Quickstarts/ReferenceServer",
    "UseSecurity": true,
    "OpcNodes": [
      {
        "Id": "i=2258",
        "OpcSamplingInterval": 2000,
        "OpcPublishingInterval": 5000,
        "DisplayName": "Current time"
      }
    ]
  }
]
```

Los recursos de OPC UA optimizan el ancho de banda de red al enviar solo los cambios de datos a OPC Publisher cuando los datos han cambiado. Si los cambios en los datos se deben publicar con más frecuencia o a intervalos regulares, OPC Publisher admite un "latido" para todos los elementos de datos configurados que se pueden habilitar especificando además la clave HeartbeatInterval en la configuración del elemento de datos. El intervalo se especifica en segundos:
```
 "HeartbeatInterval": 3600,
```

Un recurso de OPC UA siempre envía el valor actual de un elemento de datos cuando OPC Publisher se conecta por primera vez a él. Para evitar que se publiquen estos datos en IoT Hub, se puede especificar adicionalmente la clave SkipFirst en la configuración del elemento de datos:
```
 "SkipFirst": true,
```

Ambos valores también se pueden habilitar globalmente mediante las opciones de la línea de comandos.

## <a name="configuration-via-command-line-arguments"></a>Configuración mediante argumentos de la línea de comandos

Hay varios argumentos de la línea de comandos que se pueden usar para establecer la configuración global para OPC Publisher. Se describen [aquí](reference-command-line-arguments.md).


## <a name="configuration-via-the-built-in-opc-ua-server-interface"></a>Configuración mediante la interfaz integrada del servidor de OPC UA

>[!NOTE] 
> Esta característica solo está disponible en la versión 2.5 y anteriores de OPC Publisher.

OPC Publisher tiene un servidor de OPC UA integrado que se ejecuta en el puerto 62222. Implementa tres métodos de OPC UA:

  - PublishNode
  - UnpublishNode
  - GetPublishedNodes

Se puede acceder a esta interfaz mediante una aplicación cliente de OPC UA, por ejemplo, [UA Expert](https://www.unified-automation.com/products/development-tools/uaexpert.html).

## <a name="configuration-via-iot-hub-direct-methods"></a>Configuración mediante métodos directos de IoT Hub

>[!NOTE] 
> Esta característica solo está disponible en la versión 2.5 y anteriores de OPC Publisher.

OPC Publisher implementa los siguientes [métodos directos de IoT Hub](../iot-hub/iot-hub-devguide-direct-methods.md), a los que se puede llamar desde una aplicación (desde cualquier lugar del mundo) con el [SDK de dispositivo de IoT Hub](../iot-hub/iot-hub-devguide-sdks.md):

  - PublishNodes
  - UnpublishNodes
  - UnpublishAllNodes
  - GetConfiguredEndpoints
  - GetConfiguredNodesOnEndpoint
  - GetDiagnosticInfo
  - GetDiagnosticLog
  - GetDiagnosticStartupLog
  - ExitApplication
  - GetInfo

Proporcionamos una [aplicación de configuración de ejemplo](https://github.com/Azure-Samples/iot-edge-opc-publisher-nodeconfiguration), así como una [aplicación para leer la información de diagnóstico](https://github.com/Azure-Samples/iot-edge-opc-publisher-diagnostics) de código abierto de OPC Publisher, que aprovecha esta interfaz.

## <a name="configuration-via-cloud-based-companion-rest-microservice"></a>Configuración mediante el microservicio REST complementario basado en la nube

>[!NOTE] 
> Esta característica solo está disponible en la versión 2.6 y posteriores de OPC Publisher.

[Aquí](https://github.com/Azure/Industrial-IoT/blob/master/docs/services/publisher.md) se describe y está disponible un microservicio complementario basado en la nube con una interfaz REST. Se puede usar para configurar OPC Publisher mediante una interfaz compatible con OpenAPI, por ejemplo, mediante Swagger.

## <a name="configuration-of-the-simple-json-telemetry-format-via-separate-configuration-file"></a>Configuración del formato de datos de telemetría JSON simple mediante un archivo de configuración independiente

>[!NOTE] 
> Esta característica solo está disponible en la versión 2.5 y anteriores de OPC Publisher.

OPC Publisher permite filtrar las partes del formato de datos de telemetría simple y no estandarizado mediante un archivo de configuración independiente, que se puede especificar con la opción de la línea de comandos tc. Si no se especifica ningún archivo de configuración, se envía a IoT Hub el formato completo de datos de telemetría JSON. El formato del archivo de configuración de datos de telemetría independiente se describe [aquí](reference-opc-publisher-telemetry-format.md#opc-publisher-telemetry-configuration-file-format).

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha configurado OPC Publisher, el siguiente paso es aprender a optimizar el rendimiento y la memoria del módulo Edge:

> [!div class="nextstepaction"]
> [Tutorial: Optimización del rendimiento y la memoria de OPC Publisher](tutorial-publisher-performance-memory-tuning-opc-publisher.md)