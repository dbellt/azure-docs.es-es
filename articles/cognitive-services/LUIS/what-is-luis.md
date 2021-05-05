---
title: Introducción a Language Understanding (LUIS)
description: 'Language Understanding (LUIS): servicio de API basado en la nube que usa el aprendizaje automático para la conversación y el lenguaje natural para predecir el significado y extraer información.'
keywords: Azure, artificial intelligence, ai, natural language processing, nlp, natural language understanding, nlu, LUIS, conversational AI, ai chatbot, nlp ai, azure luis
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: overview
ms.date: 04/16/2021
ms.custom: cog-serv-seo-aug-2020
ms.openlocfilehash: 3a7484269f16cc55fa16597e74b1d0671d16d980
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108290603"
---
# <a name="what-is-language-understanding-luis"></a>¿Qué es Language Understanding (LUIS)?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Language Understanding (LUIS) es un servicio conversacional de inteligencia artificial basado en la nube que aplica inteligencia de aprendizaje automático personalizado a una conversación o un texto de lenguaje natural de un usuario para predecir el significado global y extraer información pertinente y detallada. LUIS proporciona acceso a través de su [portal personalizado](https://www.luis.ai), las [API][endpoint-apis] y las [bibliotecas cliente del SDK](client-libraries-rest-api.md).

Los usuarios nuevos pueden seguir estos pasos para [iniciar sesión en el portal del servicio LUIS](sign-in-luis-portal.md "Inicio de sesión en el portal de LUIS"). Para empezar, puede probar una aplicación de [dominio precompilado](luis-get-started-create-app.md) de LUIS.

Esta documentación contiene los siguientes tipos de artículos:  

* Los [**inicios rápidos**](luis-get-started-create-app.md) son instrucciones de inicio que le guiarán a la hora de hacer solicitudes al servicio.  
* Las [**guías de procedimientos**](luis-how-to-start-new-app.md) contienen instrucciones para usar el servicio de una manera más específica o personalizada.  
* Los [**conceptos**](artificial-intelligence.md) proporcionan explicaciones detalladas sobre la funcionalidad y las características del servicio.  
* Los [**tutoriales**](tutorial-intents-only.md) son guías más largas que muestran cómo usar el servicio como componente de soluciones empresariales más amplias.  

## <a name="what-does-luis-offer"></a>¿Qué ofrece LUIS? 

* **Simplicidad**: con LUIS se elimina la necesidad de expertos internos en IA o de conocimientos previos de aprendizaje automático. Con solo unos pocos clics puede crear su propia aplicación de inteligencia artificial de conversación. Puede compilar la aplicación personalizada siguiendo uno de nuestros [inicios rápidos](luis-get-started-create-app.md) o puede usar una de nuestras aplicaciones de [dominio precompilado](luis-get-started-create-app.md).
* **Seguridad, privacidad y cumplimiento:** gracias al respaldo de la infraestructura de Azure, LUIS ofrece seguridad, privacidad y cumplimiento de nivel empresarial. Los datos siguen siendo suyos y puede eliminarlos en cualquier momento. Los datos se cifran mientras están almacenados. Aprenda más al respecto [aquí](https://azure.microsoft.com/support/legal/cognitive-services-compliance-and-privacy).
* **Integración:** integre fácilmente la aplicación de LUIS con otros servicios de Microsoft, como [Microsoft Bot Framework](/composer/tutorial/tutorial-luis), [QnA Maker](../QnAMaker/choose-natural-language-processing-service.md) y el [servicio de voz](../speech-service/get-started-intent-recognition.md).


## <a name="luis-scenarios"></a>Escenarios de LUIS
* [Creación de un bot de conversación de nivel empresarial](/azure/architecture/reference-architectures/ai/conversational-bot): esta arquitectura de referencia describe cómo crear un bot de conversación de nivel empresarial (bot de chat) mediante Azure Bot Framework.
* [Bot de chat de comercio](/azure/architecture/solution-ideas/articles/commerce-chatbot): juntos, Azure Bot Service y Language Understanding permiten a los desarrolladores crear interfaces de conversación para diversos escenarios, como banca, viajes y entretenimiento.
* [Control de dispositivos IoT mediante un asistente para voz](/azure/architecture/solution-ideas/articles/iot-controlling-devices-with-voice-assistant): cree interfaces de conversación sencillas con todos los dispositivos accesibles desde Internet (desde un televisor o un frigorífico conectados hasta los dispositivos de una central eléctrica conectada).


## <a name="application-development-life-cycle"></a>Ciclo de vida del desarrollo de aplicaciones

![Ciclo de vida del desarrollo de aplicaciones de LUIS](./media/luis-overview/luis-dev-lifecycle.png "Ciclo de vida de desarrollo de aplicaciones de LUIS")

-   **Planeamiento**: identifique los escenarios con los que los usuarios pueden usar la aplicación. Defina las acciones y la información importante que debe reconocerse.
-   **Compilación**: use el recurso de creación para desarrollar la aplicación. Empiece por definir [intenciones](luis-concept-intent.md) y [entidades](luis-concept-entity-types.md). Luego, agregue [expresiones](luis-concept-utterance.md) de entrenamiento para cada intención. 
-   **Prueba y mejora**: empiece a probar el modelo con otras expresiones para obtener una idea de cómo se comporta la aplicación y decidir si se necesita alguna mejora. Puede mejorar la aplicación siguiendo estos [procedimientos recomendados](luis-concept-best-practices.md). 
-   **Publicación**: implemente la aplicación de cara a la predicción y consulte el punto de conexión mediante el recurso de predicción. Puede encontrar más información sobre la creación y predicción de puntos de conexión [aquí](luis-how-to-azure-subscription.md#luis-resources). 
-   **Conexión**: conéctese a otros servicios, como [Microsoft Bot Framework](/composer/tutorial/tutorial-luis), [QnA Maker](../QnAMaker/choose-natural-language-processing-service.md) y el [servicio de voz](../speech-service/get-started-intent-recognition.md). 
-   **Refinamiento**: [revise las expresiones de punto de conexión](luis-concept-review-endpoint-utterances.md) para mejorar la aplicación con ejemplos reales.

Puede encontrar más información sobre el planeamiento y la creación de aplicaciones [aquí](luis-how-plan-your-app.md).

## <a name="next-steps"></a>Pasos siguientes

* [Novedades](whats-new.md "Novedades") con el servicio y la documentación
* [Compilación de una aplicación de LUIS](tutorial-intents-only.md)
* [Referencia de las API][endpoint-apis]
* [procedimientos recomendados](luis-concept-best-practices.md)
* [Recursos para el desarrollador](developer-reference-resource.md "Recursos para desarrolladores") de LUIS
* [Planee la aplicación](luis-how-plan-your-app.md "Planeamiento de la aplicación") con [intenciones](luis-concept-intent.md "intenciones") y [entidades](luis-concept-entity-types.md "entities").

[bot-framework]: /bot-framework/
[flow]: /connectors/luis/
[authoring-apis]: https://go.microsoft.com/fwlink/?linkid=2092087
[endpoint-apis]: https://go.microsoft.com/fwlink/?linkid=2092356
[qnamaker]: https://qnamaker.ai/