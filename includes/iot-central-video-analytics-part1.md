---
title: archivo de inclusión
description: archivo de inclusión
services: iot-central
author: dominicbetts
ms.service: iot-central
ms.topic: include
ms.date: 10/06/2020
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 2f3e4bf640b8da31a7fa4d818b94b0372d3026b8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96763438"
---
La aplicación de ejemplo incluye dos dispositivos simulados y una puerta de enlace IoT Edge. En los siguientes tutoriales se muestran dos enfoques para experimentar y comprender las funcionalidades de la puerta de enlace:

* Creación de una puerta de enlace IoT Edge en una máquina virtual de Azure y conexión de una cámara simulada.
* Creación de una puerta de enlace IoT Edge en un dispositivo real, como un Intel NUC y conexión de una cámara real.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Usar la plantilla de aplicación análisis de vídeo de Azure IoT Central para crear una aplicación de comercio minorista
> * Personalizar la configuración de la aplicación
> * Crear una plantilla de dispositivo para un dispositivo de puerta de enlace IoT Edge
> * Agregar un dispositivo de puerta de enlace a la aplicación de IoT Central

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta serie de tutoriales, necesitará lo siguiente:

* Suscripción a Azure. Si no tiene una suscripción de Azure, puede crear una en la [página de suscripción a Azure](https://aka.ms/createazuresubscription).
* Si usa una cámara real, necesita conectividad entre el dispositivo IoT Edge y la cámara y necesita el canal **Protocolo de streaming en tiempo real**.

## <a name="initial-setup"></a>Instalación inicial

En estos tutoriales, se actualizan y se usan varios archivos de configuración. Las versiones iniciales de estos archivos están disponibles en el repositorio de GitHub [LVA-gateway](https://github.com/Azure/live-video-analytics/tree/master/ref-apps/lva-edge-iot-central-gateway). El repositorio incluye un archivo de texto para [anotaciones rápidas](https://github.com/Azure/live-video-analytics/blob/master/ref-apps/lva-edge-iot-central-gateway/setup/Scratchpad.txt) que puede descargar y usar para registrar los valores de configuración de los servicios que implemente. Este archivo le ayuda a completar los pasos posteriores de los tutoriales.

Cree una carpeta llamada *lva-configuration* en el equipo local para guardar copias de estos archivos. A continuación, haga clic con el botón derecho en cada uno de los vínculos siguientes y elija **Guardar como** para guardar el archivo en la carpeta *lva-configuration*:
