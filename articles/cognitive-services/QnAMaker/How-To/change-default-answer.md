---
title: 'Obtención de la respuesta predeterminada: QnA Maker'
description: Se devuelve la respuesta predeterminada cuando no hay ninguna coincidencia con la pregunta. Puede que desee cambiar la respuesta predeterminada de la respuesta predeterminada estándar.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: how-to
ms.date: 11/09/2020
ms.openlocfilehash: abc33a405975db9484d204c51df0a67697c2f8f8
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370811"
---
# <a name="change-default-answer-for-a-qna-maker-resource"></a>Cambiar la respuesta predeterminada para un recurso de QnA Maker

La respuesta predeterminada para una base de conocimiento está pensada para que se devuelva cuando no se encuentra una respuesta. Si usa una aplicación cliente, como [Azure Bot Service](/azure/bot-service/bot-builder-howto-qna), es posible que también tenga una respuesta predeterminada independiente, lo que indica que ninguna respuesta ha alcanzado el umbral de puntuación.

## <a name="types-of-default-answer"></a>Tipos de respuesta predeterminada

Hay dos tipos de respuesta predeterminada en la knowledge base. Es importante entender cómo y cuándo se devuelve cada una desde una consulta de predicción:

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

|Tipos de respuestas predeterminadas|Descripción de respuesta|
|--|--|
|Respuesta de KB si no se ha determinado ninguna respuesta|`No good match found in KB.` - Si la [API GenerateAnswer](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer) no encuentra ninguna respuesta coincidente para la pregunta, se devuelve el valor `DefaultAnswer` de App Service. Todas las knowledge bases del mismo recurso de QnA Maker comparten el mismo texto de respuesta predeterminado.<br>Puede administrar la configuración en Azure Portal por medio de App Service o con las API de REST para [obtener](/rest/api/appservice/webapps/listapplicationsettings) o [actualizar](/rest/api/appservice/webapps/updateapplicationsettings) la configuración.|
|Texto de instrucciones de solicitud de seguimiento|Al usar una solicitud de seguimiento en un flujo de conversación, es posible que no necesite una respuesta en el par de pregunta y respuesta porque quiere que el usuario seleccione en las solicitudes de seguimiento. En este caso, establezca un texto concreto al determinar el texto de respuesta predeterminado, que se devuelve con cada predicción de las solicitudes de seguimiento. El texto tiene que mostrarse como texto de instrucciones para la selección de solicitudes de seguimiento. Un ejemplo de este texto de respuesta predeterminado es `Please select from the following choices`. Esta configuración se explica en las siguientes secciones de este documento. También se puede establecer como parte de la definición de knowledge base de `defaultAnswerUsedForExtraction` mediante la [API de REST](/rest/api/cognitiveservices/qnamaker/knowledgebase/create).|

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

|Tipos de respuestas predeterminadas|Descripción de respuesta|
|--|--|
|Respuesta de KB si no se ha determinado ninguna respuesta|`No good match found in KB.`: si [GenerateAnswer API](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer) no encuentra ninguna respuesta coincidente para la pregunta, muestra una respuesta de texto predeterminada. En Respuesta a preguntas personalizada (versión preliminar), este texto se puede establecer en la **Configuración** de la knowledge base. <br><br> ![Respuesta predeterminada establecida de QnA Maker administrado (versión preliminar)](../media/qnamaker-how-change-default-answer/qnamaker-v2-change-default-answer.png)|
|Texto de instrucciones de solicitud de seguimiento|Al usar una solicitud de seguimiento en un flujo de conversación, es posible que no necesite una respuesta en el par de pregunta y respuesta porque quiere que el usuario seleccione en las solicitudes de seguimiento. En este caso, establezca un texto concreto al determinar el texto de respuesta predeterminado, que se devuelve con cada predicción de las solicitudes de seguimiento. El texto tiene que mostrarse como texto de instrucciones para la selección de solicitudes de seguimiento. Un ejemplo de este texto de respuesta predeterminado es `Please select from the following choices`. Esta configuración se explica en las siguientes secciones de este documento. También se puede establecer como parte de una definición de knowledge base si `defaultAnswerUsedForExtraction` usa la [API REST](/rest/api/cognitiveservices/qnamaker/knowledgebase/create).|

---

### <a name="client-application-integration"></a>Integración de aplicación cliente

En el caso de una aplicación cliente, como un bot con **Azure Bot Service**, puede elegir entre los siguientes escenarios comunes:

* Usar la configuración de la knowledge base
* Usar otro texto en la aplicación cliente para distinguir cuándo se devuelve una respuesta, pero que no cumple el umbral de puntuación. Este texto puede ser texto estático almacenado en código, o se puede almacenar en la lista de configuración de la aplicación cliente.

## <a name="set-follow-up-prompts-default-answer-when-you-create-knowledge-base"></a>Establecimiento de la respuesta predeterminada de la solicitud de seguimiento al crear la knowledge base

Al crear una nueva base de conocimiento, el texto de respuesta predeterminado es uno de los valores de configuración. Si decide no establecerlo durante el proceso de creación, puede cambiarlo más adelante con el siguiente procedimiento.

## <a name="change-follow-up-prompts-default-answer-in-qna-maker-portal"></a>Cambio de la respuesta predeterminada de la solicitud de seguimiento en el portal de QnA Maker

La respuesta predeterminada de la base de conocimiento se devuelve cuando no se devuelve ninguna respuesta del servicio QnA Maker.

1. Inicie sesión en el [portal de QnA Maker](https://www.qnamaker.ai/) y seleccione la base de conocimiento en la lista.
1. Seleccione **Configuración** en la barra de navegación.
1. Cambie el valor de **Default answer text** (Texto de respuesta predeterminado) en la sección **Manage knowledge base** (Administrar base de conocimiento).

    :::image type="content" source="../media/qnamaker-concepts-confidencescore/change-default-answer.png" alt-text="Captura de pantalla del portal de QnA Maker, página de configuración, con el cuadro de texto respuesta predeterminado resaltado.":::

1. Seleccione **Guardar y entrenar** para almacenar los cambios.

## <a name="next-steps"></a>Pasos siguientes

* [Creación de una base de conocimientos](../How-to/manage-knowledge-bases.md)
