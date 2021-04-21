---
title: 'Creación de una función de Java desde la línea de comandos: Azure Functions'
description: Aprenda a crear una función de Java desde la línea de comandos y, luego, a publicar el proyecto local en el hospedaje sin servidor en Azure Functions.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-java
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: a270f482438129b7b07506744bf3f9489b242900
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787492"
---
# <a name="quickstart-create-a-java-function-in-azure-from-the-command-line"></a>Inicio rápido: Creación de una función de Java en Azure desde la línea de comandos

> [!div class="op_single_selector" title1="Seleccione el lenguaje de la función: "]
> - [Java](create-first-function-cli-java.md)
> - [Python](create-first-function-cli-python.md)
> - [C#](create-first-function-cli-csharp.md)
> - [JavaScript](create-first-function-cli-node.md)
> - [PowerShell](create-first-function-cli-powershell.md)
> - [TypeScript](create-first-function-cli-typescript.md)

Utilice las herramientas de la línea de comandos para crear una función de Java que responda a solicitudes HTTP. Pruebe el código localmente y, a continuación, impleméntelo en el entorno sin servidor de Azure Functions.

Este inicio rápido supone un pequeño costo en su <abbr title="El perfil que mantiene la información de facturación en relación con el uso de Azure.">cuenta de Azure</abbr>.

Si Maven no es la herramienta de desarrollo elegida, consulte nuestros tutoriales análogos para desarrolladores de Java con [Gradle](./functions-create-first-java-gradle.md), [IntelliJ IDEA](/azure/developer/java/toolkit-for-intellij/quickstart-functions) y [Visual Studio Code](create-first-function-vs-code-java.md).

## <a name="1-prepare-your-environment"></a>1. Preparación del entorno

Antes de empezar, debe disponer de lo siguiente:

+ Una cuenta de Azure con una suscripción activa. <abbr title="La estructura organizativa básica en la que administra los recursos de Azure, que normalmente se asocia a un individuo o departamento dentro de una organización.">subscription</abbr>. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.x.

+ La [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o cualquier versión posterior.

+ [Kit para desarrolladores de Java](/azure/developer/java/fundamentals/java-jdk-long-term-support), versión 8 u 11. La variable de entorno `JAVA_HOME` se debe establecer en la ubicación de instalación de la versión correcta del JDK.

+ [Apache Maven](https://maven.apache.org), versión 3.0 o posterior.

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

+ En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que <abbr title="El conjunto de herramientas de línea de comandos para trabajar con Azure Functions en su equipo local.">Azure Functions Core Tools</abbr> representan la versión 3.x.

+ Ejecute `az --version` para comprobar que la versión de la CLI de Azure es la 2.4 o posterior.

+ Ejecute `az login` para iniciar sesión en Azure y comprobar una suscripción activa.

<br>
<hr/>

## <a name="2-create-a-local-function-project"></a>2. Creación de un proyecto de función local

En Azure Functions, un proyecto de función es un contenedor para una o varias funciones individuales que responden a un desencadenador específico. <abbr title="El tipo de evento que invoca el código de la función, como una solicitud HTTP, un mensaje de cola o una hora específica.">desencadenador</abbr>. Todas las funciones de un proyecto comparten las mismas configuraciones locales y de hospedaje. En esta sección, se crea un proyecto de función que contiene una sola función.

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

    <br/>
    <details>
    <summary><strong>Para ejecutar funciones en Java 11</strong></summary>

    Use `-DjavaVersion=11` si quiere que las funciones se ejecuten en Java 11. Para obtener más información, consulte las [versiones de Java](functions-reference-java.md#java-versions).
    </details>

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

<br/>
<details>
<summary><strong>¿Qué se crea en la carpeta LocalFunctionProj?</strong></summary>

Esta carpeta contiene varios archivos para el proyecto, como *Function.java*, *FunctionTest.java* y *pom.xml*. También hay archivos de configuración denominados [local.settings.json](functions-run-local.md#local-settings-file) y [host.json](functions-host-json.md). Como *local.settings.json* puede contener secretos descargados de Azure, el archivo se excluye del control de código fuente de forma predeterminada en el archivo *.gitignore*.
</details>

<br/>
<details>
<summary><strong>Código para Function.java</strong></summary>

*Function.java* contiene un método `run` que recibe datos de solicitud en la variable `request` de tipo [HttpRequestMessage](/java/api/com.microsoft.azure.functions.httprequestmessage) que está decorada con la anotación [HttpTrigger](/java/api/com.microsoft.azure.functions.annotation.httptrigger), que define el comportamiento del desencadenador. 

:::code language="java" source="~/azure-functions-samples-java/src/main/java/com/functions/Function.java":::

El mensaje de respuesta lo genera la API [HttpResponseMessage.Builder](/java/api/com.microsoft.azure.functions.httpresponsemessage.builder).

El arquetipo también genera una prueba unitaria para la función. Al cambiar la función para agregar enlaces o agregar nuevas funciones al proyecto, también deberá modificar las pruebas en el archivo *FunctionTest.java*.
</details>

<br/>
<details>
<summary><strong>Código para pom.xml</strong></summary>

La configuración de los recursos de Azure creados para hospedar su aplicación se define en el elemento de **configuración** del complemento con un **groupId** cuyo valor es `com.microsoft.azure` en el archivo *pom.xml* generado. Por ejemplo, el elemento de configuración siguiente indica a una implementación basada en Maven que cree una aplicación de funciones en el grupo de recursos `java-functions-group`, en la región `westus`. <abbr title="Una referencia geográfica a un centro de datos de Azure específico en el que se asignan recursos.">region</abbr>. La propia aplicación de función se ejecuta en Windows hospedada en el plan `java-functions-app-service-plan`, que, de forma predeterminada, es un plan de consumo sin servidor.

:::code language="java" source="~/azure-functions-samples-java/pom.xml" range="62-107":::

Puede cambiar esta configuración para controlar cómo se crean los recursos en Azure, por ejemplo, cambiando `runtime.os` de `windows` a `linux` antes de la implementación inicial. Para obtener una lista completa de los valores de configuración admitidos por el complemento Maven, consulte los [detalles de configuración](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Functions:-Configuration-Details).
</details>

<br>
<hr/>

## <a name="3-run-the-function-locally"></a>3. Ejecución local de la función

1. **Para ejecutar la función**, inicie el host en tiempo de ejecución local de Azure Functions desde la carpeta *LocalFunctionProj*:

    ```console
    mvn clean package
    mvn azure-functions:run
    ```

    Hacia el final de la salida, deberían aparecer las líneas siguientes:

    <pre class="is-monospace is-size-small has-padding-medium has-background-tertiary has-text-tertiary-invert">
    ...

    Now listening on: http://0.0.0.0:7071
    Application started. Press Ctrl+C to shut down.

    Http Functions:

            HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
    ...
    </pre>

    Si HttpExample no aparece como se ha mostrado arriba, es probable que haya iniciado el host desde fuera de la carpeta raíz del proyecto. En ese caso, use <kbd>Ctrl+C</kbd> para detener el host, vaya a la carpeta raíz del proyecto y vuelva a ejecutar el comando anterior.

1. **Copie la URL** de la función `HttpExample` de esta salida en un explorador y anexe la cadena de consulta `?name=<YOUR_NAME>`, de manera que la URL completa sea similar a `http://localhost:7071/api/HttpExample?name=Functions`. El explorador debe mostrar un mensaje como este `Hello Functions`:

    ![Resultado de la ejecución de la función localmente en el explorador](./media/functions-create-first-azure-function-azure-cli/function-test-local-browser.png)

    El terminal en el que inició el proyecto también muestra la salida del registro cuando realiza solicitudes.

1. Cuando termine, presione <kbd>Ctrl+C</kbd> y seleccione <kbd>y</kbd> para detener el host de Functions.

<br>
<hr/>

## <a name="4-deploy-the-function-project-to-azure"></a>4. Implementación del proyecto de función en Azure

La primera vez que implementa un proyecto de funciones, se crea una aplicación de funciones y los recursos relacionados en Azure. La configuración de los recursos de Azure creados para hospedar la aplicación se definen en el archivo *pom.xml*. En este artículo, aceptará los valores predeterminados.

<br/>
<details>
<summary><strong>Para crear una aplicación de funciones que se ejecute en Linux</strong></summary>

Para crear una aplicación de funciones que se ejecute en Linux en lugar de Windows, cambie el elemento `runtime.os` del archivo *pom.xml* de `windows` a `linux`. La ejecución de Linux en un plan de consumo se admite en [estas regiones](https://github.com/Azure/azure-functions-host/wiki/Linux-Consumption-Regions). No puede tener aplicaciones que se ejecuten en Linux y aplicaciones que se ejecuten en Windows en el mismo grupo de recursos.
</details>

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

    La implementación empaqueta los archivos de proyecto y los implementa en la nueva aplicación de función mediante la [implementación de archivos ZIP](functions-deployment-technologies.md#zip-deploy). El código se ejecuta desde el paquete de implementación en Azure.

<br/>
<details>
<summary><strong>¿Qué se crea en Azure?</strong></summary>

+ Un grupo de recursos Se nombra como _java-functions-group_.
+ Una cuenta de almacenamiento Necesaria con Functions. El nombre se genera aleatoriamente según los requisitos de nombre de la cuenta de almacenamiento.
+ Plan de hospedaje. El hospedaje sin servidor de la aplicación de función en la región _westus_. El nombre es _java-functions-app-service-plan_.
+ Aplicación de funciones. Una aplicación de funciones es la unidad de implementación y ejecución de las funciones. El nombre se genera de forma aleatoria según el valor _artifactId_, anexado con un número generado de forma aleatoria.
</details>

<br>
<hr/>

## <a name="5-invoke-the-function-on-azure"></a>5. Invocación de la función en Azure

Como la función usa un desencadenador HTTP, **para invocarla es preciso enviar una solicitud HTTP a la dirección URL correspondiente** en el explorador o mediante una herramienta como CURL. <abbr title="Una herramienta de línea de comandos para generar solicitudes HTTP a una dirección URL. Consulte https://curl.se/">curl</abbr>.

# <a name="browser"></a>[Browser](#tab/browser)

Copie el valor completo de **Invoke URL** completa que se muestra en la salida del comando `publish` en una barra de direcciones del explorador y anexe el parámetro de consulta `&name=Functions`. El explorador debe mostrar una salida similar a cuando ejecutó la función localmente.

![La salida de la función ejecutada en Azure en un explorador](../../includes/media/functions-run-remote-azure-cli/function-test-cloud-browser.png)

# <a name="curl"></a>[curl](#tab/curl)

Ejecute [`curl`](https://curl.haxx.se/) con la **dirección URL de invocación**, y anexe el parámetro `&name=Functions`. La salida del comando será el texto "Hello Functions".

![La salida de la función ejecutada en Azure en un mediante curl](../../includes/media/functions-run-remote-azure-cli/function-test-cloud-curl.png)

---

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

<br>
<hr/>

## <a name="6-clean-up-resources"></a>6. Limpieza de recursos

Si continúa con el [paso siguiente](#next-steps) y agrega un <abbr title="En Azure Storage, una forma de asociar una función a una cola de almacenamiento, de tal manera que pueda crear mensajes en la cola.">enlace de salida de la cola de Azure Storage,</abbr>mantenga todos los recursos en su lugar, ya que se basará en lo que ha hecho hasta ahora.

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

<br>
<hr/>

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-java)
