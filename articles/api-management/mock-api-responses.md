---
title: 'Tutorial: Simulación de respuestas de API en API Management: Azure Portal | Microsoft Docs'
description: En este tutorial, usará API Management para establecer una directiva en una API de forma que devuelva una respuesta simulada si el back-end no está disponible para enviar respuestas reales.
author: vladvino
ms.service: api-management
ms.custom: mvc, devx-track-azurecli
ms.topic: tutorial
ms.date: 02/09/2021
ms.author: apimpm
ms.openlocfilehash: a7617a36ed800f1765ed7723568a4b612fcb6518
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107587602"
---
# <a name="tutorial-mock-api-responses"></a>Tutorial: Simulación de respuestas de API

Se pueden importar API de back-end en una API de API Management (APIM), o bien se pueden crear o administrar manualmente. En los pasos de este tutorial se muestra cómo usar APIM para crear una API en blanco y administrarla manualmente. Después, establecerá una directiva en una API para que devuelva una respuesta simulada. Este método permite a los desarrolladores continuar con la implementación y las pruebas de la instancia de APIM en caso de que el back-end no esté disponible para enviar respuestas reales. 

La funcionalidad de simulación de respuestas puede resultar útil en varios escenarios:

+ Cuando la fachada de API se ha diseñado primero y la implementación de back-end se realiza más tarde. El back-end se está desarrollando en paralelo.
+ Cuando el back-end no está temporalmente operativo o no se puede escalar.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una API de prueba 
> * Adición de una operación a la API de prueba
> * Habilitación de la simulación de respuesta
> * Probar la API simulada


:::image type="content" source="media/mock-api-responses/mock-api-responses01.png" alt-text="Respuesta de API simulada":::

## <a name="prerequisites"></a>Prerrequisitos

+ Conocer la [terminología de API Management de Azure](api-management-terminology.md).
+ Comprender el [concepto de directivas en API Management de Azure](api-management-howto-policies.md).
+ Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).

## <a name="create-a-test-api"></a>Creación de una API de prueba 

Los pasos de esta sección muestran cómo crear una API en blanco sin back-end. 


1. Inicie sesión en Azure Portal y vaya a la instancia de API Management.
1. Seleccione **API** >  **+ Agregar API** > **API en blanco**.
1. En la ventana **Crear una API en blanco**, seleccione **Completa**.
1. En **Nombre para mostrar**, escriba *API de prueba*.
1. En **Productos **, seleccione** Ilimitado**.
1. Compruebe que, en **Puertas de enlace**, está seleccionada la opción **Administrada**.
1. Seleccione **Crear**.

    :::image type="content" source="media/mock-api-responses/03-mock-api-responses-01-create-test-api.png" alt-text="Crear API en blanco":::

## <a name="add-an-operation-to-the-test-api"></a>Adición de una operación a la API de prueba

Una API expone una o varias operaciones. En esta sección, agregue una operación a la API en blanco que creó. Al llamar a la operación después de completar los pasos de esta sección, se produce un error. Dejará de recibir errores después de realizar los pasos de la sección [Habilitación de la simulación de respuesta](#enable-response-mocking), más adelante.

### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Seleccione la API que creó en los pasos anteriores.
1. Seleccione **+ Agregar operación**.
1. En la ventana **Front-end**, escriba los siguientes valores.

     | Configuración             | Value                             | Descripción                                                                                                                                                                                   |
    |---------------------|-----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | **Nombre para mostrar**    | *Llamada de prueba*                       | El nombre se muestra en el [portal para desarrolladores](api-management-howto-developer-portal.md).                                                                                                                                       |
    | **URL** (verbo HTTP) | GET                               | Seleccione uno de los verbos HTTP predefinidos.                                                                                                                                         |
    | **URL**             | */test*                           | Una ruta de acceso URL de la API.                                                                                                                                                                       |
    | **Descripción**     |                                   |  Descripción opcional de la operación que se usa para proporcionar documentación en el portal para desarrolladores a aquellos que usan esta API.                                                    |
    
1. Seleccione la pestaña **Respuestas**, que se encuentra bajo los campos URL, Nombre para mostrar y Descripción. Especifique la configuración de esta pestaña para definir códigos de estado de respuesta, tipos de contenido, ejemplos y esquemas.
1. Seleccione **+ Agregar respuesta** y, en la lista, elija **200 - Correcto**.
1. En el encabezado **Representaciones** de la derecha, seleccione **+ Agregar representación**.
1. Escriba *application/json* en el cuadro de búsqueda y seleccione el tipo de contenido **application/json**.
1. En el cuadro de texto **Ejemplo**, escriba `{ "sampleField" : "test" }`.
1. Seleccione **Guardar**.

:::image type="content" source="media/mock-api-responses/03-mock-api-responses-02-add-operation.png" alt-text="Adición de operación de API" border="false":::

Aunque no es necesario en este ejemplo, se pueden configurar opciones adicionales para una operación de API en otras pestañas, por ejemplo:


|Pestaña      |Descripción  |
|---------|---------|
|**Consultar**     |  Agregue parámetros de consulta. Además de proporcionar un nombre y una descripción, puede especificar valores que se asignan a un parámetro de consulta. Uno de los valores se puede marcar como predeterminado (opcional).        |
|**Solicitud**     |  Defina esquemas, ejemplos y tipos de contenido de solicitud.       |

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para empezar a usar la CLI de Azure:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Para agregar una operación a la API de prueba, ejecute el comando [az apim api operation create](/cli/azure/apim/api/operation#az_apim_api_operation_create):

```azurecli
az apim api operation create --resource-group apim-hello-word-resource-group \
    --display-name "Test call" --api-id test-api --method GET \
    --url-template /test --service-name apim-hello-world 
```

Ejecute el comando [az apim api operation list](/cli/azure/apim/api/operation#az_apim_api_operation_list) para ver todas las operaciones de una API:

```azurecli
az apim api operation list --resource-group apim-hello-word-resource-group \
    --api-id test-api --service-name apim-hello-world --output table
```

Para quitar una operación, use el comando [az apim api operation delete](/cli/azure/apim/api/operation#az_apim_api_operation_delete). Obtenga el identificador de operación del comando anterior.

```azurecli
az apim api operation delete --resource-group apim-hello-word-resource-group \
    --api-id test-api --operation-id 00000000000000000000000000000000 \
    --service-name apim-hello-world
```

Mantenga esta operación para utilizarla en el resto de este artículo.

---

## <a name="enable-response-mocking"></a>Habilitación de la simulación de respuesta

1. Seleccione la API que creó en [Creación de una API de prueba](#create-a-test-api).
1. Seleccione la operación de prueba que agregó.
1. En la ventana de la derecha, asegúrese de que esté seleccionada la pestaña **Diseño**.
1. En la ventana **Procesamiento de entrada**, seleccione **+ Agregar directiva**.

    :::image type="content" source="media/mock-api-responses/03-mock-api-responses-03-enable-mocking.png" alt-text="Adición de directiva de procesamiento" border="false":::

1. En la galería, seleccione **Mock responses** (Simular respuestas).

    :::image type="content" source="media/mock-api-responses/mock-responses-policy-tile.png" alt-text="Icono de directiva de respuestas simuladas" border="false":::

1. En el cuadro de texto **API Management response** (Respuesta de API Management), escriba **200 OK, application/json**. Esta selección indica que la API debe devolver la respuesta de ejemplo que definió en la sección anterior.

    :::image type="content" source="media/mock-api-responses/mock-api-responses-set-mocking.png" alt-text="Establecimiento de respuestas simuladas":::

1. Seleccione **Guardar**.

    > [!TIP]
    > Una barra amarilla con el texto **La simulación de respuesta está habilitada** en la API indica que las respuestas devueltas por API Management son simuladas por la [directiva de simulación](api-management-advanced-policies.md#mock-response) y no las genera el back-end.

## <a name="test-the-mocked-api"></a>Probar la API simulada

1. Seleccione la API que creó en [Creación de una API de prueba](#create-a-test-api).
1. Seleccione la pestaña **Prueba**.
1. Asegúrese de que la API de **Llamada de prueba** está seleccionada. Seleccione **Enviar** para realizar una llamada de prueba.

   :::image type="content" source="media/mock-api-responses/03-mock-api-responses-04-test-mocking.png" alt-text="Probar la API simulada":::

1. **Respuesta HTTP** muestra el JSON especificado como un ejemplo en la primera sección del tutorial.

    :::image type="content" source="media/mock-api-responses/mock-api-responses-test-response.png" alt-text="Simulación de respuestas HTTP":::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Creación de una API de prueba
> * Adición de una operación a la API de prueba
> * Habilitación de la simulación de respuesta
> * Probar la API simulada

Avance hasta el siguiente tutorial:

> [!div class="nextstepaction"]
> [Transformación y protección de una API publicada](transform-api.md)
