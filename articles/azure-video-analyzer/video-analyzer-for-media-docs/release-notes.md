---
title: Notas de la versión de Azure Video Analyzer for Media (anteriormente, Video Indexer) | Microsoft Docs
description: Para mantenerse al día con los últimos desarrollos, en este artículo se proporcionan las actualizaciones más reciente en Azure Video Analyzer for Media (anteriormente, Video Indexer).
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.workload: na
ms.topic: article
ms.custom: references_regions
ms.date: 05/06/2021
ms.author: juliako
ms.openlocfilehash: d67d8dba31a453f4a42982519d619f294d47e04f
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111813826"
---
# <a name="video-analyzer-for-media-release-notes"></a>Notas de la versión de Video Analyzer for Media

>Reciba notificaciones para volver a visitar esta página y obtener actualizaciones; para ello, copie y pegue esta URL (`https://docs.microsoft.com/api/search/rss?search=%22Azure+Media+Services+Video+Indexer+release+notes%22&locale=en-us`) en el lector de fuentes RSS.

Para mantenerse al día con los desarrollos más recientes de Azure Video Analyzer for Media (anteriormente, Video Indexer), en este artículo encontrará información sobre lo siguiente:

* Versiones más recientes
* Problemas conocidos
* Corrección de errores
* Funciones obsoletas

## <a name="may-2021"></a>Mayo de 2021

### <a name="new-source-languages-support-for-speech-to-text-stt-translation-and-search"></a>Compatibilidad con nuevos idiomas de origen para la conversión de voz a texto (STT), la traducción y la búsqueda

Video Analyzer for Media ahora admite STT, traducción, y búsquedas en chino (cantonés) ("zh-HK"), neerlandés (Países Bajos) ("Nl-NL"), checo ("Cs-CZ"), polaco ("Pl-PL"), sueco (Suecia) ("Sv-SE"), noruego ("nb-NO"), finés ("fi-FI"), francés canadiense ("fr-CA"), tailandés ("th-TH"), árabe: (Emiratos Árabes Unidos) ("ar-AE", "ar-EG"), (Iraq) (''ar-IQ"), (Jordania) ("ar-JO"), (Kuwait) ("ar-KW"), (Líbano) ("ar-LB"), (Omán) ("ar-OM"), (Qatar) ("ar-QA"), (Autoridad Palestina) ("ar-PS"), (Siria) ("ar-SY"), y turco ("tr-TR"). 

Estos idiomas están disponibles en la API y en el sitio web de Video Analyzer for Media. Seleccione el idioma del cuadro combinado en **Idioma de origen del vídeo**.

### <a name="new-theme-for-azure-video-analyzer-for-media"></a>Nuevo tema de Azure Video Analyzer for Media

Hay disponible un nuevo tema: "Azure" junto con los temas "light" y "dark". Para seleccionar un tema, haga clic en el icono de engranaje de la esquina superior derecha del sitio web y busque temas en **Configuración de usuario**.
 
### <a name="new-open-source-code-you-can-leverage"></a>Puede utilizar nuevo código abierto 

Hay tres nuevos proyectos de Git-Hub disponibles en nuestro [repositorio de GitHub](https://github.com/Azure-Samples/media-services-video-indexer):

* Código que le ayuda a aprovechar la [personalización del widget](https://github.com/Azure-Samples/media-services-video-indexer/tree/master/Embedding%20widgets) recién agregado.
* Solución que le ayudará a agregar [búsqueda personalizada](https://github.com/Azure-Samples/media-services-video-indexer/tree/master/VideoSearchWithAutoMLVision) a las bibliotecas de vídeos.
* Solución que le ayudará a agregar [desduplicación](https://github.com/Azure-Samples/media-services-video-indexer/commit/6b828f598f5bf61ce1b6dbcbea9e8b87ba11c7b1) a las bibliotecas de vídeos.
 
### <a name="new-option-to-toggle-bounding-boxes-for-observed-people-on-the-player"></a>Nueva opción para alternar los rectángulos delimitadores (para personas observadas) en el reproductor  

Al indexar un vídeo a través de nuestra configuración de vídeo avanzada, puede ver nuestras nuevas funcionalidades de personas observadas.

Si se detectan personas en el archivo multimedia, puede habilitar un rectángulo delimitador en la persona detectada mediante el reproductor multimedia.

## <a name="april-2021"></a>Abril de 2021

El servicio Video Indexer ha cambiado de nombre a Azure Video Analyzer for Media.

### <a name="observed-people-tracing-preview"></a>Seguimiento de personas observadas (versión preliminar)  

Azure Video Analyzer for Media ahora detecta a las personas observadas en los vídeos y proporciona información, como la ubicación de la persona en el fotograma de vídeo y la marca de tiempo exacta (inicio, fin) en la que aparece una persona. La API devuelve las coordenadas del cuadro de límite (en píxeles) para cada instancia de persona detectada, incluido el grado de confianza. 

Por ejemplo, si un vídeo contiene una persona, la operación de detección mostrará las apariciones de esa persona, junto con sus coordenadas en los fotogramas de vídeo. Puede usar esta funcionalidad para establecer la ruta de una persona en un vídeo. También le permite saber si hay varias instancias de la misma persona en un vídeo. 

La característica de seguimiento de personas observadas recién agregada está disponible al indexar el archivo si elige el valor preestablecido **Opción avanzada** -> **Advanced video** (Vídeo avanzado) o **Advanced video + audio** (Vídeo y audio avanzado) en Indexación de audio y vídeo. Los valores preestablecidos de indexación estándar y básica no incluirán este nuevo modelo avanzado. 

Cuando decida ver información del vídeo en el sitio web de Video Analyzer for Media, el seguimiento de personas observadas se mostrará en la página con todas las miniaturas de las personas detectadas. Puede elegir una miniatura de una persona y ver dónde aparece la persona en el reproductor de vídeo.  

La característica también está disponible en el archivo JSON generado por Video Analyzer for Media. Para más información, vea [Seguimiento de las personas observadas en un vídeo](observed-people-tracing.md).

### <a name="detected-acoustic-events-with-audio-effects-detection-preview"></a>Eventos acústicos detectados con **Detección de efectos de audio** (versión preliminar)

Ahora puede ver los eventos acústicos detectados en el archivo de subtítulos. El archivo se puede descargar desde el portal de Video Analyzer for Media, y está disponible como artefacto en la API GetArtifact.

El componente **Detección de efectos de audio** (versión preliminar) detecta varios eventos acústicos y los clasifica en distintas categorías acústicas (como disparo de un arma, gritos, reacciones de una multitud, etc.). Para más información, consulte [Detección de efectos de audio](audio-effects-detection.md).

### <a name="improved-upload-experience-in-the-portal"></a>Experiencia de carga mejorada en el portal

Video Analyzer for Media tiene una nueva experiencia de carga en el portal:

* Nuevo portal para desarrolladores disponible en Fairfax

El nuevo [portal para desarrolladores](https://api-portal.videoindexer.ai) de Video Analyzer for Media ahora también está disponible en la nube de administración pública.

## <a name="march-2021"></a>Marzo de 2021

### <a name="audio-analysis"></a>Análisis de audio 

Ahora dispone de análisis de audio en un nuevo conjunto de características de audio con un precio diferente. El nuevo valor preestablecido modo básico de análisis **Basic Audio** (Audio básico) ofrece una opción de bajo costo que solo extrae transcripciones de voz, traducción y formato de los títulos y subtítulos de salida. Con el valor preestablecido **Basic Audio** (Audio básico) aparecerán dos medidores independientes en la factura, incluida una línea para la transcripción y una línea independiente para el formato de títulos y subtítulos. Para más información sobre los precios, consulte la página [Precios de Media Services](https://azure.microsoft.com/pricing/details/media-services/).

El conjunto recién agregado está disponible al indexar o volver a indexar el archivo; para ello, elija el valor preestablecido **Opción avanzada** -> **Basic Audio** (Audio básico) en el cuadro desplegable **Indexación de audio y vídeo**.

### <a name="new-developer-portal"></a>Nuevo portal para desarrolladores 

Video Analyzer for Media tiene un nuevo [portal para desarrolladores](https://api-portal.videoindexer.ai/). Pruebe las nuevas API de Video Analyzer for Media y encuentre todos los recursos importantes en un solo lugar: el [repositorio de GitHub](https://github.com/Azure-Samples/media-services-video-indexer), [Stack Overflow](https://stackoverflow.com/questions/tagged/video-indexer), la [comunidad tecnológica de Video Analyzer for Media](https://techcommunity.microsoft.com/t5/azure-media-services/bg-p/AzureMediaServices/label-name/Video%20Indexer) con entradas de blog relacionadas, las [preguntas frecuentes de Video Analyzer for Media](faq.md), [User Voice](https://feedback.azure.com/forums/932041-cognitive-services?category_id=399016) para proporcionar sus comentarios y sugerir características, y el [vínculo "CodePen"](https://codepen.io/videoindexer) con ejemplos de código de widgets. 
 
### <a name="advanced-customization-capabilities-for-insight-widget"></a>Funcionalidades de personalización avanzadas para el widget de conclusiones 

Ahora hay un SDK disponible para insertar el widget de información de Video Analyzer for Media en su propio servicio, y personalizar su estilo y sus datos. El SDK admite el widget de conclusiones estándar de Video Analyzer for Media y un widget de información totalmente personalizable. El ejemplo de código está disponible en el [repositorio de GitHub de Video Analyzer for Media](https://github.com/Azure-Samples/media-services-video-indexer/tree/master/Embedding%20widgets/widget-customization). Con estas funcionalidades de personalización avanzadas, el desarrollador de soluciones puede aplicar un estilo personalizado y aportar los propios datos de IA del cliente y presentarlos en el widget de información (con o sin la información de Video Analyzer for Media ). 

### <a name="video-analyzer-for-media-deployed-in-the-us-north-central--us-west-and-canada-central"></a>Implementación de Video Analyzer for Media en las regiones Centro y norte de EE. UU., Oeste de EE. UU. y Centro de Canadá 

Ahora puede crear una cuenta de pago de Video Analyzer for Media en las regiones Centro y norte de EE. UU., Oeste de EE. UU. y Centro de Canadá.
 
### <a name="new-source-languages-support-for-speech-to-text-stt-translation-and-search"></a>Compatibilidad con nuevos idiomas de origen para la conversión de voz a texto (STT), la traducción y la búsqueda 

Video Analyzer for Media admiten ahora STT, traducción y búsqueda en danés ("da-DK"), noruego ("NB-NO"), sueco ("sv-SE"), finés ("fi-FI"), francés canadiense ("fr-CA"), tailandés ("th-TH"), árabe ("ar-BH", "ar-EG", "ar-IQ", "ar-JO", "ar-KW", "ar-LB", "ar-OM", "ar-QA", "ar-S" y "ar-SY") y turco ("tr-TR"). Estos idiomas están disponibles en la API y en el sitio web de Video Analyzer for Media. 
 
### <a name="search-by-topic-in-video-analyzer-for-media-website"></a>Búsqueda por tema en el sitio web de Video Analyzer for Media 

Ahora puede usar la característica de búsqueda, en la parte superior de la página del [sitio web de Video Analyzer for Media](https://www.videoindexer.ai/account/login), para buscar vídeos con temas concretos. 

## <a name="february-2021"></a>Febrero de 2021

### <a name="multiple-account-owners"></a>Varios propietarios de cuentas 

El rol de propietario de la cuenta se agregó al Video Analyzer for Media. Puede agregar, cambiar y quitar usuarios, así como cambiar su rol. Para más información sobre cómo compartir una cuenta, consulte [Invitar a usuarios](invite-users.md).

### <a name="audio-event-detection-public-preview"></a>Detección de eventos de audio (versión preliminar pública)

> [!NOTE]
> Esta característica solo está disponible en cuentas de evaluación gratuita. 

Ahora Video Analyzer for Media detecta los siguientes efectos de audio en los segmentos del contenido que no son de voz: disparo de un arma, rotura de un cristal, alarma, sirena, explosión, ladridos, gritos, risas, reacciones de una multitud (júbilo, aplausos y abucheos) y silencio. 

La característica de efectos de audio recién agregada está disponible al indexar el archivo si elige el valor preestablecido **Opción avanzada** -> **Advanced audio** (Audio avanzado), en Indexación de audio y vídeo. La indexación estándar solo incluirá el **silencio** y la **reacción de una multitud**. 

El tipo de evento de **aplausos** que se incluyó en el modelo de efectos de audio anterior, ahora se extrae como parte del tipo de evento de **reacción de una multitud**.

Cuando elige ver la **información** del vídeo en el sitio web de [Video Analyzer for Media](https://www.videoindexer.ai/), en la página se muestran los efectos de audio.

:::image type="content" source="./media/release-notes/audio-detection.png" alt-text="Detección de eventos de audio":::

### <a name="named-entities-enhancement"></a>Mejoras en entidades con nombre  

La lista que se extrae de personas y ubicaciones se ha ampliado y actualizado en general. 

Además, el modelo ahora incluye personas y ubicaciones del contexto que no son famosas, como "Samuel" o "Casa" en el vídeo. 

## <a name="january-2021"></a>Enero de 2021

### <a name="video-analyzer-for-media-is-deployed-on-us-government-cloud"></a>Video Analyzer for Media está implementado en la nube del Gobierno de EE. UU. 

Ahora puede crear una cuenta de pago de Video Analyzer for Media en la nube US Government en las regiones Virginia y Arizona. La oferta de evaluación gratuita de Video Analyzer for Media no está disponible en la región mencionada. Para obtener más información, vaya a la documentación de Video Analyzer for Media. 

### <a name="video-analyzer-for-media-deployed-in-the-india-central-region"></a>Video Analyzer for Media implementado en la región Centro de la India 

Ya puede crear una cuenta de pago de Video Analyzer for Media en la región Centro de la India. 

### <a name="new-dark-mode-for-the-video-analyzer-for-media-website-experience"></a>Nuevo modo oscuro para la experiencia del sitio web de Video Analyzer for Media

La experiencia del sitio web de Video Analyzer for Media ahora está disponible en modo oscuro. Para habilitar el modo oscuro, abra el panel de configuración y active la opción **Modo oscuro**. 

:::image type="content" source="./media/release-notes/dark-mode.png" alt-text="Configuración del modo oscuro":::

## <a name="december-2020"></a>Diciembre de 2020

### <a name="video-analyzer-for-media-deployed-in-the-switzerland-west-and-switzerland-north"></a>Video Analyzer for Media implementado en Oeste de Suiza y Norte de Suiza

Ahora puede crear una cuenta de pago de Video Analyzer for Media en las regiones Oeste de Suiza y Norte de Suiza.

## <a name="october-2020"></a>Octubre de 2020

### <a name="animated-character-identification-improvements"></a>Mejoras en la identificación de personajes animados  

Video Analyzer for Media admite la detección, la agrupación y el reconocimiento de personajes en contenido animado a través de la integración con Custom Vision de Cognitive Services. Hemos agregado una mejora importante a este algoritmo de IA en la detección y el reconocimiento de personajes, por lo que la precisión de la información y los personajes identificados han mejorado significativamente.

### <a name="planned-video-analyzer-for-media-website-authenticatication-changes"></a>Cambios de autenticación planeados del sitio web de Video Analyzer for Media

A partir del 1 de marzo de 2021, ya no podrá registrarse ni iniciar sesión en el [portal para desarrolladores](video-indexer-use-apis.md) del [sitio web de Video Analyzer for Media](https://www.videoindexer.ai/) mediante Facebook ni LinkedIn.

Podrá registrarse e iniciar sesión con uno de estos proveedores: Azure AD, Microsoft y Google.

> [!NOTE]
> Después del 1 de marzo de 2021, no podrá acceder a las cuentas de Video Analyzer for Media conectadas a LinkedIn y Facebook. 
> 
> Para seguir teniendo acceso, debe [invitar](invite-users.md) a un correo electrónico de Azure AD, Microsoft o Google de su propiedad a la cuenta de Video Analyzer for Media. Puede agregar un propietario adicional de proveedores admitidos, como se describe en [invite](invite-users.md). <br/>
> Como alternativa, puede crear una cuenta de pago y migrar los datos.

## <a name="august-2020"></a>Agosto de 2020

### <a name="mobile-design-for-the-video-analyzer-for-media-website"></a>Diseño móvil para el sitio web de Video Analyzer for Media

La experiencia del sitio web de Video Analyzer for Media ahora es compatible con dispositivos móviles. La experiencia del usuario tiene capacidad de respuesta para adaptarse al tamaño de la pantalla del móvil (salvo las interfaces de usuario de personalización). 

### <a name="accessibility-improvements-and-bug-fixes"></a>Mejoras en la accesibilidad y correcciones de errores 

Como parte de WCAG (directrices de accesibilidad de contenido web), las experiencias del sitio web de Video Analyzer for Media se alinean con la categoría C, como parte de los estándares de accesibilidad de Microsoft. Se han solucionado varios errores y realizado mejoras relacionados con la navegación mediante teclado, el acceso mediante programación y el lector de pantalla. 

## <a name="july-2020"></a>Julio de 2020

### <a name="ga-for-multi-language-identification"></a>Disponibilidad general para la identificación en varios idiomas

La identificación de varios idiomas pasa de la versión preliminar a la fase de disponibilidad general y está lista para su uso productivo.

La transición de versión preliminar a disponibilidad general no tiene ninguna repercusión en el precio.

### <a name="video-analyzer-for-media-website-improvements"></a>Mejoras en el sitio web de Video Analyzer for Media

#### <a name="adjustments-in-the-video-gallery"></a>Ajustes en la galería de vídeos

Se ha agregado una nueva barra de búsqueda para la búsqueda de información detallada con funcionalidades de filtrado adicionales. También se han mejorado los resultados de la búsqueda.

Nueva vista de lista con capacidad para ordenar y administrar el archivo de vídeo con varios archivos.

#### <a name="new-panel-for-easy-selection-and-configuration"></a>Nuevo panel para facilitar la selección y configuración

Se ha agregado un panel lateral para facilitar la selección y la configuración de usuarios, lo que permite crear y compartir cuentas de forma sencilla y rápida, así como establecer la configuración.

El panel lateral también se usa para las preferencias de usuario y la ayuda.

## <a name="june-2020"></a>Junio de 2020

### <a name="search-by-topics"></a>Buscar por temas

Ahora puede usar la API de búsqueda para buscar vídeos con temas específicos (solo API).

Los temas se agregan como parte del `textScope` (parámetro opcional). Consulte [API](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Search-Videos) para obtener más información.  

### <a name="labels-enhancement"></a>Renovación de etiquetas

El etiquetador se ha actualizado y ahora incluye más etiquetas visuales que se pueden identificar.

## <a name="may-2020"></a>Mayo de 2020

### <a name="video-analyzer-for-media-deployed-in-the-east-us"></a>Video Analyzer for Media implementado en Este de EE. UU.

Ya puede crear una cuenta de pago de Video Analyzer for Media en la región Este de EE. UU.
 
### <a name="video-analyzer-for-media-url"></a>Dirección URL de Video Analyzer for Media

Los puntos de conexión regionales de Video Analyzer for Media se han unificado para comenzar solo con www. No se requiere ningún elemento de acción.

A partir de ahora, llegará a www.videoindexer.ai tanto para insertar widgets como para iniciar sesión en las aplicaciones web de Video Analyzer for Media.

Además, wus.videoindexer.ai se redirigirá a www. Puede encontrar más información en [Inserción de widgets de Video Analyzer for Media en las aplicaciones](video-indexer-embed-widgets.md).

## <a name="april-2020"></a>Abril de 2020

### <a name="new-widget-parameters-capabilities"></a>Nuevas capacidades de parámetros de widget

El widget **Insights** incluye los nuevos parámetros `language` y `control`.

El widget **Player** tiene el nuevo parámetro `locale`. Los parámetros `locale` y `language` controlan el idioma del reproductor.

Para más información, consulte la sección de [tipos de widgets](video-indexer-embed-widgets.md#widget-types). 

### <a name="new-player-skin"></a>Nueva máscara del reproductor

Nueva máscara del reproductor iniciada con el diseño actualizado.

### <a name="prepare-for-upcoming-changes"></a>Preparación para los próximos cambios

* En la actualidad, las siguientes API devuelven un objeto de cuenta:

    * [Create-Paid-Account](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Create-Paid-Account)
    * [Get-Account](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Account)
    * [Get-Accounts-Authorization](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Accounts-Authorization)
    * [Get-Accounts-With-Token](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Accounts-With-Token)
 
    El objeto de cuenta tiene un campo `Url` que apunta a la ubicación del [sitio web de Video Analyzer for Media](https://www.videoindexer.ai/).
En el caso de las cuentas de pago, el campo `Url` apunta actualmente a una dirección URL interna en lugar de al sitio web público.
En las próximas semanas lo cambiaremos y volverá a la dirección URL del [sitio web de Video Analyzer for Media](https://www.videoindexer.ai/) en todas las cuentas (de prueba y de pago).

    No use las direcciones URL internas; debe usar las [API públicas de Video Analyzer for Media](https://api-portal.videoindexer.ai/).
* Si va a insertar direcciones URL de Video Analyzer for Media en las aplicaciones y no apuntan al [sitio web de Video Analyzer for Media](https://www.videoindexer.ai/) ni al punto de conexión de la API de Video Analyzer for Media (`https://api.videoindexer.ai`), sino que lo hacen a un punto de conexión regional (por ejemplo, `https://wus2.videoindexer.ai`), vuelva a generar las direcciones URL.

   Para ello, haga lo siguiente:

    * Reemplace la dirección URL por otra que apunte a las API de widget de Video Analyzer for Media (por ejemplo, el [widget de información](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Video-Insights-Widget)).
    * Uso del sitio web de Video Analyzer for Media para generar una nueva dirección URL insertada:
         
         Presione **Reproducir** para ir a la página del vídeo, haga clic en el botón **&lt;/&gt; Embed** (Insertar) y copie la dirección URL en la aplicación:
   
    Las direcciones URL regionales no se admiten y se bloquearán en las próximas semanas.

## <a name="january-2020"></a>Enero de 2020
 
### <a name="custom-language-support-for-additional-languages"></a>Compatibilidad con idiomas personalizados adicionales

Video Analyzer for Media admite ahora modelos de lenguaje personalizados para `ar-SY`, `en-UK` y `en-AU` (solo API).
 
### <a name="delete-account-timeframe-action-update"></a>Actualización del período de tiempo de la acción de eliminación de una cuenta

La acción de eliminación de una cuenta ahora elimina la cuenta en un plazo de 90 días en lugar de 48 horas.
 
### <a name="new-video-analyzer-for-media-github-repository"></a>Nuevo repositorio de GitHub de Video Analyzer for Media

Ahora hay disponible un nuevo repositorio de GitHub para Video Analyzer for Media con distintos proyectos, guías de introducción y ejemplos de código: https://github.com/Azure-Samples/media-services-video-indexer
 
### <a name="swagger-update"></a>Actualización de Swagger

Video Analyzer for Media unificó las **autenticaciones** y **operaciones** en una única [especificación de OpenAPI para Video Analyzer for Media (Swagger)](https://api-portal.videoindexer.ai/api-details#api=Operations&operation). Los desarrolladores pueden encontrar las API en el [portal para desarrolladores de Video Analyzer for Media](https://api-portal.videoindexer.ai/).

## <a name="december-2019"></a>Diciembre de 2019

### <a name="update-transcript-with-the-new-api"></a>Actualización de transcripción con la nueva API

Actualice una sección específica de la transcripción mediante la API [Update-Video-Index](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Video-Index).

### <a name="fix-account-configuration-from-the-video-analyzer-for-media-portal"></a>Corrección de la configuración de cuentas desde el portal de Video Analyzer for Media

Ahora puede actualizar la configuración de conexión de Media Services y usar la autoayuda para resolver problemas como estos: 

* Recurso incorrecto de Azure Media Services
* Cambios de contraseña
* Los recursos de Media Services se han movido entre suscripciones  

Para corregir la configuración de la cuenta, en el portal de Video Analyzer for Media, vaya a Configuración > pestaña Cuenta (como propietario).

### <a name="configure-the-custom-vision-account"></a>Configuración de la cuenta de Custom Vision

Configure la cuenta de Custom Vision en cuentas de pago mediante el portal de Video Analyzer for Media (anteriormente, esto solo lo admitía la API). Para ello, inicie sesión en el portal de Video Analyzer for Media, elija Personalización de modelos > Personajes animados > Configurar. 

### <a name="scenes-shots-and-keyframes--now-in-one-insight-pane"></a>Escenas, tomas y fotogramas clave: ahora en un panel de información detallada

Ahora las escenas, las tomas y los fotogramas clave se combinan en un panel de información detallada para facilitar el consumo y la navegación. Al seleccionar la escena deseada, puede ver qué tomas y fotogramas clave contiene. 

### <a name="notification-about-a-long-video-name"></a>Notificación sobre un nombre de vídeo largo

Cuando el nombre de un vídeo tiene más de 80 caracteres, Video Analyzer for Media muestra un error descriptivo durante la carga.

### <a name="streaming-endpoint-is-disabled-notification"></a>Notificación de punto de conexión de streaming deshabilitado

Cuando el punto de conexión de streaming esté deshabilitado, Video Analyzer for Media mostrará un error descriptivo en la página del reproductor.

### <a name="error-handling-improvement"></a>Mejora en el control de errores

Ahora, si un vídeo se indexa de forma activa, se devolverá el código de estado 409 desde las API [Re-Index Video](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Re-Index-Video) (Volver a indexar el vídeo) y [Update Video Index](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Video-Index) (Actualizar índice de vídeo), para evitar que se invaliden por accidente los cambios de volver a indexar.

## <a name="november-2019"></a>Noviembre de 2019
 
* Compatibilidad con modelos de lenguaje personalizado en coreano

    Video Analyzer for Media ahora admite modelos de lenguaje personalizados en coreano (`ko-KR`) tanto en la API como en el portal. 
* Nuevos idiomas compatibles con Conversión de voz en texto (STT)

    Las API de Video Analyzer for Media ahora admiten STT en árabe levantino (ar-SY), dialecto inglés del Reino Unido (en-GB) y dialecto australiano inglés (en-AU).
    
    En el caso de la carga de vídeo, reemplazamos zh-HANS por zh-CN, se admiten ambos, pero zh-CN es el recomendado y el más preciso.
    
## <a name="october-2019"></a>Octubre de 2019
 
* Búsqueda de personajes animados en la galería

    Ahora, al indexar personajes animados, puede buscarlos en la galería de vídeos de la cuenta. Para más información, consulte [Reconocimiento de personajes animados](animated-characters-recognition.md).

## <a name="september-2019"></a>Septiembre de 2019
 
En la feria IBC 2019, se anunciaron varios avances:
 
* Reconocimiento de caracteres animados (versión preliminar pública)

    Posibilidad de detectar, agrupar y reconocer caracteres en el contenido animado, gracias a la integración con Custom Vision. Para más información, consulte [Detección de personajes animados](animated-characters-recognition.md).
* Identificación de varios idiomas (versión preliminar pública)

    Detecte segmentos en varios idiomas en la pista de audio y cree una transcripción multilingüe a partir de ellos. Compatibilidad inicial: inglés, español, alemán y francés. Para más información, consulte [Identificación y transcripción automáticas del contenido de varios idiomas](multi-language-identification-transcription.md).
* Extracción de entidades con nombre para personas y ubicaciones

    extrae marcas, ubicaciones y personas del lenguaje hablado y del texto visual mediante el procesamiento de lenguaje natural (NLP).
* Clasificación del tipo de toma editorial

    Etiquetado de tomas con tipos editoriales como cierre, toma media, dos tomas, interior, exterior, etc. Para más información, consulte [Detección del tipo de toma editorial](scenes-shots-keyframes.md#editorial-shot-type-detection).
* Mejora de la inferencia de temas: ahora abarca el nivel 2
    
    El modelo de inferencia de temas ahora admite una granularidad más profunda de la taxonomía IPTC. Lea todos los detalles en [Innovación con inteligencia artificial de Azure Media Services](https://azure.microsoft.com/blog/azure-media-services-new-ai-powered-innovation/).

## <a name="august-2019"></a>Agosto de 2019
 
### <a name="video-analyzer-for-media-deployed-in-uk-south"></a>Video Analyzer for Media implementado en Sur de Reino Unido

Ya puede crear una cuenta de pago de Video Analyzer for Media en la región Sur de Reino Unido.

### <a name="new-editorial-shot-type-insights-available"></a>Nueva información disponible sobre los tipos de capturas editoriales

Las nuevas etiquetas agregadas a las capturas de vídeo proporcionan "tipos de captura" editoriales para identificarlas con las frases editoriales que se suelen emplear en el flujo de trabajo de creación de contenido, como primer plano extremo, primer plano, plano general, plano medio, dos tomas, exteriores, interiores, cara izquierda y cara derecha (disponible en el JSON).

### <a name="new-people-and-locations-entities-extraction-available"></a>Extracción de entidades de nuevas personas y ubicaciones disponible

Video Analyzer for Media identifica las ubicaciones y las personas con nombre a través del procesamiento de lenguaje natural (NLP) desde el OCR y la transcripción del vídeo. Video Analyzer for Media usa el algoritmo de aprendizaje automático para reconocer cuándo se está haciendo mención a ubicaciones específicas (por ejemplo, la Torre Eiffel) o a personas (por ejemplo, John Doe) en un vídeo.

### <a name="keyframes-extraction-in-native-resolution"></a>Extracción de fotogramas clave en resolución nativa

Los fotogramas clave extraídos por Video Analyzer for Media están disponibles en la resolución original del vídeo.
 
### <a name="ga-for-training-custom-face-models-from-images"></a>Disponibilidad general para entrenar modelos de caras personalizadas a partir de imágenes

El entrenamiento de caras a partir de imágenes ha pasado del modo de versión preliminar a disponibilidad general (disponible mediante la API y en el portal).

> [!NOTE]
> La transición de versión preliminar a disponibilidad general no tiene ninguna repercusión en el precio.

### <a name="hide-gallery-toggle-option"></a>Ocultar opción de alternancia de la galería

El usuario puede optar por ocultar la pestaña de la galería en el portal (es similar a ocultar la pestaña de ejemplos).
 
### <a name="maximum-url-size-increased"></a>Aumento del tamaño máximo de la dirección URL

Compatibilidad con la cadena de consulta de dirección URL 4096 (en lugar de 2048) en la indexación de un vídeo.
 
### <a name="support-for-multi-lingual-projects"></a>Compatibilidad con proyectos multilingües

Ahora se pueden crear proyectos basados en vídeos indexados en distintos idiomas (solo API).

## <a name="july-2019"></a>Julio de 2019

### <a name="editor-as-a-widget"></a>Editor como un widget

El editor de IA de Video Analyzer for Media ahora está disponible como widget para insertarse en las aplicaciones de los clientes.

### <a name="update-custom-language-model-from-closed-caption-file-from-the-portal"></a>Actualización del modelo de lenguaje personalizado desde el archivo de subtítulos cerrados desde el portal

Los clientes pueden proporcionar los formatos de archivo VTT, SRT y TTML como entrada para los modelos de idioma en la página de personalización del portal.

## <a name="june-2019"></a>Junio de 2019

### <a name="video-analyzer-for-media-deployed-to-japan-east"></a>Video Analyzer for Media implementado en Este de Japón

Ya puede crear una cuenta de pago de Video Analyzer for Media en la región Este de Japón.

### <a name="create-and-repair-account-api-preview"></a>Crear y reparar la API de la cuenta (versión preliminar)

Se agregó una nueva API que le permite [actualizar el punto de conexión o la clave de la instancia Azure Media Services](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Paid-Account-Azure-Media-Services).

### <a name="improve-error-handling-on-upload"></a>Mejorar el control de errores en la carga 

Se devuelve un mensaje descriptivo en caso de que haya una configuración incorrecta de la cuenta subyacente de Azure Media Services.

### <a name="player-timeline-keyframes-preview"></a>Versión preliminar de los fotogramas clave de la escala de tiempo del reproductor 

Ya puede ver una versión preliminar de la imagen de cada hora en la escala de tiempo del reproductor.

### <a name="editor-semi-select"></a>Selección parcial del editor

Ya puede ver una versión preliminar de todas las conclusiones que se seleccionaron como resultado de elegir un período de tiempo específico en el editor.

## <a name="may-2019"></a>Mayo de 2019

### <a name="update-custom-language-model-from-closed-caption-file"></a>Actualizar el modelo de lenguaje personalizado desde el archivo de subtítulos

Las API para [crear modelos de lenguaje personalizados](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Create-Language-Model) y [actualizar los modelos de lenguaje personalizados](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Language-Model) admiten los formatos de archivo VTT, SRT y TTML como entrada para los modelos de lenguaje.

Al llamar a la [API para actualizar la transcripción de vídeo](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Video-Transcript), la transcripción se agrega automáticamente. El modelo de aprendizaje asociado con el vídeo también se actualiza automáticamente. Para obtener información sobre cómo personalizar y entrenar sus modelos de lenguaje, consulte [Personalización de un modelo de lenguaje con Video Analyzer for Media](customize-language-model-overview.md).

### <a name="new-download-transcript-formats--txt-and-csv"></a>Nuevos formatos de transcripción de descarga: TXT y CSV

Además del formato de subtítulos ya admitidos (SRT, VTT y TTML), Video Analyzer for Media ahora admite la descarga de la transcripción en los formatos TXT y CSV.

## <a name="next-steps"></a>Pasos siguientes

[Información general](video-indexer-overview.md)
