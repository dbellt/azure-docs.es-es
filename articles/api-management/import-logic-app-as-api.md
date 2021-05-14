---
title: Importación de una aplicación lógica como una API desde Azure Portal | Microsoft Docs
description: En este artículo se muestra cómo utilizar API Management (APIM) para importar una aplicación lógica como API.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 04/16/2021
ms.author: apimpm
ms.openlocfilehash: 6825e5d7849f97aac4627d4856d26b3e08ab6761
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2021
ms.locfileid: "108230644"
---
# <a name="import-a-logic-app-as-an-api"></a>Importación de una aplicación lógica como API

Este artículo muestra cómo importar una aplicación lógica como una API y probar la API importada.

En este artículo aprenderá a:

> [!div class="checklist"]
>
> -   Importación de una aplicación lógica como API
> -   Prueba de la API en Azure Portal

## <a name="prerequisites"></a>Prerrequisitos

-   Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)
-   Asegúrese de que hay alguna aplicación lógica en su suscripción que expone un punto de conexión HTTP. Para más información, consulte el artículo acerca del [desencadenamiento de flujos de trabajo con puntos de conexión HTTP](../logic-apps/logic-apps-http-endpoint.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-and-publish-a-back-end-api"></a><a name="create-api"> </a>Importación y publicación de una API de back-end

1. Vaya al servicio API Management en Azure Portal y seleccione **API** en el menú.
1. Seleccione **Aplicación lógica** en la lista **Add a new API** (Agregar una nueva API).

    :::image type="content" source="./media/import-logic-app-as-api/logic-app-select.png" alt-text="Selección de la categoría de la aplicación lógica":::

1. Presione **Examinar** para ver la lista de aplicaciones Logic Apps con desencadenador HTTP en su suscripción. 
    * Las aplicaciones lógicas *sin* desencadenador HTTP no aparecerán en la lista.

    :::image type="content" source="./media/import-logic-app-as-api/browse-logic-apps.png" alt-text="Búsqueda de las aplicaciones lógicas existentes con el desencadenador correcto":::

1. Seleccione la aplicación lógica. 

    :::image type="content" source="./media/import-logic-app-as-api/select-logic-app-import-2.png" alt-text="Selección de la aplicación lógica":::

1. API Management busca el swagger asociado a la aplicación seleccionada, lo captura y lo importa.
1. Agregue un sufijo URL de API. 
    * El sufijo identifica de forma exclusiva esta API específica de esta instancia de API Management.

    :::image type="content" source="./media/import-logic-app-as-api/create-from-logic-app.png" alt-text="Finalización de los campos":::

1. Si desea que la API se publique y esté disponible para los desarrolladores, cambie a la vista **Completa** y asóciela a un **producto**. En este ejemplo se usa el producto *"Ilimitado"* . 
    * Puede agregar la API a un producto durante su creación o posteriormente a través de la pestaña **Configuración**.

    >[!NOTE]
    > Los productos son asociaciones de una o varias API que se ofrecen a los desarrolladores a través del portal para desarrolladores. En primer lugar, los desarrolladores deben suscribirse a un producto para acceder a la API. Al suscribirse, obtienen una clave de suscripción que funciona con cualquier API de ese producto. Como creador de la instancia de API Management, ya es un administrador, así que de forma predeterminada está suscrito a todos los productos.
    >
    > De forma predeterminada, cada instancia de API Management incluye dos productos de ejemplo:
    > - **Starter**
    > - **Sin límite**

1. Escriba otros valores de la API. 
    * Puede establecer estos valores durante la creación o posteriormente accediendo a la pestaña **Configuración**. Los valores de configuración se explican en el tutorial [Importación y publicación de la primera API](import-and-publish.md#import-and-publish-a-backend-api).
1. Seleccione **Crear**.

## <a name="test-the-api-in-the-azure-portal"></a>Prueba de la API en Azure Portal

Se puede llamar a las operaciones directamente desde Azure Portal, lo que proporciona una forma cómoda de ver y probar las operaciones de una API.

:::image type="content" source="./media/import-logic-app-as-api/test-logic-app-api.png" alt-text="Probar la aplicación lógica":::

1. Seleccione la API que creó en los pasos anteriores.
2. Presione la pestaña **Prueba**.
3. Seleccione la operación que desea probar.

    * La página muestra los campos de los parámetros de consulta y de los encabezados. 
    * Uno de los encabezados es "Ocp-Apim-Suscripción-Key" para la clave de suscripción del producto que está asociado a esta API. 
    * Si ha creado la instancia de API Management, significa que ya es administrador, por lo que la clave se rellena automáticamente.

4. Presione **Enviar**.

    * Si la solicitud es correcta, el back-end responde con **200 OK** y con datos.

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

>[!NOTE]
>Cada aplicación lógica tiene una operación **manual-invoke**. Para que la API se componga de varias aplicaciones lógicas y evitar colisiones, debe cambiar el nombre de la función.

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
>
> [Transformación y protección de una API publicada](transform-api.md)
