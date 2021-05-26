---
title: 'Planeamiento de la aplicación: QnA Maker'
description: Aprenda a planear la aplicación de QnA Maker. Descripción de cómo funciona QnA Maker y cómo interactúa con otros servicios de Azure y algunos conceptos de las bases de conocimiento.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: 177c02da26ffcedd0453538bc32158cf2f014896
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110369292"
---
# <a name="plan-your-qna-maker-app"></a>Planeamiento de la aplicación de QnA Maker

Para planear la aplicación de QnA Maker, debe comprender cómo funciona QnA Maker y cómo interactúa con otros servicios de Azure. También debe tener una idea sólida de los conceptos de las bases de conocimiento.

## <a name="azure-resources"></a>Recursos de Azure

Cada [recurso de Azure](azure-resources.md#resource-purposes) creado con QnA Maker tiene un propósito específico. Cada recurso tiene su propia finalidad, límites y [plan de tarifa](azure-resources.md#pricing-tier-considerations). Es importante comprender la función de estos recursos para que pueda usar ese conocimiento en el proceso de planeamiento.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

| Recurso | Propósito |
|--|--|
| Recurso de [QnA Maker](azure-resources.md#qna-maker-resource) | Creación y predicción de consultas |
| Recurso de [Cognitive Search](azure-resources.md#cognitive-search-resource) | Almacenamiento y búsqueda de datos |
| Recurso de [App Service y plan de App Service](azure-resources.md#app-service-and-app-service-plan) | Consulta de un punto de conexión de predicción |
| Recurso de [Application Insights](azure-resources.md#application-insights) | Telemetría de predicción de consultas |


# <a name="custom-question-answering-preview-release"></a>[Respuesta personalizada a preguntas (versión preliminar)](#tab/v2)

| Resource | Propósito |
|--|--|
| Recurso de [Text Analytics](azure-resources.md#qna-maker-resource) | Creación, telemetría y punto de conexión de consulta de predicciones|
| Recurso de [Cognitive Search](azure-resources.md#cognitive-search-resource) | Almacenamiento y búsqueda de datos |

---
### <a name="resource-planning"></a>Planeamiento de recursos

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

El nivel Gratis (`F0`) de cada recurso funciona y puede proporcionar tanto la experiencia de creación como de predicción de consultas. Puede usar este nivel para aprender sobre la creación y la predicción de consultas. Cuando pase a un escenario de producción o activo, vuelva a evaluar la selección de recursos.

# <a name="custom-question-answering-preview-release"></a>[Respuesta personalizada a preguntas (versión preliminar)](#tab/v2)

Respuesta personalizada a preguntas (versión preliminar) es una característica gratuita con un rendimiento limitado actualmente a diez  transacciones por segundo para las API de administración y de predicción. Con el fin de obtener un rendimiento de diez transacciones por segundo con su servicio, se recomienda la SKU S1 (1 instancia) de Azure Cognitive Search.

### <a name="text-analytics-resource"></a>Recurso de Text Analytics

Un único recurso de Text Analytics que tenga habilitada la característica de respuesta personalizada a preguntas puede hospedar más de una knowledge base. El número de bases de conocimiento está determinado por la cantidad de índices admitidos en el plan de tarifa de Cognitive Search. Obtenga más información sobre la [relación de los índices con las bases de conocimiento](azure-resources.md#index-usage).

---

### <a name="knowledge-base-size-and-throughput"></a>Tamaño y rendimiento de la base de conocimiento

Cuando compile una aplicación real, planee recursos suficientes para el tamaño de la base de conocimiento y las solicitudes de predicción de consultas que espere.

El tamaño de la base de conocimiento lo controlan los siguientes elementos:
* Los límites del plan de tarifa del [recurso de Cognitive Search](../../../search/search-limits-quotas-capacity.md)
* Los [límites de QnA Maker](../limits.md)

La solicitud de predicción de consultas de la base de conocimiento se controla mediante el plan de la aplicación web y la propia aplicación web. Consulte la [configuración recomendada](azure-resources.md#recommended-settings) para decidir el plan de tarifa.

### <a name="resource-sharing"></a>Uso compartido de recursos

Si ya tiene algunos de estos recursos en uso, puede considerar la posibilidad de compartirlos. Consulte qué recursos [se pueden compartir](azure-resources.md#share-services-with-qna-maker) sabiendo que el uso compartido de recursos es un escenario avanzado.

Todas las bases de conocimiento creadas en el mismo recurso de QnA Maker comparten el punto de conexión de predicción de consultas de **prueba**.

### <a name="understand-the-impact-of-resource-selection"></a>Descripción del impacto de la selección de recursos

Para seleccionar los recursos de manera adecuada, la base de conocimiento debe responder a las predicciones de consultas correctamente.

Si la base de conocimiento no funciona de forma correcta, el problema suele deberse a una administración de recursos inadecuada.

En el caso de una selección de recursos incorrecta, es necesario investigar para determinar qué [recurso debe cambiar](azure-resources.md#when-to-change-a-pricing-tier).

## <a name="knowledge-bases"></a>Bases de conocimiento

Una base de conocimiento está directamente vinculada a su recurso de QnA Maker. Contiene los pares de preguntas y respuestas (PyR) que se usan para responder a las solicitudes de predicción de consultas.

### <a name="language-considerations"></a>Consideraciones de idioma

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

La primera base de conocimiento que se crea en el recurso de QnA Maker establece el idioma del recurso. Solo se puede tener un idioma para un recurso de QnA Maker.

Puede estructurar los recursos de QnA Maker por idioma o usar el [Traductor](../../translator/translator-info-overview.md) para cambiar una consulta de otro idioma al de la base de conocimiento antes de enviar la consulta al punto de conexión de predicción de consultas.

# <a name="custom-question-answering-preview-release"></a>[Respuesta personalizada a preguntas (versión preliminar)](#tab/v2)

Ahora puede tener knowledge bases en distintos idiomas dentro del mismo recurso de Text Analytics en la que esté habilitada la característica de respuesta personalizada a preguntas. Al crear la primera knowledge base, puede elegir si desea usar el recurso para las knowledge bases en un solo idioma o en varios idiomas.

![Selección de knowledge base multilingüe de QnA Maker administrado (versión preliminar)](../media/qnamaker-create-publish-knowledge-base/connect-knowledgebase-custom-qna.png)

> [!NOTE]
> Si habilita la opción de varios idiomas por knowledge base, no puede crear tantas knowledge bases en el recurso de Text Analytics. Para [más detalles sobre las limitaciones en la configuración del idioma](./azure-resources.md).

---

### <a name="ingest-data-sources"></a>Ingesta de orígenes de datos

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

Puede usar uno de los siguientes [orígenes de datos](../Concepts/data-sources-and-content.md) para crear una base de conocimiento:

* URL pública
* URL privada de SharePoint
* Archivo

# <a name="custom-question-answering-preview-release"></a>[Respuesta personalizada a preguntas (versión preliminar)](#tab/v2)

Respuesta personalizada a preguntas también admite documentos sin estructurar. Puede cargar un archivo con contenido de este tipo.

Actualmente no se admiten direcciones URL para contenido sin estructurar.

---

El proceso de ingesta convierte los [tipos de contenido admitidos](../reference-document-format-guidelines.md) a Markdown. La edición posterior de la *respuesta* se realiza con Markdown. Después de crear una base de conocimiento, puede editar los [pares de preguntas y respuestas](question-answer-set.md) en el portal de QnA Maker mediante la [creación de texto enriquecido](../how-to/edit-knowledge-base.md#rich-text-editing-for-answer).

### <a name="data-format-considerations"></a>Consideraciones sobre el formato de los datos

Dado que el formato final de un par de pregunta y respuesta es Markdown, es importante entender la [compatibilidad con Markdown](../reference-markdown-format.md).

Las imágenes vinculadas deben estar disponibles desde una dirección URL pública para que se muestren en el panel de prueba del portal de QnA Maker o en una aplicación cliente. QnA Maker no proporciona autenticación para el contenido, incluidas las imágenes.

### <a name="bot-personality"></a>Personalidad de bot

Agregue una personalidad de bot a la base de conocimiento con la función de [charla](../how-to/chit-chat-knowledge-base.md). Esta personalidad incluye respuestas en un determinado tono de conversación, como *profesional* y *amistoso*. La función de charla se proporciona como un conjunto conversacional, que puede agregar, editar y quitar como le convenga.

Se recomienda una personalidad de bot si el bot se conecta a la base de conocimiento. Puede usar la charla en la base de conocimiento incluso si también se conecta a otros servicios, pero debe revisar cómo interactúa el servicio de bot para saber si ese es el diseño de arquitectura correcto para su uso.

### <a name="conversation-flow-with-a-knowledge-base"></a>Flujo de conversación con una base de conocimiento

Normalmente, el flujo de conversación empieza con un saludo por parte de un usuario, como `Hi` o `Hello`. La base de conocimiento puede responder de manera general, como `Hi, how can I help you`, y mostrar una serie de mensajes de seguimiento para continuar con la conversación.

Debe diseñar el flujo de conversación con un bucle en mente para que el usuario sepa cómo usar el bot y que este no lo abandone durante la conversación. Los [mensajes de seguimiento](../how-to/multiturn-conversation.md) proporcionan vinculación entre los pares de preguntas y respuestas, que hacen posible el flujo de conversación.

### <a name="authoring-with-collaborators"></a>Creación con colaboradores

Los colaboradores podrían ser otros desarrolladores que comparten la pila de desarrollo completa de la aplicación de base de conocimiento, o tal vez se limiten a crear la base de conocimiento.

La creación de la base de conocimiento admite varios permisos de acceso basado en roles que se apliquen en Azure Portal para limitar el ámbito de las capacidades de un colaborador.

## <a name="integration-with-client-applications"></a>Integración con aplicaciones cliente

La integración con aplicaciones cliente se realiza enviando una consulta al punto de conexión del tiempo de ejecución de predicción. Se envía una consulta a la base de conocimiento específica con una solicitud basada en REST o en el SDK al punto de conexión de la aplicación web de QnA Maker.

Para autenticar correctamente una solicitud de cliente, la aplicación cliente debe enviar las credenciales correctas y el identificador de la base de conocimiento. Si usa Azure Bot Service, defina esta configuración como parte de la configuración del bot en Azure Portal.

### <a name="conversation-flow-in-a-client-application"></a>Flujo de conversación en una aplicación cliente

El flujo de conversación en una aplicación cliente, como un bot de Azure, puede requerir alguna funcionalidad antes y después de interactuar con la base de conocimiento.

¿La aplicación cliente admite el flujo de conversación, ya sea proporcionando medios alternativos para controlar los mensajes de seguimiento o incluyendo una charla? Si es así, diséñelos antes y asegúrese de que la consulta de la aplicación cliente la controle correctamente otro servicio o cuando se envíe a la base de conocimiento.

### <a name="dispatch-between-qna-maker-and-language-understanding-luis"></a>Envío entre QnA Maker y Language Understanding (LUIS)

Una aplicación cliente puede proporcionar varias características, pero la base de conocimiento solo responde a una de estas. Las demás características necesitan entender el texto de la conversación y extraer su significado.

Una arquitectura de aplicación cliente habitual consiste en usar juntos QnA Maker y [Language Understanding (LUIS)](../../LUIS/what-is-luis.md). LUIS ofrece la clasificación y la extracción de texto de cualquier consulta, incluso a otros servicios. QnA Maker proporciona respuestas desde la base de conocimiento.

En un escenario de [arquitectura compartida](../choose-natural-language-processing-service.md), el envío entre los dos servicios se realiza con la herramienta [Dispatch](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) de Bot Framework.

### <a name="active-learning-from-a-client-application"></a>Aprendizaje activo desde una aplicación cliente

QnA Maker usa el _aprendizaje activo_ para mejorar la base de conocimiento al sugerir preguntas alternativas a una respuesta. La aplicación cliente es responsable de una parte de este [aprendizaje activo](../How-To/use-active-learning.md). A través de los mensajes de conversación, la aplicación cliente puede determinar que la base de conocimiento ha devuelto una respuesta que no es útil para el usuario y puede determinar una respuesta mejor. La aplicación cliente debe enviar esta información de nuevo a la base de conocimiento para mejorar la calidad de la predicción.

### <a name="providing-a-default-answer"></a>Aportación de una respuesta predeterminada

Si la base de conocimiento no encuentra una respuesta, devuelve la _respuesta predeterminada_. Esta respuesta se puede configurar en la página **Configuración** del portal de QnA Maker o las [API](/rest/api/cognitiveservices/qnamaker/knowledgebase/update#request-body).

Esta respuesta predeterminada es diferente de la que ofrece el bot de Azure. La respuesta predeterminada para el bot de Azure se define en Azure Portal como parte de la configuración. Se devuelve cuando no se cumple el umbral de puntuación.

## <a name="prediction"></a>Predicción

La predicción es la respuesta de la base de conocimiento e incluye más información que simplemente la respuesta. Para obtener una respuesta de predicción de consultas, use [GeneateAnswer API](query-knowledge-base.md).

### <a name="prediction-score-fluctuations"></a>Fluctuaciones de la puntuación de predicción

Una puntuación puede cambiar en función de varios factores:

* El número de respuestas solicitadas para [GenerateAnswer](query-knowledge-base.md) con la propiedad `top`
* La variedad de preguntas alternativas disponibles
* El filtrado de metadatos
* La consulta enviada a la base de conocimiento `test` o `production`

Hay una [clasificación de las respuestas en dos fases](query-knowledge-base.md#how-qna-maker-processes-a-user-query-to-select-the-best-answer):
- Cognitive Search (primera clasificación). Establezca el número de _respuestas permitidas_ lo suficientemente alto como para que Cognitive Search devuelva las mejores respuestas y luego las pase al clasificador de QnA Maker.
- QnA Maker (segunda clasificación). Aplique la caracterización y el aprendizaje automático para determinar la mejor respuesta.

### <a name="service-updates"></a>Actualizaciones del servicio

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

Aplique las [actualizaciones del runtime más recientes](../how-to/configure-QnA-Maker-resources.md#get-the-latest-runtime-updates) para administrar automáticamente las actualizaciones de servicio.

# <a name="custom-question-answering-preview-release"></a>[Respuesta personalizada a preguntas (versión preliminar)](#tab/v2)

En Respuesta personalizada a preguntas (versión preliminar), el tiempo de ejecución lo administra el propio servicio QnA Maker. Por lo tanto, no hay actualizaciones del servicio.

---

### <a name="scaling-throughput-and-resiliency"></a>Escalado, rendimiento y resistencia

El escalado, el rendimiento y la resistencia se determinan mediante los [recursos de Azure](../how-to/set-up-qnamaker-service-azure.md), los planes de tarifa y cualquier arquitectura circundante, como [Traffic Manager](../how-to/configure-QnA-Maker-resources.md#business-continuity-with-traffic-manager).

### <a name="analytics-with-application-insights"></a>Análisis con Application Insights

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

Todas las consultas a la base de conocimiento se almacenan en Application Insights. Use nuestras [principales consultas](../how-to/get-analytics-knowledge-base.md) para entender las métricas.

# <a name="custom-question-answering-preview-release"></a>[Respuesta personalizada a preguntas (versión preliminar)](#tab/v2)

Para Respuesta personalizada a preguntas, se ofrece telemetría a través del [servicio Azure Monitor](../../../azure-monitor/index.yml). Use nuestras [principales consultas](../how-to/get-analytics-knowledge-base.md) para entender las métricas.


---

## <a name="development-lifecycle"></a>Ciclo de vida del desarrollo

El [ciclo de vida del desarrollo](development-lifecycle-knowledge-base.md) de una base de conocimiento es constante, ya que consta de la edición, la prueba y la publicación de la base de conocimiento.

### <a name="knowledge-base-development-of-qna-maker-pairs"></a>Desarrollo de pares de QnA Maker de la base de conocimiento

Los pares de preguntas y respuestas deben diseñarse y desarrollarse en función del uso de la aplicación cliente.

Cada par puede contener:
* Metadatos: se pueden filtrar al realizar consultas para permitirle etiquetar los pares de preguntas y respuestas con información adicional sobre el origen, el contenido, el formato y la finalidad de los datos.
* Mensajes de seguimiento: ayudan a determinar una ruta en la base de conocimiento para que el usuario llegue a la respuesta correcta.
* Preguntas alternativas: importantes para permitir que la búsqueda haga coincidir la respuesta con otras formas de la pregunta. Las [sugerencias de aprendizaje activo](../How-To/use-active-learning.md) se convierten en preguntas alternativas.

### <a name="devops-development"></a>Desarrollo de DevOps

El desarrollo de una knowledge base para insertarla en una canalización de DevOps requiere que la knowledge base esté aislada durante las pruebas por lotes.

Una base de conocimiento comparte el índice de Cognitive Search con todas las demás bases de conocimiento del recurso de QnA Maker. Aunque la base de conocimiento está aislada por partición, si se comparte el índice, se puede producir una diferencia en la puntuación cuando se compara con la base de conocimiento publicada.

Para tener la _misma puntuación_ en las bases de conocimiento `test` y `production`, aísle un recurso de QnA Maker en una única base de conocimiento. En esta arquitectura, el recurso solo necesita persistir tanto tiempo como la prueba por lotes aislada.

## <a name="next-steps"></a>Pasos siguientes

* [Recursos de Azure](../how-to/set-up-qnamaker-service-azure.md)
* [Pares de preguntas y respuestas](question-answer-set.md)