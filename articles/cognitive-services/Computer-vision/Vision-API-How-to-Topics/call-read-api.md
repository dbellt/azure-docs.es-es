---
title: Llamada a Read API
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo llamar a Read API y configurar su comportamiento en detalle.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 03/31/2021
ms.author: pafarley
ms.openlocfilehash: 54ec669984488d5fe429050faefa0ff2d2e8eb9b
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106287519"
---
# <a name="call-the-read-api"></a>Llamada a la API Read

En esta guía, aprenderá a llamar a Read API para extraer texto de imágenes. Conocerá las distintas maneras en que puede configurar el comportamiento de esta API para satisfacer sus necesidades.

## <a name="submit-data-to-the-service"></a>Envío de datos al servicio

La [llamada a Read](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-preview-3/operations/5d986960601faab4bf452005) de Read API toma como entrada una imagen o un documento PDF y extrae texto de forma asincrónica.

`https://{endpoint}/vision/v3.2-preview.3/read/analyze[?language][&pages][&readingOrder]`

La llamada se devuelve un campo de encabezado de respuesta denominado `Operation-Location`. El valor `Operation-Location` es una dirección URL que contiene el identificador de la operación que se va a usar en el paso siguiente.

|Encabezado de respuesta| Valor de ejemplo |
|:-----|:----|
|Operation-Location | `https://cognitiveservice/vision/v3.1/read/analyzeResults/49a36324-fc4b-4387-aa06-090cfbf0064f` |

> [!NOTE]
> **Facturación** 
>
> La página de [precios de Computer Vision](https://azure.microsoft.com/pricing/details/cognitive-services/computer-vision/) incluye el plan de tarifa de Read. Cada imagen o página analizada cuenta como una transacción. Si llama a la operación con un documento PDF o TIFF con 100 páginas, la operación de Read la contabilizará como 100 transacciones para su facturación. Si realizó 50 llamadas a la operación, y cada llamada envió un documento con 100 páginas, se facturarán 50x100 = 5000 transacciones.

## <a name="determine-how-to-process-the-data"></a>Determinación de cómo procesar los datos

### <a name="language-specification"></a>Especificación del lenguaje

La llamada a [Read](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/5d986960601faab4bf452005) tiene un parámetro de solicitud opcional para el idioma. La lectura admite la identificación automática del idioma y documentos multilingües, por lo que solo debe proporcionar un código de idioma si desea forzar el procesamiento del documento como ese idioma específico.

### <a name="natural-reading-order-output-latin-languages-only"></a>Salida de orden de lectura natural (solo idiomas procedentes del latín)
Con [Read API 3.2, versión preliminar](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-preview-3/operations/5d986960601faab4bf452005), puede especificar el orden en que se generan las líneas de texto con el parámetro de consulta `readingOrder`. Use `natural` si quiere obtener una salida de orden de lectura más natural, como se muestra en el ejemplo siguiente. Esta característica solo es compatible con los idiomas procedentes del latín.

:::image border type="content" source="../Images/ocr-reading-order-example.png" alt-text="Ejemplo de orden de lectura de OCR":::



### <a name="select-pages-or-page-ranges-for-text-extraction"></a>Selección de las páginas o de los intervalos de páginas para la extracción de texto
Con [Read API 3.2 versión preliminar](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-preview-3/operations/5d986960601faab4bf452005), en el caso de documentos grandes de varias páginas, use el parámetro de consulta `pages` para especificar números de página o intervalos de páginas para extraer texto solo de esas páginas. En el ejemplo siguiente se muestra un documento con 10 páginas, con texto extraído para ambos casos: todas las páginas (1-10) y las páginas seleccionadas (3-6).

:::image border type="content" source="../Images/ocr-select-pages.png" alt-text="Salida de páginas seleccionadas":::

## <a name="get-results-from-the-service"></a>Obtención de resultados a partir del servicio

El segundo paso es llamar a la operación [Get Read Results](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/5d9869604be85dee480c8750). Esta operación toma como entrada el identificador de operación que la operación de lectura ha creado. 

`https://{endpoint}/vision/v3.2-preview.3/read/analyzeResults/{operationId}`

Devuelve una respuesta JSON que contiene un campo de **estado** con los siguientes valores posibles. 

|Value | Significado |
|:-----|:----|
| `notStarted`| la operación no se ha iniciado. |
| `running`| la operación se está procesando. |
| `failed`| Se produjeron errores en la operación. |
| `succeeded`| La operación se ha realizado correctamente. |

Llamará a esta operación de forma iterativa hasta que se devuelva con el valor **correcto**. Use un intervalo de 1 a 2 segundos para evitar superar la tasa de solicitudes por segundo (RPS).

> [!NOTE]
> El nivel gratuito limita la tasa de solicitudes a 20 llamadas por minuto. El nivel de pago permite 10 solicitudes por segundo (RPS) que pueden aumentar a petición. Use el canal de soporte técnico de Azure o contacte con el equipo de cuentas para solicitar una tasa de solicitudes por segundo (RPS) más alta.

Cuando el campo **estado** tiene el valor `succeeded` (correcto), la respuesta JSON contiene el contenido de texto extraído de la imagen o el documento. La respuesta en JSON mantiene las agrupaciones de líneas originales de palabras reconocidas. Incluye las líneas de texto extraídas y las coordenadas del cuadro de límite correspondientes. Cada línea de texto incluye todas las palabras extraídas, con las coordenadas y las puntuaciones de confianza respectivas.

> [!NOTE]
> Los datos enviados a la operación `Read` se cifran temporalmente y se almacenan en reposo durante un corto tiempo y después se eliminan. Esto permite que las aplicaciones recuperen el texto extraído como parte de la respuesta del servicio.

### <a name="sample-json-output"></a>Salida de JSON de ejemplo

Consulte el siguiente ejemplo de una respuesta JSON correcta:

```json
{
  "status": "succeeded",
  "createdDateTime": "2020-05-28T05:13:21Z",
  "lastUpdatedDateTime": "2020-05-28T05:13:22Z",
  "analyzeResult": {
    "version": "3.1.0",
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
              }
            ]
          }
        ]
      }
    ]
  }
}
```

### <a name="handwritten-classification-for-text-lines-latin-languages-only"></a>Clasificación manuscrita de líneas de texto (solo para idiomas procedentes del latín)
La respuesta de [Read API 3.2 versión preliminar](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-preview-3/operations/5d986960601faab4bf452005) incluye la clasificación de si cada línea de texto es de estilo manuscrito o no, junto con una puntuación de confianza. Esta característica solo es compatible con los idiomas procedentes del latín. En el ejemplo siguiente se muestra la clasificación manuscrita del texto de la imagen.

:::image border type="content" source="../Images/ocr-handwriting-classification.png" alt-text="Ejemplo de clasificación de escritura a mano OCR":::

## <a name="next-steps"></a>Pasos siguientes

Para usar la API REST, vaya a la [referencia de Read API](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-preview-3/operations/5d986960601faab4bf452005).