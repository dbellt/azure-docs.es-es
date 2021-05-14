---
title: Creación de su primera función en Azure Portal
description: Obtenga información sobre cómo crear su primera función de Azure para su ejecución sin servidor mediante Azure Portal.
ms.topic: how-to
ms.date: 03/26/2020
ms.custom: devx-track-csharp, mvc, devcenter, cc996988-fb4f-47
ms.openlocfilehash: 336e531f4ec64141770fc26d7e6eea9ebfedf922
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107517153"
---
# <a name="create-your-first-function-in-the-azure-portal"></a>Creación de su primera función en Azure Portal

Azure Functions permite ejecutar el código en un entorno sin servidor y sin necesidad de crear una máquina virtual (VM) ni publicar una aplicación web. En este artículo, aprenderá a usar Azure Functions para crear una función de desencadenador de HTTP "Hola mundo" en Azure Portal.

[!INCLUDE [functions-in-portal-editing-note](../../includes/functions-in-portal-editing-note.md)] 

En su lugar, se recomienda [desarrollar las funciones localmente](functions-develop-local.md) y publicarlas en una aplicación de funciones en Azure.  
Use uno de los siguientes vínculos para empezar a trabajar con el entorno de desarrollo local y el idioma que elija:

| Visual Studio Code | Terminal y símbolo del sistema | Visual Studio |
| --- | --- | --- |
|  &bull;&nbsp;[Introducción a C#](./create-first-function-vs-code-csharp.md)<br/>&bull;&nbsp;[Introducción a Java](./create-first-function-vs-code-java.md)<br/>&bull;&nbsp;[Introducción a JavaScript](./create-first-function-vs-code-node.md)<br/>&bull;&nbsp;[Introducción a PowerShell](./create-first-function-vs-code-powershell.md)<br/>&bull;&nbsp;[Introducción a Python](./create-first-function-vs-code-python.md) |&bull;&nbsp;[Introducción a C#](./create-first-function-cli-csharp.md)<br/>&bull;&nbsp;[Introducción a Java](./create-first-function-cli-java.md)<br/>&bull;&nbsp;[Introducción a JavaScript](./create-first-function-cli-node.md)<br/>&bull;&nbsp;[Introducción a PowerShell](./create-first-function-cli-powershell.md)<br/>&bull;&nbsp;[Introducción a Python](./create-first-function-cli-python.md) | [Introducción a C#](functions-create-your-first-function-visual-studio.md) |

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

## <a name="create-a-function-app"></a>Creación de una aplicación de función

Debe tener una Function App para hospedar la ejecución de las funciones. Una aplicación de función permite agrupar funciones como una unidad lógica para facilitar la administración, la implementación, el escalado y el uso compartido de recursos.

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

Después, cree una función en la nueva aplicación de funciones.

## <a name="create-an-http-trigger-function"></a><a name="create-function"></a>Creación de una función de desencadenador de HTTP

1. En el menú de la izquierda de la ventana **Funciones**, seleccione **Funciones** y, a continuación, seleccione **Agregar** en el menú superior. 
 
1. En la ventana **Agregar función**, seleccione la plantilla **Desencadenador HTTP**.

    ![Elección de la función de desencadenador de HTTP](./media/functions-create-first-azure-function/function-app-select-http-trigger.png)

1. En **Detalles de la plantilla** utilice `HttpExample` para **Nueva función**, seleccione **Anónimo** de la lista desplegable **[Nivel de autorización](functions-bindings-http-webhook-trigger.md#authorization-keys)** y, a continuación, seleccione **Agregar**.

    Azure crea la función de desencadenador de HTTP. Ahora, puede ejecutar la nueva función mediante el envío de una solicitud HTTP.

## <a name="test-the-function"></a>Prueba de la función

1. En la nueva función de desencadenador de HTTP, seleccione **Código y prueba** en el menú de la izquierda y, a continuación, seleccione **Obtener la dirección URL de la función** en el menú superior.

    ![Selección de la opción Obtener la dirección URL de la función](./media/functions-create-first-azure-function/function-app-select-get-function-url.png)

1. En el cuadro de diálogo **Obtener la dirección URL de la función**, seleccione **valor predeterminado** en la lista desplegable y, a continuación, seleccione el icono **Copiar al Portapapeles**. 

    ![Copiar la dirección URL de la función desde Azure Portal](./media/functions-create-first-azure-function/function-app-develop-tab-testing.png)

1. Pegue la dirección URL de la función en la barra de direcciones de su explorador. Anexe el valor `?name=<your_name>` de la cadena de consulta al final de esta dirección URL y presione Entrar para ejecutar la solicitud. El explorador debe mostrar un mensaje de respuesta que devuelve el valor de la cadena de consulta. 

    Si la dirección URL de la solicitud incluye una [clave de acceso](functions-bindings-http-webhook-trigger.md#authorization-keys) (`?code=...`), significa que debe elegir **Función** en lugar del nivel de acceso **Anónimo** al crear la función. En este caso, debería anexar `&name=<your_name>` en su lugar.

1. Cuando se ejecuta la función, se escribe información de seguimiento en los registros. Para ver los resultados del seguimiento, vuelva a la página **Código y prueba** en el portal y expanda la flecha **Registros** en la parte inferior de la página. Vuelva a llamar a la función para ver la salida de seguimiento escrita en los registros. 

    :::image type="content" source="media/functions-create-first-azure-function/function-view-logs.png" alt-text="Visor de registros de las funciones en Azure Portal":::.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [Clean-up resources](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps.md)]
