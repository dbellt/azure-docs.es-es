---
title: ¿Qué es Azure Video Analyzer for Media (anteriormente Video Indexer)?
titleSuffix: Azure Media Services
description: En este artículo se proporciona información general sobre el servicio Azure Video Analyzer for Media (anteriormente Video Indexer).
services: media-services
author: Juliako
manager: femila
ms.topic: article
ms.date: 02/05/2021
ms.author: juliako
ms.openlocfilehash: acaed6bedcbfeb4ccafd61f0a907f1ccfb7e8bcb
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388682"
---
# <a name="what-is-video-analyzer-for-media"></a>¿Qué es Video Analyzer for Media?

[!INCLUDE [regulation](./includes/regulation.md)]

Azure Video Analyzer for Media (anteriormente Video Indexer) es una aplicación en la nube que forma parte de Azure Applied AI Services, basada en Azure Media Services y Azure Cognitive Services (como Face, Translator, Computer Vision y Voz). Permite extraer información de los vídeos mediante los modelos de vídeo y audio de Video Analyzer for Media.

Para empezar a extraer información con Video Analyzer for Media, tiene que crear una cuenta y cargar vídeos. Cuando se cargan vídeos en Video Analyzer for Media, este analiza los objetos visuales y el audio mediante la ejecución de diferentes modelos de IA. A medida que Video Analyzer for Media analiza el vídeo, los modelos de IA extraen la información.

Cuando crea una cuenta de Video Analyzer for Media y la conecta a Media Services, los archivos multimedia y de metadatos se almacenan en la cuenta de Azure Storage asociada a esa cuenta de Media Services. Para más información, consulte [Creación de una cuenta de Video Analyzer for Media conectada a Azure](connect-to-azure.md).

El siguiente diagrama es una ilustración, no una explicación técnica, de cómo funciona Video Analyzer for Media en el back-end.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/video-indexer-overview/model-chart.png" alt-text="Diagrama de flujo de Video Analyzer for Media":::

## <a name="compliance-privacy-and-security"></a>Cumplimiento, privacidad y seguridad

Como recordatorio importante, debe cumplir todas las leyes aplicables al uso de Video Analyzer for Media y no puede utilizar este servicio de Azure ni ningún otro de forma que infrinja los derechos de otras personas o que pueda ser perjudicial para ellas.

Antes de cargar un vídeo o una imagen en Video Analyzer for Media, debe tener todos los derechos adecuados para usar ese vídeo o imagen, incluidos (cuando lo exija la ley) todos los consentimientos necesarios de las personas (si las hay) que aparecen en ellos, para el uso, el procesamiento y el almacenamiento de sus datos en Video Analyzer for Media y Azure. Algunas jurisdicciones pueden imponer requisitos legales especiales sobre la recopilación, el procesamiento en línea y el almacenamiento de determinadas categorías de datos, como los datos biométricos. Antes de usar Video Analyzer for Media y Azure para el procesamiento y el almacenamiento de datos sujetos a requisitos legales especiales, debe garantizar el cumplimiento de tales requisitos legales que puedan aplicarse en su caso.

Para obtener información sobre el cumplimiento, la privacidad y la seguridad en Video Analyzer for Media, visite el [Centro de confianza](https://www.microsoft.com/TrustCenter/CloudServices/Azure/default.aspx) de Microsoft. En lo que respecta a las obligaciones de privacidad y a las prácticas de retención y tratamiento de datos de Microsoft, entre las que se incluye la forma de eliminar los datos, lea la [declaración de privacidad](https://privacy.microsoft.com/PrivacyStatement), los [términos de Online Services](https://www.microsoft.com/licensing/product-licensing/products?rtc=1) ("OST") y el [anexo de procesamiento de datos](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=67) ("DPA") de Microsoft. Con el uso de Video Analyzer for Media, acepta estar vinculado por el OST, el DPA y la declaración de privacidad.

## <a name="what-can-i-do-with-video-analyzer-for-media"></a>¿Qué puedo hacer con Video Analyzer for Media?

La información de Video Analyzer for Media se puede aplicar a muchos escenarios, entre ellos:

* *Búsqueda profunda*: use la información extraída del vídeo para mejorar la experiencia de búsqueda en una biblioteca de vídeos. Por ejemplo, la indexación de palabras habladas y de caras, permite la experiencia de búsqueda de momentos en un vídeo en los que una persona ha pronunciado ciertas palabras o cuando se han visto juntas a dos personas. La búsqueda basada en esta información de los vídeos se puede aplicar a agencias de noticias, instituciones educativas, emisoras, propietarios de contenido de entretenimiento, aplicaciones de línea de negocio de empresas y, en general, a cualquier sector que tenga una biblioteca de vídeos en la que los usuarios necesiten buscar.
* *Creación de contenido*: cree clips finales, momentos destacados, contenido de medios sociales o clips de noticias basados en la información que extrae Video Analyzer for Media de su contenido. Los fotogramas clave, los marcadores de escenas y las marcas de tiempo para las apariciones de personas y etiquetas hacen que el proceso de creación sea mucho más fluido y sencillo y le permiten ir a las partes del vídeo que necesita para el contenido que está creando.
* *Accesibilidad*: si desea que el contenido esté disponible para personas con diversidad funcional o que se distribuya en distintas regiones con distintos idiomas, puede usar la transcripción y traducción que proporciona Video Analyzer for Media en varios idiomas.
* *Monetización*: Video Analyzer for Media puede ayudar a aumentar el valor de los vídeos. Por ejemplo, los sectores que dependen de los ingresos por publicidad (medios de comunicación, redes sociales, etc.), pueden entregar anuncios relevantes con la información extraída como señales adicionales al servidor de anuncios.
* *Moderación de contenido*: use modelos de moderación de contenido textual y visual para proteger a los usuarios del contenido inadecuado y asegúrese de que el contenido que publica coincide con los valores de la organización. Puede bloquear automáticamente determinados vídeos o avisar a los usuarios sobre el contenido.
* *Recomendaciones*: La información de vídeo se puede usar para mejorar la interacción con los usuarios, resaltando los momentos de vídeo que les puedan resultar más pertinentes. Al etiquetar cada vídeo con metadatos adicionales, puede recomendar a los usuarios los vídeos más importantes y resaltar las partes del vídeo que se adaptan mejor a sus necesidades.

## <a name="features"></a>Características

En la siguiente lista se muestra la información que se puede recuperar de los vídeos mediante los modelos de audio y vídeo de Video Analyzer for Media:

### <a name="video-insights"></a>Información de los vídeos

* **Detección de caras**: detecta y agrupa las caras que aparecen en el vídeo.
* **Identificación de celebridades**: Video Analyzer for Media identifica automáticamente a más de un millón de famosos (como líderes mundiales, actores y actrices, atletas, investigadores y líderes empresariales y tecnológicos de todo el mundo). Los datos sobre estos famosos también pueden encontrarse en diversos sitios web (IMDB, Wikipedia, etc.).
* **Identificación facial basada en cuentas**: Video Analyzer for Media entrena un modelo para una cuenta específica. A continuación reconoce las caras en el vídeo según el modelo entrenado. Para más información, consulte [Personalización de un modelo de persona desde el sitio web de Video Analyzer for Media](customize-person-model-with-website.md) y [Personalización de un modelo de persona con la API de Video Analyzer for Media](customize-person-model-with-api.md).
* **Extracción de miniaturas de caras** ("mejor cara"): identifica automáticamente la mejor cara capturada en cada grupo de caras (según la calidad, el tamaño y la posición frontal) y la extrae como un recurso de imagen.
* **Reconocimiento de texto visual** (OCR): extrae el texto que se muestra visualmente en el vídeo.
* **Moderación de contenido visual**: Detecta los objetos visuales para adultos o subidos de tono.
* **Identificación de etiquetas**: identifica los objetos visuales y las acciones que se muestran.
* **Segmentación de escenas**: determina cuándo cambia una escena en el vídeo en función de las indicaciones visuales. Una escena representa un evento único y se compone de una serie de cortes consecutivos que están semánticamente relacionados.
* **Detección de cortes**: determina cuándo cambia un corte en el vídeo en función de las indicaciones visuales. Un corte es una serie de fotogramas tomados a partir de la misma cámara de imágenes en movimiento. Para más información, consulte [Scenes, shots, and keyframes](scenes-shots-keyframes.md) (Escenas, cortes y fotogramas clave).
* **Detección de fotogramas negros**: identifica los fotogramas negros presentados en el vídeo.
* **Extracción de fotogramas clave**: detecta los fotogramas clave estables en un vídeo.
* **Créditos de rodaje**: identifica el principio y el final de los créditos al final de las series de televisión y las películas.
* **Detección de personajes animados** (versión preliminar): detección, agrupación y reconocimiento de personajes en contenido animado a través de la integración con la [visión personalizada de Cognitive Services](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/). Para más información, consulte [Detección de personajes animados](animated-characters-recognition.md).
* **Detección del tipo de toma editorial**: etiquetado de capturas basado en su tipo (como plano general, plano medio, primer plano, primerísimo primer plano, dos capturas, varias personas, exterior e interior, etc.). Para más información, consulte [Detección del tipo de toma editorial](scenes-shots-keyframes.md#editorial-shot-type-detection).
* **Seguimiento de personas observadas**: detecta personas observadas en vídeos y proporciona información, como la ubicación de la persona en el fotograma de vídeo (mediante rectángulos delimitadores), la marca de tiempo exacta (inicio y fin) y la confianza cuando aparece una persona. Para más información, vea [Seguimiento de las personas observadas en un vídeo](observed-people-tracing.md).

### <a name="audio-insights"></a>Información de audio

* **Transcripción de audio**: convierte la voz en texto en 12 idiomas y permite extensiones. Los idiomas admitidos son los siguientes: alemán, árabe, chino (mandarín), coreano, español, francés, hindi, inglés, italiano, japonés, portugués y ruso.
* **Detección de idioma automática**: identifica automáticamente el idioma hablado dominante. Los idiomas admitidos son los siguientes: alemán, chino (mandarín), español, francés, inglés, italiano, japonés, portugués y ruso. Si el idioma hablado no se puede identificar con confianza, Video Analyzer for Media supone que es inglés. Para obtener más información, vea [Modelo de identificación de idiomas](language-identification-model.md).
* **Identificación y transcripción de voz en varios idiomas**: identifica automáticamente el idioma hablado en diferentes segmentos del audio. Envía cada segmento del archivo multimedia que se va a transcribir y luego combina la transcripción en una unificada. Para más información, consulte [Identificación y transcripción automáticas del contenido de varios idiomas](multi-language-identification-transcription.md).
* **Subtítulos (CC)** : crea subtítulos en tres formatos: VTT, TTML y SRT.
* **Procesamiento de dos canales**: detecta, separa la transcripción y se combina en una única escala de tiempo de forma automática.
* **Reducción de ruido**: borra el audio telefónico o las grabaciones ruidosas (basado en los filtros de Skype).
* **Personalización de la transcripción** (CRIS): entrena modelos personalizados de conversión de voz en texto para crear transcripciones específicas del sector. Para más información, consulte [Personalización de un modelo de idioma desde el sitio web de Video Analyzer for Media](customize-language-model-with-website.md) y [Personalización de un modelo de idioma con las API de Video Analyzer for Media](customize-language-model-with-api.md).
* **Enumeración de altavoz**: asigna y comprende las palabras que transmitió cada altavoz y cuándo. Se pueden detectar dieciséis altavoces en un solo archivo de audio.
* **Estadísticas de altavoz**: Proporciona las estadísticas de las relaciones de voz de los altavoces.
* **Moderación de contenido textual**: detecta texto explícito en la transcripción de audio.
* **Efectos de audio** (versión preliminar): detecta los siguientes efectos de audio en los segmentos del contenido que no son de voz: disparo de un arma, rotura de un cristal, alarma, sirena, explosión, ladridos, gritos, risas, reacciones de una multitud (júbilo, aplausos y abucheos) y silencio. Nota: El conjunto completo de eventos solo está disponible cuando se elige "Advanced Audio Analysis" (Análisis de audio avanzado) en Upload preset (Cargar valores predeterminados); de lo contrario, solo estarán disponibles los eventos de silencio y reacciones de una multitud.
* **Detección de emociones**: identifica emociones en función de la voz (lo que se dice) y el tono (cómo se dice). La emoción podría ser felicidad, tristeza, ira o miedo.
* **Traducción**: crea traducciones de la transcripción de audio en 54 idiomas diferentes.
* **Detección de efectos de audio** (versión preliminar): detecta varios eventos acústicos y los clasifica en distintas categorías acústicas (como disparo de un arma, gritos, reacciones de una multitud, etc.). Los eventos acústicos detectados se encuentran en el archivo de subtítulos. El archivo se puede descargar desde el portal de Video Analyzer for Media. Para más información, consulte [Detección de efectos de audio](audio-effects-detection.md).

### <a name="audio-and-video-insights-multi-channels"></a>Información de audio y vídeo (varios canales)

Al indexar por un canal, hay resultados parciales para esos modelos.

* **Extracción de palabras clave**: extrae palabras clave a partir de voz y texto visual.
* **Extracción de entidades con nombre**: extrae marcas, ubicaciones y personas del lenguaje hablado y del texto visual mediante el procesamiento de lenguaje natural (NLP).
* **Inferencia de tema**: saca conclusiones de los temas principales a partir de las transcripciones. Se incluye la taxonomía IPTC de segundo nivel.
* **Artefactos**: extrae una amplia variedad de artefactos de "detalles de siguiente nivel" para cada uno de los modelos.
* **Análisis de opiniones**: identifica opiniones positivas, negativas y neutras a partir de voz y texto visual.

## <a name="how-can-i-get-started-with-video-analyzer-for-media"></a>¿Cómo empiezo a trabajar con Video Analyzer for Media?

Puede acceder a las funcionalidades de Video Analyzer for Media de tres maneras:

* Portal de Video Analyzer for Media: una solución fácil de usar que le permite evaluar el producto, administrar la cuenta y personalizar los modelos.

    Para más información sobre el portal, consulte [Introducción al sitio web de Video Analyzer for Media](video-indexer-get-started.md).  

* Integración de API: todas las funcionalidades de Video Analyzer for Media están disponibles mediante una API REST que le permite integrar la solución en sus aplicaciones e infraestructura.

    Para empezar a trabajar como desarrollador, consulte  [Uso de la API REST de Video Analyzer for Media](video-indexer-use-apis.md).

* Widget insertable: permite insertar las experiencias de información, reproductor y editor de Video Analyzer for Media en la aplicación.

    Para más información, consulte  [Inserción de widgets de Video Indexer en las aplicaciones](video-indexer-embed-widgets.md).

Si usa el sitio web, la información se agrega como metadatos y es visible en el portal. Si usa las API, la información está disponible como archivo JSON.

## <a name="supported-browsers"></a>Exploradores compatibles

En la lista siguiente se muestran los exploradores admitidos que puede usar para el sitio web de Video Analyzer for Media y para las aplicaciones que insertan los widgets. La lista también muestra la versión mínima admitida del explorador:

- Edge, versión 16
- Firefox, versión 54
- Chrome, versión 58
- Safari, versión 11
- Opera, versión 44
- Opera Mobile, versión 59
- Explorador de Android, versión 81
- Explorador de Samsung, versión 7
- Chrome para Android, versión 87
- Firefox para Android, versión 83

## <a name="next-steps"></a>Pasos siguientes

Ya está listo para empezar a trabajar con Video Analyzer for Media. Para más información, consulte los siguientes artículos.

- [Introducción al sitio web de Video Analyzer for Media](video-indexer-get-started.md).
- [Procesamiento del contenido con la API REST de Video Analyzer for Media](video-indexer-use-apis.md).
- [Inserción de widgets visuales en la aplicación](video-indexer-embed-widgets.md).
