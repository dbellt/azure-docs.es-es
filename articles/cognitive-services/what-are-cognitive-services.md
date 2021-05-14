---
title: ¿Qué es Azure Cognitive Services?
titleSuffix: Azure Cognitive Services
description: Cognitive Services hace la AI accesible a todos los desarrolladores, sin necesidad de tener conocimientos de aprendizaje automático y de ciencia de datos. Solo tiene que realizar una llamada de API desde su aplicación para agregar la capacidad de ver (búsqueda y reconocimiento avanzados de imágenes), oír, hablar, buscar y tomar decisiones a las aplicaciones.
services: cognitive-services
author: nitinme
manager: nitinme
keywords: cognitive services, cognitive intelligence, cognitive solutions, ai services, cognitive understanding, cognitive features
ms.service: cognitive-services
ms.subservice: ''
ms.topic: overview
ms.date: 04/16/2021
ms.author: nitinme
ms.custom: cog-serv-seo-aug-2020
ms.openlocfilehash: c6b83a25816c29f1642625618e66624aa3e0a6de
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108289811"
---
# <a name="what-are-azure-cognitive-services"></a>¿Qué es Azure Cognitive Services?

Azure Cognitive Services son servicios basados en la nube con API REST y SDK de biblioteca cliente disponibles para ayudarle a integrar inteligencia cognitiva en sus aplicaciones. Puede agregar características cognitivas a sus aplicaciones sin tener conocimientos sobre inteligencia artificial (IA) o ciencia de datos. Azure Cognitive Services incluye varios servicios de inteligencia artificial que le permiten desarrollar soluciones cognitivas con capacidad de ver, oír, hablar, comprender e incluso tomar decisiones.

## <a name="categories-of-cognitive-services"></a>Categorías de Cognitive Services

El catálogo de Cognitive Services, que facilita la comprensión cognitiva, consta de cinco pilares básicos:

* Visión
* Voz
* Idioma
* Decisión
* Search

Las siguientes secciones de este artículo ofrecen una lista de los servicios que forman parte de estos cinco pilares.

## <a name="vision-apis"></a>API de visión

|Nombre de servicio|Descripción del servicio|
|:-----------|:------------------|
|[Computer Vision](./computer-vision/index.yml "Computer Vision")|El servicio Computer Vision proporciona acceso a algoritmos cognitivos avanzados para procesar imágenes y devolver información. Consulte [Inicio rápido de Computer Vision](./computer-vision/quickstarts-sdk/client-library.md) para empezar a trabajar con el servicio.|
|[Custom Vision Service](./custom-vision-service/index.yml "Custom Vision Service")|El servicio Custom Vision le permite crear, implementar y mejorar los clasificadores de imágenes propios. Un clasificador de imágenes es un servicio de inteligencia artificial que aplica etiquetas a las imágenes, en función de sus características visuales. |
|[Face](./face/index.yml "Caras")| El servicio Face proporciona acceso a algoritmos faciales avanzados, lo que permite la detección y el reconocimiento de atributos faciales. Consulte [Inicio rápido de Face](./face/quickstarts/client-libraries.md) para empezar a trabajar con el servicio.|
|[Form Recognizer](./form-recognizer/index.yml "Form Recognizer")|Form Recognizer identifica y extrae pares clave-valor y datos de tabla de documentos de formulario; luego, genera datos estructurados, incluidas las relaciones en el archivo original. Consulte [Inicio rápido de Form Recognizer](./form-recognizer/quickstarts/client-library.md) para comenzar.|
|[Video Indexer](../media-services/video-indexer/video-indexer-overview.md "Video Indexer")|Video Indexer permite extraer información de un vídeo. Consulte [Inicio rápido de Video Indexer](../media-services/video-indexer/video-indexer-get-started.md) para comenzar.|

## <a name="speech-apis"></a>API de voz

|Nombre de servicio|Descripción del servicio|
|:-----------|:------------------|
|[Servicio Voz](./speech-service/index.yml "Servicio de voz")|El servicio de voz agrega a las aplicaciones características habilitadas por voz. El servicio Voz incluye varias funciones, entre otras: voz a texto, texto a voz y traducción de voz.|
<!--
|[Speaker Recognition API](./speech-service/speaker-recognition-overview.md "Speaker Recognition API") (Preview)|The Speaker Recognition API provides algorithms for speaker identification and verification.|
|[Bing Speech](./speech-service/how-to-migrate-from-bing-speech.md "Bing Speech") (Retiring)|The Bing Speech API provides you with an easy way to create speech-enabled features in your applications.|
|[Translator Speech](/azure/cognitive-services/translator-speech/ "Translator Speech") (Retiring)|Translator Speech is a machine translation service.|
-->
## <a name="language-apis"></a>API de idioma

|Nombre de servicio|Descripción del servicio|
|:-----------|:------------------|
|[Language Understanding: LUIS](./luis/index.yml "Language Understanding")|Language Understanding (LUIS) es un servicio conversacional de inteligencia artificial basado en la nube que aplica inteligencia de aprendizaje automático personalizado a una conversación o un texto de lenguaje natural de un usuario para predecir el significado global y extraer información pertinente y detallada. Consulte [Inicio rápido de Luis](./luis/luis-get-started-create-app.md) para empezar a trabajar con el servicio.|
|[QnA Maker](./qnamaker/index.yml "QnA Maker")|QnA Maker permite generar un servicio de preguntas y respuestas a partir de contenido semiestructurado. [Consulte el inicio rápido de QnA Maker](./qnamaker/quickstarts/create-publish-knowledge-base.md) para empezar a trabajar con el servicio.|
|[Text Analytics](./text-analytics/index.yml "Text Analytics")| Proporciona procesamiento de lenguaje natural en texto sin formato para el análisis de opiniones, la extracción de frases clave y la detección del idioma. Consulte [Inicio rápido de Text Analytics](./text-analytics/quickstarts/client-libraries-rest-api.md) para empezar a trabajar con el servicio.|
|[Traductor](./translator/index.yml "Traductor")|Traductor proporciona la traducción de texto automática casi en tiempo real.|
| [Lector inmersivo](./immersive-reader/index.yml "Lector inmersivo") | El Lector inmersivo agrega a las aplicaciones funcionalidad de comprensión y lectura en pantalla. Consulte [Inicio rápido de Lector inmersivo](./immersive-reader/quickstarts/client-libraries.md) para empezar a trabajar con el servicio. |

## <a name="decision-apis"></a>API de Decision

|Nombre de servicio|Descripción del servicio|
|:-----------|:------------------|
|[Anomaly Detector](./anomaly-detector/index.yml "Anomaly Detector") |Anomaly Detector permite supervisar y detectar anomalías en datos de series temporales. Consulte [Inicio rápido de Anomaly Detector](./anomaly-detector/quickstarts/client-libraries.md) para empezar a trabajar con el servicio.|
|[Content Moderator](./content-moderator/overview.md "Content Moderator")|Content Moderator proporciona la supervisión de posibles contenidos ofensivos, indeseables y peligrosos. Consulte [Inicio rápido de Content Moderator](./content-moderator/client-libraries.md) para empezar a trabajar con el servicio.|
|[Metrics Advisor](./metrics-advisor/index.yml) (versión preliminar) | Metrics Advisor proporciona una detección de anomalías personalizable en los datos de serie temporal de varios variados y un portal web con todas las características para ayudarle a usar el servicio. Consulte [Inicio rápido de Metrics Advisor](./metrics-advisor/quickstarts/rest-api-and-client-library.md) para empezar a trabajar con el servicio. |
|[Personalizer](./personalizer/index.yml "Personalizer")|Personalizer permite elegir la mejor experiencia para mostrar a los usuarios y aprender de su comportamiento en tiempo real. Consulte [Inicio rápido de Personalizer](./personalizer/quickstart-personalizer-sdk.md) para empezar a trabajar con el servicio.|

## <a name="search-apis"></a>API de búsqueda

> [!NOTE]
> ¿Busca [Azure Cognitive Search](../search/index.yml)? Aunque usa Cognitive Services para algunas tareas, es una tecnología de búsqueda diferente que admite otros escenarios.

|Nombre de servicio|Descripción del servicio|
|:-----------|:------------------|
|[Bing News Search](/azure/cognitive-services/bing-news-search/ "Bing News Search")|Bing News Search devuelve una lista de artículos de noticias cuya relevancia se ha determinado para la consulta del usuario.|
|[Bing Video Search](/azure/cognitive-services/Bing-Video-Search/ "Bing Video Search")|Bing Video Search devuelve una lista de vídeos cuya relevancia se ha determinado para la consulta del usuario.|
|[Bing Web Search](./bing-web-search/index.yml "Bing Web Search")|Bing Web Search devuelve una lista de resultados de la búsqueda cuya relevancia se ha determinado para la consulta del usuario.|
|[Bing Autosuggest](/azure/cognitive-services/Bing-Autosuggest "Bing Autosuggest")|Bing Autosuggest permite enviar un término de consulta de búsqueda parcial a Bing y obtener una lista de consultas sugeridas.|
|[Bing Custom Search](/azure/cognitive-services/bing-custom-search "Bing Custom Search")|Bing Custom Search permite crear experiencias de búsqueda a medida de los temas que le interesan.|
|[Bing Entity Search](/azure/cognitive-services/bing-entities-search/ "Bing Entity Search")|Bing Entity Search devuelve información sobre las entidades que Bing determina que están relacionadas con la consulta del usuario.|
|[Bing Image Search](/azure/cognitive-services/bing-image-search "Bing Image Search")|Bing Image Search devuelve una lista de imágenes cuya relevancia se ha determinado para la consulta del usuario.|
|[Bing Visual Search](/azure/cognitive-services/bing-visual-search "Bing Visual Search")|Bing Visual Search devuelve información sobre una imagen, como imágenes visualmente similares, orígenes de compra de productos encontrados en la imagen y búsquedas relacionadas.|
|[Bing Local Business Search](/azure/cognitive-services/bing-local-business-search/ "Bing Local Business Search")| La API Bing Local Business Search permite que las aplicaciones busquen información de ubicación y contacto sobre las empresas locales en función de las consultas de búsqueda.|
|[Bing Spell Check](/azure/cognitive-services/bing-spell-check/ "Bing Spell Check")|Bing Spell Check permite realizar correcciones de gramática y ortografía en contexto.|

## <a name="get-started-with-cognitive-services"></a>Introducción a Cognitive Services

Aprenda a crear un recurso de Cognitive Services con inicios rápidos prácticos mediante el uso de:

* [Azure Portal](cognitive-services-apis-create-account.md?tabs=multiservice%2Cwindows "Azure portal")
* [CLI de Azure](cognitive-services-apis-create-account-cli.md?tabs=windows "CLI de Azure")
* [Bibliotecas cliente de Azure SDK](cognitive-services-apis-create-account-cli.md?tabs=windows "cognitive-services-apis-create-account-client-library?pivots=programming-language-csharp")
* [Plantillas de Azure Resource Manager (ARM)](./create-account-resource-manager-template.md?tabs=portal "Plantillas de Azure Resource Manager (ARM)")

## <a name="using-cognitive-services-in-different-development-environments"></a>Uso de Cognitive Services en entornos de desarrollo diferentes

Con Azure y Cognitive Services, dispone de varias opciones de desarrollo, como las siguientes:

* Herramientas de automatización e integración, como Logic Apps y Power Automate.
* Opciones de implementación, como Azure Functions y App Service. 
* Contenedores de Cognitive Services para el acceso seguro.
* Herramientas como Apache Spark, Azure Databricks, Azure Synapse Analytics y Azure Kubernetes Service para escenarios de macrodatos. 

Para más información, consulte [Opciones de desarrollo de Cognitive Services](./cognitive-services-development-options.md).

<!--
## Subscription management

Once you are signed in with your Microsoft Account, you can access [My subscriptions](https://www.microsoft.com/cognitive-services/subscriptions "My subscriptions") to show the products you are using, the quota remaining, and the ability to add additional products to your subscription.

## Upgrade to unlock higher limits

All APIs have a free tier, which has usage and throughput limits.  You can increase these limits by using a paid offering and selecting the appropriate pricing tier option when deploying the service in the Azure portal. [Learn more about the offerings and pricing](https://azure.microsoft.com/pricing/details/cognitive-services/ "offerings and pricing"). You'll need to set up an Azure subscriber account with a credit card and a phone number. If you have a special requirement or simply want to talk to sales, click "Contact us" button at the top the pricing page.
-->

## <a name="using-cognitive-services-securely"></a>Uso de Cognitive Services de forma segura

Azure Cognitive Services proporciona un modelo de seguridad por niveles, lo que incluye la [autenticación ](authentication.md "autenticación") a través de credenciales de Azure Active Directory, una clave de recurso válida e [instancias de Azure Virtual Network](cognitive-services-virtual-networks.md "Instancias de Azure Virtual Network").

## <a name="containers-for-cognitive-services"></a>Contenedores para Cognitive Services

 Azure Cognitive Services proporciona varios contenedores de Docker que permiten usar las mismas API disponibles en Azure de forma local. El uso de estos contenedores le brinda la flexibilidad de acercar Cognitive Services a sus datos por razones de cumplimiento, seguridad u otras razones operativas. Obtenga más información sobre [contenedores de Cognitive Services](cognitive-services-container-support.md "Contenedores de Cognitive Services").

## <a name="regional-availability"></a>Disponibilidad regional

Las API de Cognitive Services se hospedan en una red cada vez mayor de centros de datos administrados por Microsoft. Encontrará la disponibilidad regional de cada API en la [lista de regiones de Azure](https://azure.microsoft.com/regions "Lista de regiones de Azure").

¿Busca una región que aún no se admite? Díganoslo. Envíe una solicitud de característica en nuestro [foro de UserVoice](https://feedback.azure.com/forums/932041-azure-cognitive-services "Foro de UserVoice").

## <a name="supported-cultural-languages"></a>Idiomas culturales admitidos

Cognitivas Services admite una amplia gama de idiomas culturales en el nivel de servicio. Puede encontrar la disponibilidad de idiomas para cada API en la [lista de idiomas admitidos](language-support.md "lista de idiomas que se admiten").

## <a name="certifications-and-compliance"></a>Certificaciones y cumplimiento

Cognitivas Services ha recibido certificaciones como la certificación CSA STAR, FedRAMP Moderate y HIPAA BAA. Puede [descargar](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942 "descarga") certificaciones para sus propias auditorías y revisiones de seguridad.

Para comprender la administración de los datos y la privacidad, vaya al [Centro de confianza](https://servicetrust.microsoft.com/ "Centro de confianza").

## <a name="support"></a>Soporte técnico

Cognitive Services proporciona varias opciones de soporte técnico para ayudarle a avanzar en el desarrollo de aplicaciones inteligentes. Cognitive Services también cuenta con una sólida comunidad de desarrolladores que pueden ayudarle a responder preguntas específicas. Para obtener una lista completa de las opciones disponibles, consulte [Opciones de ayuda y de soporte técnico de Azure Cognitive Services](cognitive-services-support-options.md "Opciones de ayuda y de soporte técnico de Azure Cognitive Services").

## <a name="next-steps"></a>Pasos siguientes

* [Creación de una cuenta de Cognitive Services](cognitive-services-apis-create-account.md "Creación de una cuenta de Cognitive Services")
* [Novedades en la documentación de Cognitive Services](whats-new-docs.md "Novedades en la documentación de Cognitive Services")
* [Planeamiento y administración de los costos de Cognitive Services](plan-manage-costs.md)