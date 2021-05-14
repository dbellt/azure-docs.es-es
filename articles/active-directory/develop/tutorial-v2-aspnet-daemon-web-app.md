---
title: 'Tutorial: Creación de un demonio multiinquilino que acceda a los datos empresariales de Microsoft Graph | Azure'
titleSuffix: Microsoft identity platform
description: En este inicio rápido, aprenderá a llamar a una API web de ASP.NET protegida por Azure Active Directory desde una aplicación de escritorio de Windows (WPF). El cliente de WPF autentica a un usuario, solicita un token de acceso y llama a la API web.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 12/10/2019
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started, languages:ASP.NET
ms.openlocfilehash: 275aba6e9c7b87ec7c6fb1cbc986b3ebaa3a9696
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108130960"
---
# <a name="tutorial-build-a-multi-tenant-daemon-that-uses-the-microsoft-identity-platform"></a>Tutorial: Creación de un demonio multiinquilino que usa la plataforma de identidad de Microsoft

En este tutorial, descargará y ejecutará una aplicación web de demonio de ASP.NET que muestra el uso de la concesión de credenciales de cliente de OAuth 2.0 para obtener un token de acceso para llamar a Microsoft Graph API.

En este tutorial:

> [!div class="checklist"]
> * Integración de una aplicación de demonio con la Plataforma de identidad de Microsoft
> * Concesión de permisos de aplicación directamente a la aplicación por un administrador
> * Obtención de un token de acceso para llamar a Microsoft Graph API
> * Llamada a Microsoft Graph API

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

- [Visual Studio 2017 o 2019](https://visualstudio.microsoft.com/downloads/)
- Un inquilino de Azure AD. Para más información, consulte [Configuración de un inquilino de Azure AD](quickstart-create-new-tenant.md).
- Una o varias cuentas de usuario en el inquilino de Azure AD. Este ejemplo no funcionará con una cuenta Microsoft. Si ha iniciado sesión en [Azure Portal](https://portal.azure.com) con una cuenta Microsoft y nunca ha creado una cuenta de usuario en el directorio, hágalo ahora.

## <a name="scenario"></a>Escenario

La aplicación se compila como una aplicación ASP.NET MVC. Usa el middleware de OWIN OpenID Connect para iniciar la sesión de los usuarios.

El componente "demonio" de este ejemplo es un controlador de API `SyncController.cs`. Cuando se llama al controlador, se extrae de Microsoft Graph una lista de usuarios del inquilino de Azure Active Directory del cliente (Azure AD). `SyncController.cs` se desencadena mediante una llamada AJAX en la aplicación web. Dicha aplicación usa la [biblioteca de autenticación de Microsoft (MSAL) para .NET](msal-overview.md) para adquirir un token de acceso para Microsoft Graph.

Dado que la aplicación es una aplicación multiinquilino para clientes empresariales de Microsoft, debe proporcionar a los clientes una forma de "registrar" la aplicación o "conectarla" a los datos de la empresa. Durante el flujo de conexión, el administrador global concede primero los *permisos directamente a la aplicación*. De este modo, la aplicación puede acceder a los datos de la empresa sin necesidad de utilizar recursos interactivos ni la presencia de un usuario con la sesión iniciada. La mayoría de la lógica de este ejemplo muestra cómo lograr este flujo de conexión mediante el punto de conexión de [consentimiento del administrador](v2-permissions-and-consent.md#using-the-admin-consent-endpoint) de la plataforma de identidad.

![En el diagrama se muestra la aplicación UserSync con tres elementos locales que se conectan a Azure, con la autenticación de punto inicial que adquiere un token de forma interactiva para conectarse a Azure AD, AccountController que obtiene el consentimiento de administrador para conectarse a Azure AD y SyncController que lee al usuario para conectarse a Microsoft Graph.](./media/tutorial-v2-aspnet-daemon-webapp/topology.png)

Para más información sobre los conceptos que se usan en este ejemplo, lea la [documentación del protocolo de credenciales de cliente de la plataforma de identidad](v2-oauth2-client-creds-grant-flow.md).

## <a name="clone-or-download-this-repository"></a>Clonación o descarga de este repositorio

En el shell o la línea de comandos, escriba este comando:

```Shell
git clone https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2.git
```

También puede [descargar el ejemplo en un archivo ZIP](https://github.com/Azure-Samples/ms-identity-aspnet-daemon-webapp/archive/master.zip).

## <a name="register-your-application"></a>Registrar su aplicación

Este ejemplo tiene un proyecto. Para registrar la aplicación en el inquilino de Azure AD, puede:

- Seguir los pasos que se indican en [Registro del ejemplo en el inquilino de Azure Active Directory](#register-the-client-app-dotnet-web-daemon-v2) y [Configuración del ejemplo para usar el inquilino de Azure AD](#choose-the-azure-ad-tenant).
- Usar scripts de PowerShell que realizan las siguientes acciones:
  - Crean *automáticamente* las aplicaciones de Azure AD y los objetos relacionados (contraseñas, permisos, dependencias).
  - Modifican los archivos de configuración de los proyectos de Visual Studio.

Si quiere usar esta automatización:

1. En Windows, ejecute PowerShell y vaya a la raíz del directorio clonado.
1. Ejecute este comando:

   ```PowerShell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
   ```

1. Ejecute el script para crear la aplicación de Azure AD y configure el código de la aplicación de ejemplo como corresponda:

   ```PowerShell
   .\AppCreationScripts\Configure.ps1
   ```

   En la sección de [scripts de creación de aplicaciones](https://github.com/Azure-Samples/ms-identity-aspnet-daemon-webapp/blob/master/AppCreationScripts/AppCreationScripts.md), se describen otras formas de ejecutar los scripts.

1. Abra la solución de Visual Studio y seleccione **Iniciar** para ejecutar el código.

Si no desea usar la automatización, siga los pasos que se describen en las secciones siguientes.

### <a name="choose-the-azure-ad-tenant"></a>Selección del inquilino de Azure AD

1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino en el que desea registrar una aplicación.


### <a name="register-the-client-app-dotnet-web-daemon-v2"></a>Registro de la aplicación cliente (dotnet-web-daemon-v2)

1. Busque y seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Registros de aplicaciones** >  y, luego, **Nuevo registro**.
1. Escriba el **Nombre** de la aplicación, por ejemplo `dotnet-web-daemon-v2`. Los usuarios de la aplicación pueden ver este nombre, el cual se puede cambiar más tarde.
1. En la sección **Tipos de cuenta compatibles**, seleccione **Cuentas en cualquier directorio organizativo**.
1. En la sección **URI de redirección (opcional)** , seleccione **Web** en el cuadro combinado y escriba los URI de redirección `https://localhost:44316/` y `https://localhost:44316/Account/GrantPermissions`.

    Si hay más de dos URI de redirección, deberá agregarlos desde la pestaña **Autenticación** más adelante, después de que la aplicación se haya creado correctamente.
1. Seleccione **Registrar** para crear la aplicación.
1. En la página **Información general** de la aplicación, busque el valor de **Id. de aplicación (cliente)** y regístrelo para usarlo más tarde. Lo necesitará para configurar el archivo de configuración de Visual Studio para este proyecto.
1. En **Administrar**, seleccione **Autenticación**.
1. Establezca la **dirección URL de cierre de sesión del canal frontal** en `https://localhost:44316/Account/EndSession`.
1. En la sección **Implicit grant and hybrid flows** (Flujos de concesión implícita e híbridos), seleccione **Tokens de acceso** y **Tokens de id**. Para este ejemplo es necesario habilitar el [flujo de concesión implícita](v2-oauth2-implicit-grant-flow.md) para iniciar la sesión del usuario y llamar a una API.
1. Seleccione **Guardar**.
1. En **Administrar**, seleccione **Certificados y secretos**.
1. En la sección **Secretos de cliente**, seleccione **Nuevo secreto de cliente**. 
1. Escriba una descripción de clave (por ejemplo, **secreto de aplicación**).
1. Seleccione una duración de la clave: **En 1 año**, **En 2 años** o **Nunca expira**.
1. Seleccione **Agregar**. Registre el valor de la clave en una ubicación segura. Necesitará esta clave más adelante para configurar el proyecto en Visual Studio.
1. En **Administrar**, seleccione **Permisos de API** > **Add a permission** (Agregar un permiso).
1. En la sección **API de Microsoft más usadas**, seleccione **Microsoft Graph**.
1. En la sección **Permisos de aplicación**, asegúrese de que estén seleccionados los permisos correctos: **User.Read.All**.
1. Seleccione **Agregar permisos**.

## <a name="configure-the-sample-to-use-your-azure-ad-tenant"></a>Configuración del ejemplo para usar el inquilino de Azure AD

En los pasos siguientes, **ClientID** es lo mismo que "Id. de aplicación" o **AppId**.

Abra la solución en Visual Studio para configurar los proyectos.

### <a name="configure-the-client-project"></a>Configuración del proyecto de cliente

Si ha usado los scripts de instalación, se habrán aplicado los siguientes cambios.

1. Abra el archivo **UserSync\Web.Config**.
1. Busque la clave de la aplicación **ida:ClientId**. Reemplace el valor existente por el identificador de aplicación **dotnet-web-daemon-v2** copiado de Azure Portal.
1. Busque la clave de aplicación **ida:ClientSecret**. Reemplace el valor existente por la clave que guardó durante la creación de la aplicación **dotnet-web-daemon-v2** en Azure Portal.

## <a name="run-the-sample"></a>Ejecución del ejemplo

Limpie la solución, vuelva a compilarla y ejecute la aplicación UserSync. Luego, inicie sesión como administrador en el inquilino de Azure AD. Si aún no tiene un inquilino de Azure AD, [siga estas instrucciones](quickstart-create-new-tenant.md) para obtener uno.

Cuando inicie sesión, la aplicación le pedirá primero permiso para iniciar su sesión y leer su perfil de usuario. Este consentimiento permite a la aplicación asegurarse de que es un usuario empresarial.

![Consentimiento del usuario](./media/tutorial-v2-aspnet-daemon-webapp/firstconsent.png)

A continuación, la aplicación intenta sincronizar una lista de usuarios de su inquilino de Azure AD mediante Microsoft Graph. Si no puede, le pedirá al administrador de inquilinos que conecte el inquilino a la aplicación.

La aplicación le pedirá entonces permiso para leer la lista de usuarios del inquilino.

![Consentimiento de administrador](./media/tutorial-v2-aspnet-daemon-webapp/adminconsent.PNG)

Después de conceder el permiso, se cerrará la sesión en la aplicación. Este cierre de sesión garantiza que todos los tokens de acceso existentes para Microsoft Graph se quitan de la caché de tokens. Tras volver a iniciar la sesión, el token nuevo obtenido tendrá los permisos necesarios para realizar llamadas a Microsoft Graph.


Al conceder el permiso, la aplicación puede realizar consultas de usuarios en cualquier momento. Para comprobarlo, seleccione el botón **Sincronizar usuarios** y actualice la lista de usuarios. Intente agregar o quitar un usuario y volver a sincronizar la lista. (Sin embargo, tenga en cuenta que la aplicación solo sincroniza la primera página de usuarios).

## <a name="about-the-code"></a>Sobre el código

El código de interés para este ejemplo se encuentra en los siguientes archivos:

- **App_Start\Startup.Auth.cs**, **Controllers\AccountController.cs**: inicio de sesión inicial. En concreto, las acciones del controlador tienen un atributo **Authorize** que obliga al usuario a iniciar sesión. La aplicación usa el [flujo de código de autorización](v2-oauth2-auth-code-flow.md) para iniciar la sesión del usuario.
- **Controllers\SyncController.cs**: sincronización de la lista de usuarios con el almacén local en memoria.
- **Controllers\UserController.cs**: sincronización de la lista de usuarios desde el almacén en memoria local.
- **Controllers\AccountController.cs**: adquisición de permisos del administrador de inquilinos mediante el punto de conexión de consentimiento del administrador.

## <a name="re-create-the-sample-app"></a>Volver a crear la aplicación de ejemplo

1. En Visual Studio, cree un nuevo proyecto de **Aplicación web ASP.NET (.NET Framework)** de **Visual C#** .
1. En la siguiente pantalla, elija la plantilla de proyecto **MVC**. Agregue también referencias de carpeta y núcleo para **API web**, ya que más adelante agregará un controlador de API web. Deje el modo de autenticación elegido del proyecto como predeterminado: **Sin autenticación**.
1. Seleccione el proyecto en la ventana **Explorador de soluciones** y presione la tecla **F4**.
1. En las propiedades del proyecto, establezca **SSL habilitado** en **True**. Anote la información de **Dirección URL de SSL**. La necesitará al configurar el registro de esta aplicación en Azure Portal.
1. Agregue los siguientes paquetes NuGet de middleware de ASP.NET OWIN:
   - Microsoft.Owin.Security.ActiveDirectory
   - Microsoft.Owin.Security.Cookies
   - Microsoft.Owin.Host.SystemWeb
   - Microsoft.IdentityModel.Protocol.Extensions
   - Microsoft.Owin.Security.OpenIdConnect
   - Microsoft.Identity.Client
1. En la carpeta **App_Start**:
   1. Cree una clase llamada **Startup.Auth.cs**.
   1. Quite **.App_Start** del nombre del espacio de nombres.
   1. Reemplace el código de la clase **Startup** por el código del mismo archivo de la aplicación de ejemplo.
   Asegúrese de usar la definición de clase completa. La definición cambia de **clase de inicio pública** a **clase de inicio parcial pública**.
1. En **Startup.Auth.cs**, agregue instrucciones **using**, como se sugiere en Visual Studio IntelliSense, para resolver las referencias que faltan.
1. Haga clic con el botón derecho en el proyecto, seleccione **Agregar** y, luego, seleccione **Clase**.
1. En el cuadro de búsqueda, escriba **OWIN**. **Clase de inicio OWIN** aparece como selección. Selecciónela y asigne un nombre a la clase **Startup.cs**.
1. En **Startup.cs**, reemplace el código de la clase **Startup** por el código del mismo archivo de la aplicación de ejemplo. De nuevo, tenga en cuenta que la definición cambia de **clase de inicio pública** a **clase parcial de inicio pública**.
1. En la carpeta **Modelos**, agregue una nueva clase llamada **MsGraphUser.cs**. Reemplace la implementación por el contenido del archivo del mismo nombre del ejemplo.
1. Agregue una nueva instancia de **Controlador de MVC 5: en blanco** llamada **AccountController**. Reemplace la implementación por el contenido del archivo del mismo nombre del ejemplo.
1. Agregue una nueva instancia de **Controlador de MVC 5: en blanco** llamada **UserController**. Reemplace la implementación por el contenido del archivo del mismo nombre del ejemplo.
1. Agregue una nueva instancia de **Controlador de Web API 2 – en blanco** llamada **SyncController**. Reemplace la implementación por el contenido del archivo del mismo nombre del ejemplo.
1. Para la interfaz de usuario, en la carpeta **Views\Account**, agregue tres instancias de **Empty (without model) Views** (Vistas vacías [sin modelo]) llamadas **GrantPermissions**, **Index** y **UserMismatch**. Agregue la llamada **Index** a la carpeta **Views\User**. Reemplace la implementación por el contenido del archivo del mismo nombre del ejemplo.
1. Actualice **Shared\_Layout.cshtml** y **Home\Index.cshtml** para vincular correctamente las diversas vistas.

## <a name="deploy-the-sample-to-azure"></a>Implementación del ejemplo en Azure

Este proyecto tiene proyectos de aplicación web y API web. Para implementarlos en sitios web de Azure, realice los pasos siguientes para cada uno:

1. Cree un sitio web de Azure.
1. Publique la aplicación web y la API web en el sitio web.
1. Actualice los clientes para que llamen al sitio web en lugar de a IIS Express.

### <a name="create-and-publish-dotnet-web-daemon-v2-to-an-azure-website"></a>Creación y publicación de dotnet-web-daemon-v2 en un sitio web de Azure

1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
1. En la esquina superior izquierda, seleccione **Crear un recurso**.
1. Seleccione **Web** > **Aplicación web** y, luego, asigne a su sitio web un nombre. Por ejemplo, lo puede llamar **dotnet-web-daemon-v2-contoso.azurewebsites.net**.
1. Seleccione la información de **Suscripción**, **Grupo de recursos** y **App service plan and location** (Plan y ubicación de App Service). **SO** es **Windows** y **Publicar** es **Código**.
1. Seleccione **Crear** y espere a que se cree el servicio de aplicaciones.
1. Cuando reciba la notificación **Implementación correcta**, seleccione **Ir al recurso** para ir al servicio de aplicaciones recién creado.
1. Después de crear el sitio web, búsquelo en el **Panel** y selecciónelo para abrir la pantalla **Información general** del servicio de aplicaciones.
1. En la pestaña **Información general** del servicio de aplicaciones, seleccione el vínculo **Obtener perfil de publicación** para descargar el perfil de publicación y guárdelo. Puede usar otros mecanismos de implementación, como la implementación desde el control de código fuente.
1. Cambie a Visual Studio y haga lo siguiente:
   1. Vaya al proyecto **dotnet-web-daemon-v2**.
   1. Haga clic con el botón derecho en él en el Explorador de soluciones y seleccione **Publicar**.
   1. Seleccione **Importar perfil** en la barra inferior e importe el perfil de publicación que descargó anteriormente.
1. Seleccione **Configurar**.
1. En la pestaña **Conexión**, actualice la dirección URL de destino para que use "https". Por ejemplo, use `https://dotnet-web-daemon-v2-contoso.azurewebsites.net`. Seleccione **Next** (Siguiente).
1. En la pestaña **Configuración**, asegúrese de que la casilla **Habilitar la autenticación de organización** esté desactivada.
1. Seleccione **Guardar**. Seleccione **Publicar** en la pantalla principal.

Visual Studio publicará el proyecto y abrirá automáticamente un explorador en la dirección URL de este. Si ve la página web predeterminada del proyecto, significa que la publicación se realizó correctamente.

### <a name="update-the-azure-ad-tenant-application-registration-for-dotnet-web-daemon-v2"></a>Actualización del registro de aplicación del inquilino de Azure AD en dotnet-web-daemon-v2

1. Vuelva a <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
1. En el panel izquierdo, seleccione el servicio **Azure Active Directory** y, después, seleccione **Registros de aplicaciones**.
1. Seleccione la aplicación **dotnet-web-daemon-v2**.
1. En la página **Autenticación** de la aplicación, actualice los campos de **dirección URL de cierre de sesión del canal frontal** con la dirección del servicio. Por ejemplo, use `https://dotnet-web-daemon-v2-contoso.azurewebsites.net/Account/EndSession`.
1. En el menú **Personalización de marca**, actualice la **dirección URL de la página principal** con la dirección del servicio. Por ejemplo, use `https://dotnet-web-daemon-v2-contoso.azurewebsites.net`.
1. Guarde la configuración.
1. Agregue la misma dirección URL a la lista de valores del menú **Autenticación** > **URI de redirección**. Si tiene varias direcciones URL de redirección, asegúrese de que haya una nueva entrada que use el URI del servicio de aplicaciones para cada una.

## <a name="clean-up-resources"></a>Limpieza de recursos
Cuando ya no lo necesite, elimine el objeto de aplicación que creó en el paso [Registrar su aplicación](#register-your-application).  Para eliminar la aplicación, siga las instrucciones descritas en [Eliminación de una aplicación creada por usted o por su organización](./howto-remove-app.md#remove-an-application-authored-by-you-or-your-organization).

## <a name="get-help"></a>Obtener ayuda

Utilice [Microsoft Q&A](/answers/products/) para recibir ayuda de la comunidad.
Primero plantee sus preguntas en [Microsoft Q&A](/answers/products/) y examine los problemas existentes para ver si algún usuario ha hecho esa pregunta antes.
No olvide etiquetar las preguntas o comentarios con "azure-ad-adal-deprecation", "azure-ad-msal" y "dotnet-standard".

Si encuentra un error en el ejemplo, abra una incidencia en la [sección de incidencias de GitHub](https://github.com/Azure-Samples/ms-identity-aspnet-daemon-webapp/issues).

Si encuentra un error en MSAL.NET, genere el problema en la sección de [problemas de GitHub para MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues).

Para proporcionar una recomendación, vaya a la [página de UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la creación de aplicaciones de demonio que usan la Plataforma de identidad de Microsoft para acceder a las API Web protegidas:

> [!div class="nextstepaction"]
> [Escenario: Aplicación de demonio que llama a las API web](scenario-daemon-overview.md)