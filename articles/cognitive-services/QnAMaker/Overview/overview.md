---
title: ¿Qué es el servicio QnA Maker?
description: QnA Maker es un servicio NLP API basado en la nube que crea con facilidad una capa natural de conversación con los datos. Se puede usar para encontrar la respuesta más apropiada para una entrada de lenguaje natural determinada, desde la base de conocimiento personalizada (KB) de información.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: overview
ms.date: 01/22/2021
ms.custom: cog-serv-seo-aug-2020
keywords: qna maker, low code chat bots, multi-turn conversations
ms.openlocfilehash: 84678e0472284f6a9acfec53fbe31ff550775446
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108292673"
---
# <a name="what-is-qna-maker"></a>¿Qué es QnA Maker?

QnA Maker es un servicio de Procesamiento de lenguaje natural (NLP) basado en la nube que le permite crear una capa de conversación natural con los datos. Se utiliza para encontrar la respuesta más apropiada para una entrada de la base de conocimiento personalizada (KB) de información.

QnA Maker se usa normalmente para crear aplicaciones cliente de conversación, entre las que se incluyen aplicaciones de medios sociales, bots de chat y aplicaciones de escritorio habilitadas para voz.

QnA Maker no almacena datos de clientes. Todos los datos de clientes (respuestas de preguntas y registros de chat) se almacenan en la región en la que el cliente implementa las instancias de servicio dependientes. Para más información acerca de los servicios dependientes, consulte [aquí](../concepts/plan.md?tabs=v1).

## <a name="when-to-use-qna-maker"></a>Cuándo usar QnA Maker

* **Cuando tiene información estática**: use QnA Maker cuando tenga información estática en la base de conocimiento de las respuestas. Esta base de conocimiento está personalizada para sus necesidades, que ha creado con documentos como [archivos PDF y direcciones URL.](../Concepts/data-sources-and-content.md)
* **Si desea proporcionar la misma respuesta a una solicitud, una pregunta o un comando**: cuando distintos usuarios envían la misma pregunta, se devuelve la misma respuesta.
* **Si desea filtrar la información estática en función de la metainformación**: agregue etiquetas de [metadatos](../how-to/metadata-generateanswer-usage.md) para proporcionar opciones de filtrado adicionales relacionadas con los usuarios de la aplicación cliente y la información. La información común de los metadatos incluye el tipo de contenido o formato [de charla](../how-to/chit-chat-knowledge-base.md), y el propósito y la actualización del contenido.
* **Si desea administrar una conversación de bot que incluya información estática**: la base de conocimiento toma el texto o el comando de conversación de un usuario y lo responde. Si la respuesta forma parte de un flujo de conversación determinado previamente, representado en la base de conocimiento con [contexto multiturno](../how-to/multiturn-conversation.md), el bot puede proporcionar fácilmente este flujo.

## <a name="what-is-a-knowledge-base"></a>¿Qué es una base de conocimiento?

QnA Maker [importa el contenido](../Concepts/plan.md) en una base de conocimiento de pares de preguntas y respuestas. En el proceso de importación se extrae información sobre la relación entre las partes del contenido estructurado y semiestructurado para insinuar relaciones entre los conjuntos de preguntas y respuestas. Puede editar estos pares de preguntas y respuestas o agregar otros nuevos.

El contenido del par de preguntas y respuestas incluye:
* Todas las formas alternativas de la pregunta
* Etiquetas de metadatos utilizadas para filtrar las opciones de respuesta durante la búsqueda
* Indicaciones de seguimiento para continuar con el perfeccionamiento de la búsqueda

![Ejemplo de pregunta y respuesta con metadatos](../media/qnamaker-overview-learnabout/example-question-and-answer-with-metadata.png)

Después de publicar la base de conocimiento, una aplicación cliente envía una pregunta de usuario al punto de conexión. El servicio de QnA Maker procesa la pregunta y responde con la mejor respuesta.

## <a name="create-a-chat-bot-programmatically"></a>Creación de un bot de chat mediante programación

Una vez publicada una base de conocimiento de QnA Maker, una aplicación cliente envía una pregunta al punto de conexión de la base de conocimiento y recibe los resultados como una respuesta JSON. Una aplicación cliente común para QnA Maker es un bot de chat.

![Formulación de una pregunta a un bot y respuesta del contenido de la base de conocimiento](../media/qnamaker-overview-learnabout/bot-chat-with-qnamaker.png)

|Paso|Acción|
|:--|:--|
|1|La aplicación cliente envía la _pregunta_  del usuario (texto en sus propias palabras) "Cómo actualizar mi base de conocimiento mediante programación?" al punto de conexión de la base de conocimiento.|
|2|QnA Maker usa la base de conocimiento entrenada para proporcionar la respuesta correcta y las solicitudes de seguimiento que se pueden usar para refinar la búsqueda de la mejor respuesta. QnA Maker devuelve una respuesta con formato JSON.|
|3|La aplicación cliente usa la respuesta JSON para tomar decisiones acerca de cómo continuar con la conversación. Estas decisiones pueden incluir mostrar la respuesta principal y presentar más opciones para refinar la búsqueda de la mejor respuesta. |
|||

## <a name="build-low-code-chat-bots"></a>Creación de bots de chat con poco código

En el portal de QnA Maker se proporciona toda la experiencia de creación de la base de conocimiento. Puede importar documentos en su formulario actual a la base de conocimiento. Estos documentos (como las preguntas frecuentes, el manual del producto, la hoja de cálculo o la página web) se convierten en pares de preguntas y respuestas. En cada par se analizan los mensajes de seguimiento y cada par se conecta a otros pares. El formato de _marcado_ final admite una presentación enriquecida que incluye imágenes y vínculos.

Una vez editada la base de conocimiento, publíquela [en un bot de Azure Web App](https://azure.microsoft.com/services/bot-service/) en funcionamiento sin escribir ningún código. Pruebe el bot en [Azure Portal](https://portal.azure.com) o descárguelo y continúe el desarrollo.

## <a name="high-quality-responses-with-layered-ranking"></a>Respuestas de alta calidad con clasificación por capas

El sistema de QnA Maker es un enfoque de clasificación por capas. Los datos se almacenan en la búsqueda de Azure, que también actúa como la primera capa de clasificación. Los resultados principales de la búsqueda de Azure se pasan en el modelo de reclasificación NLP de QnA Maker para generar los resultados finales y la puntuación de confianza.

## <a name="multi-turn-conversations"></a>Conversaciones multiturno

QnA Maker proporciona solicitudes multiturno y aprendizaje activo para ayudarle a mejorar sus pares de preguntas y respuestas básicos.

**Los mensajes multiturno** le ofrecen la oportunidad de conectar pares de preguntas y respuestas. Esta conexión permite a la aplicación cliente proporcionar una respuesta superior y proporciona más preguntas para refinar la búsqueda de una respuesta final.

Una vez que la base de conocimiento recibe las preguntas de los usuarios en el punto de conexión publicado, QnA Maker aplica el **aprendizaje activo** a estas preguntas reales para sugerir cambios en la base de conocimiento y mejorar la calidad.

## <a name="development-lifecycle"></a>Ciclo de vida del desarrollo

QnA Maker permite la creación, el entrenamiento y la publicación, además de ofrecer permisos de colaboración para integrarse en el ciclo de vida de desarrollo completo.

> [!div class="mx-imgBorder"]
> ![Imagen conceptual del ciclo de desarrollo](../media/qnamaker-overview-learnabout/development-cycle.png)


## <a name="complete-a-quickstart"></a>Completar una guía de inicio rápido

Ofrecemos guías de inicio rápido en los lenguajes de programación más populares, cuyo diseño individual le permite ejecutar el código en menos de 10 minutos. Consulte la siguiente lista para obtener la guía de inicio rápido de cada característica.

* [Introducción a la biblioteca cliente de QnA Maker](../quickstarts/quickstart-sdk.md)
* [Introducción al portal de QnA Maker](../quickstarts/create-publish-knowledge-base.md)

## <a name="next-steps"></a>Pasos siguientes
QnA Maker proporciona todo lo que necesita para compilar, administrar e implementar la base de conocimiento personalizada.

> [!div class="nextstepaction"]
> [Revisar los últimos cambios](../whats-new.md)