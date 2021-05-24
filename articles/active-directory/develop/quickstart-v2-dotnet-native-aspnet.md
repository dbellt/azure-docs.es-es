---
title: 'Inicio rápido: Llamada a una API web de ASP.NET que está protegida por la plataforma de identidad de Microsoft | Azure'
titleSuffix: Microsoft identity platform
description: En este inicio rápido aprenderá a llamar a una API web de ASP.NET protegida por una plataforma de identidad de Microsoft desde una aplicación del Escritorio de Windows (WPF).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/05/2020
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, identityplatformtop40, scenarios:getting-started, languages:ASP.NET
ms.openlocfilehash: 79557d889e4942c7cb7e4ae849b1da84c12a3712
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108748734"
---
# <a name="quickstart-call-an-aspnet-web-api-thats-protected-by-microsoft-identity-platform"></a>Inicio rápido: Llamada a una instancia de ASP.NET Web API protegida por una plataforma de identidad de Microsoft

En este inicio rápido, descargará y ejecutará un ejemplo de código que muestra cómo proteger una API web de ASP.NET restringiendo el acceso a sus recursos solo a cuentas autorizadas. El ejemplo admite la autorización de cuentas personales de Microsoft y cuentas de cualquier organización de Azure Active Directory (Azure AD).

En el artículo también se usa una aplicación de Windows Presentation Foundation (WPF) para demostrar cómo se solicita un token de acceso para una API web.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Visual Studio 2017 o 2019. Descargue [Visual Studio de forma gratuita](https://www.visualstudio.com/downloads/).

## <a name="clone-or-download-the-sample"></a>Clonación o descarga del ejemplo

Puede obtener el ejemplo de cualquiera de estas dos maneras:

* Clonación desde el shell o la línea de comandos:

   ```console
   git clone https://github.com/AzureADQuickStarts/AppModelv2-NativeClient-DotNet.git
   ```

* [Descarga en un archivo .zip](https://github.com/AzureADQuickStarts/AppModelv2-NativeClient-DotNet/archive/complete.zip).

[!INCLUDE [active-directory-develop-path-length-tip](../../../includes/active-directory-develop-path-length-tip.md)]

## <a name="register-the-web-api-todolistservice"></a>Registro de la API web (TodoListService)

Registre la API web en **Registros de aplicaciones**, en Azure Portal.

1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino en el que desea registrar una aplicación.
1. Busque y seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Registros de aplicaciones** >  y, luego, **Nuevo registro**.
1. Escriba el **Nombre** de la aplicación, por ejemplo `AppModelv2-NativeClient-DotNet-TodoListService`. Los usuarios de la aplicación pueden ver este nombre, el cual se puede cambiar más tarde.
1. En **Supported account types** (Tipos de cuenta admitidos), seleccione **Account in any organizational directory** (Cuentas en cualquier directorio organizativo).
1. Seleccione **Registrar** para crear la aplicación.
1. En la página **Overview** (Información general) de la aplicación, busque el valor de **Application (client) ID** (Id. de cliente de la aplicación) y regístrelo para usarlo más tarde. Lo necesitará para ajustar el archivo de configuración de Visual Studio para este proyecto (es decir, `ClientId` en el archivo *TodoListService\Web.config*).
1. En **Administrar**, seleccione **Exponer una API** > **Agregar un ámbito**. Acepte el URI de identificador de aplicación propuesto (`api://{clientId}`), seleccione **Save and Continue** (Guardar y continuar) y escriba la información siguiente:

    1. Para **Scope name** (Nombre de ámbito), escriba `access_as_user`.
    1. Para **Who can consent** (Quién puede dar el consentimiento), asegúrese de que la opción **Admins and users** (Administradores y usuarios) está seleccionada.
    1. En el cuadro **Admin consent display name** (Nombre para mostrar del consentimiento del administrador), escriba `Access TodoListService as a user` (Acceder a TodoListService como usuario).
    1. En el cuadro **Admin consent description** (Descripción del consentimiento del administrador), escriba `Accesses the TodoListService web API as a user` (Accede a la API web TodoListService como usuario).
    1. En el cuadro **User consent display name** (Nombre para mostrar del consentimiento del usuario), escriba `Access TodoListService as a user` (Acceder a TodoListService como usuario).
    1. En el cuadro **User consent description** (Descripción del consentimiento del usuario), escriba `Accesses the TodoListService web API as a user` (Accede a la API web TodoListService como usuario).
    1. En **State** (Estado), mantenga **Enabled** (Habilitado).
1. Seleccione la opción **Agregar un ámbito**.

### <a name="configure-the-service-project"></a>Configuración del proyecto del servicio

Configure el proyecto de servicio para que coincida con la API web registrada.

1. Abra la solución en Visual Studio y, después, el archivo *Web.config* en la raíz de proyecto de TodoListService.

1. Reemplace el valor del parámetro `ida:ClientId` por el valor del identificador de cliente (identificador de la aplicación) de la aplicación que registró en el portal **Registros de aplicaciones**.

### <a name="add-the-new-scope-to-the-appconfig-file"></a>Incorporación del nuevo ámbito al archivo app.config

Para agregar el nuevo ámbito al archivo *app.config* de TodoListClient, haga lo siguiente:

1. En la carpeta raíz del proyecto TodoListClient, abra el archivo *app. config*.

1. Pegue el identificador de la aplicación que acaba de registrar para el proyecto TodoListService en el parámetro `TodoListServiceScope` y reemplace la cadena `{Enter the Application ID of your TodoListService from the app registration portal}`.

  > [!NOTE]
  > Asegúrese de que el identificador de la aplicación tiene el formato siguiente: `api://{TodoListService-Application-ID}/access_as_user` (donde `{TodoListService-Application-ID}` es el GUID que representa el identificador de la aplicación TodoListService).

## <a name="register-the-web-app-todolistclient"></a>Registro de la aplicación web (TodoListClient)

Registre la aplicación TodoListClient en **Registros de aplicaciones**, en Azure Portal, y, luego, configure el código en el proyecto TodoListClient. Si el cliente y el servidor se consideran la misma aplicación también puede volver a usar la aplicación que se registró en el paso 2. Use la misma aplicación si quiere que los usuarios inicien sesión con su cuenta personal de Microsoft.

### <a name="register-the-app"></a>Registre la aplicación

Para registrar la aplicación TodoListClient, haga lo siguiente:

1. Vaya al portal [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) de la plataforma de identidad de Microsoft para desarrolladores.
1. Seleccione **Nuevo registro**.
1. Cuando aparezca la **página de registro de aplicaciones**, escriba la información de registro de la aplicación:

    1. En la sección **Name** (Nombre), escriba un nombre con sentido para la aplicación que se mostrará a los usuarios (por ejemplo, **NativeClient-DotNet-TodoListClient**).
    1. En **Supported account types** (Tipos de cuenta admitidos), seleccione **Account in any organizational directory** (Cuentas en cualquier directorio organizativo).
    1. Seleccione **Registrar** para crear la aplicación.

   > [!NOTE]
   > En el archivo *app.config* del proyecto TodoListClient, el valor predeterminado de `ida:Tenant` se establece en `common`. Los valores posibles son:
   >
   > - `common`: puede iniciar sesión con una cuenta profesional o educativa o con una cuenta personal de Microsoft (porque seleccionó **Cuentas en cualquier directorio organizativo** en el paso anterior).
   > - `organizations`: puede iniciar sesión con una cuenta profesional o educativa.
   > - `consumers`: solo puede iniciar sesión con una cuenta personal de Microsoft.

1. En la página **Información general** de la aplicación, seleccione **Autenticación** y, luego, haga lo siguiente para agregar una plataforma:

    1. En **Configuraciones de plataforma**, seleccione el botón **Agregar una plataforma**.
    1. En **Mobile and desktop applications** (Aplicaciones móviles y de escritorio), seleccione **Mobile and desktop applications** (Aplicaciones móviles y de escritorio).
    1. En **URI de redirección**, active la casilla `https://login.microsoftonline.com/common/oauth2/nativeclient` .
    1. Seleccione **Configurar**.

1. Seleccione **Permisos de API** y, luego, haga lo siguiente para agregar permisos:

    1. Seleccione el botón **Agregar un permiso**.
    1. Seleccione la pestaña **Mis API**.
    1. En la lista de API, seleccione **AppModelv2-NativeClient-DotNet-TodoListService** o el nombre que especificara para la API web.
    1. Seleccione la casilla del permiso **access_as_user** si aún no lo está. Si es necesario, utilice el cuadro de búsqueda.
    1. Seleccione el botón **Agregar permisos**.

### <a name="configure-your-project"></a>Configurar el proyecto

Configure el proyecto TodoListClient agregando el identificador de aplicación al archivo *app.config*.

1. En el portal **Registros de aplicaciones**, en la página **Overview** (Información general), copie el valor de **Application (client) ID** (Id. de cliente de la aplicación).

1. En el directorio raíz del proyecto TodoListClient, abra el archivo *app.config* y pegue el valor del identificador de la aplicación en el parámetro `ida:ClientId`.

## <a name="run-your-todolistclient-project"></a>Ejecución del proyecto TodoListClient

Inicie sesión para ejecutar el proyecto TodoListClient.

1. Presione F5 para abrir el proyecto TodoListClient. Se abre la página del proyecto.

1. En a parte superior derecha, seleccione **Iniciar sesión** e inicie sesión con las mismas credenciales del registro de la aplicación o con un usuario del mismo directorio.

   Si es la primera vez que inicia sesión, es posible que se le pida que dé su consentimiento a la API web TodoListService.

   Para ayudarle a acceder a la API web TodoListService y a manipular la lista *To-Do*, en el inicio de sesión también se solicita un token de acceso al ámbito *access_as_user*.

## <a name="pre-authorize-your-client-application"></a>Preautorización de la aplicación cliente

Puede permitir que los usuarios de otros directorios accedan a la API web mediante la autorización previa de la aplicación cliente para acceder a la API web. Para ello, agregue el identificador de la aplicación cliente a la lista de aplicaciones previamente autorizadas de la API web. Al agregar un cliente previamente autorizado, permite a los usuarios acceder a la API web sin tener que dar su consentimiento.

1. En el portal **Registros de aplicaciones**, abra las propiedades de la aplicación TodoListService.
1. En la sección **Expose an API** (Exponer una API), en **Authorized client applications** (Aplicaciones cliente autorizadas), seleccione **Add a client application** (Agregar una aplicación cliente).
1. En el cuadro **Client ID** (Id. de cliente), pegue el identificador de la aplicación TodoListClient.
1. En la sección **Authorized scopes** (Ámbitos autorizados), seleccione el ámbito de esta API web: `api://<Application ID>/access_as_user`.
1. Seleccione **Agregar una aplicación**.

### <a name="run-your-project"></a>Ejecución del proyecto

1. Presione <kbd>F5</kbd> para ejecutar el proyecto. Se abre la aplicación TodoListClient.
1. En la parte superior derecha, seleccione **Iniciar sesión** e inicie sesión con un cuenta personal de Microsoft (como *live.com* o *hotmail.com*) o una cuenta profesional o educativa.

## <a name="optional-limit-sign-in-access-to-certain-users"></a>Opcional: limitación del acceso de inicio de sesión a determinados usuarios

De forma predeterminada, cualquier cuenta personal, como *outlook.com* o *live.com*, o profesional o educativa de las organizaciones integradas con Azure AD puede solicitar tokens y acceder a la API web.

Para especificar quién puede iniciar sesión en la aplicación, use una de las siguientes opciones:

### <a name="option-1-limit-access-to-a-single-organization-single-tenant"></a>Opción 1: Limitar el acceso a una sola organización (inquilino único)

Puede limitar el acceso al inicio de sesión de la aplicación para que solo puedan usarlo aquellas cuentas de usuario que forman parte de un solo inquilino de Azure AD, incluidas las cuentas de invitado de ese inquilino. Este es un escenario común para las aplicaciones de línea de negocio.

1. Abra el archivo *App_Start\Startup.Auth* y cambie el valor del punto de conexión de metadatos que se pasa al `OpenIdConnectSecurityTokenProvider` por `https://login.microsoftonline.com/{Tenant ID}/v2.0/.well-known/openid-configuration`. También puede usar el nombre del inquilino, como `contoso.onmicrosoft.com`.
1. En el mismo archivo, establezca la propiedad `ValidIssuer` de `TokenValidationParameters` en `https://sts.windows.net/{Tenant ID}/`, y el argumento `ValidateIssuer` en `true`.

### <a name="option-2-use-a-custom-method-to-validate-issuers"></a>Opción 2: Usar un método personalizado para validar emisores

Puede implementar un método personalizado para validar los emisores con el parámetro `IssuerValidator`. Para más información sobre este parámetro, consulte la [clase TokenValidationParameters](/dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters).

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el escenario de API web protegida que admite la plataforma de identidad de Microsoft.
> [!div class="nextstepaction"]
> [escenario con una API web protegida](scenario-protected-web-api-overview.md)
