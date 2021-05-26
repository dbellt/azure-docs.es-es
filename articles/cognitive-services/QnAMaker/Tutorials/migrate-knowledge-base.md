---
title: Migración de bases de conocimiento (QnA Maker)
description: La migración de una base de conocimiento requiere la exportación de una base de conocimiento y la posterior importación a otra.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: how-to
ms.date: 11/09/2020
ms.openlocfilehash: 8e1cff5a82870c5491b262f5ced95f688d5ee4ac
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110369516"
---
# <a name="migrate-a-knowledge-base-using-export-import"></a>Migrar una base de conocimiento mediante la exportación e importación

La migración es el proceso de creación de una base de conocimiento a partir de una base de conocimiento existente. Hay varios motivos para hacerlo:

* proceso de copia de seguridad y restauración
* Canalización de integración y entrega continuas
* traslado de regiones

La migración de una base de conocimiento requiere la exportación de una base de conocimiento existente y su posterior importación a otra.

> [!NOTE]
> Siga las instrucciones que se indican a continuación para migrar una knowledge base existente a un servicio QnA Maker administrado (versión preliminar) nuevo.

## <a name="prerequisites"></a>Requisitos previos

> * Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

> * Un [recurso de QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) creado en Azure Portal. Recuerde el identificador de Azure Active Directory, la suscripción y el nombre de recurso de QnA que seleccionó al crear el recurso.
> * Configurar un [servicio QnA Maker](../How-To/set-up-qnamaker-service-azure.md) nuevo

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

> * Un [recurso de Text Analytics](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) con la característica de respuesta a preguntas personalizada habilitada en Azure Portal. Recuerde el identificador de Azure Active Directory, la suscripción y el nombre del recurso de Text Analytics que seleccionó al crear el recurso.
> * Configurar un [servicio QnA Maker](../How-To/set-up-qnamaker-service-azure.md) nuevo

---

## <a name="migrate-a-knowledge-base-from-qna-maker"></a>Migración de una base de conocimiento desde QnA Maker
1. Inicie sesión en el [portal de QnA Maker](https://qnamaker.ai).
1. Seleccione la base de conocimiento que desea migrar.

1. En la página **Settings** (Configuración), seleccione **Export knowledge base** (Exportar base de conocimiento) para descargar un archivo .tsv que incluya el contenido de la base de conocimiento de origen (preguntas, respuestas, metadatos, avisos de seguimiento) y los nombres del origen de datos del que se han extraído. Los identificadores de QnA que se exportan con las preguntas y respuestas se pueden usar para actualizar un par de pregunta y respuesta concreto mediante la [API de actualización](/rest/api/cognitiveservices/qnamaker/knowledgebase/update). El identificador de QnA de un par de pregunta y respuesta concreto permanece sin cambios en varias operaciones de exportación.

1. Seleccione **Create a knowledge base** (Crear una base de conocimiento) en el menú superior y cree una base de conocimiento _vacía_. Está vacía porque, cuando se crea, no se agregan direcciones URL ni archivos, sino que lo hacen durante el paso de importación, después de la creación.

    Configure la base de conocimiento. Establezca solo el nombre de la nueva base de conocimiento. Se admiten nombres duplicados y también caracteres especiales.

    No seleccione nada en el paso 4, ya que esos valores se sobrescribirán al importar el archivo.

1. En el paso 5, seleccione **Create** (Crear).

1. En esta base de conocimiento nueva, abra la pestaña **Settings** (Configuración) y seleccione **Import knowledge base** (Importar base de conocimiento). Esta acción importa las preguntas, las respuestas, los metadatos y los avisos de seguimiento, y conserva los nombres del origen de datos del que se extrajeron. **Los pares de pregunta y respuesta creados en la knowledge base nueva tendrán el mismo identificador de QnA ID que el que hay en el archivo exportado**. Esto le sirve de ayuda a la hora de crear una réplica exacta de la knowledge base.

   > [!div class="mx-imgBorder"]
   > [![Importación de la base de conocimiento](../media/qnamaker-how-to-migrate-kb/Import.png)](../media/qnamaker-how-to-migrate-kb/Import.png#lightbox)

1. **Pruebe** la base de conocimiento nueva mediante el panel de pruebas. Vea cómo [probar la base de conocimiento](../How-To/test-knowledge-base.md).

1. **Publique** la base de conocimiento y cree un bot de chat. Vea cómo [publicar la base de conocimiento](../Quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base).

## <a name="programmatically-migrate-a-knowledge-base-from-qna-maker"></a>Migración de una base de conocimiento desde QnA Maker mediante programación

El proceso de migración está disponible mediante programación con las siguientes API REST:

**Exportarar**

* [Descarga de la API de la base de conocimiento](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase/download)

**Importar**

* [API Replace (volver a cargar con el mismo identificador de base de conocimiento)](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase/replace)
* [API Create (cargar con un identificador de base de conocimiento nuevo)](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase/create)


## <a name="chat-logs-and-alterations"></a>Modificaciones y registros de chat
Las alteraciones (sinónimos) que no distinguen mayúsculas de minúsculas no se importan de forma automática. Use las [API V4](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase) para mover las modificaciones en la nueva base de conocimiento.

No hay forma de migrar registros de chat, ya que la base de conocimiento nueva usa Application Insights para almacenar registros de chat.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Editar una base de conocimiento](../How-To/edit-knowledge-base.md)