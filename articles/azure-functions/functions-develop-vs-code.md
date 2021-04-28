---
title: Desarrollo de Azure Functions con Visual Studio Code
description: Aprenda a desarrollar y probar Azure Functions mediante la extensión de Azure Functions para Visual Studio Code.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 08/21/2019
ms.openlocfilehash: c2869b2b30722495523a9f0dfb2d70a17a205854
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107871280"
---
# <a name="develop-azure-functions-by-using-visual-studio-code"></a>Desarrollo de Azure Functions con Visual Studio Code

La [Extensión de Azure Functions para Visual Studio Code] permite desarrollar localmente las funciones e implementarlas en Azure. Si esta es su primera experiencia con Azure Functions, puede obtener más información en [Introducción a Azure Functions](functions-overview.md).

La extensión de Azure Functions proporciona las siguientes ventajas:

* Editar, compilar y ejecutar funciones en el equipo de desarrollo local.
* Publicar su proyecto de Azure Functions directamente en Azure.
* Escribir las funciones en varios lenguajes mientras se aprovechan de las ventajas de Visual Studio Code.

Puede utilizar la extensión con los siguientes lenguajes, que se admiten en el entorno en tiempo de ejecución de Azure Functions a partir de la versión 2.x:

* [C# compilado](functions-dotnet-class-library.md)
* [Script de C#](functions-reference-csharp.md)<sup>*</sup>
* [JavaScript](functions-reference-node.md)
* [Java](functions-reference-java.md)
* [PowerShell](functions-reference-powershell.md)
* [Python](functions-reference-python.md)

<sup>*</sup>Requiere el [establecimiento de script de C# como el lenguaje de proyecto predeterminado](#c-script-projects).

En este artículo, solo están disponibles ejemplos para las funciones de biblioteca de clases de JavaScript (Node.js) y de C#.  

En este artículo se dan detalles sobre cómo usar la extensión de Azure Functions para desarrollar funciones y publicarlas en Azure. Antes de leer este artículo, debería [crear la primera función mediante Visual Studio Code](./create-first-function-vs-code-csharp.md).

> [!IMPORTANT]
> No mezcle el desarrollo local con el desarrollo del portal en una aplicación de funciones individual. Al publicar desde un proyecto local en una aplicación de la función, el proceso de implementación sobrescribe todas las funciones que ha desarrollado en el portal.

## <a name="prerequisites"></a>Requisitos previos

Antes de instalar y ejecutar la [extensión de Azure Functions][extensión de azure functions para visual studio code], es necesario cumplir estos requisitos:

* Tener instalado [Visual Studio Code](https://code.visualstudio.com/) en una de las [plataformas compatibles](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

* Una suscripción de Azure activa.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Otros recursos que necesita, como una cuenta de Azure Storage, se crean en su suscripción cuando [publica con Visual Studio Code](#publish-to-azure). 

### <a name="run-local-requirements"></a>Ejecución de requisitos locales

Estos requisitos previos solo son necesarios para [ejecutar y depurar las funciones localmente](#run-functions-locally), pero no lo son para crear o publicar proyectos en Azure Functions.

# <a name="c"></a>[C\#](#tab/csharp)

+ La versión 2.x de [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools), o cualquier versión posterior. El paquete Core Tools se descarga e instala automáticamente al iniciar el proyecto de forma local. Core Tools incluye el entorno de ejecución completo de Azure Functions, por lo que la descarga y la instalación pueden tardar algún tiempo.

+ La [extensión de C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) para Visual Studio Code. 

+ [Herramientas de la CLI de .NET Core](/dotnet/core/tools/?tabs=netcore2x).  

# <a name="java"></a>[Java](#tab/java)

+ La versión 2.x de [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools), o cualquier versión posterior. El paquete Core Tools se descarga e instala automáticamente al iniciar el proyecto de forma local. Core Tools incluye el entorno de ejecución completo de Azure Functions, por lo que la descarga y la instalación pueden tardar algún tiempo.

+ [Depurador para la extensión de Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-debug).

+ Se recomienda [Java 8](/azure/developer/java/fundamentals/java-jdk-long-term-support). Para obtener información sobre otras versiones compatibles, consulte [Versiones de Java](functions-reference-java.md#java-versions).

+ [Maven 3 o versiones posteriores](https://maven.apache.org/)

# <a name="javascript"></a>[JavaScript](#tab/nodejs)

+ La versión 2.x de [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools), o cualquier versión posterior. El paquete Core Tools se descarga e instala automáticamente al iniciar el proyecto de forma local. Core Tools incluye el entorno de ejecución completo de Azure Functions, por lo que la descarga y la instalación pueden tardar algún tiempo.

+ Node.js [, las versiones Active LTS](https://nodejs.org/) y Maintenance LTS (se recomienda la 10.14.1). Use el comando `node --version` para comprobar la versión. 

# <a name="powershell"></a>[PowerShell](#tab/powershell)

+ La versión 2.x de [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools), o cualquier versión posterior. El paquete Core Tools se descarga e instala automáticamente al iniciar el proyecto de forma local. Core Tools incluye el entorno de ejecución completo de Azure Functions, por lo que la descarga y la instalación pueden tardar algún tiempo.

+ Se recomienda [PowerShell 7](/powershell/scripting/install/installing-powershell-core-on-windows). Para obtener información sobre la versión, consulte [Versiones de PowerShell](functions-reference-powershell.md#powershell-versions).

+ [.NET Core 3.1 Runtime](https://dotnet.microsoft.com/download) y [.NET Core 2.1 Runtime](https://dotnet.microsoft.com/download/dotnet/2.1)  

+ La [extensión de PowerShell](https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell) para Visual Studio Code  

# <a name="python"></a>[Python](#tab/python)

+ La versión 2.x de [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools), o cualquier versión posterior. El paquete Core Tools se descarga e instala automáticamente al iniciar el proyecto de forma local. Core Tools incluye el entorno de ejecución completo de Azure Functions, por lo que la descarga y la instalación pueden tardar algún tiempo.

+ [Python 3.x](https://www.python.org/downloads/). Para obtener información sobre la versión, consulte [Versiones de Python](functions-reference-python.md#python-version) por el runtime de Azure Functions.

+ [Extensión de Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python) para Visual Studio Code.

---

[!INCLUDE [functions-install-vs-code-extension](../../includes/functions-install-vs-code-extension.md)]

## <a name="create-an-azure-functions-project"></a>Creación de un proyecto de Azure Functions

La extensión de Functions le permite crear un proyecto de aplicación de funciones, junto con su primera función. Los pasos siguientes muestran cómo crear una función desencadenada por HTTP en un nuevo proyecto de funciones. [Desencadenador HTTP](functions-bindings-http-webhook.md) es la plantilla de desencadenador de función más sencilla de mostrar.

1. En **Azure: Functions**, seleccione el icono **Crear función**:

    ![Creación de una función](./media/functions-develop-vs-code/create-function.png)

1. Seleccione la carpeta para el proyecto de aplicación de funciones y, a continuación, **seleccione un lenguaje para el proyecto de función**.

1. Seleccione la plantilla de función del **desencadenador HTTP** o bien puede seleccionar **Omitir por ahora** para crear un proyecto sin una función. Siempre puede [agregar una función al proyecto](#add-a-function-to-your-project) en otro momento.

    ![Elegir la plantilla del desencadenador HTTP](./media/functions-develop-vs-code/create-function-choose-template.png)

1. Escriba **HttpExample** como nombre de la función, seleccione Entrar y, después, seleccione la autorización de **Función**. Este nivel de autorización requiere que proporcione un valor para la [clave de función](functions-bindings-http-webhook-trigger.md#authorization-keys) al llamar al punto de conexión de la función.

    ![Selección de la autorización de función](./media/functions-develop-vs-code/create-function-auth.png)

    Se crea una función en el lenguaje elegido y en la plantilla para una función desencadenada por HTTP.

    ![Plantilla de función desencadenada por HTTP en Visual Studio Code](./media/functions-develop-vs-code/new-function-full.png)

### <a name="generated-project-files"></a>Archivos de proyecto generados

La plantilla de proyecto crea un proyecto en el lenguaje elegido e instala las dependencias necesarias. Para cualquier lenguaje, el proyecto nuevo contiene los archivos siguientes:

* **host.json**: permite configurar el host de Functions. Esta configuración se aplica cuando se ejecutan funciones localmente y cuando se ejecutan en Azure. Para más información, consulte la [referencia sobre host.json](functions-host-json.md).

* **local.settings.json**: mantiene la configuración que se usa al ejecutar localmente las funciones. Esta configuración solo se utiliza al ejecutar las funciones de forma local. Para más información, consulte [Archivo de configuración local](#local-settings-file).

    >[!IMPORTANT]
    >Debido a que el archivo local.settings.json puede contener secretos, tiene que excluirlo del control de origen del proyecto.

En función del lenguaje utilizado, se crean estos otros archivos:

# <a name="c"></a>[C\#](#tab/csharp)

* [Archivo de biblioteca de clases HttpExample.cs](functions-dotnet-class-library.md#functions-class-library-project) que implementa la función.

# <a name="java"></a>[Java](#tab/java)

+ Un archivo pom.xml en la carpeta raíz que define el proyecto y los parámetros de implementación, lo que incluye las dependencias del proyecto y el [versión de Java](functions-reference-java.md#java-versions). El archivo pom.xml también contiene información sobre los recursos de Azure que se crean durante una implementación.   

+ Un [archivo Functions.java](functions-reference-java.md#triggers-and-annotations) en la ruta de acceso de src que implementa la función.

# <a name="javascript"></a>[JavaScript](#tab/nodejs)

* Archivo package.json de la carpeta raíz.

* Carpeta HttpExample que contiene el [archivo de definición function.json](functions-reference-node.md#folder-structure) y el [archivo index.js](functions-reference-node.md#exporting-a-function), así como un archivo Node.js que contiene el código de la función.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

* Carpeta HttpExample que contiene el [archivo de definición function.json](functions-reference-powershell.md#folder-structure) y el archivo run.ps1, que contiene el código de función.
 
# <a name="python"></a>[Python](#tab/python)
    
* Archivo requirements.txt de nivel de proyecto que muestra los paquetes que requiere Functions.
    
* Carpeta HttpExample que contiene el [archivo de definición function.json](functions-reference-python.md#folder-structure) y el archivo \_\_init\_\_.py, que contiene el código de función.

---

Llegado este punto, se pueden [agregar enlaces de entrada y salida](#add-input-and-output-bindings) a la función. También puede [agregar una nueva función al proyecto](#add-a-function-to-your-project).

## <a name="install-binding-extensions"></a>Instalación de extensiones de enlace

Excepto los desencadenadores HTTP y del temporizador, los enlaces se implementan como paquetes de extensión. Tiene que instalar los paquetes de extensión para los desencadenadores y enlaces que los necesiten. El proceso de instalar las extensiones de enlace depende del lenguaje del proyecto.

# <a name="c"></a>[C\#](#tab/csharp)

Ejecute el comando [dotnet add package](/dotnet/core/tools/dotnet-add-package) en la ventana del terminal para instalar los paquetes de extensión que necesita en el proyecto. El siguiente comando instala la extensión de Azure Storage, que implementa los enlaces para el almacenamiento de Blob, Cola y Tabla.

```bash
dotnet add package Microsoft.Azure.WebJobs.Extensions.Storage --version 3.0.4
```

# <a name="java"></a>[Java](#tab/java)

[!INCLUDE [functions-extension-bundles](../../includes/functions-extension-bundles.md)]

# <a name="javascript"></a>[JavaScript](#tab/nodejs)

[!INCLUDE [functions-extension-bundles](../../includes/functions-extension-bundles.md)]

# <a name="powershell"></a>[PowerShell](#tab/powershell)

[!INCLUDE [functions-extension-bundles](../../includes/functions-extension-bundles.md)]

# <a name="python"></a>[Python](#tab/python)

[!INCLUDE [functions-extension-bundles](../../includes/functions-extension-bundles.md)]

---

## <a name="add-a-function-to-your-project"></a>Incorporación de una función al proyecto

Puede agregar una nueva función a un proyecto existente mediante una de las plantillas de desencadenador de funciones predefinidas. Para agregar un nuevo desencadenador de función, seleccione la tecla F1 para abrir la paleta de comandos, después, busque y ejecute el comando **Azure Functions: Create Function**. Siga las indicaciones para elegir el tipo de desencadenador y definir los atributos necesarios del desencadenador. Si el desencadenador requiere una clave de acceso o cadena de conexión para conectarse a un servicio, prepárela antes de crear el desencadenador de la función.

Los resultados de esta acción dependen del lenguaje del proyecto:

# <a name="c"></a>[C\#](#tab/csharp)

Un nuevo archivo de biblioteca de clases (. cs) de C# se agrega al proyecto.

# <a name="java"></a>[Java](#tab/java)

Se agrega un nuevo archivo Java (.java) al proyecto.

# <a name="javascript"></a>[JavaScript](#tab/nodejs)

Se crea una nueva carpeta en el proyecto. La carpeta contiene un nuevo archivo function.json y el nuevo archivo de código de JavaScript.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Se crea una nueva carpeta en el proyecto. La carpeta contiene un nuevo archivo function.json y el nuevo archivo de código de PowerShell.

# <a name="python"></a>[Python](#tab/python)

Se crea una nueva carpeta en el proyecto. La carpeta contiene un nuevo archivo function.json y el nuevo archivo de código de Python.

---

## <a name="connect-to-services"></a><a name="add-input-and-output-bindings"></a>Conexión a los servicios

Para conectar la función a otros servicios de Azure, agregue enlaces de entrada o salida. Los enlaces conectan la función a otros servicios sin que sea preciso escribir el código de conexión. El proceso para agregar enlaces depende del lenguaje del proyecto. Para más información acerca de los enlaces, consulte [Conceptos básicos sobre los enlaces y desencadenadores de Azure Functions](functions-triggers-bindings.md).

Los ejemplos siguientes se conectan a una cola de almacenamiento denominada `outqueue`, en la que está establecida la cadena de conexión para la cuenta de almacenamiento en valor de aplicación `MyStorageConnection` en local.settings.json.

# <a name="c"></a>[C\#](#tab/csharp)

Actualice el método de función para agregar el siguiente parámetro a la definición de método `Run`:

:::code language="csharp" source="~/functions-docs-csharp/functions-add-output-binding-storage-queue-cli/HttpExample.cs" range="17":::

El parámetro `msg` es de tipo `ICollector<T>`, que representa una colección de mensajes escritos en un enlace de salida cuando se completa la función. El siguiente código agrega un mensaje a la colección:

:::code language="csharp" source="~/functions-docs-csharp/functions-add-output-binding-storage-queue-cli/HttpExample.cs" range="30-31":::

 Los mensajes se envían a la cola cuando se completa la función.

Para más información, consulte el artículo [Enlaces de salida de Azure Queue Storage para Azure Functions](functions-bindings-storage-queue-output.md?tabs=csharp). Para más información en general sobre qué enlaces se pueden agregar a una función, consulte [Incorporación de enlaces a una función existente de Azure Functions](add-bindings-existing-function.md?tabs=csharp). 

# <a name="java"></a>[Java](#tab/java)

Actualice el método de función para agregar el siguiente parámetro a la definición de método `Run`:

:::code language="java" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/src/main/java/com/function/Function.java" range="20-21":::

El parámetro `msg` es del tipo `OutputBinding<T>`, donde `T` es una cadena escrita en un enlace de salida cuando se completa la función. El código siguiente establece el mensaje en el enlace de salida:

:::code language="java" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/src/main/java/com/function/Function.java" range="33-34":::

Este mensaje se envían a la cola cuando se completa la función.

Para más información, consulte el artículo [Enlaces de salida de Azure Queue Storage para Azure Functions](functions-bindings-storage-queue-output.md?tabs=java). Para más información en general sobre qué enlaces se pueden agregar a una función, consulte [Incorporación de enlaces a una función existente de Azure Functions](add-bindings-existing-function.md?tabs=java). 

# <a name="javascript"></a>[JavaScript](#tab/nodejs)

[!INCLUDE [functions-add-output-binding-vs-code](../../includes/functions-add-output-binding-vs-code.md)]

En el código de función, se obtiene acceso al enlace `msg` desde el `context`, como en este ejemplo:

:::code language="javascript" range="5-7" source="~/functions-docs-javascript/functions-add-output-binding-storage-queue-cli/HttpExample/index.js":::

Este mensaje se envían a la cola cuando se completa la función.

Para más información, consulte el artículo [Enlaces de salida de Azure Queue Storage para Azure Functions](functions-bindings-storage-queue-output.md?tabs=javascript). Para más información en general sobre qué enlaces se pueden agregar a una función, consulte [Incorporación de enlaces a una función existente de Azure Functions](add-bindings-existing-function.md?tabs=javascript). 

# <a name="powershell"></a>[PowerShell](#tab/powershell)

[!INCLUDE [functions-add-output-binding-vs-code](../../includes/functions-add-output-binding-vs-code.md)]

:::code language="powershell" range="18-19" source="~/functions-docs-powershell/functions-add-output-binding-storage-queue-cli/HttpExample/run.ps1":::

Este mensaje se envían a la cola cuando se completa la función.

Para más información, consulte el artículo [Enlaces de salida de Azure Queue Storage para Azure Functions](functions-bindings-storage-queue-output.md?tabs=powershell). Para más información en general sobre qué enlaces se pueden agregar a una función, consulte [Incorporación de enlaces a una función existente de Azure Functions](add-bindings-existing-function.md?tabs=powershell). 

# <a name="python"></a>[Python](#tab/python)

[!INCLUDE [functions-add-output-binding-vs-code](../../includes/functions-add-output-binding-vs-code.md)]

Actualice la definición de `Main` para agregar un parámetro de salida `msg: func.Out[func.QueueMessage]`, con el fin de que la definición sea similar a la del ejemplo siguiente:

:::code language="python" range="6" source="~/functions-docs-python/functions-add-output-binding-storage-queue-cli/HttpExample/__init__.py":::

El código siguiente agrega los datos de la cadena de la solicitud a la cola de salida:

:::code language="python" range="18" source="~/functions-docs-python/functions-add-output-binding-storage-queue-cli/HttpExample/__init__.py":::

Este mensaje se envían a la cola cuando se completa la función.

Para más información, consulte el artículo [Enlaces de salida de Azure Queue Storage para Azure Functions](functions-bindings-storage-queue-output.md?tabs=python). Para más información en general sobre qué enlaces se pueden agregar a una función, consulte [Incorporación de enlaces a una función existente de Azure Functions](add-bindings-existing-function.md?tabs=python). 

---

[!INCLUDE [functions-sign-in-vs-code](../../includes/functions-sign-in-vs-code.md)]

## <a name="publish-to-azure"></a>Publicación en Azure

Visual Studio Code le permite publicar el proyecto de Functions directamente en Azure. En el proceso, puede crear una aplicación de función y los recursos relacionados en su suscripción de Azure. La aplicación de función proporciona un contexto de ejecución para sus funciones. El proyecto se empaqueta e implementa en la nueva aplicación de función en su suscripción de Azure.

Al publicar desde Visual Studio Code en una nueva aplicación de funciones en Azure, se puede elegir una ruta de acceso de creación rápida de la aplicación de funciones mediante los valores predeterminados, o bien una ruta de acceso avanzada, donde se tiene más control sobre los recursos remotos que se crean. 

Al publicar desde Visual Studio Code, aprovecha la tecnología de [implementación de Zip](functions-deployment-technologies.md#zip-deploy). 

### <a name="quick-function-app-create"></a>Creación rápida de aplicaciones de funciones

Cuando elija **+ Create new function app in Azure** (+ Crear una aplicación de funciones en Azure...), la extensión genera automáticamente valores para los recursos de Azure necesarios para la aplicación de funciones. Estos valores se basan en el nombre de la aplicación de funciones que elija. Para obtener un ejemplo del uso de los valores predeterminados para publicar el proyecto en una nueva aplicación de funciones en Azure, consulte el [artículo de inicio rápido de Visual Studio Code](./create-first-function-vs-code-csharp.md#publish-the-project-to-azure).

Si desea proporcionar nombres explícitos a los recursos creados, debe elegir la ruta de acceso de creación avanzada.

### <a name="publish-a-project-to-a-new-function-app-in-azure-by-using-advanced-options"></a><a name="enable-publishing-with-advanced-create-options"></a>Publicación de un proyecto en una nueva aplicación de función en Azure mediante opciones avanzadas

Los pasos siguientes publican el proyecto en una nueva aplicación de funciones que se creó con opciones de creación avanzadas.

1. En la paleta de comandos, escriba **Azure Functions: Deploy to function app** (Azure Functions: Implementar en la aplicación de funciones).

1. Se le pedirá que **inicie sesión en Azure** si todavía no lo ha hecho. También puede **crear una cuenta de Azure gratis**. Después de iniciar sesión en el explorador, vuelva a Visual Studio Code.

1. Si tiene varias suscripciones, **seleccione una suscripción** para la aplicación de funciones y, después, elija **+ Create New Function App in Azure... _Advanced_** (+ Crear una aplicación de función en Azure... Avanzado). Esta opción _avanzada_ le proporciona más control sobre los recursos que crea en Azure. 

1. Siguiendo las indicaciones, proporcione esta información:

    | Prompt | Valor | Descripción |
    | ------ | ----- | ----------- |
    | Selección de una aplicación de funciones en Azure | Crear una aplicación de funciones en Azure | En el siguiente aviso, escriba un nombre único global que identifique la nueva aplicación de funciones y seleccione ENTRAR. Los siguientes son caracteres válidos para un nombre de aplicación de función: `a-z`, `0-9` y `-`. |
    | Seleccione un sistema operativo | Windows | La aplicación de funciones se ejecuta en Windows. |
    | Seleccione un plan de hospedaje | Plan de consumo | Se utiliza un [hospedaje del plan de consumo](consumption-plan.md) sin servidor. |
    | Seleccione un tiempo de ejecución para la nueva aplicación | El lenguaje del proyecto | El entorno de ejecución tiene que coincidir con el proyecto que va a publicar. |
    | Seleccionar un grupo de recursos para los nuevos recursos | Crear un nuevo grupo de recursos | En el siguiente aviso, escriba un nombre de grupo de recursos, como `myResourceGroup`, y presione ENTRAR. También puede seleccionar un grupo de recursos existente. |
    | Seleccione una cuenta de almacenamiento | Creación de una nueva cuenta de almacenamiento | En el siguiente aviso, escriba un nombre único global para la nueva cuenta de almacenamiento que va a usar la aplicación de funciones y seleccione ENTRAR. Los nombres de las cuentas de almacenamiento deben tener entre tres y 24 caracteres y solo pueden incluir números y letras en minúscula. También puede seleccionar una cuenta existente. |
    | Seleccione una ubicación para los nuevos recursos | region | Seleccione una ubicación en una [región](https://azure.microsoft.com/regions/) próxima a usted o a otros servicios a los que accedan las funciones. |

    Se muestra una notificación después de crear la aplicación de funciones y se aplica el paquete de implementación. Seleccione **View Output** (Ver salida) en esta notificación para ver la creación y los resultados de la implementación, incluidos los recursos de Azure que ha creado.

### <a name="get-the-url-of-an-http-triggered-function-in-azure"></a><a name="get-the-url-of-the-deployed-function"></a>Obtención de la dirección URL de una función desencadenada mediante HTTP en Azure

Para llamar a una función desencadenada por HTTP desde un cliente, necesitará la dirección URL de la función cuando se implementa en la aplicación de funciones. Esta dirección URL incluye todas las teclas de función necesarias. Puede usar la extensión para obtener estas direcciones URL para las funciones implementadas. Si solo desea ejecutar la función remota en Azure, [use la funcionalidad Execute function now](#run-functions-in-azure) de la extensión.

1. Seleccione F1 para abrir la paleta de comandos, y busque y ejecute el comando **Azure Functions: Copy Function URL** (Copiar la dirección URL de la función).

1. Siga las indicaciones para seleccionar la aplicación de funciones en Azure y, a continuación, el desencadenador HTTP específico que desea invocar.

La dirección URL de la función se copia en el Portapapeles, junto con las claves necesarias pasadas mediante el parámetro de consulta `code`. Use una herramienta de HTTP para enviar las solicitudes POST, o un explorador para las solicitudes GET a la función remota.  

Al obtener la dirección URL de las funciones en Azure, la extensión usa su cuenta de Azure para recuperar automáticamente las claves que necesita para iniciar la función. [Más información sobre las claves de acceso de las funciones](security-concepts.md#function-access-keys). El inicio de funciones no desencadenadas por HTTP requiere el uso de la clave de administración.

## <a name="republish-project-files"></a>Volver a publicar los archivos de proyecto

Al configurar la [implementación continua](functions-continuous-deployment.md), la aplicación de funciones de Azure se actualiza cuando se actualizan los archivos de origen en la ubicación de origen conectada. Aunque se recomienda la implementación continua, también puede volver a publicar las actualizaciones del archivo de proyecto desde Visual Studio Code.

> [!IMPORTANT]
> La publicación en una aplicación de función existente sobrescribe el contenido de esa aplicación en Azure.

[!INCLUDE [functions-republish-vscode](../../includes/functions-republish-vscode.md)]

## <a name="run-functions"></a>Ejecución de funciones

La extensión de Azure Functions permite ejecutar funciones individuales, ya sea en el proyecto del equipo de desarrollo local o en la suscripción de Azure. 

En el caso de las funciones de desencadenador HTTP, la extensión llama al punto de conexión HTTP. En los restantes tipos de desencadenadores, llama a las API de administrador para iniciar la función. El cuerpo del mensaje de la solicitud enviada a la función depende del tipo de desencadenador. Si un desencadenador requiere datos de prueba, se le pedirá que escriba los datos en un formato JSON específico.

### <a name="run-functions-in-azure"></a>Ejecución de funciones en Azure

Para ejecutar la función en Azure desde Visual Studio Code. 

1. En la paleta de comandos, escriba **Azure Functions: Execute function now** y elija su suscripción de Azure. 

1. Elija la aplicación de funciones existente de Azure en la lista. Si no ve la aplicación de funciones, asegúrese de que ha iniciado sesión en la suscripción correcta. 

1. Elija en la lista la función que desea ejecutar y escriba el cuerpo del mensaje de la solicitud en **Enter request body** (Especificar cuerpo de la solicitud). Presione Entrar para enviar este mensaje de solicitud a la función. El texto predeterminado de **Enter request body** (Especificar cuerpo de la solicitud) debe indicar el formato del cuerpo. Si la aplicación de funciones no tiene funciones, se muestra un error de notificación con este error. 

1. Cuando la función se ejecuta en Azure y devuelve una respuesta, se genera una notificación en Visual Studio Code.
 
La función también se puede ejecutar desde el área **Azure: Functions**; para ello, haga clic con el botón derecho (Ctrl + clic en Mac) la función que desea ejecutar desde la aplicación de funciones en la suscripción de Azure y seleccione **Execute Function Now...** (Ejecutar función ahora).

Al ejecutar funciones en Azure, la extensión usa su cuenta de Azure para recuperar automáticamente las claves que necesita para iniciar la función. [Más información sobre las claves de acceso de las funciones](security-concepts.md#function-access-keys). El inicio de funciones no desencadenadas por HTTP requiere el uso de la clave de administración.

### <a name="run-functions-locally"></a>Ejecución local de funciones

El tiempo de ejecución local es el mismo tiempo de ejecución que hospeda la aplicación de funciones en Azure. La configuración local se lee desde el [archivo local.settings.json](#local-settings-file). Para ejecutar el proyecto de Functions localmente, debe cumplir varios [requisitos adicionales](#run-local-requirements).

#### <a name="configure-the-project-to-run-locally"></a>Configuración del proyecto para que se ejecute localmente

El tiempo de ejecución de Functions usa una cuenta de Azure Storage internamente para todos los tipos de desencadenadores distintos de HTTP y webhooks. Por tanto, tiene que establecer la clave **Values.AzureWebJobsStorage** en una cadena de conexión de cuenta de Azure Storage válida.

En esta sección se utiliza la [extensión de Azure Storage para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestorage) con el [Explorador de Azure Storage](https://storageexplorer.com/) para conectarse y recuperar la cadena de conexión de almacenamiento.

Para establecer la cadena de conexión de cuenta de almacenamiento:

1. En Visual Studio, abra **Cloud Explorer**, expanda **Cuenta de almacenamiento** > **Su cuenta de almacenamiento** y seleccione **Propiedades** y copie el valor de **Cadena de conexión principal**.

2. En el proyecto, abra el archivo local.settings.json y establezca el valor de la clave **AzureWebJobsStorage** en la cadena de conexión que copió.

3. Repita el paso anterior para agregar claves únicas a la matriz de **valores** para cualquier otra conexión que requieran sus funciones.

Para más información, consulte [Archivo de configuración local](#local-settings-file).

#### <a name="debug-functions-locally"></a><a name="debugging-functions-locally"></a>Depuración local de funciones  

Para depurar las funciones, seleccione F5. Si aún no ha descargado [Core Tools][Azure Functions Core Tools], se le pedirá que lo haga. Una vez que Core Tools esté instalado y en ejecución, la salida se muestra en el Terminal. Esto equivale a ejecutar el comando `func host start` de Core Tools desde el terminal, pero con tareas de compilación adicionales y un depurador asociado.  

Con el proyecto en ejecución, puede usar la característica **Execute Function Now...** (Ejecutar función ahora) de la extensión para desencadenar las funciones como lo haría al implementar el proyecto en Azure. Con el proyecto ejecutándose en modo de depuración, los puntos de interrupción se alcanzan en Visual Studio Code tal como se esperaría. 

1. En la paleta de comandos, escriba **Azure Functions: Execute function now** y elija **Local project** (Proyecto local). 

1. Elija en el proyecto la función que desea ejecutar y escriba el cuerpo del mensaje de la solicitud en **Enter request body** (Especificar cuerpo de la solicitud). Presione Entrar para enviar este mensaje de solicitud a la función. El texto predeterminado de **Enter request body** (Especificar cuerpo de la solicitud) debe indicar el formato del cuerpo. Si la aplicación de funciones no tiene funciones, se muestra un error de notificación con este error. 

1. Cuando la función se ejecuta localmente y después de que se recibe una respuesta, se genera una notificación en Visual Studio Code. La información sobre la ejecución de la función se muestra en el panel **Terminal**.

La ejecución local de funciones no requiere el uso de claves. 

[!INCLUDE [functions-local-settings-file](../../includes/functions-local-settings-file.md)]

De manera predeterminada, estas opciones de configuración no se migran automáticamente cuando el proyecto se publica en Azure. Una vez finalizada la publicación, tiene la opción de publicar la configuración en local.settings.json en Azure. Para más información, consulte [Publicación de la configuración de la aplicación](#publish-application-settings).

Los valores de **ConnectionStrings** nunca se publican.

Los valores de la configuración de la aplicación de funciones también se pueden leer en el código como variables de entorno. Para más información, consulte las secciones Variables de entorno de estos artículos de referencia específicos de cada lenguaje:

* [C# precompilado](functions-dotnet-class-library.md#environment-variables)
* [Script de C# (.csx)](functions-reference-csharp.md#environment-variables)
* [Java](functions-reference-java.md#environment-variables)
* [JavaScript](functions-reference-node.md#environment-variables)
* [PowerShell](functions-reference-powershell.md#environment-variables)
* [Python](functions-reference-python.md#environment-variables)

## <a name="application-settings-in-azure"></a>Configuración de la aplicación en Azure

La configuración en el archivo local.settings.json en el proyecto debe ser la misma que la configuración de la aplicación en la aplicación de funciones en Azure. Todos los valores que agregue a local.settings.json también deben agregarse a la aplicación de funciones en Azure. Esta configuración no se carga automáticamente cuando publica el proyecto. Del mismo modo, cualquier configuración que se crea en la aplicación de funciones [en el portal](functions-how-to-use-azure-function-app-settings.md#settings) tiene que descargarse para el proyecto local.

### <a name="publish-application-settings"></a>Publicación de la configuración de la aplicación

La manera más sencilla de publicar la configuración requerida en la aplicación de funciones en Azure es usar el vínculo **Configuración de carga** que aparece después de publicado el proyecto:

![Actualización de la configuración de la aplicación](./media/functions-develop-vs-code/upload-app-settings.png)

También puede publicar la configuración mediante el comando **Azure Functions: Upload Local Settings** (Cargar configuración local) en la paleta de comandos. Puede agregar valores individuales a la configuración de la aplicación en Azure mediante el uso del comando **Azure Functions: Add new setting** (Agregar nueva configuración).

> [!TIP]
> Asegúrese de guardar el archivo local.settings.json antes de publicarlo.

Si el archivo local está cifrado, se descifra, se publica y se cifra de nuevo. Si hay configuraciones con valores en conflicto en las dos ubicaciones, se le pedirá que elija cómo proceder.

Visualice la configuración de la aplicación en el área **Azure: Funciones** expandiendo su suscripción, la aplicación de funciones, y **Configuración de aplicaciones**.

![Visualización de la configuración de la aplicación de funciones en Visual Studio Code](./media/functions-develop-vs-code/view-app-settings.png)

### <a name="download-settings-from-azure"></a>Descarga de configuración desde Azure

Si ha creado la configuración de la aplicación en Azure, puede descargarla en el archivo local.settings.json mediante el comando **Azure Functions: Descargar configuración remota**.

Como en el caso de las cargas, si el archivo local está cifrado, se descifra, se actualiza y se cifra de nuevo. Si hay configuraciones con valores en conflicto en las dos ubicaciones, se le pedirá que elija cómo proceder.

## <a name="monitoring-functions"></a>Supervisión de funciones

Cuando se realice la [ejecución de funciones localmente](#run-functions-locally), los datos de registro se transmiten a la consola del terminal. También puede obtener datos de registro cuando se ejecuta el proyecto de funciones en una aplicación de funciones en Azure. Puede conectarse a los registros de streaming en Azure para ver los datos de registro prácticamente en tiempo real, o puede habilitar Application Insights para un reconocimiento más completo de cómo se comporta la aplicación de funciones.

### <a name="streaming-logs"></a>Registros de streaming

Al implementar una aplicación, suele resultar útil ver la información de registro casi en tiempo real. Puede ver una secuencia de archivos de registro que generan las funciones. Esta salida es un ejemplo de registros de streaming para una solicitud a una función desencadenada por HTTP:

![Salida de registros de streaming para el desencadenador HTTP](media/functions-develop-vs-code/streaming-logs-vscode-console.png)

Para más información, consulte [Registros de streaming](functions-monitoring.md#streaming-logs).

[!INCLUDE [functions-enable-log-stream-vs-code](../../includes/functions-enable-log-stream-vs-code.md)]

> [!NOTE]
> Los registros de secuencias admiten una sola instancia del host de Functions. Cuando la función se escala a varias instancias, no se muestran los datos de otras instancias en la secuencia de registro. [Live Metrics Stream](../azure-monitor/app/live-stream.md) en Application Insights admite varias instancias. Si bien también funciona casi en tiempo real, el análisis de secuencias también se basa en [datos muestreados](configure-monitoring.md#configure-sampling).

### <a name="application-insights"></a>Application Insights

Recomendamos supervisar la ejecución de las funciones mediante la integración de la aplicación de funciones con Application Insights. Cuando crea una aplicación de funciones en Azure Portal, esta integración se realiza de forma predeterminada. Al crear la aplicación de funciones durante la publicación en Visual Studio, debe integrar Application Insights usted mismo. Para conocer más, consulte [Habilitación de la integración de Application Insights](configure-monitoring.md#enable-application-insights-integration).

Para obtener más información sobre la supervisión mediante Application Insights, consulte [Supervisión de Azure Functions](functions-monitoring.md).

## <a name="c-script-projects"></a>Proyectos de script C\#

De forma predeterminada, todos los proyectos C# se crean como [proyectos de biblioteca de clases de C# compilados](functions-dotnet-class-library.md). Si prefiere en su lugar trabajar con proyectos de script de C#, tiene que seleccionar script de C# como el lenguaje predeterminado en la configuración de la extensión de Azure Functions.

1. Seleccione **Archivo** > **Preferencias** > **Configuración**.

1. Vaya a **Configuración de usuario** > **Extensiones** > **Azure Functions**.

1. Elija **Script de C#** en **Función de Azure: Lenguaje del proyecto**.

Después de haber completado estos pasos, las llamadas realizadas a la instancia subyacente de Core Tools incluyen la opción `--csx`, que genera y publica archivos de proyecto de script de C# (.csx). Cuando tenga el lenguaje predeterminado especificado, todos los proyectos creados son, de forma predeterminada, proyectos de script de C#. Cuando hay establecido un lenguaje predeterminado, no se le va a pedir que elija un lenguaje de proyecto. Para crear proyectos con otros lenguajes, tiene que cambiar esta configuración o quitarla del archivo de usuario settings.json. Después de quitar esta configuración, al crear un proyecto, se le volverá a pedir que elija su lenguaje.

## <a name="command-palette-reference"></a>Referencia de la paleta de comandos

La extensión de Azure Functions proporciona una interfaz gráfica útil en el área de Azure para interactuar con las aplicación de funciones en Azure. La misma funcionalidad también está disponible como comandos en la paleta de comandos (F1). Están disponibles estos comandos Azure Functions:

|Comando de Azure Functions  | Descripción  |
|---------|---------|
|**Add new settings** (Agregar nueva configuración)  |  Crea una nueva configuración de aplicación en Azure. Para más información, consulte [Publicación de la configuración de la aplicación](#publish-application-settings). También es posible que tenga que [descargar esta configuración en la configuración local](#download-settings-from-azure). |
| **Configure Deployment Source** (Configurar origen de implementación) | Conecta la aplicación de funciones de Azure con un repositorio Git local. Para más información consulte [Implementación continua para Azure Functions](functions-continuous-deployment.md). |
| **Connect to GitHub Repository** (Conectar al repositorio de GitHub) | Conecta la aplicación de funciones con un repositorio de GitHub. |
| **Copy Function URL** (Copiar la dirección URL de la función) | Obtiene la dirección URL remota de una función desencadenada por HTTP que se ejecuta en Azure. Para más información, consulte [cómo obtener la dirección URL de la función implementada](#get-the-url-of-the-deployed-function). |
| **Create function app in Azure** (Creación de una aplicación de funciones en Azure) | Crea una nueva aplicación de funciones en su suscripción de Azure. Para más información, consulte [cómo publicar una nueva aplicación de funciones en Azure](#publish-to-azure).        |
| **Decrypt Settings** (Descifrar la configuración) | Descifra la [configuración local](#local-settings-file) que se ha cifrado mediante **Azure Functions: Encrypt settings** (Cifrar configuración).  |
| **Delete Function App** (Eliminar aplicación de funciones) | Quita una aplicación de funciones existente de la suscripción de Azure. Cuando no hay ninguna otra aplicación en el plan de App Service, se le da la opción de eliminar este también. Otros recursos, como las cuentas de almacenamiento y grupos de recursos, no se eliminan. Para quitar todos los recursos, en su lugar debe [eliminar el grupo de recursos](functions-add-output-binding-storage-queue-vs-code.md#clean-up-resources). El proyecto local no se verá afectado. |
|**Delete Function**(Eliminar función)  | Quita una función existente de una aplicación de funciones en Azure. Dado que esta eliminación no afecta al proyecto local, en su lugar, considere la posibilidad de quitar la función localmente y después [volver a publicar el proyecto](#republish-project-files). |
| **Delete Proxy** (Eliminar proxy) | Quita a un servidor proxy de Azure Functions desde la aplicación de funciones en Azure. Para más información sobre servidores proxy, consulte [Uso de Azure Functions Proxies](functions-proxies.md). |
| **Delete Setting** (Eliminar opción) | Elimina una configuración de aplicación de funciones de Azure. Esta eliminación no afecta a la configuración del archivo local.settings.json. |
| **Disconnect from Repo** (Desconectar del repositorio)  | Quita la conexión de [implementación continua](functions-continuous-deployment.md) entre una aplicación de funciones en Azure y un repositorio de control de código fuente. |
| **Download Remote Settings** (Descargar configuración remota) | Descarga la configuración de la aplicación de funciones elegida en Azure en el archivo local.settings.json. Si el archivo local está cifrado, se descifra, se actualiza y se cifra de nuevo. Si hay configuraciones con valores en conflicto en las dos ubicaciones, se le pedirá que elija cómo proceder. Asegúrese de guardar los cambios en el archivo local.settings.json antes de ejecutar este comando. |
| **Edit Settings** (Editar configuración) | Cambia el valor de una configuración de aplicación de funciones existente en Azure. Este comando no afecta a la configuración en el archivo local.settings.json.  |
| **Encrypt settings** (Cifrar configuración) | Cifra los elementos individuales de la matriz de `Values` en la [configuración local](#local-settings-file). En este archivo, `IsEncrypted` también se establece en `true`, lo que indica al tiempo de ejecución local que descifre la configuración antes de usarla. Cifre la configuración local para reducir el riesgo de pérdida de información valiosa. En Azure, las configuraciones de aplicación siempre se almacenan cifradas. |
| **Execute Function Now** (Ejecutar la función ahora) | Inicia manualmente una función mediante las API de administración. Este comando se usa para realizar pruebas, tanto localmente durante la depuración como con las funciones que se ejecutan en Azure. Al desencadenar una función en Azure, en primer lugar la extensión obtiene automáticamente una clave de administración, que se usa para llamar a las API de administración remota que inician las funciones en Azure. El cuerpo del mensaje enviado a la API depende del tipo de desencadenador. Los desencadenadores del temporizador no requieren que pase ningún dato. |
| **Initialize Project for Use with VS Code** (Inicializar el proyecto para su uso con VS Code) | Agrega los archivos de proyecto de Visual Studio Code necesarios a un proyecto existente de Functions. Use este comando para trabajar con un proyecto creado mediante Core Tools. |
| **Install of Update Azure Functions Core Tools** (Instalación de actualización de Azure Functions Core Tools) | Instala o actualiza [Azure Functions Core Tools], que se utilizan para ejecutar funciones localmente. |
| **Redeploy** (Volver a implementar)  | Le permite volver a implementar archivos de proyecto desde un repositorio de Git conectado a una implementación específica de Azure. Para volver a publicar actualizaciones locales de Visual Studio Code, [vuelva a publicar el proyecto](#republish-project-files). |
| **Rename Settings** (Cambiar el nombre de configuración) | Cambia el nombre clave de una configuración de aplicación de funciones existente en Azure. Este comando no afecta a la configuración en el archivo local.settings.json. Después de cambiar el nombre de configuración en Azure, debe [descargar esos cambios en el proyecto local](#download-settings-from-azure). |
| **Restart** (Reiniciar) | Reinicia la aplicación de funciones en Azure. La implementación de actualizaciones, también reinicia la aplicación de funciones. |
| **Set AzureWebJobsStorage**| Establece el valor de la configuración de la aplicación`AzureWebJobsStorage`. Esta configuración la requiere Azure Functions. Se establece cuando se crea una aplicación de funciones en Azure. |
| **Iniciar** | Inicia una aplicación de funciones detenida en Azure. |
| **Start Streaming Logs** (Iniciar registros de streaming) | Inicia los registros de streaming para la aplicación de funciones en Azure. Use los registros de streaming durante la solución de problemas remota en Azure si tiene que ver esta información en tiempo casi real. Para más información, consulte [Registros de streaming](#streaming-logs). |
| **Detención** | Detiene una aplicación de funciones que se esté ejecutando en Azure. |
| **Stop Streaming Logs** (Detener registros de streaming) | Detiene los registros de streaming para la aplicación de funciones en Azure. |
| **Toggle as Slot Setting** (Alternar como configuración de ranura) | Cuando está habilitada esta opción, se asegura de que se conserve una configuración de aplicación para una ranura de implementación determinada. |
| **Uninstall Azure Functions Core Tools** (Desinstalar Azure Functions Core Tools) | Quita Azure Functions Core Tools, que es necesaria para la extensión. |
| **Upload local settings** (Cargar configuración local) | Carga la configuración del archivo local.settings.json a la aplicación de funciones elegida en Azure. Si el archivo local está cifrado, se descifra, se carga y se cifra de nuevo. Si hay configuraciones con valores en conflicto en las dos ubicaciones, se le pedirá que elija cómo proceder. Asegúrese de guardar los cambios en el archivo local.settings.json antes de ejecutar este comando. |
| **View Commit in GitHub** (Ver confirmación en GitHub) | Muestra la última confirmación en una implementación específica cuando la aplicación de funciones está conectada a un repositorio. |
| **View Deployment Logs** (Ver registros de implementación) | Muestra los registros para una implementación específica de la aplicación de funciones en Azure. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Functions Core Tools, consulte [Uso de Azure Functions Core Tools](functions-run-local.md).

Para más información sobre el desarrollo de funciones como las bibliotecas de clases de .NET, consulte [Referencia para desarrolladores de C# de Azure Functions](functions-dotnet-class-library.md). En este artículo también incluye vínculos a ejemplos de cómo usar atributos para declarar los distintos tipos de enlaces compatibles con Azure Functions.

[Extensión de Azure Functions para Visual Studio Code]: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions
[Azure Functions Core Tools]: functions-run-local.md