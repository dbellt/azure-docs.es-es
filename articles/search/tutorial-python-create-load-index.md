---
title: 'Tutorial de Python: incorporación de funcionalidad de búsqueda a las aplicaciones web'
titleSuffix: Azure Cognitive Search
description: Cree un índice e importe datos en formato CSV en el índice de búsqueda con Python mediante azure-search-documents del SDK del paquete de PYPI.
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 05/25/2021
ms.custom: devx-track-python
ms.devlang: python
ms.openlocfilehash: a9678e542e577c13141f18e59fe2e628ed465321
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110580672"
---
# <a name="2---create-and-load-search-index-with-python"></a>2\. Creación y carga de un índice de búsqueda con Python

Continúe con la creación del sitio web habilitado para la búsqueda mediante:
* Creación de un recurso Search con la extensión VS Code
* Creación de un nuevo índice e importación de datos con Python mediante el script de ejemplo y [azure-search-documents](https://pypi.org/project/azure-search-documents/) del SDK de Azure.

## <a name="create-an-azure-cognitive-search-resource"></a>Creación de un recurso de Azure Cognitive Search 

Cree un nuevo recurso de búsqueda con la extensión de [Azure Cognitive Search](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurecognitivesearch) para Visual Studio Code.

1. En Visual Studio Code, abra la [barra de actividad](https://code.visualstudio.com/docs/getstarted/userinterface) y seleccione el icono de Azure. 

1. En la barra lateral, **haga clic con el botón derecho en la suscripción de Azure** en el área `Azure: Cognitive Search` y seleccione **Create new search service** (Crear nuevo servicio de búsqueda).

    :::image type="content" source="./media/tutorial-javascript-create-load-index/visual-studio-code-create-search-resource.png" alt-text="En la barra lateral, haga clic con el botón derecho en la suscripción de Azure en el área **Azure: Cognitive Search** y seleccione **Create new search service** (Crear nuevo servicio de búsqueda).":::

1. Siga los mensajes para proporcionar la siguiente información:

    |Prompt|Entrar|
    |--|--|
    |Escriba un nombre único global para el nuevo servicio Search.|**Recuerde este nombre**. Este nombre de recurso se convierte en parte del punto de conexión del recurso.|
    |Seleccionar un grupo de recursos para los nuevos recursos|Utilice el grupo de recursos que ha creado para este tutorial.|
    |Seleccione la SKU del servicio Search.|Seleccione **Gratis** para este tutorial. No se puede cambiar un plan de tarifa una vez creado el servicio.|
    |Seleccione una ubicación para los nuevos recursos.|Seleccione una región cercana.|

1. Después de completar las solicitudes, se crea el nuevo recurso Search. 

## <a name="get-your-search-resource-admin-key"></a>Obtención de la clave de administración del recurso Search

Obtenga la clave de administración del recurso Search con la extensión Visual Studio Code. 

1. En Visual Studio Code, en la barra lateral, haga clic con el botón derecho en el recurso Search y seleccione **Copy Admin Key** (Copiar clave de administración).

    :::image type="content" source="./media/tutorial-javascript-create-load-index/visual-studio-code-copy-admin-key.png" alt-text="En la barra lateral, haga clic con el botón derecho en el recurso Search y seleccione **Copy Admin Key** (Copiar clave de administración).":::

1. Mantenga esta clave de administración, ya que tendrá que utilizarla para crear objetos en [una sección posterior](#prepare-the-bulk-import-script-for-search). 

## <a name="prepare-the-bulk-import-script-for-search"></a>Preparación del script de importación en bloque para Search

El script utiliza el SDK de Azure para Cognitive Search:

* [azure-search-documents de paquete NuGet](https://pypi.org/project/azure-search-documents/)
* [Documentación de referencia](/python/api/azure-search-documents)

1. En Visual Studio Code, abra el archivo `bulk_upload.py` del subdirectorio, `search-website/bulk-upload`, y reemplace las siguientes variables por sus propios valores para autenticarse con el SDK de Azure Search:

    * YOUR-SEARCH-RESOURCE-NAME
    * YOUR-SEARCH-ADMIN-KEY

    :::code language="python" source="~/azure-search-python-samples/search-website/bulk-upload/bulk-upload.py" highlight="12,13, 117" :::

1. Abra un terminal integrado en Visual Studio para el subdirectorio del directorio del proyecto, `search-website/bulk-upload`, y ejecute el siguiente comando para instalar las dependencias. 

    # <a name="macoslinux"></a>[macOS y Linux](#tab/linux-install)
    
    ```bash
    python3 -m pip install -r requirements.txt 
    ```
    
    # <a name="windows"></a>[Windows](#tab/windows-install)

    ```bash
    py -m pip install -r requirements.txt 
    ```

## <a name="run-the-bulk-import-script-for-search"></a>Ejecución del script de importación en bloque para Search

1. Siga utilizando el terminal integrado en Visual Studio para el subdirectorio del directorio del proyecto, `search-website/bulk-upload`, a fin de ejecutar el siguiente comando de Bash para ejecutar el script `bulk_upload.py`:

    # <a name="macoslinux"></a>[macOS y Linux](#tab/linux-run)
    
    ```bash
    python3 bulk-upload.py
    ```
    
    # <a name="windows"></a>[Windows](#tab/windows-run)

    ```bash
    py bulk-upload.py
    ```


1. A medida que se va ejecutando el código, la consola muestra el progreso. 
1. Cuando la carga se completa, la última instrucción impresa en la consola es "Done. Press any key to close the terminal" (Hecho. Pulse cualquier tecla para cerrar el terminal).

## <a name="review-the-new-search-index"></a>Revisión del nuevo índice de búsqueda

Una vez que la carga se completa, el índice de búsqueda está preparado para su uso. Examine el nuevo índice.

1. En Visual Studio Code, abra la extensión Azure Cognitive Search y seleccione el recurso Search.  

    :::image type="content" source="media/tutorial-javascript-create-load-index/visual-studio-code-search-extension-view-resource.png" alt-text="En Visual Studio Code, abra la extensión Azure Cognitive Search y, después, el recurso Search.":::

1. Expanda Índices, Documentos y `good-books`, y seleccione un documento para ver todos sus datos específicos.
 
    :::image type="content" source="media/tutorial-javascript-create-load-index/visual-studio-code-search-extension-view-docs.png" lightbox="media/tutorial-javascript-create-load-index/visual-studio-code-search-extension-view-docs.png" alt-text="Expanda Índices, luego &quot;good-books&quot; y, a continuación, seleccione un documento.":::

## <a name="copy-your-search-resource-name"></a>Copia del nombre del recurso Search

Anote el **nombre del recurso Search**. Lo necesitará para conectar la aplicación Función de Azure a su recurso Search. 

> [!CAUTION]
> Aunque es posible que se sienta tentado a utilizar la clave de administración de búsqueda en Función de Azure, no sigue el principio de privilegio mínimo. Función de Azure usará la clave de consulta para ajustarse al privilegio mínimo. 

## <a name="next-steps"></a>Pasos siguientes

[Implementación de la aplicación web estática](tutorial-python-deploy-static-web-app.md)