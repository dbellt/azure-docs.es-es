---
title: 'Creación de una función de Java desde la línea de comandos: Azure Functions'
description: Aprenda a crear una función de Java desde la línea de comandos y, luego, a publicar el proyecto local en el hospedaje sin servidor en Azure Functions.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-java
- devx-track-azurecli
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./create-first-function-cli-java-uiex
ms.openlocfilehash: 1a1571e83e33ee0aec40df4cf68b3d01b6ef5f04
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787510"
---
# <a name="quickstart-create-a-java-function-in-azure-from-the-command-line"></a>Inicio rápido: Creación de una función de Java en Azure desde la línea de comandos

[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

En este artículo se usan herramientas de línea de comandos para crear una función de Java que responda a solicitudes HTTP. Después de probar el código localmente, se implementa en el entorno sin servidor de Azure Functions.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

> [!NOTE]
> Si Maven no es la herramienta de desarrollo elegida, consulte nuestros tutoriales análogos para desarrolladores de Java con [Gradle](./functions-create-first-java-gradle.md), [IntelliJ IDEA](/azure/developer/java/toolkit-for-intellij/quickstart-functions) y [Visual Studio Code](create-first-function-vs-code-java.md).

## <a name="configure-your-local-environment"></a>Configuración del entorno local

Antes de empezar, debe disponer de lo siguiente:

+ Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.x.

+ La [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o cualquier versión posterior.

+ [Kit para desarrolladores de Java](/azure/developer/java/fundamentals/java-jdk-long-term-support), versión 8 u 11. La variable de entorno `JAVA_HOME` se debe establecer en la ubicación de instalación de la versión correcta del JDK.     

+ [Apache Maven](https://maven.apache.org), versión 3.0 o posterior.

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

+ En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que la versión de Azure Functions Core Tools es la 3.x.

+ Ejecute `az --version` para comprobar que la versión de la CLI de Azure es la 2.4 o posterior.

+ Ejecute `az login` para iniciar sesión en Azure y comprobar una suscripción activa.

## <a name="create-a-local-function-project"></a>Creación de un proyecto de función local

En Azure Functions, un proyecto de función es un contenedor para una o varias funciones individuales que responden a un desencadenador específico. Todas las funciones de un proyecto comparten las mismas configuraciones locales y de hospedaje. En esta sección, se crea un proyecto de función que contiene una sola función.

1. En una carpeta vacía, ejecute el comando siguiente para generar el proyecto de Functions desde un [arquetipo Maven](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html). 

    # <a name="bash"></a>[Bash](#tab/bash)
    
    ```bash
    mvn archetype:generate -DarchetypeGroupId=com.microsoft.azure -DarchetypeArtifactId=azure-functions-archetype -DjavaVersion=8
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    mvn archetype:generate "-DarchetypeGroupId=com.microsoft.azure" "-DarchetypeArtifactId=azure-functions-archetype" "-DjavaVersion=8" 
    ```
    
    # <a name="cmd"></a>[Cmd](#tab/cmd)
    
    ```cmd
    mvn archetype:generate "-DarchetypeGroupId=com.microsoft.azure" "-DarchetypeArtifactId=azure-functions-archetype" "-DjavaVersion=8"
    ```
    
    ---

    > [!IMPORTANT]
    > + Use `-DjavaVersion=11` si quiere que las funciones se ejecuten en Java 11. Para obtener más información, consulte las [versiones de Java](functions-reference-java.md#java-versions). 
    > + Para llevar a cabo los pasos de este artículo, la variable de entorno `JAVA_HOME` se debe establecer en la ubicación de instalación de la versión correcta del JDK.

1. Maven le pide los valores necesarios para finalizar la generación del proyecto en la implementación.   
    Indique los siguientes valores cuando se le solicite:

    | Prompt | Value | Descripción |
    | ------ | ----- | ----------- |
    | **groupId** | `com.fabrikam` | Un valor que identifica de forma única su proyecto entre todos los demás y que sigue las [reglas de nomenclatura de paquetes](https://docs.oracle.com/javase/specs/jls/se6/html/packages.html#7.7) de Java. |
    | **artifactId** | `fabrikam-functions` | Un valor que es el nombre del archivo jar, sin un número de versión. |
    | **version** | `1.0-SNAPSHOT` | Elija el valor predeterminado. |
    | **package** | `com.fabrikam` | Un valor que es el paquete de Java para el código de función generado. Use el valor predeterminado. |

1. Escriba `Y` o presione Entrar para confirmar.

    Maven crea los archivos del proyecto en una carpeta nueva llamada _artifactId_ que, en este ejemplo, es `fabrikam-functions`. 

1. Vaya a la carpeta del proyecto:

    ```console
    cd fabrikam-functions
    ```

    Esta carpeta contiene varios archivos del proyecto, incluidos los archivos de configuración [local.settings.json](functions-run-local.md#local-settings-file) y [host.json](functions-host-json.md). Como *local.settings.json* puede contener secretos descargados de Azure, el archivo se excluye del control de código fuente de forma predeterminada en el archivo *.gitignore*.

### <a name="optional-examine-the-file-contents"></a>(Opcional) Examen del contenido del archivo

Si lo desea, puede ir a [Ejecución local de la función](#run-the-function-locally) y examine el contenido del archivo posteriormente.

#### <a name="functionjava"></a>Function.java
*Function.java* contiene un método `run` que recibe datos de solicitud en la variable `request` de tipo [HttpRequestMessage](/java/api/com.microsoft.azure.functions.httprequestmessage) que está decorada con la anotación [HttpTrigger](/java/api/com.microsoft.azure.functions.annotation.httptrigger), que define el comportamiento del desencadenador. 

:::code language="java" source="~/azure-functions-samples-java/src/main/java/com/functions/Function.java":::

El mensaje de respuesta lo genera la API [HttpResponseMessage.Builder](/java/api/com.microsoft.azure.functions.httpresponsemessage.builder).

#### <a name="pomxml"></a>pom.xml

La configuración de los recursos de Azure creados para hospedar la aplicación se define en el elemento **configuration** del complemento con un valor de **groupId** de `com.microsoft.azure` en el archivo pom.xml generado. Por ejemplo, el elemento de configuración siguiente indica a una implementación basada en Maven que cree una aplicación de función en el grupo de recursos `java-functions-group` de la región `westus`. La propia aplicación de función se ejecuta en Windows hospedada en el plan `java-functions-app-service-plan`, que, de forma predeterminada, es un plan de consumo sin servidor.

:::code language="java" source="~/azure-functions-samples-java/pom.xml" range="62-107":::

Puede cambiar esta configuración para controlar cómo se crean los recursos en Azure, por ejemplo, cambiando `runtime.os` de `windows` a `linux` antes de la implementación inicial. Para obtener una lista completa de los valores de configuración admitidos por el complemento Maven, consulte los [detalles de configuración](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Functions:-Configuration-Details).

#### <a name="functiontestjava"></a>FunctionTest.java

El arquetipo también genera una prueba unitaria para la función. Al cambiar la función para agregar enlaces o agregar nuevas funciones al proyecto, también deberá modificar las pruebas en el archivo *FunctionTest.java*.

## <a name="run-the-function-locally"></a>Ejecución local de la función

1. Para ejecutar una función, inicie el host en tiempo de ejecución local de Azure Functions desde la carpeta *LocalFunctionProj*:

    ```console
    mvn clean package
    mvn azure-functions:run
    ```
    
    Hacia el final de la salida, deberían aparecer las líneas siguientes:
    
    <pre>
    ...
    
    Now listening on: http://0.0.0.0:7071
    Application started. Press Ctrl+C to shut down.
    
    Http Functions:
    
            HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
    ...
    
    </pre>
    
    > [!NOTE]  
    > Si HttpExample no aparece como se ha mostrado arriba, es probable que haya iniciado el host desde fuera de la carpeta raíz del proyecto. En ese caso, use **Ctrl**+**C** para detener el host, vaya a la carpeta raíz del proyecto y vuelva a ejecutar el comando anterior.

1. Copie la dirección URL de la función `HttpExample` de esta salida en un explorador y anexe la cadena de consulta `?name=<YOUR_NAME>`, lo que hará que la dirección URL completa sea `http://localhost:7071/api/HttpExample?name=Functions`. El explorador debe mostrar un mensaje que devuelva el valor de la cadena de consulta. El terminal en el que inició el proyecto también muestra la salida del registro cuando realiza solicitudes.

1. Cuando termine, presione **Ctrl**+**C** y seleccione `y` para detener el host de Functions.

## <a name="deploy-the-function-project-to-azure"></a>Implementación del proyecto de función en Azure

La primera vez que implementa un proyecto de funciones, se crea una aplicación de funciones y los recursos relacionados en Azure. La configuración de los recursos de Azure que se crean para hospedar la aplicación se define en el [archivo pom.xml](#pomxml). En este artículo, aceptará los valores predeterminados.

> [!TIP]
> Para crear una aplicación de función que se ejecute en Linux en lugar de Windows, cambie el elemento `runtime.os` del archivo pom.xml de `windows` a `linux`. La ejecución de Linux en un plan de consumo se admite en [estas regiones](https://github.com/Azure/azure-functions-host/wiki/Linux-Consumption-Regions). No puede tener aplicaciones que se ejecuten en Linux y aplicaciones que se ejecuten en Windows en el mismo grupo de recursos.

1. Antes de poder implementar, inicie sesión en su suscripción a Azure mediante la CLI de Azure o Azure PowerShell. 

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
    ```azurecli
    az login
    ```

    El comando [az login](/cli/azure/reference-index#az_login) inicia sesión en su cuenta de Azure.

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell) 
    ```azurepowershell
    Connect-AzAccount
    ```

    El cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) inicia sesión en su cuenta de Azure.

    ---

1. Use el siguiente comando para implementar el proyecto en una nueva aplicación de funciones.

    ```console
    mvn azure-functions:deploy
    ```
    
    Esta operación crea los siguientes recursos en Azure:
    
    + Un grupo de recursos Se nombra como _java-functions-group_.
    + Una cuenta de almacenamiento Necesaria con Functions. El nombre se genera aleatoriamente según los requisitos de nombre de la cuenta de almacenamiento.
    + Plan de hospedaje. El hospedaje sin servidor de la aplicación de función en la región _westus_. El nombre es _java-functions-app-service-plan_.
    + Aplicación de funciones. Una aplicación de funciones es la unidad de implementación y ejecución de las funciones. El nombre se genera de forma aleatoria según el valor _artifactId_, anexado con un número generado de forma aleatoria.
    
    La implementación empaqueta los archivos de proyecto y los implementa en la nueva aplicación de función mediante la [implementación de archivos ZIP](functions-deployment-technologies.md#zip-deploy). El código se ejecuta desde el paquete de implementación en Azure.

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Si continúa con el [paso siguiente](#next-steps) y agrega un enlace de salida de la cola de Azure Storage, conserve todos los recursos intactos, ya que va a crear a partir de lo que ya ha hecho.

De lo contrario, use el siguiente comando para eliminar el grupo de recursos y todos los recursos que contiene para evitar incurrir en costos adicionales.

 # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az group delete --name java-functions-group
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Remove-AzResourceGroup -Name java-functions-group
```

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-java)
