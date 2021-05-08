---
title: Desarrollo y publicación de funciones de .NET 5 mediante Azure Functions
description: Aprenda a crear y depurar funciones de C# mediante .NET 5.0 y, a continuación, implementar el proyecto local en el hospedaje sin servidor en Azure Functions.
ms.date: 03/03/2021
ms.topic: how-to
zone_pivot_groups: development-environment-functions
ms.openlocfilehash: c76fde9a61ca60171ac094ef541e8c5841846aab
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107866278"
---
# <a name="develop-and-publish-net-5-functions-using-azure-functions"></a>Desarrollo y publicación de funciones de .NET 5 mediante Azure Functions 

En este artículo se muestra cómo trabajar con funciones de C# mediante .NET 5.0, que se ejecutan fuera de proceso desde el tiempo de ejecución de Azure Functions. Aprenderá a crear, depurar localmente y publicar estas funciones de proceso aislado de .NET en Azure. En Azure, estas funciones se ejecutan en un proceso aislado que admite .NET 5.0. Para más información, consulte la [Guía para ejecutar funciones en .NET 5.0 en Azure](dotnet-isolated-process-guide.md).

Si no necesita admitir .NET 5.0 o ejecutar las funciones fuera de proceso, es posible que en su lugar quiera [crear una función de la biblioteca de clases C#](functions-create-your-first-function-visual-studio.md).

>[!NOTE]
>Actualmente no se admite el desarrollo de funciones de proceso aislado .NET en Azure Portal. Debe usar la publicación de la CLI de Azure o Visual Studio Code para crear una aplicación de funciones en Azure que admita la ejecución de aplicaciones .NET 5.0 fuera de proceso.   

## <a name="prerequisites"></a>Requisitos previos

+ Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [SDK de .NET 5.0](https://dotnet.microsoft.com/download)

+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.0.3381 u otra posterior.

+ La [CLI de Azure](/cli/azure/install-azure-cli), versión 2.20 u otra posterior.  
::: zone pivot="development-environment-vscode"
+ [Visual Studio Code](https://code.visualstudio.com/) en una de las [plataformas admitidas](https://code.visualstudio.com/docs/supporting/requirements#_platforms).  

+ La [extensión de C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) para Visual Studio Code.  

+ La [extensión de Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) para Visual Studio Code, versión 1.3.0 u otra más reciente.
::: zone-end
::: zone pivot="development-environment-vs"
+ [Visual Studio 2019](https://azure.microsoft.com/downloads/), con la carga de trabajo **Desarrollo de Azure**.  
La publicación y las plantillas de proyecto de funciones aisladas .NET no están actualmente disponibles en Visual Studio.
::: zone-end

## <a name="create-a-local-function-project"></a>Creación de un proyecto de función local

En Azure Functions, un proyecto de función es un contenedor para una o varias funciones individuales que responden a un desencadenador específico. Todas las funciones de un proyecto comparten las mismas configuraciones locales y de hospedaje. En esta sección, se crea un proyecto de función que contiene una sola función.

::: zone pivot="development-environment-vs"

>[!NOTE]  
> En este momento, no hay ninguna plantilla de proyecto de Visual Studio que admita la creación de proyectos de funciones aisladas .NET. En este artículo se muestra cómo usar Core Tools para crear el proyecto de C#, que se puede ejecutar localmente y depurar en Visual Studio.  

::: zone-end

::: zone pivot="development-environment-vscode"  
1. Seleccione el icono de Azure en la barra de actividades y después en el área **Azure: Functions**, seleccione el icono **Crear un proyecto**.

    ![Elija Crear un proyecto.](./media/functions-create-first-function-vs-code/create-new-project.png)

1. Elija una ubicación de directorio para el área de trabajo del proyecto y elija **Seleccionar**.

    > [!NOTE]
    > Estos pasos se han diseñado para completarse fuera de un área de trabajo. Por tanto, no seleccione una carpeta de proyecto que forme parte de un área de trabajo.

1. Escriba la siguiente información cuando se le indique:

    + **Seleccione un lenguaje para el proyecto de funciones**: Elija `C#`.

    + **Seleccione un entorno de ejecución .NET**: elija `.NET 5 isolated`.

    + **Seleccione una plantilla para la primera función del proyecto**: Elija `HTTP trigger`.

    + **Especifique un nombre de función**: Escriba `HttpExample`.

    + **Especifique un espacio de nombres**: Escriba `My.Functions`.

    + **Nivel de autorización**: Elija `Anonymous`, que permite que cualquier llame al punto de conexión de la función. Para obtener información sobre el nivel de autorización, consulte [Claves de autorización](functions-bindings-http-webhook-trigger.md#authorization-keys).

    + **Seleccione cómo desea que se abra el proyecto**: Elija `Add to workspace`.

1. Con esta información, Visual Studio Code genera un proyecto de Azure Functions con un desencadenador HTTP. Los archivos del proyecto locales se pueden ver en Explorer. Para obtener más información sobre los archivos que se crean, consulte [Archivos del proyecto generados](functions-develop-vs-code.md#generated-project-files).
::: zone-end  
::: zone pivot="development-environment-cli,development-environment-vs"  

1. Ejecute el comando `func init`, de la manera siguiente, para crear un proyecto de funciones en una carpeta llamada *LocalFunctionProj*:  

    ```console
    func init LocalFunctionProj --worker-runtime dotnetisolated
    ```
    
    Al especificar, `dotnetisolated` se crea un proyecto que se ejecuta en .NET 5.0.


1. Vaya a la carpeta del proyecto:

    ```console
    cd LocalFunctionProj
    ```

    Esta carpeta contiene varios archivos del proyecto, incluidos los archivos de configuración [local.settings.json](functions-run-local.md#local-settings-file) y [host.json](functions-host-json.md). Como *local.settings.json* puede contener secretos descargados de Azure, el archivo se excluye del control de código fuente de forma predeterminada en el archivo *.gitignore*.

1. Agregue una función al proyecto mediante el comando siguiente, donde el argumento `--name` es el nombre único de la función (HttpExample) y el argumento `--template` especifica el desencadenador de esta (HTTP).

    ```console
    func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
    ``` 

    `func new` crea un archivo de código HttpExample.cs.
::: zone-end  

::: zone pivot="development-environment-vscode"  

[!INCLUDE [functions-run-function-test-local-vs-code](../../includes/functions-run-function-test-local-vs-code.md)]

::: zone-end  
::: zone pivot="development-environment-cli" 

[!INCLUDE [functions-run-function-test-local-cli](../../includes/functions-run-function-test-local-cli.md)]

::: zone-end

::: zone pivot="development-environment-vs"

## <a name="run-the-function-locally"></a>Ejecución local de la función

En este momento, puede ejecutar el comando `func start` desde la raíz de la carpeta del proyecto para compilar y ejecutar el proyecto de funciones aisladas de C#. Actualmente, si quiere depurar el código de la función fuera de proceso en Visual Studio, debe asociar manualmente un depurador al proceso del entorno de ejecución de Functions en ejecución mediante los pasos siguientes:  

1. Abra el archivo del proyecto (. csproj) en Visual Studio. Puede revisar y modificar el código del proyecto y establecer los puntos de interrupción deseados en el código. 

1. En la carpeta raíz del proyecto, use el siguiente comando desde el terminal o desde un símbolo del sistema para iniciar el host en tiempo de ejecución:

    ```console
    func start --dotnet-isolated-debug
    ```

    La opción `--dotnet-isolated-debug` indica al proceso que espere a que se adjunte un depurador antes de continuar. Hacia el final de la salida, debería ver algo parecido a las líneas siguientes: 
    
    <pre>
    ...
    
    Functions:

        HttpExample: [GET,POST] http://localhost:7071/api/HttpExample

    For detailed output, run func with --verbose flag.
    [2021-03-09T08:41:41.904Z] Azure Functions .NET Worker (PID: 81720) initialized in debug mode. Waiting for debugger to attach...
    ...
    
    </pre> 

    `PID: XXXXXX` indica el identificador de proceso (PID) del proceso de dotnet.exe que es el host de Functions en ejecución.
 
1. En la salida del tiempo de ejecución de Azure Functions, tome nota del identificador de proceso del proceso de host, al que adjuntará un depurador. Fíjese también en la dirección URL de la función local.

1. En el menú **Depurar** de Visual Studio, seleccione **Asociar al proceso...** , busque el proceso que coincida con el identificador de proceso y seleccione **Asociar**. 
    
    :::image type="content" source="media/dotnet-isolated-process-developer-howtos/attach-to-process.png" alt-text="Asociar el depurador a los procesos de host de Functions":::    

    Con el depurador asociado puede depurar el código de función de forma habitual.

1. En la barra de direcciones del explorador, escriba la dirección URL de la función local, que tiene un aspecto similar al siguiente, y ejecute la solicitud. 

    `http://localhost:7071/api/HttpExample`

    Debería ver la salida de seguimiento de la solicitud escrita en el terminal en ejecución. La ejecución de código se detiene en los puntos de interrupción establecidos en el código de función.

1. Cuando haya terminado, vaya al terminal y presione CTRL + C para detener el proceso de host.
 
Después de comprobar que la función se ejecuta correctamente en el equipo local es el momento de publicar el proyecto en Azure.

> [!NOTE]  
> La publicación de Visual Studio no está disponible actualmente para aplicaciones de proceso aislado de .NET. Una vez que haya terminado de desarrollar el proyecto en Visual Studio, debe usar la CLI de Azure para crear los recursos remotos de Azure. Después, puede volver a usar Azure Functions Core Tools desde la línea de comandos para publicar el proyecto en Azure.
::: zone-end

::: zone pivot="development-environment-cli,development-environment-vs" 
## <a name="create-supporting-azure-resources-for-your-function"></a>Creación de recursos auxiliares de Azure para la función

Antes de poder implementar el código de la función en Azure, debe crear tres recursos:

- Un [grupo de recursos](../azure-resource-manager/management/overview.md), que es un contenedor lógico de recursos relacionados.
- Una [cuenta de almacenamiento](../storage/common/storage-account-create.md), que se usa para mantener el estado y otra información sobre sus funciones.
- Una aplicación de funciones, que proporciona el entorno para ejecutar el código de función. Una aplicación de funciones se asigna al proyecto de funciones y le permite agrupar funciones como una unidad lógica para facilitar la administración, la implementación y el uso compartido de recursos.

Utilice los comandos siguientes para crear los elementos.

1. Si todavía no lo ha hecho, inicie sesión en Azure:

    ```azurecli
    az login
    ```

    El comando [az login](/cli/azure/reference-index#az_login) inicia sesión en su cuenta de Azure.

1. Cree un grupo de recursos denominado `AzureFunctionsQuickstart-rg` en la región `westeurope`:

    ```azurecli
    az group create --name AzureFunctionsQuickstart-rg --location westeurope
    ```
 
    El comando [az group create](/cli/azure/group#az_group_create) crea un grupo de recursos. Por lo general, los recursos y los grupos de recursos se crean en una región cerca de usted, y se utiliza alguna de las regiones devueltas por el comando `az account list-locations`.

1. Cree una cuenta de almacenamiento de uso general en su grupo de recursos y región:

    ```azurecli
    az storage account create --name <STORAGE_NAME> --location westeurope --resource-group AzureFunctionsQuickstart-rg --sku Standard_LRS
    ```

    El comando [az storage account create](/cli/azure/storage/account#az_storage_account_create) crea la cuenta de almacenamiento. 

    En el ejemplo anterior, reemplace `<STORAGE_NAME>` por un nombre que sea apropiado para usted y único en Azure Storage. Los nombres deben contener entre 3 y 24 caracteres y solo letras minúsculas. `Standard_LRS` especifica una cuenta de uso general, que es [compatible con Functions](storage-considerations.md#storage-account-requirements).

1. Cree la aplicación de funciones en Azure:
   
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime dotnet-isolated --runtime-version 5.0 --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME>
    ```

    El comando [az functionapp create](/cli/azure/functionapp#az_functionapp_create) crea la aplicación de funciones en Azure. 
    
    En el ejemplo anterior, reemplace `<STORAGE_NAME>` por el nombre de la cuenta que usó en el paso anterior y reemplace `<APP_NAME>` por un nombre único global que le resulte adecuado. `<APP_NAME>` también es el dominio DNS predeterminado de la aplicación de función. 
    
    Este comando crea una aplicación de funciones que ejecuta .NET 5.0 en el [plan de consumo de Azure Functions](consumption-plan.md). Este plan debe ser gratuito para la cantidad de uso que incurre en este artículo. El comando también aprovisiona una instancia asociada de Azure Application Insights en el mismo grupo de recursos. Puede usar esta instancia para supervisar la aplicación de funciones y ver los registros. Para más información, consulte [Supervisión de Azure Functions](functions-monitoring.md). La instancia no incurrirá en ningún costo hasta que se active.

[!INCLUDE [functions-publish-project-cli](../../includes/functions-publish-project-cli.md)]

::: zone-end

::: zone pivot="development-environment-vscode"

[!INCLUDE [functions-sign-in-vs-code](../../includes/functions-sign-in-vs-code.md)]

## <a name="publish-the-project-to-azure"></a>Publicar el proyecto en Azure

En esta sección, va a crear una aplicación de funciones y los recursos relacionados en su suscripción de Azure y, después, va a implementar el código.

> [!IMPORTANT]
> La publicación en una aplicación de función existente sobrescribe el contenido de esa aplicación en Azure.


1. Seleccione el icono de Azure en la barra de actividades y después en el área **Azure: Functions**, seleccione el botón de **implementación en la aplicación de funciones**.

    ![Publicación del proyecto en Azure](../../includes/media/functions-publish-project-vscode/function-app-publish-project.png)

1. Escriba la siguiente información cuando se le indique:

    - **Seleccione la carpeta**: elija una carpeta en el área de trabajo o busque una que contenga la aplicación de funciones. No verá esta opción si ya tiene una aplicación de funciones válida abierta.

    - **Seleccione la suscripción**: elija la suscripción que desee usar. No se mostrará esta solicitud si solo tiene una suscripción.

    - **Seleccione la aplicación de funciones en Azure**: Elija `- Create new Function App`. (No elija la opción `Advanced`, que no se trata en este artículo).
      
    - **Escriba un nombre único global para la aplicación de funciones**: Escriba un nombre que sea válido en una ruta de acceso de la dirección URL. El nombre que escriba se valida para asegurarse de que es único en Azure Functions.
    
    - **Seleccione una pila del entorno en tiempo de ejecución**: Elija `.NET 5 (non-LTS)`. 
    
    - **Seleccione una ubicación para los nuevos recursos**:  Para mejorar el rendimiento, elija una [región](https://azure.microsoft.com/regions/) cerca de usted. 
    
    En el área de notificación, verá el estado de los recursos individuales a medida que se crean en Azure.

    :::image type="content" source="../../includes/media/functions-publish-project-vscode/resource-notification.png" alt-text="Notificación de creación de recursos de Azure":::
    
1.  Cuando se complete, se crearán los siguientes recursos de Azure en la suscripción con nombres que se basan en el nombre de la aplicación de funciones:
    
    [!INCLUDE [functions-vs-code-created-resources](../../includes/functions-vs-code-created-resources.md)]

    Una vez que se haya creado la aplicación de función se mostrará una notificación y se aplicará el paquete de implementación. 

    [!INCLUDE [functions-vs-code-create-tip](../../includes/functions-vs-code-create-tip.md)]

4. Seleccione **View Output** (Ver salida) en esta notificación para ver la creación y los resultados de la implementación, incluidos los recursos de Azure que ha creado. Si se pierde la notificación, seleccione el icono de campana en la esquina inferior derecha para verlo de nuevo.

    ![Creación de la notificación completa](../../includes/media/functions-publish-project-vscode/function-create-notifications.png)

::: zone-end

::: zone pivot="development-environment-cli"  
[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]  
::: zone-end  

::: zone pivot="development-environment-vscode"  
[!INCLUDE [functions-vs-code-run-remote](../../includes/functions-vs-code-run-remote.md)]  
::: zone-end  

## <a name="clean-up-resources"></a>Limpieza de recursos

Ha creado recursos para completar este artículo. Se le pueden facturar por estos recursos, dependiendo del [estado de la cuentade los ](https://azure.microsoft.com/account/) y [precios de los servicios](https://azure.microsoft.com/pricing/). 

::: zone pivot="development-environment-cli"  
Use el siguiente comando para eliminar el grupo de recursos y todos los recursos que contiene para evitar incurrir en costos adicionales.

```azurecli
az group delete --name AzureFunctionsQuickstart-rg
```
::: zone-end  

::: zone pivot="development-environment-vscode"  
Use los pasos siguientes para eliminar la aplicación de funciones y sus recursos relacionados para evitar incurrir en costos adicionales.

[!INCLUDE [functions-cleanup-resources-vs-code-inner.md](../../includes/functions-cleanup-resources-vs-code-inner.md)]  
::: zone-end  
::: zone pivot="development-environment-vs"   
Use los pasos siguientes para eliminar la aplicación de funciones y sus recursos relacionados para evitar incurrir en costos adicionales.

1. En Cloud Explorer, expanda su suscripción > **App Services**, haga clic con el botón derecho en su aplicación de funciones y elija **Abrir en el portal**. 

1. En la página de la aplicación de funciones, seleccione la pestaña **Información general** y, después, seleccione el vínculo situado bajo **Grupo de recursos**.

   :::image type="content" source="media/functions-create-your-first-function-visual-studio/functions-app-delete-resource-group.png" alt-text="Seleccione el grupo de recursos que se va a eliminar de la página de la aplicación de funciones":::

2. En la página **Grupo de recursos**, revise la lista de recursos incluidos y compruebe que son los que desea eliminar.
 
3. Seleccione **Eliminar grupo de recursos** y siga las instrucciones.

   El proceso de eliminación tardará un par de minutos. Cuando termine, aparece una notificación durante unos segundos. También puede seleccionar el icono de campana en la parte superior de la página para ver la notificación.
::: zone-end

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre las funciones aisladas de .NET](dotnet-isolated-process-guide.md)

