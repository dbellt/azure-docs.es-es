---
title: Descripción del agente de Device Update para Azure IoT Hub | Microsoft Docs
description: Descripción del agente de Device Update para Azure IoT Hub.
author: ValOlson
ms.author: valls
ms.date: 2/12/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: 0d97287657b1e1fe7d540e8811c90794aaa5fece
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739573"
---
# <a name="device-update-for-iot-hub-agent-overview"></a>Introducción al agente de Device Update para IoT Hub

El agente de Device Update consta de dos niveles conceptuales:

* El nivel de interfaz se basa en [Azure IoT Plug and Play (PNP)](../iot-pnp/overview-iot-plug-and-play.md), lo que permite que la mensajería fluya entre el agente de Device Update y los servicios de Device Update.
* El nivel de plataforma es responsable de las acciones de actualización Descargar, Instalar y Aplicar de alto nivel, que pueden ser específicas de la plataforma o del dispositivo.

:::image type="content" source="media/understand-device-update/client-agent-reference-implementations.png" alt-text="Implementaciones del agente." lightbox="media/understand-device-update/client-agent-reference-implementations.png":::

## <a name="the-interface-layer"></a>El nivel de interfaz

El nivel de interfaz se compone de las interfaces [ADU Core](https://github.com/Azure/iot-hub-device-update/tree/main/src/agent/adu_core_interface) y [Device Information](https://github.com/Azure/iot-hub-device-update/tree/main/src/agent/device_info_interface).

Estas interfaces dependen de un archivo de configuración para los valores predeterminados. Los valores predeterminados incluyen aduc_manufacturer y aduc_model para la interfaz AzureDeviceUpdateCore, así como los de model y manufacturer para la interfaz DeviceInformation. [Obtenga más información](device-update-configuration-file.md) sobre el archivo de configuración.

### <a name="adu-core-interface"></a>Interfaz ADU Core

La interfaz "ADU Core" es el canal de comunicación principal entre los servicios y el agente de Device Update. [Obtenga más información](device-update-plug-and-play.md#adu-core-interface) acerca de esta interfaz.

### <a name="device-information-interface"></a>Interfaz Device Information

La interfaz Device Information se usa para implementar la interfaz `Azure IoT PnP DeviceInformation`. [Obtenga más información](device-update-plug-and-play.md#device-information-interface) acerca de esta interfaz.

## <a name="the-platform-layer"></a>El nivel de plataforma

Existen dos implementaciones del nivel de plataforma. El nivel de plataforma del simulador tiene una implementación trivial de las acciones de actualización, y se usa para probar y evaluar rápidamente los servicios y la configuración de Device Update para IoT Hub. Cuando el agente de Device Update se compila con el nivel de plataforma del simulador, se hace referencia a este como el agente del simulador de Device Update o simplemente como el simulador. [Obtenga más información](https://github.com/Azure/iot-hub-device-update/blob/main/docs/agent-reference/how-to-run-agent.md) sobre cómo usar el agente del simulador. El nivel de plataforma de Linux se integra con el servicio [Optimización de distribución](https://github.com/microsoft/do-client) de las descargas y se usa en nuestra imagen de referencia de Raspberry Pi y en todos los clientes que se ejecutan en sistemas Linux.

### <a name="simulator-platform-layer"></a>Nivel de plataforma del simulador

La implementación del nivel de plataforma del simulador se puede encontrar en `src/platform_layers/simulator_platform_layer` y se puede usar para probar y evaluar Device Update para IoT Hub.  Muchas de las acciones de la implementación del "simulador" se simulan para reducir los cambios físicos a fin de experimentar con Device Update para IoT Hub.  Se puede realizar una actualización "simulada"completa con este nivel de plataforma. [Obtenga más información](https://github.com/Azure/iot-hub-device-update/blob/main/docs/agent-reference/how-to-run-agent.md) sobre la ejecución del agente del simulador.

### <a name="linux-platform-layer"></a>Nivel de plataforma de Linux

La implementación del nivel de plataforma de Linux se puede encontrar en `src/platform_layers/linux_platform_layer`, se integra con el [cliente de Optimización de distribución](https://github.com/microsoft/do-client/releases) para las descargas y se usa en nuestra imagen de referencia de Raspberry Pi y en todos los clientes que se ejecutan en sistemas Linux.

Este nivel se puede integrar con distintos controladores de actualización para implementar el instalador. Por ejemplo, el controlador de actualización `SWUpdate` invoca un script de shell para llamar al ejecutable `SWUpdate` y realizar una actualización.

## <a name="update-handlers"></a>Controladores de actualización

Los controladores de actualización son componentes que controlan contenido o partes específicas del instalador de la actualización. Las implementaciones del controlador de actualización se encuentran en `src/content_handlers`.

### <a name="simulator-update-handler"></a>Controlador de actualización del simulador

El controlador de actualización del simulador se usa en el nivel de plataforma del simulador y se puede utilizar con el nivel de plataforma de Linux para falsificar las interacciones con un controlador de contenido. El controlador de actualización del simulador implementa las API del controlador de actualización prácticamente sin operaciones. La implementación del controlador de actualización del simulador se puede encontrar a continuación:
* [Simulador de actualización de imagen](https://github.com/Azure/iot-hub-device-update/blob/main/src/content_handlers/swupdate_handler/inc/aduc/swupdate_simulator_handler.hpp)
* [Simulador APT de actualización del paquete](https://github.com/Azure/iot-hub-device-update/blob/main/src/content_handlers/apt_handler/inc/aduc/apt_simulator_handler.hpp)

Nota: El campo InstalledCriteria de la interfaz PnP de AzureDeviceUpdateCore debe ser el hash SHA256 del contenido. Este es el mismo hash que se encuentra en el [objeto de manifiesto de importación](import-update.md#create-a-device-update-import-manifest). [Obtenga más información](device-update-plug-and-play.md) sobre `installedCriteria` y la interfaz `AzureDeviceUpdateCore`.

### <a name="swupdate-update-handler"></a>Controlador de actualización de `SWUpdate`

El controlador de actualización de `SWUpdate` se integra con el ejecutable de línea de comandos `SWUpdate` y otros comandos de shell para implementar las actualizaciones de A/B específicamente para la imagen de referencia de Raspberry Pi. Busque la imagen de referencia de Raspberry Pi más reciente [aquí](https://github.com/Azure/iot-hub-device-update/releases). El controlador de actualización de `SWUpdate` se implementa en [src/content_handlers/swupdate_content_handler](https://github.com/Azure/iot-hub-device-update/tree/main/src/content_handlers/swupdate_handler).

### <a name="apt-update-handler"></a>APT Update Handler

APT Update Handler procesa un manifiesto de actualización específico de APT e invoca APT para instalar o actualizar los paquetes de Debian especificados.

## <a name="self-update-device-update-agent"></a>Agente de Device Update de actualización automática

El agente de Device Update y sus dependencias se pueden actualizar a través de la canalización de Device Update para IoT Hub. Si usa una actualización basada en imágenes, incluya el agente de Device Update más reciente en la nueva imagen. Si usa una actualización basada en paquetes, incluya el agente de Device Update y su versión deseada en el manifiesto APT como cualquier otro paquete. [Obtenga más información](device-update-apt-manifest.md) sobre el manifiesto APT. Puede comprobar la versión instalada del agente de Device Update y el agente de Optimización de distribución en la sección Propiedades del dispositivo de su [dispositivo gemelo de IoT](../iot-hub/iot-hub-devguide-device-twins.md). [Obtenga más información sobre las propiedades del dispositivo en la interfaz ADU Core](device-update-plug-and-play.md#device-properties).

## <a name="next-steps"></a>Pasos siguientes
[Descripción del archivo de configuración del agente de Device Update](device-update-configuration-file.md)