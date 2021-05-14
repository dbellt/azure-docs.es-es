---
title: 'Inicio rápido: Obtención de un token y llamada a Microsoft Graph en una aplicación de consola | Azure'
titleSuffix: Microsoft identity platform
description: En esta guía de inicio rápido, obtendrá información sobre cómo una aplicación de ejemplo de .NET Core puede usar el flujo de credenciales de cliente para obtener un token y llamar a Microsoft Graph.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/05/2020
ms.author: jmprieur
ms.reviewer: marsma
ms.custom: devx-track-csharp, aaddev, identityplatformtop40, scenarios:getting-started, languages:aspnet-core
ms.openlocfilehash: a31bf345f523eea940be5d56495890e8ab5c6dbd
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107861650"
---
# <a name="quickstart-get-a-token-and-call-the-microsoft-graph-api-by-using-a-console-apps-identity"></a>Inicio rápido: Adquisición de un token y llamada a Microsoft Graph API mediante una identidad de aplicación de consola

En este inicio rápido descargará y ejecutará un código de ejemplo que muestra cómo una aplicación de consola de .NET Core puede obtener un token de acceso para llamar a Microsoft Graph API y mostrar una [lista de usuarios](/graph/api/user-list) del directorio. En el ejemplo de código también se muestra cómo se puede ejecutar un trabajo o un servicio de Windows con una identidad de aplicación, en lugar de la identidad de un usuario. La aplicación de consola de ejemplo de este inicio rápido también es una aplicación demonio, por lo que es una aplicación cliente confidencial.

> [!div renderon="docs"]
> El diagrama siguiente muestra cómo funciona la aplicación de ejemplo:
>
> ![Diagrama que muestra el funcionamiento de la aplicación de ejemplo que se ha generado en este inicio rápido.](media/quickstart-v2-netcore-daemon/netcore-daemon-intro.svg)
>

## <a name="prerequisites"></a>Requisitos previos

Este inicio rápido requiere el [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download), pero también funcionará con el SDK de .NET 5.0.

> [!div renderon="docs"]
> ## <a name="register-and-download-the-app"></a>Registro y descarga de la aplicación

> [!div renderon="docs" class="sxs-lookup"]
>
> Tiene dos opciones para empezar a crear la aplicación: usar la configuración automática o la manual.
>
> ### <a name="automatic-configuration"></a>Configuración automática
>
> Si desea registrar y configurar automáticamente la aplicación y, a continuación, descargar el ejemplo de código, siga estos pasos:
>
> 1. Vaya a la <a href="https://portal.azure.com/?Microsoft_AAD_RegisteredApps=true#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/DotNetCoreDaemonQuickstartPage/sourceType/docs" target="_blank">página de Azure Portal para el registro de aplicaciones</a>.
> 1. Escriba un nombre para la aplicación y seleccione **Registrar**.
> 1. Siga las instrucciones para descargar y configurar automáticamente la nueva aplicación con un clic.
>
> ### <a name="manual-configuration"></a>Configuración manual
>
> Si desea configurar manualmente la aplicación y el ejemplo de código, siga estos pasos.
>
> [!div renderon="docs"]
> #### <a name="step-1-register-your-application"></a>Paso 1: Registrar su aplicación
> Para registrar la aplicación y agregar la información de registro de la aplicación a la solución de forma manual, siga estos pasos:
>
> 1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</span></a>.
> 1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino en el que desea registrar la aplicación.
> 1. Busque y seleccione **Azure Active Directory**.
> 1. En **Administrar**, seleccione **Registros de aplicaciones** >  y, luego, **Nuevo registro**.
> 1. Escriba un **nombre** para la aplicación. Por ejemplo, escriba **Daemon-console**. Los usuarios de la aplicación verán este nombre, que puede cambiar más tarde.
> 1. Seleccione **Registrar** para crear la aplicación.
> 1. En **Administrar**, seleccione **Certificados y secretos**.
> 1. En **Secretos de cliente**, seleccione **Nuevo secreto de cliente**, escriba un nombre y seleccione **Agregar**. Grabe el valor del secreto en una ubicación segura para usarlo en un paso posterior.
> 1. En **Administrar**, seleccione **Permisos de API** > **Add a permission** (Agregar un permiso). Seleccione **Microsoft Graph**.
> 1. Seleccione **Permisos de aplicación**.
> 1. En el nodo **Usuario**, seleccione **User.Read.All** y, luego, **Agregar permisos**.

> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="download-and-configure-your-quickstart-app"></a>Descarga y configuración de la aplicación de inicio rápido
>
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Paso 1: Configuración de la aplicación en Azure Portal
> Para que el ejemplo de código de este inicio rápido funcione, cree un secreto de cliente y agregue el permiso de aplicación **User.Read.All** de Graph API.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Realizar estos cambios por mí]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Ya configurada](media/quickstart-v2-netcore-daemon/green-check.png) La aplicación está configurada con estos atributos.

#### <a name="step-2-download-your-visual-studio-project"></a>Paso 2: Descarga del proyecto de Visual Studio

> [!div renderon="docs"]
> [Descargue el proyecto de Visual Studio](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/archive/master.zip)
>
> Puede ejecutar el proyecto proporcionado en Visual Studio o en Visual Studio para Mac.


> [!div class="sxs-lookup" renderon="portal"]
> Ejecute el proyecto con Visual Studio 2019.
> [!div class="sxs-lookup" renderon="portal" id="autoupdate" class="nextstepaction"]
> [Descargar el código de ejemplo](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/archive/master.zip)

[!INCLUDE [active-directory-develop-path-length-tip](../../../includes/active-directory-develop-path-length-tip.md)]

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
> #### <a name="step-3-configure-your-visual-studio-project"></a>Paso 3: Configuración del proyecto de Visual Studio
>
> 1. Extraiga el archivo. zip en una carpeta local que esté próxima a la raíz del disco. Por ejemplo, en *C:\Azure-Samples*.
>
>    Se recomienda extraer el archivo en un directorio próximo a la raíz de la unidad para evitar errores provocados por las limitaciones de longitud de la ruta de acceso en Windows.
>
> 1. Abra la solución en Visual Studio: *1-Call-MSGraph\daemon-console.sln* (opcional).
> 1. En *appsettings.json*, reemplace los valores de `Tenant`, `ClientId` y `ClientSecret`:
>
>    ```json
>    "Tenant": "Enter_the_Tenant_Id_Here",
>    "ClientId": "Enter_the_Application_Id_Here",
>    "ClientSecret": "Enter_the_Client_Secret_Here"
>    ```
>    En el código:
>    - `Enter_the_Application_Id_Here` es el identificador de aplicación (cliente) de la aplicación que se registró.
        Para buscar los valores del identificador de aplicación (cliente) y del identificador de directorio (inquilino), vaya a la página **Información general** de Azure Portal.
>    - Reemplace `Enter_the_Tenant_Id_Here` por el identificador de inquilino o el nombre del inquilino (por ejemplo, `contoso.microsoft.com`).
>    - Reemplace `Enter_the_Client_Secret_Here` por el secreto de cliente que creó en el paso 1.
    Para generar una nueva clave, vaya a la página **Certificados y secretos**.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-admin-consent"></a>Paso 3: Consentimiento de administrador

> [!div renderon="docs"]
> #### <a name="step-4-admin-consent"></a>Paso 4: Consentimiento de administrador

Si intenta ejecutar la aplicación en este momento, recibirá un error *HTTP 403 (Prohibido)* : "No tiene privilegios suficientes para completar la operación". Este error sucede porque cualquier permiso de solo aplicación requiere que un administrador global del directorio debe otorgue su consentimiento a la aplicación. Seleccione una de las opciones siguientes según el rol.

##### <a name="global-tenant-administrator"></a>Administrador de inquilinos global

> [!div renderon="docs"]
> Si es un administrador de inquilinos global, vaya a **Aplicaciones empresariales** en Azure Portal. Seleccione el registro de la aplicación y seleccione **Permisos** en la sección **Seguridad** del panel izquierdo. Seleccione el botón grande denominado **Conceder consentimiento de administrador para {Tenant Name}** , donde **{Tenant Name}** es el nombre de su directorio.

> [!div renderon="portal" class="sxs-lookup"]
> Si es un administrador global, vaya a la página **Permisos de API** y seleccione **Conceder consentimiento de administrador para _escribir_aquí_el_nombre_del_inquilino**.
> > [!div id="apipermissionspage"]
> > [Ir a la página Permisos de API]()

##### <a name="standard-user"></a>Usuario estándar

Si es usuario estándar de su inquilino, pídale a un administrador global que conceda consentimiento del administrador para su aplicación. Para ello, proporcione la siguiente dirección URL a su administrador:

```url
https://login.microsoftonline.com/Enter_the_Tenant_Id_Here/adminconsent?client_id=Enter_the_Application_Id_Here
```

> [!div renderon="docs"]
> En esa dirección URL:
> * Reemplace `Enter_the_Tenant_Id_Here` por el identificador de inquilino o el nombre del inquilino (por ejemplo, `contoso.microsoft.com`).
> * `Enter_the_Application_Id_Here` es el identificador de aplicación (cliente) de la aplicación que se registró.

Es posible que después de conceder consentimiento a la aplicación mediante la URL anterior, aparezca el error "AADSTS50011: no hay direcciones de respuesta registradas para la aplicación". Este error se produce porque esta aplicación y la dirección URL no tienen un URI de redirección. Puede pasarla por alto.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-4-run-the-application"></a>Paso 4: Ejecución de la aplicación

> [!div renderon="docs"]
> #### <a name="step-5-run-the-application"></a>Paso 5: Ejecución de la aplicación

Si utiliza Visual Studio o Visual Studio para Mac, presione **F5** para ejecutar la aplicación. De lo contrario, ejecute la aplicación mediante el símbolo del sistema, la consola o el terminal:

```dotnetcli
cd {ProjectFolder}\1-Call-MSGraph\daemon-console
dotnet run
```
En el código:
* `{ProjectFolder}` es la carpeta donde extrajo el archivo zip. Un ejemplo es `C:\Azure-Samples\active-directory-dotnetcore-daemon-v2`.

Como resultado, verá una lista de usuarios en Azure Active Directory.

Esta aplicación de inicio rápido usa un secreto de cliente para identificarse como un cliente confidencial. El secreto de cliente se agrega como un archivo de texto sin formato a los archivos de proyecto. Por motivos de seguridad, se recomienda que use un certificado en lugar de un secreto de cliente antes de considerar la aplicación como de producción. Para más información sobre cómo usar un certificado, consulte [estas instrucciones](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/#variation-daemon-application-using-client-credentials-with-certificates) en el repositorio de GitHub para este ejemplo.

## <a name="more-information"></a>Más información
En esta sección, se proporciona una introducción al código necesario para el inicio de sesión de usuarios. Esta introducción puede ser útil para comprender cómo funciona el código, cuáles son los argumentos principales y cómo agregar el inicio de sesión a una aplicación de consola de .NET Core existente.

> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="how-the-sample-works"></a>Funcionamiento del ejemplo
>
> ![Diagrama que muestra el funcionamiento de la aplicación de ejemplo que se ha generado en este inicio rápido.](media/quickstart-v2-netcore-daemon/netcore-daemon-intro.svg)

### <a name="msalnet"></a>MSAL.NET

Microsoft Authentication Library (MSAL, en el paquete [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)) es la biblioteca que se usa para el inicio de sesión de los usuarios y para solicitar tokens para acceder a una API protegida por la plataforma de identidad de Microsoft. En este inicio rápido se solicitan tokens mediante la propia identidad de la aplicación, en lugar de permisos delegados. El flujo de autenticación usado en este caso se conoce como [flujo de OAuth de credenciales de cliente](v2-oauth2-client-creds-grant-flow.md). Para más información sobre cómo usar MSAL.NET con el flujo de credenciales de cliente, consulte [este artículo](https://aka.ms/msal-net-client-credentials).

 Puede instalar MSAL.NET mediante la ejecución del siguiente comando en la Consola del Administrador de paquetes de Visual Studio:

```dotnetcli
dotnet add package Microsoft.Identity.Client
```

### <a name="msal-initialization"></a>Inicialización de MSAL

Puede agregar la referencia de MSAL con el código siguiente:

```csharp
using Microsoft.Identity.Client;
```

A continuación, realice la inicialización de MSAL con el siguiente código:

```csharp
IConfidentialClientApplication app;
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
                                          .WithClientSecret(config.ClientSecret)
                                          .WithAuthority(new Uri(config.Authority))
                                          .Build();
```

 | Elemento | Descripción |
 |---------|---------|
 | `config.ClientSecret` | Es el secreto de cliente creado para la aplicación en Azure Portal. |
 | `config.ClientId` | Es el identificador de aplicación (cliente) de la aplicación registrada en Azure Portal. Puede encontrar este valor en la página **Información general** de la aplicación en Azure Portal. |
 | `config.Authority`    | (Opcional) El punto de conexión del servicio de token de seguridad (STS) para que el usuario se autentique. Normalmente `https://login.microsoftonline.com/{tenant}` en la nube pública, donde `{tenant}` es el nombre o el identificador del inquilino.|

Para más información, consulte la [documentación de referencia de `ConfidentialClientApplication`](/dotnet/api/microsoft.identity.client.iconfidentialclientapplication).

### <a name="requesting-tokens"></a>Solicitud de tokens

Para solicitar un token mediante la identidad de la aplicación, use el método `AcquireTokenForClient`:

```csharp
result = await app.AcquireTokenForClient(scopes)
                  .ExecuteAsync();
```

|Elemento| Descripción |
|---------|---------|
| `scopes` | Contiene los ámbitos solicitados. Para clientes confidenciales, este valor debe tener un formato similar a `{Application ID URI}/.default`. Este formato indica que los ámbitos solicitados son los que están definidos estáticamente en el conjunto de objetos de la aplicación en Azure Portal. Para Microsoft Graph, `{Application ID URI}` apunta a `https://graph.microsoft.com`. Con API web personalizadas, `{Application ID URI}` se define en Azure Portal, en **Registro de aplicaciones (versión preliminar)**  > **Exponer una API**. |

Para más información, consulte la [documentación de referencia de `AcquireTokenForClient`](/dotnet/api/microsoft.identity.client.confidentialclientapplication.acquiretokenforclient).

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las aplicaciones demonio, consulte la información general del escenario:

> [!div class="nextstepaction"]
> [Aplicación demonio que llama a las API web](scenario-daemon-overview.md)
