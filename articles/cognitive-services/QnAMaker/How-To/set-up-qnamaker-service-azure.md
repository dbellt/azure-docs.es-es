---
title: 'Configuración de un servicio QnA Maker: QnA Maker'
description: Antes de crear alguna base de conocimiento de QnA Maker, primero debe configurar un servicio QnA Maker en Azure. Cualquiera que tenga autorización para crear recursos en una suscripción puede configurar un servicio QnA Maker.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: a5bcc536a51da13934fb98f383fb1812d92a0073
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111964356"
---
# <a name="manage-qna-maker-resources"></a>Administración de recursos de QnA Maker

Antes de crear alguna base de conocimiento de QnA Maker, primero debe configurar un servicio QnA Maker en Azure. Cualquiera que tenga autorización para crear recursos en una suscripción puede configurar un servicio QnA Maker. Si está probando la característica de respuesta a preguntas personalizada, deberá crear el recurso de Text Analytics y agregar esta característica.

Antes de crear el recurso, es útil comprender perfectamente los conceptos siguientes:

* [Recursos de QnA Maker](../Concepts/azure-resources.md)
* [Creación y publicación de claves](../Concepts/azure-resources.md#keys-in-qna-maker)

## <a name="create-a-new-qna-maker-service"></a>Creación de un servicio QnA Maker

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

En este procedimiento se crean los recursos de Azure necesarios para administrar el contenido de la base de conocimiento. Después de completar estos pasos, encontrará las claves de _suscripción_ en la página **Claves** del recurso en Azure Portal.

1. Inicie sesión en Azure Portal y [cree un recurso de QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker).

1. Después de leer los términos y condiciones, seleccione **Crear**:

    ![Creación de un servicio QnA Maker](../media/qnamaker-how-to-setup-service/create-new-resource-button.png)

1. En **QnA Maker**, seleccione las regiones y los niveles apropiados:

    ![Creación de un nuevo servicio QnA Maker: plan de tarifa y regiones](../media/qnamaker-how-to-setup-service/enter-qnamaker-info.png)

    * En el campo **Nombre**, escriba un nombre único que identifique este servicio QnA Maker. Este nombre también identifica el punto de conexión de QnA Maker con el que se asociarán las bases de conocimiento.
    * En **Suscripción**, elija la suscripción en la que se implementará el recurso de QnA Maker.
    * Seleccione el **plan de tarifa** para los servicios de administración de QnA Maker (API de portal y administración). Vea [más detalles sobre los precios de las SKU](https://aka.ms/qnamaker-pricing).
    * Cree un **grupo de recursos** (se recomienda) para implementar este recurso de QnA Maker, o use uno existente. QnA Maker crea varios recursos de Azure. Cuando se crea un grupo de recursos que contenga estos recursos, puede encontrar, administrar y eliminar fácilmente estos recursos por su nombre.
    * Seleccione una **ubicación del grupo de recursos**.
    * Elija el **Plan de tarifa de búsqueda** del servicio Azure Cognitive Search. Si la opción de nivel Gratis no está disponible (aparece atenuada), significa que ya tiene un servicio gratuito implementado a través de la suscripción. En ese caso, tendrá que comenzar con el nivel Básico. Consulte los [detalles de precios de Azure Cognitive Search](https://azure.microsoft.com/pricing/details/search/).
    * Elija la **Ubicación de búsqueda** en la que desea que se implementen los índices de Azure Cognitive Search. Las restricciones sobre dónde deben almacenarse los datos de los clientes ayudarán a determinar la ubicación elegida para Azure Cognitive Search.
    * En el campo **Nombre de aplicación**, escriba el nombre de la instancia de Azure App Service.
    * De forma predeterminada, el nivel predeterminado de App Service es Estándar (S1). Puede cambiar el plan después de la creación. Más información sobre los [precios de App Service](https://azure.microsoft.com/pricing/details/app-service/).
    * Elija la **ubicación de sitio web** en la que se implementará App Service.

        > [!NOTE]
        > La **ubicación de la búsqueda** puede ser diferente de la **ubicación del sitio web**.

    * Elija si desea habilitar **Application Insights**. Si **Application Insights** está habilitado, QnA Maker recopila telemetría de tráfico, registros de chat y errores.
    * Elija la **ubicación de App Insights** en la que se implementará el recurso de Application Insights.
    * Como medidas de ahorro de los costos, puede [compartir](configure-QnA-Maker-resources.md#configure-qna-maker-to-use-different-cognitive-search-resource) algunos pero no todos los recursos de Azure creados para QnA Maker.

1. Una vez que se hayan validado todos los campos, seleccione **Crear**. El proceso puede tardar unos minutos en completarse.

1. Después de que se complete la implementación, verá que los siguientes recursos se crean en la suscripción:

   ![Recurso creado de un nuevo servicio QnA Maker](../media/qnamaker-how-to-setup-service/resources-created.png)

    El recurso con el tipo _Cognitive Services_ tiene las claves de la _suscripción_.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

En este procedimiento se crean los recursos de Azure necesarios para administrar el contenido de la base de conocimiento. Después de completar estos pasos, encontrará las claves de *suscripción* en la página **Claves** del recurso en Azure Portal.

1.  Inicie sesión en Azure Portal y [cree un recurso de Text Analytics](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics).

2.  Seleccione la característica de respuesta a preguntas personalizada para agregarla al recurso de Text Analytics. Haga clic en **Continue to create your resource** (Continuar para crear el recurso).

> [!div class="mx-imgBorder"]
> ![Adición de preguntas y respuestas a TA](../media/qnamaker-how-to-setup-service/select-qna-feature-create-flow.png)

3.  Seleccione los niveles y las regiones adecuados para el recurso de Text Analytics. Para la característica de respuesta a preguntas personalizada, seleccione la ubicación de búsqueda y el plan de tarifa.

> [!div class="mx-imgBorder"]
> ![Creación de un servicio de TA: plan de tarifa y regiones](../media/qnamaker-how-to-setup-service/custom-qna-create-button.png)

   * En **Suscripción**, elija la suscripción en la que se implementará el recurso de Text Analytics.
   * Cree un **grupo de recursos** (se recomienda) o use uno existente en el que implementar este recurso de Text Analytics. La habilitación de la respuesta a preguntas personalizada con un recurso de Text Analytics crea menos recursos de Azure. Cuando se crea un grupo de recursos que contenga estos recursos, puede encontrar, administrar y eliminar fácilmente estos recursos por su nombre.
   * En el campo **Nombre**, escriba un nombre único que identifique este recurso de Text Analytics. 
   * Elija la **ubicación** donde quiere que se implemente el recurso de Text Analytics. Tanto las API de administración como el punto de conexión de servicio se hospedarán en esta ubicación. 
   * Seleccione el **plan de tarifa** del servicio de Text Analytics. Vea [más detalles sobre los precios de las SKU](https://aka.ms/qnamaker-pricing).
   * Elija la **Ubicación de búsqueda** en la que desea que se implementen los índices de Azure Cognitive Search. Las restricciones sobre dónde deben almacenarse los datos de los clientes ayudarán a determinar la ubicación elegida para Azure Cognitive Search.
   * Elija el **Plan de tarifa de búsqueda** del servicio Azure Cognitive Search. Si la opción de nivel Gratis no está disponible (aparece atenuada), significa que ya tiene un servicio gratuito implementado a través de la suscripción. En ese caso, tendrá que comenzar con el nivel Básico. Consulte los [detalles de precios de Azure Cognitive Search](https://azure.microsoft.com/pricing/details/search/).

4.  Una vez que se hayan validado todos los campos, seleccione **Revisar y crear**. El proceso puede tardar unos minutos en completarse.

> [!div class="mx-imgBorder"]
> ![Revisión del recurso de TA](../media/qnamaker-how-to-setup-service/custom-qna-review-resource.png)

5.  Después de que se complete la implementación, verá que los siguientes recursos se crean en la suscripción:

> [!div class="mx-imgBorder"]
> ![El recurso ha creado un servicio QnA Maker administrado (versión preliminar)](../media/qnamaker-how-to-setup-service/resources-created-question-answering.png)

   El recurso con el tipo _Cognitive Services_ tiene las claves de la _suscripción_.

---

## <a name="upgrade-azure-resources"></a>Actualización de recursos de Azure

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

### <a name="upgrade-qna-maker-sku"></a>Actualización de la SKU de QnA Maker

Cuando desee tener más preguntas y respuestas en su base de conocimiento, más allá de su plan actual, actualice el plan de tarifa del servicio de QnA Maker.

Para actualizar la SKU de administración de QnA Maker:

1. Vaya al recurso QnA Maker en Azure Portal y seleccione **Plan de tarifa**.

    ![Recurso QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-resource.png)

1. Elija la SKU adecuada y presione **Seleccionar**.

    ![Precios de QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-pricing-page.png)
    
### <a name="upgrade-app-service"></a>Actualización de App Service

Cuando la base de conocimiento necesite atender más solicitudes de la aplicación cliente, actualice el plan de tarifa de App Service.

Puede [escalar verticalmente](../../../app-service/manage-scale-up.md) o escalar horizontalmente App Service.

Vaya al recurso App Service en Azure Portal y seleccione la opción **Escalar verticalmente** o **Escalar horizontalmente** según proceda.

![Escalado de App Service en QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-appservice-scale.png)

### <a name="upgrade-the-azure-cognitive-search-service"></a>Actualización del servicio Azure Cognitive Search

Si planea tener muchas bases de conocimiento, actualice el plan de tarifa del servicio Azure Cognitive Search.

Actualmente, no puede realizar una actualización local de la SKU de Azure Search. Sin embargo, puede crear un recurso de Azure Search con la SKU deseada, restaurar los datos al nuevo recurso y luego vincularlo a la pila de QnA Maker. Para ello, siga estos pasos.

1. Cree un recurso de Azure Search en Azure Portal y elija la SKU que desee.

    ![Recurso Azure Search en QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-azuresearch-new.png)

1. Restaure los índices a partir del recurso Azure Search original en uno nuevo. Vea el [código de ejemplo de restauración de copia de seguridad](https://github.com/pchoudhari/QnAMakerBackupRestore).

1. Después de que se hayan restaurado los datos, vaya al nuevo recurso de Azure Search, seleccione **Keys** (Claves) y anote el **nombre** y la **clave de administrador**:

    ![Claves de Azure Search en QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-azuresearch-keys.png)

1. Para vincular el nuevo recurso de Azure Search a la pila de QnA Maker, vaya a la instancia de App Service en QnA Maker.

    ![Instancia de App Service en QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-resource-list-appservice.png)

1. Seleccione **Configuración de la aplicación**  y modifique la configuración en los campos **AzureSearchName** y **AzureSearchAdminKey** del paso 3.

    ![Valor de App Service en QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-appservice-settings.png)

1. Reinicie la instancia de App Service.

    ![Reinicie la instancia de App Service en QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-appservice-restart.png)
    
### <a name="inactivity-policy-for-free-search-resources"></a>Directiva de inactividad para recursos de búsqueda gratuitos

Si no va a usar un recurso de QnA Maker, debe quitar todos los recursos. Si no quita los recursos no utilizados, la base de conocimiento dejará de funcionar si ha creado un recurso de búsqueda gratuito.

Los recursos de búsqueda gratuitos se eliminan al cabo de 90 días sin recibir una llamada API.
    
# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

### <a name="upgrade-the-azure-cognitive-search-service"></a>Actualización del servicio Azure Cognitive Search

Si planea tener muchas bases de conocimiento, actualice el plan de tarifa del servicio Azure Cognitive Search.

Actualmente, no puede realizar una actualización local de la SKU de Azure Search. Sin embargo, puede crear un recurso de búsqueda de Azure con la SKU deseada, restaurar los datos en el nuevo recurso y, luego, vincularlo a la pila de la respuesta a preguntas personalizada. Para ello, siga estos pasos.

1. Cree un recurso de Azure Search en Azure Portal y elija la SKU que desee.

    ![Recurso Azure Search en QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-azuresearch-new.png)

1. Restaure los índices a partir del recurso Azure Search original en uno nuevo. Vea el [código de ejemplo de restauración de copia de seguridad](https://github.com/pchoudhari/QnAMakerBackupRestore).

1. Vincule el nuevo recurso de búsqueda de Azure a la característica de respuesta a preguntas personalizada en la [pestaña de características del recurso de Text Analytics](./configure-qna-maker-resources.md?tabs=v2#configure-text-analytics-resource-with-custom-question-answering-feature-to-use-a-different-cognitive-search-resource).

### <a name="inactivity-policy-for-free-search-resources"></a>Directiva de inactividad para recursos de búsqueda gratuitos

Si no va a usar un recurso de QnA Maker o de Text Analytics, debe quitar todos los recursos. Si no quita los recursos no utilizados, la base de conocimiento dejará de funcionar si ha creado un recurso de búsqueda gratuito.

Los recursos de búsqueda gratuitos se eliminan al cabo de 90 días sin recibir una llamada API.

---

## <a name="delete-azure-resources"></a>Eliminación de recursos de Azure

Si elimina cualquiera de los recursos de Azure usados para las bases de conocimiento, estas dejarán de funcionar. Antes de eliminar todos los recursos, asegúrese de exportar las bases de conocimiento desde la página **Settings** (Configuración).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre [App Service](../../../app-service/index.yml) y el [servicio Azure Search](../../../search/index.yml).

> [!div class="nextstepaction"]
> [Aprenda a crear en colaboración](../index.yml)