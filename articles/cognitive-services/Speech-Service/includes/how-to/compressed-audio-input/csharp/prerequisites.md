---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/09/2020
ms.author: trbye
ms.openlocfilehash: 2be7255d753cef729266e42df49c6b3eaff73e7a
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111854507"
---
El control del audio comprimido se implementa mediante [GStreamer](https://gstreamer.freedesktop.org). Por motivos de licencia, los binarios de GStreamer no se compilan ni se vinculan con el SDK de Voz. Los desarrolladores deben instalar varias dependencias y complementos; consulte [Instalación en Windows](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html?gi-language=c) o [Instalación en Linux](https://gstreamer.freedesktop.org/documentation/installing/on-linux.html?gi-language=c). Los archivos binarios de GStreamer deben estar en la ruta de acceso del sistema, con el fin de que el SDK de Voz pueda cargarlos en tiempo de ejecución. Por ejemplo, en Windows, si el SDK de Voz es capaz de encontrar `libgstreamer-1.0-0.dll` o `gstreamer-1.0-0.dll` (en la versión más reciente de GStreamer) durante el tiempo de ejecución, significa que los archivos binarios de GStreamer se encuentran en la ruta de acceso del sistema.

