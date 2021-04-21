---
title: Actualización a Read v3.0 de Computer Vision API
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo actualizar a la API de lectura Computer Vision v3.0 desde v2.0/v2.1.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: sample
ms.date: 08/11/2020
ms.author: pafarley
ROBOTS: NOINDEX
ms.openlocfilehash: 7a05b04872b4f957e879d93972edc45e2932d059
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107364097"
---
# <a name="upgrade-from-read-v2x-to-read-v3x"></a>Actualice de Read v2.x a Read v3.x

En esta guía se muestra cómo actualizar el código de la API de la nube o el contenedor existente de Read v2.x a Read v3.x.

## <a name="determine-your-api-path"></a>Determinación de la ruta de acceso de la API
Use la tabla siguiente para determinar la **cadena de versión** en la ruta de acceso de la API en función de la versión de Read 3.x a la que está realizando la migración.

|Tipo de producto| Versión | Cadena de versión en la ruta de acceso de la API 3.x |
|:-----|:----|:----|
|Servicio | Read 3.0 o 3.1 | **v3.0** o **v3.1** respectivamente |
|Servicio | Read 3.2 versión preliminar | **v3.2-preview.1** |
|Contenedor | Read 3.0 versión preliminar o Read 3.1 versión preliminar | **v3.0** o **v3.1-preview.2** respectivamente |


A continuación, use las secciones siguientes para restringir las operaciones y reemplazar la **cadena de versión** de la ruta de acceso de la API por el valor de la tabla. Por ejemplo, para las versiones de nube y contenedor de **Read v3.2 versión preliminar**, actualice la ruta de acceso de la API a **https://{endpoint}/vision/v3.2-preview.1/read/analyze[?language]** .

## <a name="servicecontainer"></a>Servicio/contenedor

### `Batch Read File`

|Read 2.x |Read 3.x  |
|----------|-----------|
|https://{endpoint}/vision/**v2.0/read/core/asyncBatchAnalyze**     |https://{endpoint}/vision/<**version string**>/read/analyze[?language]|
    
Está disponible el nuevo parámetro opcional _language_. Si no conoce el idioma del documento o puede que sea multilingüe, no lo incluya. 

### `Get Read Results`

|Read 2.x |Read 3.x  |
|----------|-----------|
|https://{endpoint}/vision/**v2.0/read/operations**/{operationId}     |https://{endpoint}/vision/<**version string**>/read/analyzeResults/{operationId}|

### <a name="get-read-operation-result-status-flag"></a>Marca de estado de `Get Read Operation Result`

Cuando la llamada a `Get Read Operation Result` se realiza correctamente, devuelve un campo de cadena de estado en el cuerpo JSON.
 
|Read 2.x |Read 3.x  |
|----------|-----------|
|`"NotStarted"` |    `"notStarted"`|
|`"Running"` | `"running"`|
|`"Failed"` | `"failed"`|
|`"Succeeded"` | `"succeeded"`|
    
### <a name="api-response-json"></a>Respuesta de la API (JSON) 

Tenga en cuenta los siguientes cambios en el código JSON:
* En la versión 2.x, `Get Read Operation Result` devolverá el código JSON de reconocimiento de OCR cuando el estado sea `Succeeded"`. En la versión 3.0, este campo es `succeeded`.
* Para obtener la raíz de la matriz de páginas, cambie la jerarquía JSON de `recognitionResults` a `analyzeResult`/`readResults`. La jerarquía JSON de palabras y líneas por página permanece sin cambios, por lo que no se requiere ningún cambio en el código.
* Se ha cambiado el nombre del elemento `clockwiseOrientation` del ángulo de la página a `angle` y el intervalo de 0 a 360 grados pasa a ser de -180 a 180 grados. En función del código, es posible que tenga que realizar cambios o no, ya que la mayoría de las funciones matemáticas pueden controlar ambos intervalos.

La versión 3.0 de la API también presenta las siguientes mejoras que tiene la opción de aprovechar:
* `createdDateTime` y `lastUpdatedDateTime` se agregan para que pueda realizar un seguimiento de la duración del procesamiento. Consulte la documentación para más detalles. 
* El elemento `version` indica la versión de la API usada para generar resultados.
* Se ha agregado el valor de `confidence` (confianza) por palabra. Este valor se calibra, de modo que un valor de 0,95 significa que hay un 95 % de probabilidad de que el reconocimiento sea correcto. La puntuación de confianza se puede usar para seleccionar el texto que se va a enviar para revisión humana. 
    
    
En la versión 2.X, el formato de salida es el siguiente: 
    
```json
    {
        {
                "status": "Succeeded",
                "recognitionResults": [
                    {
                    "page": 1,
                    "language": "en",
                    "clockwiseOrientation": 349.59,
                    "width": 2661,
                    "height": 1901,
                    "unit": "pixel",
                    "lines": [
                        {
                        "boundingBox": [
                            67,
                            646,
                            2582,
                            713,
                            2580,
                            876,
                            67,
                            821
                        ],
                        "text": "The quick brown fox jumps",
                        "words": [
                            {
                            "boundingBox": [
                                143,
                                650,
                                435,
                                661,
                                436,
                                823,
                                144,
                                824
                            ],
                            "text": "The",
                            },
            // The rest of result is omitted for brevity 
            
}
```
    
En la versión 3.0, se ha ajustado:
    
```json
    {
        {
            "status": "succeeded",
            "createdDateTime": "2020-05-28T05:13:21Z",
            "lastUpdatedDateTime": "2020-05-28T05:13:22Z",
            "analyzeResult": {
            "version": "3.0.0",
            "readResults": [
                {
                "page": 1,
                "language": "en",
                "angle": 0.8551,
                "width": 2661,
                "height": 1901,
                "unit": "pixel",
                "lines": [
                    {
                    "boundingBox": [
                        67,
                        646,
                        2582,
                        713,
                        2580,
                        876,
                        67,
                        821
                    ],
                    "text": "The quick brown fox jumps",
                    "words": [
                        {
                        "boundingBox": [
                            143,
                            650,
                            435,
                            661,
                            436,
                            823,
                            144,
                            824
                        ],
                        "text": "The",
                        "confidence": 0.958
                        },
        // The rest of result is omitted for brevity 
        
    }
```

## <a name="service-only"></a>Solo servicio

### `Recognize Text`
`Recognize Text` es una operación en *versión preliminar* que está *en desuso en todas las versiones de Computer Vision API*. Debe migrar desde `Recognize Text` a `Read` (v3.0) o `Batch Read File` (v2.0, v2.1). La versión 3.0 de `Read` incluye modelos más recientes y mejores para el reconocimiento de texto y características adicionales, por lo que se recomienda. Para actualizar de `Recognize Text` a `Read`:

|Recognize Text 2.x |Read 3.x  |
|----------|-----------|
|https://{endpoint}/vision/**v2.0/recognizeText[?mode]**|https://{endpoint}/vision/<**version string**>/read/analyze[?language]|
    
El parámetro _mode_ no se admite en `Read`. Se admitirá automáticamente el texto escrito a mano e impreso.
    
En la versión 3.0 está disponible el nuevo parámetro opcional _language_. Si no conoce el idioma del documento o puede que sea multilingüe, no lo incluya. 

### `Get Recognize Text Operation Result`

|Recognize Text 2.x |Read 3.x  |
|----------|-----------|
|https://{endpoint}/vision/**v2.0/textOperations/** {operationId}|https://{endpoint}/vision/<**version string**>/read/analyzeResults/{operationId}|

### <a name="get-recognize-text-operation-result-status-flags"></a>Marcas de estado de `Get Recognize Text Operation Result`
Cuando la llamada a `Get Recognize Text Operation Result` se realiza correctamente, devuelve un campo de cadena de estado en el cuerpo JSON. 
 
|Recognize Text 2.x |Read 3.x  |
|----------|-----------|
|`"NotStarted"` |    `"notStarted"`|
|`"Running"` | `"running"`|
|`"Failed"` | `"failed"`|
|`"Succeeded"` | `"succeeded"`|

### <a name="api-response-json"></a>Respuesta de la API (JSON)

Tenga en cuenta los siguientes cambios en el código JSON:    
* En la versión 2.x, `Get Read Operation Result` devolverá el código JSON de reconocimiento de OCR cuando el estado sea `Succeeded`. En la versión 3.x, este campo es `succeeded`.
* Para obtener la raíz de la matriz de páginas, cambie la jerarquía JSON de `recognitionResult` a `analyzeResult`/`readResults`. La jerarquía JSON de palabras y líneas por página permanece sin cambios, por lo que no se requiere ningún cambio en el código.

La versión 3.0 de la API también presenta las siguientes mejoras que puede aprovechar opcionalmente. Consulte la referencia de la API para obtener más detalles:
* `createdDateTime` y `lastUpdatedDateTime` se agregan para que pueda realizar un seguimiento de la duración del procesamiento. Consulte la documentación para más detalles. 
* El elemento `version` indica la versión de la API usada para generar resultados.
* Se ha agregado el valor de `confidence` (confianza) por palabra. Este valor se calibra, de modo que un valor de 0,95 significa que hay un 95 % de probabilidad de que el reconocimiento sea correcto. La puntuación de confianza se puede usar para seleccionar el texto que se va a enviar para revisión humana. 
* La orientación general (`angle`) del texto va en el sentido de las agujas del reloj, medido en grados entre -180 y 180.
* `width` y `"height"` proporcionan las dimensiones del documento y `"unit"` proporciona la unidad de esas dimensiones (píxeles o pulgadas, en función del tipo de documento).
* Con `page` se admiten documentos de varias páginas.
* `language` es el idioma de entrada del documento (del parámetro opcional _language_).


En la versión 2.X, el formato de salida es el siguiente: 
    
```json
    {
        {
                "status": "Succeeded",
                "recognitionResult": [
                    {
                    "lines": [
                        {
                        "boundingBox": [
                            67,
                            646,
                            2582,
                            713,
                            2580,
                            876,
                            67,
                            821
                        ],
                        "text": "The quick brown fox jumps",
                        "words": [
                            {
                            "boundingBox": [
                                143,
                                650,
                                435,
                                661,
                                436,
                                823,
                                144,
                                824
                            ],
                            "text": "The",
                            },
            // The rest of result is omitted for brevity 
            
    }
```
    
En la versión 3.x, se ha ajustado:
    
```json
    {
        {
            "status": "succeeded",
            "createdDateTime": "2020-05-28T05:13:21Z",
            "lastUpdatedDateTime": "2020-05-28T05:13:22Z",
            "analyzeResult": {
            "version": "3.0.0",
            "readResults": [
                {
                "page": 1,
                "angle": 0.8551,
                "width": 2661,
                "height": 1901,
                "unit": "pixel",
                "lines": [
                    {
                    "boundingBox": [
                        67,
                        646,
                        2582,
                        713,
                        2580,
                        876,
                        67,
                        821
                    ],
                    "text": "The quick brown fox jumps",
                    "words": [
                        {
                        "boundingBox": [
                            143,
                            650,
                            435,
                            661,
                            436,
                            823,
                            144,
                            824
                        ],
                        "text": "The",
                        "confidence": 0.958
                        },
        // The rest of result is omitted for brevity 
        
    }
```

## <a name="container-only"></a>Solo el contenedor

### `Synchronous Read`

|Read 2.0 |Read 3.x  |
|----------|-----------|
|https://{endpoint}/vision/**v2.0/read/core/Analyze**     |https://{endpoint}/vision/<**version string**>/read/syncAnalyze[?language]|
