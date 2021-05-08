---
title: Información sobre la compatibilidad con la actualización de dispositivos desconectados mediante Caché conectada de Microsoft | Microsoft Docs
titleSuffix: Device Update for Azure IoT Hub
description: Información sobre la compatibilidad con la actualización de dispositivos desconectados mediante Caché conectada de Microsoft
author: andyriv
ms.author: andyriv
ms.date: 2/16/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: e216d42ff1f279d87e657126514fcfb50960f806
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107811911"
---
# <a name="understand-support-for-disconnected-device-updates"></a>Información sobre la compatibilidad con las actualizaciones de dispositivos desconectados

En un escenario de puerta de enlace transparente, uno o varios dispositivos pueden pasar sus mensajes a través de un dispositivo de puerta de enlace único que mantiene la conexión con Azure IoT Hub. En estos casos, es posible que los dispositivos secundarios no tengan conectividad a Internet o que no se les permita descargar contenido de Internet. La versión preliminar del módulo IoT Edge de Caché conectada de Microsoft proporcionará el servicio Device Update a los clientes de Azure IoT Hub con la funcionalidad de una memoria caché inteligente en la red, que permite actualizaciones basadas en imágenes y en paquetes de dispositivos basados en el sistema operativo Linux detrás de una puerta de enlace IoT Edge (dispositivos IoT de bajada), y también ayuda a ahorrar ancho de banda para los clientes de Device Update para Azure IoT Hub.

## <a name="how-does-microsoft-connected-cache-preview-for-device-update-for-azure-iot-hub-work"></a>¿Cómo funciona la versión preliminar de Caché conectada de Microsoft para Device Update para Azure IoT Hub?

La versión preliminar de la caché conectada de Microsoft es una memoria caché inteligente y transparente para el contenido publicado para Device Update para Azure IoT Hub y también se puede personalizar para almacenar en caché contenido de otros orígenes, como los repositorios de paquetes. Caché conectada de Microsoft es una memoria caché de uso poco frecuente que se activa con las solicitudes de cliente para los intervalos de archivos exactos solicitados por el cliente de Optimización de distribución y no realiza una inicialización previa del contenido. En el diagrama y la descripción paso a paso que se muestran a continuación se explica cómo funciona Caché conectada de Microsoft dentro de la infraestructura de Device Update para Azure IoT Hub.

>[!Note]
>En la definición de este flujo se supone que la puerta de enlace IoT Edge tiene conectividad a Internet. Para el escenario de puerta de enlace IoT Edge de bajada (perímetro anidado), la red de entrega de contenido (CDN) se puede considerar la MCC hospedada en la puerta de enlace IoT Edge primaria.

  :::image type="content" source="media/connected-cache-overview/disconnected-device-update.png" alt-text="Actualización de un dispositivo desconectado" lightbox="media/connected-cache-overview/disconnected-device-update.png":::

1. Caché conectada de Microsoft se implementa como un módulo IoT Edge en el servidor local.
2. Los clientes de Device Update para Azure IoT Hub están configurados para descargar contenido de Caché conectada de Microsoft en virtud del atributo GatewayHostName de la cadena de conexión del dispositivo para dispositivos de nodo hoja IoT **o** cuando se establece parent_hostname en el archivo config.toml para los dispositivos IoT Edge secundarios.
3. Los clientes de Device Update para Azure IoT Hub reciben en ambos casos los comandos de descarga del contenido de la actualización desde el servicio Device Update para Azure IoT Hub y solicitan el contenido de la actualización a Caché conectada de Microsoft en lugar de a la red CDN. De manera predeterminada, Caché conectada de Microsoft está configurada para escuchar en el puerto http 80, y el cliente de Optimización de distribución realiza la solicitud de contenido en el puerto 80, por lo que el elemento primario debe estar configurado para escuchar en este puerto.  En este momento solo se admite el protocolo http.
4. El servidor de Caché conectada de Microsoft descarga el contenido de la red CDN, inicializa su memoria caché local almacenada en el disco y entrega el contenido al cliente de Device Update para Azure IoT Hub.
   
>[!Note]
>Al utilizar actualizaciones basadas en paquetes, el administrador configurará el servidor de Caché conectada de Microsoft con el nombre de host del paquete necesario.

5. Las solicitudes posteriores de otros clientes de Device Update para Azure IoT Hub para el mismo contenido de actualización procederán de la memoria caché y Caché conectada de Microsoft no realizará solicitudes a la red CDN para el mismo contenido.

### <a name="supporting-industrial-iot-iiot-with-parentchild-hosting-scenarios"></a>Compatibilidad con IoT industrial (IIoT) con escenarios de hospedaje de elementos primarios y secundarios

Cuando una puerta de enlace IoT Edge de bajada o secundaria hospeda el servidor de Caché conectada de Microsoft, se configurará para solicitar el contenido de actualización desde la puerta de enlace IoT Edge primaria, que hospeda el servidor de Caché conectada de Microsoft. Esto es obligatorio para tantos niveles como sea necesario antes de llegar a la puerta de enlace IoT Edge primaria que hospeda un servidor de Caché conectada de Microsoft que tenga acceso a Internet. Desde el servidor conectado a Internet se solicita el contenido desde la red CDN, momento en el que el contenido se devuelve a la puerta de enlace IoT Edge secundaria que solicitó originalmente el contenido. El contenido se almacenará en el disco en cada nivel.

## <a name="access-to-the-microsoft-connected-cache-preview-for-device-update-for-azure-iot-hub"></a>Acceso a la versión preliminar de Caché conectada de Microsoft para Device Update para Azure IoT Hub

El módulo IoT Edge de Caché conectada de Microsoft se ha publicado como una versión preliminar para los clientes que implementan soluciones con Device Update para Azure IoT Hub. El acceso a la versión preliminar es por invitación. [Solicite acceso](https://aka.ms/MCCForDeviceUpdateForIoT) a la versión preliminar de Caché conectada de Microsoft para Device Update para IoT Hub y proporcione la información solicitada si desea acceder al módulo.
