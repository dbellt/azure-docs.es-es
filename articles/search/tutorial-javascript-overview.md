---
title: 'Tutorial de JavaScript: Introducción a la integración de búsqueda'
titleSuffix: Azure Cognitive Search
description: Información general técnica y configuración para agregar búsquedas a un sitio web y su implementación en Azure Static Web Apps.
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 03/18/2021
ms.custom: devx-track-js
ms.devlang: javascript
ms.openlocfilehash: da341d202dcfd0fc81a6becec6646b1116069aa7
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108123292"
---
# <a name="1---overview-of-adding-search-to-a-website"></a>1\. Introducción a la adición de búsquedas a un sitio web

En este tutorial se crea un sitio web para buscar en un catálogo de libros y, a continuación, se implementa el sitio web en una aplicación web estática de Azure. 

La aplicación está disponible: 
* [Ejemplo](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/search-website)
* [Sitio web de demostración: aka.ms/azs-good-books](https://aka.ms/azs-good-books)

## <a name="what-does-the-sample-do"></a>¿Qué hace el ejemplo? 

En este sitio web de ejemplo se proporciona acceso a un catálogo de 10 000 libros. Un usuario puede buscar en el catálogo escribiendo un texto en la barra de búsqueda. Mientras el usuario escribe el texto, el sitio web usa la característica de sugerencias del índice de búsqueda para completar el texto. Una vez finalizada la consulta, se muestra la lista de libros con una parte de los detalles. Un usuario puede seleccionar un libro para ver todos los detalles del libro, almacenados en el índice de búsqueda. 

:::image type="content" source="./media/tutorial-javascript-overview/cognitive-search-enabled-book-website.png" alt-text="En este sitio web de ejemplo se proporciona acceso a un catálogo de 10 000 libros. Un usuario puede buscar en el catálogo escribiendo un texto en la barra de búsqueda. Mientras el usuario escribe el texto, el sitio web usa la característica de sugerencias del índice de búsqueda para completar el texto. Una vez finalizada la búsqueda, se muestra la lista de libros con una parte de los detalles. Un usuario puede seleccionar un libro para ver todos los detalles del libro, almacenados en el índice de búsqueda.":::

La experiencia de búsqueda incluye: 

* Búsqueda: proporciona la funcionalidad de búsqueda para la aplicación.
* Sugerencias: proporciona sugerencias a medida que el usuario escribe en la barra de búsqueda.
* Búsqueda de documentos: busca un documento por identificador para recuperar todo su contenido para la página de detalles.

## <a name="how-is-the-sample-organized"></a>¿Cómo se organiza el ejemplo?

El [ejemplo](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/search-website) incluye lo siguiente:

|Aplicación|Propósito|GitHub<br>Repositorio<br>Location|
|--|--|--|
|Remoto|Aplicación de React (capa de presentación) para mostrar los libros, con la búsqueda. Llama a la aplicación de funciones de Azure. |[/search-website/src](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/search-website/src)|
|Servidor|Aplicación de funciones de Azure (capa de negocio): llama a Azure Cognitive Search API mediante el SDK para JavaScript |[/search-website/api](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/search-website/src)|
|Inserción masiva|Archivo JavaScript para crear el índice y agregarle documentos.|[/search-website/bulk-insert](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/search-website/bulk-insert)|

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo

Instale lo siguiente para el entorno de desarrollo local. 

- [Node.js 12 o 14](https://nodejs.org/en/download)
    - Si tiene otra versión de Node.js instalada en el equipo local, considere la posibilidad de usar el [Administrador de versiones de Node](https://github.com/nvm-sh/nvm) (nvm) o un contenedor de Docker.  
- [Git](https://git-scm.com/downloads)
- [Visual Studio Code](https://code.visualstudio.com/) y las siguientes extensiones
    - [Recursos de Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureresourcegroups)
    - [Azure Cognitive Search 0.2.0+](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurecognitivesearch)
    - [Azure Static Web Apps](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestaticwebapps) 
- Opcional:
    - En este tutorial no se ejecuta la API de Azure Functions localmente, pero si piensa ejecutarla localmente, debe instalar [azure-functions-core-tools](../azure-functions/functions-run-local.md?tabs=linux%2ccsharp%2cbash) globalmente con el siguiente comando de Bash: 
    
    ```bash
    npm install -g azure-functions-core-tools
    ```

## <a name="fork-and-clone-the-search-sample-with-git"></a>Bifurcación y clonación del ejemplo de búsqueda con git

La bifurcación del repositorio de ejemplo es fundamental para poder implementar la aplicación web estática. Las aplicaciones web determinan las acciones de compilación y el contenido de implementación en función de su propia ubicación de la bifurcación de GitHub. La ejecución de código en la aplicación web estática es remota, Azure Static Web Apps lee desde el código del ejemplo bifurcado.

1. En GitHub, bifurque el [repositorio de ejemplo](https://github.com/Azure-Samples/azure-search-javascript-samples). 

    Complete el proceso de bifurcación en el explorador web con su cuenta de GitHub. En este tutorial se usa la bifurcación como parte de la implementación en una aplicación web estática de Azure. 

1. En un terminal de Bash, descargue la aplicación de ejemplo en el equipo local. 

    Reemplace `YOUR-GITHUB-ALIAS` por su alias de GitHub. 

    ```bash
    git clone https://github.com/YOUR-GITHUB-ALIAS/azure-search-javascript-samples
    ```

1. En Visual Studio Code, abra la carpeta local del repositorio clonado. Las tareas restantes se realizan desde Visual Studio Code, a menos que se especifique.

## <a name="create-a-resource-group-for-your-azure-resources"></a>Creación de un grupo de recursos para los recursos de Azure

1. En Visual Studio Code, abra la [barra de actividad](https://code.visualstudio.com/docs/getstarted/userinterface) y seleccione el icono de Azure. 
1. En la barra lateral, **haga clic con el botón derecho en la suscripción de Azure** en el área `Resource Groups` y seleccione **Create resource group** (Crear grupo de recursos).

    :::image type="content" source="./media/tutorial-javascript-overview/visual-studio-code-create-resource-group.png" alt-text="En la barra lateral, haga clic con el botón derecho en la suscripción de Azure en el área de grupos de recursos y seleccione **Crear grupo de recursos**.":::
1. Escriba el nombre de un grupo de recursos; por ejemplo, `cognitive-search-website-tutorial`. 
1. Seleccione una ubicación cercana.
1. Al crear los recursos de Cognitive Search y Static Web Apps más adelante en el tutorial, use este grupo de recursos. 

    La creación de un grupo de recursos proporciona una unidad lógica para administrar los recursos, lo que incluye eliminarlos cuando haya terminado de usarlos.

## <a name="next-steps"></a>Pasos siguientes

* [Creación y carga de un índice de búsqueda con JavaScript](tutorial-javascript-create-load-index.md)
* [Implementación del sitio web habilitado para búsqueda](tutorial-javascript-deploy-static-web-app.md)