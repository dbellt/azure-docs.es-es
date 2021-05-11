---
title: 'Recibos: Form Recognizer'
titleSuffix: Azure Cognitive Services
description: 'Conozca los conceptos relacionados con el análisis de recibos mediante Form Recognizer API: uso y límites.'
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 04/30/2021
ms.author: lajanuar
ms.openlocfilehash: 96625959c089c46b04b13216bbb9ea4b74ef4feb
ms.sourcegitcommit: dd425ae91675b7db264288f899cff6add31e9f69
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2021
ms.locfileid: "108331869"
---
# <a name="form-recognizer-prebuilt-receipt-model"></a>Modelo de recibo creado previamente de Form Recognizer

Azure Form Recognizer puede analizar y extraer información de recibos de ventas mediante sus modelos de recibos creados previamente. Combina nuestras eficaces funcionalidades de [reconocimiento óptico de caracteres (OCR)](../computer-vision/overview-ocr.md) con modelos de aprendizaje profundo para extraer información clave de recibos escritos en inglés.

## <a name="understanding-receipts"></a>Descripción de los recibos

Muchas empresas y usuarios siguen confiando en los datos extraídos manualmente de los recibos de compra. La extracción automática de datos de dichos recibos puede resultar complicada. Los recibos pueden estar arrugados, ser difíciles de leer, tener partes manuscritas y contener imágenes de smartphone de baja calidad. Además, las plantillas y los campos de los recibos pueden variar considerablemente según el mercado, la región y el comerciante. Estos desafíos en la extracción de datos y la detección de campos hacen que el procesamiento de recibos sea un problema único.

La API Receipt usa el reconocimiento óptico de caracteres (OCR) y nuestro modelo creado previamente para habilitar escenarios de procesamiento de recibos amplios. Con la API Receipt no es necesario entrenar un modelo. Tan solo hay que enviar la imagen del recibo a la API de análisis de recibos para extraer los datos.

![recibo de ejemplo](./media/receipts-example.jpg)


## <a name="what-does-the-receipt-service-do"></a>¿Qué hace el servicio de recibos?

El servicio de recibos pregenerado extrae el contenido de los recibos de venta: el tipo de recibo que se obtendría normalmente en un restaurante, una tienda minorista o una tienda de comestibles.

### <a name="fields-extracted"></a>Campos extraídos

|Nombre| Tipo | Descripción | Texto | Valor (salida estándar) |
|:-----|:----|:----|:----| :----|
| ReceiptType | string | Tipo de recibo de venta | Detallados |  |
| MerchantName | string | Nombre del comerciante que emite el recibo | Contoso |  |
| MerchantPhoneNumber | phoneNumber | Número de teléfono mostrado del comerciante | 987-654-3210 | +19876543210 |
| MerchantAddress | string | Dirección mostrada del comerciante | 123 Main St Redmond WA 98052 |  |
| Fecha de transacción | date | Fecha de emisión del recibo | 06 de junio de 2019 | 2019-06-26  |
| TransactionTime | time | Hora de emisión del recibo | 4:49 pm | 16:49:00  |
| Total | number | Número total de transacciones de recibos | $14.34 | 14,34 |
| Subtotal | number | Subtotal del recibo, a menudo antes de aplicar impuestos | $12.34 | 12.34 |
| Impuesto | number | Impuestos sobre el recibo o equivalentes | $2.00 | 2.00 |
| Sugerencia | number | Propina incluida por el comprador | $1.00 | 1.00 |
| Elementos | matriz de objetos | Líneas del artículo extraídas, con el nombre, la cantidad, el precio por unidad y el precio total extraídos | |
| Nombre | string | Nombre del elemento | Surface Pro 6 | |
| Cantidad | number | Cantidad de cada artículo | 1 | |
| Precio | number | Precio individual de cada unidad del artículo | $999.00 | 999.00 |
| Precio total | number | Precio total del artículo | $999.00 | 999.00 |

### <a name="additional-features"></a>Características adicionales

Receipt API también devuelve la siguiente información:

* Nivel de confianza de campo (cada campo devuelve un valor de confianza asociado)
* Texto sin formato OCR (salida de texto extraído mediante OCR para todo el recibo)
* Rectángulo delimitador de cada valor, apunte y palabra

## <a name="try-it-out"></a>Prueba

Para probar el servicio de recibos de Form Recognizer, vaya a la herramienta de interfaz de usuario de ejemplo en línea:

> [!div class="nextstepaction"]
> [Uso de modelos precompilados](https://fott-preview.azurewebsites.net/)

## <a name="input-requirements"></a>Requisitos de entrada

[!INCLUDE [input requirements](./includes/input-requirements-receipts.md)]

## <a name="supported-locales"></a>Configuraciones regionales admitidas

* **Pre-built receipt v2.0** (GA) admite recibos de ventas en la configuración regional **en-us**.
* **Pre-built receipt v2.1-preview.3** (versión preliminar pública) agrega compatibilidad adicional para las siguientes configuraciones locales en inglés de los recibos:

* **en-au**
* **en-ca**
* **en-gb**
* **en-in**

  > [!NOTE]
  > Entrada de idioma
  >
  > La versión precompilada de Receipt v2.1-preview.3 tiene un parámetro de solicitud opcional para especificar una configuración regional de recibos en mercados en inglés adicionales. En el caso de los recibos de ventas en inglés de Australia (EN-AU), Canadá (EN-CA), Gran Bretaña (EN-GB) e India (EN-IN), puede especificar la configuración regional para obtener resultados mejorados. Si no se especifica ninguna configuración regional en v2.1-preview.3, el modelo se establecerá de forma predeterminada en EN-US.


## <a name="the-analyze-receipt-operation"></a>La operación Analyze Receipt

[Analyze Receipt](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeReceiptAsync) toma una imagen o PDF de un recibo como entrada y extrae los valores de interés y el texto. La llamada devuelve un campo de encabezado de respuesta denominado `Operation-Location`. El valor `Operation-Location` es una dirección URL que contiene el id. de resultado que se va a usar en el paso siguiente.

|Encabezado de respuesta| Dirección URL del resultado |
|:-----|:----|
|Operation-Location | `https://cognitiveservice/formrecognizer/v2.0/prebuilt/receipt/analyzeResults/56a36454-fc4d-4354-aa07-880cfbf0064f` |

## <a name="the-get-analyze-receipt-result-operation"></a>La operación Get Analyze Receipt Result

El segundo paso consiste en llamar a la operación [Get Analyze Receipt Result](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/GetAnalyzeReceiptResult). Esta operación toma como entrada el id. de resultado que la operación Analyze Receipt ha creado. Devuelve una respuesta JSON que contiene un campo de **estado** con los siguientes valores posibles. Llamará a esta operación de forma iterativa hasta que se devuelva con el valor **correcto**. Use un intervalo de 3 a 5 segundos para evitar superar la tasa de solicitudes por segundo (RPS).

|Campo| Tipo | Valores posibles |
|:-----|:----:|:----|
|status | string | notStarted: la operación no se ha iniciado. |
| |  | running: la operación de análisis está en curso. |
| |  | failed: error en la operación de análisis. |
| |  | succeeded: la operación de análisis se realizó correctamente. |

Cuando el campo **status** tenga el valor **succeeded**, la respuesta JSON incluirá los resultados de la descripción del recibo y el reconocimiento del texto. El resultado del reconocimiento de los recibos se organiza como un diccionario de valores de campos con nombre. Cada valor contiene el texto extraído, el valor normalizado, el rectángulo delimitador, el nivel de confianza y los elementos de la palabra correspondiente. El resultado del reconocimiento de texto se organiza como una jerarquía de líneas y palabras, con texto, rectángulo delimitador e información de confianza.

![resultados de ejemplo del recibo](./media/contoso-receipt-2-information.png)

### <a name="sample-json-output"></a>Salida de JSON de ejemplo


La respuesta a la operación Get Analyze Receipt Result será la representación estructurada del recibo con toda la información extraída.  Aquí encontrará un [archivo con un recibo de ejemplo](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/contoso-allinone.jpg) y su salida estructurada, una [salida de un recibo de ejemplo](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/receipt-result.json).

Consulte el siguiente ejemplo de una respuesta JSON correcta:
* El nodo `"readResults"` contiene todo el texto reconocido. El texto se organiza por página, después, por líneas y, finalmente, por palabras individuales.
* El nodo `"documentResults"` contiene los valores específicos de la tarjeta de presentción que el modelo haya descubierto. Aquí encontrará pares clave-valor útiles, como el nombre, el apellido, el nombre de la empresa y más.

```json
{
  "status":"succeeded",
  "createdDateTime":"2019-12-17T04:11:24Z",
  "lastUpdatedDateTime":"2019-12-17T04:11:32Z",
  "analyzeResult":{
    "version":"2.0.0",
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

## <a name="customer-scenarios"></a>Escenarios de cliente

Los datos extraídos con Receipt API se pueden usar para realizar varias tareas. A continuación se muestran algunos ejemplos de lo que nuestros clientes han realizado con la API Receipt.

### <a name="business-expense-reporting"></a>Informe de gastos empresariales

A menudo, la tramitación de gastos empresariales implica dedicar tiempo a introducir datos de las imágenes de los recibos de forma manual. Con Receipt API, puede usar los campos extraídos para automatizar parcialmente este proceso y analizar sus recibos rápidamente.

Dado que la API Receipt tiene una salida JSON sencilla, puede usar los valores de campo extraídos de varias maneras. Intégrelos con las aplicaciones de gastos internas para rellenar previamente los informes de gastos. Para obtener más información sobre este escenario, consulte cómo Acumatica usa Receipt API para [facilitar el proceso de los informes de gastos](https://customers.microsoft.com/story/762684-acumatica-partner-professional-services-azure).

### <a name="auditing-and-accounting"></a>Auditoría y contabilidad

El resultado de Receipt API también se puede usar para realizar el análisis de un gran número de gastos en distintos puntos del proceso de informes y reembolso de los gastos. Puede procesar los recibos para evaluarlos para auditorías manuales o aprobaciones rápidas.

El resultado de Receipt también es útil para el registro contable general para su uso en la empresa o a nivel personal. Use Receipt API para transformar cualquier dato de PDF o de imágenes de recibos sin formato en una salida digital que se pueda procesar.

### <a name="consumer-behavior"></a>Comportamiento del consumidor

Los recibos contienen datos útiles que puede utilizar para analizar el comportamiento del consumidor y las tendencias de compra.

Receipt API también contribuye a la [característica de procesamiento de recibos de AI Builder](/ai-builder/prebuilt-receipt-processing).

## <a name="next-steps"></a>Pasos siguientes

 .Empiece a escribir una aplicación de procesamiento de formularios con Form Recognizer en el lenguaje de desarrollo que prefiera.

> [!div class="nextstepaction"]
> [Completar un inicio rápido de Form Recognizer](quickstarts/client-library.md)

## <a name="see-also"></a>Consulte también

* [¿Qué es Form Recognizer?](overview.md)
* [Referencia de la API Form Recognizer](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeReceiptAsync)
>
