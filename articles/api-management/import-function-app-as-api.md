---
title: Importación de una instancia de Azure Function App como API en API Management
titleSuffix: Azure API Management
description: En este artículo se muestra cómo importar una instancia de Azure Function App en Azure API Management como API.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 04/16/2021
ms.author: apimpm
ms.openlocfilehash: 8e8047fc6865bab8f4aac2315b269bf7526b1e42
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2021
ms.locfileid: "107599385"
---
# <a name="import-an-azure-function-app-as-an-api-in-azure-api-management"></a>Importación de una instancia de Azure Function App como API en Azure API Management

Azure API Management admite la importación de instancias de Azure Function App como API nuevas o su anexión a las API existentes. El proceso genera automáticamente una clave de host la aplicación de funciones de Azure que, después, se asigna a un valor con nombre en Azure API Management.

En este artículo se explica cómo importar y probar una instancia de Azure Function App como API en Azure API Management. 

Aprenderá a:

> [!div class="checklist"]
> * Importar una instancia de Azure Function App como una API
> * Anexar una instancia de Azure Function App a una API
> * Ver la nueva clave de host de Azure Function App y el valor con nombre de Azure API Management
> * Prueba de la API en Azure Portal

## <a name="prerequisites"></a>Prerrequisitos

* Complete el inicio rápido [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
* Asegúrese de que tiene una aplicación de Azure Functions en la suscripción. Para más información, consulte [Creación de una instancia de Azure Function App](../azure-functions/functions-get-started.md). Functions debe tener un desencadenador HTTP y el nivel de autorización establecido en *Anónimo* o *Función*.

> [!NOTE]
> Puede usar la extensión de API Management para Visual Studio Code para importar y administrar las API. Siga el [tutorial de la extensión de API Management](visual-studio-code-tutorial.md) para instalar y empezar a trabajar.

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-an-azure-function-app-as-a-new-api"></a><a name="add-new-api-from-azure-function-app"></a> Importación de una instancia de Azure Function App como una API nueva

Siga estos pasos para crear una API desde una instancia de Azure Function App.

1. Vaya al servicio API Management en Azure Portal y seleccione **API** en el menú.

2. En la lista **Add a new API** (Agregar una nueva API), seleccione **Function App**.

    :::image type="content" source="./media/import-function-app-as-api/add-01.png" alt-text="Captura de pantalla que muestra el icono de aplicación de funciones.":::

3. Haga clic en **Examinar** para seleccionar las funciones que se van a importar.

    :::image type="content" source="./media/import-function-app-as-api/add-02.png" alt-text="Captura de pantalla que resalta el botón Examinar.":::

4. Haga clic en la sección **Function App** para elegir en la lista de instancias de Function App.

    :::image type="content" source="./media/import-function-app-as-api/add-03.png" alt-text="Captura de pantalla que resalta la sección Aplicación de funciones.":::

5. Busque la instancia de Function App de la que desea importar funciones, haga clic en ella y presione **Seleccionar**.

    :::image type="content" source="./media/import-function-app-as-api/add-04.png" alt-text="Captura de pantalla que resalta la aplicación de funciones desde la que desea importar funciones y el botón Seleccionar.":::

6. Seleccione las funciones que desea importar y haga clic en **Seleccionar**.
    * Solo puede importar el desencadenador HTTP basado en Functions con los niveles de autorización *Anónimo* o *Función*.

    :::image type="content" source="./media/import-function-app-as-api/add-05.png" alt-text="Captura de pantalla que resalta las funciones que desea importar y el botón Seleccionar.":::

7. Cambie a la vista **Completa** y asigne el **Producto** a la nueva API. 
8. Si es necesario, especifique otros campos durante la creación o configúrelos más adelante en la pestaña **Ajustes**. 
    * Los valores de configuración se explican en el tutorial [Importación y publicación de la primera API](import-and-publish.md#import-and-publish-a-backend-api).

    >[!NOTE]
    > Los productos son asociaciones de una o varias API que se ofrecen a los desarrolladores a través del portal para desarrolladores. En primer lugar, los desarrolladores deben suscribirse a un producto para acceder a la API. Al suscribirse, obtienen una clave de suscripción que funciona con cualquier API de ese producto. Como creador de la instancia de API Management, ya es un administrador, así que de forma predeterminada está suscrito a todos los productos.
    >
    > De forma predeterminada, cada instancia de API Management incluye dos productos de ejemplo:
    > - **Starter**
    > - **Sin límite**

9. Haga clic en **Crear**.

## <a name="append-azure-function-app-to-an-existing-api"></a><a name="append-azure-function-app-to-api"></a>Anexión de una instancia de Azure Function App a una API existente

Siga estos pasos para anexar Azure Function App a una API existente.

1. En la instancia del servicio **Azure API Management**, seleccione **API** en el menú de la izquierda.

2. Elija una API que desea importar en una instancia de Azure Function App. Haga clic en **...**  y seleccione **Importar** en el menú contextual.

    :::image type="content" source="./media/import-function-app-as-api/append-function-api-1.png" alt-text="Captura de pantalla que resalta la opción de menú Importar.":::

3. Haga clic en el icono **Function App**.

    :::image type="content" source="./media/import-function-app-as-api/append-function-api-2.png" alt-text="Captura de pantalla que resalta el icono de aplicación de funciones.":::

4. En la ventana emergente, haga clic en **Examinar**.

    :::image type="content" source="./media/import-function-app-as-api/append-function-api-3.png" alt-text="Captura de pantalla que muestra el botón Examinar.":::

5. Haga clic en la sección **Function App** para elegir en la lista de instancias de Function App.

    :::image type="content" source="./media/import-function-app-as-api/add-03.png" alt-text="Captura de pantalla que resalta la lista de aplicaciones de funciones.":::

6. Busque la instancia de Function App de la que desea importar funciones, haga clic en ella y presione **Seleccionar**.

    :::image type="content" source="./media/import-function-app-as-api/add-04.png" alt-text="Captura de pantalla que resalta la aplicación de funciones desde la que desea importar funciones.":::

7. Seleccione las funciones que desea importar y haga clic en **Seleccionar**.

    :::image type="content" source="./media/import-function-app-as-api/add-05.png" alt-text="Captura de pantalla que resalta las funciones que desea importar":::.

8. Haga clic en **Import**.

    :::image type="content" source="./media/import-function-app-as-api/append-function-api-4.png" alt-text="Anexar desde Function App":::

## <a name="authorization"></a><a name="authorization"></a>Autorización

La importación de una instancia de Azure Function App genera automáticamente:

* una clave de host en la aplicación de función con el nombre apim-{*nombre de instancia del servicio Azure API Management*}.
* un valor con nombre dentro de la instancia de Azure API Management con el nombre {*nombre de la instancia de Azure Function App*}-key, que contiene la clave de host creada.

En el caso de las API creadas después del 4 de abril de 2019, la clave de host se pasa en las solicitudes HTTP desde API Management a la aplicación de función en un encabezado. Las API más antiguas pasan la clave de host como [un parámetro de consulta](../azure-functions/functions-bindings-http-webhook-trigger.md#api-key-authorization). Se puede cambiar este comportamiento mediante la `PATCH Backend` [llamada API REST](/rest/api/apimanagement/2019-12-01/backend/update#backendcredentialscontract) en la entidad de *back-end* asociada con la aplicación de funciones.

> [!WARNING]
> Si se quita o se cambia el valor de la clave de host de Azure Function App o el valor con nombre de Azure API Management, se interrumpirá la comunicación entre los servicios. Los valores no se sincronizan automáticamente.
>
> Si necesita rotar la clave de host, asegúrese de que también se modifica el valor con nombre de Azure API Management.

### <a name="access-azure-function-app-host-key"></a>Acceso a la clave de host de Azure Function App

1. Vaya a la instancia de Azure Function App.

    :::image type="content" source="./media/import-function-app-as-api/keys-01.png" alt-text="Captura de pantalla que resalta la selección de la instancia de Function App.":::

2. En la sección **Funciones** del menú de navegación lateral, seleccione **Claves de aplicación**.

    :::image type="content" source="./media/import-function-app-as-api/keys-02b.png" alt-text="Captura de pantalla que resalta la opción de configuración Aplicación de funciones.":::

3. Busque las claves en la sección **Claves de host**.

    :::image type="content" source="./media/import-function-app-as-api/keys-03.png" alt-text="Captura de pantalla que resalta la sección Host Keys (Claves de host).":::

### <a name="access-the-named-value-in-azure-api-management"></a>Acceso al valor con nombre en Azure API Management

Vaya a la instancia de Azure API Management y seleccione **Valores con nombre** en el menú de la izquierda. La clave de Azure Function App se almacena ahí.

:::image type="content" source="./media/import-function-app-as-api/api-named-value.png" alt-text="Agregar desde Function App":::

## <a name="test-the-new-api-in-the-azure-portal"></a><a name="test-in-azure-portal"></a> Prueba de la nueva API en Azure Portal

Puede llamar a operaciones directamente desde Azure Portal. Mediante Azure Portal es una manera cómoda de ver y probar las operaciones de una API.  

:::image type="content" source="./media/import-function-app-as-api/test-api.png" alt-text="Captura de pantalla que resalta el procedimiento de prueba.":::

1. Seleccione la API que ha creado en la sección anterior.

2. Seleccione la pestaña **Prueba**.

3. Seleccione la operación que desea probar.

    * La página muestra los campos de los parámetros de consulta y de los encabezados. 
    * Uno de los encabezados es "Ocp-Apim-Suscripción-Key" para la clave de suscripción del producto que está asociado a esta API. 
    * Si ha creado la instancia de API Management, significa que ya es administrador, por lo que la clave se rellena automáticamente. 

4. Seleccione **Enviar**.

    * Si la solicitud es correcta, el back-end responde con **200 OK** y algunos datos.

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Transformación y protección de una API publicada](transform-api.md)
