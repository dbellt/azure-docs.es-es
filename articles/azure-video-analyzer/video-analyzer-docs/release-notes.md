---
title: 'Notas de la versión de Azure Video Analyzer: Azure'
description: En este tema encontrará las notas de la versión de las mejoras, las versiones, las correcciones de errores y los problemas conocidos de Azure Video Analyzer.
ms.topic: conceptual
ms.date: 05/25/2021
ms.openlocfilehash: ef1c395bb10000599f901e62cf77b3573146aeee
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111591231"
---
# <a name="azure-video-analyzer-release-notes"></a>Notas de la versión de Azure Video Analyzer

>Reciba notificaciones para volver a visitar esta página y obtener actualizaciones; para ello, copie y pegue esta URL (`https://docs.microsoft.com/api/search/rss?search=%22Azure+Video+Analyzer+on+IoT+Edge+release+notes%22&locale=en-us`) en el lector de fuentes RSS.

En este artículo se proporciona información acerca de lo siguiente:

* Versiones más recientes
* Problemas conocidos
* Corrección de errores
* Funciones obsoletas

<hr width=100%>

## <a name="june-3-2021"></a>3 de junio de 2021

La etiqueta de versión de la actualización de julio de 2021 del módulo es la siguiente:

```
     mcr.microsoft.com/media/video-analyzer:1.0.1
```
> [!NOTE]
> En los artículos de inicio rápido y tutoriales, los manifiestos de implementación usan una etiqueta 1 (video-analyzer:1). Por lo tanto, con tan solo reimplementar tales manifiestos se debería actualizar el módulo en los dispositivos perimetrales cuando se publiquen etiquetas más nuevas.

### <a name="module-updates"></a>Actualizaciones del módulo
* Admite caracteres Unicode en las credenciales para conectarse a una cámara RTSP
* Actualizaciones para habilitar registros detallados en modo de depuración

<hr width=100%>

## <a name="may-25-2021"></a>25 de mayo de 2021

Esta versión es la primera versión preliminar pública de Azure Video Analyzer. La etiqueta de versión es la siguiente:

```
mcr.microsoft.com/media/video-analyzer:1.0.0
```

> [!NOTE]
> En los artículos de inicio rápido y tutoriales, los manifiestos de implementación usan una etiqueta 1 (video-analyzer:1). Por lo tanto, con tan solo reimplementar tales manifiestos se debería actualizar el módulo en los dispositivos perimetrales cuando se publiquen etiquetas más nuevas.

### <a name="supported-functionalities"></a>Funcionalidades admitidas

* Analice secuencias de vídeo en directo con los módulos de inteligencia artificial que prefiera y, de manera opcional, grabe vídeos en su dispositivo perimetral o en la nube.
* Grabación de vídeo junto con metadatos de inferencia en la nube
* Desencadenamiento de eventos cuando los objetos cruzan una línea con su propio modelo de detección de objetos
* Seguimiento de los objetos detectados por su propio modelo de detección 
* Uso de widgets (componentes web) del reproductor de Video Analyzer para reproducir metadatos de inferencia y vídeo grabado
* Implemente y configure el módulo a través de IoT Hub mediante Azure Portal o Visual Studio Code.
* Administración de [topologías de canalización](pipeline.md#pipeline-topologies) de forma remota o local mediante llamadas a [métodos directos](direct-methods.md)

## <a name="next-steps"></a>Pasos siguientes

[Información general](overview.md)
