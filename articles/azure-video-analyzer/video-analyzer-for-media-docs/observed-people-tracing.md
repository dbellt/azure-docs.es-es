---
title: Seguimiento de las personas observadas en un vídeo
titleSuffix: Azure Media Services
description: En este tema se proporciona información general sobre el seguimiento de las personas observadas en un vídeo de prueba.
services: media-services
author: Juliako
manager: femila
ms.service: azure-video-analyzer
ms.topic: article
ms.date: 04/30/2021
ms.author: juliako
ms.openlocfilehash: c1e4ca3dc15bb7d390d8525d1819fe9efa3e4ba8
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388588"
---
# <a name="trace-observed-people-in-a-video"></a>Seguimiento de las personas observadas en un vídeo

Video Indexer detecta a las personas observadas en los vídeos y proporciona información como la ubicación de la persona en el fotograma de vídeo y la marca de tiempo exacta (inicio, fin) en la que aparece una persona. La API devuelve las coordenadas del cuadro de límite (en píxeles) para cada instancia de persona detectada, incluido el grado de confianza de la detección.  
 
Estos son algunos escenarios en los que esta característica podría ser útil:

* Análisis posterior al evento: detecte y realice un seguimiento del movimiento de una persona para analizar mejor un accidente o un evento delictivo después de producirse (por ejemplo, una explosión, el robo de un banco o un incidente).  
* Mejore la eficacia al crear datos sin procesar para creadores de contenido como publicidad en vídeo, noticias o juegos deportivos (por ejemplo, buscar personas con una camiseta roja en un archivo de vídeo).
* Cree un resumen de un vídeo largo, como una prueba judicial de la aparición de una persona específica en un vídeo, mediante el mismo identificador de la persona detectada.
* Investigue y analice tendencias a lo largo del tiempo, por ejemplo, cómo se mueven los clientes por los pasillos de un centro comercial o cuánto tiempo invierten para pagar en las cajas.

Por ejemplo, si un vídeo contiene una persona, la operación de detección mostrará las apariciones de esa persona junto con sus coordenadas en los fotogramas de vídeo. Puede usar esta funcionalidad para determinar la ruta de una persona en un vídeo. También le permite saber si hay varias instancias de la misma persona en un vídeo.

La característica de **seguimiento de personas observadas** recién agregada está disponible al indexar el archivo si elige el valor preestablecido **Opción avanzada** -> **Advanced video** (Vídeo avanzado) o **Advanced video + audio** (Vídeo y audio avanzado) en **Indexación de audio y vídeo**. La indexación estándar no incluirá este nuevo modelo avanzado. 


:::image type="content" source="./media/observed-people-tracing/youtube-trailer.png" alt-text="Captura de pantalla del seguimiento de personas observadas":::  
 
Cuando decida ver **información detallada** del vídeo en el sitio web de [Video Indexer](https://www.videoindexer.ai/account/login), el seguimiento de personas observadas se mostrará en la página con todas las miniaturas de las personas detectadas. Puede elegir una miniatura de una persona y ver dónde aparece la persona en el reproductor de vídeo. 

La siguiente respuesta JSON muestra lo que Video Indexer devuelve al realizar el seguimiento de las personas observadas: 

```json
    {
    ...
    "videos": [
        {
            ...
            "insights": {
                ...
                "observedPeople": [{
                    "id": 1,
                    "thumbnailId": "560f2cfb-90d0-4d6d-93cb-72bd1388e19d",
                    "instances": [
                        {
                            "adjustedStart": "0:00:01.5682333",
                            "adjustedEnd": "0:00:02.7027",
                            "start": "0:00:01.5682333",
                            "end": "0:00:02.7027"
                        }
                    ]
                },
                {
                    "id": 2,
                    "thumbnailId": "9c97ae13-558c-446b-9989-21ac27439da0",
                    "instances": [
                        {
                            "adjustedStart": "0:00:16.7167",
                            "adjustedEnd": "0:00:18.018",
                            "start": "0:00:16.7167",
                            "end": "0:00:18.018"
                        }
                    ]
                },]
            }
            ...
            }
    ]
}
```

## <a name="limitations-and-assumptions"></a>Suposiciones y limitaciones 

Es importante tener en cuenta las limitaciones del seguimiento de personas observadas para evitar o mitigar los efectos de los falsos negativos (detecciones que faltan) y los detalles limitados.

* Para optimizar los resultados del detector, use secuencias de vídeo de cámaras estáticas (aunque una cámara móvil o escenas mixtas también ofrecerán resultados). 
* Por lo general, no se detectan personas si aparecen con un tamaño pequeño (la altura mínima de la persona es de 200 píxeles).
* El tamaño máximo del fotograma es HD.
* Por lo general, las personas no se detectan si no están de pie o caminando. 
* Un vídeo de baja calidad (por ejemplo, con condiciones de iluminación oscuras) puede afectar a los resultados de la detección. 
* Velocidad de fotogramas recomendada: 30 FPS como mínimo. 
* La entrada de vídeo recomendada debe contener un máximo de 10 personas en un solo fotograma. La característica podría funcionar con más personas en un solo fotograma, pero el resultado de la detección recupera 10 personas como máximo en un fotograma con el mayor grado de confianza de la detección. 
* Las personas con ropa similar (por ejemplo, personas que llevan uniformes o jugadores de equipos deportivos) podrían detectarse como una misma persona con el mismo número de identificación. 
* Oclusiones: es posible que haya errores en los que haya oclusiones (escenas propias u oclusiones de otras personas).
* Postura: las pistas se pueden dividir mediante diferentes posturas (trasera o frontal)       

## <a name="next-steps"></a>Pasos siguientes

Revise la [información general](video-indexer-overview.md).
