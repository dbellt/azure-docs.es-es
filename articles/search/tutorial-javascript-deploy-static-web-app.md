---
title: 'Tutorial de JavaScript: Implementación de un sitio web habilitado para búsquedas'
titleSuffix: Azure Cognitive Search
description: Implemente un sitio web habilitado para búsquedas en la aplicación web estática de Azure.
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 03/18/2021
ms.custom: devx-track-js
ms.devlang: javascript
ms.openlocfilehash: 021f7aeda86f5ff9f11eb2991a74c6ad37a203e6
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110469790"
---
# <a name="3---deploy-the-search-enabled-website"></a>3 - Implementación del sitio web habilitado para búsquedas

Implemente el sitio web habilitado para búsquedas como una aplicación web estática de Azure. Esta implementación incluye la aplicación React y la aplicación de funciones.  

La aplicación web estática extrae la información y los archivos para la implementación desde GitHub mediante la bifurcación del repositorio de ejemplos.  

## <a name="create-a-static-web-app-in-visual-studio-code"></a>Creación de una aplicación web estática en Visual Studio Code

1. Seleccione **Azure** en la barra de actividades y, a continuación, seleccione **Static Web Apps** en la barra lateral. 

1. Si ve una ventana emergente en VS Code que le pregunta desde qué rama quiere realizar la implementación, seleccione la rama predeterminada, normalmente **maestra** o **principal**. 

    Esta configuración significa que solo los cambios que se confirman en esa rama se implementan en la aplicación web estática. 

1. Si ve una ventana emergente que le pide que confirme los cambios, no lo haga. Los secretos del paso de importación masiva no se deben confirmar en el repositorio. 

    Para revertir los cambios, en VS Code, seleccione el icono Control de código fuente de la barra de actividad, elija cada archivo modificado en la lista Cambios y seleccione el icono **Descartar cambios**.

1. Haga clic con el botón derecho en el nombre de la suscripción y seleccione **Create Static Web App (Advanced)** [(Crear aplicación web estática (avanzada)].    

    :::image type="content" source="media/tutorial-javascript-create-load-index/visual-studio-code-create-static-web-app-resource-advanced.png" alt-text="Haga clic con el botón derecho en el nombre de la suscripción y seleccione **Create Static Web App (Advanced)** [(Crear aplicación web estática (avanzada)]":::.

1. Siga los mensajes para proporcionar la siguiente información:

    |Prompt|Entrar|
    |--|--|
    |Escriba el nombre de la nueva aplicación web estática.|Cree un nombre único para el recurso. Por ejemplo, puede anteponer su nombre al nombre del repositorio, como en `joansmith-azure-search-javascript-samples`. |
    |Seleccione un grupo de recursos para los nuevos recursos.|Utilice el grupo de recursos que ha creado para este tutorial.|
    |Seleccionar una SKU| En este tutorial, seleccione la SKU gratuita.|
    |Elija el valor preestablecido de compilación para configurar la estructura predeterminada del proyecto.|Seleccionar **Personalizado**|
    |Seleccione la ubicación del código de la aplicación.|`search-website`<br><br>Esta es la ruta de acceso, desde la raíz del repositorio hasta la aplicación web estática de Azure. |
    |Seleccione la ubicación del código de Azure Functions.|`search-website/api`<br><br>Esta es la ruta de acceso, desde la raíz del repositorio hasta la aplicación de funciones de Azure. |
    |Escriba la ruta de la salida de la compilación...|`build`<br><br>Esta es la ruta de acceso, desde la aplicación web estática de Azure hasta los archivos generados.|
    |Seleccione una ubicación para los nuevos recursos.|Seleccione una región cercana.|

1. Una vez creado el recurso, seleccione **Open Actions in GitHub** (Abrir acciones en GitHub) en las notificaciones. Se abrirá una ventana del explorador que apunta al repositorio bifurcado. 

    La lista de acciones indica que la aplicación web, el cliente y las funciones, se insertaron correctamente en la aplicación web estática de Azure. 

    Espere a que finalicen la compilación y la implementación antes de continuar. Esto puede tardar uno o dos minutos en completarse.

## <a name="get-cognitive-search-query-key-in-visual-studio-code"></a>Obtención de la clave de consulta de Cognitive Search en Visual Studio Code

1. En Visual Studio Code, abra la [barra de actividad](https://code.visualstudio.com/docs/getstarted/userinterface) y seleccione el icono de Azure. 

1. En la barra lateral, seleccione su suscripción de Azure en el área **Azure: Cognitive Search**, haga clic con el botón derecho en el recurso de búsqueda y seleccione **Copy Query Key** (Copiar clave de consulta). 

    :::image type="content" source="./media/tutorial-javascript-create-load-index/visual-studio-code-copy-query-key.png" alt-text="En la barra lateral, seleccione su suscripción de Azure en el área **Azure: Cognitive Search**, haga clic con el botón derecho en el recurso de búsqueda y seleccione **Copy Query Key** (Copiar clave de consulta)":::.

1. Conserve esta clave de consulta, tendrá que utilizarla en la sección siguiente. La clave de consulta puede realizar consultas en el índice. 

## <a name="add-configuration-settings-in-azure-portal"></a>Adición de valores de configuración en Azure Portal

La aplicación de funciones de Azure no devolverá datos de Search hasta que los secretos de Search estén en la configuración. 

1. Seleccione **Azure** en la barra de actividades. 
1. Haga clic con el botón derecho en el recurso de la aplicación web estática y seleccione **Abrir en el portal**.

    :::image type="content" source="media/tutorial-javascript-static-web-app/open-static-web-app-in-azure-portal.png" alt-text="Haga clic con el botón derecho en el recurso de la aplicación web estática de JavaScript y seleccione Abrir en el portal":::.

1. Seleccione **Configuración** y, a continuación, seleccione **+ Agregar**.

    :::image type="content" source="media/tutorial-javascript-static-web-app/add-new-application-setting-to-static-web-app-in-portal.png" alt-text="Seleccione Configuración y, a continuación, seleccione Agregar para la aplicación JavaScript.":::

1. Agregue cada una de las siguientes opciones:

    |Valor|El valor del recurso de Search|
    |--|--|
    |SearchApiKey|La clave de consulta de Search|
    |SearchServiceName|El nombre del recurso de Search|
    |SearchIndexName|`good-books`|
    |SearchFacets|`authors*,language_code`|

    Azure Cognitive Search requiere sintaxis diferentes para filtrar colecciones y cadenas. Agregue un `*` después de un nombre de campo para indicar que el campo es de tipo `Collection(Edm.String)`. Esto permite que la función de Azure agregue filtros a las consultas correctamente.

1. Haga clic en **Save** (Guardar) para guardar la configuración. 

    :::image type="content" source="media/tutorial-javascript-static-web-app/save-new-application-setting-to-static-web-app-in-portal.png" alt-text="Seleccione Guardar para guardar la configuración.":::

1. Vuelva a VS Code. 
1. Actualice la aplicación web estática para ver su configuración. 

    :::image type="content" source="media/tutorial-javascript-static-web-app/visual-studio-code-extension-fresh-resource.png" alt-text="Actualice la aplicación web estática para ver su configuración.":::

## <a name="use-search-in-your-static-web-app"></a>Uso de Search en la aplicación web estática

1. En Visual Studio Code, abra la [barra de actividad](https://code.visualstudio.com/docs/getstarted/userinterface) y seleccione el icono de Azure.
1. En la barra lateral, **haga clic con el botón derecho en la suscripción de Azure** en el área `Static web apps` y busque la aplicación web estática que creó para este tutorial.
1. Haga clic con el botón derecho en el nombre de la aplicación web estática y seleccione **Browse site** (Examinar sitio).
    
    :::image type="content" source="media/tutorial-javascript-create-load-index/visual-studio-code-browse-static-web-app.png" alt-text="Haga clic con el botón derecho en el nombre de la aplicación web estática y seleccione **Browse site** (Examinar sitio).":::    

1. Seleccione **Open** (Abrir) en el cuadro de diálogo emergente.
1. En la barra de búsqueda de sitios web, escriba una consulta de búsqueda como `code`, _lentamente_, de modo que la característica de sugerencia sugiera títulos de libros. Seleccione una sugerencia o continúe escribiendo su propia consulta. Presione ENTRAR cuando haya completado la consulta de búsqueda. 
1. Revise los resultados y, a continuación, seleccione uno de los libros para ver más detalles. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Para borrar los recursos que ha creado en este tutorial, elimine el grupo de recursos.

1. En Visual Studio Code, abra la [barra de actividad](https://code.visualstudio.com/docs/getstarted/userinterface) y seleccione el icono de Azure. 

1. En la barra lateral, **haga clic con el botón derecho en la suscripción de Azure** en el área `Resource Groups` y busque el grupo de recursos que creó para este tutorial.
1. Haga clic con el botón derecho en el nombre del grupo de recursos y seleccione **Eliminar**.
    Esto elimina los recursos de Search y de la aplicación web estática.
1. Si ya no desea la bifurcación de GitHub del ejemplo, no olvide eliminarla en GitHub. Vaya a la **configuración** de la bifurcación y elimínela. 


## <a name="next-steps"></a>Pasos siguientes

* [Descripción de la integración de Search con el sitio web habilitado para búsqueda](tutorial-javascript-search-query-integration.md)
