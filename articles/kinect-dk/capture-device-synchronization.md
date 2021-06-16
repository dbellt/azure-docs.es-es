---
title: Captura de la sincronización de dispositivos Azure Kinect
description: Aprenda a sincronizar los dispositivos de captura Azure Kinect mediante el SDK del sensor Azure Kinect.
author: martinekuan
ms.author: martinek
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, azure, sensor, sdk, profundidad, rgb, interna, externa, sincronización, cadena margarita, desplazamiento de fase
ms.openlocfilehash: 2dc7408751fa7a7c721bf50ca1c0f6103b691e5a
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111747102"
---
# <a name="capture-azure-kinect-device-synchronization"></a>Captura de la sincronización de dispositivos Azure Kinect

El hardware de Azure Kinect puede alinear el tiempo de captura de las imágenes de color y de profundidad. La alineación entre las cámaras del mismo dispositivo se denomina **sincronización interna**. La alineación del tiempo de captura de varios dispositivos conectados se denomina **sincronización externa**. La matriz de micrófonos funciona independientemente de las cámaras de color y de profundidad.

## <a name="device-internal-synchronization"></a>Sincronización interna del dispositivo

La captura de imágenes entre las cámaras individuales se sincroniza en el hardware. En cada [k4a_capture_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__capture__t.html) que contiene imágenes del sensor de color y de profundidad, las marcas de tiempo de las imágenes se alinean en función del modo de funcionamiento del hardware. De forma predeterminada, las imágenes de una captura son centros de exposición alineados. El tiempo relativo de las capturas de profundidad y de color se puede ajustar mediante el campo `depth_delay_off_color_usec` de [k4a_device_configuration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html).

## <a name="device-external-synchronization"></a>Sincronización externa de dispositivos

Vea [Configuración de sincronización externa](https://support.microsoft.com/help/4494429/sync-multiple-azure-kinect-dk-devices) para obtener la configuración del hardware.

El software de cada dispositivo conectado debe configurarse para funcionar en un modo **maestro** o **subordinado**. Esta configuración se establece en [k4a_device_configuration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html).

Cuando se usa sincronización externa, las cámaras subordinadas siempre deben iniciarse antes que la maestra para que las marcas de tiempo se alineen correctamente.

### <a name="subordinate-mode"></a>Modo subordinado

```C
k4a_device_configuration_t deviceConfig;
deviceConfig.wired_sync_mode = K4A_WIRED_SYNC_MODE_SUBORDINATE
```

### <a name="master-mode"></a>Modo maestro

```C
k4a_device_configuration_t deviceConfig;
deviceConfig.wired_sync_mode = K4A_WIRED_SYNC_MODE_MASTER;
```

### <a name="retrieving-synchronization-jack-state"></a>Recuperación del estado del conector de sincronización

Para recuperar mediante programación el estado actual de los conectores de entrada y salida de sincronización, use la función [k4a_device_get_sync_jack](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga0209ac87bfd055163677321b0304e962.html#ga0209ac87bfd055163677321b0304e962).

## <a name="next-steps"></a>Pasos siguientes

Ya sabe cómo habilitar y capturar la sincronización de dispositivos. También puede ver cómo se usa 

>[!div class="nextstepaction"]
>[API de grabación y reproducción del SDK del sensor Azure Kinect](record-playback-api.md)
