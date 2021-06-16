---
title: 'Inicio rápido: Creación de la primera función de Azure mediante Visual Studio'
description: En este inicio rápido, aprenderá a crear y publicar una función de Azure de desencadenador HTTP mediante Visual Studio.
ms.assetid: 82db1177-2295-4e39-bd42-763f6082e796
ms.topic: quickstart
ms.date: 09/30/2020
ms.custom: devx-track-csharp, mvc, devcenter, vs-azure, 23113853-34f2-4f
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 5dffc4b1ffaa221e507c84c8a9b5654f9e5355c8
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110785252"
---
# <a name="quickstart-create-your-first-function-in-azure-using-visual-studio"></a>Inicio rápido: Creación de la primera función de Azure mediante Visual Studio

En este artículo se usa Visual Studio para crear una función basada en biblioteca de clases de C# que responde a solicitudes HTTP. Después de probar el código localmente, lo implementará en el entorno <abbr title="Un entorno informático en tiempo de ejecución en el que todos los detalles del servidor son transparentes para los desarrolladores de aplicaciones, lo que simplifica el proceso de implementación y administración de código.">sin servidor</abbr> de <abbr title="Un servicio de Azure que proporciona un entorno informático sin servidor de bajo costo dirigido a las aplicaciones.">Azure Functions</abbr>.

Este inicio rápido supone un pequeño costo en su <abbr title="El perfil que mantiene la información de facturación en relación con el uso de Azure.">Cuenta de Azure</abbr>.

## <a name="1-prepare-your-environment"></a>1. Preparación del entorno

+ Creación de una cuenta de <abbr title="El perfil que mantiene la información de facturación en relación con el uso de Azure.">account</abbr> con una suscripción activa. <abbr title="La estructura organizativa básica en la que administra los recursos de Azure, que normalmente se asocia a un individuo o departamento dentro de una organización.">subscription</abbr>. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ Instale [Visual Studio 2019](https://azure.microsoft.com/downloads/) y seleccione la carga de trabajo de **desarrollo de Azure** durante la instalación. 

<br/>
<details>
<summary><strong>Uso de un proyecto de Azure Functions en su lugar</strong></summary>
Si desea crear en su lugar un proyecto <abbr title="Un contenedor lógico para una o varias funciones individuales que se pueden implementar y administrar juntas.">local de Azure Functions</abbr> con Visual Studio 2017, antes debe instalar las [herramientas de Azure Functions más recientes](functions-develop-vs.md#check-your-tools-version).
</details>

## <a name="2-create-a-function-app-project"></a>2. Creación de un proyecto de aplicación de función

1. En el menú de Visual Studio, seleccione **Archivo** > **Nuevo** > **Proyecto**.

1. En **Crear un proyecto**, escriba *functions* en el cuadro de búsqueda, elija la plantilla **Azure Functions** y seleccione **Siguiente**.

1. En **Configurar el nuevo proyecto**, escriba un **<abbr title="El nombre de la aplicación de funciones debe ser válido como espacio de nombres de C#, por lo que no debe contener caracteres de subrayado, guiones ni ningún otro carácter no alfanumérico. "> </abbr>nombre** para el proyecto y, después, seleccione **Crear**. 

1. Incluya la siguiente información en la configuración de **Crear una aplicación de Azure Functions**:

    + Seleccione **<abbr title=" Este valor crea un proyecto de funciones que usa la versión 3.x del runtime de Azure Functions, que es compatible con .NET Core 3.x. Azure Functions 1.x es compatible con .NET Framework.">Azure Functions v3 (.NET Core)</abbr>** en la lista desplegable Runtime de Functions (para más información, consulte [Introducción a las versiones de tiempo de ejecución de Azure Functions](functions-versions.md)).
    
    + Seleccione **<abbr title="Este valor crea una función desencadenada por una solicitud HTTP.">Desencadenador HTTP</abbr>** como plantilla de función.
    
    + Seleccione **<abbr title="Dado que cada función de Azure necesita una cuenta de almacenamiento, se asigna o se crea una al publicar el proyecto en Azure. Los desencadenadores HTTP no utilizan una cadena de conexión de cuenta de Azure Storage; los restantes tipos de desencadenador requieren una cadena de conexión de cuenta de Azure Storage válida. ">Emulador de Storage</abbr>** en la lista desplegable Cuenta de almacenamiento.
        
    + Seleccione **Anónimo** en la <abbr title="Cualquier cliente puede desencadenar una función creada sin tener que proporcionar una clave. Esta configuración de autorización facilita probar la función nueva.">lista desplegable</abbr> Nivel de autorización (para más información sobre claves y autorización, consulte los artículos sobre [claves de autorización](functions-bindings-http-webhook-trigger.md#authorization-keys) y [enlaces de webhook y HTTP](functions-bindings-http-webhook.md)).

    + Seleccione **Crear**
        
## <a name="3-rename-the-function"></a>3. Cambio del nombre de la función

El atributo del método `FunctionName` establece el nombre de la función que, de forma predeterminada, se genera como `Function1`. Dado que las herramientas no permiten reemplazar el nombre de función predeterminado al crear un proyecto, dedique un minuto a crear un nombre mejor para la clase, el archivo y los metadatos de la función.

1. En el **Explorador de archivos**, haga clic con el botón derecho en el archivo Function1.cs y cambie el nombre por *HttpExample.cs*.

1. En el código, cambie el nombre de la clase Function1 a "HttpExample".

1. En el método `HttpTrigger` denominado `Run`, cambie el nombre del atributo del método `FunctionName` a `HttpExample`.


## <a name="4-run-the-function-locally"></a>4. Ejecución local de la función

1. Para ejecutar la función, presione <kbd>F5</kbd> en Visual Studio.

1. Copie la dirección URL de la función de los resultados del runtime de Azure Functions.

    ![Runtime local de Azure](../../includes/media/functions-run-function-test-local-vs/functions-debug-local-vs.png)

1. Pegue la dirección URL de la solicitud HTTP en la barra de direcciones del explorador. Anexe la cadena de consulta **?name=<YOUR_NAME>** a esta dirección URL y ejecute la solicitud. 

    ![Respuesta de localhost de la función en el explorador](../../includes/media/functions-run-function-test-local-vs/functions-run-browser-local-vs.png)

1. Para detener la depuración, presione <kbd>Mayús</kbd>+<kbd>F5</kbd> en Visual Studio.

<br/>
<details>
<summary><strong>Solución de problemas</strong></summary>
 Es preciso habilitar una excepción de firewall para que las herramientas para controlen las solicitudes de HTTP. Los niveles de autorización nunca se aplican cuando se ejecuta una función localmente.
</details>

## <a name="5-publish-the-project-to-azure"></a>5. Publicar el proyecto en Azure

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Publicar**.

1. En **Destino**, seleccione **Azure**

    :::image type="content" source="../../includes/media/functions-vstools-publish/functions-visual-studio-publish-profile-step-1.png" alt-text="Seleccionar destino de Azure":::

1. En **Destino específico**, seleccione **Azure Function App (Windows)** .

    :::image type="content" source="../../includes/media/functions-vstools-publish/functions-visual-studio-publish-profile-step-2.png" alt-text="Seleccionar Azure Function App":::

1. En **Instancia de Functions**, seleccione **Crear una instancia de Azure Functions...** y, después, use los valores especificados en la tabla siguiente:

    + En **Nombre**, especifique un <abbr title="Use un nombre que identifique de forma única la nueva aplicación de funciones. Acepte este nombre o escriba uno nuevo. Los caracteres válidos son `a-z`, `0-9` y `-`.">Nombre único globalmente</abbr>
    
    + **Seleccione** una suscripción en la lista desplegable.
    
    + **Seleccione** un grupo de recursos <abbr title="Un contenedor lógico para recursos relacionados de Azure que se pueden administrar como una unidad.">existente</abbr> en la lista desplegable, o bien elija **Nuevo** para crearlo.
    
    + **Select** <abbr title="Cuando publique el proyecto en una aplicación de funciones que se ejecute en un plan Consumo, solo pagará por las ejecuciones de la aplicación. Otros planes de hospedaje suponen costos más elevados.">Consumo</abbr> en la lista desplegable de tipo de reproducción (para más información, consulte [Plan de consumo](consumption-plan.md)).
    
    + **Seleccione** una  <abbr title="Una referencia geográfica a un centro de datos específico de Azure en el que se asignan los recursos. En [regiones](https://azure.microsoft.com/regions/) encontrará una lista de las regiones disponibles.">ubicación</abbr> en la lista desplegable.
    
    + **Seleccione** un <abbr="El runtime de Functions requiere una cuenta de Azure Storage. Seleccione Nueva para configurar una cuenta de almacenamiento de uso general. También puede elegir una cuenta existente que cumpla los requisitos de la cuenta de almacenamiento.">Azure Storage</abbr> cuenta en la lista desplegable

    ![Cuadro de diálogo Crear servicio de aplicaciones](../../includes/media/functions-vstools-publish/functions-visual-studio-publish.png)

1. Seleccione **Crear** 

1. En la **instancia de Functions**, asegúrese de que **Ejecutar desde archivo de paquete** está activada. 

    :::image type="content" source="../../includes/media/functions-vstools-publish/functions-visual-studio-publish-profile-step-4.png" alt-text="Finalizar la creación del perfil":::

    <br/>
    <details>
    <summary><strong>¿Qué hace este valor?</strong></summary>
    Si se utiliza **Run from package file** (Ejecutar desde archivo de paquetes), la aplicación de funciones se implementa mediante [Implementación de archivo ZIP](functions-deployment-technologies.md#zip-deploy) con el modo de [ejecución desde paquete](run-functions-from-deployment-package.md) habilitado. Este es el método de implementación recomendado para el proyecto de Functions, ya que se obtiene un mejor rendimiento.    
    </details>   

1. Seleccione **Finalizar**.

1. En la página Publicar, seleccione **Publicar**.

1. En la página Publicar, consulte la dirección URL raíz de la aplicación de funciones.

1. En la pestaña Publicar, elija **Administrar en<abbr title=" Cloud Explorer para poder usar Visual Studio para ver el contenido del sitio, iniciar y detener la aplicación de funciones, y desplazarse directamente a los recursos de la aplicación de funciones en Azure y en Azure Portal. ">Cloud Explorer</abbr>** .
    
    :::image type="content" source="../../includes/media/functions-vstools-publish/functions-visual-studio-publish-complete.png" alt-text="Publicación de un mensaje de operación correcta":::
    

## <a name="6-test-your-function-in-azure"></a>6. Prueba de una función en Azure

1. En Cloud Explorer, debería estar seleccionada la nueva aplicación de funciones. Si no es así, expanda su suscripción, expanda **App Services** y seleccione la nueva aplicación de funciones.

1. Haga clic con el botón derecho en la aplicación de funciones y elija **Abrir en explorador**. Se abre la raíz de su aplicación de funciones en el explorador web predeterminado y muestra la página que indica que su aplicación de funciones se está ejecutando. 

    :::image type="content" source="media/functions-create-your-first-function-visual-studio/function-app-running-azure.png" alt-text="Aplicación de funciones en ejecución":::

1. En la barra de direcciones del explorador, anexe la cadena **/api/HttpExample?name=Functions** a la URL base y ejecute la solicitud.

    La dirección URL que llama a la función de desencadenador HTTP tiene el formato siguiente:

    `http://<APP_NAME>.azurewebsites.net/api/HttpExample?name=Functions`

2. Vaya a esta dirección URL y verá una respuesta en el explorador para la solicitud GET remota devuelta por la función, que es similar a la del siguiente ejemplo:

    :::image type="content" source="media/functions-create-your-first-function-visual-studio/functions-create-your-first-function-visual-studio-browser-azure.png" alt-text="Respuesta de la función en el explorador":::

## <a name="7-clean-up-resources"></a>7. Limpieza de recursos

[!INCLUDE [functions-vstools-cleanup](../../includes/functions-vstools-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo, donde aprenderá a agregar un <abbr title="Una forma de asociar una función a una cola de almacenamiento, de tal manera que pueda crear mensajes en la cola. Los enlaces son conexiones declarativas entre una función y otros recursos. Un enlace de entrada proporciona datos a la función, mientras que un enlace de salida proporciona datos de la función a otros recursos.">enlace de salida de la cola de Azure Storage</abbr> con su función:

> [!div class="nextstepaction"]
> [Adición de un enlace de cola de Azure Storage a una función](functions-add-output-binding-storage-queue-vs.md)
