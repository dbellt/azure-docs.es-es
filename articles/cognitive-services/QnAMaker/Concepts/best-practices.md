---
title: 'Procedimientos recomendados: QnA Maker'
description: Siga estos procedimientos recomendados para mejorar la base de conocimiento y ofrecer mejores resultados a los usuarios finales del bot de chat o de la aplicación.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: 6b3f12881bd0cb7114f32cab5b7a67d80a73f4fe
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110372848"
---
# <a name="best-practices-of-a-qna-maker-knowledge-base"></a>Procedimientos recomendados para una base de conocimiento de QnA Maker

El artículo sobre el [ciclo de vida de desarrollo de una base de conocimiento](../Concepts/development-lifecycle-knowledge-base.md) indica cómo administrar una base de conocimiento de principio a fin. Siga estos procedimientos recomendados para mejorar la base de conocimiento y ofrecer mejores resultados a los usuarios finales del bot de chat o de la aplicación cliente.

## <a name="extraction"></a>Extracción

El servicio QnA Maker mejora continuamente los algoritmos que extraen preguntas y respuestas del contenido y amplía la lista de formatos HTML y de archivos que admite. Siga las [directrices](../Concepts/data-sources-and-content.md) para la extracción de datos según su tipo de documento.

En general, las páginas de preguntas más frecuentes deben ser independientes y no combinarse con otra información. Los manuales de productos deben tener encabezados claros y, preferiblemente, una página de índice.

### <a name="configuring-multi-turn"></a>Configuración de varios turnos

[Cree su base de conocimiento](../how-to/multiturn-conversation.md#create-a-multi-turn-conversation-from-a-documents-structure) tras habilitar la extracción multiturno. Si la base de conocimiento admite o debería admitir la jerarquía de preguntas, esta jerarquía se puede extraer del documento o crear después de extraer el documento.

## <a name="creating-good-questions-and-answers"></a>Creación de buenas preguntas y respuestas

### <a name="good-questions"></a>Buenas preguntas

Las mejores preguntas son simples. Identifique la palabra o frase clave de cada pregunta y formule una pregunta sencilla sobre ella.

Agregue tantas preguntas alternativas como necesite, pero mantenga las variantes simples. Agregar más palabras o expresiones que no forman parte del objetivo principal de la pregunta no ayuda a que QnA Maker encuentre una coincidencia.


### <a name="add-relevant-alternative-questions"></a>Agregar preguntas alternativas pertinentes

El usuario puede escribir preguntas con un estilo de conversación de texto, `How do I add a toner cartridge to my printer?`, o bien con una búsqueda por palabra clave, como `toner cartridge`. La base de conocimiento debe tener ambos estilos de preguntas para devolver correctamente la mejor respuesta. Si no está seguro de qué palabras clave está escribiendo un cliente, utilice los datos de Application Insights para analizar las consultas.

### <a name="good-answers"></a>Buenas respuestas

Las mejores respuestas son simples, pero no demasiado. No use respuestas como `yes` y `no`. Si la respuesta debe remitir a otros orígenes o incluir una experiencia enriquecida con vínculos y contenido multimedia, use el [etiquetado de metadatos](../how-to/edit-knowledge-base.md#add-metadata) para distinguir las respuestas y, a continuación, [envíe la consulta](../how-to/metadata-generateanswer-usage.md#generateanswer-request-configuration) con las etiquetas de metadatos de la propiedad `strictFilters` para obtener la versión correcta de la respuesta.

|Respuesta|Avisos de seguimiento|
|--|--|
|Apague el dispositivo Surface Laptop con el botón de encendido del teclado.|* Combinaciones de teclas para suspender, apagar y reiniciar.<br>* Cómo arrancar en frío un dispositivo Surface Laptop.<br>* Cómo cambiar el BIOS de un dispositivo Surface Laptop.<br>* Diferencias entre suspender, apagar y reiniciar.|
|El servicio de atención al cliente está disponible por vía telefónica, Skype y mensaje de texto 24 horas al día.|* Información de contacto del equipo de ventas.<br> * Ubicación de las oficinas y tiendas, así como los horarios para visitas en persona.<br> * Accesorios para dispositivos Surface Laptop.|

## <a name="chit-chat"></a>Charla
Agregue una charla a su bot para que sea más conversacional y atractivo fácilmente. Puede agregar fácilmente conjuntos de datos de charla de personalidades predefinidas al crear la base de conocimiento y cambiarlos en cualquier momento. Obtenga información sobre cómo [agregar una charla a la base de conocimiento](../How-To/chit-chat-knowledge-base.md).

La charla es compatible con [muchos idiomas](../how-to/chit-chat-knowledge-base.md#language-support).

### <a name="choosing-a-personality"></a>Selección de una personalidad
La charla es compatible con varias personalidades predefinidas:

|Personalidad |Archivo de conjunto de datos de QnA Maker |
|---------|-----|
|Profesional |[qna_chitchat_professional.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_professional.tsv) |
|Descriptiva |[qna_chitchat_friendly.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_friendly.tsv) |
|Ingenioso |[qna_chitchat_witty.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_witty.tsv) |
|Cariñoso |[qna_chitchat_caring.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_caring.tsv) |
|Entusiasta |[qna_chitchat_enthusiastic.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_enthusiastic.tsv) |

El rango de respuestas va de formal a informal e irreverente. Debe seleccionar la personalidad que sea más acorde con el tono que quiera para el bot. Puede ver los [conjuntos de datos](https://github.com/Microsoft/BotBuilder-PersonalityChat/tree/master/CSharp/Datasets), elegir uno que sirva como base para el bot y, a continuación, personalizar las respuestas.

### <a name="edit-bot-specific-questions"></a>Edición de preguntas específicas de bot
Hay algunas preguntas específicas de bot que forman parte del conjunto de datos de charla y que se han rellenado con respuestas genéricas. Cambie estas respuestas para reflejar mejor los detalles del bot.

Se recomienda hacer que las siguientes preguntas y respuestas de charla sean más específicas:

* ¿Quién es usted?
* ¿Qué puede hacer?
* ¿Cuántos años tiene?
* ¿Quién le ha creado?
* Hola

### <a name="adding-custom-chit-chat-with-a-metadata-tag"></a>Adición de una charla personalizado con una etiqueta de metadatos

Si agrega sus propios pares de QnA de charla, asegúrese de agregar metadatos para que se devuelvan estas respuestas. El par nombre-valor de metadatos es `editorial:chitchat`.

## <a name="searching-for-answers"></a>Búsqueda de respuestas

GenerateAnswer API usa tanto las preguntas como la respuesta para buscar las mejores respuestas a la consulta de un usuario.

### <a name="searching-questions-only-when-answer-is-not-relevant"></a>Búsqueda de preguntas solo cuando la respuesta no es pertinente

Use [`RankerType=QuestionOnly`](#choosing-ranker-type) si no quiere buscar respuestas.

Un ejemplo es una base de conocimiento que es un catálogo con acrónimos como preguntas y su forma completa como respuesta. El valor de la respuesta no ayudará a buscar la respuesta adecuada.

## <a name="rankingscoring"></a>Clasificación y puntuación
Asegúrese de aprovechar al máximo las características de clasificación que QnA Maker admite. Eso aumentará las probabilidades de que una determinada consulta de usuario se responda con una respuesta correcta.

### <a name="choosing-a-threshold"></a>Selección de un umbral

La [puntuación de confianza](confidence-score.md) predeterminada que se usa como umbral es 0, pero puede [cambiar el umbral](confidence-score.md#set-threshold) de la base de conocimiento según sus necesidades. Dado que cada base de conocimiento es diferente, conviene probar y elegir el umbral que mejor se adapte a cada caso.

### <a name="choosing-ranker-type"></a>Selección del tipo de clasificador
De manera predeterminada, QnA Maker busca a través de preguntas y respuestas. Si quiere buscar solo en las preguntas, para generar una respuesta, utilice el objeto `RankerType=QuestionOnly` del cuerpo de POST de la solicitud GenerateAnswer.

### <a name="add-alternate-questions"></a>Agregar preguntas alternativas
[Alternar las preguntas](../How-To/edit-knowledge-base.md) aumenta las probabilidades de encontrar una coincidencia con una consulta de usuario. Las preguntas alternativas son útiles cuando hay varias maneras de plantear la misma pregunta. Esto puede incluir cambios en la estructura de la frase y el estilo de redacción.

|Consulta original|Consultas alternativas|Change|
|--|--|--|
|¿Hay estacionamiento disponible?|¿Dispone de estacionamiento?|estructura de la frase|
 |Hola|Buenas<br>¿Cómo estamos?|estilo de redacción o jerga|

<a name="use-metadata-filters"></a>

### <a name="use-metadata-tags-to-filter-questions-and-answers"></a>Uso de etiquetas de metadatos para el filtrado de preguntas y respuestas

Los [metadatos](../How-To/edit-knowledge-base.md) agregan la posibilidad de que una aplicación cliente pueda saber que no debe tomar todas las respuestas, sino restringir los resultados de una consulta de usuario en función de las etiquetas de metadatos. La respuesta de la base de conocimiento puede diferir según la etiqueta de metadatos, aunque la consulta sea la misma. Por ejemplo, *¿Dónde está ubicado el estacionamiento?* puede tener otra respuesta si la ubicación del restaurante es diferente; es decir, los metadatos son *ubicación: Seattle* frente a *ubicación: Redmond*.

### <a name="use-synonyms"></a>Usar sinónimos
# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)
Aunque hay cierta compatibilidad con los sinónimos en inglés, use alteraciones de palabras que no distinguen mayúsculas de minúsculas a través de [Alterations API](/rest/api/cognitiveservices/qnamaker/alterations/replace) para agregar sinónimos a las palabras clave que tengan una forma diferente. Los sinónimos se agregan en el nivel del servicio QnA Maker y **todas las bases de conocimiento del servicio los comparten**.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)
Aunque hay cierta compatibilidad con los sinónimos en inglés, use alteraciones de palabras que no distinguen mayúsculas de minúsculas a través de [Alterations API](/rest/api/cognitiveservices/qnamaker/alterations/replace) para agregar sinónimos a las palabras clave que tengan una forma diferente. Los sinónimos en la característica de respuesta a preguntas personalizada (versión preliminar) se **agregan por knowledge base**.

|Palabra original|Sinónimos|
|--|--|
|comprar|compra<br>banca online<br>banca electrónica|

---

### <a name="use-distinct-words-to-differentiate-questions"></a>Usar diferentes palabras para diferenciar preguntas
Los algoritmos de clasificación de QnA Maker que buscan la correspondencia entre la consulta de un usuario y una pregunta de la base de conocimiento funcionan mejor si cada pregunta aborda una necesidad distinta. La repetición del mismo conjunto de palabras en diferentes preguntas reduce la probabilidad de que se seleccione la respuesta correcta para una consulta de usuario determinado con esas palabras.

Por ejemplo, puede tener dos preguntas y respuestas independientes con las siguientes preguntas:

|Preguntas y respuestas|
|--|
|dónde está *ubicado* el estacionamiento|
|dónde está *ubicado* el cajero automático|

Puesto que estas dos preguntas y respuestas están expresadas con palabras muy similares, esta similitud podría provocar que las puntuaciones sean muy parecidas para muchas de las consultas de usuario que están expresadas como *dónde está ubicado `<x>`* . Como alternativa, intente diferenciarlas claramente con consultas como *"dónde está el estacionamiento"* y *"dónde está el cajero automático"* , evitando palabras como "ubicado", que pueden estar en muchas preguntas en la base de conocimiento.

## <a name="collaborate"></a>Colaborar
QnA Maker permite a los usuarios colaborar en una base de conocimiento. Los usuarios necesitan acceso al grupo de recursos de QnA Maker en Azure para tener acceso a las bases de conocimiento. Algunas organizaciones quizás quieran externalizar la edición y el mantenimiento de la base de conocimiento, sin dejar de proteger el acceso a sus recursos de Azure. Para realizar este modelo de editor-aprobador, se configuran dos [servicios QnA Maker](../How-to/set-up-qnamaker-service-azure.md) idénticos en suscripciones distintas y seleccionar uno para el ciclo de pruebas de edición. Una vez finalizada la prueba, se transfiere el contenido de la base de conocimiento con un proceso de [importación y exportación](../Tutorials/migrate-knowledge-base.md) al servicio QnA Maker del aprobador que, finalmente, publicará la base de conocimiento y actualizará el punto de conexión.



## <a name="active-learning"></a>Aprendizaje activo

[Aprendizaje activo](../How-to/use-active-learning.md) ofrece el mejor resultado al sugerir preguntas alternativas cuando tiene una amplia variedad de calidad y cantidad de consultas basadas en el usuario. Es importante permitir que las consultas de usuario de las aplicaciones cliente participen en el bucle de comentarios de aprendizaje activo sin censura. Una vez que se sugieren preguntas en el portal de QnA Maker, puede **[filtrar por sugerencias](../How-To/improve-knowledge-base.md)** y, a continuación, revisar y aceptar las sugerencias o rechazarlas.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Editar una base de conocimiento](../How-to/edit-knowledge-base.md)
