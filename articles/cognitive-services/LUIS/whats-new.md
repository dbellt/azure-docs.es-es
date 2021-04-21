---
title: Novedades de Language Understanding (LUIS)
description: Este artículo se actualiza periódicamente con noticias sobre la API de Language Understanding de Azure Cognitive Services.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: overview
ms.date: 04/07/2021
ms.openlocfilehash: f6c6f2ba20a67cd39d4142ffadddc803927c936c
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107227065"
---
# <a name="whats-new-in-language-understanding"></a>Novedades de Language Understanding

Conozca las novedades del servicio. Estos elementos incluyen notas de la versión, vídeos, entradas de blogs y otros tipos de información. Marque esta página para mantenerse actualizado con el servicio.

## <a name="release-notes"></a>Notas de la versión

### <a name="april-2021"></a>Abril de 2021

* [Región de creación](luis-reference-regions.md#publishing-to-europe) Norte de Suiza.

### <a name="january-2021"></a>Enero de 2021

* La API de predicción V3 ahora es compatible con [Bing Spellcheck API](luis-tutorial-bing-spellcheck.md).
* Los portales regionales (au.luis.ai y eu.luis.ai) se han consolidado en un único portal y dirección URL. Si usa alguno de estos portales, se obtendrá acceso automáticamente a luis.ai.

### <a name="december-2020"></a>Diciembre de 2020

* Todos los usuarios de LUIS deben [migrar a un recurso de creación de LUIS](luis-migration-authoring.md).
* Nuevos [puntos de conexión de evaluación](luis-how-to-batch-test.md#batch-testing-using-the-rest-api) que permiten enviar pruebas por lotes con la API REST y obtener resultados de precisión para las intenciones y entidades. Disponible a partir dela versión v3.0-preview del punto de conexión de LUIS.

### <a name="june-2020"></a>Junio de 2020

* SDK de [creación de la versión preliminar 3.0](luis-migration-authoring-entities.md)
    * Versión 3.2.0-Preview. 3-[.NET-NuGet](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring/)
    * Versión 4.0.0-preview.3 - [JS - NPM](https://www.npmjs.com/package/@azure/cognitiveservices-luis-authoring)
* Aplicación de prácticas de DevOps con LUIS
    * Conceptos
        * [Prácticas de DevOps para LUIS](luis-concept-devops-sourcecontrol.md)
        * [Flujos de trabajo de integración continua y entrega continua para LUIS DevOps](luis-concept-devops-automation.md)
        * [Prueba de LUIS DevOps](luis-concept-devops-testing.md)
    * Instrucciones
        * [Aplicación de DevOps al desarrollo de aplicaciones de LUIS mediante Acciones de GitHub](luis-how-to-devops-with-github.md)
    * [Repositorio de GitHub de código completo](https://github.com/Azure-Samples/LUIS-DevOps-Template)

### <a name="may-2020---build"></a>Mayo de 2020 -//Build

* Publicada como **disponibilidad general** (GA):
    * [Contenedor de Language Understanding](luis-container-howto.md)
    * Portal en versión preliminar promovido al [portal actual](https://www.luis.ai), el portal [anterior](https://previous.luis.ai) todavía está disponible
    * Nueva experiencia de creación y etiquetado de entidades de aprendizaje automático
    * [Proceso de actualización](migrate-from-composite-entity.md) de entidades compuestas y sencillas a entidades de aprendizaje automático
    * Compatibilidad de [configuraciones](how-to-application-settings-portal.md) para normalizar variantes de palabras
* Cambios en las API de creación en versión preliminar
    * Esquema de aplicación 7.x para entidades anidadas de aprendizaje automático
    * [Migración a la característica necesaria](luis-migration-authoring-entities.md#api-change-constraint-replaced-with-required-feature)
* Nuevos recursos para desarrolladores
    * [Herramientas de integración continua](developer-reference-resource.md#continuous-integration-tools)
    * Taller: Aprenda procedimientos recomendados para la [_Comprensión del lenguaje natural_ (NLU) con LUIS](developer-reference-resource.md#workshops)
* [Claves administradas por el cliente](./encrypt-data-at-rest.md): cifre todos los datos que use en LUIS mediante su propia clave
* [Show de IA](https://channel9.msdn.com/Shows/AI-Show/New-Features-in-Language-Understanding) (vídeo): consulte las nuevas características de LUIS



### <a name="march-2020"></a>Marzo de 2020

* TLS 1.2 ya se exige en todas las solicitudes HTTP para este servicio. Para más información, consulte [Seguridad de Azure Cognitive Services](../cognitive-services-security.md).

### <a name="november-4-2019---ignite"></a>4 de noviembre de 2019: Ignite

* Vídeo: [Modelos avanzados de comprensión del lenguaje natural (NLU) con LUIS y Azure Cognitive Services | BRK2188](https://www.youtube.com/watch?v=JdJEV2jV0_Y)

* Mayor productividad de los desarrolladores
    * Disponibilidad general de nuestro [punto de conexión de predicción V3](luis-migration-api-v3.md).
    * Capacidad de importar y exportar aplicaciones con el formato `.lu` ([LUDown](https://github.com/microsoft/botbuilder-tools/tree/master/packages/Ludown)). Esto prepara el camino para un proceso de CI/CD efectivo.
* Expansión de idioma
    * [Árabe e hindi](luis-language-support.md) en versión preliminar pública.
* Modelos creados previamente
    * Los [dominios precompilados](luis-reference-prebuilt-domains.md) ya están disponibles con carácter general (GA)
    * [Entidades pregeneradas](luis-reference-prebuilt-entities.md#japanese-entity-support) en japonés: edad, moneda, número y porcentaje no se admiten en la versión V3.
    * [Entidades pregeneradas](luis-reference-prebuilt-entities.md#italian-entity-support) en italiano: edad, moneda, dimensión, número y resolución porcentual se cambiaron desde la versión V2.
* Mejore la experiencia del usuario en el [portal de preview.luis.ai](https://preview.luis.ai): una experiencia de etiquetado renovada para habilitar la compilación y depuración de modelos complejos. Pruebe los tutoriales del portal de versión preliminar:
    * [Solo intenciones](tutorial-intents-only.md)
    * [Entidad de aprendizaje automático que se puede descomponer](tutorial-machine-learned-entity.md)
* Funcionalidades de comprensión del lenguaje avanzadas: [cree modelos de lenguaje sofisticados](luis-concept-entity-types.md) con menos esfuerzo.
* Defina características de aprendizaje automático en el nivel de modelo y permita que los modelos se usen como señales para otro modelo; por ejemplo, use entidades como características para las intenciones y otras entidades.
* [Límites](luis-limits.md) nuevos y ampliados: un límite máximo más elevado para las listas de frases y las frases totales, nuevos límites de modelo como característica
* Extraiga información de texto en el formato de la estructura de jerarquía profunda, lo que permite que las aplicaciones de conversación sean más eficaces.

    ![imagen de entidad de aprendizaje automático](./media/whats-new/deep-entity-extraction-example.png)

### <a name="september-3-2019"></a>3 de septiembre de 2019

* Recurso de creación de Azure: [migrar ahora](luis-migration-authoring.md).
    * 500 aplicaciones por recurso de Azure
    * 100 versiones por aplicación
* Compatibilidad del turco con las entidades precompiladas
* Compatibilidad del italiano con datetimeV2

### <a name="july-23-2019"></a>23 de julio de 2019

* Actualización de [Recognizers-Text](https://github.com/microsoft/Recognizers-Text/releases/tag/dotnet-v1.2.3) a 1.2.3
    * Reconocedores de edad, temperatura, dimensión y moneda en italiano.
    * Mejora en el reconocimiento de festividades en inglés para calcular correctamente las fechas basadas en el Día de Acción de Gracias.
    * Mejoras en DateTime en francés para reducir los falsos positivos de entidades que no son de fecha y hora.
    * Compatibilidad con el año natural, escolar o fiscal y acrónimos en DateRange en inglés.
    * Reconocimiento de PhoneNumber mejorado en chino y japonés.
    * Compatibilidad mejorada con NumberRange en inglés.
    * Mejoras en el rendimiento.

### <a name="june-24-2019"></a>24 de junio de 2019

* [Entidad creada previamente OrdinalV2](luis-reference-prebuilt-ordinal-v2.md) para permitir la ordenación, como siguiente, anterior y último. Referencia cultural inglesa únicamente.

### <a name="may-6-2019---build-conference"></a>6 de mayo de 2019: conferencia Build

Las siguientes características se presentaron en el congreso Build 2019:

* [Versión preliminar de la guía de migración de la API v3](luis-migration-api-v3.md)
* [Panel de análisis mejorado](luis-how-to-use-dashboard.md)
* [Dominios creados previamente mejorados](luis-reference-prebuilt-domains.md)
* [Entidades de lista dinámica](schema-change-prediction-runtime.md#dynamic-lists-passed-in-at-prediction-time)
* [Entidades externas](schema-change-prediction-runtime.md#external-entities-passed-in-at-prediction-time)

## <a name="blogs"></a>Blogs

[Bot Framework](https://blog.botframework.com/)

## <a name="videos"></a>Vídeos

### <a name="2019-ignite-videos"></a>Vídeos de Ignite de 2019

[Modelos avanzados de comprensión del lenguaje natural (NLU) con LUIS y Azure Cognitive Services | BRK2188](https://www.youtube.com/watch?v=JdJEV2jV0_Y)

### <a name="2019-build-videos"></a>Vídeos de Build 2019

[How to use Azure Conversational AI to scale your business for the next generation](https://www.youtube.com/watch?v=_k97jd-csuk&feature=youtu.be) (Cómo usar la inteligencia artificial de Azure Conversational para llevar su negocio a la próxima generación)

## <a name="service-updates"></a>Actualizaciones del servicio

[Anuncios de actualización de Azure para Cognitive Services](https://azure.microsoft.com/updates/?product=cognitive-services)