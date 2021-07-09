---
title: Implementación de Microsoft OPC Publisher
description: En este tutorial aprenderá a implementar OPC Publisher en modo independiente.
author: jehona-m
ms.author: jemorina
ms.service: industrial-iot
ms.topic: tutorial
ms.date: 3/22/2021
ms.openlocfilehash: 80d4ef5fc474ff40807e01b6ecb9014db1ae8f51
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110677880"
---
# <a name="tutorial-deploy-the-opc-publisher"></a>Tutorial: Implementación de OPC Publisher

OPC Publisher es un producto de Microsoft totalmente compatible, desarrollado con código abierto, que cierra la brecha entre los recursos industriales y la nube de Microsoft Azure. Para ello, se conecta a recursos habilitados para OPC UA o a software de conectividad industrial, y publica datos de telemetría en [Azure IoT Hub](https://azure.microsoft.com/services/iot-hub/) en distintos formatos, incluido el formato estándar IEC62541 OPC UA PubSub (a partir de la versión 2.6).

Se ejecuta en [Azure IoT Edge](https://azure.microsoft.com/services/iot-edge/) como un módulo o en Docker sin formato como contenedor. Dado que aprovecha el [runtime multiplataforma de .NET](/dotnet/core/introduction), también se ejecuta de forma nativa en Linux y Windows 10.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Implementar OPC Publisher
> * Ejecutar la versión lanzada más recientemente de OPC Publisher como si fuera un contenedor
> * Especifique las opciones de creación de contenedor en Azure Portal

Si no tiene ninguna suscripción a Azure, cree una cuenta de evaluación gratuita.

## <a name="prerequisites"></a>Prerrequisitos

- Se debe crear una instancia de IoT Hub
- Se debe crear un dispositivo IoT Edge
- Se debe registrar un dispositivo IoT Edge

## <a name="deploy-the-opc-publisher-from-the-azure-marketplace"></a>Implementación de OPC Publisher desde Azure Marketplace

1. Elija la suscripción de Azure que va a usar. Si no hay ninguna suscripción de Azure disponible, debe crearla.
2. Elija la instancia de IoT Hub al que se supone que OPC Publisher va a enviar datos. Si no hay ninguna instancia de IoT Hub disponible, debe crearla.
3. Elija el dispositivo IoT Edge en el que se supone que OPC Publisher se va a ejecutar (o escriba un nombre para el nuevo dispositivo IoT Edge que se va a crear).
4. Haga clic en Crear. Se abre la página "Set modules on Device" (Establecer módulos en dispositivo) para el dispositivo IoT Edge seleccionado.
5. Haga clic en "OPCPublisher" para abrir la página "Actualizar módulo IoT Edge" de OPC Publisher y seleccione "Opciones de creación del contenedor".
6. Especifique otras opciones de creación del contenedor en función del uso de OPC Publisher, consulte la siguiente sección.

Todas las imágenes de Docker compatibles con OPC Publisher de Docker se enumeran [aquí](https://mcr.microsoft.com/v2/iotedge/opc-publisher/tags/list). En el caso de los recursos que no son de OPC UA, nos hemos asociado con los principales proveedores de conectividad industrial y les hemos ayudado a portar el software del adaptador OPC UA a Azure IoT Edge. Estos adaptadores están disponibles en Azure [Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps?page=1).

## <a name="specifying-container-create-options-in-the-azure-portal"></a>Especificación de las opciones de creación de contenedor en Azure Portal
Al implementar OPC Publisher desde Azure Portal, se pueden especificar las opciones de creación del contenedor en la página Actualizar módulo IoT Edge de OPC Publisher. Estas opciones de creación deben estar en formato JSON. Los argumentos de la línea de comandos de OPC Publisher se pueden especificar mediante la clave Cmd, por ejemplo:
```
"Cmd": [
    "--pf=./pn.json",
    "--aa"
],
```

Un conjunto típico de las opciones de creación del contenedor de módulos IoT Edge para OPC Publisher es:
```
{
    "Hostname": "opcpublisher",
    "Cmd": [
        "--pf=./pn.json",
        "--aa"
    ],
    "HostConfig": {
        "Binds": [
            "/iiotedge:/appdata"
        ]
    }
}
```

Con estas opciones especificadas, OPC Publisher leerá el archivo de configuración `./pn.json`. El directorio de trabajo de OPC Publisher se establece en `/appdata` en el inicio y, por consiguiente, OPC Publisher leerá el archivo `/appdata/pn.json` dentro de su contenedor de Docker. El archivo de registro de OPC Publisher se escribirá en `/appdata` y el directorio `CertificateStores` (que se usa para los certificados de OPC UA) también se creará en este directorio. Para que estos archivos estén disponibles en el sistema de archivos del host IoT Edge, la configuración del contenedor requiere un volumen de montaje de enlace. El enlace `/iiotedge:/appdata` asignará el directorio `/appdata` al directorio del host `/iiotedge` (que creará el runtime de IoT Edge en tiempo de ejecución si no existe).
**Sin este volumen de montaje de enlace, todos los archivos de configuración de OPC Publisher se perderán al reiniciarse el contenedor.**

Para lograr una conexión a un servidor de OPC UA que use su nombre de host sin un servidor DNS configurado en la red, agregue una entrada `ExtraHosts` a la sección `HostConfig`:

```
"HostConfig": {
    "ExtraHosts": [
        "opctestsvr:192.168.178.26"
    ]
}
```

## <a name="next-steps"></a>Pasos siguientes 
Ahora que ha implementado el módulo OPC Publisher Edge, el siguiente paso es configurarlo:

> [!div class="nextstepaction"]
> [Configuración de OPC Publisher](tutorial-publisher-configure-opc-publisher.md)