---
title: 'Inicio rápido: API REST Form Recognizer'
description: Use la API REST Form Recognizer para crear una aplicación de procesamiento de formularios que extraiga pares clave-valor y datos de tabla de los documentos personalizados.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 05/12/2021
ms.author: lajanuar
ms.openlocfilehash: 1a18dbeb1c2a55c469b392cba0949d419d55ad18
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110374231"
---
<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD034 -->

> [!NOTE]
> En esta guía se usa cURL para ejecutar llamadas a la API REST.

|[API REST de Form Recognizer](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeWithCustomForm)|[Referencia de la API REST de Azure](/rest/api/azure/)|

## <a name="prerequisites"></a>Requisitos previos

* [cURL](https://curl.haxx.se/windows/) instalado.
* [PowerShell versión 6.0, o superior,](/powershell/scripting/install/installing-powershell-core-on-windows) o una aplicación de la línea de comandos similar.
* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* Un blob de Azure Storage que contenga un conjunto de datos de entrenamiento. Consulte [Creación de un conjunto de datos de aprendizaje para un modelo personalizado](../../build-training-data-set.md) para ver sugerencias y opciones para reunir el conjunto de datos de aprendizaje. En este inicio rápido puede usar los archivos de la carpeta **Train** del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451) (descargue y extraiga *sample_data.zip*).
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="cree un recurso de Form Recognizer"  target="_blank">create a Form Recognizer resource </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
  * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Form Recognizer API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
  * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* Una dirección URL para la imagen de un recibo. En este inicio rápido, puede usar una [imagen de ejemplo](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/contoso-allinone.jpg).
* La dirección URL de una imagen de una tarjeta de presentación. En este inicio rápido, puede usar una [imagen de ejemplo](https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/business_cards/business-card-english.jpg).
* La dirección URL de una imagen de un recibo. Puede usar un [documento de ejemplo](https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/forms/Invoice_1.pdf) para esta guía de inicio rápido.

## <a name="analyze-layout"></a>Análisis de diseño

Puede usar Form Recognizer para analizar y extraer tablas, marcas de selección, texto y estructuras de los documentos sin necesidad de entrenar un modelo. Para más información sobre la extracción del diseño, consulte la [guía conceptual sobre diseño](../../concept-layout.md). Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `{subscription key}` con la clave de suscripción que copió en el paso anterior.
1. Reemplace `\"{your-document-url}` por una de las direcciones URL de ejemplo.

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -v -i POST "https://{Endpoint}/formrecognizer/v2.1/layout/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{'source': '{your-document-url}'}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -v -i POST "https://{Endpoint}/formrecognizer/v2.0/layout/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{'source': '{your-document-url}'}"
```

---

Recibirá una respuesta `202 (Success)` que incluye un encabezado **Operation-Location**. El valor de este encabezado contiene un identificador de operación que puede usar para consultar el estado de la operación asincrónica y obtener los resultados. En el siguiente ejemplo, la cadena después de `analyzeResults/` es el identificador de operación.

```console
https://cognitiveservice/formrecognizer/v2/layout/analyzeResults/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-layout-results"></a>Obtención de los resultados del diseño

Tras la llamada a la API **[Analyze Layout](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeLayoutAsync)** , llame a la API **[Get Analyze Layout Result](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/GetAnalyzeLayoutResult)** para obtener el estado de la operación y los datos extraídos. Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `{subscription key}` con la clave de suscripción que copió en el paso anterior.
1. Reemplace `{resultId}` por el identificador de operación del paso anterior.
<!-- markdownlint-disable MD024 -->

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1/layout/analyzeResults/{resultId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.0/layout/analyzeResults/{resultId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

### <a name="examine-the-results"></a>Examen de los resultados

Recibirá una respuesta `200 (success)` con contenido JSON.

Consulte la siguiente imagen de una factura y su correspondiente salida JSON.

* El nodo `"readResults"` contiene cada línea de texto con su correspondiente posición de cuadro de límite en la página.
* El nodo `"selectionMarks"` (en la versión v2.1) muestra todas las casillas de selección (casilla, botón de opción) y si su estado es "activado" o "no activado".
* En la sección `"pageResults"` se incluyen las tablas extraídas. Para cada tabla, se extraen el texto, el índice de filas y columnas, la expansión de filas y columnas, el rectángulo de selección, etc.

:::image type="content" source="../../media/contoso-invoice.png" alt-text="Documento de instrucciones de proyecto de Contoso con una tabla.":::

### <a name="v21"></a>[v2.1](#tab/2-1)

Esta salida se ha abreviado por motivos de simplicidad. Consulte la [salida de ejemplo completa en GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/sample-layout-output.json).

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-08-20T20:40:50Z",
    "lastUpdatedDateTime": "2020-08-20T20:40:55Z",
    "analyzeResult": {
        "version": "2.1.0",
        "readResults": [
            {
                "page": 1,
                "angle": 0,
                "width": 8.5,
                "height": 11,
                "unit": "inch",
                "lines": [
                    {
                        "boundingBox": [
                            0.5826,
                            0.4411,
                            2.3387,
                            0.4411,
                            2.3387,
                            0.7969,
                            0.5826,
                            0.7969
                        ],
                        "text": "Contoso, Ltd.",
                        "words": [
                            {
                                "boundingBox": [
                                    0.5826,
                                    0.4411,
                                    1.744,
                                    0.4411,
                                    1.744,
                                    0.7969,
                                    0.5826,
                                    0.7969
                                ],
                                "text": "Contoso,",
                                "confidence": 1
                            },
                            {
                                "boundingBox": [
                                    1.8448,
                                    0.4446,
                                    2.3387,
                                    0.4446,
                                    2.3387,
                                    0.7631,
                                    1.8448,
                                    0.7631
                                ],
                                "text": "Ltd.",
                                "confidence": 1
                            }
                        ]
                    },
                    ...
                        ]
                    }
                ],
                "selectionMarks": [
                    {
                        "boundingBox": [
                            3.9737,
                            3.7475,
                            4.1693,
                            3.7475,
                            4.1693,
                            3.9428,
                            3.9737,
                            3.9428
                        ],
                        "confidence": 0.989,
                        "state": "selected"
                    },
                    ...
                ]
            }
        ],
        "pageResults": [
            {
                "page": 1,
                "tables": [
                    {
                        "rows": 5,
                        "columns": 5,
                        "cells": [
                            {
                                "rowIndex": 0,
                                "columnIndex": 0,
                                "text": "Training Date",
                                "boundingBox": [
                                    0.5133,
                                    4.2167,
                                    1.7567,
                                    4.2167,
                                    1.7567,
                                    4.4492,
                                    0.5133,
                                    4.4492
                                ],
                                "elements": [
                                    "#/readResults/0/lines/12/words/0",
                                    "#/readResults/0/lines/12/words/1"
                                ]
                            },
                            ...
                        ]
                    },
                    ...
                ]
            }
        ]
    }
}
```

### <a name="v20"></a>[v2.0](#tab/2-0)

Esta salida se ha abreviado por motivos de simplicidad. Consulte la [salida de ejemplo completa en GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/sample-layout-output.json).

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-08-20T20:36:52Z",
    "lastUpdatedDateTime": "2020-08-20T20:36:58Z",
    "analyzeResult": {
        "version": "2.0.0",
        "readResults": [
            {
                "page": 1,
                "language": "en",
                "angle": 0,
                "width": 8.5,
                "height": 11,
                "unit": "inch",
                "lines": [
                    {
                        "boundingBox": [
                            0.5826,
                            0.4411,
                            2.3387,
                            0.4411,
                            2.3387,
                            0.7969,
                            0.5826,
                            0.7969
                        ],
                        "text": "Contoso, Ltd.",
                        "words": [
                            {
                                "boundingBox": [
                                    0.5826,
                                    0.4411,
                                    1.744,
                                    0.4411,
                                    1.744,
                                    0.7969,
                                    0.5826,
                                    0.7969
                                ],
                                "text": "Contoso,",
                                "confidence": 1
                            },
                            {
                                "boundingBox": [
                                    1.8448,
                                    0.4446,
                                    2.3387,
                                    0.4446,
                                    2.3387,
                                    0.7631,
                                    1.8448,
                                    0.7631
                                ],
                                "text": "Ltd.",
                                "confidence": 1
                            }
                        ]
                    },
                    ...
                ]
            }
        ],
        "pageResults": [
            {
                "page": 1,
                "tables": [
                    {
                        "rows": 5,
                        "columns": 5,
                        "cells": [
                            {
                                "rowIndex": 0,
                                "columnIndex": 0,
                                "text": "Training Date",
                                "boundingBox": [
                                    0.5133,
                                    4.2167,
                                    1.7567,
                                    4.2167,
                                    1.7567,
                                    4.4492,
                                    0.5133,
                                    4.4492
                                ],
                                "elements": [
                                    "#/readResults/0/lines/14/words/0",
                                    "#/readResults/0/lines/14/words/1"
                                ]
                            },
                            ...
                        ]
                    },
                    ...
                ]
            }
        ]
    }
}
```

---

## <a name="analyze-receipts"></a>Análisis de las confirmaciones de recepción

En esta sección se muestra cómo analizar y extraer campos comunes de recibos de EE. UU. mediante un modelo de recibos entrenado previamente. Para más información sobre el análisis de recibos, consulte la [guía conceptual sobre recibos](../../concept-receipts.md). Para empezar a analizar un recibo, llame a la API **[Analyze Receipt](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeReceiptAsync)** con el siguiente comando cURL. Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `{your receipt URL}` por la dirección URL de una imagen del recibo.
1. Reemplace `{subscription key>` con la clave de suscripción que copió en el paso anterior.

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.1/prebuilt/receipt/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{your receipt URL}'}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.0/prebuilt/receipt/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{your receipt URL}'}"
```

---

Recibirá una respuesta `202 (Success)` que incluye un encabezado **Operation-Location**. El valor de este encabezado contiene un identificador de operación que puede usar para consultar el estado de la operación asincrónica y obtener los resultados. En el siguiente ejemplo, la cadena después de `operations/` es el identificador de operación.

```console
https://cognitiveservice/formrecognizer/v2.0/prebuilt/receipt/operations/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-the-receipt-results"></a>Obtención de los resultados del recibo

Tras la llamada a la API **Analyze Receipt**, llame a la API **[Get Receipt Result](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/GetAnalyzeReceiptResult)** para obtener el estado de la operación y los datos extraídos. Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la clave de suscripción de Form Recognizer. Lo encontrará en la pestaña **Información general** del recurso Form Recognizer.
1. Reemplace `{operationId}` por el identificador de operación del paso anterior.
1. Reemplace `{subscription key}` por la clave de suscripción.

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.1/prebuilt/receipt/analyzeResults/{operationId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.0/prebuilt/receipt/analyzeResults/{operationId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

### <a name="examine-the-response"></a>Examen de la respuesta

Recibirá una respuesta `200 (Success)` con la salida JSON. El primer campo, `"status"`, indica el estado de la operación. Si la operación no está completa, el valor de `"status"` será `"running"` o `"notStarted"`, y debe llamar a la API de nuevo, ya sea manualmente o a través de un script. Se recomienda un intervalo de uno o varios segundos entre llamadas.

El nodo `"readResults"` contiene todo el texto reconocido (si el parámetro *includeTextDetails* opcional se ha establecido en `true`). El texto se organiza por página, después, por líneas y, finalmente, por palabras individuales. El nodo `"documentResults"` contiene los valores específicos del recibo que el modelo ha descubierto. Aquí encontrará pares clave-valor útiles como los impuestos, el total, la dirección del vendedor, etc.

Consulte la siguiente imagen del recibo y su correspondiente salida JSON.

![Un recibo del almacén de Contoso](../../media/contoso-allinone.jpg)

Esta salida se ha abreviado para que se pueda leer mejor. Consulte la [salida de ejemplo completa en GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/receipt-result.json).

```json
{
  "status":"succeeded",
  "createdDateTime":"2019-12-17T04:11:24Z",
  "lastUpdatedDateTime":"2019-12-17T04:11:32Z",
  "analyzeResult":{
    "version":"2.1.0",
    "readResults":[
      {
        "page":1,
        "angle":0.6893,
        "width":1688,
        "height":3000,
        "unit":"pixel",
        "language":"en",
        "lines":[
          {
            "text":"Contoso",
            "boundingBox":[
              635,
              510,
              1086,
              461,
              1098,
              558,
              643,
              604
            ],
            "words":[
              {
                "text":"Contoso",
                "boundingBox":[
                  639,
                  510,
                  1087,
                  461,
                  1098,
                  551,
                  646,
                  604
                ],
                "confidence":0.955
              }
            ]
          },
          ...
        ]
      }
    ],
    "documentResults":[
      {
        "docType":"prebuilt:receipt",
        "pageRange":[
          1,
          1
        ],
        "fields":{
          "ReceiptType":{
            "type":"string",
            "valueString":"Itemized",
            "confidence":0.692
          },
          "MerchantName":{
            "type":"string",
            "valueString":"Contoso Contoso",
            "text":"Contoso Contoso",
            "boundingBox":[
              378.2,
              292.4,
              1117.7,
              468.3,
              1035.7,
              812.7,
              296.3,
              636.8
            ],
            "page":1,
            "confidence":0.613,
            "elements":[
              "#/readResults/0/lines/0/words/0",
              "#/readResults/0/lines/1/words/0"
            ]
          },
          "MerchantAddress":{
            "type":"string",
            "valueString":"123 Main Street Redmond, WA 98052",
            "text":"123 Main Street Redmond, WA 98052",
            "boundingBox":[
              302,
              675.8,
              848.1,
              793.7,
              809.9,
              970.4,
              263.9,
              852.5
            ],
            "page":1,
            "confidence":0.99,
            "elements":[
              "#/readResults/0/lines/2/words/0",
              "#/readResults/0/lines/2/words/1",
              "#/readResults/0/lines/2/words/2",
              "#/readResults/0/lines/3/words/0",
              "#/readResults/0/lines/3/words/1",
              "#/readResults/0/lines/3/words/2"
            ]
          },
          "MerchantPhoneNumber":{
            "type":"phoneNumber",
            "valuePhoneNumber":"+19876543210",
            "text":"987-654-3210",
            "boundingBox":[
              278,
              1004,
              656.3,
              1054.7,
              646.8,
              1125.3,
              268.5,
              1074.7
            ],
            "page":1,
            "confidence":0.99,
            "elements":[
              "#/readResults/0/lines/4/words/0"
            ]
          },
          "TransactionDate":{
            "type":"date",
            "valueDate":"2019-06-10",
            "text":"6/10/2019",
            "boundingBox":[
              265.1,
              1228.4,
              525,
              1247,
              518.9,
              1332.1,
              259,
              1313.5
            ],
            "page":1,
            "confidence":0.99,
            "elements":[
              "#/readResults/0/lines/5/words/0"
            ]
          },
          "TransactionTime":{
            "type":"time",
            "valueTime":"13:59:00",
            "text":"13:59",
            "boundingBox":[
              541,
              1248,
              677.3,
              1261.5,
              668.9,
              1346.5,
              532.6,
              1333
            ],
            "page":1,
            "confidence":0.977,
            "elements":[
              "#/readResults/0/lines/5/words/1"
            ]
          },
          "Items":{
            "type":"array",
            "valueArray":[
              {
                "type":"object",
                "valueObject":{
                  "Quantity":{
                    "type":"number",
                    "text":"1",
                    "boundingBox":[
                      245.1,
                      1581.5,
                      300.9,
                      1585.1,
                      295,
                      1676,
                      239.2,
                      1672.4
                    ],
                    "page":1,
                    "confidence":0.92,
                    "elements":[
                      "#/readResults/0/lines/7/words/0"
                    ]
                  },
                  "Name":{
                    "type":"string",
                    "valueString":"Cappuccino",
                    "text":"Cappuccino",
                    "boundingBox":[
                      322,
                      1586,
                      654.2,
                      1601.1,
                      650,
                      1693,
                      317.8,
                      1678
                    ],
                    "page":1,
                    "confidence":0.923,
                    "elements":[
                      "#/readResults/0/lines/7/words/1"
                    ]
                  },
                  "TotalPrice":{
                    "type":"number",
                    "valueNumber":2.2,
                    "text":"$2.20",
                    "boundingBox":[
                      1107.7,
                      1584,
                      1263,
                      1574,
                      1268.3,
                      1656,
                      1113,
                      1666
                    ],
                    "page":1,
                    "confidence":0.918,
                    "elements":[
                      "#/readResults/0/lines/8/words/0"
                    ]
                  }
                }
              },
              ...
            ]
          },
          "Subtotal":{
            "type":"number",
            "valueNumber":11.7,
            "text":"11.70",
            "boundingBox":[
              1146,
              2221,
              1297.3,
              2223,
              1296,
              2319,
              1144.7,
              2317
            ],
            "page":1,
            "confidence":0.955,
            "elements":[
              "#/readResults/0/lines/13/words/1"
            ]
          },
          "Tax":{
            "type":"number",
            "valueNumber":1.17,
            "text":"1.17",
            "boundingBox":[
              1190,
              2359,
              1304,
              2359,
              1304,
              2456,
              1190,
              2456
            ],
            "page":1,
            "confidence":0.979,
            "elements":[
              "#/readResults/0/lines/15/words/1"
            ]
          },
          "Tip":{
            "type":"number",
            "valueNumber":1.63,
            "text":"1.63",
            "boundingBox":[
              1094,
              2479,
              1267.7,
              2485,
              1264,
              2591,
              1090.3,
              2585
            ],
            "page":1,
            "confidence":0.941,
            "elements":[
              "#/readResults/0/lines/17/words/1"
            ]
          },
          "Total":{
            "type":"number",
            "valueNumber":14.5,
            "text":"$14.50",
            "boundingBox":[
              1034.2,
              2617,
              1387.5,
              2638.2,
              1380,
              2763,
              1026.7,
              2741.8
            ],
            "page":1,
            "confidence":0.985,
            "elements":[
              "#/readResults/0/lines/19/words/0"
            ]
          }
        }
      }
    ]
  }
}
```

## <a name="analyze-business-cards"></a>Análisis de tarjetas de presentación

### <a name="v21"></a>[v2.1](#tab/2-1)

En esta sección se muestra cómo analizar y extraer campos comunes de tarjetas de presentación inglesas mediante un modelo entrenado previamente. Para más información acerca del análisis de tarjetas de presentación, consulte la [guía conceptual sobre tarjetas de presentación](../../concept-business-cards.md). Para empezar a analizar una tarjeta de presentación, llame a la API **[Analyze Business Card](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeBusinessCardAsync)** con el siguiente comando cURL. Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `{your business card URL}` por la dirección URL de una imagen del recibo.
1. Reemplace `{subscription key}` con la clave de suscripción que copió en el paso anterior.

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.1/prebuilt/businessCard/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{your receipt URL}'}"
```

Recibirá una respuesta `202 (Success)` que incluye un encabezado **Operation-Location**. El valor de este encabezado contiene un identificador de operación que puede usar para consultar el estado de la operación asincrónica y obtener los resultados.

```console
https://cognitiveservice/formrecognizer/v2.1/prebuilt/businessCard/analyzeResults/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-business-card-results"></a>Obtención de los resultados de la tarjeta de presentación

Tras la llamada a la API **Analyze Business Card**, llame a la API **[Get Analyze Business Card Result](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/GetAnalyzeBusinessCardResult)** para obtener el estado de la operación y los datos extraídos. Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la clave de suscripción de Form Recognizer. Lo encontrará en la pestaña **Información general** del recurso Form Recognizer.
1. Reemplace `{resultId}` por el identificador de operación del paso anterior.
1. Reemplace `{subscription key}` por la clave de suscripción.

```bash
curl -v -X GET "https://westcentralus.api.cognitive.microsoft.com/formrecognizer/v2.1/prebuilt/businessCard/analyzeResults/{resultId}"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="examine-the-response"></a>Examen de la respuesta

Recibirá una respuesta `200 (Success)` con la salida JSON.

El nodo `"readResults"` contiene todo el texto reconocido. El texto se organiza por página, después, por líneas y, finalmente, por palabras individuales. El nodo `"documentResults"` contiene los valores específicos de la tarjeta de presentción que el modelo haya descubierto. Aquí encontrará información de contacto útil, como el nombre de la empresa, el nombre de pila, el apellido, el número de teléfono, etc.

![Una tarjeta de presentación de la compañía Contoso](../../media/business-card-english.jpg)

Esta salida JSON de ejemplo se ha abreviado para que se pueda leer mejor. Consulte la [salida de ejemplo completa en GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/business-card-result.json).

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-06-04T08:19:29Z",
    "lastUpdatedDateTime": "2020-06-04T08:19:35Z",
    "analyzeResult": {
        "version": "2.1.1",
        "readResults": [
            {
                "page": 1,
                "angle": -17.0956,
                "width": 4032,
                "height": 3024,
                "unit": "pixel"
            }
        ],
        "documentResults": [
            {
                "docType": "prebuilt:businesscard",
                "pageRange": [
                    1,
                    1
                ],
                "fields": {
                    "ContactNames": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "object",
                                "valueObject": {
                                    "FirstName": {
                                        "type": "string",
                                        "valueString": "Avery",
                                        "text": "Avery",
                                        "boundingBox": [
                                            703,
                                            1096,
                                            1134,
                                            989,
                                            1165,
                                            1109,
                                            733,
                                            1206
                                        ],
                                        "page": 1
                                },
                                "text": "Dr. Avery Smith",
                                "boundingBox": [
                                    419.3,
                                    1154.6,
                                    1589.6,
                                    877.9,
                                    1618.9,
                                    1001.7,
                                    448.6,
                                    1278.4
                                ],
                                "confidence": 0.993
                            }
                        ]
                    },
                    "Emails": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "avery.smith@contoso.com",
                                "text": "avery.smith@contoso.com",
                                "boundingBox": [
                                    2107,
                                    934,
                                    2917,
                                    696,
                                    2935,
                                    764,
                                    2126,
                                    995
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Websites": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "https://www.contoso.com/",
                                "text": "https://www.contoso.com/",
                                "boundingBox": [
                                    2121,
                                    1002,
                                    2992,
                                    755,
                                    3014,
                                    826,
                                    2143,
                                    1077
                                ],
                                "page": 1,
                                "confidence": 0.995
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```

El script imprimirá las respuestas en la consola hasta que se complete la operación de **Analyze Business Card**.

### <a name="v20"></a>[v2.0](#tab/2-0)

> [!IMPORTANT]
> Esta característica no está disponible en la versión de API seleccionada.

---

## <a name="analyze-invoices"></a>Análisis de facturas

### <a name="v21"></a>[v2.1](#tab/2-1)

Para empezar a analizar una factura, use el siguiente comando de cURL. Para más información sobre el análisis de facturas, consulte la [guía conceptual sobre facturas](../../concept-invoices.md). Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `{your invoice URL}` por la dirección URL del documento de una factura.
1. Reemplace `{subscription key}` por la clave de suscripción.

```bash
curl -v -i POST "https://{Endpoint}/formrecognizer/v2.1/prebuilt/invoice/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key:  {subscription key}" --data-ascii "{'source': '{your invoice URL}'}"
```

Recibirá una respuesta `202 (Success)` que incluye un encabezado **Operation-Location**. El valor de este encabezado contiene un identificador de operación que puede usar para consultar el estado de la operación asincrónica y obtener los resultados.

```console
https://cognitiveservice/formrecognizer/v2.1/prebuilt/invoice/analyzeResults/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-invoice-results"></a>Obtención de los resultados de la factura

Tras la llamada a la **[API Analyze Invoice](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/5ed8c9843c2794cbb1a96291)** , llame a la **[API Get Analyze Invoice Result](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/5ed8c9acb78c40a2533aee83)** para obtener el estado de la operación y los datos extraídos. Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la clave de suscripción de Form Recognizer. Lo encontrará en la pestaña **Información general** del recurso Form Recognizer.
1. Reemplace `{resultId}` por el identificador de operación del paso anterior.
1. Reemplace `{subscription key}` por la clave de suscripción.

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1/prebuilt/invoice/analyzeResults/{resultId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="examine-the-response"></a>Examen de la respuesta

Recibirá una respuesta `200 (Success)` con la salida JSON.

* El campo `"readResults"` contiene todas las líneas de texto que se extrajeron de la factura.
* El campo `"pageResults"` incluye las marcas de tablas y selecciones extraídas de la factura.
* El campo `"documentResults"` contiene información de pares clave-valor para las partes más importantes de la factura.

Consulte la siguiente factura y su correspondiente salida JSON.

* [Factura de ejemplo](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/curl/form-recognizer/sample-invoice.pdf)

Este contenido JSON se ha abreviado para facilitar la lectura. Consulte la [salida de ejemplo completa en GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/sample-invoice-output.json).

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-11-06T23:32:11Z",
    "lastUpdatedDateTime": "2020-11-06T23:32:20Z",
    "analyzeResult": {
        "version": "2.1.0",
        "readResults": [{
            "page": 1,
            "angle": 0,
            "width": 8.5,
            "height": 11,
            "unit": "inch"
        }],
        "pageResults": [{
            "page": 1,
            "tables": [{
                "rows": 3,
                "columns": 4,
                "cells": [{
                    "rowIndex": 0,
                    "columnIndex": 0,
                    "text": "QUANTITY",
                    "boundingBox": [0.4953,
                    5.7306,
                    1.8097,
                    5.7306,
                    1.7942,
                    6.0122,
                    0.4953,
                    6.0122]
                },
                {
                    "rowIndex": 0,
                    "columnIndex": 1,
                    "text": "DESCRIPTION",
                    "boundingBox": [1.8097,
                    5.7306,
                    5.7529,
                    5.7306,
                    5.7452,
                    6.0122,
                    1.7942,
                    6.0122]
                },
                ...
                ],
                "boundingBox": [0.4794,
                5.7132,
                8.0158,
                5.714,
                8.0118,
                6.5627,
                0.4757,
                6.5619]
            },
            {
                "rows": 2,
                "columns": 6,
                "cells": [{
                    "rowIndex": 0,
                    "columnIndex": 0,
                    "text": "SALESPERSON",
                    "boundingBox": [0.4979,
                    4.963,
                    1.8051,
                    4.963,
                    1.7975,
                    5.2398,
                    0.5056,
                    5.2398]
                },
                {
                    "rowIndex": 0,
                    "columnIndex": 1,
                    "text": "P.O. NUMBER",
                    "boundingBox": [1.8051,
                    4.963,
                    3.3047,
                    4.963,
                    3.3124,
                    5.2398,
                    1.7975,
                    5.2398]
                },
                ...
                ],
                "boundingBox": [0.4976,
                4.961,
                7.9959,
                4.9606,
                7.9959,
                5.5204,
                0.4972,
                5.5209]
            }]
        }],
        "documentResults": [{
            "docType": "prebuilt:invoice",
            "pageRange": [1,
            1],
            "fields": {
                "AmountDue": {
                    "type": "number",
                    "valueNumber": 610,
                    "text": "$610.00",
                    "boundingBox": [7.3809,
                    7.8153,
                    7.9167,
                    7.8153,
                    7.9167,
                    7.9591,
                    7.3809,
                    7.9591],
                    "page": 1,
                    "confidence": 0.875
                },
                "BillingAddress": {
                    "type": "string",
                    "valueString": "123 Bill St, Redmond WA, 98052",
                    "text": "123 Bill St, Redmond WA, 98052",
                    "boundingBox": [0.594,
                    4.3724,
                    2.0125,
                    4.3724,
                    2.0125,
                    4.7125,
                    0.594,
                    4.7125],
                    "page": 1,
                    "confidence": 0.997
                },
                "BillingAddressRecipient": {
                    "type": "string",
                    "valueString": "Microsoft Finance",
                    "text": "Microsoft Finance",
                    "boundingBox": [0.594,
                    4.1684,
                    1.7907,
                    4.1684,
                    1.7907,
                    4.2837,
                    0.594,
                    4.2837],
                    "page": 1,
                    "confidence": 0.998
                },
                ...
            }
        }]
    }
}
```

### <a name="v20"></a>[v2.0](#tab/2-0)

> [!IMPORTANT]
> Esta característica no está disponible en la versión de API seleccionada.

---

## <a name="analyze-identity-documents"></a>Análisis de documentos de identidad

### <a name="v21"></a>[v2.1](#tab/2-1)

Para empezar a analizar un documento de identificación, use el siguiente comando de cURL. Para más información sobre el análisis de documentos de identificación, consulte la [guía conceptual de documentos de identificación](../../concept-identification-cards.md). Para empezar a analizar un documento de identificación, llame a la API **Analyze ID Document https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/5f74a7738978e467c5fb8707)** mediante el comando de cURL siguiente. Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{endpoint}` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `{your ID document URL}` por la dirección URL de una imagen del recibo.
1. Reemplace `{subscription key}` con la clave de suscripción que copió en el paso anterior.

```bash
curl -i -X POST "https://{endpoint}/formrecognizer/v2.1/prebuilt/idDocument/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{your  ID document URL}'}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

> [!IMPORTANT]
> Esta característica no está disponible en la versión de API seleccionada.

---

Recibirá una respuesta `202 (Success)` que incluye un encabezado **Operation-Location**. El valor de este encabezado contiene un identificador de resultado que puede usar para consultar el estado de la operación asincrónica y obtener los resultados. En el siguiente ejemplo, la cadena después de `analyzeResults/` es el identificador de resultado.

```console
https://westus.api.cognitive.microsoft.com/formrecognizer/v2.1/prebuilt/idDocument/analyzeResults/0c6cb19e-538f-4b8d-98b7-e105c9995ba6
```

### <a name="get-the-analyze-id-document-result"></a>Obtención del resultado de Analyze ID Document

Tras la llamada a la API **Analyze ID Document**, llame a la API **[Get Analyze Id Document Result](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/5f74a7daad1f2612c46f5822)** para obtener el estado de la operación y los datos extraídos.  Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{endpoint}` por el punto de conexión que obtuvo con la clave de suscripción de Form Recognizer. Lo encontrará en la pestaña **Información general** del recurso Form Recognizer.
1. Reemplace `{resultId}` por el identificador de operación del paso anterior.
1. Reemplace `{subscription key}` por la clave de suscripción.

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -X GET "https://{endpoint}/formrecognizer/v2.1/prebuilt/idDocument/analyzeResults/{resultId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="examine-the-response"></a>Examen de la respuesta

Recibirá una respuesta `200 (Success)` con la salida JSON. El primer campo, `"status"`, indica el estado de la operación. Si la operación no está completa, el valor de `"status"` será `"running"` o `"notStarted"`, y debe llamar a la API de nuevo, ya sea manualmente o mediante un script hasta que reciba el valor `succeeded`. Se recomienda un intervalo de uno o varios segundos entre llamadas.

* El campo `"readResults"` contiene todas las líneas de texto que se extrajeron del documento de identificación.
* El campo `"documentResults"` contiene una matriz de objetos, donde cada uno de los cuales representa un documento de identificación detectado en el documento de entrada.

A continuación se muestra un documento de identificación de ejemplo y su salida JSON correspondiente

* :::image type="content" source="https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/id-license.jpg" alt-text="permiso de conducir de ejemplo":::

```json
{
    "status": "succeeded",
    "createdDateTime": "2021-04-13T17:24:52Z",
    "lastUpdatedDateTime": "2021-04-13T17:24:55Z",
    "analyzeResult": {
      "version": "2.1.0",
      "readResults": [
        {
          "page": 1,
          "angle": -0.2823,
          "width": 450,
          "height": 294,
          "unit": "pixel"
        }
      ],
      "documentResults": [
        {
          "docType": "prebuilt:idDocument:driverLicense",
          "docTypeConfidence": 0.995,
          "pageRange": [
            1,
            1
          ],
          "fields": {
            "Address": {
              "type": "string",
              "valueString": "123 STREET ADDRESS YOUR CITY WA 99999-1234",
              "text": "123 STREET ADDRESS YOUR CITY WA 99999-1234",
              "boundingBox": [
                158,
                151,
                326,
                151,
                326,
                177,
                158,
                177
              ],
              "page": 1,
              "confidence": 0.965
            },
            "Country": {
              "type": "country",
              "valueCountry": "USA",
              "confidence": 0.99
            },
            "DateOfBirth": {
              "type": "date",
              "valueDate": "1958-01-06",
              "text": "01/06/1958",
              "boundingBox": [
                187,
                133,
                272,
                132,
                272,
                148,
                187,
                149
              ],
              "page": 1,
              "confidence": 0.99
            },
            "DateOfExpiration": {
              "type": "date",
              "valueDate": "2020-08-12",
              "text": "08/12/2020",
              "boundingBox": [
                332,
                230,
                414,
                228,
                414,
                244,
                332,
                245
              ],
              "page": 1,
              "confidence": 0.99
            },
            "DocumentNumber": {
              "type": "string",
              "valueString": "LICWDLACD5DG",
              "text": "LIC#WDLABCD456DG",
              "boundingBox": [
                162,
                70,
                307,
                68,
                307,
                84,
                163,
                85
              ],
              "page": 1,
              "confidence": 0.99
            },
            "FirstName": {
              "type": "string",
              "valueString": "LIAM R.",
              "text": "LIAM R.",
              "boundingBox": [
                158,
                102,
                216,
                102,
                216,
                116,
                158,
                116
              ],
              "page": 1,
              "confidence": 0.985
            },
            "LastName": {
              "type": "string",
              "valueString": "TALBOT",
              "text": "TALBOT",
              "boundingBox": [
                160,
                86,
                213,
                85,
                213,
                99,
                160,
                100
              ],
              "page": 1,
              "confidence": 0.987
            },
            "Region": {
              "type": "string",
              "valueString": "Washington",
              "confidence": 0.99
            },
            "Sex": {
              "type": "gender",
              "valueGender": "M",
              "text": "M",
              "boundingBox": [
                226,
                190,
                232,
                190,
                233,
                201,
                226,
                201
              ],
              "page": 1,
              "confidence": 0.99
            }
          }
        }
      ]
    }
  }
```

### <a name="v20"></a>[v2.0](#tab/2-0)

> [!IMPORTANT]
> Esta característica no está disponible en la versión de API seleccionada.

---

## <a name="train-a-custom-model"></a>Entrenamiento de un modelo personalizado

Para entrenar un modelo personalizado, necesitará un conjunto de datos de entrenamiento en un blob de Azure Storage. Necesita un mínimo de cinco formularios rellenados (documentos PDF o imágenes) de la misma estructura y tipo. Consulte [Build a training data set for a custom model](../../build-training-data-set.md) (Creación de un conjunto de datos de aprendizaje para un modelo personalizado) para ver sugerencias y opciones para reunir los datos de entrenamiento.

El entrenamiento sin datos etiquetados es la operación predeterminada y la más sencilla. De forma alternativa, puede etiquetar manualmente algunos o todos los datos de entrenamiento de antemano. Se trata de un proceso más complejo, pero el resultado es un modelo mejor entrenado.

> [!NOTE]
> También puede entrenar modelos con una interfaz gráfica de usuario, como la [herramienta de etiquetado de ejemplo de Form Recognizer](../../label-tool.md).

### <a name="train-a-model-without-labels"></a>Entrenamiento de un modelo sin etiquetas

Para entrenar un modelo de Form Recognizer con los documentos del contenedor de blobs de Azure, llame a la API **[Train Custom Model](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/TrainCustomModelAsync)** mediante la ejecución del siguiente comando de cURL. Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `{subscription key}` con la clave de suscripción que copió en el paso anterior.
1. Reemplace `{SAS URL}` por la dirección URL de la firma de acceso compartido (SAS) del contenedor de almacenamiento de blobs de Azure. [!INCLUDE [get SAS URL](../../includes/sas-instructions.md)]

   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="Recuperación de la dirección URL de SAS":::

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.1/custom/models" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{SAS URL}'}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.0/custom/models" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{SAS URL}'}"
```

---

Recibirá una respuesta `201 (Success)` con un encabezado **Location** (Ubicación). El valor de este encabezado es el identificador del nuevo modelo que se va a entrenar.

### <a name="train-a-model-with-labels"></a>Entrenamiento de un modelo con etiquetas

Para realizar el entrenamiento con etiquetas, debe disponer de archivos de información con etiquetas especiales (`\<filename\>.pdf.labels.json`) en el contenedor de almacenamiento de blobs junto con los documentos para el entrenamiento. La [herramienta de etiquetado de ejemplo de Form Recognizer](../../label-tool.md) proporciona una interfaz de usuario para ayudarle a crear estos archivos de etiqueta. Una vez que los tenga, puede llamar a la API **[Train Custom Model](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/TrainCustomModelAsync)** con el parámetro `"useLabelFile"` establecido en `true` en el cuerpo JSON.

Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `{subscription key}` con la clave de suscripción que copió en el paso anterior.
1. Reemplace `{SAS URL}` por la dirección URL de la firma de acceso compartido (SAS) del contenedor de almacenamiento de blobs de Azure. [!INCLUDE [get SAS URL](../../includes/sas-instructions.md)]

   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="Recuperación de la dirección URL de SAS":::

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.1/custom/models" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{SAS URL}', 'useLabelFile':true}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.0/custom/models" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ 'source': '{SAS URL}', 'useLabelFile':true }"
```

---

Recibirá una respuesta `201 (Success)` con un encabezado **Location** (Ubicación). El valor de este encabezado es el identificador del nuevo modelo que se va a entrenar.

### <a name="get-training-results"></a>Obtención de los resultados del entrenamiento

Una vez iniciada la operación de entrenamiento, utilizará una nueva operación, **[Obtención del modelo personalizado](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/GetCustomModel)** , para comprobar el estado del entrenamiento. Pase el identificador de modelo a esta llamada API para comprobar el estado del entrenamiento:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la clave de suscripción de Form Recognizer.
1. Reemplace `{subscription key}` por la clave de suscripción.
1. Reemplace `{model ID}` por el identificador de modelo que recibió en el paso anterior.

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.1/custom/models/{model ID}" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models/{model ID}" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

Recibirá una respuesta `200 (Success)` con un cuerpo JSON con el formato siguiente: Observe el campo `"status"`. Este tendrá el valor `"ready"` una vez que se complete el entrenamiento. Si el modelo no ha finalizado el entrenamiento, tendrá que consultar de nuevo el servicio volviendo a ejecutar el comando. Se recomienda un intervalo de uno o varios segundos entre llamadas.

El campo `"modelId"` contiene el identificador del modelo que se está entrenando. Lo necesitará para el siguiente paso.

```json
{
  "modelInfo":{
    "status":"ready",
    "createdDateTime":"2019-10-08T10:20:31.957784",
    "lastUpdatedDateTime":"2019-10-08T14:20:41+00:00",
    "modelId":"1cfb372bab404ba3aa59481ab2c63da5"
  },
  "trainResult":{
    "trainingDocuments":[
      {
        "documentName":"invoices\\Invoice_1.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_2.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_3.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_4.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_5.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      }
    ],
    "errors":[

    ]
  },
  "keys":{
    "0":[
      "Address:",
      "Invoice For:",
      "Microsoft",
      "Page"
    ]
  }
}
```

## <a name="analyze-forms-with-a-custom-model"></a>Analizar formularios con un modelo personalizado

A continuación, utilizará el modelo recién entrenado para analizar un documento y extraerá de él tanto los pares clave-valor como las tablas. Llame a la API **[Analyze Form](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeWithCustomForm)** mediante la ejecución del siguiente comando cURL. Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo de la clave de suscripción de Form Recognizer. Lo encontrará en la pestaña **Información general** del recurso Form Recognizer.
1. Reemplace `{model ID}` por el identificador de modelo que recibió en la sección anterior.
1. Reemplace `{SAS URL}` por una dirección URL de SAS en el archivo de Azure Storage. Siga los pasos de la sección de entrenamiento, pero en lugar de obtener una dirección URL de SAS para todo el contenedor de blobs, obtenga una para el archivo específico que desea analizar.
1. Reemplace `{subscription key}` por la clave de suscripción.

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -v "https://{Endpoint}/formrecognizer/v2.1/custom/models/{model ID}/analyze?includeTextDetails=true" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" -d "{ 'source': '{SAS URL}' } "
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -v "https://{Endpoint}/formrecognizer/v2.0/custom/models/{model ID}/analyze?includeTextDetails=true" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" -d "{ 'source': '{SAS URL}' } "
```

---

Recibirá una respuesta `202 (Success)` con un encabezado **Operation-Location** (Operación-ubicación). El valor de este encabezado incluye un identificador de resultados que se usa para realizar el seguimiento de los resultados de la operación de análisis. Guarde este identificador de resultados para el siguiente paso.

### <a name="get-the-analyze-results"></a>Obtención de los resultados del análisis

Llame a la API **[Get Analyze Form Result](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/GetAnalyzeFormResult)** para consultar los resultados de la operación de análisis.

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo de la clave de suscripción de Form Recognizer. Lo encontrará en la pestaña **Información general** del recurso Form Recognizer.
1. Reemplace `{result ID}` por el identificador que recibió en la sección anterior.
1. Reemplace `{subscription key}` por la clave de suscripción.

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.1/custom/models/{model ID}/analyzeResults/{result ID}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models/{model ID}/analyzeResults/{result ID}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

Recibirá una respuesta `200 (Success)` con un cuerpo JSON con el formato siguiente: La salida se ha abreviado por motivos de simplicidad. Observe el campo `"status"` situado cerca de la parte inferior. Este tendrá el valor `"succeeded"` cuando se complete la operación de análisis. Si la operación de análisis no se ha completado, tendrá que consultar de nuevo el servicio volviendo a ejecutar el comando. Se recomienda un intervalo de uno o varios segundos entre llamadas.

En los modelos personalizados entrenados sin etiquetas, las tablas y asociaciones de pares clave-valor se encuentran en el nodo `"pageResults"` de la salida JSON. En los modelos personalizados entrenados con etiquetas, las asociaciones de pares clave-valor se encuentran en el nodo `"documentResults"`. Si también especificó la extracción de texto sin formato mediante el parámetro de URL *includeTextDetails*, el nodo `"readResults"` mostrará el contenido y las posiciones de todo el texto del documento.

Esta salida JSON de ejemplo se ha abreviado por motivos de simplicidad. Consulte la [salida de ejemplo completa en GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/analyze-result-invoice-6.pdf.json).

### <a name="v21"></a>[v2.1](#tab/2-1)

```JSON
{
  "status": "succeeded",
  "createdDateTime": "2020-08-21T01:13:28Z",
  "lastUpdatedDateTime": "2020-08-21T01:13:42Z",
  "analyzeResult": {
    "version": "2.1.0",
    "readResults": [
      {
        "page": 1,
        "angle": 0,
        "width": 8.5,
        "height": 11,
        "unit": "inch",
        "lines": [
          {
            "text": "Project Statement",
            "boundingBox": [
              5.0444,
              0.3613,
              8.0917,
              0.3613,
              8.0917,
              0.6718,
              5.0444,
              0.6718
            ],
            "words": [
              {
                "text": "Project",
                "boundingBox": [
                  5.0444,
                  0.3587,
                  6.2264,
                  0.3587,
                  6.2264,
                  0.708,
                  5.0444,
                  0.708
                ]
              },
              {
                "text": "Statement",
                "boundingBox": [
                  6.3361,
                  0.3635,
                  8.0917,
                  0.3635,
                  8.0917,
                  0.6396,
                  6.3361,
                  0.6396
                ]
              }
            ]
          },
          ...
        ]
      }
    ],
    "pageResults": [
      {
        "page": 1,
        "keyValuePairs": [
          {
            "key": {
              "text": "Date:",
              "boundingBox": [
                6.9833,
                1.0615,
                7.3333,
                1.0615,
                7.3333,
                1.1649,
                6.9833,
                1.1649
              ],
              "elements": [
                "#/readResults/0/lines/2/words/0"
              ]
            },
            "value": {
              "text": "9/10/2020",
              "boundingBox": [
                7.3833,
                1.0802,
                7.925,
                1.0802,
                7.925,
                1.174,
                7.3833,
                1.174
              ],
              "elements": [
                "#/readResults/0/lines/3/words/0"
              ]
            },
            "confidence": 1
          },
          ...
        ],
        "tables": [
          {
            "rows": 5,
            "columns": 5,
            "cells": [
              {
                "text": "Training Date",
                "rowIndex": 0,
                "columnIndex": 0,
                "boundingBox": [
                  0.6944,
                  4.2779,
                  1.5625,
                  4.2779,
                  1.5625,
                  4.4005,
                  0.6944,
                  4.4005
                ],
                "confidence": 1,
                "rowSpan": 1,
                "columnSpan": 1,
                "elements": [
                  "#/readResults/0/lines/15/words/0",
                  "#/readResults/0/lines/15/words/1"
                ],
                "isHeader": true,
                "isFooter": false
              },
              ...
            ]
          }
        ],
        "clusterId": 0
      }
    ],
    "documentResults": [],
    "errors": []
  }
}
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```JSON
{
  "status": "succeeded",
  "createdDateTime": "2020-08-21T00:46:25Z",
  "lastUpdatedDateTime": "2020-08-21T00:46:32Z",
  "analyzeResult": {
    "version": "2.0.0",
    "readResults": [
      {
        "page": 1,
        "angle": 0,
        "width": 8.5,
        "height": 11,
        "unit": "inch",
        "lines": [
          {
            "text": "Project Statement",
            "boundingBox": [
              5.0153,
              0.275,
              8.0944,
              0.275,
              8.0944,
              0.7125,
              5.0153,
              0.7125
            ],
            "words": [
              {
                "text": "Project",
                "boundingBox": [
                  5.0153,
                  0.275,
                  6.2278,
                  0.275,
                  6.2278,
                  0.7125,
                  5.0153,
                  0.7125
                ]
              },
              {
                "text": "Statement",
                "boundingBox": [
                  6.3292,
                  0.275,
                  8.0944,
                  0.275,
                  8.0944,
                  0.7125,
                  6.3292,
                  0.7125
                ]
              }
            ]
          },
        ...
        ]
      }
    ],
    "pageResults": [
      {
        "page": 1,
        "keyValuePairs": [
          {
            "key": {
              "text": "Date:",
              "boundingBox": [
                6.9722,
                1.0264,
                7.3417,
                1.0264,
                7.3417,
                1.1931,
                6.9722,
                1.1931
              ],
              "elements": [
                "#/readResults/0/lines/2/words/0"
              ]
            },
            "confidence": 1
          },
         ...
        ],
        "tables": [
          {
            "rows": 4,
            "columns": 5,
            "cells": [
              {
                "text": "Training Date",
                "rowIndex": 0,
                "columnIndex": 0,
                "boundingBox": [
                  0.6931,
                  4.2444,
                  1.5681,
                  4.2444,
                  1.5681,
                  4.4125,
                  0.6931,
                  4.4125
                ],
                "confidence": 1,
                "rowSpan": 1,
                "columnSpan": 1,
                "elements": [
                  "#/readResults/0/lines/15/words/0",
                  "#/readResults/0/lines/15/words/1"
                ],
                "isHeader": true,
                "isFooter": false
              },
              ...
            ]
          }
        ],
        "clusterId": 0
      }
    ],
    "documentResults": [],
    "errors": []
  }
}
```

---

### <a name="improve-results"></a>Mejora de los resultados

[!INCLUDE [improve results](../../includes/improve-results-unlabeled.md)]

## <a name="manage-custom-models"></a>Administración de modelos personalizados

### <a name="get-a-list-of-custom-models"></a>Obtener una lista de modelos personalizados

Use la API **[List Custom Models](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/GetCustomModels)** del comando siguiente para devolver una lista de todos los modelos personalizados que pertenecen a la suscripción.

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `{subscription key}` con la clave de suscripción que copió en el paso anterior.

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1/custom/models?op=full"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models?op=full"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

Recibirá una respuesta `200` correcta, con datos JSON como los siguientes. El elemento `"modelList"` contiene todos los modelos creados y su información.

```json
{
  "summary": {
    "count": 0,
    "limit": 0,
    "lastUpdatedDateTime": "string"
  },
  "modelList": [
    {
      "modelId": "string",
      "status": "creating",
      "createdDateTime": "string",
      "lastUpdatedDateTime": "string"
    }
  ],
  "nextLink": "string"
}
```

### <a name="get-a-specific-model"></a>Obtención de un modelo específico

Para recuperar información detallada sobre un modelo personalizado específico, use la API **[Get Custom Model](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/GetCustomModel)** en el siguiente comando.

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `{subscription key}` con la clave de suscripción que copió en el paso anterior.
1. Reemplace `{modelId}` por el identificador del modelo personalizado que desea buscar.

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1/custom/models/{modelId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models/{modelId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

Recibirá una respuesta `200` correcta, con datos JSON como los siguientes.

```json
{
  "modelInfo": {
    "modelId": "string",
    "status": "creating",
    "createdDateTime": "string",
    "lastUpdatedDateTime": "string"
  },
  "keys": {
    "clusters": {}
  },
  "trainResult": {
    "trainingDocuments": [
      {
        "documentName": "string",
        "pages": 0,
        "errors": [
          "string"
        ],
        "status": "succeeded"
      }
    ],
    "fields": [
      {
        "fieldName": "string",
        "accuracy": 0.0
      }
    ],
    "averageModelAccuracy": 0.0,
    "errors": [
      {
        "message": "string"
      }
    ]
  }
}
```

### <a name="delete-a-model-from-the-resource-account"></a>Eliminación de un modelo de la cuenta de recursos

También puede eliminar un modelo de su cuenta haciendo referencia a su identificador. Este comando llama a la API **[Delete Custom Model](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/DeleteCustomModel)** API para eliminar el modelo usado en la sección anterior.
código

1. Reemplace `{Endpoint}` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `{subscription key}` con la clave de suscripción que copió en el paso anterior.
1. Reemplace `{modelId}` por el identificador del modelo personalizado que desea buscar.

### <a name="v21"></a>[v2.1](#tab/2-1)

```bash
curl -v -X DELETE "https://{Endpoint}/formrecognizer/v2.1/custom/models/{modelId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="v20"></a>[v2.0](#tab/2-0)

```bash
curl -v -X DELETE "https://{Endpoint}/formrecognizer/v2.0/custom/models/{modelId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```

---

Recibirá una respuesta `204` de operación correcta que indica que el modelo está marcado para su eliminación. Los artefactos del modelo se eliminarán en un plazo máximo de 48 horas.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha usado la API REST de Form Recognizer para entrenar modelos y analizar formularios de maneras diferentes. A continuación, consulte la documentación de referencia para explorar la API de Form Recognizer con mayor profundidad.

> [!div class="nextstepaction"]
> [Documentación de referencia de API REST](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeWithCustomForm)

* [¿Qué es Form Recognizer?](../../overview.md)
