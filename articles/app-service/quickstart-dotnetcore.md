---
title: 'Inicio rápido: Implementación de una aplicación web de ASP.NET'
description: Aprenda a ejecutar aplicaciones web en Azure App Service mediante la implementación de la primera aplicación web de ASP.NET.
ms.assetid: b1e6bd58-48d1-4007-9d6c-53fd6db061e3
ms.topic: quickstart
ms.date: 06/08/2021
ms.custom: devx-track-csharp, mvc, devcenter, vs-azure, seodec18, contperf-fy21q1
zone_pivot_groups: app-service-ide
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./quickstart-dotnetcore-uiex
ms.openlocfilehash: 3b3abdf40d5aa9d56421361237432ddf08c0c016
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111746544"
---
<!-- NOTES:

I'm a .NET developer who wants to deploy my web app to App Service. I may develop apps with
Visual Studio, Visual Studio for Mac, Visual Studio Code, or the .NET SDK/CLI. This article
should be able to guide .NET devs, whether they're app is .NET Core, .NET, or .NET Framework.

As a .NET developer, when choosing an IDE and .NET TFM - you map to various OS requirements.
For example, if you choose Visual Studio - you're developing the app on Windows, but you can still
target cross-platform with .NET Core 3.1 or .NET 5.0.

| .NET / IDE         | Visual Studio | Visual Studio for Mac | Visual Studio Code | Command line   |
|--------------------|---------------|-----------------------|--------------------|----------------|
| .NET Core 3.1      | Windows       | macOS                 | Cross-platform     | Cross-platform |
| .NET 5.0           | Windows       | macOS                 | Cross-platform     | Cross-platform |
| .NET Framework 4.8 | Windows       | N/A                   | Windows            | Windows        |

-->

# <a name="quickstart-deploy-an-aspnet-web-app"></a>Inicio rápido: Implementación de una aplicación web de ASP.NET

En este inicio rápido, aprenderá a crear e implementar su primera aplicación web de ASP.NET en [Azure App Service](overview.md). App Service admite varias versiones de aplicaciones .NET y proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones. Las aplicaciones web de ASP.NET son multiplataforma y se pueden hospedar tanto en Linux como en Windows. Cuando termine, tendrá un grupo de recursos de Azure que consta de un plan de hospedaje de App Service y una aplicación web implementada.

## <a name="prerequisites"></a>Prerrequisitos

:::zone target="docs" pivot="development-environment-vs"

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/dotnet).
- <a href="https://www.visualstudio.com/downloads" target="_blank">Visual Studio 2019</a> con la carga de trabajo **ASP.NET y desarrollo web**.

    Si ya ha instalado Visual Studio 2019:

    - Para instalar las actualizaciones más recientes de Visual Studio, seleccione **Ayuda** > **Buscar actualizaciones**.
    - Para agregar la carga de trabajo, seleccione **Herramientas** > **Obtener herramientas y características**.

:::zone-end

:::zone target="docs" pivot="development-environment-vscode"

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/dotnet).
- <a href="https://www.visualstudio.com/downloads" target="_blank">Visual Studio Code</a>.
- La extensión <a href="https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack" target="_blank">Azure Tools</a>.

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

<a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Instale el SDK de .NET Core 3.1 más reciente</a>.

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

<a href="https://dotnet.microsoft.com/download/dotnet/5.0" target="_blank">Instale el SDK de .NET 5.0 más reciente</a>.

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

<a href="https://dotnet.microsoft.com/download/dotnet-framework/net48" target="_blank">Instale el paquete para desarrolladores de .NET Framework 4.8</a>.

> [!NOTE]
> Visual Studio Code es multiplataforma, pero .NET Framework no lo es. Si va a desarrollar aplicaciones de .NET Framework con Visual Studio Code, considere la posibilidad de usar una máquina Windows para cubrir las dependencias de compilación.

---

:::zone-end

<!-- markdownlint-disable MD044 -->
:::zone target="docs" pivot="development-environment-cli"
<!-- markdownlint-enable MD044 -->

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/dotnet).
- La<a href="/cli/azure/install-azure-cli" target="_blank">CLI de Azure</a>.
- El SDK de .NET SDK (incluye el runtime y la CLI).

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

<a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Instale el SDK de .NET Core 3.1 más reciente</a>.

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

<a href="https://dotnet.microsoft.com/download/dotnet/5.0" target="_blank">Instale el SDK de .NET 5.0 más reciente</a>.

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

<a href="https://dotnet.microsoft.com/download/dotnet/5.0" target="_blank">Instale el SDK de .NET 5.0 más reciente </a> y <a href="https://dotnet.microsoft.com/download/dotnet-framework/net48" target="_blank"> el paquete para desarrolladores de .NET Framework 4.8</a>.

> [!NOTE]
> La [CLI de .NET](/dotnet/core/tools) es multiplataforma, pero .NET Framework no lo es. Si va a desarrollar aplicaciones de .NET Framework con la CLI de .NET, considere la posibilidad de usar una máquina Windows para cubrir las dependencias de compilación.

---

:::zone-end

## <a name="create-an-aspnet-web-app"></a>Creación de una aplicación web de ASP.NET

> [!TIP]
> .NET Core 3.1 es la versión actual del soporte técnico a largo plazo (LTS) de .NET. Para más información, consulte la [directiva de soporte técnico de .NET](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).

:::zone target="docs" pivot="development-environment-vs"

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

1. Abra Visual Studio y seleccione **Crear un proyecto**.
1. En **Crear un proyecto**, busque y elija **Aplicación web ASP.NET Core** y, después, seleccione **Siguiente**.
1. En **Configurar el nuevo proyecto**, asigne a la aplicación el nombre _MyFirstAzureWebApp_ y seleccione **Siguiente**.

   :::image type="content" source="media/quickstart-dotnet/configure-webapp-net.png" alt-text="Configurar una aplicación de ASP.NET Core 3.1" border="true":::

1. Seleccione **.NET Core 3.1 (soporte técnico a largo plazo)** .
1. Asegúrese de que en **Tipo de autenticación** está seleccionada la opción **Ninguno**. Seleccione **Crear**.

   :::image type="content" source="media/quickstart-dotnet/vs-additional-info-netcoreapp31.png" alt-text="Visual Studio: seleccione .NET Core 3.1 y Ninguno en Tipo de autenticación." border="true":::

1. En el menú de Visual Studio, seleccione **Depurar** > **Iniciar sin depurar** para ejecutar la aplicación web localmente.

   :::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio: exploración local de .NET Core 3.1" lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

1. Abra Visual Studio y seleccione **Crear un proyecto**.
1. En **Crear un proyecto**, busque y elija **Aplicación web ASP.NET Core** y, después, seleccione **Siguiente**.
1. En **Configurar el nuevo proyecto**, asigne a la aplicación el nombre _MyFirstAzureWebApp_ y seleccione **Siguiente**.

   :::image type="content" source="media/quickstart-dotnet/configure-webapp-net.png" alt-text="Visual Studio: configuración de una aplicación web de ASP.NET 5.0." border="true":::

1. Seleccione **.NET Core 5.0 (actual)** .
1. Asegúrese de que en **Tipo de autenticación** está seleccionada la opción **Ninguno**. Seleccione **Crear**.

   :::image type="content" source="media/quickstart-dotnet/vs-additional-info-net50.png" alt-text="Visual Studio: información adicional al seleccionar .NET Core 5.0." border="true":::

1. En el menú de Visual Studio, seleccione **Depurar** > **Iniciar sin depurar** para ejecutar la aplicación web localmente.

   :::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio: ASP.NET Core 5.0 se ejecuta localmente." lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

1. Abra Visual Studio y seleccione **Crear un proyecto**.
1. En **Crear un proyecto**, busque y elija **Aplicación web de ASP.NET (.NET Framework)** y, después, seleccione **Siguiente**.
1. En **Configurar el nuevo proyecto**, asigne a la aplicación el nombre _MyFirstAzureWebApp_ y **Crear**.

   :::image type="content" source="media/quickstart-dotnet/configure-webapp-netframework48.png" alt-text="Visual Studio: configuración de una aplicación web de ASP.NET Framework 4.8." border="true":::

1. Seleccione la plantilla **MVC** .
1. Asegúrese de que **Autenticación** esté establecida en **Sin autenticación**. Seleccione **Crear**.

   :::image type="content" source="media/quickstart-dotnet/vs-mvc-no-auth-netframework48.png" alt-text="Visual Studio: selección de la plantilla de MVC." border="true":::

1. En el menú de Visual Studio, seleccione **Depurar** > **Iniciar sin depurar** para ejecutar la aplicación web localmente.

   :::image type="content" source="media/quickstart-dotnet/vs-local-webapp-netframework48.png" alt-text="Visual Studio: ASP.NET Framework 4.8 se ejecuta localmente." lightbox="media/quickstart-dotnet/vs-local-webapp-netframework48.png" border="true":::

---

:::zone-end

:::zone target="docs" pivot="development-environment-vscode"

Cree una carpeta denominada _MyFirstAzureWebApp_ y ábrala en Visual Studio Code. Abra la ventana <a href="https://code.visualstudio.com/docs/editor/integrated-terminal" target="_blank">Terminal</a> y cree una aplicación web de .NET con el comando [`dotnet new webapp`](/dotnet/core/tools/dotnet-new#web-options) .

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

```dotnetcli
dotnet new webapp -f netcoreapp3.1
```

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

```dotnetcli
dotnet new webapp -f net5.0
```

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

```dotnetcli
dotnet new webapp --target-framework-override net48
```

> [!IMPORTANT]
> La marca `--target-framework-override` es un reemplazo del texto de forma libre del moniker de la plataforma de destino (TFM) del proyecto y no ofrece *ninguna garantía* de que la plantilla de apoyo exista o se compile. Las aplicaciones de .NET Framework solo se pueden compilar y ejecutar en Windows.

---

En Visual Studio Code, desde el **Terminal** ejecute la aplicación localmente mediante el comando [`dotnet run`](/dotnet/core/tools/dotnet-run).

```dotnetcli
dotnet run
```

Abra un explorador web y vaya a la aplicación en `https://localhost:5001`.


### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

Verá que la aplicación web de ASP.NET Core 3.1 de la plantilla se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio Code: ejecute .NET Core 3.1 en un explorador localmente." lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

Verá que la aplicación web de ASP.NET Core 5.0 de la plantilla se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio Code: ejecute .NET 5.0 en un explorador localmente." lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

Verá que la aplicación web de ASP.NET Framework 4.8 de la plantilla se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net48.png" alt-text="Visual Studio Code: ejecute .NET 4.8 en un explorador local." lightbox="media/quickstart-dotnet/local-webapp-net48.png" border="true":::

---

:::zone-end

<!-- markdownlint-disable MD044 -->
:::zone target="docs" pivot="development-environment-cli"
<!-- markdownlint-enable MD044 -->

Abra una ventana de terminal en la máquina en un directorio de trabajo. Cree una aplicación web de .NET con el comando [`dotnet new webapp`](/dotnet/core/tools/dotnet-new#web-options) y vaya a los directorios de la aplicación recién creada.

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

```dotnetcli
dotnet new webapp -n MyFirstAzureWebApp -f netcoreapp3.1 && cd MyFirstAzureWebApp
```

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

```dotnetcli
dotnet new webapp -n MyFirstAzureWebApp -f net5.0 && cd MyFirstAzureWebApp
```

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

```dotnetcli
dotnet new webapp -n MyFirstAzureWebApp --target-framework-override net48 && cd MyFirstAzureWebApp
```

> [!IMPORTANT]
> La marca `--target-framework-override` es un reemplazo del texto de forma libre del moniker de la plataforma de destino (TFM) del proyecto y no ofrece *ninguna garantía* de que la plantilla de apoyo exista o se compile. Las aplicaciones de .NET Framework solo se pueden compilar en Windows.

---

En la misma sesión de terminal, ejecute la aplicación localmente mediante el comando [`dotnet run`](/dotnet/core/tools/dotnet-run).

```dotnetcli
dotnet run
```

Abra un explorador web y vaya a la aplicación en `https://localhost:5001`.

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

Verá que la aplicación web de ASP.NET Core 3.1 de la plantilla se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio Code: ASP.NET Core 3.1 en un explorador local." lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

Verá que la aplicación web de ASP.NET Core 5.0 de la plantilla se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio Code: ASP.NET Core 5.0 en un explorador local." lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

Verá que la aplicación web de ASP.NET Framework 4.8 de la plantilla se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net48.png" alt-text="Visual Studio Code: ASP.NET Framework 4.8 en un explorador local." lightbox="media/quickstart-dotnet/local-webapp-net48.png" border="true":::

---

:::zone-end

## <a name="publish-your-web-app"></a>Publicación de la aplicación web

Para publicar la aplicación web, primero debe crear y configurar una nueva instancia de App Service en la que pueda publicar la aplicación.

Como parte de la configuración de la instancia de App Service, creará:

- Un nuevo [grupo de recursos](../azure-resource-manager/management/overview.md#terminology) que contendrá todos los recursos de Azure para el servicio.
- Un [plan de hospedaje](overview-hosting-plans.md) que especifique la ubicación, el tamaño y las características de la granja de servidores web que hospeda la aplicación.

Siga estos pasos para crear la instancia de App Service y publicar la aplicación web:

:::zone target="docs" pivot="development-environment-vs"

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **MyFirstAzureWebApp** y seleccione **Publicar**.
1. En **Publicar**, seleccione **Azure** y, después, **Siguiente**.

    :::image type="content" source="media/quickstart-dotnet/vs-publish-target-Azure.png" alt-text="Visual Studio: publique la aplicación web y use Azure como destino." border="true":::

1. Las opciones dependen de si ya ha iniciado sesión en Azure y de si tiene una cuenta de Visual Studio vinculada a una cuenta de Azure. Seleccione **Agregar una cuenta** o **Iniciar sesión** para iniciar sesión en la suscripción de Azure. Si ya ha iniciado sesión, seleccione la cuenta que desee.

    :::image type="content" source="media/quickstart-dotnetcore/sign-in-Azure-vs2019.png" border="true" alt-text="Visual Studio: seleccione iniciar sesión en el cuadro de diálogo de Azure.":::

1. Elija el valor de **Destino específico**, sea **Azure App Service (Linux)** o **Azure App Service (Windows)** .

    > [!IMPORTANT]
    > Cuando el destino sea ASP.NET Framework 4.8, usará **Azure App Service (Windows)** .

1. A la derecha de **App Service instances** (Instancias de App Service), seleccione **+** .

    :::image type="content" source="media/quickstart-dotnetcore/publish-new-app-service.png" border="true" alt-text="Visual Studio: cuadro de diálogo de la aplicación del nuevo App Service.":::

1. En **Suscripción**, acepte la suscripción que aparece o seleccione otra en la lista desplegable.
1. En **Grupo de recursos**, seleccione **Nuevo**. En **Nuevo nombre de grupo de recursos**, escriba *myResourceGroup* y seleccione **Aceptar**.
1. En **Plan de hospedaje**, seleccione **Nuevo**.
1. En el cuadro de diálogo **Plan de hospedaje: Crear nuevo**, escriba los valores especificados en la tabla siguiente:

    | Configuración          | Valor sugerido          | Descripción                                                           |
    |------------------|--------------------------|-----------------------------------------------------------------------|
    | **Plan de hospedaje** | *MyFirstAzureWebAppPlan* | Nombre del plan de App Service.                                         |
    | **Ubicación**     | *Oeste de Europa*            | El centro de datos donde se hospeda la aplicación web.                           |
    | **Tamaño**         | *Gratis*                   | [Plan de tarifa][app-service-pricing-tier] determina las características de hospedaje. |

    :::image type="content" source="media/quickstart-dotnetcore/create-new-hosting-plan-vs2019.png" border="true" alt-text="Creación de un nuevo plan de hospedaje":::

1. En **Nombre**, escriba un nombre de aplicación único que incluya solo los caracteres válidos, que son `a-z`, `A-Z`, `0-9` y `-`. Puede aceptar el nombre único generado automáticamente. La dirección URL de la aplicación web es `http://<app-name>.azurewebsites.net`, donde `<app-name>` es el nombre de la aplicación.
1. Seleccione **Crear** para crear los recursos de Azure.

    :::image type="content" source="media/quickstart-dotnetcore/web-app-name-vs2019.png" border="true" alt-text="Visual Studio: cuadro de diálogo de creación de recursos de la aplicación.":::

   Una vez finalizado el asistente, los recursos de Azure se crean automáticamente y ya puede publicarlos.

1. Seleccione **Finalizar** para cerrar el asistente.
1. En la página **Publicar**, seleccione **Publicar**. Visual Studio compila, empaqueta y publica la aplicación en Azure y, luego, la inicia en el explorador predeterminado.

    ### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

    Verá que la aplicación web de ASP.NET Core 3.1 se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="Visual Studio: aplicación web de ASP.NET Core 3.1 en Azure.":::

    ### <a name="net-50"></a>[.NET 5.0](#tab/net50)

    Verá que la aplicación web de ASP.NET Core 5.0 se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="Visual Studio: aplicación web de ASP.NET Core 5.0 en Azure.":::

    ### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

    Verá que la aplicación web de ASP.NET Framework 4.8 se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/vs-Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/vs-Azure-webapp-net48.png" border="true" alt-text="Visual Studio: una aplicación web de ASP.NET Framework 4.8 en Azure.":::

    ---

:::zone-end

:::zone target="docs" pivot="development-environment-vscode"

Para implementar una aplicación web con la extensión Azure Tools para Visual Studio:

1. En Visual Studio Code, abra la [**paleta de comandos**](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette), <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd>.
1. Busque y seleccione "Azure App Service: Implementar en aplicación web".
1. Responda a los mensajes como se indica a continuación:

    - Seleccione *MyFirstAzureWebApp* como la carpeta que se va a implementar.
    - Seleccione **Agregar configuración** cuando se le solicite.
    - Si se le pide, inicie sesión en su cuenta de Azure.

    :::image type="content" source="media/quickstart-dotnet/vscode-sign-in-to-Azure.png" alt-text="Visual Studio Code: inicio de sesión en Azure" border="true":::.

    - Seleccione su **suscripción**.
    - Seleccione **Create new Web App...Advanced** (Crear aplicación web...Avanzada).
    - En **Enter a globally unique name**, (Especificar un nombre único global) use un nombre que sea único en todo Azure (los *caracteres válidos son `a-z`, `0-9` y `-`* ). Un buen patrón es usar una combinación del nombre de la empresa y un identificador de la aplicación.
    - Seleccione **Crear nuevo grupo de recursos** y escriba un nombre, por ejemplo, `myResourceGroup`.
    - Cuando se le pida, **seleccione una pila en tiempo de ejecución**:
      - En el caso de *.NET Core 3.1*, seleccione **.NET Core 3.1 (LTS)**
      - En el caso de *.NET 5.0*, seleccione **.NET 5**
      - En el caso de *.NET Framework 4.8*, seleccione **ASP.NET v4.8**
    - Seleccione un sistema operativo (Windows o Linux).
        - En el caso de *.NET Framework 4.8*, Windows se seleccionará implícitamente.
    - Seleccione **Crear un nuevo plan de App Service**, especifique un nombre y seleccione el **plan de tarifa** [F1 Gratis][app-service-pricing-tier].
    - Seleccione **Omitir por ahora** para el recurso de Application Insights.
    - Seleccione una ubicación cerca de usted.

1. Cuando se complete la publicación, seleccione **Examinar sitio web** en la notificación y seleccione **Abrir** cuando se le solicite.

    ### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

    Verá que la aplicación web de ASP.NET Core 3.1 se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="Visual Studio Code: aplicación web de ASP.NET Core 3.1 en Azure.":::

    ### <a name="net-50"></a>[.NET 5.0](#tab/net50)

    Verá que la aplicación web de ASP.NET Core 5.0 se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="Visual Studio Code: aplicación web de ASP.NET Core 5.0 en Azure.":::

    ### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

    Verá que la aplicación web de ASP.NET Framework 4.8 se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/vs-Azure-webapp-net48.png" border="true" alt-text="Visual Studio Code: aplicación web de ASP.NET Framework 4.8 en Azure.":::

    ---

:::zone-end

<!-- markdownlint-disable MD044 -->
:::zone target="docs" pivot="development-environment-cli"
<!-- markdownlint-enable MD044 -->

Implemente el código en el directorio *MyFirstAzureWebApp* local, para lo que debe usar el comando [`az webapp up`](/cli/azure/webapp#az_webapp_up):

```azurecli
az webapp up --sku F1 --name <app-name> --os-type <os>
```

- Si no se reconoce el comando `az`, asegúrese de tener instalada la CLI de Azure, como se describe en los [requisitos previos](#prerequisites).
- Reemplace `<app-name>` por un nombre que sea único en todo Azure (*los caracteres válidos son `a-z`, `0-9` y `-`* ). Un buen patrón es usar una combinación del nombre de la empresa y un identificador de la aplicación.
- El argumento `--sku F1` crea la aplicación web en el **plan de tarifa** [Gratis][app-service-pricing-tier]. Omita este argumento para usar un nivel Premium más rápido, lo que supondrá un costo por hora.
- Reemplace `<os>` por `linux` o `windows`. Debe usar `windows` cuando el destino sea *ASP.NET Framework 4.8*.
- Opcionalmente, incluya el argumento `--location <location-name>`, donde `<location-name>` es una región de Azure disponible. Puede recuperar una lista de las regiones permitidas para su cuenta de Azure mediante la ejecución del comando [`az account list-locations`](/cli/azure/appservice#az_appservice_list_locations).

El comando puede tardar varios minutos en completarse. Mientras se ejecuta, proporciona mensajes sobre cómo crear el grupo de recursos, el plan de App Service y la aplicación de hospedaje, configurar el registro y, a continuación, realizar la implementación del archivo ZIP. A continuación, genera un mensaje con la dirección URL de la aplicación:

```azurecli
You can launch the app at http://<app-name>.azurewebsites.net
```

Abra un explorador web y vaya a dicha dirección:

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

Verá que la aplicación web de ASP.NET Core 3.1 se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="CLI: aplicación web de ASP.NET Core 3.1 en Azure.":::

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

Verá que la aplicación web de ASP.NET Core 5.0 se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="CLI: aplicación web de ASP.NET Core 5.0 en Azure.":::

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

Verá que la aplicación web de ASP.NET Framework 4.8 se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/Azure-webapp-net48.png" border="true" alt-text="CLI: aplicación web de ASP.NET Core 4.8 en Azure.":::

---

:::zone-end

## <a name="update-the-app-and-redeploy"></a>Actualización de la aplicación y nueva implementación

Siga estos pasos para actualizar y volver a implementar la aplicación web:

:::zone target="docs" pivot="development-environment-vs"

1. En el **Explorador de soluciones**, en el proyecto, abra *index.cshtml*.
1. Reemplace el primer elemento `<div>` por el código siguiente:

    ```razor
    <div class="jumbotron">
        <h1>.NET 💜 Azure</h1>
        <p class="lead">Example .NET app to Azure App Service.</p>
    </div>
    ```

   Guarde los cambios.

1. Para volver a realizar la implementación en Azure, haga clic con el botón derecho en el proyecto **MyFirstAzureWebApp** en el **Explorador de soluciones** y seleccione **Publicar**.
1. En la página de resumen **Publicar**, seleccione **Publicar**.

    Cuando se completa la publicación, Visual Studio inicia un explorador en la dirección URL de la aplicación web.

    ### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

    Verá que la aplicación web de ASP.NET Core 3.1 actualizada se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="Visual Studio: aplicación web de ASP.NET Core 3.1 actualizada en Azure.":::

    ### <a name="net-50"></a>[.NET 5.0](#tab/net50)

    Verá que la aplicación web de ASP.NET Core 5.0 actualizada se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="Visual Studio: aplicación web de ASP.NET Core 5.0 actualizada en Azure.":::

    ### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

    Verá que la aplicación web de ASP.NET Framework 4.8 actualizada se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/vs-updated-Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/vs-updated-Azure-webapp-net48.png" border="true" alt-text="Visual Studio: aplicación web de ASP.NET Framework 4.8 actualizada en Azure.":::

    ---

:::zone-end

:::zone target="docs" pivot="development-environment-vscode"

1. Abra *index.cshtml*.
1. Reemplace el primer elemento `<div>` por el código siguiente:

    ```razor
    <div class="jumbotron">
        <h1>.NET 💜 Azure</h1>
        <p class="lead">Example .NET app to Azure App Service.</p>
    </div>
    ```

   Guarde los cambios.

1. Abra la **barra lateral** de Visual Studio Code y seleccione el icono de **Azure** para expandir sus opciones.
1. En el nodo **APP SERVICE**, expanda su suscripción y haga clic con el botón derecho en **MyFirstAzureWebApp**.
1. Seleccione **Implementar en aplicación web...**
1. Cuando se le solicite, seleccione **Implementar**.
1. Cuando se complete la publicación, seleccione **Examinar sitio web** en la notificación y seleccione **Abrir** cuando se le solicite.

    ### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

    Verá que la aplicación web de ASP.NET Core 3.1 actualizada se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="Visual Studio Code: aplicación web de ASP.NET Core 3.1 actualizada en Azure.":::

    ### <a name="net-50"></a>[.NET 5.0](#tab/net50)

    Verá que la aplicación web de ASP.NET Core 5.0 actualizada se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="Visual Studio Code: aplicación web de ASP.NET Core 5.0 actualizada en Azure.":::

    ### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

    Verá que la aplicación web de ASP.NET Framework 4.8 actualizada se muestra en la página.

    :::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net48.png" border="true" alt-text="Visual Studio Code: aplicación web de ASP.NET Framework 4.8 actualizada en Azure.":::

    ---

:::zone-end

<!-- markdownlint-disable MD044 -->
:::zone target="docs" pivot="development-environment-cli"
<!-- markdownlint-enable MD044 -->

En el directorio local, abra el archivo *Index.cshtml*. Reemplace el primer elemento `<div>`:

```razor
<div class="jumbotron">
    <h1>.NET 💜 Azure</h1>
    <p class="lead">Example .NET app to Azure App Service.</p>
</div>
```

Guarde los cambios y vuelva a implementar la aplicación con el comando `az webapp up`:

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

ASP.NET Core 3.1 es multiplataforma, en función de la implementación anterior, reemplace `<os>` por `linux` o `windows`.

```azurecli
az webapp up --os-type <os>
```

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

ASP.NET Core 5.0 es multiplataforma, en función de la implementación anterior, reemplace `<os>` por `linux` o `windows`.

```azurecli
az webapp up --os-type <os>
```

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

ASP.NET Framework 4.8 tiene dependencias del marco y se debe hospedar en Windows.

```azurecli
az webapp up --os-type windows
```

> [!TIP]
> Si le interesa hospedar aplicaciones .NET en Linux, considere la posibilidad de migrar de [ASP.NET Framework a ASP.NET Core](/aspnet/core/migration/proper-to-2x).

---

Este comando utiliza valores que se almacenan en caché de forma local en el archivo *.azure/config*, incluidos el nombre de la aplicación, el grupo de recursos y el plan de App Service.

Una vez que la implementación haya finalizado, vuelva a cambiar la ventana del explorador que se abrió en el paso **Navegación hasta la aplicación** y actualice la vista.

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

Verá que la aplicación web de ASP.NET Core 3.1 actualizada se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="CLI: aplicación web de ASP.NET Core 3.1 actualizada en Azure.":::

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

Verá que la aplicación web de ASP.NET Core 5.0 actualizada se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="CLI: aplicación web de ASP.NET Core 5.0 actualizada en Azure.":::

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

Verá que la aplicación web de ASP.NET Framework 4.8 actualizada se muestra en la página.

:::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net48.png" border="true" alt-text="CLI: aplicación web de ASP.NET Framework 4.8 actualizada en Azure.":::

---

:::zone-end

## <a name="manage-the-azure-app"></a>Administración de la aplicación de Azure

Para administrar la aplicación web, vaya a [Azure Portal](https://portal.azure.com) y busque y seleccione **App Services**.

:::image type="content" source="media/quickstart-dotnetcore/app-services.png" alt-text="Azure Portal: seleccione la opción App Services.":::

En la página **App Services**, seleccione el nombre de la aplicación web.

:::image type="content" source="./media/quickstart-dotnetcore/select-app-service.png" alt-text="Azure Portal: la página de App Services con una aplicación web de ejemplo seleccionada.":::

La página **Información general** de la aplicación web contiene opciones para la administración básica como examinar, detener, iniciar, reiniciar y eliminar. El menú izquierdo proporciona varias páginas para configurar la aplicación.

:::image type="content" source="media/quickstart-dotnetcore/web-app-overview-page.png" alt-text="Azure Portal: página de información general de App Service.":::

<!--
## Clean up resources - H2 added from the next three includes
-->
:::zone target="docs" pivot="development-environment-vs"
[!INCLUDE [Clean-up Portal web app resources](../../includes/clean-up-section-portal-web-app.md)]
:::zone-end

:::zone target="docs" pivot="development-environment-vscode"
[!INCLUDE [Clean-up Portal web app resources](../../includes/clean-up-section-portal-web-app.md)]
:::zone-end

<!-- markdownlint-disable MD044 -->
:::zone target="docs" pivot="development-environment-cli"
<!-- markdownlint-enable MD044 -->
[!INCLUDE [Clean-up CLI resources](../../includes/cli-samples-clean-up.md)]
:::zone-end

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado e implementado una aplicación web de ASP.NET en Azure App Service.

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

Pase al siguiente artículo para aprender a crear una aplicación de .NET Core y conectarla a una instancia de SQL Database:

> [!div class="nextstepaction"]
> [Tutorial: Aplicación de ASP.NET Core con SQL Database](tutorial-dotnetcore-sqldb-app.md)

> [!div class="nextstepaction"]
> [Configuración de una aplicación de ASP.NET Core 3.1](configure-language-dotnetcore.md)

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

Pase al siguiente artículo para aprender a crear una aplicación de .NET Core y conectarla a una instancia de SQL Database:

> [!div class="nextstepaction"]
> [Tutorial: Aplicación de ASP.NET Core con SQL Database](tutorial-dotnetcore-sqldb-app.md)

> [!div class="nextstepaction"]
> [Configuración de una aplicación de ASP.NET Core 5.0](configure-language-dotnetcore.md)

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

En el siguiente artículo aprenderá a crear una aplicación de .NET Framework y conectarla a una instancia de SQL Database:

> [!div class="nextstepaction"]
> [Tutorial: Aplicación de ASP.NET con SQL Database](app-service-web-tutorial-dotnet-sqldatabase.md)

> [!div class="nextstepaction"]
> [Configuración de una aplicación de ASP.NET Framework](configure-language-dotnet-framework.md)

---

[app-service-pricing-tier]: https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
