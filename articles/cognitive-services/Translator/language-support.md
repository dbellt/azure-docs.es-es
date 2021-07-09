---
title: 'Idiomas admitidos: Translator'
titleSuffix: Azure Cognitive Services
description: Translator de Cognitive Services admite los siguientes idiomas para la conversión de texto a texto con traducción automática neuronal (NMT).
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 06/10/2020
ms.author: lajanuar
ms.openlocfilehash: 9bfb125a9906c89fb7c607ce241a9b7696b805bc
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111537612"
---
# <a name="language-support-for-text-and-speech-translation"></a>Compatibilidad de idiomas con la traducción de texto y de voz

Use Translator para traducir a y desde cualquiera de los 90 idiomas y dialectos de traducción de texto. La traducción automática neuronal (NMT) es el nuevo estándar de traducción automática de alta calidad con tecnologías de inteligencia artificial, y está disponible de forma predeterminada con la versión V3 de Translator cuando hay un sistema neuronal disponible.

También puede usar Translator junto con Traductor personalizado para compilar sistemas de traducción neuronal que entiendan la terminología que se usa en su empresa y su sector y con Servicio Voz de Microsoft para agregar traducción de voz a la aplicación.

[Más información sobre cómo funciona la traducción automática](https://www.microsoft.com/translator/mt.aspx)

## <a name="text-translation"></a>Traducción de texto
La traducción de texto está disponible mediante la operación Traducir a o desde cualquiera de los idiomas disponibles en Translator. La API también ofrece detección de idioma mediante la operación Detectar, transliteración mediante la operación Transliterar y diccionarios bilingües mediante las operaciones Buscar en diccionario y Ejemplos de diccionario. A continuación se indican los idiomas disponibles en cada una de estas operaciones. 

### <a name="translate"></a>Translate

Translator admite los siguientes idiomas para la traducción de texto a texto. 

[Vea documentación de referencia de la operación Traducir](reference/v3-0-translate.md)

| Idioma | Código de lenguaje |
|:-|:-:|
| Afrikáans | `af` |
| Albanés | `sq` |
| Amárico | `am` |
| Árabe | `ar` |
| Armenio | `hy` |
| Asamés | `as` |
| Azerbaiyano | `az` |
| Bengalí | `bn` |
| Bosnio (latino) | `bs` |
| Búlgaro | `bg` |
| Cantonés (tradicional) | `yue` |
| Catalán | `ca` |
| Chino simplificado | `zh-Hans` |
| Chino tradicional | `zh-Hant` |
| Croata | `hr` |
| Checo | `cs` |
| Danés | `da` |
| Dari | `prs` |
| Neerlandés | `nl` |
| Inglés | `en` |
| Estonio | `et` |
| Fiyiano | `fj` |
| Filipino | `fil` |
| Finés | `fi` |
| Francés | `fr` |
| Francés (Canadá) | `fr-ca` |
| Alemán | `de` |
| Griego | `el` |
| Gujarati | `gu` |
| Criollo haitiano | `ht` |
| Hebreo | `he` |
| Hindi | `hi` |
| Hmong Daw | `mww` |
| Húngaro | `hu` |
| Islandés | `is` |
| Indonesio | `id` |
| Inuktitut | `iu` |
| Irlandés | `ga` |
| Italiano | `it` |
| Japonés | `ja` |
| Canarés | `kn` |
| Kazajo | `kk` |
| Jemer | `km` |
| Klingon | `tlh-Latn` |
| Klingon (plqaD) | `tlh-Piqd` |
| Coreano | `ko` |
| Kurdo (central) | `ku` |
| Kurdo (norte) | `kmr` |
| Lao | `lo` |
| Letón | `lv` |
| Lituano | `lt` |
| Malgache | `mg` |
| Malayo | `ms` |
| Malayalam | `ml` |
| Maltés | `mt` |
| Maori | `mi` |
| Maratí | `mr` |
| Myanmar | `my` |
| Nepalí | `ne` |
| Noruego | `nb` |
| Odia | `or` |
| Pastún | `ps` |
| Persa | `fa` |
| Polaco | `pl` |
| Portugués (Brasil) | `pt` |
| Portugués (Portugal) | `pt-pt` |
| Punjabi | `pa` |
| Otomí Querétaro | `otq` |
| Rumano | `ro` |
| Ruso | `ru` |
| Samoano | `sm` |
| Serbio (cirílico) | `sr-Cyrl` |
| Serbio (latino) | `sr-Latn` |
| Eslovaco | `sk` |
| Esloveno | `sl` |
| Español | `es` |
| Swahili | `sw` |
| Sueco | `sv` |
| Tahitiano | `ty` |
| Tamil | `ta` |
| Telugu | `te` |
| Tailandés | `th` |
| Tigriña | `ti` |
| Tongano | `to` |
| Turco | `tr` |
| Ucraniano | `uk` |
| Urdu | `ur` |
| Vietnamita | `vi` |
| Galés | `cy` |
| Maya Yucateco | `yua` |

> [!NOTE]
> El código de idioma `pt` se establece de forma predeterminada en `pt-br`, Portugués (Brasil).

> [!NOTE]
> Actualmente, los siguientes idiomas no están disponibles en los contenedores: samoano, tahitiano, klingon, klingon (plqaD) y tailandés.

### <a name="detect"></a>Detect

Translator detecta los siguientes idiomas para traducción y transliteración.

[Vea documentación de referencia de la operación Detectar](reference/v3-0-detect.md)

| Idioma | Código de lenguaje |
|:-|:-:|
| Afrikáans | `af` |
| Albanés | `sq` |
| Árabe | `ar` |
| Búlgaro | `bg` |
| Catalán | `ca` |
| Chino simplificado | `zh-Hans` |
| Chino tradicional | `zh-Hant` |
| Croata | `hr` |
| Checo | `cs` |
| Danés | `da` |
| Neerlandés | `nl` |
| Inglés | `en` |
| Estonio | `et` |
| Finés | `fi` |
| Francés | `fr` |
| Alemán | `de` |
| Griego | `el` |
| Gujarati | `gu` |
| Criollo haitiano | `ht` |
| Hebreo | `he` |
| Hindi | `hi` |
| Húngaro | `hu` |
| Islandés | `is` |
| Indonesio | `id` |
| Inuktitut | `iu` |
| Irlandés | `ga` |
| Italiano | `it` |
| Japonés | `ja` |
| Klingon | `tlh-Latn` |
| Coreano | `ko` |
| Kurdo (central) | `ku-Arab` |
| Letón | `lv` |
| Lituano | `lt` |
| Malayo | `ms` |
| Maltés | `mt` |
| Noruego | `nb` |
| Pastún | `ps` |
| Persa | `fa` |
| Polaco | `pl` |
| Portugués | `pt` |
| Rumano | `ro` |
| Ruso | `ru` |
| Serbio (cirílico) | `sr-Cyrl` |
| Serbio (latino) | `sr-Latn` |
| Eslovaco | `sk` |
| Esloveno | `sl` |
| Español | `es` |
| Swahili | `sw` |
| Sueco | `sv` |
| Tahitiano | `ty` |
| Tailandés | `th` |
| Turco | `tr` |
| Ucraniano | `uk` |
| Urdu | `ur` |
| Vietnamita | `vi` |
| Galés | `cy` |
| Maya Yucateco | `yua` |

### <a name="transliterate"></a>Transliterar

El método Transliterate admite los siguientes idiomas. En "Hacia/Desde", "<-->" indica que el idioma se puede transliterar hacia o desde cualquiera de los alfabetos enumerados. "-->" indica que el idioma solo se puede transliterar de un idioma al otro.

[Vea documentación de referencia de la operación Transliterar](reference/v3-0-translate.md)


| Idioma    | Código de lenguaje | Script | Hacia/Desde | Script|
|:----------- |:-------------:|:-------------:|:-------------:|:-------------:|
| Árabe | `ar` | Árabe `Arab` | <--> | Latino `Latn` |
| Asamés | `as` | Bengalí `Beng` | <--> | Latino `Latn` |
| Bengalí  | `bn` | Bengalí `Beng` | <--> | Latino `Latn` |
|Bielorruso| `be` | Cirílico `Cyrl`  | <--> | Latino `Latn` |
|Búlgaro| `bg` | Cirílico `Cyrl`  | <--> | Latino `Latn` |
| Chino (simplificado) | `zh-Hans` | Chino simplificado `Hans`| <--> | Latino `Latn` |
| Chino (simplificado) | `zh-Hans` | Chino simplificado `Hans`| <--> | Chino tradicional `Hant`|
| Chino (tradicional) | `zh-Hant` | Chino tradicional `Hant`| <--> | Latino `Latn` |
| Chino (tradicional) | `zh-Hant` | Chino tradicional `Hant`| <--> | Chino simplificado `Hans` |
|Griego| `el` | Griego `Grek`  | <--> | Latino `Latn` |
| Gujarati | `gu`  | Guyaratí `Gujr` | <--> | Latino `Latn` |
| Hebreo | `he` | Hebreo `Hebr` | <--> | Latino `Latn` |
| Hindi | `hi` | Devanagari `Deva` | <--> | Latino `Latn` |
| Japonés | `ja` | Japonés `Jpan` | <--> | Latino `Latn` |
| Canarés | `kn` | Canarés `Knda` | <--> | Latino `Latn` |
|Kazajo| `kk` | Cirílico `Cyrl`  | <--> | Latino `Latn` |
|Coreano| `ko` | Coreano `Kore`  | <--> | Latino `Latn` |
|Kirguís| `ky` | Cirílico `Cyrl`  | <--> | Latino `Latn` |
|Macedonio| `mk` | Cirílico `Cyrl`  | <--> | Latino `Latn` |
| Malayalam | `ml` | Malayalam `Mlym` | <--> | Latino `Latn` |
| Maratí | `mr` | Devanagari `Deva` | <--> | Latino `Latn` |
|Mongol| `mn` | Cirílico `Cyrl`  | <--> | Latino `Latn` |
| Odia | `or` | Odia `Orya` | <--> | Latino `Latn` |
|Persa| `fa` | Árabe `Arab`  | <--> | Latino `Latn` |
| Punjabi | `pa` | Gurmukhi `Guru`  | <--> | Latino `Latn`  |
|Ruso| `ru` | Cirílico `Cyrl`  | <--> | Latino `Latn` |
| Serbio (cirílico) | `sr-Cyrl` | Cirílico `Cyrl`  | --> | Latino `Latn` |
| Serbio (latino) | `sr-Latn` | Latino `Latn` | --> | Cirílico `Cyrl`|
|Sindhi| `sd` | Árabe `Arab`  | <--> | Latino `Latn` |
|Cingalés| `si` | Cingalés `Sinh`  | <--> | Latino `Latn` |
|Tayiko| `tg` | Cirílico `Cyrl`  | <--> | Latino `Latn` |
| Tamil | `ta` | Tamil `Taml` | <--> | Latino `Latn` |
|Tatar| `tt` | Cirílico `Cyrl`  | <--> | Latino `Latn` |
| Telugu | `te` | Telugu `Telu` | <--> | Latino `Latn` |
| Tailandés | `th` | Tailandés `Thai` | --> | Latino `Latn` |
|Ucraniano| `uk` | Cirílico `Cyrl`  | <--> | Latino `Latn` |
|Urdu| `ur` | Árabe `Arab`  | <--> | Latino `Latn` |

### <a name="dictionary"></a>Diccionario

El diccionario admite los siguientes idiomas desde o hacia el inglés con los métodos Lookup y Examples.

Vea documentación de referencia de las operaciones [Buscar en diccionario](reference/v3-0-dictionary-lookup.md) y [Ejemplos de diccionario](reference/v3-0-dictionary-examples.md).

| Idioma    | Código de lenguaje |
|:----------- |:-------------:|
| Afrikáans      | `af`          |
| Árabe       | `ar`          |
| Bengalí      | `bn`          |
| Bosnio (latino)      | `bs`          |
| Búlgaro      | `bg`          |
| Catalán      | `ca`          |
| Chino simplificado      | `zh-Hans`          |
| Croata      | `hr`          |
| Checo      | `cs`          |
| Danés      | `da`          |
| Neerlandés      | `nl`          |
| Estonio      | `et`          |
| Finés      | `fi`          |
| Francés      | `fr`          |
| Alemán      | `de`          |
| Griego      | `el`          |
| Criollo haitiano      | `ht`          |
| Hebreo      | `he`          |
| Hindi      | `hi`          |
| Hmong Daw      | `mww`          |
| Húngaro      | `hu`          |
| Islandés    | `is`  |
| Indonesio      | `id`          |
| Italiano      | `it`          |
| Japonés      | `ja`          |
| Klingon      | `tlh`          |
| Coreano      | `ko`          |
| Letón      | `lv`          |
| Lituano      | `lt`          |
| Malayo      | `ms`          |
| Maltés      | `mt`          |
| Noruego      | `nb`          |
| Persa      | `fa`          |
| Polaco      | `pl`          |
| Portugués (Brasil)     | `pt`          |
| Rumano      | `ro`          |
| Ruso      | `ru`          |
| Serbio (latino)      | `sr-Latn`          |
| Eslovaco     | `sk`          |
| Esloveno      | `sl`          |
| Español      | `es`          |
| Swahili      | `sw`          |
| Sueco      | `sv`          |
| Tamil      | `ta`          |
| Tailandés      | `th`          |
| Turco      | `tr`          |
| Ucraniano      | `uk`          |
| Urdu      | `ur`          |
| Vietnamita      | `vi`          |
| Galés      | `cy`          |

### <a name="access-the-translator-language-list-programmatically"></a>Acceso a la lista de idiomas de Translator mediante programación

Puede recuperar una lista de los idiomas admitidos por Translator con el método Languages. Puede ver la lista por característica, código de idioma o por nombre del idioma en inglés o en cualquier otro idioma admitido. El servicio Microsoft Translator actualiza esta lista automáticamente cuando hay nuevos idiomas disponibles.

[Ver la documentación de referencia de la operación Languages](reference/v3-0-languages.md)

## <a name="customization"></a>Personalización

Los siguientes idiomas están disponibles para personalización al y del inglés mediante [Traductor personalizado](https://aka.ms/CustomTranslator).

| Idioma    | Código de lenguaje |
|:----------- |:-------------:|
|Afrikáans| `af`|
| Árabe       | `ar`          |
| Bengalí      | `bn`          |
| Bosnio (latino)      | `bs`          |
| Búlgaro      | `bg`          |
|Catalán|   `ca`    |
| Chino simplificado      | `zh-Hans`          |
|Chino tradicional|   `zh-Hant`   |
| Croata      | `hr`          |
| Checo      | `cs`          |
| Danés      | `da`          |
| Neerlandés      | `nl`          |
| Inglés    | `en`     |
| Estonio      | `et`          |
|Fiyiano|    `fj`    |
|Filipino|  `fil`   |
| Finés      | `fi`          |
| Francés      | `fr`          |
| Alemán      | `de`          |
| Griego      | `el`          |
| Gujarati| `gu`    |
| Hebreo      | `he`          |
| Hindi      | `hi`          |
| Húngaro      | `hu`          |
| Islandés | `is` |
| Indonesio|   `id`    |
| Inuktitut|    `iu`    |
| Irlandés | `ga`  |
| Italiano      | `it`          |
| Japonés      | `ja`          |
|Canarés|`kn`|
| Coreano      | `ko`          |
| Letón      | `lv`          |
| Lituano      | `lt`          |
| Malgache| `mg`    |
| Malayo|    `ms` |
|Maltés|   `mt`    |
| Maori| `mi`  |
| Maratí| `mr`  |
| Noruego      | `nb`          |
| Persa      | `fa`          |
| Polaco      | `pl`          |
| Portugués (Brasil) | `pt` |
| Punjabi|`pa`|
| Rumano      | `ro`          |
| Ruso      | `ru`          |
| Samoano|   `sm`    |
| Serbio (latino)      | `sr-Latn`          |
| Eslovaco     | `sk`          |
| Esloveno      | `sl`          |
| Español      | `es`          |
| Swahili|  `sw`    |
| Sueco      | `sv`          |
|Tahitiano|  `ty`    |
| Tailandés      | `th`          |
|Tongano|    `to`    |
| Turco      | `tr`          |
| Ucraniano      | `uk`          |
| Urdu| `ur`    |
| Vietnamita      | `vi`          |
| Galés | `cy` |

## <a name="speech-translation"></a>Speech Translation
La traducción de voz está disponible mediante Translator con el Servicio Voz de Cognitive Services. Vea [Documentación sobre el servicio de voz](../speech-service/index.yml) para obtener más información sobre el uso de la traducción de voz y para ver todas las [opciones de idioma disponibles](../speech-service/language-support.md).

### <a name="speech-to-text"></a>Voz a texto
Convierta la voz en texto para traducir al idioma de texto que prefiera. La conversión de voz en texto se usa para la traducción de voz a texto o para la traducción de voz a voz cuando se usa junto con la síntesis de voz.

| Idioma    |
|:----------- |
|Árabe|
|Cantonés (tradicional)|
|Catalán|
|Chino simplificado|
|Chino tradicional|
|Danés|
|Neerlandés|
|Inglés|
|Finés|
|Francés|
|Francés (Canadá)|
|Alemán|
|Gujarati|
|Hindi|
|Italiano|
|Japonés|
|Coreano|
|Maratí|
|Noruego|
|Polaco|
|Portugués (Brasil)|
|Portugués (Portugal)|
|Ruso|
|Español|
|Sueco|
|Tamil|
|Telugu|
|Tailandés|
|Turco|

### <a name="text-to-speech"></a>Texto a voz
Convertir texto a voz. La conversión de texto en voz se usa para agregar una salida con sonido de los resultados de traducción o para la traducción de voz a voz cuando se usa con la conversión de voz en texto. 

| Idioma |
|:-|
| Árabe |
| Búlgaro |
| Cantonés (tradicional) |
| Catalán |
| Chino simplificado |
| Chino tradicional |
| Croata |
| Checo |
| Danés |
| Neerlandés |
| Inglés |
| Finés |
| Francés |
| Francés (Canadá) |
| Alemán |
| Griego |
| Hebreo |
| Hindi |
| Húngaro |
| Indonesio |
| Italiano |
| Japonés |
| Coreano |
| Malayo |
| Noruego |
| Polaco |
| Portugués (Brasil) |
| Portugués (Portugal) |
| Rumano |
| Ruso |
| Eslovaco |
| Esloveno |
| Español |
| Sueco |
| Tamil |
| Telugu |
| Tailandés |
| Turco |
| Vietnamita |

## <a name="view-the-language-list-on-the-microsoft-translator-website"></a>Vista de la lista de idiomas en el sitio web de Microsoft Translator

Para obtener una vista rápida de los idiomas, el sitio web de Microsoft Translator muestra todos los idiomas compatibles con Translator para la traducción de texto y el Servicio Voz para la traducción de voz. Esta lista no incluye información específica para desarrolladores, por ejemplo, los códigos de idioma.

[Ver la lista de idiomas](https://www.microsoft.com/translator/languages.aspx)
