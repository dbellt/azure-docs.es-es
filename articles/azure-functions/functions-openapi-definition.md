---
title: Exposición de API de funciones mediante Azure API Management
description: Cree una definición de OpenAPI que permita que otras aplicaciones y servicios llamen a la función en Azure.
ms.topic: how-to
ms.date: 05/04/2021
ms.openlocfilehash: ec25b03b3d93fcd69297899653279fea86eabb02
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110375711"
---
# <a name="expose-serverless-apis-from-http-endpoints-using-azure-api-management"></a>Exposición de API sin servidor desde puntos de conexión HTTP mediante Azure API Management

Azure Functions se integra con Azure API Management en el portal para permitirle exponer los puntos de conexión de la función de desencadenador HTTP como API REST. Estas API se describen mediante una definición de OpenAPI. Este archivo JSON (o YAML) contiene información sobre qué operaciones están disponibles en una API. Incluye detalles sobre cómo se deben estructurar los datos de solicitudes y respuestas de la API. Mediante la integración de la aplicación de funciones, puede hacer que API Management genere estas definiciones de OpenAPI.  

En este artículo se muestra cómo integrar la aplicación de funciones con API Management. Esta integración se puede utilizar con las aplicaciones de funciones desarrolladas en cualquier [lenguaje compatible](supported-languages.md). También puede [importar la aplicación de funciones desde Azure API Management](../api-management/import-function-app-as-api.md).

En el caso de las funciones de biblioteca de clases C#, también puede [usar Visual Studio](openapi-apim-integrate-visual-studio.md) para crear y publicar API sin servidor que se integren con API Management.  

## <a name="create-the-api-management-instance"></a>Creación de la instancia de API Management

Para crear una instancia de API Management vinculada a la aplicación de funciones:

1. Seleccione la aplicación de funciones, elija **API Management** en el menú de la izquierda y, a continuación, seleccione **Crear nuevo** en **API Management**.

    :::image type="content" source="media/functions-openapi-definitions/select-all-settings-openapi.png" alt-text="Elegir API Management":::


1. Utilice los valores de API Management tal como se especifica en la tabla siguiente:

    | Configuración      | Valor sugerido  | Descripción                                        |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nombre** | Nombre único globalmente | Se genera un nombre basado en el nombre de la aplicación de función. |
    | **Suscripción** | Su suscripción | Suscripción en la que se crea este nuevo recurso. |  
    | **[Grupo de recursos](../azure-resource-manager/management/overview.md)** |  myResourceGroup | El mismo recurso que la aplicación de función, que debe establecerse automáticamente. |
    | **Ubicación** | Ubicación del servicio | Considere la posibilidad de elegir la misma ubicación que en el caso de la aplicación de funciones. |
    | **Nombre de la organización** | Contoso | El nombre de la organización utilizado en el portal para desarrolladores y para notificaciones de correo electrónico. |
    | **Correo electrónico del administrador** | Su correo electrónico | Correo electrónico que recibe las notificaciones del sistema desde API Management. |
    | **Plan de tarifa** | Consumo | El nivel de consumo no está disponible en todas las regiones. Para obtener información completa sobre los precios, vea la [página de precios de API Management](https://azure.microsoft.com/pricing/details/api-management/). |

    ![Creación de un nuevo servicio de API Management](media/functions-openapi-definitions/new-apim-service-openapi.png)

1. Elija **Crear** para crear la instancia de API Management, que puede tardar varios minutos.

1. Una vez que Azure crea la instancia, habilita la opción **Habilitar Application Insights** en la página. Selecciónela para enviar registros al mismo lugar que la aplicación de funciones.

## <a name="import-functions"></a>Importación de funciones

Después de crear la instancia de API Management, puede importar los puntos de conexión de la función desencadenada por HTTP. En este ejemplo se importa un punto de conexión denominado TurbineRepair.   

1. En la página API Management, seleccione **Vincular API**.

1. Se abre **Importar Azure Functions** con la función **TurbineRepair** resaltada. Haga clic en **Seleccionar** para continuar.

    ![Importación de Azure Functions en API Management](media/functions-openapi-definitions/import-function-openapi.png)

1. En la página **Crear a partir de Function App**, acepte los valores predeterminados y seleccione **Crear**.

    :::image type="content" source="media/functions-openapi-definitions/create-function-openapi.png" alt-text="Crear a partir de Function App":::

    Azure crea la API para la función.

## <a name="download-the-openapi-definition"></a>Descarga de la definición de OpenAPI

Una vez importadas las funciones, puede descargar la definición de OpenAPI desde la instancia de API Management.

1. Seleccione **Descargar definición de OpenAPI** en la parte superior de la página.
   
   ![Descarga de la definición de OpenAPI](media/functions-openapi-definitions/download-definition.png)

2. Guarde el archivo JSON descargado y, a continuación, ábralo. Revise la definición.

## <a name="next-steps"></a>Pasos siguientes

Ahora puede refinar la definición de API Management en el portal. También puede obtener [más información acerca de API Management](../api-management/api-management-key-concepts.md).

> [!div class="nextstepaction"]
> [Edición de la definición de OpenAPI en API Management](../api-management/edit-api.md)
