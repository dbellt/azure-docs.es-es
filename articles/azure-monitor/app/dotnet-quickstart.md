---
title: 'Inicio rápido: Supervisión de una aplicación de ASP.NET Core con Azure Monitor Application Insights'
description: Instrumente una aplicación web de ASP.NET Core para la supervisión con Azure Monitor Application Insights.
ms.topic: quickstart
author: lgayhardt
ms.author: lagayhar
ms.date: 06/11/2021
ms.custom: devx-track-dotnet
ms.openlocfilehash: 90d4fd76350d03bd876d97a7191011079b11845e
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112300999"
---
# <a name="quickstart-monitor-an-aspnet-core-app-with-azure-monitor-application-insights"></a>Inicio rápido: Supervisión de una aplicación de ASP.NET Core con Azure Monitor Application Insights

En este inicio rápido, instrumentará una aplicación de ASP.NET Core mediante el SDK de Application Insights para recopilar telemetría del lado cliente y del lado servidor.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/dotnet).
- SDK de .NET 5.0 o posterior. [Instale el SDK de .NET 5.0 más reciente](https://dotnet.microsoft.com/download/dotnet/5.0) para su plataforma.

## <a name="create-an-application-insights-resource"></a>Creación de recursos en Application Insights

Para comenzar a ingerir telemetría, cree un recurso de Application Insights en la suscripción de Azure.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Seleccione **Crear un recurso** > **Herramientas de desarrollo** > **Application Insights**.

1. Complete el formulario que aparece:
    1. Compruebe la **Suscripción** seleccionada.
    1. Seleccione un **Grupo de recursos** existente o uno nuevo.
    1. Especifique un **Nombre** para este recurso de Application Insights.
    1. Seleccione una **Región** cerca de su ubicación.
    1. Establezca el **Modo de recursos** en *Clásico*. 

1. Seleccione el botón **Revisar y Crear**.
1. Seleccione el botón **Crear**.
1. Una vez completada la implementación, seleccione el botón **Ir al recurso**.
1. En la **Información general** que aparece, copie la **Clave de instrumentación** (que se encuentra en **Información esencial**).

## <a name="create-and-configure-an-aspnet-core-web-app"></a>Creación y configuración de una aplicación web de ASP.NET Core

Complete los pasos siguientes para crear y configurar una nueva aplicación web de ASP.NET Core:

1. Creación de una nueva aplicación Razor Pages de ASP.NET Core:
    
    ```dotnetcli
    dotnet new razor -o ai.quickstart
    ```
    
    El comando anterior crea una nueva aplicación Razor Pages de ASP.NET Core en un directorio denominado *ai.quickstart*. 
    
    > [!TIP]
    > Es posible que prefiera [usar Visual Studio para crear la aplicación](/visualstudio/ide/quickstart-aspnet-core).

1. Dentro del directorio del proyecto, agregue el paquete `Microsoft.ApplicationInsights.AspNetCore` al proyecto. Si utiliza Visual Studio, puede usar el [Administrador de paquetes NuGet](/nuget/consume-packages/install-use-packages-visual-studio).

    ```dotnetcli
    dotnet add package Microsoft.ApplicationInsights.AspNetCore --version 2.17.0 
    ```

1. Con un editor de texto o entorno de desarrollo integrado (IDE), modifique *appsettings.jsen* para que contenga un valor para `ApplicationInsights.InstrumentationKey`, tal como se muestra. Utilice la clave de instrumentación copiada anteriormente.

    :::code language="json" source="~/dotnet-samples/azure/app-insights-aspnet-core-quickstart/appsettings.json" range="1-12" highlight="2-4":::
    
    > [!IMPORTANT]
    > El SDK de Application Insights espera el valor de configuración `ApplicationInsights.InstrumentationKey`. Asegúrese de asignarle un nombre correcto.

## <a name="configure-server-side-telemetry"></a>Configuración de la telemetría del lado servidor

En el método `ConfigureServices` de *Startup.cs*, agregue el servicio Application Insights a la canalización. Agregue la línea resaltada:

:::code language="csharp" source="~/dotnet-samples/azure/app-insights-aspnet-core-quickstart/startup.cs" range="24-28" highlight="4":::

## <a name="configure-client-side-telemetry"></a>Configuración de la telemetría en el lado cliente

Complete los pasos siguientes a fin de instrumentar la aplicación para enviar telemetría del lado cliente:

1. En *Pages/_ViewImports.cshtml*, agregue la siguiente línea:

    ```cshtml
    @inject Microsoft.ApplicationInsights.AspNetCore.JavaScriptSnippet JavaScriptSnippet
    ```

    El cambio anterior registra una dependencia `Microsoft.ApplicationInsights.AspNetCore.JavaScriptSnippet` que contiene el elemento de script del lado cliente de Application Insights.

1. En *Pages/Shared/_Layout.cshtml*, en el elemento `<head>`, agregue la línea resaltada:

    :::code language="cshtml" source="~/dotnet-samples/azure/app-insights-aspnet-core-quickstart/pages/shared/_layout.cshtml" range="3-10" highlight="7":::

   Este cambio usa el objeto `JavaScriptSnippet` inyectado para asegurarse de que el elemento `<script>` se representa en el elemento `<head>` de cada página de la aplicación.

## <a name="validate-telemetry-ingestion"></a>Validación de la ingesta de telemetría

La telemetría tarda varios minutos en ingerirse en Application Insights para su análisis. Para comprobar que la aplicación envía telemetría en tiempo real, utilice **Live Metrics**:

1. Ejecute la aplicación web mediante `dotnet run` o el entorno de desarrollo integrado (IDE).
1. En Azure Portal, cuando vea el recurso de Application Insights, seleccione **Métricas en directo** en **Investigar**.
1. En la aplicación, seleccione los vínculos **Inicio** y **Privacidad** repetidamente.
1. Observe la actividad en la pantalla **Métricas en directo** a medida que se realizan solicitudes en la aplicación.

## <a name="next-steps"></a>Pasos siguientes

Felicidades. Ya puede usar la telemetría enviada por la aplicación para:

- [Buscar excepciones del runtime](tutorial-runtime-exceptions.md).
- [Buscar problemas de rendimiento](tutorial-performance.md).
- [Alertar sobre el estado de la aplicación](tutorial-alert.md).

> [!div class="nextstepaction"]
> [Obtener más información sobre Application Insights en ASP.NET Core](asp-net-core.md)
