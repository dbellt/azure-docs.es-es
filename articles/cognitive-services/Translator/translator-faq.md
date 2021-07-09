---
title: 'Preguntas más frecuentes: Translator'
titleSuffix: Azure Cognitive Services
description: Obtenga respuestas a las preguntas más frecuentes sobre la API de Translator en Azure Cognitive Services.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 05/18/2021
ms.author: lajanuar
ms.openlocfilehash: db1d981305ee62267f72f3050475c88dbcd5cc54
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110116645"
---
# <a name="frequently-asked-questionstranslator-api"></a>Preguntas más frecuentes: API de Translator

## <a name="how-does-translator-count-characters"></a>¿Cómo cuenta Translator los caracteres?

Translator cuenta cada punto de código definido en Unicode como un carácter. Cada traducción se considera una traducción independiente, incluso si la solicitud se ha hecho en una única llamada API de traducción a varios idiomas. La longitud de la respuesta no importa y el número de solicitudes, palabras, bytes u oraciones no es relevante para el recuento de caracteres.

Translator cuenta la entrada siguiente:

* El texto que se pasa a Translator en el cuerpo de una solicitud.
  * `Text` cuando se utilizan los métodos [Translate](reference/v3-0-translate.md), [Transliterate](reference/v3-0-transliterate.md) y [Dictionary Lookup](reference/v3-0-dictionary-lookup.md)
  * `Text` y `Translation` cuando se utiliza el método [Dictionary Examples](reference/v3-0-dictionary-examples.md).

* Todo el marcado: HTML, etiquetas XML, etc., en el campo de texto del cuerpo de la solicitud. La notación JSON que se usa para compilar la solicitud (por ejemplo, la clave "Text:") **no** se cuenta.
* Una letra individual.
* La puntuación.
* Un espacio, una pestaña, un marcado o cualquier carácter de espacio en blanco.
* Una traducción repetida, incluso si ha traducido previamente el mismo texto. Se cuentan todos los caracteres que se envían a la función translate, incluso cuando el contenido no cambie o el idioma de origen y de destino coincidan.

Para los scripts basados en símbolos gráficos, como el chino y el kanji japonés escritos, el servicio Translator cuenta el número de puntos de código Unicode. Un carácter por símbolo. Excepción: los pares suplentes Unicode cuentan como dos caracteres.

Las llamadas a los métodos **Detect** y **BreakSentence** no se cuentan en el consumo de caracteres. Sin embargo, se espera que las llamadas a los métodos Detect y BreakSentence estén razonablemente proporcionadas frente al uso de otras funciones de recuento. Si el número de llamadas Detect o BreakSentence supera en 100 veces el número de otros métodos de recuento, Microsoft se reserva el derecho a restringir el uso de los métodos Detect y BreakSentence.

## <a name="where-can-i-see-my-monthly-usage"></a>¿Dónde puedo ver mi uso mensual?

Puede utilizar la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/) para calcular los costos. También puede supervisar, ver y agregar alertas de Azure para los servicios de Azure en su cuenta de usuario en Azure Portal:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a la página Información general del recurso Translator.
1. Seleccione la **suscripción** del recurso Translator.

    :::image type="content" source="media/azure-portal-overview.png" alt-text="Captura de pantalla del vínculo de suscripción en la página Información general de Azure Portal.":::

2. En el raíl izquierdo, haga su selección en **Administración de costos**:

    :::image type="content" source="media/azure-portal-cost-management.png" alt-text="Captura de pantalla de los vínculos de recursos de administración de costos en Azure Portal.":::

## <a name="is-attribution-required-when-using-translator"></a>¿Es necesaria la atribución cuando se usa Translator?

La atribución no es necesaria cuando se usa Translator para la traducción de texto y voz. Se recomienda informar a los usuarios de que el contenido que están viendo está traducido mediante traducción automática.

Si la atribución está presente, debe cumplir las [directrices de atribución de Translator](https://www.microsoft.com/translator/business/attribution/).

## <a name="is-translator-a-replacement-for-human-translator"></a>¿Translator es un sustituto del traductor humano?

No, ambos tienen su lugar como herramientas esenciales para la comunicación. Utilice la traducción automática cuando la cantidad de contenido, la velocidad de creación y las restricciones de presupuesto hagan imposible el uso de la traducción humana.

Varios de nuestros asociados [proveedores de servicios de lenguaje (LSP)](https://www.microsoft.com/translator/business/partners/) han utilizado la traducción automática como primer paso, antes de usar la traducción humana, ya que puede mejorar la productividad hasta un 50 %. Para obtener una lista de asociados LSP, visite la página de asociados de Translator.

---
> [!TIP]
> Si no puede encontrar las respuestas en estas preguntas más frecuentes, intente plantearlas en la comunidad de la API de Translator en [StackOverflow](https://stackoverflow.com/search?q=%5Bmicrosoft-cognitive%5D+or+%5Bmicrosoft-cognitive%5D+translator&s=34bf0ce2-b6b3-4355-86a6-d45a1121fe27).