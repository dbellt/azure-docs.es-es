---
title: Búsqueda y censura de caras en la API de Azure Media Services v3 | Microsoft Docs
description: Azure Media Services v3 proporciona un valor preestablecido de detección y censura (desenfoque) de caras que permite enviar un archivo de vídeo, detectar caras y, opcionalmente, aplicarles una censura (desenfoque) en un solo paso combinado o a través de una operación de dos fases que permite la edición. En este artículo se muestra cómo buscar y censurar caras con el valor preestablecido de Face Detector en la API v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/25/2021
ms.author: johndeu
ms.custom: devx-track-csharp
ms.openlocfilehash: 4907a81fc8cb55499fa97f2b02a3e19e7117bbbc
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106286392"
---
# <a name="find-and-redact-blur-faces-with-the-face-detector-preset"></a>Búsqueda y censura (desenfoque) de caras con el valor preestablecido de Face Detector

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

La API de Azure Media Services v3 incluye un valor preestablecido de Face Detector que ofrece la detección de caras escalable y la censura (desenfoque) en la nube. La censura de rostros le permite modificar un vídeo con el fin de difuminar las caras de personas seleccionadas. Puede usar el servicio de censura de rostros en escenarios de seguridad pública y de noticias en los medios de comunicación. Se puede tardar horas en censurar manualmente unos minutos de imágenes que contengan varias caras, pero con este valor preestablecido, el proceso de censura de caras requiere solamente unos pocos pasos sencillos.

En este artículo se proporcionan detalles sobre el **Valor preestablecido de Face Detector** y se muestra cómo se usa con el SDK de Media Services para .NET.

[!INCLUDE [regulation](../video-indexer/includes/regulation.md)]

## <a name="compliance-privacy-and-security"></a>Cumplimiento, privacidad y seguridad
 
Como recordatorio importante, debe cumplir todas las leyes aplicables en el uso del análisis en Azure Media Services. No debe usar Azure Media Services ni ningún otro servicio de Azure de forma que infrinja los derechos de los demás. Antes de cargar vídeos que incluyan datos biométricos en el servicio Azure Media Services para procesarlos y almacenarlos, debe tener todos los derechos apropiados (por ejemplo, todos los consentimientos adecuados) de las personas del vídeo. Para información sobre el cumplimiento, la privacidad y la seguridad de Azure Media Services, consulte [Términos de Azure Cognitive Services](https://azure.microsoft.com/support/legal/cognitive-services-compliance-and-privacy/). En lo que respecta a las obligaciones de privacidad de Microsoft y al control de los datos, consulte la [declaración de privacidad](https://privacy.microsoft.com/PrivacyStatement), los [Términos de los Servicios en Línea](https://www.microsoft.com/licensing/product-licensing/products) (OST) y el [anexo de procesamiento de datos](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=67) ("DPA") de Microsoft. Puede encontrar información adicional sobre la privacidad, como la retención, eliminación o destrucción de los datos, en los términos de OST y [aquí](../video-indexer/faq.md). Al usar Azure Media Services, acepta las obligaciones de los Términos de Cognitive Services, OST, DPA y la declaración de privacidad.

## <a name="face-redaction-modes"></a>Modos de censura de rostros

La censura facial funciona detectando caras en cada fotograma de vídeo y realizando un seguimiento del objeto de cara tanto hacia delante como hacia atrás en el tiempo, para que la imagen de la misma persona pueda difuminarse también desde otros ángulos. El proceso de censura automatizado es complejo y no siempre se desenfocan todas las caras al 100 %. Por este motivo, el valor preestablecido se puede usar en modo de dos pasos para mejorar la calidad y la precisión del desenfoque a través de una fase de edición antes de enviar el archivo para el último paso de desenfoque. 

Además de un modo **Combinado** totalmente automático, el flujo de trabajo de dos pasos le permite elegir las caras que quiere desenfocar (o no desenfocar) mediante una lista de id. de caras. Para realizar ajustes arbitrarios por fotograma, el valor preestablecido usa un archivo de metadatos en formato JSON como entrada para el segundo paso. Este flujo de trabajo se divide en los modos **Analyze** (Analizar) y **Redact** (Censurar).

Puede combinar los dos modos en un único paso que ejecuta ambas tareas en un trabajo; este modo se denomina **Combinado**.  En este artículo, el código de ejemplo mostrará cómo usar el modo **Combinado** de paso único simplificado en un archivo de código fuente de ejemplo.

### <a name="combined-mode"></a>Modo combinado

Esto genera un archivo de vídeo MP4 censurado en un solo paso sin que sea necesario editar manualmente el archivo JSON. La salida de la carpeta de recursos para el trabajo será un único archivo .mp4 que contendrá caras desenfocadas con el efecto de desenfoque seleccionado. Use la propiedad de resolution establecida en **SourceResolution** para obtener los mejores resultados de censura.

| Fase | Nombre de archivo | Notas |
| --- | --- | --- |
| Recurso de entrada |"ignite-sample.mp4" |Vídeo en formato WMV, MOV o MP4 |
| Configuración de valor preestablecido |Configuración de Face Detector | **modo**: FaceRedactorMode.Combined, **blurType**: BlurType.Med, **resolución**: AnalysisResolution.SourceResolution |
| Recurso de salida |"ignite-redacted.mp4 |Vídeo con efecto de desenfoque aplicado a caras |

### <a name="analyze-mode"></a>Modo Analyze (Análisis)

El paso **Analizar** del flujo de trabajo de dos pasos toma una entrada de vídeo y genera un archivo JSON con una lista de ubicaciones de caras, id. de caras e imágenes jpg de cada cara detectada. 

| Fase | Nombre de archivo | Notas |
| --- | --- | --- |
| Recurso de entrada |"ignite-sample.mp4" |Vídeo en formato WMV, MOV o MP4 |
| Configuración de valor preestablecido |Configuración de Face Detector |**modo**: FaceRedactorMode.Analyze, **resolución**: AnalysisResolution.SourceResolution|
| Recurso de salida |ignite-sample_annotations.json |Datos de anotación de ubicaciones de rostros en formato JSON. El usuario puede editarlo para modificar los rectángulos de selección para el difuminado. Vea el ejemplo a continuación. |
| Recurso de salida |foo_thumb%06d.jpg [foo_thumb000001.jpg, foo_thumb000002.jpg] |Un jpg recortado de cada rostro detectado, donde el número indica el identificador (labelId) de la cara |

#### <a name="output-example"></a>Ejemplo de salida

```json
{
  "version": 1,
  "timescale": 24000,
  "offset": 0,
  "framerate": 23.976,
  "width": 1280,
  "height": 720,
  "fragments": [
    {
      "start": 0,
      "duration": 48048,
      "interval": 1001,
      "events": [
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [
          {
            "index": 13,
            "id": 1138,
            "x": 0.29537,
            "y": -0.18987,
            "width": 0.36239,
            "height": 0.80335
          },
          {
            "index": 13,
            "id": 2028,
            "x": 0.60427,
            "y": 0.16098,
            "width": 0.26958,
            "height": 0.57943
          }
        ],

    ... truncated
```

### <a name="redact-blur-mode"></a>Modo de censura (desenfoque)

El segundo paso del flujo de trabajo tiene un mayor número de entradas que tienen que combinarse en un solo recurso.

Esto incluye una lista de identificadores para difuminar, el vídeo original y las anotaciones JSON. Este modo usa las anotaciones para aplicar difuminado en el vídeo de entrada.

El resultado de la fase de análisis no incluye el vídeo original. El vídeo tiene que estar cargado en el recurso de entrada para la tarea de modo Redact (Censurar) y seleccionado como el archivo principal.

| Fase | Nombre de archivo | Notas |
| --- | --- | --- |
| Recurso de entrada |"ignite-sample.mp4" |Vídeo en formato WMV, MOV o MP4. El mismo vídeo que en el paso 1. |
| Recurso de entrada |"ignite-sample_annotations.json" |Archivo de metadatos de anotaciones de la fase uno, con modificaciones opcionales si desea cambiar las caras desenfocadas. Debe editarse en una aplicación externa, código o editor de texto. |
| Recurso de entrada | "ignite-sample_IDList.txt" (opcional) | Nueva lista opcional separada por líneas de identificadores de rostro para censurar. Si se deja en blanco, se aplicará el desenfoque a todas las caras del origen. Puede usar la lista para elegir de manera selectiva no desenfocar caras específicas. |
| Valor preestablecido de Face Detector  |Configuración de valor preestablecido  | **modo**: FaceRedactorMode.Redact, **blurType**: BlurType.Med |
| Recurso de salida |"ignite-sample-redacted.mp4" |Vídeo con difuminado aplicado en base a las anotaciones |

#### <a name="example-output"></a>Salida de ejemplo

Este es el resultado de una lista de identificadores con un identificador seleccionado.

foo_IDList.txt de ejemplo

```
1
2
3
```

## <a name="blur-types"></a>Tipos de desenfoque

En los modos **Combinado** o **Redact** (Censurar), hay cinco modos de desenfoque diferentes entre los que puede elegir en la configuración de la entrada JSON: **Bajo**, **Medio**, **Alto**, **Box** y **Negro**. Se usa **Medio** de forma predeterminada.

Puede encontrar ejemplos de los tipos de desenfoque a continuación.


#### <a name="low"></a>Bajo

![Ejemplo de configuración de desenfoque de resolución baja.](./media/media-services-face-redaction/blur-1.png)

#### <a name="med"></a>Medio

![Ejemplo de configuración de desenfoque de resolución media.](./media/media-services-face-redaction/blur-2.png)

#### <a name="high"></a>Alto

![Ejemplo de configuración de desenfoque de resolución alta.](./media/media-services-face-redaction/blur-3.png)

#### <a name="box"></a>Box

![Modo Box para usar en la depuración de la salida.](./media/media-services-face-redaction/blur-4.png)

#### <a name="black"></a>Negro

![El modo de caja negra cubre todas las caras con cajas negras.](./media/media-services-face-redaction/blur-5.png)

## <a name="elements-of-the-output-json-file"></a>Elementos del archivo JSON de salida

El procesador multimedia de censura proporciona detección de ubicación y seguimiento de rostros de alta precisión que puede detectar hasta 64 caras humanas en un fotograma de vídeo. Las caras de frente ofrecen los mejores resultados, mientras que las que se encuentran de lado y las caras pequeñas (inferiores o iguales a 24x24 píxeles) podrían no ser tan precisas.

[!INCLUDE [media-services-analytics-output-json](../../../includes/media-services-analytics-output-json.md)]

## <a name="net-sample-code"></a>Código de ejemplo de .NET

En el programa siguiente se muestra cómo utilizar el modo de censura de un solo paso **Combinado**:

- Crear un recurso y cargar un archivo multimedia en él.
- Configure el valor preestablecido de Face Detector que usa la configuración de modo y blurType.
- Creación de una nueva transformación mediante el valor preestablecido de Face Detector
- Descargue el archivo de vídeo censurado de salida.

## <a name="download-and-configure-the-sample"></a>Descarga y configuración del ejemplo

Clone un repositorio GitHub que contenga el ejemplo de .NET en la máquina mediante el siguiente comando:  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet.git
 ```

El ejemplo se encuentra en la carpeta [FaceRedactor](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoAnalytics/FaceRedactor). Abra [appsettings.json](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/main/VideoAnalytics/FaceRedactor/appsettings.json) en el proyecto que ha descargado. Sustituya los valores por las credenciales que obtuvo al [acceder a las API](./access-api-howto.md).

**Opcionalmente**, puede copiar el archivo **[sample.env](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/main/sample.env)** en la raíz del repositorio y rellenar los detalles allí y cambiar el nombre del archivo a **.env** (tenga en cuenta el punto de delante) para que se pueda usar en todos los proyectos de ejemplo del repositorio.  Así se elimina la necesidad de tener que rellenar un archivo appsettings.json en cada ejemplo y también le protege en la comprobación de cualquier configuración en sus propios repositorios clonados de GitHub.

#### <a name="examples"></a>Ejemplos

Este código muestra cómo configurar **FaceDetectorPreset** para un desenfoque en el modo **Combinado**.

[!code-csharp[Main](../../../media-services-v3-dotnet/VideoAnalytics/FaceRedactor/Program.cs#FaceDetectorPreset)]

En este ejemplo de código se muestra cómo se pasa el valor preestablecido a un objeto de transformación durante la creación. Después de crear la transformación, los trabajos se pueden enviar directamente a ella.

[!code-csharp[Main](../../../media-services-v3-dotnet/VideoAnalytics/FaceRedactor/Program.cs#FaceDetectorPresetTransform)]

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

