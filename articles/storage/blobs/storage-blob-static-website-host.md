---
title: 'Tutorial: Hospedaje de un sitio web estático en Blob Storage: Azure Storage'
description: Aprenda a configurar una cuenta de almacenamiento para hospedar sitios web estáticos e implementar un sitio web estático en Azure Storage.
author: normesta
ms.service: storage
ms.subservice: blobs
ms.topic: tutorial
ms.date: 1/22/2020
ms.author: normesta
ms.reviewer: dineshm
ms.openlocfilehash: 0a8dfdea8ad743d0355daa16cc10f60bef96c213
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112081864"
---
<!---Customer intent: I want to host files for a static website in Blob storage and access the website from an Azure endpoint.--->

# <a name="tutorial-host-a-static-website-on-blob-storage"></a>Tutorial: Hospedaje de un sitio web estático en Blob Storage

En este tutorial, aprenderá a crear e implementar un sitio web estático para Azure Storage. Cuando haya terminado, tendrá un sitio web estático al que los usuarios podrán acceder públicamente. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar el hospedaje del sitio web estático
> * Implementar un sitio web de Hola mundo

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

> [!NOTE] 
> Los sitios web estáticos ahora están disponibles para las cuentas de almacenamiento estándar de uso general v2, así como para las cuentas de almacenamiento con el espacio de nombres jerárquico habilitado.

En este tutorial se utiliza [Visual Studio Code](https://code.visualstudio.com/download), una herramienta gratuita para programadores, para crear el sitio web estático e implementarlo en una cuenta de Azure Storage.

Después de instalar Visual Studio Code, instale la extensión de versión preliminar de Azure Storage. Esta extensión integra la funcionalidad de administración de Azure Storage con Visual Studio Code. Utilizará la extensión para implementar el sitio web estático en Azure Storage. Para instalar la extensión:

1. Inicie Visual Studio Code.
2. En la barra de herramientas, haga clic en **Extensiones**. Busque *Azure Storage* y seleccione la extensión **Azure Storage** en la lista. A continuación, haga clic en el botón **Instalar** para instalar la extensión.

    ![Instalación de la extensión de Azure Storage en VS Code](media/storage-blob-static-website-host/install-extension-vs-code.png)

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/) para empezar a trabajar.

## <a name="configure-static-website-hosting"></a>Configurar el hospedaje del sitio web estático

El primer paso es configurar la cuenta de almacenamiento para hospedar un sitio web estático en Azure Portal. Cuando configura la cuenta para el hospedaje de sitios web estáticos, Azure Storage crea automáticamente un contenedor llamado *$web*. El contenedor *$web* contendrá los archivos del sitio web estático. 

1. Abra [Azure Portal](https://portal.azure.com/) en el explorador web. 
1. Busque la cuenta de almacenamiento y muestre la información general de la cuenta.
1. Seleccione **Sitio web estático** para mostrar la página de configuración de los sitios web estáticos.
1. Seleccione **Habilitado** para habilitar el hospedaje del sitio web estático para la cuenta de almacenamiento.
1. En el campo **Nombre del documento de índice**, especifique una página de índice predeterminada de *index.html*. La página de índice predeterminada se muestra cuando un usuario navega hasta la raíz del sitio web estático.  
1. En el campo **Ruta de acceso del documento de error**, especifique una página de error predeterminada de *404.html*. La página de error predeterminada se muestra cuando un usuario intenta navegar a una página que no existe en el sitio web estático.
1. Haga clic en **Save**(Guardar). Azure Portal ahora muestra el punto de conexión estático del sitio web. 

    ![Habilitación del hospedaje del sitio web estático para una cuenta de almacenamiento](media/storage-blob-static-website-host/enable-static-website-hosting.png)

## <a name="deploy-a-hello-world-website"></a>Implementar un sitio web de Hola mundo

A continuación, cree una página web de Hola mundo con Visual Studio Code e impleméntela en el sitio web estático hospedado en la cuenta de Azure Storage.

1. Cree una carpeta vacía denominada *mywebsite* en el sistema de archivos local. 
1. Inicie Visual Studio Code y abra la carpeta que acaba de crear desde el panel **Explorador**.

    ![Apertura de la carpeta en Visual Studio Code](media/storage-blob-static-website-host/open-folder-vs-code.png)

1. Cree el archivo de índice predeterminado en la carpeta *mywebsite* y asígnele el nombre *index.html*.

    ![Creación del archivo de índice predeterminado en Visual Studio Code](media/storage-blob-static-website-host/create-index-file-vs-code.png)

1. Abra *index.html* en el editor, pegue el siguiente texto en el archivo y guárdelo:

    ```
    <!DOCTYPE html>
    <html>
      <body>
        <h1>Hello World!</h1>
      </body>
    </html>
    ```

1. Cree el archivo de error predeterminado y asígnele el nombre *404.html*.
1. Abra *404.html* en el editor, pegue el siguiente texto en el archivo y guárdelo:

    ```
    <!DOCTYPE html>
    <html>
      <body>
        <h1>404</h1>
      </body>
    </html>
    ```

1. Haga clic con el botón derecho en la carpeta *mywebsite* en el panel **Explorador** y seleccione **Implementar en el sitio web estático...** para implementar el sitio web. Se le pedirá que inicie sesión en Azure para recuperar una lista de suscripciones.

1. Seleccione la suscripción que contiene la cuenta de almacenamiento para la que habilitó el hospedaje estático de sitios web. A continuación, seleccione la cuenta de almacenamiento cuando se le solicite.

Visual Studio Code ahora cargará los archivos al punto de conexión web y mostrará la barra de estado de proceso realizado correctamente. Inicie el sitio web para verlo en Azure.

Ha completado el tutorial correctamente y ha implementado un sitio web estático en Azure.

## <a name="next-steps"></a>Pasos siguientes

En la primera parte de este tutorial, ha aprendido cómo configurar la cuenta de Azure Storage para el hospedaje estático de sitios web y cómo crear e implementar un sitio web estático en un punto de conexión de Azure.

A continuación, aprenda a configurar un dominio personalizado con el sitio web estático.

> [!div class="nextstepaction"]
> [Asignación de un dominio personalizado a un punto de conexión de Azure Blob Storage](storage-custom-domain-name.md)
