---
title: 'Inicio rápido: Creación de la primera función de C# en Azure mediante Visual Studio'
description: En este inicio rápido, aprenderá a usar Visual Studio para crear y publicar una función de C# desencadenada por HTTP en Azure Functions que se ejecuta en .NET Core 3.1.
ms.assetid: 82db1177-2295-4e39-bd42-763f6082e796
ms.topic: quickstart
ms.date: 05/18/2021
ms.custom: devx-track-csharp, mvc, devcenter, vs-azure, 23113853-34f2-4f, contperf-fy21q3-portal
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./functions-create-your-first-function-visual-studio-uiex
ms.openlocfilehash: b76d9cf86a1bb00ea73c396ce1e563374a834f89
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110459476"
---
# <a name="quickstart-create-your-first-c-function-in-azure-using-visual-studio"></a>Inicio rápido: Creación de la primera función de C# en Azure mediante Visual Studio

Azure Functions permite ejecutar el código de C# en un entorno sin servidor en Azure. 

En este artículo aprenderá a:

> [!div class="checklist"]
> * Use Visual Studio para crear un proyecto de biblioteca de clases de C# (.NET Core 3.1).
> * Cree una función que responda a solicitudes HTTP. 
> * Ejecute el código localmente para comprobar el comportamiento de la función.
> * Implemente el proyecto de código en Azure Functions. 
 
Este inicio rápido supone un pequeño costo en su cuenta de Azure.
 
El proyecto creado se ejecuta en .NET Core 3.1. Si en su lugar desea crear un proyecto que se ejecute en .NET 5.0, consulte [Desarrollo y publicación de funciones de .NET 5 mediante Azure Functions](dotnet-isolated-process-developer-howtos.md).

## <a name="prerequisites"></a>Prerrequisitos

+ [Visual Studio 2019](https://azure.microsoft.com/downloads/). Asegúrese de seleccionar la carga de trabajo de **desarrollo de Azure** durante la instalación. 

+ [Suscripción de Azure](../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing). Si aún no tiene ninguna cuenta, cree una [gratuita](https://azure.microsoft.com/free/dotnet/) antes de empezar.

## <a name="create-a-function-app-project"></a>Creación de un proyecto de aplicación de función

[!INCLUDE [Create a project using the Azure Functions template](../../includes/functions-vstools-create.md)]

Visual Studio crea un proyecto y una clase que contiene código reutilizable para el tipo de función de desencadenador HTTP. El código reutilizable envía una respuesta HTTP que incluye un valor del cuerpo de la solicitud o de la cadena de consulta. El atributo `HttpTrigger` especifica que la función es desencadenada por una solicitud HTTP. 

## <a name="rename-the-function"></a>Cambio del nombre de la función

El atributo del método `FunctionName` establece el nombre de la función que, de forma predeterminada, se genera como `Function1`. Como las herramientas no le permiten reemplazar el nombre de la función predeterminada al crear un proyecto, dedique un minuto a crear un nombre mejor para la clase, el archivo y los metadatos de la función.

1. En el **Explorador de archivos**, haga clic con el botón derecho en el archivo Function1.cs y cambie su nombre a `HttpExample.cs`.

1. En el código, cambie el nombre de la clase Function1 a `HttpExample`.

1. En el método `HttpTrigger` denominado `Run`, cambie el nombre del atributo del método `FunctionName` a `HttpExample`.

La definición de la función ahora debe parecerse al código siguiente:

:::code language="csharp" source="~/functions-docs-csharp/http-trigger-template/HttpExample.cs" range="13-18"::: 
 
Ahora que ha cambiado el nombre de la función, puede probarla en el equipo local.

## <a name="run-the-function-locally"></a>Ejecución local de la función

Visual Studio se integra con Azure Functions Core Tools; de este modo puede probar las funciones localmente mediante el sistema en tiempo de ejecución completo de Azure Functions.  

[!INCLUDE [functions-run-function-test-local-vs](../../includes/functions-run-function-test-local-vs.md)]

Después de comprobar que la función se ejecuta correctamente en el equipo local es el momento de publicar el proyecto en Azure.

## <a name="publish-the-project-to-azure"></a>Publicar el proyecto en Azure

Para poder publicar el proyecto, debe tener una aplicación de funciones en la suscripción de Azure. La publicación de Visual Studio crea una aplicación de funciones la primera vez que se publica el proyecto.

[!INCLUDE [Publish the project to Azure](../../includes/functions-vstools-publish.md)]

## <a name="test-your-function-in-azure"></a>Prueba de una función en Azure

1. En Cloud Explorer, se debe seleccionar la nueva aplicación de funciones. Si no es así, expanda su suscripción > **App Services** y seleccione la nueva aplicación de funciones.

1. Haga clic con el botón derecho en la aplicación de funciones y elija **Abrir en explorador**. Se abre la raíz de su aplicación de funciones en el explorador web predeterminado y muestra la página que indica que su aplicación de funciones se está ejecutando. 

    :::image type="content" source="media/functions-create-your-first-function-visual-studio/function-app-running-azure.png" alt-text="Aplicación de funciones en ejecución":::

1. En la barra de direcciones del explorador, anexe la cadena `/api/HttpExample?name=Functions` a la URL base y ejecute la solicitud.

    La dirección URL que llama a la función de desencadenador HTTP tiene el formato siguiente:

    `http://<APP_NAME>.azurewebsites.net/api/HttpExample?name=Functions`

2. Vaya a esta dirección URL y verá una respuesta en el explorador para la solicitud GET remota devuelta por la función, que es similar a la del siguiente ejemplo:

    :::image type="content" source="media/functions-create-your-first-function-visual-studio/functions-create-your-first-function-visual-studio-browser-azure.png" alt-text="Respuesta de la función en el explorador":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido de esta colección se basan en los valores de esta. Si tiene previsto trabajar con las siguientes guías de inicio rápido, tutoriales o con cualquiera de los servicios que haya creado en esta guía de inicio rápido, no elimine los recursos.

En Azure, los *recursos* son aplicaciones de función, funciones o cuentas de almacenamiento, entre otros. Se agrupan en *grupos de recursos* y se puede eliminar todo el contenido de un grupo si este se elimina. 

Ha creado recursos para completar estas guías de inicio rápido. Se le pueden facturar por estos recursos, dependiendo del [estado de la cuentade los ](https://azure.microsoft.com/account/) y [precios de los servicios](https://azure.microsoft.com/pricing/). Si ya no necesita los recursos, aquí se indica cómo eliminarlos:

1. En Cloud Explorer, expanda su suscripción > **App Services**, haga clic con el botón derecho en su aplicación de funciones y elija **Abrir en el portal**. 

1. En la página de la aplicación de funciones, seleccione la pestaña **Información general** y, después, seleccione el vínculo situado bajo **Grupo de recursos**.

   :::image type="content" source="media/functions-create-your-first-function-visual-studio/functions-app-delete-resource-group.png" alt-text="Seleccione el grupo de recursos que se va a eliminar de la página de la aplicación de funciones":::

2. En la página **Grupo de recursos**, revise la lista de recursos incluidos y compruebe que son los que desea eliminar.
 
3. Seleccione **Eliminar grupo de recursos** y siga las instrucciones.

   El proceso de eliminación tardará un par de minutos. Cuando termine, aparece una notificación durante unos segundos. También puede seleccionar el icono de campana en la parte superior de la página para ver la notificación.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha usado Visual Studio para crear y publicar una aplicación de funciones en C# en Azure con una función de desencadenador HTTP sencilla. 

Prosiga en el siguiente artículo para aprender a agregar un enlace de cola de Azure Storage a la función:
> [!div class="nextstepaction"]
> [Adición de un enlace de cola de Azure Storage a una función](functions-add-output-binding-storage-queue-vs.md)

