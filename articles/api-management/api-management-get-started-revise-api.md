---
title: 'Tutorial: Uso de revisiones para realizar cambios de API que no producen interrupciones de forma segura en API Management'
titleSuffix: Azure API Management
description: Siga los pasos de este tutorial para obtener información sobre cómo realizar cambios que no producen interrupciones a través de revisiones en API Management.
services: api-management
documentationcenter: ''
author: vladvino
ms.service: api-management
ms.custom: mvc, devx-track-azurecli
ms.topic: tutorial
ms.date: 02/09/2021
ms.author: apimpm
ms.openlocfilehash: 1d99d6f876e4896bb4321afb8dc4d8e7c3a404e7
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107483595"
---
# <a name="tutorial-use-revisions-to-make-non-breaking-api-changes-safely"></a>Tutorial: Uso de revisiones para realizar cambios de API que no producen interrupciones de forma segura
Cuando la API esté lista y los desarrolladores empiecen a usarla, tendrá que realizar cambios en dicha API y, al mismo tiempo, no interrumpir a quienes la llaman. También resulta útil informar a los desarrolladores de los cambios realizados. 

En Azure API Management, use *revisiones* para realizar cambios de API que no producen interrupciones para que pueda modelar y probar los cambios de forma segura. Cuando esté listo, puede hacer una revisión actualizada y reemplazar la API actual. 

Para más información, consulte [Versiones y revisiones](https://azure.microsoft.com/blog/versions-revisions/) y [Control de versiones con Azure API Management](https://azure.microsoft.com/blog/api-versioning-with-azure-api-management/).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar una nueva revisión
> * Realizar cambios que no producen interrupciones en la revisión
> * Convertir la revisión en actual y agregar una entrada en el registro de cambios
> * Examinar el portal para desarrolladores para ver los cambios y registro de cambios

:::image type="content" source="media/api-management-getstarted-revise-api/azure-portal.png" alt-text="Revisiones de API en Azure Portal":::

## <a name="prerequisites"></a>Requisitos previos

+ Conocer la [terminología de API Management de Azure](api-management-terminology.md).
+ Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
+ Además, realice el siguiente tutorial: [Importación y publicación de la primera API](import-and-publish.md).

## <a name="add-a-new-revision"></a>Agregar una nueva revisión

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la instancia de API Management.
1. Seleccione **API**.
2. Seleccione **Demo Conference API** en la lista de API (u otra API a la que desee agregar revisiones).
3. Seleccione la pestaña **Revisiones**.
4. Seleccione **+ Agregar revisión**.

   :::image type="content" source="media/api-management-getstarted-revise-api/07-add-revisions-01-add-new-revision.png" alt-text="Agregar revisión de API":::

    > [!TIP]
    > También puede seleccionar **Agregar revisión** en el menú contextual ( **...** ) de la API.

5. Incluya una descripción de la nueva revisión que ayude a saber para qué va a servir.
6. Seleccione **Crear**,
7. Ya tenemos creada la revisión.

    > [!NOTE]
    > La API original permanece en **Revisión 1**. Esta es la revisión que los usuarios siguen llamando hasta que decida convertir otra en actual.

## <a name="make-non-breaking-changes-to-your-revision"></a>Realizar cambios que no producen interrupciones en la revisión

1. Seleccione **Demo Conference API** en la lista de API.
1. Seleccione la pestaña **Diseño** situada junto a la parte superior de la pantalla.
1. Fíjese en que el **selector de revisiones** (que está inmediatamente encima de la pestaña de diseño) muestra que la revisión seleccionada es **Revisión 2**.

    > [!TIP]
    > Use el selector de revisión para alternar entre las revisiones con las que vaya a trabajar.
1. Seleccione **+ Agregar operación**.
1. Establezca que la nueva operación sea **POST** y especifique **test** como nombre, nombre para mostrar y dirección URL de la operación.
1. **Guarde** la nueva operación.

   :::image type="content" source="media/api-management-getstarted-revise-api/07-add-revisions-02-make-changes.png" alt-text="Modificar revisión":::
1. Ya ha hecho un cambio en **Revisión 2**. Use el **selector de revisión** junto a la parte superior de la página para volver a **Revisión 1**.
1. Observe que la nueva operación no figura en **Revisión 1**. 

## <a name="make-your-revision-current-and-add-a-change-log-entry"></a>Convertir la revisión en actual y agregar una entrada en el registro de cambios

### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Seleccione la pestaña **Revisiones** en el menú junto a la parte superior de la página.
1. Abra el menú contextual (**...**) de **Revisión 2**.
1. Seleccione **Convertir en actual**.
1. Seleccione la casilla **Post to Public Change log for this API** (Publicar en el registro de cambios público de esta API), si quiere publicar notas sobre este cambio. Especifique una descripción del cambio que puedan ver los desarrolladores, por ejemplo: **Revisiones de prueba. Se agregó una nueva operación de "prueba".**
1. **Revisión 2** es ahora la revisión actual.

    :::image type="content" source="media/api-management-getstarted-revise-api/revisions-menu.png" alt-text="Menú Revisión en la ventana Revisiones":::

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para empezar a usar la CLI de Azure:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Utilice este procedimiento para crear y actualizar una versión.

1. Ejecute el comando [az apim api list](/cli/azure/apim/api#az_apim_api_list) para ver los identificadores de API:

   ```azurecli
   az apim api list --resource-group apim-hello-word-resource-group \
       --service-name apim-hello-world --output table
   ```

   El identificador de API que se usará en el siguiente comando es el valor `Name`. La revisión de la API está en la columna `ApiRevision`.

1. Para crear la versión, con una nota de la versión, ejecute el comando [az apim api release create](/cli/azure/apim/api/release#az_apim_api_release_create):

   ```azurecli
   az apim api release create --resource-group apim-hello-word-resource-group \
       --api-id demo-conference-api --api-revision 2 --service-name apim-hello-world \
       --notes 'Testing revisions. Added new "test" operation.'
   ```

   La revisión que se publica se convierte en la revisión actual.

1. Para ver las versiones, use el comando [az apim api release list](/cli/azure/apim/api/release#az_apim_api_release_list):

   ```azurecli
   az apim api release list --resource-group apim-hello-word-resource-group \
       --api-id echo-api --service-name apim-hello-world --output table
   ```

   Las notas que especifique aparecen en el registro de cambios. Puede verlas en la salida del comando anterior.

1. Cuando crea una versión, el parámetro `--notes` es opcional. Puede agregar o cambiar las notas más adelante mediante el comando [az apim api release update](/cli/azure/apim/api/release#az_apim_api_release_update):

   ```azurecli
   az apim api release update --resource-group apim-hello-word-resource-group \
       --api-id demo-conference-api --release-id 00000000000000000000000000000000 \
       --service-name apim-hello-world --notes "Revised notes."
   ```

   Use el valor de la columna `Name` como identificador de la versión.

Puede quitar cualquier versión ejecutando el comando [az apim api release delete ](/cli/azure/apim/api/release#az_apim_api_release_delete):

```azurecli
az apim api release delete --resource-group apim-hello-word-resource-group \
    --api-id demo-conference-api --release-id 00000000000000000000000000000000 
    --service-name apim-hello-world
```

---

## <a name="browse-the-developer-portal-to-see-changes-and-change-log"></a>Examinar el portal para desarrolladores para ver los cambios y registro de cambios

Si ha probado [el portal para desarrolladores](api-management-howto-developer-portal-customize.md), puede revisar los cambios de la API y el registro de cambios ahí.

1. En Azure Portal, seleccione **API**.
1. Seleccione **Portal para desarrolladores** en el menú superior.
1. En el portal para desarrolladores, seleccione **API** y, después, **Demo Conference API**.
1. Observe que la nueva operación **prueba** ahora aparece disponible.
1. Seleccione **Registro de cambios** cerca del nombre de la API.
1. Vea que la entrada en el registro de cambios aparece en esta lista.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Agregar una nueva revisión
> * Realizar cambios que no producen interrupciones en la revisión
> * Convertir la revisión en actual y agregar una entrada en el registro de cambios
> * Examinar el portal para desarrolladores para ver los cambios y registro de cambios

Avance hasta el siguiente tutorial:

> [!div class="nextstepaction"]
> [Publicación de varias versiones de la API](api-management-get-started-publish-versions.md)
