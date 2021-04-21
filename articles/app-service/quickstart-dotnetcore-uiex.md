---
title: 'Inicio rápido: Creación de una aplicación de ASP.NET Core en C#'
description: Aprenda a ejecutar aplicaciones web en Azure App Service mediante la implementación de su primera aplicación web ASP.NET Core.
ms.assetid: b1e6bd58-48d1-4007-9d6c-53fd6db061e3
ms.topic: quickstart
ms.date: 11/23/2020
ms.custom: devx-track-csharp, mvc, devcenter, vs-azure, seodec18, contperf-fy21q1
zone_pivot_groups: app-service-platform-windows-linux
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: ff86bedf47395b50dc25e552b8b3ed4176e23b65
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107769110"
---
# <a name="quickstart-create-an-aspnet-core-web-app-in-azure"></a>Inicio rápido: Creación de una aplicación web ASP.NET Core en Azure

::: zone pivot="platform-windows"  

En esta guía de inicio rápido, aprenderá a crear e implementar su primera aplicación web de ASP.NET Core en <abbr title="Un servicio basado en HTTP para hospedar aplicaciones web, API REST y aplicaciones de back-end para dispositivos móviles.">Azure App Service</abbr>. App Service es compatible con aplicaciones de .NET 5.0.

Cuando haya terminado, tendrá un <abbr title="Un contenedor lógico para recursos relacionados de Azure que se pueden administrar como una unidad.">resource group</abbr>de Azure, que consta de un <abbr title="El plan especifica la ubicación, el tamaño y las características de la granja de servidores web que hospeda la aplicación.">Plan de App Service</abbr> y una <abbr title="La representación de la aplicación web, que contiene el código de la aplicación, los nombres de host DNS, los certificados y los recursos relacionados.">Aplicación de App Service</abbr> con una aplicación de ASP.NET Core implementada.

<hr/> 

## <a name="1-prepare-your-environment"></a>1. Preparación del entorno

- **Obtenga una cuenta de Azure** con una <abbr title="La estructura organizativa básica en la que administra los recursos de Azure, que normalmente se asocia a un individuo o departamento dentro de una organización.">subscription</abbr>. [Cree una cuenta gratuita](https://azure.microsoft.com/free/dotnet/).
- **Instale <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019</a>** con la carga de trabajo **ASP.NET y desarrollo web**.

<details>
<summary>¿Ya tiene Visual Studio 2019?</summary>
Si ya ha instalado Visual Studio 2019:

<ul>
<li><strong>Instale las actualizaciones más recientes</strong> de Visual Studio. Para ello, seleccione <strong>Ayuda</strong> &gt; <strong>Buscar actualizaciones</strong>. Las actualizaciones más recientes contienen el SDK de .NET 5.0.</li>
<li><strong>Agregue la carga de trabajo</strong>. Para ello, seleccione <strong>Herramientas</strong> &gt; <strong>Obtener herramientas y características</strong>.</li>
</ul>
</details>

<hr/> 

## <a name="2-create-an-aspnet-core-web-app"></a>2. Cree una aplicación web ASP.NET Core

1. Abra Visual Studio y seleccione **Crear un proyecto**.

1. En **Crear un nuevo proyecto**, seleccione **Aplicación web ASP.NET Core**  y confirme que **C#** aparece en los lenguajes de esa opción y, a continuación, seleccione **Siguiente**.

1. En **Configure su nuevo proyecto**, asigne al proyecto de aplicación web el nombre *miPrimeraAzureWebApp* y luego seleccione **Crear**.

   ![Configuración del proyecto de aplicación web](./media/quickstart-dotnetcore/configure-web-app-project.png)

1. Para una aplicación de .NET 5.0, seleccione **ASP.NET Core 5.0** en el menú desplegable. De lo contrario, use el valor predeterminado.

1. Puede implementar cualquier tipo de aplicación web de ASP.NET Core en Azure pero, para este inicio rápido, elija la plantilla de **Aplicación web de ASP.NET Core**. Asegúrese de que **Autenticación** esté establecido en **Sin autenticación** y que no hay ninguna otra opción seleccionada. Seleccione **Crear**.

   ![Creación de una nueva aplicación web de ASP.NET Core](./media/quickstart-dotnetcore/create-aspnet-core-web-app-5.png) 
   
1. En el menú de Visual Studio, seleccione **Depurar** > **Iniciar sin depurar** para ejecutar la aplicación web localmente.

   ![Aplicación web que se ejecuta localmente](./media/quickstart-dotnetcore/web-app-running-locally.png)

<hr/> 

## <a name="3-publish-your-web-app"></a>3. Publicación de la aplicación web

1. En **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **myFirstAzureWebApp** y seleccione **Publicar**. 

1. En **Publicar**, seleccione **Azure** y haga clic en **Siguiente**.

1. Las opciones dependen de si ya ha iniciado sesión en Azure y de si tiene una cuenta de Visual Studio vinculada a una cuenta de Azure. Seleccione **Agregar una cuenta** o **Iniciar sesión** para iniciar sesión en la suscripción de Azure. Si ya ha iniciado sesión, seleccione la cuenta que desee.

   ![Inicio de sesión en Azure](./media/quickstart-dotnetcore/sign-in-azure-vs2019.png)

1. A la derecha de **App Service instances** (Instancias de App Service), haga clic en **+** .

   ![Nueva aplicación de App Service](./media/quickstart-dotnetcore/publish-new-app-service.png)

1. En **Suscripción**, acepte la suscripción que aparece o seleccione otra en la lista desplegable.

1. En **Grupo de recursos**, seleccione **Nuevo**. En **Nuevo nombre de grupo de recursos**, escriba *myResourceGroup* y seleccione **Aceptar**. 

1. En **Plan de hospedaje**, seleccione **Nuevo**. 

1. En el cuadro de diálogo **Plan de hospedaje: Crear nuevo**, escriba los valores especificados en la tabla siguiente:

   | Configuración  | Valor sugerido |
   | -------- | --------------- |
   | **Plan de hospedaje**  | *miPrimerPlanDeAzureWebApp* |
   | **Ubicación**      | *Oeste de Europa* |
   | **Tamaño**          | *Gratis* |
   
   ![Creación de un nuevo plan de hospedaje](./media/quickstart-dotnetcore/create-new-hosting-plan-vs2019.png)

1. En **Nombre**, escriba un nombre de aplicación único.

    <details>
        <summary>¿Qué caracteres se pueden usar?</summary>
        Los caracteres válidos son a-z, A-Z, 0-9 y -. Puede aceptar el nombre único generado automáticamente. La dirección URL de la aplicación web es http://<code>&lt;app-name&gt;.azurewebsites.net</code>, donde <code>&lt;app-name&gt;</code> es el nombre de la aplicación.
    </details>

1. Seleccione **Crear** para crear los recursos de Azure. 

   ![Creación de recursos de aplicación](./media/quickstart-dotnetcore/web-app-name-vs2019.png)

1. Espere a que el asistente termine de crear los recursos de Azure. Seleccione **Finalizar** para cerrar el asistente.

1. En la página **Publicar**, haga clic en **Publicar** para implementar el proyecto. 

    <details>
        <summary>¿Qué hace Visual Studio?</summary>
        Visual Studio compila, empaqueta y publica la aplicación en Azure y, luego, la inicia en el explorador predeterminado.
    </details>

   ![Aplicación web de ASP.NET publicada en ejecución en Azure](./media/quickstart-dotnetcore/web-app-running-live.png)

<hr/> 

## <a name="4-update-the-app-and-redeploy"></a>4. Actualización de la aplicación y nueva implementación

1. En **Explorador de soluciones**, en el proyecto, abra **Pages** > **Index.cshtml**.

1. Reemplace la etiqueta `<div>` entera por el siguiente código:

   ```html
   <div class="jumbotron">
       <h1>ASP.NET in Azure!</h1>
       <p class="lead">This is a simple app that we've built that demonstrates how to deploy a .NET app to Azure App Service.</p>
   </div>
   ```

1. Para volver a implementar en Azure, haga clic con el botón derecho en el proyecto **myFirstAzureWebApp**, en el **Explorador de soluciones** y seleccione **Publicar**.

1. En la página de resumen **Publicar**, seleccione **Publicar**.

   <!-- ![Publish update to web app](./media/quickstart-dotnetcore/publish-update-to-web-app-vs2019.png) -->

    Cuando se completa la publicación, Visual Studio inicia un explorador en la dirección URL de la aplicación web.

    ![Aplicación web actualizada de ASP.NET en Azure](./media/quickstart-dotnetcore/updated-web-app-running-live.png)

<hr/> 

## <a name="5-manage-the-azure-app"></a>5. Administración de la aplicación de Azure

1. Vaya a [Azure Portal](https://portal.azure.com) y busque y seleccione **App Services**.

    ![Selección de App Services](./media/quickstart-dotnetcore/app-services.png)
    
1. En la página **App Services**, seleccione el nombre de la aplicación web.

    :::image type="content" source="./media/quickstart-dotnetcore/select-app-service.png" alt-text="Captura de pantalla de la página de App Services con una aplicación web de ejemplo seleccionada.":::

1. La página **Información general** de la aplicación web contiene opciones para la administración básica como examinar, detener, iniciar, reiniciar y eliminar. El menú izquierdo proporciona varias páginas para configurar la aplicación.

    ![App Service en Azure Portal](./media/quickstart-dotnetcore/web-app-overview-page.png)
    
<hr/> 

## <a name="6-clean-up-resources"></a>6. Limpieza de recursos

1. En el menú de Azure Portal o la página **Inicio**, seleccione **Grupos de recursos**. En la página **Grupos de recursos**, seleccione **myResourceGroup**.

1. En la página **myResourceGroup**, asegúrese de que los recursos enumerados sean los que desea eliminar.

1. Seleccione **Eliminar grupo de recursos**, escriba **myResourceGroup** en el cuadro de texto para confirmar y, después, seleccione **Eliminar**.

<hr/> 

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo para aprender a crear una aplicación de .NET Core y conectarla a una instancia de SQL Database:

- [ASP.NET Core con SQL Database](tutorial-dotnetcore-sqldb-app.md)
- [Configure ASP.NET Core app](configure-language-dotnetcore.md) (Configuración de una aplicación de ASP.NET Core)

::: zone-end  

::: zone pivot="platform-linux"
En esta guía de inicio rápido se muestra cómo crear una aplicación de [.NET Core](/aspnet/core/) en <abbr title="App Service en Linux proporciona un servicio de hospedaje web muy escalable y con aplicación automática de revisiones utilizando el sistema operativo Linux.">App Service en Linux</abbr>. Creará la aplicación web con la [CLI de Azure](/cli/azure/get-started-with-azure-cli) y usará Git para implementar el código de .NET Core en la aplicación.

<hr/> 

## <a name="1-prepare-your-environment"></a>1. Preparación del entorno

- **Obtenga una cuenta de Azure** con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/dotnet/).
- **Instale** el <a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">SDK para .NET Core 3.1</a> más reciente o el <a href="https://dotnet.microsoft.com/download/dotnet/5.0" target="_blank">SDK para .NET 5.0</a>.
- **<a href="/cli/azure/install-azure-cli" target="_blank">Instale la CLI de Azure más reciente</a>** .

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="2-create-the-app-locally"></a>2. Creación de la aplicación localmente

1. Ejecute `mkdir hellodotnetcore` para crear el directorio.

    ```bash
    mkdir hellodotnetcore
    ```

1. Ejecute `cd hellodotnetcore` para cambiar al directorio. 

    ```bash
    cd hellodotnetcore
    ```

1. Ejecute `dotnet new web` para crear una nueva aplicación de .NET Core.

    ```bash
    dotnet new web
    ```

<hr/> 

## <a name="3-run-the-app-locally"></a>3. Ejecución de la aplicación de forma local

1. Ejecute `dotnet run` para ver su aspecto cuando la implemente en Azure.

    ```bash
    dotnet run
    ```
    
1. **Abra un explorador web** y vaya a la aplicación en `http://localhost:5000`.

![Prueba con un explorador](media/quickstart-dotnetcore/dotnet-browse-local.png)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="4-sign-into-azure"></a>4. Inicio de sesión en Azure

Ejecute `az login` para iniciar sesión en Azure.

```azurecli
az login
```

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="5-deploy-the-app"></a>5. Implementar la aplicación

1. **Ejecute** `az webapp up` en la carpeta local. **Reemplace** por un nombre único globalmente.

    ```azurecli
    az webapp up --sku F1 --name <app-name> --os-type linux
    ```
    
    <details>
    <summary>Solución de problemas</summary>
    <ul>
    <li>Si no se reconoce el comando <code>az</code>, asegúrese de tener instalada la CLI de Azure, tal y como se describe en <a href="#1-prepare-your-environment">Preparación del entorno inicial</a>.</li>
    <li>Reemplace <code>&lt;app-name&gt;</code> por un nombre que sea único en todo Azure (<em>los caracteres válidos son <code>a-z</code>, <code>0-9</code> y <code>-</code></em>). Un buen patrón es usar una combinación del nombre de la empresa y un identificador de la aplicación.</li>
    <li>El argumento <code>--sku F1</code> crea la aplicación web en el plan de tarifa Gratuito. Omita este argumento para usar un nivel Premium más rápido, lo que supondrá un costo por hora.</li>
    <li>Opcionalmente, incluya el argumento <code>--location &lt;location-name&gt;</code>, donde <code>&lt;location-name&gt;</code> es una región de Azure disponible. Puede recuperar una lista de las regiones permitidas para su cuenta de Azure mediante la ejecución del comando <a href="/cli/azure/appservice#az_appservice_list_locations"><code>az account list-locations</code></a>.</li>
    </ul>
    </details>
    
1. Espere hasta que finalice el comando. Puede tardar algunos minutos y termina con el mensaje "Puede iniciar la aplicación en http://&lt;app-name&gt;.azurewebsites.net".

    <details>
    <summary>¿Qué hace <code>az webapp up</code>?</summary>
    <p>El comando <code>az webapp up</code> realiza las acciones siguientes:</p>
    <ul>
    <li>Crear un grupo de recursos predeterminado.</li>
    <li>Crear un plan de App Service predeterminado.</li>
    <li><a href="/cli/azure/webapp#az_webapp_create">Cree una aplicación de App Service</a> con el nombre especificado.</li>
    <li><a href="/azure/app-service/deploy-zip">Implementar con ZIP</a> archivos desde el directorio de trabajo actual a la aplicación.</li>
    <li>Mientras se ejecuta, proporciona mensajes sobre la creación de recursos, el registro y la implementación con zip.</li>
    </ul>
    </details>
    
# <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

![Salida de ejemplo del comando az webapp up](./media/quickstart-dotnetcore/az-webapp-up-output-3.1.png)

# <a name="net-50"></a>[.NET 5.0](#tab/net50)

![Salida de ejemplo del comando az webapp up](./media/quickstart-dotnetcore/az-webapp-up-output-5.0.png)

---

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="6-browse-to-the-app"></a>6. Navegación hasta la aplicación

**Vaya a la aplicación implementada** mediante el explorador web.

```bash
http://<app_name>.azurewebsites.net
```

![Aplicación de ejemplo que se ejecuta en Azure](media/quickstart-dotnetcore/dotnet-browse-azure.png)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="7-update-and-redeploy-the-code&quot;></a>7. Actualización del código y nueva implementación

1. **Abra el archivo _Startup.cs_** en el directorio local. 

1. **Realice un pequeño cambio** en el texto en la llamada al método `context.Response.WriteAsync`.

    ```csharp
    await context.Response.WriteAsync(&quot;Hello Azure!");
    ```
    
1. **Guarde los cambios**.

1. **Ejecute** `az webapp up` para volver a implementar:

    ```azurecli
    az webapp up --os-type linux
    ```
    
    <details>
    <summary>¿Qué hace ahora <code>az webapp up</code>?</summary>
    La primera vez que ejecutó el comando, este guardó el nombre de la aplicación, el grupo de recursos y el plan de App Service en el archivo <i>.azure/config</i> de la raíz del proyecto. Cuando se vuelve a ejecutar desde la raíz del proyecto, usa los valores guardados en <i>.azure/config</i>, detecta que los recursos de App Service ya existen y realiza la implementación con zip de nuevo.
    </details>
    
1. Una vez finalizada la implementación, **pulse actualizar** en la ventana del explorador que abrió anteriormente.

    ![Aplicación de ejemplo actualizada que se ejecuta en Azure](media/quickstart-dotnetcore/dotnet-browse-azure-updated.png)
    
[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="8-manage-your-new-azure-app"></a>8. Administración de la nueva aplicación de Azure

1. Vaya a <a href="https://portal.azure.com" target="_blank">Azure Portal</a>.

1. En el menú izquierdo, haga clic en **App Services** y, luego, en el nombre de la aplicación de Azure.

    :::image type="content" source="./media/quickstart-dotnetcore/portal-app-service-list-up.png" alt-text="Captura de pantalla de la página de App Services en la que se muestra una aplicación de ejemplo de Azure seleccionada.":::

1. Desde la página de información general puede realizar tareas de administración básicas como examinar, detener, iniciar, reiniciar y eliminar. El menú izquierdo proporciona distintas páginas para configurar la aplicación. 

    ![Página de App Service en Azure Portal](media/quickstart-dotnetcore/portal-app-overview-up.png)
    
<hr/> 

## <a name="9-clean-up-resources"></a>9. Limpieza de recursos

**Ejecute** `az group delete --name myResourceGroup` para eliminar el grupo de recursos.

```azurecli-interactive
az group delete --name myResourceGroup
```

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Aplicación de ASP.NET Core con SQL Database](tutorial-dotnetcore-sqldb-app.md)
- [Configure ASP.NET Core app](configure-language-dotnetcore.md) (Configuración de una aplicación de ASP.NET Core)

::: zone-end
