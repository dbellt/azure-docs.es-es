---
title: 'Reproducción de grabaciones de vídeo: Azure Video Analyzer'
description: Puede usar Azure Video Analyzer para la grabación continua de vídeo, lo que permite grabar vídeo en la nube durante semanas o meses. También puede limitar la grabación a clips de interés mediante la grabación basada en eventos. En este artículo se describe cómo reproducir las grabaciones.
ms.service: azure-video-analyzer
ms.topic: how-to
ms.date: 05/13/2021
ms.openlocfilehash: c2b8ceea0778634f3396ac324da093c030f5acc9
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388651"
---
# <a name="playback-of-video-recordings"></a>Reproducción de grabaciones de vídeo 

## <a name="pre-read"></a>Lectura previa  

* [Grabación de vídeo continua](continuous-video-recording.md)
* [Grabación de vídeo basada en eventos](event-based-video-recording-concept.md)

## <a name="background"></a>Información previa  

Puede usar Azure Video Analyzer para la [grabación continua de vídeo](continuous-video-recording.md) (CVR), lo que permite grabar vídeo en la nube durante semanas o meses. También puede limitar la grabación a clips de interés mediante la [grabación de vídeo basada en eventos](event-based-video-recording-concept.md) (EVR). En ambos casos, si la [canalización](pipeline.md) usa inteligencia artificial para generar resultados de inferencia, debería [grabar estos resultados junto con el vídeo](record-stream-inference-data-with-video.md). 

Si va a evaluar las funcionalidades de Video Analyzer, debe seguir el [tutorial sobre CVR](use-continuous-video-recording.md) o el [tutorial sobre reproducción de vídeo con metadatos de inferencia](record-stream-inference-data-with-video.md), con la que puede reproducir las grabaciones mediante Azure Portal.

Si va a crear una aplicación o un servicio mediante las API de Video Analyzer, debe revisar lo siguiente para comprender cómo puede reproducir las grabaciones, además de revisar el artículo sobre las [directivas de acceso](access-policies.md) y sobre el [widget del reproductor de Video Analyzer](player-widget.md).

<!-- TODO - add a section here about 1P/3P SaaS and how to use widgets to allow end users to view videos without talking to ARM APIs -->

## <a name="determining-that-a-video-recording-is-ready-for-playback"></a>Determinación de que una grabación de vídeo está lista para la reproducción

La cuenta de Video Analyzer está vinculada a una cuenta de Azure Storage y, al grabar vídeo en la nube, el contenido se escribe en un [recurso de vídeo](terminology.md#video). Puede [transmitir el contenido](terminology.md#streaming) al finalizar la grabación o mientras está en curso. Esto se indica mediante la [marca](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/videoanalyzer/resource-manager/Microsoft.Media/preview/2021-05-01-preview/Videos.json) `canStream`, que se establecerá en `true` para el recurso de vídeo. 

## <a name="video-analyzer-player-widget"></a>Widget del reproductor de Video Analyzer
Video Analyzer le ofrece las funcionalidades necesarias para entregar secuencias mediante los protocolos HLS o MPEG-DASH a los dispositivos de reproducción (clientes). Puede utilizar el widget del reproductor de Video Analyzer para obtener la dirección URL de streaming y el token de autorización de reproducción, y utilizarlos en las aplicaciones cliente para reproducir el vídeo y los metadatos de inferencia.

Puede instalar el widget del reproductor de Video Analyzer para reproducir grabaciones de vídeo. El widget se puede instalar mediante `npm` o `yarn`, y esto le permitirá incluirlo en su propia aplicación del lado cliente. Ejecute uno de los siguientes comandos para incluir el widget en su propia aplicación:

NPM:
```
npm install –-save @azure/video-analyzer-widgets
```
YARN:
```
yarn add @azure/video-analyzer-widgets 
```
También puede insertar un script previo a la compilación existente; para ello, agregue type="module" al elemento del script que hace referencia a la ubicación previa a la compilación mediante el ejemplo siguiente:

```
<script async type="module" src="https://unpkg.com/@azure/video-analyzer-widgets"></script> 
``` 

## <a name="recording-and-playback-latencies"></a>Latencias de grabación y reproducción

Al usar Video Analyzer para grabar en un recurso de vídeo, especifique la [propiedad `segmentLength`](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/videoanalyzer/data-plane/VideoAnalyzer.Edge/preview/1.0.0/AzureVideoAnalyzer.json), que indica al módulo que agregue una duración mínima de vídeo (en segundos) antes de que se escriba en la nube. Por ejemplo, si `segmentLength` se establece en 300, el módulo acumulará 5 minutos de vídeo antes de cargar un "fragmento" de 5 minutos y, a continuación, entrará en el modo de acumulación durante los 5 minutos siguientes y volverá a cargarlos. Aumentar el valor de `segmentLength` tiene la ventaja de reducir los costos de las transacciones de Azure Storage, ya que el número de lecturas y escrituras no tendrá una frecuencia superior a una vez cada `segmentLength` segundos.

Por lo tanto, el streaming del vídeo de la cuenta de Video Analyzer se retrasará al menos durante ese tiempo. 

Otro factor que determina la latencia de reproducción (el retraso entre el momento en que se produce un evento delante de la cámara hasta el momento en que se puede ver en un dispositivo de reproducción) es la duración del grupo de imágenes ([GOP](https://en.wikipedia.org/wiki/Group_of_pictures)). Tal como se describe en [Reducción del retraso del streaming en vivo mediante tres sencillas técnicas](https://medium.com/vrt-digital-studio/reducing-the-delay-of-live-streams-by-using-3-simple-techniques-e8e028b0a641), cuanto mayor sea la duración del GOP, mayor será la latencia. Es habitual que las cámaras IP que se usan en escenarios de vigilancia y seguridad se configuren con un valor de GOP de más de 30 segundos. Esto tiene un gran impacto sobre la latencia general.

## <a name="next-steps"></a>Pasos siguientes

[Tutorial de grabación continua de vídeo](use-continuous-video-recording.md)
