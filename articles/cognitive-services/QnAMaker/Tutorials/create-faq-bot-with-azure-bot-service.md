---
title: 'Tutorial: Creación de un bot de preguntas frecuentes con Azure Bot Service'
description: En este tutorial se crea un bot de preguntas frecuentes sin código con QnA Maker y Azure Bot Service.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: tutorial
ms.date: 08/31/2020
ms.openlocfilehash: 59d24ebd16d25658d75cb36509183233d2f2e500
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111412158"
---
# <a name="tutorial-create-an-faq-bot-with-azure-bot-service"></a>Tutorial: Creación de un bot de preguntas frecuentes con Azure Bot Service
Cree un bot de preguntas frecuentes con QnA Maker y Azure [Bot Service](https://azure.microsoft.com/services/bot-service/) sin código.

En este tutorial, aprenderá a:

<!-- green checkmark -->
> [!div class="checklist"]
> * Vincular una base de conocimiento de QnA Maker a una instancia de Azure Bot Service
> * Implementar el bot
> * Probar el bot en Chat en web
> * Destacar el bot en los canales admitidos

## <a name="create-and-publish-a-knowledge-base"></a>Crear y publicar una base de conocimiento

Siga el [inicio rápido](../Quickstarts/create-publish-knowledge-base.md) para crear una base de conocimiento. Una vez que la base de conocimiento se haya publicado correctamente, llegará a la página siguiente.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

> [!div class="mx-imgBorder"]
> ![Captura de pantalla de una publicación correcta](../media/qnamaker-create-publish-knowledge-base/publish-knowledge-base-to-endpoint.png)

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

> [!div class="mx-imgBorder"]
> ![Captura de pantalla de una publicación correcta administrada](../media/qnamaker-create-publish-knowledge-base/publish-knowledge-base-to-endpoint-managed.png)

---

## <a name="create-a-bot"></a>Creación de un bot

Después de la publicación, puede crear un bot en la página **Publicar**:

* Puede crear varios bots rápidamente, todos ellos apuntando a la misma base de conocimiento para diferentes regiones o planes de precios para los bots individuales.
* Si desea solo un bot para la base de conocimiento, utilice el vínculo **View all your bots on the Azure portal** (Ver todos los bots en Azure Portal) para ver una lista de los bots actuales.

Si realiza cambios en la base de conocimiento y vuelve a publicarla, no es necesario realizar ninguna acción más con el bot. Ya está configurado para funcionar con la base de conocimiento y funciona con todos los cambios futuros de la base de conocimiento. Cada vez que publique una base de conocimiento, todos los bots conectados a ella se actualizarán automáticamente.

1. En el portal de QnA Maker, en la página **Publish** (Publicar), seleccione **Create bot** (Crear bot). Este botón solo aparece cuando se publica la base de conocimiento.

    # <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de creación de un bot](../media/qnamaker-create-publish-knowledge-base/create-bot-from-published-knowledge-base-page.png)

    # <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de creación de la versión preliminar administrada de un bot](../media/qnamaker-create-publish-knowledge-base/create-bot-from-published-knowledge-base-page-managed.png)

    ---
    

1. Se abre una nueva pestaña del explorador para Azure Portal, con la página de creación de Azure Bot Service. Configure Azure Bot Service. El bot y QnA Maker pueden compartir el plan del servicio de aplicación web pero no se puede compartir la aplicación web. Esto significa que el **nombre de aplicación** para el bot debe ser diferente del nombre de aplicación para el servicio QnA Maker.

    * **Sí**
        * Cambie el identificador del bot si no es único.
        * Seleccione el lenguaje del SDK. Una vez creado el bot, puede descargar el código en el entorno de desarrollo local y continuar el proceso de desarrollo.
    * **No**
        * Cambie las siguientes opciones en Azure Portal al crear el bot. Se rellenan automáticamente para la base de conocimiento existente:
           * Clave de autenticación de QnA
           * Plan de App Service y ubicación


1. Una vez creado el bot, abra el recurso **Servicio de bots**.
1. En **Administración del bot**, seleccione **Probar en Chat en web**.
1. En el aviso de chat **Escriba su mensaje**, escriba:

    `Azure services?`

    El bot de chat muestra una respuesta de la base de conocimiento.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de un bot que devuelve una respuesta](../media/qnamaker-create-publish-knowledge-base/test-web-chat.png)


1. Destacar el bot en los [canales admitidos](/azure/bot-service/bot-service-manage-channels) adicionales.
    
## <a name="integrate-the-bot-with-channels"></a>Integración del bot con canales

Haga clic en **Canales** en el recurso de Bot Service que ha creado. Puede activar el bot en otros [canales admitidos](/azure/bot-service/bot-service-manage-channels).

   >[!div class="mx-imgBorder"]
   >![Captura de pantalla de la integración con Teams](../media/qnamaker-tutorial-updates/connect-with-teams.png)

