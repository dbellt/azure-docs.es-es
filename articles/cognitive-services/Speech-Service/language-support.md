---
title: 'Compatibilidad con idiomas: servicio de voz'
titleSuffix: Azure Cognitive Services
description: El servicio de voz admite un gran número de idiomas para la conversión de texto a voz y voz a texto, junto con la traducción de voz. En este artículo se proporciona una lista completa de idiomas compatibles por servicio.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/07/2021
ms.author: trbye
ms.custom: references_regions
ms.openlocfilehash: 2879ecbc6b50c3cd2bd7541357886f5d6dfe483b
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111752502"
---
# <a name="language-and-voice-support-for-the-speech-service"></a>Compatibilidad con idiomas y voces en el servicio de voz

La compatibilidad con los idiomas varía según la funcionalidad del servicio de voz. En las tablas siguientes se resume la compatibilidad con idiomas para la [conversión de voz a texto](#speech-to-text), [conversión de texto a voz](#text-to-speech) y las ofertas del servicio [Speech Translation](#speech-translation).

## <a name="speech-to-text"></a>Voz a texto

El SDK de Voz de Microsoft y la API REST admiten los siguientes idiomas (configuraciones regionales). 

Para mejorar la precisión, se ofrece la posibilidad de personalizar un subconjunto de idiomas mediante la carga de **audio y transcripciones etiquetadas por humanos** o **texto relacionado: oraciones**. La compatibilidad con la personalización del modelo acústico con **audio + transcripciones etiquetadas por usuarios** se limita a los modelos base específicos que se enumeran a continuación. Otros modelos e idiomas base solo usarán el texto de las transcripciones para entrenar modelos personalizados como con **Texto relacionado: oraciones**. Para más información sobre la personalización, consulte [Introducción a Custom Speech](./custom-speech-overview.md).

<!--
To get the AM and ML bits:
https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetSupportedLocalesForModels

To get pronunciation bits:
https://cris.ai -> Click on Adaptation Data -> scroll down to section "Pronunciation Datasets" -> Click on Import -> Locale: the list of locales there correspond to the supported locales
-->

| Idioma                 | Configuración regional (BCP-47) | Personalizaciones  | [Identificación del idioma](how-to-automatic-language-detection.md) | [Evaluación de la pronunciación](how-to-pronunciation-assessment.md) |
|------------------------------------|--------|---------------------------------------------------|-------------------------------|--------------------------|
| Árabe (Argelia)                   | `ar-DZ` | Texto                                   |                           |                          |
| Árabe (Bahréin), estándar moderno  | `ar-BH` | Texto                                   |                           |                          |
| Árabe (Egipto)                     | `ar-EG` | Texto                                   | Sí                          |                          |
| Árabe (Iraq)                      | `ar-IQ` | Texto                                   |                           |                          |
| Árabe (Israel)                    | `ar-IL` | Texto                                   |                           |                          |
| Árabe (Jordania)                    | `ar-JO` | Texto                                   |                           |                          |
| Árabe (Kuwait)                    | `ar-KW` | Texto                                   |                           |                          |
| Árabe (Líbano)                   | `ar-LB` | Texto                                   |                           |                          |
| Árabe (Libia)                     | `ar-LY` | Texto                                   |                           |                          |
| Árabe (Marruecos)                   | `ar-MA` | Texto                                   |                           |                          |
| Árabe (Omán)                      | `ar-OM` | Texto                                   |                           |                          |
| Árabe (Qatar)                     | `ar-QA` | Texto                                   |                           |                          |
| Árabe (Arabia Saudí)              | `ar-SA` | Texto                                   |                           |                          |
| Árabe (Autoridad Palestina)     | `ar-PS` | Texto                                   |                           |                          |
| Árabe (Siria)                     | `ar-SY` | Texto                                   |                           |                          |
| Árabe (Túnez)                   | `ar-TN` | Texto                                   |                           |                          |
| Árabe (Emiratos Árabes Unidos)      | `ar-AE` | Texto                                   |                           |                          |
| Árabe (Yemen)                     | `ar-YE` | Texto                                   |                           |                          |
| Búlgaro (Bulgaria)               | `bg-BG` | Texto                                   |                           |                          |
| Catalán (España)                    | `ca-ES` | Texto                                   | Sí                          |                          |
| Chino (cantonés, tradicional)   | `zh-HK` | Audio (20201015)<br>Texto                 |        Sí                   |                          |
| Chino (mandarín, simplificado)     | `zh-CN` | Audio (20200910)<br>Texto                 |     Sí                      | Sí                         |
| Chino (mandarín, Taiwán)       | `zh-TW` | Audio (20190701, 20201015)<br>Texto                 |           Sí                |                          |
| Croata (Croacia)                 | `hr-HR` | Texto                                   |                           |                          |
| Checo (República Checa)             | `cs-CZ` | Texto                                   |                           |                          |
| Danés (Dinamarca)                   | `da-DK` | Texto                                   | Sí                          |                          |
| Neerlandés (Países Bajos)                | `nl-NL` | Audio (20201015)<br>Texto<br>Pronunciación|    Sí                       |                          |
| Inglés (Australia)                | `en-AU` | Audio (20201019)<br>Texto                 | Sí                          |                          |
| Inglés (Canadá)                   | `en-CA` | Audio (20201019)<br>Texto                 | Sí                          |                          |
| Inglés (Ghana)                    | `en-GH` | Texto                                   |                           |                          |
| Inglés (Hong Kong)                | `en-HK` | Texto                                   |                           |                          |
| Inglés (India)                    | `en-IN` | Audio (20200923)<br>Texto                 |                          |                          |
| Inglés (Irlanda)                  | `en-IE` | Texto                                   |                           |                          |
| Inglés (Kenia)                    | `en-KE` | Texto                                   |                           |                          |
| Inglés (Nueva Zelanda)              | `en-NZ` | Audio (20201019)<br>Texto                 |                          |                          |
| Inglés (Nigeria)                  | `en-NG` | Texto                                   |                           |                          |
| Inglés (Filipinas)              | `en-PH` | Texto                                   |                           |                          |
| Inglés (Singapur)                | `en-SG` | Texto                                   |                           |                          |
| Inglés (Sudáfrica)             | `en-ZA` | Texto                                   |                           |                          |
| Inglés (Tanzania)                 | `en-TZ` | Texto                                   |                           |                          |
| Inglés (Reino Unido)           | `en-GB` | Audio (20201019)<br>Texto<br>Pronunciación| Sí                          | Sí                         |
| Spanish (Traditional Sort) - Spain            | `en-US` | Audio (20201019, 20210223)<br>Texto<br>Pronunciación| Sí                          | Sí                         |
| Estonio (Estonia)                  | `et-EE` | Texto                                   |                           |                          |
| Filipino (Filipinas)             | `fil-PH`| Texto                                   |                           |                          |
| Finés (Finlandia)                  | `fi-FI` | Texto                                   |     Sí                      |                          |
| Francés (Canadá)                    | `fr-CA` | Audio (20201015)<br>Texto<br>Pronunciación|     Sí                      |                          |
| Francés (Francia)                    | `fr-FR` | Audio (20201015)<br>Texto<br>Pronunciación|      Sí                     |                          |
| Francés (Suiza)               | `fr-CH` | Texto<br>Pronunciación                  |                           |                          |
| Alemán (Austria)                   | `de-AT` | Texto<br>Pronunciación                  |                           |                          |
| Alemán (Alemania)                   | `de-DE` | Audio (20190701, 20200619, 20201127)<br>Texto<br>Pronunciación|  Sí                         |                          |
| Griego (Grecia)                     | `el-GR` | Texto                                   |  Sí                         |                          |
| Gujarati (India)                  | `gu-IN` | Texto                                   |                           |                          |
| Hebreo (Israel)                    | `he-IL` | Texto                                   |                           |                          |
| Hindi (India)                      | `hi-IN` | Audio (20200701)<br>Texto                 |     Sí                      |                          |
| Húngaro (Hungría)                | `hu-HU` | Texto                                   |                           |                          |
| Indonesio (Indonesia)             | `id-ID` | Texto                                   |                           |                          |
| Irlandés (Irlanda)                     | `ga-IE` | Texto                                   |                           |                          |
| Italiano (Italia)                    | `it-IT` | Audio (20201016)<br>Texto<br>Pronunciación|      Sí                     |                          |
| Japonés (Japón)                   | `ja-JP` | Texto                                   |      Sí                     |                          |
| Coreano (Corea)                     | `ko-KR` | Audio (20201015)<br>Texto                 |      Sí                     |                          |
| Letón (Letonia)                   | `lv-LV` | Texto                                   |                           |                          |
| Lituano (Lituania)             | `lt-LT` | Texto                                   |                           |                          |
| Malayo (Malasia)                   | `ms-MY` | Texto                                   |                           |                          |
| Maltés (Malta)                    | `mt-MT` | Texto                                   |                           |                          |
| Maratí (India)                    | `mr-IN` | Texto                                   |                           |                          |
| Noruego (Bokmål, Noruega)         | `nb-NO` | Texto                                   |     Sí                      |                          |
| Polaco (Polonia)                    | `pl-PL` | Texto                                   |       Sí                    |                          |
| Portugués (Brasil)                | `pt-BR` | Audio (20190620, 20201015)<br>Texto<br>Pronunciación|          Sí                 |                          |
| Portugués (Portugal)              | `pt-PT` | Texto<br>Pronunciación                  |             Sí              |                          |
| Rumano (Rumanía)                 | `ro-RO` | Texto                                   |  Sí                         |                          |
| Ruso (Rusia)                   | `ru-RU` | Audio (20200907)<br>Texto                 |                Sí           |                          |
| Eslovaco (Eslovaquia)                  | `sk-SK` | Texto                                   |                           |                          |
| Esloveno (Eslovenia)               | `sl-SI` | Texto                                   |                           |                          |
| Español (Argentina)                | `es-AR` | Texto<br>Pronunciación                  |                           |                          |
| Español (Bolivia)                  | `es-BO` | Texto<br>Pronunciación                  |                           |                          |
| Español (Chile)                    | `es-CL` | Texto<br>Pronunciación                  |                           |                          |
| Español (Colombia)                 | `es-CO` | Texto<br>Pronunciación                  |                           |                          |
| Español (Costa Rica)               | `es-CR` | Texto<br>Pronunciación                  |                           |                          |
| Español (Cuba)                     | `es-CU` | Texto<br>Pronunciación                  |                           |                          |
| Español (República Dominicana)       | `es-DO` | Texto<br>Pronunciación                  |                           |                          |
| Español (Ecuador)                  | `es-EC` | Texto<br>Pronunciación                  |                           |                          |
| Español (El Salvador)              | `es-SV` | Texto<br>Pronunciación                  |                           |                          |
| Español (Guinea Ecuatorial)        | `es-GQ` | Texto                                   |                           |                          |
| Español (Guatemala)                | `es-GT` | Texto<br>Pronunciación                  |                           |                          |
| Español (Honduras)                 | `es-HN` | Texto<br>Pronunciación                  |                           |                          |
| Español (México)                   | `es-MX` | Audio (20200907)<br>Texto<br>Pronunciación|    Sí                       |                          |
| Español (Nicaragua)                | `es-NI` | Texto<br>Pronunciación                  |                           |                          |
| Español (Panamá)                   | `es-PA` | Texto<br>Pronunciación                  |                           |                          |
| Español (Paraguay)                 | `es-PY` | Texto<br>Pronunciación                  |                           |                          |
| Español (Perú)                     | `es-PE` | Texto<br>Pronunciación                  |                           |                          |
| Español (Puerto Rico)              | `es-PR` | Texto<br>Pronunciación                  |                           |                          |
| Español (España)                    | `es-ES` | Audio (20201015)<br>Texto<br>Pronunciación|  Sí                         |                          |
| Español (Uruguay)                  | `es-UY` | Texto<br>Pronunciación                  |                           |                          |
| Español (EE. UU.)                      | `es-US` | Texto<br>Pronunciación                  |                           |                          |
| Español (Venezuela)                | `es-VE` | Texto<br>Pronunciación                  |                           |                          |
| Sueco (Suecia)                   | `sv-SE` | Texto                                   |   Sí                        |                          |
| Tamil (India)                      | `ta-IN` | Texto                                   |                           |                          |
| Telugu (India)                     | `te-IN` | Texto                                   |                           |                          |
| Tailandés (Tailandia)                    | `th-TH` | Texto                                   |      Sí                     |                          |
| Turco (Turquía)                   | `tr-TR` | Texto                                   |                           |                          |
| Vietnamita (Vietnam)               | `vi-VN` | Texto                                   |                           |                          |

## <a name="text-to-speech"></a>Texto a voz

Tanto el SDK de Voz de Microsoft como las API REST admiten estas voces, y cada una de ellas admite un idioma y un dialecto específicos, que se identifican mediante la configuración regional. También puede obtener una lista completa de los idiomas y las voces que se admiten para cada región o punto de conexión específico a través de la [API de la lista de voces](rest-text-to-speech.md#get-a-list-of-voices). 

> [!IMPORTANT]
> Los precios son distintos para voces estándar, personalizadas y neuronales. Consulte la página de [precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/) para más información.

### <a name="neural-voices"></a>Voces neuronales

Texto a voz neuronal es un nuevo tipo de síntesis de voz con tecnología de redes neuronales profundas. Cuando se usa una voz neuronal, es prácticamente imposible distinguir la voz sintetizada de las grabaciones humanas.

Las voces neuronales se pueden usar para que las interacciones con los bots de chat y los asistentes de voz sean más naturales y atractivas, para convertir textos digitales (por ejemplo, los libros electrónicos) en audiolibros y para mejorar los sistemas de navegación de los automóviles. Con su prosodia natural similar a la humana y la clara articulación de las palabras, las voces neuronales reducen considerablemente la fatiga de la escucha que aparece cuando los usuarios interactúan con sistemas de inteligencia artificial.

> [!NOTE]
> Las voces neuronales se crean a partir de ejemplos que usan una frecuencia de muestreo de 24 kHz.
> Todas las voces pueden aumentar o reducir las frecuencias de muestreo al sintetizar.


| Idioma | Configuración regional | Sexo | Nombre de voz | Compatibilidad de estilo |
|---|---|---|---|---|
| Árabe (Egipto) | `ar-EG` | Female | `ar-EG-SalmaNeural` | General |
| Árabe (Egipto) | `ar-EG` | Male | `ar-EG-ShakirNeural` | General |
| Árabe (Arabia Saudí) | `ar-SA` | Female | `ar-SA-ZariyahNeural` | General |
| Árabe (Arabia Saudí) | `ar-SA` | Male | `ar-SA-HamedNeural` | General |
| Búlgaro (Bulgaria) | `bg-BG` | Female | `bg-BG-KalinaNeural` | General |
| Búlgaro (Bulgaria) | `bg-BG` | Male | `bg-BG-BorislavNeural` | General |
| Catalán (España) | `ca-ES` | Female | `ca-ES-AlbaNeural` | General |
| Catalán (España) | `ca-ES` | Female | `ca-ES-JoanaNeural` | General |
| Catalán (España) | `ca-ES` | Male | `ca-ES-EnricNeural` | General |
| Chino (cantonés, tradicional) | `zh-HK` | Female | `zh-HK-HiuGaaiNeural` | General |
| Chino (cantonés, tradicional) | `zh-HK` | Female | `zh-HK-HiuMaanNeural` | General |
| Chino (cantonés, tradicional) | `zh-HK` | Male | `zh-HK-WanLungNeural` | General |
| Chino (mandarín, simplificado) | `zh-CN` | Female | `zh-CN-XiaoxiaoNeural` | General, varios estilos de voz disponibles [mediante SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chino (mandarín, simplificado) | `zh-CN` | Female | `zh-CN-XiaoyouNeural` | Voz de niño, optimizado para la narración de historias |
| Chino (mandarín, simplificado) | `zh-CN` | Female | `zh-CN-XiaomoNeural` | General, varios estilos y representaciones disponibles [mediante SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chino (mandarín, simplificado) | `zh-CN` | Female | `zh-CN-XiaoxuanNeural` | General, varios estilos y representaciones disponibles [mediante SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chino (mandarín, simplificado) | `zh-CN` | Female | `zh-CN-XiaohanNeural` | General, varios estilos disponibles [mediante SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chino (mandarín, simplificado) | `zh-CN` | Female | `zh-CN-XiaoruiNeural` | Voz de adulto de edad avanzada, varios estilos disponibles [mediante SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chino (mandarín, simplificado) | `zh-CN` | Male | `zh-CN-YunyangNeural` | Optimizado para lectura de noticias,<br /> varios estilos de voz disponibles [mediante SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chino (mandarín, simplificado) | `zh-CN` | Male | `zh-CN-YunyeNeural` | Optimizado para la narración de historias |
| Chino (mandarín, simplificado) | `zh-CN` | Male   | `zh-CN-YunxiNeural` | General, varios estilos disponibles [mediante SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chino (mandarín, Taiwán) | `zh-TW` | Female | `zh-TW-HsiaoChenNeural` | General |
| Chino (mandarín, Taiwán) | `zh-TW` | Female | `zh-TW-HsiaoYuNeural` | General |
| Chino (mandarín, Taiwán) | `zh-TW` | Male | `zh-TW-YunJheNeural` | General |
| Croata (Croacia) | `hr-HR` | Female | `hr-HR-GabrijelaNeural` | General |
| Croata (Croacia) | `hr-HR` | Male | `hr-HR-SreckoNeural` | General |
| Checo (República Checa) | `cs-CZ` | Female | `cs-CZ-VlastaNeural` | General |
| Checo (República Checa) | `cs-CZ` | Male | `cs-CZ-AntoninNeural` | General |
| Danés (Dinamarca) | `da-DK` | Female | `da-DK-ChristelNeural` | General |
| Danés (Dinamarca) | `da-DK` | Male | `da-DK-JeppeNeural` | General |
| Neerlandés (Bélgica) | `nl-BE` | Female | `nl-BE-DenaNeural` | General | 
| Neerlandés (Bélgica) | `nl-BE` | Male | `nl-BE-ArnaudNeural` | General | 
| Neerlandés (Países Bajos) | `nl-NL` | Female | `nl-NL-ColetteNeural` | General |
| Neerlandés (Países Bajos) | `nl-NL` | Female | `nl-NL-FennaNeural` | General |
| Neerlandés (Países Bajos) | `nl-NL` | Male | `nl-NL-MaartenNeural` | General |
| Inglés (Australia) | `en-AU` | Female | `en-AU-NatashaNeural` | General |
| Inglés (Australia) | `en-AU` | Male | `en-AU-WilliamNeural` | General |
| Inglés (Canadá) | `en-CA` | Female | `en-CA-ClaraNeural` | General |
| Inglés (Canadá) | `en-CA` | Male | `en-CA-LiamNeural` | General |
| Inglés (Hong Kong) | `en-HK` | Female | `en-HK-YanNeural` <sup>Nuevo</sup> | General |
| Inglés (Hong Kong) | `en-HK` | Male | `en-HK-SamNeural` <sup>Nuevo</sup> | General |
| Inglés (India) | `en-IN` | Female | `en-IN-NeerjaNeural` | General |
| Inglés (India) | `en-IN` | Male | `en-IN-PrabhatNeural` | General |
| Inglés (Irlanda) | `en-IE` | Female | `en-IE-EmilyNeural` | General |
| Inglés (Irlanda) | `en-IE` | Male | `en-IE-ConnorNeural` | General |
| Inglés (Nueva Zelanda) | `en-NZ` | Female | `en-NZ-MollyNeural` <sup>Nuevo</sup> | General |
| Inglés (Nueva Zelanda) | `en-NZ` | Male | `en-NZ-MitchellNeural` <sup>Nuevo</sup> | General |
| Inglés (Filipinas) | `en-PH` | Female | `en-PH-RosaNeural` | General | 
| Inglés (Filipinas) | `en-PH` | Male | `en-PH-JamesNeural` | General | 
| Inglés (Singapur) | `en-SG` | Female | `en-SG-LunaNeural` <sup>Nuevo</sup> | General |
| Inglés (Singapur) | `en-SG` | Male | `en-SG-WayneNeural` <sup>Nuevo</sup> | General |
| Inglés (Sudáfrica) | `en-ZA` | Female | `en-ZA-LeahNeural` <sup>Nuevo</sup> | General |
| Inglés (Sudáfrica) | `en-ZA` | Male | `en-ZA-LukeNeural` <sup>Nuevo</sup> | General |
| Inglés (Reino Unido) | `en-GB` | Female | `en-GB-LibbyNeural` | General |
| Inglés (Reino Unido) | `en-GB` | Female | `en-GB-MiaNeural` | General |
| Inglés (Reino Unido) | `en-GB` | Male | `en-GB-RyanNeural` | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Female | `en-US-AriaNeural` | General, varios estilos de voz disponibles [mediante SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Spanish (Traditional Sort) - Spain | `en-US` | Female | `en-US-JennyNeural` | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Male | `en-US-GuyNeural` | General |
| Estonio (Estonia) | `et-EE` | Female | `et-EE-AnuNeural` | General |
| Estonio (Estonia) | `et-EE` | Male | `et-EE-KertNeural` | General |
| Finés (Finlandia) | `fi-FI` | Female | `fi-FI-NooraNeural` | General |
| Finés (Finlandia) | `fi-FI` | Female | `fi-FI-SelmaNeural` | General |
| Finés (Finlandia) | `fi-FI` | Male | `fi-FI-HarriNeural` | General |
| Francés (Bélgica) | `fr-BE` | Female | `fr-BE-CharlineNeural` | General | 
| Francés (Bélgica) | `fr-BE` | Male | `fr-BE-GerardNeural` | General | 
| Francés (Canadá) | `fr-CA` | Female | `fr-CA-SylvieNeural` | General |
| Francés (Canadá) | `fr-CA` | Male | `fr-CA-AntoineNeural` | General |
| Francés (Canadá) | `fr-CA` | Male | `fr-CA-JeanNeural` | General |
| Francés (Francia) | `fr-FR` | Female | `fr-FR-DeniseNeural` | General |
| Francés (Francia) | `fr-FR` | Male | `fr-FR-HenriNeural` | General |
| Francés (Suiza) | `fr-CH` | Female | `fr-CH-ArianeNeural` | General |
| Francés (Suiza) | `fr-CH` | Male | `fr-CH-FabriceNeural` | General |
| Alemán (Austria) | `de-AT` | Female | `de-AT-IngridNeural` | General |
| Alemán (Austria) | `de-AT` | Male | `de-AT-JonasNeural` | General |
| Alemán (Alemania) | `de-DE` | Female | `de-DE-KatjaNeural` | General |
| Alemán (Alemania) | `de-DE` | Male | `de-DE-ConradNeural` | General |
| Alemán (Suiza) | `de-CH` | Female | `de-CH-LeniNeural` | General |
| Alemán (Suiza) | `de-CH` | Male | `de-CH-JanNeural` | General |
| Griego (Grecia) | `el-GR` | Female | `el-GR-AthinaNeural` | General |
| Griego (Grecia) | `el-GR` | Male | `el-GR-NestorasNeural` | General |
| Gujarati (India) | `gu-IN` | Female | `gu-IN-DhwaniNeural` <sup>Nuevo</sup> | General |
| Gujarati (India) | `gu-IN` | Male | `gu-IN-NiranjanNeural` <sup>Nuevo</sup> | General |
| Hebreo (Israel) | `he-IL` | Female | `he-IL-HilaNeural` | General |
| Hebreo (Israel) | `he-IL` | Male | `he-IL-AvriNeural` | General |
| Hindi (India) | `hi-IN` | Female | `hi-IN-SwaraNeural` | General |
| Hindi (India) | `hi-IN` | Male | `hi-IN-MadhurNeural` | General |
| Húngaro (Hungría) | `hu-HU` | Female | `hu-HU-NoemiNeural` | General |
| Húngaro (Hungría) | `hu-HU` | Male | `hu-HU-TamasNeural` | General |
| Indonesio (Indonesia) | `id-ID` | Female | `id-ID-GadisNeural` | General |
| Indonesio (Indonesia) | `id-ID` | Male | `id-ID-ArdiNeural` | General |
| Irlandés (Irlanda) | `ga-IE` | Female | `ga-IE-OrlaNeural` | General |
| Irlandés (Irlanda) | `ga-IE` | Male | `ga-IE-ColmNeural` | General |
| Italiano (Italia) | `it-IT` | Female | `it-IT-ElsaNeural` | General |
| Italiano (Italia) | `it-IT` | Female | `it-IT-IsabellaNeural` | General |
| Italiano (Italia) | `it-IT` | Male | `it-IT-DiegoNeural` | General |
| Japonés (Japón) | `ja-JP` | Female | `ja-JP-NanamiNeural` | General |
| Japonés (Japón) | `ja-JP` | Male | `ja-JP-KeitaNeural` | General |
| Coreano (Corea) | `ko-KR` | Female | `ko-KR-SunHiNeural` | General |
| Coreano (Corea) | `ko-KR` | Male | `ko-KR-InJoonNeural` | General |
| Letón (Letonia) | `lv-LV` | Female | `lv-LV-EveritaNeural` | General |
| Letón (Letonia) | `lv-LV` | Male | `lv-LV-NilsNeural` | General |
| Lituano (Lituania) | `lt-LT` | Female | `lt-LT-OnaNeural` | General |
| Lituano (Lituania) | `lt-LT` | Male | `lt-LT-LeonasNeural` | General |
| Malayo (Malasia) | `ms-MY` | Female | `ms-MY-YasminNeural` | General |
| Malayo (Malasia) | `ms-MY` | Male | `ms-MY-OsmanNeural` | General |
| Maltés (Malta) | `mt-MT` | Female | `mt-MT-GraceNeural` | General |
| Maltés (Malta) | `mt-MT` | Male | `mt-MT-JosephNeural` | General |
| Maratí (India) | `mr-IN` | Female | `mr-IN-AarohiNeural` <sup>Nuevo</sup> | General |
| Maratí (India) | `mr-IN` | Male | `mr-IN-ManoharNeural` <sup>Nuevo</sup> | General |
| Noruego (Bokmål, Noruega) | `nb-NO` | Female | `nb-NO-IselinNeural` | General |
| Noruego (Bokmål, Noruega) | `nb-NO` | Female | `nb-NO-PernilleNeural` | General |
| Noruego (Bokmål, Noruega) | `nb-NO` | Male | `nb-NO-FinnNeural` | General |
| Polaco (Polonia) | `pl-PL` | Female | `pl-PL-AgnieszkaNeural` | General |
| Polaco (Polonia) | `pl-PL` | Female | `pl-PL-ZofiaNeural` | General |
| Polaco (Polonia) | `pl-PL` | Male | `pl-PL-MarekNeural` | General |
| Portugués (Brasil) | `pt-BR` | Female | `pt-BR-FranciscaNeural` | General, varios estilos de voz disponibles [mediante SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Portugués (Brasil) | `pt-BR` | Male | `pt-BR-AntonioNeural` | General |
| Portugués (Portugal) | `pt-PT` | Female | `pt-PT-FernandaNeural` | General |
| Portugués (Portugal) | `pt-PT` | Female | `pt-PT-RaquelNeural` | General |
| Portugués (Portugal) | `pt-PT` | Male | `pt-PT-DuarteNeural` | General |
| Rumano (Rumanía) | `ro-RO` | Female | `ro-RO-AlinaNeural` | General |
| Rumano (Rumanía) | `ro-RO` | Male | `ro-RO-EmilNeural` | General |
| Ruso (Rusia) | `ru-RU` | Female | `ru-RU-DariyaNeural` | General |
| Ruso (Rusia) | `ru-RU` | Female | `ru-RU-SvetlanaNeural` | General |
| Ruso (Rusia) | `ru-RU` | Male | `ru-RU-DmitryNeural` | General |
| Eslovaco (Eslovaquia) | `sk-SK` | Female | `sk-SK-ViktoriaNeural` | General |
| Eslovaco (Eslovaquia) | `sk-SK` | Male | `sk-SK-LukasNeural` | General |
| Esloveno (Eslovenia) | `sl-SI` | Female | `sl-SI-PetraNeural` | General |
| Esloveno (Eslovenia) | `sl-SI` | Male | `sl-SI-RokNeural` | General |
| Español (Argentina) | `es-AR` | Female | `es-AR-ElenaNeural` <sup>Nuevo</sup> | General |
| Español (Argentina) | `es-AR` | Male | `es-AR-TomasNeural` <sup>Nuevo</sup> | General |
| Español (Colombia) | `es-CO` | Female | `es-CO-SalomeNeural` <sup>Nuevo</sup> | General |
| Español (Colombia) | `es-CO` | Male | `es-CO-GonzaloNeural` <sup>Nuevo</sup> | General |
| Español (México) | `es-MX` | Female | `es-MX-DaliaNeural` | General |
| Español (México) | `es-MX` | Male | `es-MX-JorgeNeural` | General |
| Español (España) | `es-ES` | Female | `es-ES-ElviraNeural` | General |
| Español (España) | `es-ES` | Male | `es-ES-AlvaroNeural` | General |
| Español (EE. UU.) | `es-US` | Female | `es-US-PalomaNeural` <sup>Nuevo</sup> | General |
| Español (EE. UU.) | `es-US` | Male | `es-US-AlonsoNeural` <sup>Nuevo</sup> | General |
| Swahili (Kenia) | `sw-KE` | Female | `sw-KE-ZuriNeural` <sup>Nuevo</sup> | General |
| Swahili (Kenia) | `sw-KE` | Male | `sw-KE-RafikiNeural` <sup>Nuevo</sup> | General |
| Sueco (Suecia) | `sv-SE` | Female | `sv-SE-HilleviNeural` | General |
| Sueco (Suecia) | `sv-SE` | Female | `sv-SE-SofieNeural` | General |
| Sueco (Suecia) | `sv-SE` | Male | `sv-SE-MattiasNeural` | General |
| Tamil (India) | `ta-IN` | Female | `ta-IN-PallaviNeural` | General |
| Tamil (India) | `ta-IN` | Male | `ta-IN-ValluvarNeural` | General |
| Telugu (India) | `te-IN` | Female | `te-IN-ShrutiNeural` | General |
| Telugu (India) | `te-IN` | Male | `te-IN-MohanNeural` | General |
| Tailandés (Tailandia) | `th-TH` | Female | `th-TH-AcharaNeural` | General |
| Tailandés (Tailandia) | `th-TH` | Female | `th-TH-PremwadeeNeural` | General |
| Tailandés (Tailandia) | `th-TH` | Male | `th-TH-NiwatNeural` | General |
| Turco (Turquía) | `tr-TR` | Female | `tr-TR-EmelNeural` | General |
| Turco (Turquía) | `tr-TR` | Male | `tr-TR-AhmetNeural` | General |
| Ucraniano (Ucrania) | `uk-UA` | Female | `uk-UA-PolinaNeural` | General | 
| Ucraniano (Ucrania) | `uk-UA` | Male | `uk-UA-OstapNeural` | General | 
| Urdú (Pakistán) | `ur-PK` | Female | `ur-PK-UzmaNeural`  | General | 
| Urdú (Pakistán) | `ur-PK` | Male | `ur-PK-AsadNeural` | General | 
| Vietnamita (Vietnam) | `vi-VN` | Female | `vi-VN-HoaiMyNeural` | General |
| Vietnamita (Vietnam) | `vi-VN` | Male | `vi-VN-NamMinhNeural` | General |
| Galés (Reino Unido) | `cy-GB` | Female | `cy-GB-NiaNeural` | General | 
| Galés (Reino Unido) | `cy-GB` | Male | `cy-GB-AledNeural` | General | 

#### <a name="neural-voices-in-preview"></a>Voces neuronales en versión preliminar

Las siguientes voces neuronales se encuentran en versión preliminar pública. 

| Idioma                         | Configuración regional  | Sexo | Nombre de voz                             | Compatibilidad de estilo |
|----------------------------------|---------|--------|----------------------------------------|---------------|
| Spanish (Traditional Sort) - Spain | `en-US` | Female | `en-US-AmberNeural` <sup>Nuevo</sup> | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Female | `en-US-AshleyNeural` <sup>Nuevo</sup> | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Female | `en-US-CoraNeural` <sup>Nuevo</sup> | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Female | `en-US-ElizabethNeural` <sup>Nuevo</sup> | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Female | `en-US-JennyMultilingualNeural` <sup>Nuevo</sup> | Funcionalidades generales multilingües disponibles [mediante SSML](speech-synthesis-markup.md#create-an-ssml-document) |
| Spanish (Traditional Sort) - Spain | `en-US` | Female | `en-US-MichelleNeural` <sup>Nuevo</sup> | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Female | `en-US-MonicaNeural` <sup>Nuevo</sup> | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Niño | `en-US-AnaNeural` <sup>Nuevo</sup> | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Male | `en-US-BrandonNeural` <sup>Nuevo</sup> | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Male | `en-US-ChristopherNeural` <sup>Nuevo</sup> | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Male | `en-US-JacobNeural` <sup>Nuevo</sup> | General |
| Spanish (Traditional Sort) - Spain | `en-US` | Male | `en-US-EricNeural` <sup>Nuevo</sup> | General |

> [!IMPORTANT]
> Las voces en versión preliminar pública solo están disponibles en tres regiones del servicio: Este de EE. UU., Oeste de Europa y Sudeste de Asia.

> [!TIP]
> `en-US-JennyNeuralMultilingual` admite varios idiomas. Compruebe la [API de la lista de voces](rest-text-to-speech.md#get-a-list-of-voices) para obtener la lista de idiomas admitidos.

Para más información acerca de la disponibilidad regional, consulte las [regiones](regions.md#neural-and-standard-voices).

Para más información sobre cómo configurar y ajustar las voces neuronales, por ejemplo, los estilos de habla, consulte [Lenguaje de marcado de síntesis de voz](speech-synthesis-markup.md#adjust-speaking-styles).

> [!IMPORTANT]
> La voz `en-US-JessaNeural` ha cambiado a `en-US-AriaNeural`. Si usaba "Jessa" antes, conviértalo a "Aria".

> [!TIP]
> Puede seguir usando la asignación de nombre completo de servicio, como "Microsoft Server Speech Text to Speech Voice (en-US, AriaNeural)", en sus solicitudes de síntesis de voz.

### <a name="standard-voices"></a>Voces estándar

Hay más de 75 voces estándar disponibles en más de 45 idiomas y configuraciones regionales, lo que le permite convertir texto en voz sintetizada. Para más información acerca de la disponibilidad regional, consulte las [regiones](regions.md#neural-and-standard-voices).

> [!NOTE]
> Con dos excepciones, las voces neuronales estándar se crean a partir de ejemplos que usan una frecuencia de muestreo de 16 kHz.
> Las voces **en-US-AriaRUS** y **en-US-GuyRUS** también se crean a partir de ejemplos que usan una frecuencia de muestreo de 24 kHz.
> Todas las voces pueden aumentar o reducir las frecuencias de muestreo al sintetizar.

| Idioma | Configuración regional (BCP-47) | Sexo | Nombre de voz |
|--|--|--|--|
| Árabe (árabe) | `ar-EG` | Female | `ar-EG-Hoda`|
| Árabe (Arabia Saudí) | `ar-SA` | Male | `ar-SA-Naayf`|
| Búlgaro (Bulgaria) | `bg-BG` | Male | `bg-BG-Ivan`|
| Catalán (España) | `ca-ES` | Female | `ca-ES-HerenaRUS`|
| Chino (cantonés, tradicional) | `zh-HK` | Male | `zh-HK-Danny`|
| Chino (cantonés, tradicional) | `zh-HK` | Female | `zh-HK-TracyRUS`|
| Chino (mandarín, simplificado) | `zh-CN` | Female | `zh-CN-HuihuiRUS`|
| Chino (mandarín, simplificado) | `zh-CN` | Male | `zh-CN-Kangkang`|
| Chino (mandarín, simplificado) | `zh-CN` | Female | `zh-CN-Yaoyao`|
| Chino (mandarín, Taiwán) |  `zh-TW` | Female | `zh-TW-HanHanRUS`|
| Chino (mandarín, Taiwán) |  `zh-TW` | Female | `zh-TW-Yating`|
| Chino (mandarín, Taiwán) |  `zh-TW` | Male | `zh-TW-Zhiwei`|
| Croata (Croacia) | `hr-HR` | Male | `hr-HR-Matej`|
| Checo (República Checa) | `cs-CZ` | Male | `cs-CZ-Jakub`|
| Danés (Dinamarca) | `da-DK` | Female | `da-DK-HelleRUS`|
| Neerlandés (Países Bajos) | `nl-NL` | Female | `nl-NL-HannaRUS`|
| Inglés (Australia) | `en-AU` | Female | `en-AU-Catherine`|
| Inglés (Australia) | `en-AU` | Female | `en-AU-HayleyRUS`|
| Inglés (Canadá) | `en-CA` | Female | `en-CA-HeatherRUS`|
| Inglés (Canadá) | `en-CA` | Female | `en-CA-Linda`|
| Inglés (India) | `en-IN` | Female | `en-IN-Heera`|
| Inglés (India) | `en-IN` | Female | `en-IN-PriyaRUS`|
| Inglés (India) | `en-IN` | Male | `en-IN-Ravi`|
| Inglés (Irlanda) | `en-IE` | Male | `en-IE-Sean`|
| Inglés (Reino Unido) | `en-GB` | Male | `en-GB-George`|
| Inglés (Reino Unido) | `en-GB` | Female | `en-GB-HazelRUS`|
| Inglés (Reino Unido) | `en-GB` | Female | `en-GB-Susan`|
| Spanish (Traditional Sort) - Spain | `en-US` | Male | `en-US-BenjaminRUS`|
| Spanish (Traditional Sort) - Spain | `en-US` | Male | `en-US-GuyRUS`|
| Spanish (Traditional Sort) - Spain | `en-US` | Female | `en-US-AriaRUS`|
| Spanish (Traditional Sort) - Spain | `en-US` | Female | `en-US-ZiraRUS`|
| Finés (Finlandia) | `fi-FI` | Female | `fi-FI-HeidiRUS`|
| Francés (Canadá) | `fr-CA` | Female | `fr-CA-Caroline`|
| Francés (Canadá) | `fr-CA` | Female | `fr-CA-HarmonieRUS`|
| Francés (Francia) | `fr-FR` | Female | `fr-FR-HortenseRUS`|
| Francés (Francia) | `fr-FR` | Female | `fr-FR-Julie`|
| Francés (Francia) | `fr-FR` | Male | `fr-FR-Paul`|
| Francés (Suiza) | `fr-CH` | Male | `fr-CH-Guillaume`|
| Alemán (Austria) | `de-AT` | Male | `de-AT-Michael`|
| Alemán (Alemania) | `de-DE` | Female | `de-DE-HeddaRUS`|
| Alemán (Alemania) | `de-DE` | Male | `de-DE-Stefan`|
| Alemán (Suiza) | `de-CH` | Male | `de-CH-Karsten`|
| Griego (Grecia) | `el-GR` | Male | `el-GR-Stefanos`|
| Hebreo (Israel) | `he-IL` | Male | `he-IL-Asaf`|
| Hindi (India) | `hi-IN` | Male | `hi-IN-Hemant`|
| Hindi (India) | `hi-IN` | Female | `hi-IN-Kalpana`|
| Húngaro (Hungría) | `hu-HU` | Male | `hu-HU-Szabolcs`|
| Indonesio (Indonesia) | `id-ID` | Male | `id-ID-Andika`|
| Italiano (Italia) | `it-IT` | Male | `it-IT-Cosimo`|
| Italiano (Italia) | `it-IT` | Female | `it-IT-LuciaRUS`|
| Japonés (Japón) | `ja-JP` | Female | `ja-JP-Ayumi`|
| Japonés (Japón) | `ja-JP` | Female | `ja-JP-HarukaRUS`|
| Japonés (Japón) | `ja-JP` | Male | `ja-JP-Ichiro`|
| Coreano (Corea) | `ko-KR` | Female | `ko-KR-HeamiRUS`|
| Malayo (Malasia) | `ms-MY` | Male | `ms-MY-Rizwan`|
| Noruego (Bokmål, Noruega) | `nb-NO` | Female | `nb-NO-HuldaRUS`|
| Polaco (Polonia) | `pl-PL` | Female | `pl-PL-PaulinaRUS`|
| Portugués (Brasil) | `pt-BR` | Male | `pt-BR-Daniel`|
| Portugués (Brasil) | `pt-BR` | Female | `pt-BR-HeloisaRUS`|
| Portugués (Portugal) | `pt-PT` | Female | `pt-PT-HeliaRUS`|
| Rumano (Rumanía) | `ro-RO` | Male | `ro-RO-Andrei`|
| Ruso (Rusia) | `ru-RU` | Female | `ru-RU-EkaterinaRUS`|
| Ruso (Rusia) | `ru-RU` | Female | `ru-RU-Irina`|
| Ruso (Rusia) | `ru-RU` | Male | `ru-RU-Pavel`|
| Eslovaco (Eslovaquia) | `sk-SK` | Male | `sk-SK-Filip`|
| Esloveno (Eslovenia) | `sl-SI` | Male | `sl-SI-Lado`|
| Español (México) | `es-MX` | Female | `es-MX-HildaRUS`|
| Español (México) | `es-MX` | Male | `es-MX-Raul`|
| Español (España) | `es-ES` | Female | `es-ES-HelenaRUS`|
| Español (España) | `es-ES` | Female | `es-ES-Laura`|
| Español (España) | `es-ES` | Male | `es-ES-Pablo`|
| Sueco (Suecia) | `sv-SE` | Female | `sv-SE-HedvigRUS`|
| Tamil (India) | `ta-IN` | Male | `ta-IN-Valluvar`|
| Telugu (India) | `te-IN` | Female | `te-IN-Chitra`|
| Tailandés (Tailandia) | `th-TH` | Male | `th-TH-Pattara`|
| Turco (Turquía) | `tr-TR` | Female | `tr-TR-SedaRUS`|
| Vietnamita (Vietnam) | `vi-VN` | Male | `vi-VN-An` |

> [!IMPORTANT]
> La voz `en-US-Jessa` ha cambiado a `en-US-Aria`. Si usaba "Jessa" antes, conviértalo a "Aria".

> [!TIP]
> Puede seguir usando la asignación de nombre completo de servicio, como "Microsoft Server Speech Text to Speech Voice (en-US, AriaRUS)", en sus solicitudes de síntesis de voz.

### <a name="customization"></a>Personalización

Voz personalizada está disponible en el nivel neuronal (conocido también como Voz neuronal personalizada). En función de la tecnología TTS neuronal y el modelo universal multilingüe de varios hablantes, Voz neuronal personalizada permite crear voces sintéticas ricas en estilos de habla o en idiomas cruzados adaptables. Compruebe a continuación los idiomas admitidos.  

> [!IMPORTANT]
> El nivel estándar, incluidos los métodos de entrenamiento concatenativo y paramétrico estadístico de la voz personalizada, quedará en desuso y se retirará el 29/2/2024. Si usa Voz personalizada estándar o no neuronal, migre a Voz neuronal personalizada inmediatamente para disfrutar de la mejor calidad e implementar las voces de forma responsable. 

| Idioma | Configuración regional | Neuronal | Multilingüe |
|--|--|--|--|
| Búlgaro (Bulgaria)| `bg-BG` | Sí | No |
| Chino (mandarín, simplificado) | `zh-CN` | Sí | Sí |
| Chino (mandarín, simplificado), inglés bilingüe | `zh-CN` bilingüe | Sí | Sí |
| Neerlandés (Países Bajos)   | `nl-NL` | Sí | No |
| Inglés (Australia) | `en-AU` | Sí | Sí |
| Inglés (India) | `en-IN` | Sí | No |
| Inglés (Reino Unido) | `en-GB` | Sí | Sí |
| Spanish (Traditional Sort) - Spain | `en-US` | Sí | Sí |
| Francés (Canadá) | `fr-CA` | Sí | Sí |
| Francés (Francia) | `fr-FR` | Sí | Sí |
| Alemán (Alemania) | `de-DE` | Sí | Sí |
| Italiano (Italia) | `it-IT` | Sí | Sí |
| Japonés (Japón) | `ja-JP` | Sí | Sí |
| Coreano (Corea) | `ko-KR` | Sí | Sí |
| Noruego (Bokmål, Noruega) | `nb-NO` | Sí | No |
| Portugués (Brasil) | `pt-BR` | Sí | Sí |
| Español (México) | `es-MX` | Sí | Sí |
| Español (España) | `es-ES` | Sí | Sí |

Seleccione la configuración regional adecuada que coincida con los datos de entrenamiento que tiene para entrenar un modelo de voz personalizado. Por ejemplo, si los datos de grabación que tienen que hablar en inglés con acento británico, seleccione `en-GB`.

> [!NOTE]
> No se admite el entrenamiento de modelos bilingües en la funcionalidad de voz personalizada, excepto en el caso de chino-inglés bilingüe. Seleccione la opción de chino-inglés bilingüe si quiere entrenar una voz china que pueda hablar también inglés. El entrenamiento de modelos de chino-inglés bilingüe con el método estándar está disponible únicamente en Norte de Europa y Centro y norte de EE. UU. El entrenamiento de Voz neuronal personalizada está disponible en Sur de Reino Unido y Este de EE. UU.

## <a name="speech-translation"></a>Traducción de voz

**Speech Translation** API admite varios idiomas para la traducción de voz a voz y de texto a voz. El idioma de origen debe incluirse siempre en la siguiente tabla de conversión de voz en texto. Los idiomas de destino admitidos dependen de si el destino de traducción es voz o texto. Puede traducir la voz entrante en más de [60 idiomas](https://www.microsoft.com/translator/business/languages/). Un subconjunto de estos idiomas está disponible para la [síntesis de voz](language-support.md#text-languages).

### <a name="text-languages"></a>Idiomas de texto

| Idioma de texto           | Código de lenguaje |
|:------------------------|:-------------:|
| Afrikáans               | `af`          |
| Árabe                  | `ar`          |
| Bengalí                  | `bn`          |
| Bosnio (latino)         | `bs`          |
| Búlgaro               | `bg`          |
| Cantonés (tradicional) | `yue`         |
| Catalán                 | `ca`          |
| Chino simplificado      | `zh-Hans`     |
| Chino tradicional     | `zh-Hant`     |
| Croata                | `hr`          |
| Checo                   | `cs`          |
| Danés                  | `da`          |
| Neerlandés                   | `nl`          |
| Inglés                 | `en`          |
| Estonio                | `et`          |
| Fiyiano                  | `fj`          |
| Filipino                | `fil`         |
| Finés                 | `fi`          |
| Francés                  | `fr`          |
| Alemán                  | `de`          |
| Griego                   | `el`          |
| Gujarati                | `gu`          |
| Criollo haitiano          | `ht`          |
| Hebreo                  | `he`          |
| Hindi                   | `hi`          |
| Hmong Daw               | `mww`         |
| Húngaro               | `hu`          |
| Indonesio              | `id`          |
| Irlandés                   | `ga`          |
| Italiano                 | `it`          |
| Japonés                | `ja`          |
| Canarés                 | `kn`          |
| Kiswahili               | `sw`          |
| Klingon                 | `tlh-Latn`    |
| Klingon (plqaD)         | `tlh-Piqd`    |
| Coreano                  | `ko`          |
| Letón                 | `lv`          |
| Lituano              | `lt`          |
| Malgache                | `mg`          |
| Malayo                   | `ms`          |
| Malayalam               | `ml`          |
| Maltés                 | `mt`          |
| Maori                   | `mi`          |
| Maratí                 | `mr`          |
| Noruego               | `nb`          |
| Persa                 | `fa`          |
| Polaco                  | `pl`          |
| Portugués (Brasil)     | `pt-br`       |
| Portugués (Portugal)   | `pt-pt`       |
| Punjabi                 | `pa`          |
| Otomí Querétaro         | `otq`         |
| Rumano                | `ro`          |
| Ruso                 | `ru`          |
| Samoano                  | `sm`          |
| Serbio (cirílico)      | `sr-Cyrl`     |
| Serbio (latino)         | `sr-Latn`     |
| Eslovaco                  | `sk`          |
| Esloveno               | `sl`          |
| Español                 | `es`          |
| Sueco                 | `sv`          |
| Tahitiano                | `ty`          |
| Tamil                   | `ta`          |
| Telugu                  | `te`          |
| Tailandés                    | `th`          |
| Tongano                  | `to`          |
| Turco                 | `tr`          |
| Ucraniano               | `uk`          |
| Urdu                    | `ur`          |
| Vietnamita              | `vi`          |
| Galés                   | `cy`          |
| Maya Yucateco            | `yua`         |

## <a name="speaker-recognition"></a>Speaker Recognition

Consulte la tabla siguiente para ver los idiomas compatibles con las distintas API de Speaker Recognition. Consulte la [introducción](speaker-recognition-overview.md) para obtener información adicional sobre Speaker Recognition.

| Idioma | Configuración regional (BCP-47) | Comprobación dependiente del texto | Comprobación independiente del texto | Identificación independiente del texto |
|----|----|----|----|----|
|Inglés (EE.UU.)  |  es-ES  |  sí  |  sí  |  sí |
|Chino (mandarín, simplificado) | zh-CN     |     N/D |     sí |     sí|
|Inglés (Australia)     | en-AU     | N/D     | sí     | sí|
|Inglés (Canadá)     | en-CA     | N/D |     sí |     sí|
|English (Reino Unido)     | en-GB     | N/D     | sí     | sí|
|Francés (Canadá)     | fr-CA     | N/D     | sí |     sí|
|Francés (Francia)     | fr-FR     | N/D     | sí     | sí|
|Alemán (Alemania)     | de-DE     | N/D     | sí     | sí|
|Italiano | it-IT     |     N/D     | sí |     sí|
|Japonés     | ja-JP | N/D     | sí     | sí|
|Portugués (Brasil) | pt-BR |     N/D |     sí |     sí|
|Español (México)     | es-MX     | N/D |     sí |     sí|
|Español (España)     | es-ES | N/D     | sí |     sí|

## <a name="custom-keyword-and-keyword-verification"></a>Palabra clave personalizada y verificación de palabras clave

En la tabla siguiente se detallan los idiomas admitidos para la palabra clave personalizada y la verificación de palabras clave.

| Idioma | Configuración regional (BCP-47) | Palabra clave personalizada | Verificación de palabras clave |
| -------- | --------------- | -------------- | -------------------- |
| Chino (mandarín, simplificado) | zh-CN | Sí | Sí |
| Spanish (Traditional Sort) - Spain | es-ES | Sí | Sí |
| Japonés (Japón) | ja-JP | No | Sí |
| Portugués (Brasil) | pt-BR | No | Sí |

## <a name="next-steps"></a>Pasos siguientes

* [Creación de una cuenta de Azure gratuita](https://azure.microsoft.com/free/cognitive-services/)
* [Vea cómo funciona el reconocimiento de voz en C#](./get-started-speech-to-text.md?pivots=programming-language-chsarp)
