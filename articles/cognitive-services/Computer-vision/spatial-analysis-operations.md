---
title: Operaciones de análisis espacial
titleSuffix: Azure Cognitive Services
description: Las operaciones de análisis espacial.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 01/12/2021
ms.author: aahi
ms.openlocfilehash: 37ac7573a1794c97c81fe5364204f85ff14d9fa6
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107538086"
---
# <a name="spatial-analysis-operations"></a>Operaciones de análisis espacial

El análisis espacial permite analizar vídeo de streaming en tiempo real para dispositivos de cámara. Para cada dispositivo de cámara que configure, las operaciones para el análisis espacial generarán un flujo de salida de mensajes JSON enviados a la instancia de Azure IoT Hub. 

El contenedor de análisis espacial implementa las operaciones siguientes:

| Identificador de operación| Descripción|
|---------|---------|
| cognitiveservices.vision.spatialanalysis-personcount | Cuenta a las personas que se encuentran en una zona designada del campo visual de la cámara. Una sola cámara debe cubrir completamente la zona para que PersonCount registre un total preciso. <br> Emite un evento _personCountEvent_ inicial y, luego, eventos _personCountEvent_ cuando el recuento cambia.  |
| cognitiveservices.vision.spatialanalysis-personcrossingline | Hace un seguimiento de cuando una persona cruza una línea designada en el campo visual de la cámara. <br>Emite un evento _personLineEvent_ cuando la persona cruza la línea y proporciona información direccional. 
| cognitiveservices.vision.spatialanalysis-personcrossingpolygon | Emite un evento _personZoneEnterExitEvent_ cuando una persona entra o sale de la zona y proporciona información direccional con el lado numerado de la zona que se ha cruzado. Emite un evento _personZoneDwellTimeEvent_ cuando la persona sale de la zona y proporciona información direccional, así como el número de milisegundos que la persona pasó dentro de la zona. |
| cognitiveservices.vision.spatialanalysis-persondistance | Hace un seguimiento de cuando las personas infringen una regla de distancia. <br> Emite un evento _personDistanceEvent_ de manera periódica con la ubicación de cada infracción de distancia. |
| cognitiveservices.vision.spatialanalysis | Operación genérica que se puede usar para ejecutar todos los escenarios mencionados anteriormente. Esta opción es más útil si quiere ejecutar varios escenarios en la misma cámara o usar los recursos del sistema (por ejemplo, la GPU) más eficazmente. |

Todas las operaciones anteriores también están disponibles en la versión `.debug`, que tienen la capacidad de visualizar los fotogramas de vídeo a medida que se procesan. Deberá ejecutar `xhost +` en el equipo host para habilitar la visualización de los eventos y los fotogramas de vídeo.

| Identificador de operación| Descripción|
|---------|---------|
| cognitiveservices.vision.spatialanalysis-personcount.debug | Cuenta a las personas que se encuentran en una zona designada del campo visual de la cámara. <br> Emite un evento _personCountEvent_ inicial y, luego, eventos _personCountEvent_ cuando el recuento cambia.  |
| cognitiveservices.vision.spatialanalysis-personcrossingline.debug | Hace un seguimiento de cuando una persona cruza una línea designada en el campo visual de la cámara. <br>Emite un evento _personLineEvent_ cuando la persona cruza la línea y proporciona información direccional. 
| cognitiveservices.vision.spatialanalysis-personcrossingpolygon.debug | Emite un evento _personZoneEnterExitEvent_ cuando una persona entra o sale de la zona y proporciona información direccional con el lado numerado de la zona que se ha cruzado. Emite un evento _personZoneDwellTimeEvent_ cuando la persona sale de la zona y proporciona información direccional, así como el número de milisegundos que la persona pasó dentro de la zona. |
| cognitiveservices.vision.spatialanalysis-persondistance.debug | Hace un seguimiento de cuando las personas infringen una regla de distancia. <br> Emite un evento _personDistanceEvent_ de manera periódica con la ubicación de cada infracción de distancia. |
| cognitiveservices.vision.spatialanalysis.debug | Operación genérica que se puede usar para ejecutar todos los escenarios mencionados anteriormente. Esta opción es más útil si quiere ejecutar varios escenarios en la misma cámara o usar los recursos del sistema (por ejemplo, la GPU) más eficazmente. |

El análisis espacial también se puede ejecutar con [Live Video Analytics](../../media-services/live-video-analytics-edge/spatial-analysis-tutorial.md) como su módulo de IA de vídeo. 

<!--more details on the setup can be found in the [LVA Setup page](LVA-Setup.md). Below is the list of the operations supported with Live Video Analytics. -->

| Identificador de operación| Descripción|
|---------|---------|
| cognitiveservices.vision.spatialanalysis-personcount.livevideoanalytics | Cuenta a las personas que se encuentran en una zona designada del campo visual de la cámara. <br> Emite un evento _personCountEvent_ inicial y, luego, eventos _personCountEvent_ cuando el recuento cambia.  |
| cognitiveservices.vision.spatialanalysis-personcrossingline.livevideoanalytics | Hace un seguimiento de cuando una persona cruza una línea designada en el campo visual de la cámara. <br>Emite un evento _personLineEvent_ cuando la persona cruza la línea y proporciona información direccional. 
| cognitiveservices.vision.spatialanalysis-personcrossingpolygon.livevideoanalytics | Emite un evento _personZoneEnterExitEvent_ cuando una persona entra o sale de la zona y proporciona información direccional con el lado numerado de la zona que se ha cruzado. Emite un evento _personZoneDwellTimeEvent_ cuando la persona sale de la zona y proporciona información direccional, así como el número de milisegundos que la persona pasó dentro de la zona.  |
| cognitiveservices.vision.spatialanalysis-persondistance.livevideoanalytics | Hace un seguimiento de cuando las personas infringen una regla de distancia. <br> Emite un evento _personDistanceEvent_ de manera periódica con la ubicación de cada infracción de distancia. |
| cognitiveservices.vision.spatialanalysis.livevideoanalytics | Operación genérica que se puede usar para ejecutar todos los escenarios mencionados anteriormente. Esta opción es más útil si quiere ejecutar varios escenarios en la misma cámara o usar los recursos del sistema (por ejemplo, la GPU) más eficazmente. |

Las operaciones de Live Video Analytics también están disponibles en la versión `.debug` (por ejemplo, cognitiveservices.vision.spatialanalysis-personcount.livevideoanalytics.debug), que tiene la capacidad de visualizar los fotogramas de vídeo a medida que se procesan. Deberá ejecutar `xhost +` en el equipo host para habilitar la visualización de los eventos y los fotogramas de vídeo.

> [!IMPORTANT]
> Los modelos de IA de Computer Vision detectan y localizan la presencia humana en salida y secuencias de vídeo con un rectángulo de selección alrededor de un cuerpo humano. Los modelos de IA no intentan detectar las identidades ni los datos demográficos de los individuos.

Estos son los parámetros necesarios para cada una de estas operaciones de análisis espacial.

| Parámetros de la operación.| Descripción|
|---------|---------|
| Id. de operación | Identificador de operación de la tabla anterior.|
| enabled | Booleano: true o false|
| VIDEO_URL| Dirección URL de RTSP del dispositivo de cámara (ejemplo: `rtsp://username:password@url`). El análisis espacial admite la secuencia con codificación H.264 mediante RTSP, HTTP o mp4. Video_URL se puede proporcionar como un valor de cadena de Base64 ofuscado mediante el cifrado AES, y, si la dirección URL del vídeo se ofusca, `KEY_ENV` y `IV_ENV` se deben proporcionar como variables de entorno. Se puede encontrar una utilidad de ejemplo para generar claves y cifrado [aquí](/dotnet/api/system.security.cryptography.aesmanaged). |
| VIDEO_SOURCE_ID | Nombre descriptivo para el dispositivo de cámara o la secuencia de vídeo. Se devolverá con la salida JSON del evento.|
| VIDEO_IS_LIVE| True para dispositivos de cámara; false para vídeos grabados.|
| VIDEO_DECODE_GPU_INDEX| GPU con la que se va a decodificar el fotograma de vídeo. De manera predeterminada, es 0. Esta información debe corresponder a la misma de `gpu_index` en otra configuración de nodo, como `VICA_NODE_CONFIG`, `DETECTOR_NODE_CONFIG`.|
| INPUT_VIDEO_WIDTH | Ancho de fotograma de la secuencia o vídeo de entrada (por ejemplo, 1920). Es un campo opcional y, si se proporciona, el fotograma se escalará hasta esta dimensión mientras se conserva la relación de aspecto.|
| DETECTOR_NODE_CONFIG | JSON que indica en qué GPU se va a ejecutar el nodo detector. Debería tener el siguiente formato: `"{ \"gpu_index\": 0 }",`.|
| SPACEANALYTICS_CONFIG | Configuración JSON para la zona y la línea tal como se describe a continuación.|
| ENABLE_FACE_MASK_CLASSIFIER | `True` para habilitar la detección de personas que lleven máscaras faciales en la secuencia de vídeo; `False` para deshabilitarla. De manera predeterminada, esta opción está deshabilitada. La detección de máscaras faciales requiere que el parámetro de ancho de vídeo de entrada sea 1920 `"INPUT_VIDEO_WIDTH": 1920`. El atributo de máscara facial no se devolverá si la persona detectada no está mirando a la cámara o está demasiado lejos de ella. Consulte la guía de [selección de ubicación de la cámara](spatial-analysis-camera-placement.md) para más información. |

### <a name="detector-node-parameter-settings"></a>Configuración de parámetros del nodo de detector
Este es un ejemplo de los parámetros DETECTOR_NODE_CONFIG para todas las operaciones de análisis espacial.

```json
{
"gpu_index": 0,
"do_calibration": true,
"enable_recalibration": true,
"calibration_quality_check_frequency_seconds":86400,
"calibration_quality_check_sample_collect_frequency_seconds": 300,
"calibration_quality_check_one_round_sample_collect_num":10,
"calibration_quality_check_queue_max_size":1000
}
```

| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `gpu_index` | string| Índice de la GPU en que se ejecutará esta operación.|
| `do_calibration` | string | Indica que la calibración está activada. `do_calibration` debe ser true para que **cognitiveservices.vision.spatialanalysis-persondistance** funcione correctamente. De manera predeterminada, do_calibration se establece en true. |
| `enable_recalibration` | bool | Indica si la recalibración automática está activada. El valor predeterminado es `true`.|
| `calibration_quality_check_frequency_seconds` | int | Número mínimo de segundos entre cada comprobación de calidad para determinar si se necesita o no una nueva calibración. El valor predeterminado es `86400` (24 horas). Solo se usa con `enable_recalibration=True`.|
| `calibration_quality_check_sample_collect_frequency_seconds` | int | Número mínimo de segundos entre la recopilación de nuevas muestras de datos para la recalibración y la comprobación de calidad. El valor predeterminado es `300` (5 minutos). Solo se usa con `enable_recalibration=True`.|
| `calibration_quality_check_one_round_sample_collect_num` | int | Número mínimo de muestras de datos nuevas que se van a recopilar por cada ronda de recopilación de muestras. El valor predeterminado es `10`. Solo se usa con `enable_recalibration=True`.|
| `calibration_quality_check_queue_max_size` | int | Número máximo de muestras de datos que se van a almacenar cuando se calibre el modelo de cámara. El valor predeterminado es `1000`. Solo se usa con `enable_recalibration=True`.|
| `enable_breakpad`| bool | Indica si quiere habilitar Breakpad, que se usa para generar volcados de memoria para su uso en la depuración. De forma predeterminada, su valor es `false`. Si lo establece en `true`, también debe agregar `"CapAdd": ["SYS_PTRACE"]` en la parte `HostConfig` del contenedor `createOptions`. De forma predeterminada, el volcado de memoria se carga en la aplicación [RealTimePersonTracking](https://appcenter.ms/orgs/Microsoft-Organization/apps/RealTimePersonTracking/crashes/errors?version=&appBuild=&period=last90Days&status=&errorType=all&sortCol=lastError&sortDir=desc) de AppCenter; si quiere que se carguen en su propia aplicación de AppCenter, puede invalidar la variable de entorno `RTPT_APPCENTER_APP_SECRET` con el secreto de su aplicación.

## <a name="spatial-analysis-operations-configuration-and-output"></a>Configuración y salida de operaciones de análisis espacial
### <a name="zone-configuration-for-cognitiveservicesvisionspatialanalysis-personcount"></a>Configuración de zona para cognitiveservices.vision.spatialanalysis-personcount

 Ejemplo de una entrada JSON para el parámetro SPACEANALYTICS_CONFIG que configura una zona. Puede configurar varias zonas para esta operación.

```json
{
"zones":[{
       "name": "lobbycamera",
       "polygon": [[0.3,0.3], [0.3,0.9], [0.6,0.9], [0.6,0.3], [0.3,0.3]],
       "events":[{
              "type": "count",
              "config":{
                     "trigger": "event",
            "threshold": 16.00,
            "focus": "footprint"
      }
       }]
}
```

| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `zones` | list| Lista de zonas. |
| `name` | string| Nombre descriptivo de esta zona.|
| `polygon` | list| Cada par de valores representa la x e y de los vértices de un polígono. El polígono representa las áreas en las que se hace un seguimiento o un recuento de las personas y los puntos de polígono se basan en coordenadas normalizadas (0-1), donde la esquina superior izquierda es (0.0, 0.0) y la esquina inferior derecha es (1.0, 1.0).   
| `threshold` | FLOAT| Los eventos se generan cuando la persona tiene un tamaño mayor que este número de píxeles dentro de la zona. |
| `type` | string| Para **cognitiveservices.vision.spatialanalysis-personcount**, debe ser `count`.|
| `trigger` | string| Tipo de desencadenador para enviar un evento. Los valores admitidos son `event` para enviar eventos cuando el recuento cambia o `interval` para enviar eventos de manera periódica, independientemente de si el recuento ha cambiado o no.
| `output_frequency` | int | Velocidad a la que se generan los eventos. Cuando `output_frequency` = X, se genera cada X evento, por ejemplo, `output_frequency` = 2 significa que se genera evento por medio. `output_frequency` se aplica a `event` y `interval`. |
| `focus` | string| Ubicación del punto en el rectángulo delimitador de la persona usado para calcular los eventos. El valor del foco puede ser `footprint` (la superficie de la persona), `bottom_center` (la parte inferior central del rectángulo delimitador de la persona) o `center` (el centro del cuadro delimitador de la persona).|

### <a name="line-configuration-for-cognitiveservicesvisionspatialanalysis-personcrossingline"></a>Configuración de línea para cognitiveservices.vision.spatialanalysis-personcrossingline

Ejemplo de una entrada JSON para el parámetro SPACEANALYTICS_CONFIG que configura una línea. Puede configurar varias líneas de cruce para esta operación.

```json
{
   "lines": [
       {
           "name": "doorcamera",
           "line": {
               "start": {
                   "x": 0,
                   "y": 0.5
               },
               "end": {
                   "x": 1,
                   "y": 0.5
               }
           },
           "events": [
               {
                   "type": "linecrossing",
                   "config": {
                       "trigger": "event",
                       "threshold": 16.00,
                       "focus": "footprint"
                   }
               }
           ]
       }
   ]
}
```

| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `lines` | list| Lista de líneas.|
| `name` | string| Nombre descriptivo de esta línea.|
| `line` | list| Definición de la línea. Se trata de una línea direccional que le permite comprender "entrada" frente a "salida".|
| `start` | value pair| Coordenadas x, y para el punto inicial de la línea. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma. |
| `end` | value pair| Coordenadas x, y para el punto final de la línea. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma. |
| `threshold` | FLOAT| Los eventos se generan cuando la persona tiene un tamaño mayor que este número de píxeles dentro de la zona. El valor predeterminado es 16. Este es el valor recomendado para lograr la máxima precisión. |
| `type` | string| Para **cognitiveservices.vision.spatialanalysis-personcrossingline**, debe ser `linecrossing`.|
|`trigger`|string|Tipo de desencadenador para enviar un evento.<br>Valores admitidos: "event": se activa cuando alguna persona cruza la línea.|
| `focus` | string| Ubicación del punto en el rectángulo delimitador de la persona usado para calcular los eventos. El valor del foco puede ser `footprint` (la superficie de la persona), `bottom_center` (la parte inferior central del rectángulo delimitador de la persona) o `center` (el centro del cuadro delimitador de la persona). El valor predeterminado es footprint.|

### <a name="zone-configuration-for-cognitiveservicesvisionspatialanalysis-personcrossingpolygon"></a>Configuración de zona para cognitiveservices.vision.spatialanalysis-personcrossingpolygon

Ejemplo de una entrada JSON para el parámetro SPACEANALYTICS_CONFIG que configura una zona. Puede configurar varias zonas para esta operación.

 ```json
{
"zones":[
   {
       "name": "queuecamera",
       "polygon": [[0.3,0.3], [0.3,0.9], [0.6,0.9], [0.6,0.3], [0.3,0.3]],
       "events":[{
           "type": "zonecrossing",
           "config":{
               "trigger": "event",
               "threshold": 48.00,
               "focus": "footprint"
               }
           }]
   },
   {
       "name": "queuecamera1",
       "polygon": [[0.3,0.3], [0.3,0.9], [0.6,0.9], [0.6,0.3], [0.3,0.3]],
       "events":[{
           "type": "zonedwelltime",
           "config":{
               "trigger": "event",
               "threshold": 16.00,
               "focus": "footprint"
               }
           }]
   }]
}
```

| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `zones` | list| Lista de zonas. |
| `name` | string| Nombre descriptivo de esta zona.|
| `polygon` | list| Cada par de valores representa la x e y de los vértices del polígono. El polígono representa las áreas en las que se realiza el seguimiento o el recuento de personas. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma. 
| `threshold` | FLOAT| Los eventos se generan cuando la persona tiene un tamaño mayor que este número de píxeles dentro de la zona. El valor predeterminado es 48 cuando el tipo es zonecrossing, y 16 cuando la hora es DwellTime. Estos son los valores recomendados para lograr la máxima precisión.  |
| `type` | string| Para **cognitiveservices.vision.spatialanalysis-personcrossingpolygon**, debe ser `zonecrossing` o `zonedwelltime`.|
| `trigger`|string|Tipo de desencadenador para enviar un evento.<br>Valores admitidos: "event": se activa cuando alguna persona entra o sale de la zona.|
| `focus` | string| Ubicación del punto en el rectángulo delimitador de la persona usado para calcular los eventos. El valor del foco puede ser `footprint` (la superficie de la persona), `bottom_center` (la parte inferior central del rectángulo delimitador de la persona) o `center` (el centro del cuadro delimitador de la persona). El valor predeterminado es footprint.|

### <a name="zone-configuration-for-cognitiveservicesvisionspatialanalysis-persondistance"></a>Configuración de zona para cognitiveservices.vision.spatialanalysis-persondistance

Este es un ejemplo de una entrada JSON para el parámetro SPACEANALYTICS_CONFIG que configura una zona para **cognitiveservices.vision.spatialanalysis-persondistance**. Puede configurar varias zonas para esta operación.

```json
{
"zones":[{
   "name": "lobbycamera",
   "polygon": [[0.3,0.3], [0.3,0.9], [0.6,0.9], [0.6,0.3], [0.3,0.3]],
   "events":[{
       "type": "persondistance",
       "config":{
           "trigger": "event",
           "output_frequency":1,
           "minimum_distance_threshold":6.0,
           "maximum_distance_threshold":35.0,
        "aggregation_method": "average"
           "threshold": 16.00,
           "focus": "footprint"
                   }
          }]
   }]
}
```

| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `zones` | list| Lista de zonas. |
| `name` | string| Nombre descriptivo de esta zona.|
| `polygon` | list| Cada par de valores representa la x e y de los vértices del polígono. El polígono representa las áreas en las que se cuentan las personas y se mide la distancia entre ellas. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma. 
| `threshold` | FLOAT| Los eventos se generan cuando la persona tiene un tamaño mayor que este número de píxeles dentro de la zona. |
| `type` | string| Para **cognitiveservices.vision.spatialanalysis-persondistance**, debe ser `people_distance`.|
| `trigger` | string| Tipo de desencadenador para enviar un evento. Los valores admitidos son `event` para enviar eventos cuando el recuento cambia o `interval` para enviar eventos de manera periódica, independientemente de si el recuento ha cambiado o no.
| `output_frequency` | int | Velocidad a la que se generan los eventos. Cuando `output_frequency` = X, se genera cada X evento, por ejemplo, `output_frequency` = 2 significa que se genera evento por medio. `output_frequency` se aplica a `event` y `interval`.|
| `minimum_distance_threshold` | FLOAT| Distancia en pies que desencadenará un evento "TooClose" cuando las personas estén más cerca de esa distancia entre sí.|
| `maximum_distance_threshold` | FLOAT| Distancia en pies que desencadenará un evento "TooFar" cuando las personas estén más lejos de esa distancia entre sí.|
| `aggregation_method` | string| Método para agregar el resultado de persondistance. El elemento aggregation_method se aplica tanto a `mode` como a `average`.|
| `focus` | string| Ubicación del punto en el rectángulo delimitador de la persona usado para calcular los eventos. El valor del foco puede ser `footprint` (la superficie de la persona), `bottom_center` (la parte inferior central del rectángulo delimitador de la persona) o `center` (el centro del cuadro delimitador de la persona).|

### <a name="configuration-for-cognitiveservicesvisionspatialanalysis"></a>Configuración de cognitiveservices.vision.spatialanalysis
Este es un ejemplo de una entrada JSON para el parámetro SPACEANALYTICS_CONFIG que configura una línea y zona para **cognitiveservices.vision.spatialanalysis**. Puede configurar varias líneas o zonas para esta operación y cada línea o zona puede tener eventos diferentes.

 ```
{
  "lines": [
    {
      "name": "doorcamera",
      "line": {
        "start": {
          "x": 0,
          "y": 0.5
        },
        "end": {
          "x": 1,
          "y": 0.5
        }
      },
      "events": [
        {
          "type": "linecrossing",
          "config": {
            "trigger": "event",
            "threshold": 16.00,
            "focus": "footprint"
          }
        }
      ]
    }
  ],
  "zones": [
    {
      "name": "lobbycamera",
      "polygon": [[0.3, 0.3],[0.3, 0.9],[0.6, 0.9],[0.6, 0.3],[0.3, 0.3]],
      "events": [
        {
          "type": "persondistance",
          "config": {
            "trigger": "event",
            "output_frequency": 1,
            "minimum_distance_threshold": 6.0,
            "maximum_distance_threshold": 35.0,
            "threshold": 16.00,
            "focus": "footprint"
          }
        },
        {
          "type": "count",
          "config": {
            "trigger": "event",
            "output_frequency": 1,
            "threshold": 16.00,
            "focus": "footprint"
          }
        },
        {
          "type": "zonecrossing",
          "config": {
            "threshold": 48.00,
            "focus": "footprint"
          }
        },
        {
          "type": "zonedwelltime",
          "config": {
            "threshold": 16.00,
            "focus": "footprint"
          }
        }
      ]
    }
  ]
}
```
## <a name="camera-configuration"></a>Configuración de la cámara

Consulte las instrucciones de la [colocación de la cámara](spatial-analysis-camera-placement.md) para más información sobre cómo configurar zonas y líneas.

## <a name="spatial-analysis-operation-output"></a>Salida de la operación de análisis espacial

Los eventos de cada operación se emiten a Azure IoT Hub en formato JSON.

### <a name="json-format-for-cognitiveservicesvisionspatialanalysis-personcount-ai-insights"></a>Formato JSON para Conclusiones de IA de cognitiveservices.vision.spatialanalysis-personcount

JSON de ejemplo de un evento generado por esta operación.

```json
{
    "events": [
        {
            "id": "b013c2059577418caa826844223bb50b",
            "type": "personCountEvent",
            "detectionIds": [
                "bc796b0fc2534bc59f13138af3dd7027",
                "60add228e5274158897c135905b5a019"
            ],
            "properties": {
                "personCount": 2
            },
            "zone": "lobbycamera",
            "trigger": "event"
        }
    ],
    "sourceInfo": {
        "id": "camera_id",
        "timestamp": "2020-08-24T06:06:57.224Z",
        "width": 608,
        "height": 342,
        "frameId": "1400",
        "cameraCalibrationInfo": {
            "status": "Calibrated",
            "cameraHeight": 10.306597709655762,
            "focalLength": 385.3199462890625,
            "tiltupAngle": 1.0969393253326416
        },
        "imagePath": ""
    },
    "detections": [
        {
            "type": "person",
            "id": "bc796b0fc2534bc59f13138af3dd7027",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.612683747944079,
                        "y": 0.25340268765276636
                    },
                    {
                        "x": 0.7185954043739721,
                        "y": 0.6425260577285499
                    }
                ]
            },
            "confidence": 0.9559211134910583,
            "centerGroundPoint": {
                "x": 0.0,
                "y": 0.0
            },
            "metadata": {
            "attributes": {
                "face_mask": 0.99
            }
        }
        },
        {
            "type": "person",
            "id": "60add228e5274158897c135905b5a019",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.22326200886776573,
                        "y": 0.17830915618361087
                    },
                    {
                        "x": 0.34922296122500773,
                        "y": 0.6297955429344847
                    }
                ]
            },
            "confidence": 0.9389744400978088,
            "centerGroundPoint": {
                "x": 0.0,
                "y": 0.0
            },
            "metadata":{
            "attributes": {
            "face_nomask": 0.99
            }
            }
       }
    ],
    "schemaVersion": "1.0"
}
```

| Nombre de campo de evento | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de evento|
| `type` | string| Tipo de evento|
| `detectionsId` | array| Matriz de tamaño 1 del identificador único de la detección de personas que desencadenó este evento|
| `properties` | collection| Colección de valores|
| `trackinId` | string| Identificador único de la persona detectada|
| `zone` | string | Campo "name" del polígono que representa la zona que se cruzó|
| `trigger` | string| El tipo de desencadenador es "event" o "interval" en función del valor de `trigger` en SPACEANALYTICS_CONFIG|

| Nombre del campo de detecciones | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de detección|
| `type` | string| Tipo de detección|
| `region` | collection| Colección de valores|
| `type` | string| Tipo de región|
| `points` | collection| Puntos superior izquierdo e inferior derecho cuando el tipo de región es RECTANGLE (RECTÁNGULO) |
| `confidence` | FLOAT| Confianza del algoritmo|
| `face_mask` | FLOAT | El valor de confianza del atributo con el intervalo (0-1) indica que la persona detectada lleva una máscara facial. |
| `face_nomask` | FLOAT | El valor de confianza del atributo con el intervalo (0-1) indica que la persona detectada **no** lleva una máscara facial. |

| Nombre del campo SourceInfo | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Camera ID (Identificador de la cámara)|
| `timestamp` | Fecha| Fecha UTC cuando se emitió la carga de JSON|
| `width` | int | Ancho del fotograma de vídeo|
| `height` | int | Alto del fotograma de vídeo|
| `frameId` | int | Identificador del fotograma|
| `cameraCallibrationInfo` | collection | Colección de valores|
| `status` | string | Estado de la calibración en el formato de `state[;progress description]`. El estado puede ser `Calibrating`, `Recalibrating` (si está habilitada la recalibración) o `Calibrated`. La parte de descripción del progreso solo es válida cuando está en estado `Calibrating` y `Recalibrating`, y se usa para mostrar el progreso del proceso de calibración actual.|
| `cameraHeight` | FLOAT | Alto de la cámara sobre el suelo en pies. Se deduce a partir de la calibración automática. |
| `focalLength` | FLOAT | Longitud focal de la cámara en píxeles. Se deduce a partir de la calibración automática. |
| `tiltUpAngle` | FLOAT | Ángulo de inclinación de la cámara desde el vertical. Se deduce a partir de la calibración automática.|


### <a name="json-format-for-cognitiveservicesvisionspatialanalysis-personcrossingline-ai-insights"></a>Formato JSON para Conclusiones de IA de cognitiveservices.vision.spatialanalysis-personcrossingline

Ejemplo de JSON para las detecciones generadas por esta operación.

```json
{
    "events": [
        {
            "id": "3733eb36935e4d73800a9cf36185d5a2",
            "type": "personLineEvent",
            "detectionIds": [
                "90d55bfc64c54bfd98226697ad8445ca"
            ],
            "properties": {
                "trackingId": "90d55bfc64c54bfd98226697ad8445ca",
                "status": "CrossLeft"
            },
            "zone": "doorcamera"
        }
    ],
    "sourceInfo": {
        "id": "camera_id",
        "timestamp": "2020-08-24T06:06:53.261Z",
        "width": 608,
        "height": 342,
        "frameId": "1340",
        "imagePath": ""
    },
    "detections": [
        {
            "type": "person",
            "id": "90d55bfc64c54bfd98226697ad8445ca",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.491627341822574,
                        "y": 0.2385801348769874
                    },
                    {
                        "x": 0.588894994635331,
                        "y": 0.6395559924387793
                    }
                ]
            },
            "confidence": 0.9005028605461121,
            "metadata": {
            "attributes": {
                "face_mask": 0.99
            }
        }
        }
    ],
    "schemaVersion": "1.0"
}
```
| Nombre de campo de evento | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de evento|
| `type` | string| Tipo de evento|
| `detectionsId` | array| Matriz de tamaño 1 del identificador único de la detección de personas que desencadenó este evento|
| `properties` | collection| Colección de valores|
| `trackinId` | string| Identificador único de la persona detectada|
| `status` | string| Dirección de los cruces de líneas, ya sea "CrossLeft" o "CrossRight" La dirección se basa en imaginarse de pie en el "inicio" de cara al "final" de la línea. CrossRight cruza de izquierda a derecha. CrossLeft cruza de derecha a izquierda.|
| `zone` | string | Campo "name" de la línea que se cruzó|

| Nombre del campo de detecciones | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de detección|
| `type` | string| Tipo de detección|
| `region` | collection| Colección de valores|
| `type` | string| Tipo de región|
| `points` | collection| Puntos superior izquierdo e inferior derecho cuando el tipo de región es RECTANGLE (RECTÁNGULO) |
| `confidence` | FLOAT| Confianza del algoritmo|
| `face_mask` | FLOAT | El valor de confianza del atributo con el intervalo (0-1) indica que la persona detectada lleva una máscara facial. |
| `face_nomask` | FLOAT | El valor de confianza del atributo con el intervalo (0-1) indica que la persona detectada **no** lleva una máscara facial. |

| Nombre del campo SourceInfo | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Camera ID (Identificador de la cámara)|
| `timestamp` | Fecha| Fecha UTC cuando se emitió la carga de JSON|
| `width` | int | Ancho del fotograma de vídeo|
| `height` | int | Alto del fotograma de vídeo|
| `frameId` | int | Identificador del fotograma|


> [!IMPORTANT]
> El modelo de IA detecta a una persona independientemente de si el rostro de la persona está frente a la cámara o no. El modelo de IA no ejecuta el reconocimiento facial y no emite información biométrica alguna. 

### <a name="json-format-for-cognitiveservicesvisionspatialanalysis-personcrossingpolygon-ai-insights"></a>Formato JSON para Conclusiones de IA de cognitiveservices.vision.spatialanalysis-personcrossingpolygon

Ejemplo JSON de las detecciones generadas por esta operación con el tipo `zonecrossing` SPACEANALYTICS_CONFIG.

```json
{
    "events": [
        {
            "id": "f095d6fe8cfb4ffaa8c934882fb257a5",
            "type": "personZoneEnterExitEvent",
            "detectionIds": [
                "afcc2e2a32a6480288e24381f9c5d00e"
            ],
            "properties": {
                "trackingId": "afcc2e2a32a6480288e24381f9c5d00e",
                "status": "Enter",
                "side": "1"
            },
            "zone": "queuecamera"
        }
    ],
    "sourceInfo": {
        "id": "camera_id",
        "timestamp": "2020-08-24T06:15:09.680Z",
        "width": 608,
        "height": 342,
        "frameId": "428",
        "imagePath": ""
    },
    "detections": [
        {
            "type": "person",
            "id": "afcc2e2a32a6480288e24381f9c5d00e",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.8135572734631991,
                        "y": 0.6653949670624315
                    },
                    {
                        "x": 0.9937645761590255,
                        "y": 0.9925406829655519
                    }
                ]
            },
            "confidence": 0.6267998814582825,
        "metadata": {
        "attributes": {
        "face_mask": 0.99
        }
        }
           
        }
    ],
    "schemaVersion": "1.0"
}
```

Ejemplo JSON de las detecciones generadas por esta operación con el tipo `zonedwelltime` SPACEANALYTICS_CONFIG.

```json
{
    "events": [
        {
            "id": "f095d6fe8cfb4ffaa8c934882fb257a5",
            "type": "personZoneDwellTimeEvent",
            "detectionIds": [
                "afcc2e2a32a6480288e24381f9c5d00e"
            ],
            "properties": {
                "trackingId": "afcc2e2a32a6480288e24381f9c5d00e",
                "status": "Exit",
                "side": "1",
              "durationMs": 7132.0
            },
            "zone": "queuecamera"
        }
    ],
    "sourceInfo": {
        "id": "camera_id",
        "timestamp": "2020-08-24T06:15:09.680Z",
        "width": 608,
        "height": 342,
        "frameId": "428",
        "imagePath": ""
    },
    "detections": [
        {
            "type": "person",
            "id": "afcc2e2a32a6480288e24381f9c5d00e",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.8135572734631991,
                        "y": 0.6653949670624315
                    },
                    {
                        "x": 0.9937645761590255,
                        "y": 0.9925406829655519
                    }
                ]
            },
            "confidence": 0.6267998814582825,
            "metadataType": ""
        }
    ],
    "schemaVersion": "1.0"
}
```

| Nombre de campo de evento | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de evento|
| `type` | string| Tipo de evento. El valor puede ser _personZoneDwellTimeEvent_ o _personZoneEnterExitEvent_.|
| `detectionsId` | array| Matriz de tamaño 1 del identificador único de la detección de personas que desencadenó este evento|
| `properties` | collection| Colección de valores|
| `trackinId` | string| Identificador único de la persona detectada|
| `status` | string| Dirección de los cruces de los polígonos, ya sea "Enter" "Exit"|
| `side` | int| Número del lado del polígono que cruzó la persona. Cada lado es un borde numerado entre los dos vértices del polígono que representa la zona. El borde entre los dos primeros vértices del polígono representa el primer lado. El "lado" está vacío cuando el evento no está asociado a un lado específico debido a una oclusión. Por ejemplo, se produce una salida cuando una persona desaparece, pero no se la ve cruzar un lado de la zona o se produce una entrada cuando una persona aparece en la zona, pero no se la ve cruzar un lado.|
| `durationMs` | FLOAT | El número de milisegundos que representa el tiempo que la persona pasó en la zona. Este campo se proporciona cuando el tipo de evento es _personZoneDwellTimeEvent_.|
| `zone` | string | Campo "name" del polígono que representa la zona que se cruzó|

| Nombre del campo de detecciones | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de detección|
| `type` | string| Tipo de detección|
| `region` | collection| Colección de valores|
| `type` | string| Tipo de región|
| `points` | collection| Puntos superior izquierdo e inferior derecho cuando el tipo de región es RECTANGLE (RECTÁNGULO) |
| `confidence` | FLOAT| Confianza del algoritmo|
| `face_mask` | FLOAT | El valor de confianza del atributo con el intervalo (0-1) indica que la persona detectada lleva una máscara facial. |
| `face_nomask` | FLOAT | El valor de confianza del atributo con el intervalo (0-1) indica que la persona detectada **no** lleva una máscara facial. |

### <a name="json-format-for-cognitiveservicesvisionspatialanalysis-persondistance-ai-insights"></a>Formato JSON para Conclusiones de IA de cognitiveservices.vision.spatialanalysis-persondistance

Ejemplo de JSON para las detecciones generadas por esta operación.

```json
{
    "events": [
        {
            "id": "9c15619926ef417aa93c1faf00717d36",
            "type": "personDistanceEvent",
            "detectionIds": [
                "9037c65fa3b74070869ee5110fcd23ca",
                "7ad7f43fd1a64971ae1a30dbeeffc38a"
            ],
            "properties": {
                "personCount": 5,
                "averageDistance": 20.807043981552123,
                "minimumDistanceThreshold": 6.0,
                "maximumDistanceThreshold": "Infinity",
                "eventName": "TooClose",
                "distanceViolationPersonCount": 2
            },
            "zone": "lobbycamera",
            "trigger": "event"
        }
    ],
    "sourceInfo": {
        "id": "camera_id",
        "timestamp": "2020-08-24T06:17:25.309Z",
        "width": 608,
        "height": 342,
        "frameId": "1199",
        "cameraCalibrationInfo": {
            "status": "Calibrated",
            "cameraHeight": 12.9940824508667,
            "focalLength": 401.2800598144531,
            "tiltupAngle": 1.057669997215271
        },
        "imagePath": ""
    },
    "detections": [
        {
            "type": "person",
            "id": "9037c65fa3b74070869ee5110fcd23ca",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.39988183975219727,
                        "y": 0.2719132942065858
                    },
                    {
                        "x": 0.5051516984638414,
                        "y": 0.6488402517218339
                    }
                ]
            },
            "confidence": 0.948630690574646,
            "centerGroundPoint": {
                "x": -1.4638760089874268,
                "y": 18.29732322692871
            },
            "metadataType": ""
        },
        {
            "type": "person",
            "id": "7ad7f43fd1a64971ae1a30dbeeffc38a",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.5200299714740954,
                        "y": 0.2875368218672903
                    },
                    {
                        "x": 0.6457497446160567,
                        "y": 0.6183311060855263
                    }
                ]
            },
            "confidence": 0.8235412240028381,
            "centerGroundPoint": {
                "x": 2.6310102939605713,
                "y": 18.635927200317383
            },
            "metadataType": ""
        }
    ],
    "schemaVersion": "1.0"
}
```

| Nombre de campo de evento | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de evento|
| `type` | string| Tipo de evento|
| `detectionsId` | array| Matriz de tamaño 1 del identificador único de la detección de personas que desencadenó este evento|
| `properties` | collection| Colección de valores|
| `personCount` | int| Número de personas detectadas cuando se emitió el evento|
| `averageDistance` | FLOAT| Distancia promedio en pies entre todas las personas detectadas|
| `minimumDistanceThreshold` | FLOAT| Distancia en pies que desencadenará un evento "TooClose" cuando las personas estén más cerca de esa distancia entre sí.|
| `maximumDistanceThreshold` | FLOAT| Distancia en pies que desencadenará un evento "TooFar" cuando las personas estén más lejos de esa distancia entre sí.|
| `eventName` | string| El nombre de evento es `TooClose` cuando se infringe `minimumDistanceThreshold`, `TooFar` cuando se infringe `maximumDistanceThreshold` o `unknown` cuando no se ha completado la calibración automática|
| `distanceViolationPersonCount` | int| Número de personas detectadas en infracción de `minimumDistanceThreshold` o `maximumDistanceThreshold`|
| `zone` | string | Campo "name" del polígono que representa la zona en la que se supervisó la distancia entre las personas|
| `trigger` | string| El tipo de desencadenador es "event" o "interval" en función del valor de `trigger` en SPACEANALYTICS_CONFIG|

| Nombre del campo de detecciones | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de detección|
| `type` | string| Tipo de detección|
| `region` | collection| Colección de valores|
| `type` | string| Tipo de región|
| `points` | collection| Puntos superior izquierdo e inferior derecho cuando el tipo de región es RECTANGLE (RECTÁNGULO) |
| `confidence` | FLOAT| Confianza del algoritmo|
| `centerGroundPoint` | 2 valores flotantes| Valores `x`, `y` con las coordenadas de la ubicación deducida de la persona en el suelo, en pies. `x` y `y` son coordenadas en el plano del suelo, suponiendo que el suelo sea el nivel. La ubicación de la cámara es el origen. |

Al calcular `centerGroundPoint`, `x` es la distancia desde la cámara hasta la persona a lo largo de una línea perpendicular al plano de la imagen de la cámara. `y` es la distancia desde la cámara hasta la persona a lo largo de una línea paralela al plano de la imagen de la cámara. 

![Punto del suelo central de ejemplo](./media/spatial-analysis/x-y-chart.png) 

En este ejemplo `centerGroundPoint` es `{x: 4, y: 5}`. Esto significa que hay una persona a 4 pies de la cámara y 5 pies hacia la derecha, mirando la habitación de arriba abajo.


| Nombre del campo SourceInfo | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Camera ID (Identificador de la cámara)|
| `timestamp` | Fecha| Fecha UTC cuando se emitió la carga de JSON|
| `width` | int | Ancho del fotograma de vídeo|
| `height` | int | Alto del fotograma de vídeo|
| `frameId` | int | Identificador del fotograma|
| `cameraCallibrationInfo` | collection | Colección de valores|
| `status` | string | Estado de la calibración en el formato de `state[;progress description]`. El estado puede ser `Calibrating`, `Recalibrating` (si está habilitada la recalibración) o `Calibrated`. La parte de descripción del progreso solo es válida cuando está en estado `Calibrating` y `Recalibrating`, y se usa para mostrar el progreso del proceso de calibración actual.|
| `cameraHeight` | FLOAT | Alto de la cámara sobre el suelo en pies. Se deduce a partir de la calibración automática. |
| `focalLength` | FLOAT | Longitud focal de la cámara en píxeles. Se deduce a partir de la calibración automática. |
| `tiltUpAngle` | FLOAT | Ángulo de inclinación de la cámara desde el vertical. Se deduce a partir de la calibración automática.|

### <a name="json-format-for-cognitiveservicesvisionspatialanalysis-ai-insights"></a>Formato JSON para las Conclusiones de IA de cognitiveservices.vision.spatialanalysis

La salida de esta operación depende del elemento `events` configurado, por ejemplo, si hay un evento `zonecrossing` configurado para esta operación, el resultado será el mismo que el de `cognitiveservices.vision.spatialanalysis-personcrossingpolygon`.

## <a name="use-the-output-generated-by-the-container"></a>Uso de la salida generada por el contenedor

Puede que quiera integrar eventos o la detección del análisis espacial en la aplicación. Estos son algunos de los enfoques que se deben tener en cuenta: 

* Use el SDK del Centro de eventos de Azure del lenguaje de programación elegido para conectarse al punto de conexión de Azure IoT Hub y recibir los eventos. Para más información, consulte [Leer mensajes del dispositivo a la nube desde el punto de conexión integrado](../../iot-hub/iot-hub-devguide-messages-read-builtin.md). 
* Configure **Enrutamiento de mensajes** en la instancia de Azure IoT Hub para enviar los eventos a otros puntos de conexión o guardar los eventos en el almacenamiento de datos. Para más información, consulte [Enrutamiento de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-d2c.md). 
* Configure un trabajo de Azure Stream Analytics para procesar los eventos en tiempo real a medida que llegan y crear visualizaciones. 

## <a name="deploying-spatial-analysis-operations-at-scale-multiple-cameras"></a>Implementación de operaciones de análisis espacial a escala (varias cámaras)

Con el fin de obtener el mejor rendimiento y uso de las GPU, puede implementar cualquier operación de análisis espacial en varias cámaras mediante instancias de grafos. A continuación, se muestra un ejemplo para ejecutar la operación `cognitiveservices.vision.spatialanalysis-personcrossingline` en quince cámaras.

```json
  "properties.desired": {
      "globalSettings": {
          "PlatformTelemetryEnabled": false,
          "CustomerTelemetryEnabled": true
      },
      "graphs": {
        "personzonelinecrossing": {
        "operationId": "cognitiveservices.vision.spatialanalysis-personcrossingline",
        "version": 1,
        "enabled": true,
        "sharedNodes": {
            "shared_detector0": {
                "node": "PersonCrossingLineGraph.detector",
                "parameters": {
                    "DETECTOR_NODE_CONFIG": "{ \"gpu_index\": 0, \"batch_size\": 7, \"do_calibration\": true}",
                }
            },
            "shared_detector1": {
                "node": "PersonCrossingLineGraph.detector",
                "parameters": {
                    "DETECTOR_NODE_CONFIG": "{ \"gpu_index\": 0, \"batch_size\": 8, \"do_calibration\": true}",
                }
            }
        },
        "parameters": {
            "VIDEO_DECODE_GPU_INDEX": 0,
            "VIDEO_IS_LIVE": true
        },
        "instances": {
            "1": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector0",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 1>",
                    "VIDEO_SOURCE_ID": "camera 1",
                    "SPACEANALYTICS_CONFIG": "{\"zones\":[{\"name\":\"queue\",\"polygon\":[[0,0],[1,0],[0,1],[1,1],[0,0]]}]}"
                }
            },
            "2": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector0",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 2>",
                    "VIDEO_SOURCE_ID": "camera 2",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "3": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector0",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 3>",
                    "VIDEO_SOURCE_ID": "camera 3",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "4": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector0",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 4>",
                    "VIDEO_SOURCE_ID": "camera 4",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "5": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector0",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 5>",
                    "VIDEO_SOURCE_ID": "camera 5",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "6": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector0",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 6>",
                    "VIDEO_SOURCE_ID": "camera 6",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "7": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector0",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 7>",
                    "VIDEO_SOURCE_ID": "camera 7",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "8": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector1",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 8>",
                    "VIDEO_SOURCE_ID": "camera 8",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "9": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector1",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 9>",
                    "VIDEO_SOURCE_ID": "camera 9",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "10": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector1",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 10>",
                    "VIDEO_SOURCE_ID": "camera 10",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "11": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector1",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 11>",
                    "VIDEO_SOURCE_ID": "camera 11",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "12": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector1",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 12>",
                    "VIDEO_SOURCE_ID": "camera 12",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "13": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector1",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 13>",
                    "VIDEO_SOURCE_ID": "camera 13",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "14": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector1",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 14>",
                    "VIDEO_SOURCE_ID": "camera 14",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            },
            "15": {
                "sharedNodeMap": {
                    "PersonCrossingLineGraph/detector": "shared_detector1",
                },
                "parameters": {
                    "VIDEO_URL": "<Replace RTSP URL for camera 15>",
                    "VIDEO_SOURCE_ID": "camera 15",
                    "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                }
            }
          }
        },
      }
  }
  ```
| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `batch_size` | int | Si todas las cámaras tienen la misma resolución, establezca `batch_size` en el número de cámaras que se utilizarán en esa operación; de lo contrario, establezca `batch_size` en 1 o déjelo como predeterminado (1), lo que indica que no se admite ningún lote. |

## <a name="next-steps"></a>Pasos siguientes

* [Implementación de una aplicación web de recuento de personas](spatial-analysis-web-app.md)
* [Registro y solución de problemas](spatial-analysis-logging.md)
* [Guía de selección de ubicación de la cámara](spatial-analysis-camera-placement.md)
* [Guía de selección de ubicación de zonas y líneas](spatial-analysis-zone-line-placement.md)
