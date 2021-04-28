---
title: Administración de recursos con Microsoft Graph
titleSuffix: Azure AD B2C
description: Procedimiento para administrar los recursos de un inquilino de Azure AD B2C llamando a Microsoft Graph API y mediante una identidad de aplicación para automatizar el proceso.
services: B2C
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/19/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: cf62330fd677dc978c8f25a81c6a1e5bfbb612ac
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107717610"
---
# <a name="manage-azure-ad-b2c-with-microsoft-graph"></a>Administración de Azure AD B2C con Microsoft Graph

Microsoft Graph permite administrar recursos en el directorio de Azure AD B2C. Se admiten las siguientes operaciones de Microsoft Graph API para la administración de recursos de Azure AD B2C, incluidos usuarios, proveedores de identidades, flujos de usuarios, directivas personalizadas y claves de directivas. Cada vínculo de las secciones siguientes se dirige a la página correspondiente dentro de la referencia de Microsoft Graph API para esa operación. 

## <a name="prerequisites"></a>Requisitos previos

Para usar MS Graph API e interactuar con los recursos de su inquilino de Azure AD B2C, necesita un registro de aplicación que conceda permisos para hacerlo. Siga los pasos del artículo [Administración de Azure AD B2C con Microsoft Graph](microsoft-graph-get-started.md) para crear un registro de aplicación que pueda usar la aplicación de administración. 

## <a name="user-management"></a>Administración de usuarios

- [Enumerar usuarios](/graph/api/user-list)
- [Crear usuario](/graph/api/user-post-users)
- [Obtener un usuario](/graph/api/user-get)
- [Actualizar usuario](/graph/api/user-update)
- [Eliminar un usuario](/graph/api/user-delete)

## <a name="user-phone-number-management-beta"></a>Administración de números de teléfono de usuarios (versión beta)

Número de teléfono que un usuario puede emplear para iniciar sesión mediante [llamadas de voz o SMS](identity-provider-local.md#phone-sign-in-preview) o [autenticación multifactor](multi-factor-authentication.md). Para obtener más información, vea [API de métodos de autenticación de Azure AD](/graph/api/resources/phoneauthenticationmethod).

- [Add (Agregar)](/graph/api/authentication-post-phonemethods)
- [Lista](/graph/api/authentication-list-phonemethods)
- [Get](/graph/api/phoneauthenticationmethod-get)
- [Actualizar](/graph/api/phoneauthenticationmethod-update)
- [Eliminar](/graph/api/phoneauthenticationmethod-delete)

Tenga en cuenta que la operación [Enumerar](/graph/api/authentication-list-phonemethods) solo devuelve los números de teléfono habilitados. El siguiente número de teléfono debe estar habilitado para su uso con las operaciones Enumerar. 

![Habilitación de inicio de sesión telefónico](./media/microsoft-graph-operations/enable-phone-sign-in.png)

> [!NOTE]
> En la versión beta actual, esta API solo funciona si el número de teléfono está almacenado con un espacio entre el código de país y el número de teléfono. El servicio Azure AD B2C no agrega actualmente este espacio de manera predeterminada.

## <a name="self-service-password-reset-email-address-beta"></a>Dirección de correo electrónico de autoservicio de restablecimiento de contraseña (versión beta)

Dirección de correo electrónico que una [cuenta de inicio de sesión mediante nombre de usuario](identity-provider-local.md#username-sign-in) puede usar para restablecer la contraseña. Para obtener más información, vea [API de métodos de autenticación de Azure AD](/graph/api/resources/emailauthenticationmethod).

- [Add (Agregar)](/graph/api/emailauthenticationmethod-post)
- [Lista](/graph/api/emailauthenticationmethod-list)
- [Get](/graph/api/emailauthenticationmethod-get)
- [Actualizar](/graph/api/emailauthenticationmethod-update)
- [Eliminar](/graph/api/emailauthenticationmethod-delete)

## <a name="identity-providers"></a>Proveedores de identidades

Administre los [proveedores de identidades](add-identity-provider.md) disponibles para los flujos de usuario en el inquilino de Azure AD B2C.

- [Enumerar proveedores de identidades en el inquilino de Azure AD B2C](/graph/api/identityprovider-list)
- [Crear proveedores de identidades](/graph/api/identityprovider-post-identityproviders)
- [Obtener proveedores de identidades](/graph/api/identityprovider-get)
- [Actualizar proveedores de identidades](/graph/api/identityprovider-update)
- [Eliminar proveedores de identidades](/graph/api/identityprovider-delete)

## <a name="user-flow"></a>Flujo de usuario

Configure las directivas predefinidas para el registro, el inicio de sesión, el registro y el inicio de sesión combinados, el restablecimiento de contraseña y la actualización del perfil.

- [Enumerar flujos de usuario](/graph/api/identitycontainer-list-b2cuserflows)
- [Crear flujos de usuario](/graph/api/identitycontainer-post-b2cuserflows)
- [Obtener flujos de usuario](/graph/api/b2cidentityuserflow-get)
- [Eliminar flujos de usuario](/graph/api/b2cidentityuserflow-delete)

## <a name="user-flow-authentication-methods-beta"></a>Métodos de autenticación de flujos de usuario (versión beta)

Elija un mecanismo para permitir a los usuarios registrarse mediante cuentas locales. Las cuentas locales son aquellas en las que Azure AD realiza la aserción de identidad. Para obtener más información, vea [tipo de recurso b2cAuthenticationMethodsPolicy](/graph/api/resources/b2cauthenticationmethodspolicy).

- [Get](/graph/api/b2cauthenticationmethodspolicy-get)
- [Actualizar](/graph/api/b2cauthenticationmethodspolicy-update)

## <a name="custom-policies"></a>Directivas personalizadas

Las siguientes operaciones permiten administrar las directivas del marco de confianza de Azure AD B2C, conocidas como [directivas personalizadas](custom-policy-overview.md).

- [Enumerar directivas de marco de confianza](/graph/api/trustframework-list-trustframeworkpolicies)
- [Crear directivas de marco de confianza](/graph/api/trustframework-post-trustframeworkpolicy)
- [Obtener directivas de marco de confianza](/graph/api/trustframeworkpolicy-get)
- [Actualizar o crear directivas de marco de confianza](/graph/api/trustframework-put-trustframeworkpolicy)
- [Eliminar directivas de marco de confianza](/graph/api/trustframeworkpolicy-delete)

## <a name="policy-keys"></a>Claves de directiva

Identity Experience Framework almacena los secretos a los que se hace referencia en una directiva personalizada para establecer la confianza entre los componentes. Estos secretos pueden ser claves/valores simétricos o asimétricos. En Azure Portal, estas entidades se muestran como **claves de directiva**.

El recurso de nivel superior para las claves de directiva de Microsoft Graph API es el [conjunto de claves del marco de confianza](/graph/api/resources/trustframeworkkeyset). Cada **conjunto de claves** contiene al menos una **clave**. Para crear una clave, primero debe crear un conjunto de claves vacío y, a continuación, generar una clave en el conjunto de claves. Puede crear un secreto manual, cargar un certificado o una clave PKCS12. La clave puede ser un secreto generado, una cadena (como el secreto de aplicación de Facebook) o un certificado que cargue. Si un conjunto de claves tiene varias claves, solo una de las claves está activa.

### <a name="trust-framework-policy-keyset"></a>Conjunto de claves de directiva de marco de confianza

- [Enumerar conjuntos de claves de confianza](/graph/api/trustframework-list-keysets)
- [Crear conjuntos de claves de confianza](/graph/api/trustframework-post-keysets)
- [Obtener conjuntos de claves de confianza](/graph/api/trustframeworkkeyset-get)
- [Actualizar conjuntos de claves de confianza](/graph/api/trustframeworkkeyset-update)
- [Eliminar conjuntos de claves de confianza](/graph/api/trustframeworkkeyset-delete)

### <a name="trust-framework-policy-key"></a>Clave de directiva de marco de confianza

- [Obtener clave activa](/graph/api/trustframeworkkeyset-getactivekey)
- [Generar clave](/graph/api/trustframeworkkeyset-generatekey)
- [Cargar secreto](/graph/api/trustframeworkkeyset-uploadsecret)
- [Cargar certificado X.509](/graph/api/trustframeworkkeyset-uploadcertificate)
- [Cargar certificado PKCS12](/graph/api/trustframeworkkeyset-uploadpkcs12)

## <a name="applications"></a>APLICACIONES

- [Lista de aplicaciones](/graph/api/application-list)
- [Crear aplicación](/graph/api/resources/application)
- [Actualizar aplicación](/graph/api/application-update)
- [Tipo de recurso servicePrincipal](/graph/api/resources/serviceprincipal)
- [Tipo de recurso oAuth2PermissionGrant](/graph/api/resources/oauth2permissiongrant)
- [Eliminar aplicación](/graph/api/application-delete)

## <a name="application-extension-properties"></a>Propiedades de extensión de aplicación

- [Enumerar propiedades de extensión](/graph/api/application-list-extensionproperty)

Azure AD B2C proporciona un directorio que puede contener 100 atributos personalizados por usuario. En el caso de los flujos de usuarios, estas propiedades de extensión se [administran mediante Azure Portal](user-flow-custom-attributes.md). En el caso de las directivas personalizadas, Azure AD B2C crea la propiedad automáticamente la primera vez que la directiva escribe un valor en la propiedad de extensión.

## <a name="audit-logs"></a>Registros de auditoría

- [Enumerar auditorías de auditorio](/graph/api/directoryaudit-list)

Para más información sobre cómo acceder a los registros de auditoría de Azure AD B2C, consulte [Acceso a los registros de auditoría de Azure AD B2C](view-audit-logs.md).

## <a name="conditional-access"></a>Acceso condicional

- [Enumeración de todas las directivas de acceso condicional](/graph/api/conditionalaccessroot-list-policies?tabs=http)
- [Lectura de propiedades y relaciones de una directiva de acceso condicional](/graph/api/conditionalaccesspolicy-get)
- [Creación de una directiva de acceso condicional nueva](/graph/api/resources/application)
- [Actualización de una directiva de acceso condicional](/graph/api/conditionalaccesspolicy-update)
- [Eliminación de una directiva de acceso condicional](/graph/api/conditionalaccesspolicy-delete)

## <a name="code-sample-how-to-programmatically-manage-user-accounts"></a>Ejemplo de código: Administración de cuentas de usuario mediante programación

Este ejemplo de código es una aplicación de consola .NET Core que usa el [SDK de Microsoft Graph](/graph/sdks/sdks-overview) para interactuar con Microsoft Graph API. En su código se muestra cómo llamar a la API para administrar usuarios mediante programación en un inquilino de Azure AD B2C.
Puede [descargar el archivo de ejemplo](https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management/archive/master.zip) (*.zip), [examinar el repositorio](https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management) de GitHub o clonar el repositorio:

```cmd
git clone https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management.git
```

Después de obtener el ejemplo de código, configúrelo para su entorno y, a continuación, compile el proyecto:

1. Abra el proyecto en [Visual Studio](https://visualstudio.microsoft.com) o en [Visual Studio Code](https://code.visualstudio.com).
1. Abra `src/appsettings.json`.
1. En la sección `appSettings`, reemplace `your-b2c-tenant` por el nombre de su inquilino, y `Application (client) ID` y `Client secret` por los valores para el registro de la aplicación de administración. Para más información, consulte [Registro de una aplicación de Microsoft Graph](microsoft-graph-get-started.md).
1. Abra una ventana de consola en el clon local del repositorio, cambie al directorio `src` y, a continuación, compile el proyecto:

    ```console
    cd src
    dotnet build
    ```
    
1. Ejecute la aplicación con el comando `dotnet`:

    ```console
    dotnet bin/Debug/netcoreapp3.1/b2c-ms-graph.dll
    ```

La aplicación muestra una lista de los comandos que se pueden ejecutar. Por ejemplo, obtener todos los usuarios, obtener un solo usuario, eliminar un usuario, actualizar la contraseña de un usuario y realizar una importación en bloque.

### <a name="code-discussion"></a>Discusión de código

En el código de ejemplo se usa el [SDK de Microsoft Graph](/graph/sdks/sdks-overview), que se ha diseñado para simplificar la creación de aplicaciones de alta calidad, eficientes y resistentes que tienen acceso a Microsoft Graph.

Cualquier solicitud a Microsoft Graph API requiere un token de acceso para la autenticación. La solución usa el paquete de NuGet [Microsoft.Graph.Auth](https://www.nuget.org/packages/Microsoft.Graph.Auth/) que proporciona un contenedor basado en escenario de autenticación de la biblioteca de autenticación de Microsoft (MSAL) para su uso con el SDK de Microsoft Graph.

El método `RunAsync` en el archivo _Program.cs_:

1. Lee la configuración de la aplicación del archivo _appsettings.json_.
1. Inicializa el proveedor de autenticación mediante el flujo de [concesión de credenciales de cliente de OAuth 2.0](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md). Con el flujo de concesión de credenciales de cliente, la aplicación puede obtener un token de acceso para llamar a Microsoft Graph API.
1. Configura el cliente del servicio de Microsoft Graph con el proveedor de autenticación:

    ```csharp
    // Read application settings from appsettings.json (tenant ID, app ID, client secret, etc.)
    AppSettings config = AppSettingsFile.ReadFromJsonFile();

    // Initialize the client credential auth provider
    IConfidentialClientApplication confidentialClientApplication = ConfidentialClientApplicationBuilder
        .Create(config.AppId)
        .WithTenantId(config.TenantId)
        .WithClientSecret(config.ClientSecret)
        .Build();
    ClientCredentialProvider authProvider = new ClientCredentialProvider(confidentialClientApplication);

    // Set up the Microsoft Graph service client with client credentials
    GraphServiceClient graphClient = new GraphServiceClient(authProvider);
    ```

A continuación, se usa la instancia inicializada de *GraphServiceClient* en _GraphService.cs_ para realizar las operaciones de administración de usuarios. Por ejemplo, obtener una lista de las cuentas de usuario en el inquilino:

```csharp
public static async Task ListUsers(GraphServiceClient graphClient)
{
    Console.WriteLine("Getting list of users...");

    // Get all users (one page)
    var result = await graphClient.Users
        .Request()
        .Select(e => new
        {
            e.DisplayName,
            e.Id,
            e.Identities
        })
        .GetAsync();

    foreach (var user in result.CurrentPage)
    {
        Console.WriteLine(JsonConvert.SerializeObject(user));
    }
}
```

El documento [Realizar llamadas API con los SDK de Microsoft Graph](/graph/sdks/create-requests) incluye información sobre cómo leer y escribir información de Microsoft Graph, usar `$select` para controlar las propiedades devueltas, proporcionar parámetros de consulta personalizados y usar los parámetros de consulta `$filter` y `$orderBy`.

<!-- LINK -->

[graph-objectIdentity]: /graph/api/resources/objectidentity
[graph-user]: (https://docs.microsoft.com/graph/api/resources/user)
