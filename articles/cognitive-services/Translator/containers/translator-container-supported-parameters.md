---
title: 'Contenedor: método translate de Translator'
titleSuffix: Azure Cognitive Services
description: Comprenda los parámetros, los encabezados y el cuerpo de los mensajes del método translate de contenedores de Azure Cognitive Services Translator para traducir texto.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 05/12/2021
ms.author: lajanuar
ms.openlocfilehash: 25fefcdbf5fc7b8bd9ad8bc81a9a1bd6bd7d3e11
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111957438"
---
# <a name="container-translator-translate-method"></a>Contenedor: método translate de Translator

Traduce texto.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `POST` a:

```HTTP
http://localhost:{port}/translate?api-version=3.0
```

Ejemplo: http://<span></span>localhost:5000/translate?api-version=3.0

## <a name="request-parameters"></a>Parámetros de solicitud

Los parámetros de solicitud que se pasaron en la cadena de consulta son:

### <a name="required-parameters"></a>Parámetros obligatorios

| Parámetro de consulta | Descripción |
| --- | --- |
| api-version | _Parámetro obligatorio_.  <br>Versión de la API que el cliente solicitó. El valor debe ser `3.0`. |
| desde | _Parámetro obligatorio_.  <br>Especifica el idioma del texto de entrada. Busque los idiomas que están disponibles desde los que realizar la traducción mediante la busca de [idiomas admitidos](../reference/v3-0-languages.md) con el ámbito `translation`.|
| to  | _Parámetro obligatorio_.  <br>Especifica el idioma del texto de salida. El idioma de destino debe ser uno de los [idiomas admitidos](../reference/v3-0-languages.md) que están incluidos en el ámbito `translation`. Por ejemplo, utilice `to=de` para traducir al alemán.  <br>Es posible traducir a varios idiomas simultáneamente mediante la repetición del parámetro en la cadena de consulta. Por ejemplo, utilice `to=de&to=it` para traducir al alemán e italiano. |

### <a name="optional-parameters"></a>Parámetros opcionales

| Parámetro de consulta | Descripción |
| --- | --- |
| textType | _Parámetro opcional_.  <br>Define si el texto que se está traduciendo es texto sin formato o texto HTML. El código HTML debe ser un elemento completo y bien formado. Los valores posibles son `plain` (valor predeterminado) o `html`. |
| includeSentenceLength | _Parámetro opcional_.  <br>Especifica si se deben incluir los límites de oraciones del texto de entrada y el texto traducido. Los valores posibles son `true` o `false` (valor predeterminado). |

Los encabezados de solicitud incluyen lo siguiente:

| encabezados | Descripción |
| --- | --- |
| Encabezados de autenticación | _Encabezado de solicitud obligatorio_.  <br>Consulte las [opciones disponibles para la autenticación](../reference/v3-0-reference.md#authentication). |
| Content-Type | _Encabezado de solicitud obligatorio_.  <br>Especifica el tipo de contenido de la carga.  <br>El valor aceptado es `application/json; charset=UTF-8`. |
| Content-Length | _Encabezado de solicitud obligatorio_.  <br>Longitud del cuerpo de la solicitud. |
| X-ClientTraceId | _Opcional_.  <br>GUID generado por el cliente para identificar de forma única la solicitud. Puede omitir este encabezado si incluye el id. de seguimiento en la cadena de la consulta mediante un parámetro de consulta denominado `ClientTraceId`. |

## <a name="request-body"></a>Cuerpo de la solicitud

El cuerpo de la solicitud es una matriz JSON. Cada elemento de la matriz es un objeto JSON con una propiedad de cadena denominada `Text`, que representa la cadena que se va a traducir.

```json
[
    {"Text":"I would really like to drive your car around the block a few times."}
]
```

Se aplican las siguientes limitaciones:

* La matriz puede tener como máximo 100 elementos.
* El texto completo incluido en la solicitud no puede superar los 10 000 caracteres, incluidos los espacios.

## <a name="response-body"></a>Response body

Una respuesta correcta es una matriz JSON con un resultado para cada cadena en la matriz de entrada. Un objeto del resultado incluye las siguientes propiedades:

* `translations`: matriz de resultados de la traducción. El tamaño de la matriz coincide con el número de idiomas de destino especificados a través del parámetro de consulta `to`. Cada elemento de la matriz incluye:

* `to`: cadena que representa el código del idioma de destino.

* `text`: cadena que proporciona el texto traducido.

* `sentLen`: objeto que devuelve los límites de las oraciones en los textos de entrada y salida.

* `srcSentLen`: matriz de enteros que representan las longitudes de las oraciones en el texto de entrada. La longitud de la matriz es el número de oraciones y los valores son la longitud de cada oración.

* `transSentLen`:  matriz de enteros que representan las longitudes de las oraciones en el texto traducido. La longitud de la matriz es el número de oraciones y los valores son la longitud de cada oración.

    Los límites de oraciones solo se incluyen cuando el parámetro de solicitud `includeSentenceLength` es `true`.

  * `sourceText`: objeto con una propiedad de cadena única denominada `text`, que proporciona el texto de entrada en el script predeterminado del idioma de origen. La propiedad `sourceText` solo está presente cuando la entrada se expresa en un script que no es el habitual del idioma. Por ejemplo, si la entrada era árabe escrito en alfabeto latino, `sourceText.text` será el mismo texto árabe convertido al alfabeto árabe.

En la sección de [ejemplos](#examples) se proporcionan ejemplos de respuestas JSON.

## <a name="response-headers"></a>Encabezados de respuesta

| encabezados | Descripción |
| --- | --- |
| X-RequestId | Valor generado por el servicio para identificar la solicitud. Se usa para solucionar problemas. |
| Sistema X-MT | Especifica el tipo de sistema que se usó para la traducción en cada idioma de 'destino' solicitado para la traducción. El valor es una lista de cadenas separadas por comas. Cada cadena indica un tipo:  </br></br>&FilledVerySmallSquare; Custom: la solicitud incluye un sistema personalizado y se ha usado al menos un sistema personalizado durante la traducción.</br>&FilledVerySmallSquare; Team: todas las demás solicitudes. |

## <a name="response-status-codes"></a>Códigos de estado de respuesta

Si se produce un error, la solicitud también devolverá una respuesta de error JSON. El código de error es un número de 6 dígitos que combina el código de estado HTTP de 3 dígitos y otro número de 3 dígitos que ayuda a categorizar aún más el error. En la [página de referencia de Traductor v3](../reference/v3-0-reference.md#errors) pueden encontrarse los códigos de error comunes.

## <a name="examples"></a>Ejemplos

### <a name="translate-a-single-input"></a>Traducción de una única entrada

En este ejemplo se muestra cómo traducir una única oración del inglés al chino simplificado.

```curl
curl -X POST "http://localhost:{port}/translate?api-version=3.0&from=en&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'Hello, what is your name?'}]"
```

El cuerpo de la respuesta es:

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"}
        ]
    }
]
```

La matriz `translations` incluye un elemento, que proporciona la traducción de la parte única de texto en la entrada.

### <a name="translate-multiple-pieces-of-text"></a>Traducción de varios fragmentos de texto

Traducir varias cadenas a la vez es simplemente cuestión de especificar una matriz de cadenas en el cuerpo de la solicitud.

```curl
curl -X POST "http://localhost:{port}/translate?api-version=3.0&from=en&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'Hello, what is your name?'}, {'Text':'I am fine, thank you.'}]"
```

La respuesta contiene la traducción de todos los fragmentos de texto exactamente en el mismo orden que en la solicitud.
El cuerpo de la respuesta es:

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"}
        ]
    },
    {
        "translations":[
            {"text":"我很好，谢谢你。","to":"zh-Hans"}
        ]
    }
]
```

### <a name="translate-to-multiple-languages"></a>Traducción a varios idiomas

En este ejemplo se muestra cómo traducir la misma entrada a diferentes idiomas en una sola solicitud.

```curl
curl -X POST "http://localhost:{port}/translate?api-version=3.0&from=en&to=zh-Hans&to=de" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'Hello, what is your name?'}]"
```

El cuerpo de la respuesta es:

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"},
            {"text":"Hallo, was ist dein Name?","to":"de"}
        ]
    }
]
```

### <a name="translate-content-with-markup-and-decide-whats-translated"></a>Traducción del contenido con marcado y decisión de qué traducir

Es común traducir el contenido que incluye marcado como contenido de una página HTML o de un documento XML. Incluya el parámetro de consulta `textType=html` al traducir contenido con etiquetas. Además, a veces resulta útil para excluir contenido específico de la traducción. Puede utilizar el atributo `class=notranslate` para especificar contenido que debería permanecer en el idioma original. En el ejemplo siguiente, el contenido del primer elemento `div` no se traducirá, mientras que el del segundo elemento `div`, sí.

```
<div class="notranslate">This will not be translated.</div>
<div>This will be translated. </div>
```

A continuación, se muestra una solicitud de ejemplo para ilustrar.

```curl
curl -X POST "http://localhost:{port}/translate?api-version=3.0&from=en&to=zh-Hans&textType=html" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'<div class=\"notranslate\">This will not be translated.</div><div>This will be translated.</div>'}]"
```

La respuesta es:

```
[
    {
        "translations":[
            {"text":"<div class=\"notranslate\">This will not be translated.</div><div>这将被翻译。</div>","to":"zh-Hans"}
        ]
    }
]
```

### <a name="translate-with-dynamic-dictionary"></a>Traducción con diccionario dinámico

Si ya conoce la traducción que quiere aplicar a una palabra o frase, puede proporcionarla como marcado dentro de la solicitud. El diccionario dinámico solo es seguro para los nombres propios, como nombres de personas y nombres de productos.

El marcado que se va a proporcionar utiliza la sintaxis siguiente.

```
<mstrans:dictionary translation="translation of phrase">phrase</mstrans:dictionary>
```

Por ejemplo, considere la frase en inglés "The word wordomatic is a dictionary entry" (La palabra "wordomatic" es una entrada del diccionario). Para conservar la palabra _wordomatic_ en la traducción, envíe la solicitud:

```
curl -X POST "http://localhost:{port}/translate?api-version=3.0&from=en&to=de" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'The word <mstrans:dictionary translation=\"wordomatic\">word or phrase</mstrans:dictionary> is a dictionary entry.'}]"
```

El resultado es el siguiente:

```
[
    {
        "translations":[
            {"text":"Das Wort \"wordomatic\" ist ein Wörterbucheintrag.","to":"de"}
        ]
    }
]
```

Esta característica funciona del mismo modo con `textType=text` o con `textType=html`. La característica debe usarse con moderación. La mejor forma de personalización de traducción, y más adecuada, es mediante el uso de Custom Translator. Custom Translator hace un uso completo del contexto y las probabilidades estadísticas. Si tiene o puede permitirse crear datos de entrenamiento que muestren el trabajo o una frase en contexto, obtendrá resultados mucho mejores. [Más información acerca de Custom Translator](../customization.md).

## <a name="request-limits"></a>Límites de solicitud

Cada solicitud de traducción está limitada a 10 000 caracteres, en todos los idiomas de destino a los que se va a traducir. Por ejemplo, si se envía una solicitud de traducción de 3000 caracteres para traducir a tres idiomas distintos, se obtiene un tamaño de solicitud de 3000 x 3=9000 caracteres, lo que satisface el límite de solicitudes. Se le cobrará por cada carácter, no por el número de solicitudes. Se recomienda enviar solicitudes más cortas.

En la siguiente tabla se muestran los límites de caracteres y elementos de matriz correspondientes a la operación **translation** de Translator.

| Operación | Tamaño máximo del elemento de matriz | Número máximo de elementos de matriz | Tamaño máximo de la solicitud (caracteres) |
|:----|:----|:----|:----|
| translate | 10 000 | 100 | 10 000 |