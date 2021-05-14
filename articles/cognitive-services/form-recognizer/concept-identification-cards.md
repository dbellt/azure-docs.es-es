---
title: 'ID: Form Recognizer'
titleSuffix: Azure Cognitive Services
description: Conozca los conceptos relacionados con la extracción de datos de los documentos de identidad con la API de identificadores precompilados de Form Recognizer.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 04/30/2021
ms.author: lajanuar
ms.openlocfilehash: d59df677fda920be5ed9547bee3855d4c9511187
ms.sourcegitcommit: dd425ae91675b7db264288f899cff6add31e9f69
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2021
ms.locfileid: "108330843"
---
# <a name="form-recognizer-prebuilt-identification-id-document-model"></a>Modelo de documento de identificación (ID) precompilado de Form Recognizer

Azure Form Recognizer puede analizar y extraer información de documentos de identificación (ID) expedidos por la Administración mediante sus modelos de ID precompilados. Combina nuestras eficaces funcionalidades de [reconocimiento óptico de caracteres (OCR)](../computer-vision/overview-ocr.md) con capacidades de reconocimiento de ID para extraer información clave de pasaportes de todo el mundo y de permisos de conducir de EE. UU. (de los 50 estados y del D.C.). La API de ID extrae información clave de estos documentos de identidad, como el nombre, el apellido, la fecha de nacimiento, el número del documento, etc. Esta API está disponible en la versión preliminar de Form Recognizer v2.1 como un servicio en la nube y como un contenedor local.

## <a name="what-does-the-id-service-do"></a>¿Qué hace el servicio de ID?

El servicio de identificaciones precompiladas extrae los principales valores de pasaportes de todo el mundo y permisos de conducir de EE. UU. y los devuelve en una respuesta JSON estructurada organizada.

### <a name="drivers-license-example"></a>**Ejemplo de permiso de conducir**

![Permiso de conducir de ejemplo](./media/id-example-drivers-license.JPG)

### <a name="passport-example"></a>**Ejemplo de pasaporte**

![Pasaporte de ejemplo](./media/id-example-passport-result.JPG)

### <a name="fields-extracted"></a>Campos extraídos

|Nombre| Tipo | Descripción | Value |
|:-----|:----|:----|:----|
|  País | country | Código de país conforme con la norma ISO 3166 estándar. | "USA" |
|  DateOfBirth | date | Fecha de nacimiento en formato AAAA-MM-DD. | "1980-01-01" |
|  DateOfExpiration | date | Fecha de expiración en formato AAAA-MM-DD. | "2019-05-05" |
|  DocumentNumber | string | Número de pasaporte, número de permiso de conducir, etc. pertinente. | "340020013" |
|  FirstName | string | Nombre extraído e inicial del segundo nombre, si procede. | "JENNIFER" |
|  LastName | string | Apellido extraído | "BROOKS" |
|  Nacionalidad | country | Código de país conforme con la norma ISO 3166 estándar. | "USA" |
|  Sex | gender | Los valores extraídos posibles son "M", "F" y "X". | "F" |
|  MachineReadableZone | object | Zona de lectura automática del pasaporte extraída, incluidas dos líneas de 44 caracteres cada una. | "P<USABROOKS<<JENNIFER<<<<<<<<<<<<<<<<<<<<<<< 3400200135USA8001014F1905054710000307<715816" |
|  DocumentType | string | Tipo de documento, por ejemplo, pasaporte o permiso de conducir. | "passport" |
|  Dirección | string | Dirección extraída (solo permiso de conducir) | "123 STREET ADDRESS YOUR CITY WA 99999-1234"|
|  Region (Región) | string | Valor extraído de región, estado, provincia, etc. (solo permiso de conducir) | "Washington" |

### <a name="additional-features"></a>Características adicionales

La API de ID también devuelve la siguiente información:

* Nivel de confianza de campo (cada campo devuelve un valor de confianza asociado)
* Texto sin formato OCR (salida de texto extraído mediante OCR para todo el recibo)
* Cuadro de límite de cada campo extraído en los permisos de conducir de EE. UU.
* Cuadro de límite para la zona de lectura automática (MRZ) en pasaportes.

  > [!NOTE]
  > Las identificaciones precompiladas no detectan la autenticidad del ID.
  >
  > Las identificaciones precompiladas de Form Recognizer extraen información clave de los datos de identificador. Sin embargo, no detectan la validez ni la autenticidad del documento de identidad original.

## <a name="try-it-out"></a>Prueba

Para probar el servicio de ID de Form Recognizer, vaya a la herramienta de interfaz de usuario de ejemplo en línea:

> [!div class="nextstepaction"]
> [Uso de modelos precompilados](https://fott-preview.azurewebsites.net/)

## <a name="input-requirements"></a>Requisitos de entrada

[!INCLUDE [input requirements](./includes/input-requirements-receipts.md)]

## <a name="supported-locales"></a>Configuraciones regionales admitidas

 **Identificador precompilado v2.1-preview.3** (versión preliminar) admite documentos de identidad en la configuración regional **en-us**.

## <a name="supported-identity-document-types"></a>Tipos de documentos de identidad admitidos

* **Identificaciones precompiladas, v2.1-preview.3** Extrae valores clave de pasaportes de todo el mundo y de permisos de conducir de EE. UU.

  > [!NOTE]
  > Compatibilidad con el tipo de ID.
  >
  > Los tipos de ID admitidos actualmente son los pasaportes de todo el mundo y los permisos de conducir de EE. UU. Estamos tratando de ampliar nuestra compatibilidad con identificaciones a otros documentos de identidad de todo el mundo.

## <a name="post-analyze-id-document"></a>Aplicación de POST a documento de Analyze ID

La operación [Analyze ID](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/5f74a7daad1f2612c46f5822) toma una imagen o un archivo PDF de una ID como entrada y extrae los valores de interés. La llamada devuelve un campo de encabezado de respuesta denominado `Operation-Location`. El valor `Operation-Location` es una dirección URL que contiene el id. de resultado que se va a usar en el paso siguiente.

|Encabezado de respuesta| Dirección URL del resultado |
|:-----|:----|
|Operation-Location | `https://cognitiveservice/formrecognizer/v2.1-preview.3/prebuilt/idDocument/analyzeResults/49a36324-fc4b-4387-aa06-090cfbf0064f` |

## <a name="get-analyze-id-document-result"></a>Aplicación de GET a resultado de documento de Analyze Id

<!---
Need to update this with updated APIM links when available
-->

El segundo paso consiste en llamar a la operación [**Get Analyze idDocument Result**](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/5f74a7738978e467c5fb8707). Esta operación toma como entrada el identificador de resultado que la operación Analyze ID ha creado. Devuelve una respuesta JSON que contiene un campo de **estado** con los siguientes valores posibles. Llamará a esta operación de forma iterativa hasta que se devuelva con el valor **correcto**. Use un intervalo de 3 a 5 segundos para evitar superar la tasa de solicitudes por segundo (RPS).

|Campo| Tipo | Valores posibles |
|:-----|:----:|:----|
|status | string | notStarted: la operación de análisis no se ha iniciado. |
| |  | running: la operación de análisis está en curso. |
| |  | failed: error en la operación de análisis. |
| |  | succeeded: la operación de análisis se realizó correctamente. |

Cuando el campo **status** tenga el valor **succeeded**, la respuesta JSON incluirá los resultados de la descripción del recibo y el reconocimiento del texto. El resultado de la ID se organiza como un diccionario de valores de campo con nombre, en el que cada valor contiene el texto extraído, el valor normalizado, el rectángulo delimitador, el nivel de confianza y los elementos de la palabra correspondiente. El resultado del reconocimiento de texto se organiza como una jerarquía de líneas y palabras, con texto, rectángulo delimitador e información de confianza.

![resultados de ejemplo del recibo](./media/id-example-passport-result.JPG)

### <a name="sample-json-output"></a>Salida de JSON de ejemplo

Consulte el siguiente ejemplo de una respuesta JSON correcta: El nodo `readResults` contiene todo el texto reconocido. El texto se organiza por página, después, por líneas y, finalmente, por palabras individuales. El nodo `documentResults` contiene los valores específicos de ID que el modelo ha descubierto. Este nodo es también donde encontrará pares clave-valor útiles, como el nombre, el apellido, el número de documento, etc.

```json
{
   "status": "succeeded",
  "createdDateTime": "2021-03-04T22:29:33Z",
  "lastUpdatedDateTime": "2021-03-04T22:29:36Z",
  "analyzeResult": {
    "version": "2.1.0",
    "readResults": [
     {
        "page": 1,
        "angle": 0.3183,
        "width": 549,
        "height": 387,
        "unit": "pixel",
        "lines": [
          {
            "text": "PASSPORT",
            "boundingBox": [
              57,
              10,
              120,
              11,
              119,
              22,
              57,
              22
            ],
            "words": [
              {
                "text": "PASSPORT",
                "boundingBox": [
                  57,
                  11,
                  119,
                  11,
                  118,
                  23,
                  57,
                  22
                ],
                "confidence": 0.994
              }
            ],
          ...
      }
    ],

     "documentResults": [
      {
        "docType": "prebuilt:idDocument:passport",
        "docTypeConfidence": 0.995,
        "pageRange": [
          1,
          1
        ],
        "fields": {
          "Country": {
            "type": "country",
            "valueCountry": "USA",
            "text": "USA"
          },
          "DateOfBirth": {
            "type": "date",
            "valueDate": "1980-01-01",
            "text": "800101"
          },
          "DateOfExpiration": {
            "type": "date",
            "valueDate": "2019-05-05",
            "text": "190505"
          },
          "DocumentNumber": {
            "type": "string",
            "valueString": "340020013",
            "text": "340020013"
          },
          "FirstName": {
            "type": "string",
            "valueString": "JENNIFER",
            "text": "JENNIFER"
          },
          "LastName": {
            "type": "string",
            "valueString": "BROOKS",
            "text": "BROOKS"
          },
          "Nationality": {
            "type": "country",
            "valueCountry": "USA",
            "text": "USA"
          },
          "Sex": {
            "type": "gender",
            "valueGender": "F",
            "text": "F"
          },
          "MachineReadableZone": {
            "type": "object",
            "text": "P<USABROOKS<<JENNIFER<<<<<<<<<<<<<<<<<<<<<<< 3400200135USA8001014F1905054710000307<715816",
            "boundingBox": [
              16,
              314.1,
              504.2,
              317,
              503.9,
              363,
              15.7,
              360.1
            ],
            "page": 1,
            "confidence": 0.384,
            "elements": [
              "#/readResults/0/lines/33/words/0",
              "#/readResults/0/lines/33/words/1",
              "#/readResults/0/lines/33/words/2",
              "#/readResults/0/lines/33/words/3",
              "#/readResults/0/lines/33/words/4",
              "#/readResults/0/lines/34/words/0"
            ]
          },
          "DocumentType": {
            "type": "string",
            "text": "passport",
            "confidence": 0.995
          }
        }
      }
    ]
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

* Pruebe sus propios ID y ejemplos en la [interfaz de usuario de ejemplo de Form Recognizer](https://fott-preview.azurewebsites.net/).
* Realice un [inicio rápido de Form Recognizer](quickstarts/client-library.md) para empezar a escribir una aplicación de procesamiento de ID con Form Recognizer en el lenguaje de desarrollo que prefiera.

## <a name="see-also"></a>Consulte también

* [**¿Qué es Form Recognizer?**](./overview.md)
* [**Documentos de referencia de la API de REST**](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeWithCustomForm)
