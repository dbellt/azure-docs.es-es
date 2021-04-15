---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/09/2020
ms.author: trbye
ms.openlocfilehash: 8c77efe9e3e301573b032d1dc1dd32bb4a5ab1a1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "103417761"
---
El control del audio comprimido se implementa mediante [GStreamer](https://gstreamer.freedesktop.org). Por motivos de licencia, los binarios de GStreamer no se compilan ni se vinculan con el SDK de Voz. Los desarrolladores deben instalar varias dependencias y complementos; consulte [Instalación en Windows](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html?gi-language=c) o [Instalación en Linux](https://gstreamer.freedesktop.org/documentation/installing/on-linux.html?gi-language=c). Los archivos binarios de GStreamer deben estar en la ruta de acceso del sistema, con el fin de que el SDK de Voz pueda cargarlos en tiempo de ejecución. Por ejemplo, en Windows, si el SDK de voz es capaz de encontrar `libgstreamer-1.0-0.dll` o `gstreamer-1.0-0.dll` (en la versión más reciente de GStreamer) durante el tiempo de ejecución, significa que los archivos binarios de GStreamer se encuentran en la ruta de acceso del sistema.

