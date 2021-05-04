---
title: 'Tutorial de .NET: incorporación de funcionalidad de búsqueda a las aplicaciones web'
titleSuffix: Azure Cognitive Search
description: Cree un índice e importe datos CSV al índice de búsqueda con .NET.
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 04/23/2021
ms.custom: devx-track-csharp
ms.devlang: dotnet
ms.openlocfilehash: b7bbf9f1ec572680f02e4b6d3adca6d807b45f83
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107981996"
---
# <a name="2---create-and-load-search-index-with-net"></a>2\. Creación y carga de un índice de búsqueda con .NET

Continúe con la creación del sitio web habilitado para la búsqueda mediante:
* Creación de un recurso Search con la extensión VS Code
* Creación de un nuevo índice e importación de datos con .NET mediante el script de ejemplo y [Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/) del SDK de Azure.

## <a name="create-an-azure-search-resource"></a>Creación de un recurso de Azure Search 

Cree un nuevo recurso de búsqueda con la extensión de [Azure Cognitive Search](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurecognitivesearch) para Visual Studio Code.

1. En Visual Studio Code, abra la [barra de actividad](https://code.visualstudio.com/docs/getstarted/userinterface) y seleccione el icono de Azure. 

1. En la barra lateral, **haga clic con el botón derecho en la suscripción de Azure** en el área `Azure: Cognitive Search` y seleccione **Create new search service** (Crear nuevo servicio de búsqueda).

    :::image type="content" source="./media/tutorial-javascript-create-load-index/visual-studio-code-create-search-resource.png" alt-text="En la barra lateral, haga clic con el botón derecho en la suscripción de Azure en el área **Azure: Cognitive Search** y seleccione **Create new search service** (Crear nuevo servicio de búsqueda).":::

1. Siga los mensajes para proporcionar la siguiente información:

    |Prompt|Entrar|
    |--|--|
    |Escriba un nombre único global para el nuevo servicio Search.|**Recuerde este nombre**. Este nombre de recurso se convierte en parte del punto de conexión del recurso.|
    |Seleccionar un grupo de recursos para los nuevos recursos|Utilice el grupo de recursos que ha creado para este tutorial.|
    |Seleccione la SKU del servicio Search.|Seleccione **Gratis** para este tutorial. No se puede cambiar un plan de tarifa de SKU una vez creado el servicio.|
    |Seleccione una ubicación para los nuevos recursos.|Seleccione una región cercana.|

1. Después de completar las solicitudes, se crea el nuevo recurso Search. 

## <a name="get-your-search-resource-admin-key"></a>Obtención de la clave de administración del recurso Search

Obtenga la clave de administración del recurso Search con la extensión Visual Studio Code. 

1. En Visual Studio Code, en la barra lateral, haga clic con el botón derecho en el recurso Search y seleccione **Copy Admin Key** (Copiar clave de administración).

    :::image type="content" source="./media/tutorial-javascript-create-load-index/visual-studio-code-copy-admin-key.png" alt-text="En la barra lateral, haga clic con el botón derecho en el recurso Search y seleccione **Copy Admin Key** (Copiar clave de administración).":::

1. Mantenga esta clave de administración, ya que tendrá que utilizarla en [una sección posterior](#prepare-the-bulk-import-script-for-search). 

## <a name="prepare-the-bulk-import-script-for-search"></a>Preparación del script de importación en bloque para Search

El script utiliza el SDK de Azure para Cognitive Search:

* [Paquete NuGet Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/)
* [Documentación de referencia](/dotnet/api/overview/azure/search)

1. En Visual Studio Code, abra el archivo `Program.cs` del subdirectorio, `search-website/bulk-insert`, y reemplace las siguientes variables por sus propios valores para autenticarse con el SDK de Azure Search:

    * YOUR-SEARCH-RESOURCE-NAME
    * YOUR-SEARCH-ADMIN-KEY

    :::code language="csharp" source="~/azure-search-dotnet-samples/search-website/bulk-insert/Program.cs" highlight="16-19" :::

1. Abra un terminal integrado en Visual Studio Code para el subdirectorio del directorio del proyecto, `search-website/bulk-insert`, y ejecute el siguiente comando para instalar las dependencias. 

    ```bash
    dotnet restore
    ```

## <a name="run-the-bulk-import-script-for-search"></a>Ejecución del script de importación en bloque para Search

1. Siga utilizando el terminal integrado en Visual Studio para el subdirectorio del directorio del proyecto, `search-website/bulk-insert`, a fin de ejecutar el siguiente comando de Bash para ejecutar el script `Program.cs`:

    ```bash
    dotnet run
    ```

1. A medida que se va ejecutando el código, la consola muestra el progreso. 
1. Una vez que se ha completado la carga, la última instrucción impresa en la consola es "Finished bulk inserting book data" (Inserción masiva de datos de libro finalizada).

## <a name="review-the-new-search-index"></a>Revisión del nuevo índice de búsqueda

Una vez finalizada la carga, el índice de búsqueda está preparado para su uso. Revise el nuevo índice.

1. En Visual Studio Code, abra la extensión Azure Cognitive Search y seleccione el recurso Search.  

    :::image type="content" source="media/tutorial-javascript-create-load-index/visual-studio-code-search-extension-view-resource.png" alt-text="En Visual Studio Code, abra la extensión Azure Cognitive Search y, después, el recurso Search.":::

1. Expanda Índices, Documentos y `good-books`, y seleccione un documento para ver todos sus datos específicos.
 
    :::image type="content" source="media/tutorial-javascript-create-load-index/visual-studio-code-search-extension-view-docs.png" lightbox="media/tutorial-javascript-create-load-index/visual-studio-code-search-extension-view-docs.png" alt-text="Expanda Índices, luego &quot;good-books&quot; y, a continuación, seleccione un documento.":::

## <a name="copy-your-search-resource-name"></a>Copia del nombre del recurso Search

Anote el **nombre del recurso Search**. Lo necesitará para conectar la aplicación Función de Azure a su recurso Search. 

> [!CAUTION]
> Aunque es posible que se sienta tentado a utilizar la clave de administración de búsqueda en Función de Azure, no sigue el principio de privilegio mínimo. Función de Azure usará la clave de consulta para ajustarse al privilegio mínimo. 

## <a name="next-steps"></a>Pasos siguientes

[Implementación de la aplicación web estática](tutorial-csharp-deploy-static-web-app.md)