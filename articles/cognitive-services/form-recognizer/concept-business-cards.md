---
title: 'Tarjetas de presentación: Form Recognizer'
titleSuffix: Azure Applied AI Services
description: 'Conozca los conceptos relacionados con el análisis de tarjetas de presentación con la API Form Recognizer: uso y límites.'
services: cognitive-services
author: laujan
manager: nitinme
ms.service: applied-ai-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 04/30/2021
ms.author: lajanuar
ms.openlocfilehash: 92817318108370f1edf9ca2b38bf01226612b53a
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111890787"
---
# <a name="form-recognizer-prebuilt-business-cards-model"></a>Modelo de tarjetas de presentación pregenerado de Form Recognizer

Azure Form Recognizer puede analizar y extraer información de contacto de tarjetas de presentación mediante alguno de los modelos pregenerados. Combina funcionalidades eficaces de reconocimiento óptico de caracteres (OCR) con nuestro modelo de comprensión de tarjetas de presentación para extraer información clave de las tarjetas de presentación en inglés. Extrae información de contacto personal, el nombre de la empresa, el puesto y mucho más. La API de tarjeta de visita compilada previamente está disponible públicamente en la versión Form Recognizer v2.1.

## <a name="customer-scenarios"></a>Escenarios de cliente

Los datos extraídos con Business Card API se pueden usar para realizar varias tareas. La extracción de esta información de contacto supone automáticamente un ahorro de tiempo para los usuarios en el caso de los roles orientados al cliente. A continuación se muestran algunos ejemplos de lo que nuestros clientes han realizado con la API de tarjeta de presentación:

* Extraer información de contacto de tarjetas de presentación y crear contactos de teléfono rápidamente.
* Integrarse con CRM para crear automáticamente el contacto mediante las imágenes de la tarjeta de presentación.
* Realizar un seguimiento de los clientes potenciales.
* Extraer información de contacto en masa de imágenes de tarjetas de presentación existentes.

Business Card API también contribuye a la [característica de procesamiento de tarjetas de presentación de AI Builder](/ai-builder/prebuilt-business-card).


## <a name="try-it-out"></a>Prueba

Para probar el servicio de recibos de Form Recognizer, vaya a la herramienta de interfaz de usuario de ejemplo en línea:

> [!div class="nextstepaction"]
> [Uso de modelos precompilados](https://aka.ms/fott-2.1-ga)

## <a name="what-does-the-business-card-service-do"></a>¿Qué hace el servicio de tarjetas de presentación?

La API de tarjeta de presentación pregenerada extrae campos clave de las tarjetas de presentación y los devuelve en una respuesta JSON organizada.

![Imagen de Contoso detallada de FOTT y salida JSON](./media/business-card-example.jpg)

### <a name="fields-extracted"></a>Campos extraídos:

|Nombre| Tipo | Descripción | Texto | Valor (salida estándar) |
|:-----|:----|:----|:----|:----|
| ContactNames | matriz de objetos | Nombre de contacto extraído de la tarjeta de presentación | [{ "FirstName": "Chris", "LastName": "Smith" }] | |
| FirstName | string | Nombre (de nacimiento) del contacto | "Chris" | "Chris" |
| LastName | string | Apellidos del contacto |     "Smith" | "Smith" |
| CompanyNames | Matriz de cadenas | Nombre de la empresa extraído de la tarjeta de presentación | ["CONTOSO"] | CONTOSO |
| Departments | Matriz de cadenas | Departamento u organización del contacto | ["Cloud & Al Department"] | Departamento de la nube e inteligencia artificial |
| JobTitles | Matriz de cadenas | Puesto mostrado del contacto | ["Senior Researcher"] | Investigador sénior |
| Correos electrónicos | Matriz de cadenas | Dirección de correo electrónico de contacto extraída de la tarjeta de presentación | ["chris.smith@contoso.com"] | chris.smith@contoso.com |
| Websites | Matriz de cadenas | Sitio web extraído de la tarjeta de presentación | ["https://www.contoso.com"] | https://www.contoso.com |
| Direcciones | Matriz de cadenas | Dirección extraída de la tarjeta de presentación | ["4001 1st Ave NE Redmond, WA 98052"] | 4001 1st Ave NE Redmond, WA 98052 |
| MobilePhones | matriz de números de teléfono | Número de teléfono móvil extraído de la tarjeta de presentación | ["+1 (987) 123-4567"] | +19871234567 |
| Faxes | matriz de números de teléfono | Número de fax extraído de la tarjeta de presentación | ["+1 (987) 312-6745"] | +19873126745 |
| WorkPhones | matriz de números de teléfono | Número de teléfono en el trabajo extraído de la tarjeta de presentación | ["+1 (987) 213-5674"] | +19872135674 | 
| OtherPhones     | matriz de números de teléfono | Otros números de teléfono extraídos de la tarjeta de presentación | ["+1 (987) 213-5673"] | +19872135673 |


La API de tarjeta de presentación también puede devolver todo el texto reconocido de la tarjeta de presentación. Esta salida de OCR se incluye en la respuesta JSON.

### <a name="input-requirements"></a>Requisitos de entrada

[!INCLUDE [input requirements](./includes/input-requirements-receipts.md)]

## <a name="supported-locales"></a>Configuraciones regionales admitidas

**La versión 2.1 de Pre-built business cards** admite las siguientes configuraciones regionales: **en-us**, **en-au**, **en-ca**, **en-gb** y **en-in**

## <a name="the-analyze-business-card-operation"></a>Operación Analyze Business Card

[Analyze Business Card](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeBusinessCardAsync) toma una imagen o PDF de una tarjeta de presentación como entrada y extrae los valores de interés. La llamada devuelve un campo de encabezado de respuesta denominado `Operation-Location`. El valor `Operation-Location` es una dirección URL que contiene el id. de resultado que se va a usar en el paso siguiente.

|Encabezado de respuesta| Dirección URL del resultado |
|:-----|:----|
|Operation-Location | `https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeBusinessCardAsync` |

## <a name="the-get-analyze-business-card-result-operation"></a>Operación Get Analyze Business Card Result

El segundo paso consiste en llamar a la operación [Get Analyze Business Card Result](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/GetAnalyzeBusinessCardResult). Esta operación toma como entrada el id. de resultado que la operación Analyze Business Card ha creado. Devuelve una respuesta JSON que contiene un campo de **estado** con los siguientes valores posibles. Llamará a esta operación de forma iterativa hasta que se devuelva con el valor **correcto**. Use un intervalo de 3 a 5 segundos para evitar superar la tasa de solicitudes por segundo (RPS).

|Campo| Tipo | Valores posibles |
|:-----|:----:|:----|
|status | string | notStarted: la operación de análisis no se ha iniciado.<br /><br />running: la operación de análisis está en curso.<br /><br />failed: error en la operación de análisis.<br /><br />succeeded: la operación de análisis se realizó correctamente.|

Cuando el campo **status** tenga el valor **succeeded**, la respuesta JSON incluirá los resultados del reconocimiento de texto opcional y la tarjeta de presentación, si es necesario. El resultado de la descripción de la tarjeta de presentación se organiza como un diccionario de valores de campo con nombre, en el que cada valor contiene el texto extraído, el valor normalizado, el cuadro de límite, el nivel de confianza y los elementos de la palabra correspondiente. El resultado del reconocimiento de texto se organiza como una jerarquía de líneas y palabras, con texto, cuadro de límite e información de confianza.

![salida de tarjeta de presentación de ejemplo](./media/business-card-results.png)

### <a name="sample-json-output"></a>Salida de JSON de ejemplo

La respuesta a la operación Get Analyze Business Card Result será la representación estructurada de la tarjeta de presentación con toda la información extraída.  Aquí encontrará un [archivo con una tarjeta de presentación de ejemplo](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/business-card-english.jpg) y su salida estructurada, [salida de una tarjeta de presentación de ejemplo](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/business-card-result.json).

Consulte el siguiente ejemplo de una respuesta JSON correcta (la salida se ha abreviado por simplicidad):
* El nodo `"readResults"` contiene todo el texto reconocido. El texto se organiza por página, después, por líneas y, finalmente, por palabras individuales.
* El nodo `"documentResults"` contiene los valores específicos de la tarjeta de presentción que el modelo haya descubierto. Aquí encontrará información de contacto útil como el nombre, el apellido, el nombre de la compañía y más.

```json
{
    "status": "succeeded",
    "createdDateTime": "2021-05-27T02:18:35Z",
    "lastUpdatedDateTime": "2021-05-27T02:18:37Z",
    "analyzeResult": {
        "version": "2.1.0",
        "readResults": [
            {
                "page": 1,
                "angle": 0.0255,
                "width": 2592,
                "height": 4608,
                "unit": "pixel",
                "lines": [
                    {
                        "text": "CONTOSO",
                        "boundingBox": [
                            533,
                            1570,
                            1334,
                            1570,
                            1333,
                            1721,
                            533,
                            1720
                        ],
                        "words": [
                            {
                                "text": "CONTOSO",
                                "boundingBox": [
                                    535,
                                    1571,
                                    1278,
                                    1571,
                                    1279,
                                    1722,
                                    534,
                                    1719
                                ],
                                "confidence": 0.994
                            }
                        ],
                        "appearance": {
                            "style": {
                                "name": "other",
                                "confidence": 0.878
                            }
                        }
                    },
                    ...
                ]
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
                    "Addresses": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "4001 1st Ave NE Redmond, WA 98052",
                                "text": "4001 1st Ave NE Redmond, WA 98052",
                                "boundingBox": [
                                    400,
                                    2789,
                                    1514,
                                    2789,
                                    1514,
                                    2857,
                                    400,
                                    2857
                                ],
                                "page": 1,
                                "confidence": 0.986,
                                "elements": [
                                    "#/readResults/0/lines/9/words/0",
                                    "#/readResults/0/lines/9/words/1",
                                    "#/readResults/0/lines/9/words/2",
                                    "#/readResults/0/lines/9/words/3",
                                    "#/readResults/0/lines/9/words/4",
                                    "#/readResults/0/lines/9/words/5",
                                    "#/readResults/0/lines/9/words/6"
                                ]
                            }
                        ]
                    },
                    "CompanyNames": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "CONTOSO",
                                "text": "CONTOSO",
                                "boundingBox": [
                                    535,
                                    1571,
                                    1278,
                                    1571,
                                    1279,
                                    1722,
                                    534,
                                    1719
                                ],
                                "page": 1,
                                "confidence": 0.985,
                                "elements": [
                                    "#/readResults/0/lines/0/words/0"
                                ]
                            }
                        ]
                    },
                    "ContactNames": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "object",
                                "valueObject": {
                                    "FirstName": {
                                        "type": "string",
                                        "valueString": "Chris",
                                        "text": "Chris",
                                        "boundingBox": [
                                            1556,
                                            2018,
                                            1915,
                                            2021,
                                            1915,
                                            2156,
                                            1558,
                                            2154
                                        ],
                                        "page": 1,
                                        "elements": [
                                            "#/readResults/0/lines/1/words/0"
                                        ]
                                    },
                                    "LastName": {
                                        "type": "string",
                                        "valueString": "Smith",
                                        "text": "Smith",
                                        "boundingBox": [
                                            1944,
                                            2021,
                                            2368,
                                            2016,
                                            2364,
                                            2156,
                                            1944,
                                            2156
                                        ],
                                        "page": 1,
                                        "elements": [
                                            "#/readResults/0/lines/1/words/1"
                                        ]
                                    }
                                },
                                "text": "Chris Smith",
                                "boundingBox": [
                                    1556.1,
                                    2010.3,
                                    2368,
                                    2016,
                                    2367,
                                    2159.6,
                                    1555.1,
                                    2154
                                ],
                                "page": 1,
                                "confidence": 0.99,
                                "elements": [
                                    "#/readResults/0/lines/1/words/0",
                                    "#/readResults/0/lines/1/words/1"
                                ]
                            }
                        ]
                    },
                    "Departments": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "Cloud & Al Department",
                                "text": "Cloud & Al Department",
                                "boundingBox": [
                                    1578,
                                    2288.8,
                                    2277,
                                    2295.1,
                                    2276.3,
                                    2367.8,
                                    1577.3,
                                    2361.5
                                ],
                                "page": 1,
                                "confidence": 0.989,
                                "elements": [
                                    "#/readResults/0/lines/3/words/0",
                                    "#/readResults/0/lines/3/words/1",
                                    "#/readResults/0/lines/3/words/2",
                                    "#/readResults/0/lines/3/words/3"
                                ]
                            }
                        ]
                    },
                    "Emails": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "chris.smith@contoso.com",
                                "text": "chris.smith@contoso.com",
                                "boundingBox": [
                                    1583,
                                    2381,
                                    2309,
                                    2382,
                                    2308,
                                    2445,
                                    1584,
                                    2447
                                ],
                                "page": 1,
                                "confidence": 0.99,
                                "elements": [
                                    "#/readResults/0/lines/4/words/0"
                                ]
                            }
                        ]
                    },
                    "Faxes": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "valuePhoneNumber": "+19873126745",
                                "text": "+1 (987) 312-6745",
                                "boundingBox": [
                                    740,
                                    2703.8,
                                    1273,
                                    2702.1,
                                    1273.2,
                                    2774.1,
                                    740.2,
                                    2775.8
                                ],
                                "page": 1,
                                "confidence": 0.99,
                                "elements": [
                                    "#/readResults/0/lines/8/words/1",
                                    "#/readResults/0/lines/8/words/2",
                                    "#/readResults/0/lines/8/words/3"
                                ]
                            }
                        ]
                    },
                    "JobTitles": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "Senior Researcher",
                                "text": "Senior Researcher",
                                "boundingBox": [
                                    1578,
                                    2206,
                                    2117,
                                    2207.6,
                                    2116.8,
                                    2272.6,
                                    1577.8,
                                    2271
                                ],
                                "page": 1,
                                "confidence": 0.99,
                                "elements": [
                                    "#/readResults/0/lines/2/words/0",
                                    "#/readResults/0/lines/2/words/1"
                                ]
                            }
                        ]
                    },
                    "MobilePhones": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "valuePhoneNumber": "+19871234567",
                                "text": "+1 (987) 123-4567",
                                "boundingBox": [
                                    744,
                                    2529,
                                    1281,
                                    2529,
                                    1281,
                                    2603,
                                    744,
                                    2603
                                ],
                                "page": 1,
                                "confidence": 0.99,
                                "elements": [
                                    "#/readResults/0/lines/5/words/1",
                                    "#/readResults/0/lines/5/words/2",
                                    "#/readResults/0/lines/5/words/3"
                                ]
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
                                    1576,
                                    2462,
                                    2383,
                                    2462,
                                    2380,
                                    2535,
                                    1576,
                                    2535
                                ],
                                "page": 1,
                                "confidence": 0.99,
                                "elements": [
                                    "#/readResults/0/lines/6/words/0"
                                ]
                            }
                        ]
                    },
                    "WorkPhones": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "valuePhoneNumber": "+19872135674",
                                "text": "+1 (987) 213-5674",
                                "boundingBox": [
                                    736,
                                    2617.6,
                                    1267.1,
                                    2618.5,
                                    1267,
                                    2687.5,
                                    735.9,
                                    2686.6
                                ],
                                "page": 1,
                                "confidence": 0.984,
                                "elements": [
                                    "#/readResults/0/lines/7/words/1",
                                    "#/readResults/0/lines/7/words/2",
                                    "#/readResults/0/lines/7/words/3"
                                ]
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

* Pruebe sus propias tarjetas de visita y ejemplos en la [interfaz de usuario de ejemplo de Form Recognizer](https://fott-preview.azurewebsites.net/).
* Realice un [inicio rápido de Form Recognizer](quickstarts/client-library.md) para empezar a escribir una aplicación de procesamiento de tarjetas de visita con Form Recognizer en el lenguaje de desarrollo que prefiera.
