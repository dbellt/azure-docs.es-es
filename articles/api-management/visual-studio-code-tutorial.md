---
title: 'Tutorial: Importación y administración de API: Azure API Management y Visual Studio Code | Microsoft Docs'
description: En este tutorial aprenderá a utilizar la extensión de Azure API Management para Visual Studio Code a fin de importar, probar y administrar las API.
ms.service: api-management
author: dlepow
ms.author: apimpm
ms.topic: tutorial
ms.date: 12/10/2020
ms.openlocfilehash: 0090d981e93cee12f2feaaf7d2c12f341564f6ec
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107482337"
---
# <a name="tutorial-use-the-api-management-extension-for-visual-studio-code-to-import-and-manage-apis"></a>Tutorial: Uso de la extensión de API Management para Visual Studio Code a fin de importar y administrar las API

En este tutorial aprenderá a utilizar la extensión de API Management para Visual Studio Code para realizar operaciones comunes en API Management. Utilice el entorno familiar de Visual Studio Code para importar, actualizar, probar y administrar las API.

Aprenderá a:

> [!div class="checklist"]
> * Importación de una API en API Management
> * Editar la API
> * Aplicar las directivas de API Management
> * Prueba de la API


:::image type="content" source="media/visual-studio-code-tutorial/tutorial-api-result.png" alt-text="API en la extensión de API Management":::

Para ver una introducción a otras características de API Management, consulte los tutoriales de API Management mediante [Azure Portal](import-and-publish.md).

## <a name="prerequisites"></a>Prerrequisitos
- Reconocimiento de la [terminología de Azure API Management](api-management-terminology.md)
- Asegurarse de haber instalado [Visual Studio Code](https://code.visualstudio.com/) y la [extensión de Azure API Management para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-apimanagement&ssr=false#overview) más reciente.
- [Creación de una instancia de API Management](vscode-create-service-instance.md)

## <a name="import-an-api"></a>Importación de una API

En el ejemplo siguiente se importa una especificación de OpenAPI en formato JSON a API Management. Microsoft proporciona la API de back-end usada en este ejemplo y la hospeda en Azure en `https://conferenceapi.azurewebsites.net?format=json`.

1. En Visual Studio Code, seleccione el icono de Azure en la barra de actividades.
1. En el panel del Explorador, expanda la instancia de API Management que haya creado.
1. Haga clic con el botón derecho en **API** y seleccione **Import from OpenAPI Link** (Importar desde el vínculo de OpenAPI). 
1. Cuando se le solicite, escriba los siguientes valores:
    1. Un **vínculo de OpenAPI** para el contenido en formato JSON. En este ejemplo: *https://conferenceapi.azurewebsites.net?format=json* .
    Esta dirección URL es el servicio que implementa la API de ejemplo. API Management reenvía las solicitudes a esta dirección.
    1. Un **nombre de API**, como *demo-conference-api*, que sea único en la instancia de API Management. Este nombre solo puede contener letras, números y guiones. El primer y el último carácter deben ser alfanuméricos. Este nombre se utiliza en la ruta de acceso para llamar a la API.

Una vez que la API se haya importado correctamente, aparecerá en el panel del Explorador, y las operaciones de API disponibles se mostrarán bajo el nodo **Operations** (Operaciones).

:::image type="content" source="media/visual-studio-code-tutorial/tutorial-api-operations.png" alt-text="API importada en el panel del Explorador":::

## <a name="edit-the-api"></a>Edición de la API

Puede editar la API en Visual Studio Code. Por ejemplo, edite la descripción JSON de Resource Manager de la API en la ventana del Editor a fin de quitar el protocolo **http** que se utiliza para acceder a la API. Luego, seleccione **Archivo** > **Guardar**.

:::image type="content" source="media/visual-studio-code-tutorial/import-demo-api.png" alt-text="Descripción de la edición de JSON":::

Para editar el formato de OpenAPI, haga clic con el botón derecho en el nombre de la API en el panel del Explorador y seleccione **Edit OpenAPI** (Editar OpenAPI). Realice los cambios y después seleccione **Archivo** > **Guardar**.

## <a name="apply-policies-to-the-api"></a>Aplicación de directivas a la API 

API Management proporciona [directivas](api-management-policies.md) que puede configurar para las API. Las directivas son una colección de declaraciones que se ejecutan secuencialmente en la solicitud o respuesta de una API. Las directivas pueden ser globales, que se aplican a todas las API de la instancia de API Management, o bien se pueden limitar a una API o una operación de API en concreto.

En esta sección se muestra cómo aplicar algunas directivas de salida comunes a la API, que transforman la respuesta de la API. Las directivas de este ejemplo cambian los encabezados de respuesta y ocultan las direcciones URL de back-end originales que aparecen en el cuerpo de la respuesta.

1. En el panel del Explorador, seleccione **Policy** (Directiva) bajo el nombre *demo-conference-api* que ha importado. El archivo de directivas se abre en la ventana del Editor. Este archivo configura las directivas para todas las operaciones de la API. 

1. Actualice el archivo con el siguiente contenido en el elemento `<outbound>`:
    ```html
    [...]
    <outbound>
        <set-header name="Custom" exists-action="override">
            <value>"My custom value"</value>
        </set-header>
        <set-header name="X-Powered-By" exists-action="delete" />
        <redirect-content-urls />
        <base />
    </outbound>
    [...]
    ```

    * La primera directiva `set-header` agrega un encabezado de respuesta personalizado para fines de demostración.
    * En la segunda directiva `set-header` se elimina el encabezado **X-Powered-By**, si existe. Este encabezado puede revelar la plataforma de la aplicación que se usa en el back-end de la API, y los publicadores lo suelen quitar.
    * La directiva `redirect-content-urls` reescribe (enmascara) los vínculos del cuerpo de la respuesta para que apunten al vinculo equivalente a través de la puerta de enlace de API Management.
    
1. Guarde el archivo. Si se le solicita, seleccione **Cargar** para cargar el archivo en la nube.

## <a name="test-the-api"></a>Prueba de la API

### <a name="get-the-subscription-key"></a>Establecimiento de la clave de suscripción

Para probar la API que ha importado y las directivas que se aplican, necesita una clave de suscripción para la instancia de API Management.

1. En el panel del Explorador, haga clic con el botón derecho en el nombre de la instancia de API Management.
1. Seleccione **Copy Subscription Key** (Copiar la clave de suscripción).

    :::image type="content" source="media/visual-studio-code-tutorial/copy-subscription-key.png" alt-text="Copia de la clave de suscripción":::

### <a name="test-an-api-operation"></a>Prueba de una operación de API

1. En el panel del Explorador, seleccione el nodo **Operations**(Operaciones) bajo el nombre *demo-conference-api* que ha importado.
1. Seleccione una operación, como *GetSpeakers*, y, después, haga clic con el botón derecho en ella y seleccione **Operación de prueba**.
1. En la ventana del Editor, junto a **Ocp-Apim-Subscription-Key**, reemplace `{{SubscriptionKey}}` por la clave de suscripción copiada.
1. Seleccione **Enviar solicitud**. 

:::image type="content" source="media/visual-studio-code-tutorial/test-api.png" alt-text="Envío de solicitud de API desde Visual Studio Code":::

Si la solicitud es correcta, el back-end responde con **200 OK** y algunos datos.

:::image type="content" source="media/visual-studio-code-tutorial/test-api-policies.png" alt-text="Operación de prueba de API":::

Observe los siguientes detalles en la respuesta:
* El encabezado **Custom** se agrega a la respuesta.
* El encabezado **X-Powered-by** no aparece en la respuesta.
* Las direcciones URL al back-end de API se redirigen a la puerta de enlace de API Management, en este caso `https://apim-hello-world.azure-api.net/demo-conference-api`.

### <a name="trace-the-api-operation"></a>Seguimiento de la operación de API

Para una información detallada del seguimiento que le ayude a depurar la operación de API, seleccione el vínculo que aparece junto a **Ocp-APIM-Trace-Location**. 

El archivo JSON en esa ubicación contiene información de seguimiento de entrada, de back-end y de salida para que pueda determinar dónde se producen los problemas una vez realizada la solicitud.

> [!TIP]
> Al probar las operaciones de API, la extensión de API Management permite la [depuración de directivas](api-management-debug-policies.md) opcional (disponible en el nivel de servicio del desarrollador).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no la necesite, elimine la instancia de API Management; para ello, haga clic con el botón derecho y seleccione **Abrir en el portal** para [eliminar el servicio API Management](get-started-create-service-instance.md#clean-up-resources) y su grupo de recursos.

O bien, puede seleccionar **Eliminar API Management** para eliminar solo la instancia de API Management (esta operación no elimina su grupo de recursos).

:::image type="content" source="media/visual-studio-code-tutorial/vscode-apim-delete.png" alt-text="Eliminación de la instancia de API Management de VS Code":::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial se han presentado varias características de la extensión de API Management para Visual Studio Code que puede utilizar para importar y administrar las API. Ha aprendido a:

> [!div class="checklist"]
> * Importación de una API en API Management
> * Editar la API
> * Aplicar las directivas de API Management
> * Prueba de la API

La extensión de API Management proporciona características adicionales para trabajar con las API. Por ejemplo, [depuración de directivas](api-management-debug-policies.md) (disponible en el nivel de servicio del desarrollador) o creación y administración de [valores con nombre](api-management-howto-properties.md).
