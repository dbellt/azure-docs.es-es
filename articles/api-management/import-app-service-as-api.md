---
title: Importación de una aplicación web de Azure a Azure API Management | Microsoft Docs
description: En este artículo se muestra cómo usar Azure API Management para importar una API web hospedada en Azure App Service.
services: api-management
documentationcenter: ''
author: vladvino
ms.service: api-management
ms.topic: article
ms.date: 04/27/2021
ms.author: apimpm
ms.openlocfilehash: de51a32f737e71501723847ec473387ac419f8a6
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108230791"
---
# <a name="import-an-azure-web-app-as-an-api"></a>Importación de una aplicación web de Azure como API

En este artículo se muestra cómo importar una aplicación web de Azure a Azure API Management y probar la API importada mediante Azure Portal.

> [!NOTE]
> Puede usar la extensión de API Management para Visual Studio Code para importar y administrar las API. Siga el [tutorial de la extensión de API Management](visual-studio-code-tutorial.md) para instalar y empezar a trabajar.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Importar una aplicación web hospedada en App Service
> * Prueba de la API en Azure Portal

## <a name="expose-web-app-with-api-management"></a>Exponer la aplicación web con API Management

[Azure App Service](../app-service/overview.md) es un servicio basado en HTTP para hospedar aplicaciones web, API REST y back-ends para dispositivos móviles. Los desarrolladores de API pueden usar sus pilas de tecnología y canalizaciones preferidas para desarrollar API y publicar sus back-end de API como aplicaciones web en un entorno seguro y escalable. A continuación, use API Management para exponer las aplicaciones web, administrar y proteger las API a lo largo de su ciclo de vida y publicarlas para los consumidores.

API Management es el entorno recomendado para exponer una API hospedada en una aplicación web, por varios motivos:

* Separa la administración y protección del front-end expuesto a los consumidores de la API de la administración y supervisión de la aplicación web de back-end.
* Administra las API web hospedadas como aplicaciones web en el mismo entorno que las otras API.
* Aplica [directivas](api-management-policies.md) para cambiar el comportamiento de la API, como la limitación de la frecuencia de las llamadas.
* Dirige a los consumidores de las API al [portal para desarrolladores](api-management-howto-developer-portal.md) personalizable de API Management para detectar y obtener información sobre las API, solicitar acceso y probarlas.

Para obtener más información, consulte [Acerca de API Management](api-management-key-concepts.md).

## <a name="openapi-specification-versus-wildcard-operations"></a>Especificación de OpenAPI frente a operaciones con caracteres comodín

API Management admite la importación de aplicaciones web hospedadas en App Service que incluyen una especificación de OpenAPI (definición de Swagger). Sin embargo, no se requiere una especificación de OpenAPI.

* Si la aplicación web tiene una especificación de OpenAPI configurada en la definición de una API, API Management crea operaciones de API que se asignan directamente a la definición, incluidas las rutas de acceso, los parámetros y los tipos de respuesta necesarios. 

  Se recomienda tener una especificación de OpenAPI, ya que la API se importa a API Management con alta fidelidad, lo que da flexibilidad para validar, administrar, proteger y actualizar las configuraciones para cada operación por separado.

* Si no se proporciona una especificación de OpenAPI, API Management [operaciones con caracteres comodín](add-api-manually.md#add-and-test-a-wildcard-operation) para los verbos HTTP comunes (GET, PUT, etc.). Anexe una ruta de acceso o parámetros necesarios a una operación con caracteres comodín para pasar una solicitud de API a la API de back-end.

  Con las operaciones con caracteres comodín, todavía puede aprovechar las mismas características de API Management, pero las operaciones no se definen con el mismo nivel de detalle de manera predeterminada. En cualquier caso, puede [editar](edit-api.md) o [agregar](add-api-manually.md) operaciones a la API importada.
 
### <a name="example"></a>Ejemplo
La aplicación web de back-end puede admitir dos operaciones GET: 
*  `https://myappservice.azurewebsites.net/customer/{id}`
*  `https://myappservice.azurewebsites.net/customers`

Importa la aplicación web al servicio API Management en una ruta de acceso como `https://contosoapi.azureapi.net/store`. En la tabla siguiente se muestran las operaciones que se importan a API Management, ya sea con una especificación de OpenAPI o sin ella: 

| Tipo |Operaciones importadas  |Solicitudes de ejemplo |
|---------|---------|---------|
|Especificación de OpenAPI    | `GET  /customer/{id}`<br/><br/> `GET  /customers`         |  `GET https://contosoapi.azureapi.net/store/customer/1`<br/><br/>`GET https://contosoapi.azureapi.net/store/customers`       |
|Wildcard (Carácter comodín)     | `GET  /*`         | `GET https://contosoapi.azureapi.net/store/customer/1`<br/><br/>`GET https://contosoapi.azureapi.net/store/customers`  |

La operación con caracteres comodín permite las mismas solicitudes al servicio de back-end que las operaciones de la especificación de OpenAPI. Sin embargo, las operaciones especificadas por OpenAPI se pueden administrar de manera independiente en API Management. 

## <a name="prerequisites"></a>Requisitos previos

+ Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
+ Asegúrese de que haya una instancia de App Service en su suscripción. Para obtener más información, consulte la [Documentación de App Service](../app-service/index.yml).

  Para ver los pasos para crear una API web de ejemplo y publicarla como aplicación web de Azure, consulte:

    * [Tutorial: Creación de una API web con ASP.NET Core](/aspnet/core/tutorials/first-web-api)
    * [Publicación de una aplicación de ASP.NET Core en Azure con Visual Studio Code](/aspnet/core/tutorials/publish-to-azure-webapp-using-vscode)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-and-publish-a-backend-api"></a><a name="create-api"> </a>Importación y publicación de una API de back-end

> [!TIP]
> Los pasos siguientes inician la importación mediante Azure API Management en Azure Portal. También se puede vincular a API Management directamente desde la aplicación web; para ello, seleccione **API Management** en el menú **API** de la aplicación.  

1. Vaya al servicio API Management en Azure Portal y seleccione **API** en el menú.
1. Seleccione **App Service** de la lista.

    :::image type="content" source="media/import-app-service-as-api/app-service.png" alt-text="Creación desde App Service":::
1. Seleccione **Examinar** para ver la lista de instancias de App Service en su suscripción.
1. Seleccione una instancia de App Service. Si hay una definición de OpenAPI asociada a la aplicación web seleccionada, API Management la captura e importa. 

    Si no se encuentra una definición de OpenAPI, API Management expone la API generando operaciones con caracteres comodín para verbos HTTP comunes. 
1. Agregue un sufijo URL de API. El sufijo es un nombre que identifica esta API específica en esta instancia de API Management. Debe ser exclusivo en esta instancia de APIM.
1. Publique la API asociándola a un producto. En este caso, se usa el producto "*Unlimited*". Si quiere que la API se publique y esté disponible para los desarrolladores, agréguela a un producto. Puede hacerlo durante la creación de la API o configurarla más adelante.

    > [!NOTE]
    > Los productos son asociaciones de una o varias API. Puede incluir muchas API y ofrecerlas a los desarrolladores mediante el portal para desarrolladores. En primer lugar, los desarrolladores deben suscribirse a un producto para acceder a la API. Al suscribirse, obtienen una clave de suscripción que funciona con cualquier API de ese producto. Si creó la instancia de APIM, ya es un administrador, así que, de forma predeterminada, está suscrito a todos los productos.
    >
    > De forma predeterminada, cada instancia de API Management incluye dos productos de ejemplo:
    > * **Starter**
    > * **Sin límite**   
1. Escriba otros valores de la API. Puede establecer los valores durante la creación o luego accediendo a la pestaña **Ajustes**. Los valores de configuración se explican en el tutorial [Importación y publicación de la primera API](import-and-publish.md#import-and-publish-a-backend-api).
1. Seleccione **Crear**.
    :::image type="content" source="media/import-app-service-as-api/import-app-service.png" alt-text="Creación de API desde App Service":::

## <a name="test-the-new-api-in-the-azure-portal"></a>Prueba de la nueva API en Azure Portal

Se puede llamar a las operaciones directamente desde Azure Portal, lo que proporciona una forma cómoda de ver y probar las operaciones de una API. También puede probar la API en el [portal para desarrolladores](api-management-howto-developer-portal.md) o con sus propias herramientas de cliente REST.

1. Seleccione la API que creó en los pasos anteriores.
1. Seleccione la pestaña **Prueba**.
1. Seleccione una operación.

    La página muestra los campos de parámetros de consulta y los campos para los encabezados. Uno de los encabezados es "Ocp-Apim-Suscripción-Key", para la clave de suscripción del producto que está asociado a esta API. Si ha creado la instancia de API Management, significa que ya es administrador, por lo que la clave se rellena automáticamente. 
1. Presione **Enviar**.

    Si la prueba se completa correctamente, el back-end responde con **200 - Correcto** y algunos datos.

### <a name="test-wildcard-operation-in-the-portal"></a>Prueba de la operación con caracteres comodín en el portal

Cuando se generan operaciones con caracteres comodín, es posible que las operaciones no se asignen directamente a la API de back-end. Por ejemplo, una operación GET con caracteres comodín importada a API Management usa la ruta de acceso `/` de manera predeterminada. Sin embargo, la API de back-end podría admitir una operación GET en la ruta de acceso siguiente:

`/api/TodoItems`

Puede probar la ruta de acceso `/api/TodoItems` como se muestra a continuación. 

1. Seleccione la API que ha creado y seleccione la operación.
1. Seleccione la pestaña **Prueba**.
1. En **Parámetros de la plantilla**, actualice el valor junto al nombre del carácter comodín (*). Por ejemplo, escriba `api/TodoItems`. Este valor se anexa a la ruta de acceso `/` de la operación con caracteres comodín.

    :::image type="content" source="media/import-app-service-as-api/test-wildcard-operation.png" alt-text="Prueba de la operación con caracteres comodín":::
1. Seleccione **Enviar**.

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Transformación y protección de una API publicada](transform-api.md)
