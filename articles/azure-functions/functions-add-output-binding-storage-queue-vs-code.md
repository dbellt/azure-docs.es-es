---
title: Conexión de Azure Functions a Azure Storage mediante Visual Studio Code
description: Aprenda a conectar Azure Functions a una cola de Azure Storage mediante la incorporación de un enlace de salida al proyecto de Visual Studio Code.
ms.date: 02/07/2020
ms.topic: quickstart
ms.custom: devx-track-python, devx-track-js
zone_pivot_groups: programming-languages-set-functions
ms.openlocfilehash: 785ed0ee38e27087750542c0719ab9e85ecd00ed
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109516675"
---
# <a name="connect-azure-functions-to-azure-storage-using-visual-studio-code"></a>Conexión de Azure Functions a Azure Storage mediante Visual Studio Code

[!INCLUDE [functions-add-storage-binding-intro](../../includes/functions-add-storage-binding-intro.md)]

En este artículo se muestra cómo usar Visual Studio Code para conectar Azure Storage a la función que creó en el artículo de inicio rápido anterior. El enlace de salida que se agrega a esta función escribe datos de la solicitud HTTP en un mensaje de la cola de Azure Queue Storage. 

La mayoría de los enlaces requieren una cadena de conexión almacenada que se usa en Functions para acceder al servicio enlazado. Para que sea más fácil, usará la cuenta de almacenamiento que creó con la aplicación de función. La conexión a esta cuenta ya está almacenada en una configuración de aplicación llamada `AzureWebJobsStorage`.  

## <a name="configure-your-local-environment"></a>Configuración del entorno local

Para empezar este artículo debe cumplir antes los requisitos siguientes:

* Instale la [extensión de Azure Storage para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestorage).

* Instale el [Explorador de Azure Storage](https://storageexplorer.com/). El Explorador de Storage una herramienta que se va a utilizar para examinar los mensajes en cola que genera el enlace de salida. El Explorador de Storage se admite en sistemas operativos basados en Linux, Windows y macOS.

::: zone pivot="programming-language-csharp"
* Instale las [herramientas de la CLI de .NET Core](/dotnet/core/tools/?tabs=netcore2x).
::: zone-end

::: zone pivot="programming-language-csharp"  
* Complete los pasos de la [parte 1 del inicio rápido de Visual Studio Code](create-first-function-vs-code-csharp.md). 
::: zone-end  
::: zone pivot="programming-language-javascript"  
* Complete los pasos de la [parte 1 del inicio rápido de Visual Studio Code](create-first-function-vs-code-node.md). 
::: zone-end   
::: zone pivot="programming-language-java"  
* Complete los pasos de la [parte 1 del inicio rápido de Visual Studio Code](create-first-function-vs-code-java.md). 
::: zone-end   
::: zone pivot="programming-language-typescript"  
* Complete los pasos de la [parte 1 del inicio rápido de Visual Studio Code](create-first-function-vs-code-typescript.md). 
::: zone-end   
::: zone pivot="programming-language-python"  
* Complete los pasos de la [parte 1 del inicio rápido de Visual Studio Code](create-first-function-vs-code-python.md). 
::: zone-end   
::: zone pivot="programming-language-powershell"  
* Complete los pasos de la [parte 1 del inicio rápido de Visual Studio Code](create-first-function-vs-code-powershell.md). 
::: zone-end   

En este artículo se da por supuesto que ya inició sesión en la suscripción de Azure desde Visual Studio Code. Puede iniciar sesión mediante la ejecución de `Azure: Sign In` desde la paleta de comandos. 

## <a name="download-the-function-app-settings"></a>Descarga de la configuración de la aplicación de función

En el [artículo de inicio rápido anterior](./create-first-function-vs-code-csharp.md) creó una aplicación de funciones en Azure junto con una cuenta de Storage. La cadena de conexión de esta cuenta se almacena de forma segura en la configuración de la aplicación en Azure. En este artículo, escribirá mensajes en una cola de almacenamiento de la misma cuenta. Para conectarse a su cuenta de almacenamiento cuando se ejecuta la función localmente, debe descargar la configuración de la aplicación en el archivo local.settings.json. 

1. Presione la tecla F1 para abrir la paleta de comandos, y busque y ejecute el comando `Azure Functions: Download Remote Settings....`. 

1. Elija la aplicación de funciones que creó en el artículo anterior. Seleccione **Sí a todo** para sobrescribir la configuración local existente. 

    > [!IMPORTANT]  
    > Como contiene secretos, el archivo local.settings.json nunca se publica, se excluye del control de código fuente.

1. Copie el valor `AzureWebJobsStorage`, que es la clave del valor de la cadena de conexión de la cuenta de Storage. Esta conexión se usa para comprobar que el enlace de salida funciona según lo previsto.

## <a name="register-binding-extensions"></a>Registro de extensiones de enlace

Dado que está utilizando un enlace de salida de Queue Storage, para poder ejecutar el proyecto debe tener instalada la extensión de enlaces de Storage. 

::: zone pivot="programming-language-javascript,programming-language-typescript,programming-language-python,programming-language-powershell,programming-language-java"

El proyecto se ha configurado para usar [conjuntos de extensiones](functions-bindings-register.md#extension-bundles), que instalan automáticamente un conjunto predefinido de paquetes de extensiones. 

El uso de conjuntos de extensiones se habilita en el archivo host.json en la raíz del proyecto, que tiene un aspecto similar al siguiente:

:::code language="json" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/host.json":::

::: zone-end

::: zone pivot="programming-language-csharp"

A excepción de los desencadenadores HTTP y el temporizador, los enlaces se implementan como paquetes de extensión. Ejecute el siguiente comando [dotnet add package](/dotnet/core/tools/dotnet-add-package) en la ventana Terminal para agregar el paquete de extensión de Storage al proyecto.

```bash
dotnet add package Microsoft.Azure.WebJobs.Extensions.Storage
```

::: zone-end

Ahora podrá agregar el enlace de salida de almacenamiento al proyecto.

## <a name="add-an-output-binding"></a>Adición de un enlace de salida

En Functions, para cada tipo de enlace es necesario definir los elementos `direction`, `type` y un valor único de `name` en el archivo function.json. La manera de definir estos atributos depende del lenguaje de la aplicación de funciones.

::: zone pivot="programming-language-javascript,programming-language-typescript,programming-language-python,programming-language-powershell"

[!INCLUDE [functions-add-output-binding-json](../../includes/functions-add-output-binding-json.md)]

::: zone-end

::: zone pivot="programming-language-csharp"

[!INCLUDE [functions-add-storage-binding-csharp-library](../../includes/functions-add-storage-binding-csharp-library.md)]

::: zone-end

::: zone pivot="programming-language-java"

[!INCLUDE [functions-add-output-binding-java](../../includes/functions-add-output-binding-java.md)]

::: zone-end

## <a name="add-code-that-uses-the-output-binding"></a>Adición de código que utilice el enlace de salida

Una vez definido el enlace, podrá usar el valor de `name` de este para acceder a él como atributo en la firma de función. Con un enlace de salida, no tiene que usar el código del SDK de Azure Storage para autenticarse, obtener una referencia de cola o escribir datos. El sistema en tiempo de ejecución de Functions y el enlace de salida de cola realizan esas tareas automáticamente.

::: zone pivot="programming-language-javascript"  
[!INCLUDE [functions-add-output-binding-js](../../includes/functions-add-output-binding-js.md)]
::: zone-end  

::: zone pivot="programming-language-typescript"  
[!INCLUDE [functions-add-output-binding-ts](../../includes/functions-add-output-binding-ts.md)]
::: zone-end  

::: zone pivot="programming-language-powershell"

[!INCLUDE [functions-add-output-binding-powershell](../../includes/functions-add-output-binding-powershell.md)]

::: zone-end

::: zone pivot="programming-language-python"

[!INCLUDE [functions-add-output-binding-python](../../includes/functions-add-output-binding-python.md)]

::: zone-end

::: zone pivot="programming-language-csharp"  

[!INCLUDE [functions-add-storage-binding-csharp-library-code](../../includes/functions-add-storage-binding-csharp-library-code.md)]

::: zone-end  

::: zone pivot="programming-language-java"  

[!INCLUDE [functions-add-storage-binding-java-code](../../includes/functions-add-storage-binding-java-code.md)]

## <a name="update-the-tests"></a>Actualización de las pruebas

[!INCLUDE [functions-add-output-binding-java-test](../../includes/functions-add-output-binding-java-test.md)]

::: zone-end  

## <a name="run-the-function-locally"></a>Ejecución local de la función

1. Como en el artículo anterior, presione <kbd>F5</kbd> para iniciar el proyecto de aplicación de función y Core Tools. 

1. Mientras se ejecuta Core Tools, vaya al área de **Azure: Área Functions**. En **Functions**, expanda **Proyecto local** > **Functions**. Haga clic con el botón derecho (Ctrl + clic en Mac) en la función `HttpExample` y elija **Execute Function Now...** (Ejecutar la función ahora...).

    :::image type="content" source="../../includes/media/functions-run-function-test-local-vs-code/execute-function-now.png" alt-text="Ejecutar la función ahora desde Visual Studio Code":::

1. En **Enter request body** (Especificar el cuerpo de la solicitud) verá el valor del cuerpo del mensaje de solicitud de `{ "name": "Azure" }`. Presione Entrar para enviar este mensaje de solicitud a la función.  
 
1. Una vez que se haya devuelto una respuesta, presione <kbd>Ctrl + C</kbd> para detener Core Tools.

Dado que está usando la cadena de conexión de almacenamiento, la función se conectará a la cuenta de almacenamiento de Azure durante la ejecución local. Cuando el enlace de salida se usa por primera vez, el entorno de ejecución de Functions crea una nueva cola denominada **outqueue** en la cuenta de almacenamiento. Usará el Explorador de Storage para comprobar que se han creado la cola y un mensaje.

### <a name="connect-storage-explorer-to-your-account"></a>Conexión del Explorador de Storage con la cuenta

Omita esta sección si ya ha instalado el Explorador de Azure Storage y lo ha conectado a su cuenta de Azure.

1. Ejecute la herramienta [Explorador de Azure Storage](https://storageexplorer.com/), y seleccione el icono de conexión de la izquierda y **Agregar una cuenta**.

    ![Incorporación de una cuenta de Azure al Explorador de Microsoft Azure Storage](./media/functions-add-output-binding-storage-queue-vs-code/storage-explorer-add-account.png)

1. En el cuadro de diálogo **Conectar**, elija **Add an Azure account** (Agregar una cuenta de Azure), elija el **Entorno Azure** y seleccione **Iniciar sesión...** 

    ![Inicio de sesión en la cuenta de Azure.](./media/functions-add-output-binding-storage-queue-vs-code/storage-explorer-connect-azure-account.png)

Después de iniciar sesión correctamente en su cuenta, verá todas las suscripciones de Azure asociadas con ella.

### <a name="examine-the-output-queue"></a>Examen de la cola de salida

1. En Visual Studio Code, presione la tecla F1 para abrir la paleta de comandos, busque y ejecute el comando `Azure Storage: Open in Storage Explorer` y elija el nombre de la cuenta de Storage. En el menú de la cuenta de almacenamiento, seleccione Explorador de Azure Storage.  

1. Expanda el nodo **Colas** y, después, seleccione la cola con el nombre **outqueue**. 

   La cola contiene el mensaje que creó el enlace de salida de la cola al ejecutar la función desencadenada por HTTP. Si se invoca la función con el valor predeterminado `name` de *Azure*, el mensaje de cola es *Name passed to the function: Azure* (Nombre pasado a la función: Azure).

    ![Mensaje de la cola que se muestra en el Explorador de Azure Storage](./media/functions-add-output-binding-storage-queue-vs-code/function-queue-storage-output-view-queue.png)

1. Vuelva a ejecutar la función, envíe otra solicitud y verá aparecer un nuevo mensaje en la cola.  

Ahora, es el momento de volver a publicar la aplicación de función actualizada en Azure.

## <a name="redeploy-and-verify-the-updated-app"></a>Reimplementación y comprobación de la aplicación actualizada

1. En Visual Studio Code, presione F1 para abrir la paleta de comandos. En la paleta de comandos, busque y seleccione `Azure Functions: Deploy to function app...`.

1. Elija la aplicación de funciones que creó en el primer artículo. Dado que va a volver a implementar el proyecto en la misma aplicación, seleccione **Implementar** para descartar la advertencia sobre la sobrescritura de archivos.

1. Una vez finalizada la implementación, puede volver a usar la característica **Execute Function Now...** (Ejecutar la función ahora...) para desencadenar la función en Azure.

1. De nuevo, [consulte el mensaje de la cola de almacenamiento](#examine-the-output-queue) para comprobar que el enlace de salida genera otra vez un nuevo mensaje en la cola.

## <a name="clean-up-resources"></a>Limpieza de recursos

En Azure, los *recursos* son aplicaciones de funciones, funciones o cuentas de almacenamiento, entre otros. Se agrupan en *grupos de recursos* y se puede eliminar todo el contenido de un grupo si este se elimina.

Ha creado recursos para completar estas guías de inicio rápido. Se le pueden facturar por estos recursos, dependiendo del [estado de la cuentade los ](https://azure.microsoft.com/account/) y [precios de los servicios](https://azure.microsoft.com/pricing/). Si ya no necesita los recursos, aquí se indica cómo eliminarlos:

[!INCLUDE [functions-cleanup-resources-vs-code-inner.md](../../includes/functions-cleanup-resources-vs-code-inner.md)]

## <a name="next-steps"></a>Pasos siguientes

Ha actualizado la función desencadenada por HTTP para escribir datos en una cola de almacenamiento. Ahora, puede obtener más información sobre el desarrollo de Functions mediante Visual Studio Code:

+ [Desarrollo de Azure Functions mediante Visual Studio Code](functions-develop-vs-code.md)

+ [Enlaces y desencadenadores de Azure Functions](functions-triggers-bindings.md).
::: zone pivot="programming-language-csharp"  
+ [Ejemplos de proyectos de Function completos en C#](/samples/browse/?products=azure-functions&languages=csharp).

+ [Referencia para desarrolladores de C# de Azure Functions](functions-dotnet-class-library.md)  
::: zone-end 
::: zone pivot="programming-language-javascript"  
+ [Ejemplos de proyectos de Function completos en JavaScript](/samples/browse/?products=azure-functions&languages=javascript).

+ [Guía para el desarrollador de JavaScript para Azure Functions](functions-reference-node.md)  
::: zone-end  
::: zone pivot="programming-language-java"  
+ [Ejemplos de proyectos de Functions completos en Java](/samples/browse/?products=azure-functions&languages=java).

+ [Guía de Azure Functions para desarrolladores de Java](functions-reference-java.md)  
::: zone-end  
::: zone pivot="programming-language-typescript"  
+ [Ejemplos de proyectos de Function completos en TypeScript](/samples/browse/?products=azure-functions&languages=typescript).

+ [Guía para desarrolladores de TypeScript para Azure Functions](functions-reference-node.md#typescript)  
::: zone-end  
::: zone pivot="programming-language-python"  
+ [Ejemplos de proyectos de Function completos en Python](/samples/browse/?products=azure-functions&languages=python).

+ [Guía de Azure Functions para desarrolladores de Python](functions-reference-python.md)  
::: zone-end  
::: zone pivot="programming-language-powershell"  
+ [Ejemplos de proyectos de Function completos en PowerShell](/samples/browse/?products=azure-functions&languages=azurepowershell).

+ [Guía para desarrolladores de PowerShell para Azure Functions](functions-reference-powershell.md) 
::: zone-end
