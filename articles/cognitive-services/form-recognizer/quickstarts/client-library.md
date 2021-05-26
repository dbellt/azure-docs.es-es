---
title: 'Inicio rápido: Biblioteca cliente o API REST de Form Recognizer'
titleSuffix: Azure Applied AI Services
description: Use la biblioteca cliente o API REST de Form Recognizer con el fin de crear una aplicación de procesamiento de formularios que extraiga pares clave-valor y datos de tabla de los documentos personalizados.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: quickstart
ms.date: 04/14/2021
ms.author: lajanuar
zone_pivot_groups: programming-languages-set-formre
ms.custom: devx-track-js, devx-track-csharp, cog-serv-seo-aug-2020
keywords: forms processing, automated data processing
ms.openlocfilehash: 34b2e591c8d050a168e82bb40c5167823c136e52
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110374203"
---
# <a name="quickstart-use-the-form-recognizer-client-library-or-rest-api"></a>Inicio rápido: Uso de la biblioteca cliente o API REST de Form Recognizer

Comience a usar Form Recognizer mediante el lenguaje de desarrollo de su elección. Azure Form Recognizer es un servicio cognitivo que le permite crear software de procesamiento de datos automatizado mediante tecnología de aprendizaje automático. Identifique y extraiga texto, pares clave-valor, marcas de selección, datos de tabla, etc., de los documentos de formulario; el servicio genera datos estructurados que incluyen las relaciones en el archivo original. Puede usar Form Recognizer a través de la API REST o el SDK. Se recomienda usar el servicio gratuito cuando se está aprendiendo la tecnología. Recuerde que el número de páginas gratuitas se limita a 500 al mes.

Va a utilizar las siguientes API para extraer datos estructurados de formularios y documentos:

|Nombre |Descripción |
|---|---|
| **[Análisis de diseño](#analyze-layout)** | Analice un documento pasado como flujo para extraer de él texto, marcas de selección, tablas y su estructura. |
| **[Análisis de las confirmaciones de recepción](#analyze-receipts)** | Analice un recibo para extraer de él la información clave y cualquier otro texto.|
| **[Análisis de tarjetas de presentación](#analyze-business-cards)** | Analice una tarjeta de presentación para extraer el texto y la información más importante.|
| **[Análisis de facturas](#analyze-invoices)** | Analice una factura para extraer de ella información clave, tablas y otro tipo de texto.|
| **[Análisis de documentos de identidad](#analyze-identity-documents)** | Analice un documento de identidad para extraer información importante y otro texto del carné de identidad.|
| **[Entrenamiento de un modelo personalizado](#train-a-custom-model)**| Entrene un nuevo modelo para analizar los formularios con cinco formularios del mismo tipo. Establezca el parámetro _useLabelFile_ en `true` para entrenar con datos etiquetados manualmente. |
| **[Analizar formularios con un modelo personalizado](#analyze-forms-with-a-custom-model)**|Analice un formulario pasado como una secuencia para extraer texto, pares clave-valor y tablas del formulario con su modelo personalizado.  |
|**[Administración de modelos personalizados](#manage-custom-models)**| Puede comprobar el número de modelos personalizados en su cuenta de Form Recognizer, obtener un modelo específico mediante su identificador y eliminar un modelo de su cuenta.|

::: zone pivot="programming-language-csharp"

[!INCLUDE [C# SDK quickstart](../includes/quickstarts/csharp-sdk.md)]

::: zone-end

::: zone pivot="programming-language-java"

[!INCLUDE [Java SDK quickstart](../includes/quickstarts/java-sdk.md)]

::: zone-end

::: zone pivot="programming-language-javascript"

[!INCLUDE [NodeJS SDK quickstart](../includes/quickstarts/javascript-sdk.md)]

::: zone-end

::: zone pivot="programming-language-python"

[!INCLUDE [Python SDK quickstart](../includes/quickstarts/python-sdk.md)]

::: zone-end

::: zone pivot="programming-language-rest-api"

[!INCLUDE [REST API quickstart](../includes/quickstarts/rest-api.md)]

::: zone-end
