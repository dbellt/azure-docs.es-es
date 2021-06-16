---
title: 'Inicio rápido: Llamada a Microsoft Graph desde un demonio de Java | Azure'
titleSuffix: Microsoft identity platform
description: En este inicio rápido, obtendrá información sobre cómo una aplicación de Java puede obtener un token de acceso y llamar a una API protegida por un punto de conexión de la Plataforma de identidad de Microsoft mediante la propia identidad de la aplicación.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 01/22/2021
ms.author: mtillman
ms.custom: aaddev, scenarios:getting-started, languages:Java, devx-track-java
ms.openlocfilehash: 866fc8c0a44f9380dd38227d2b5a2d2b332ec8b4
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112075464"
---
# <a name="quickstart-acquire-a-token-and-call-microsoft-graph-api-from-a-java-console-app-using-apps-identity"></a>Inicio rápido: Adquisición de un token y llamada a Microsoft Graph API desde una aplicación de consola de Java mediante la identidad de la aplicación

En este inicio rápido descargará y ejecutará un código de ejemplo que muestra cómo puede obtener una aplicación de Java un token de acceso mediante la identidad de la aplicación para llamar a Microsoft Graph API y mostrar una [lista de usuarios](/graph/api/user-list) en el directorio. En el ejemplo de código se muestra cómo se puede ejecutar un trabajo desatendido o un servicio de Windows con una identidad de aplicación, en lugar de la identidad de un usuario. 

> [!div renderon="docs"]
> ![Muestra cómo funciona la aplicación de muestra generada en este inicio rápido](media/quickstart-v2-java-daemon/java-console-daemon.svg).

## <a name="prerequisites"></a>Prerrequisitos

Para ejecutar esta muestra, necesita:

- [Kit de desarrollo de Java (JDK)](https://openjdk.java.net/) 8 o posterior
- [Maven](https://maven.apache.org/)

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-app"></a>Registro y descarga de la aplicación de inicio rápido

> [!div renderon="docs" class="sxs-lookup"]
>
> Tiene dos opciones para comenzar con la aplicación de inicio rápido: Rápido (opción 1) y Manual (opción 2)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Opción 1: registrar y configurar de modo automático la aplicación y, a continuación, descargar el código de ejemplo
>
> 1. Vaya a la experiencia de inicio rápido <a href="https://portal.azure.com/?Microsoft_AAD_RegisteredApps=true#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/JavaDaemonQuickstartPage/sourceType/docs" target="_blank">Azure Portal: Registros de aplicaciones</a>.
> 1. Escriba un nombre para la aplicación y seleccione **Registrar**.
> 1. Siga las instrucciones para descargar y configurar automáticamente la nueva aplicación con un solo clic.
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>Opción 2: registrar y configurar manualmente la aplicación y el código de ejemplo

> [!div renderon="docs"]
> #### <a name="step-1-register-your-application"></a>Paso 1: Registrar su aplicación
> Para registrar la aplicación y agregar la información de registro de la aplicación a la solución de forma manual, siga estos pasos:
>
> 1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
> 1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino en el que desea registrar una aplicación.
> 1. Busque y seleccione **Azure Active Directory**.
> 1. En **Administrar**, seleccione **Registros de aplicaciones** >  y, luego, **Nuevo registro**.
> 1. Escriba el **Nombre** de la aplicación, por ejemplo `Daemon-console`. Los usuarios de la aplicación pueden ver este nombre, el cual se puede cambiar más tarde.
> 1. Seleccione **Registrar**.
> 1. En **Administrar**, seleccione **Certificados y secretos**.
> 1. En **Secretos de cliente**, seleccione **Nuevo secreto de cliente**, escriba un nombre y seleccione **Agregar**. Grabe el valor del secreto en una ubicación segura para usarlo en un paso posterior.
> 1. En **Administrar**, seleccione **Permisos de API** > **Add a permission** (Agregar un permiso). Seleccione **Microsoft Graph**.
> 1. Seleccione **Permisos de aplicación**.
> 1. En el nodo **Usuario**, seleccione **User.Read.All** y, luego, **Agregar permisos**.

> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="download-and-configure-the-quickstart-app"></a>Descarga y configuración de la aplicación de inicio rápido
>
> #### <a name="step-1-configure-the-application-in-azure-portal"></a>Paso 1: Configuración de la aplicación en Azure Portal
> Para que el ejemplo de código de esta guía de inicio rápido funcione, debe crear un secreto de cliente y agregar el permiso de aplicación **User.Read.All** de Graph API.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Realizar estos cambios por mí]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Ya configurada](media/quickstart-v2-netcore-daemon/green-check.png) La aplicación está configurada con estos atributos.

#### <a name="step-2-download-the-java-project"></a>Paso 2: Descarga del proyecto de Java

> [!div renderon="docs"]
> [Descargar el proyecto de demonio de Java](https://github.com/Azure-Samples/ms-identity-java-daemon/archive/master.zip)

> [!div renderon="portal" id="autoupdate" class="sxs-lookup nextstepaction"]
> [Descargar el código de ejemplo](https://github.com/Azure-Samples/ms-identity-java-daemon/archive/master.zip)

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`


> [!div renderon="docs"]
> #### <a name="step-3-configure-the-java-project"></a>Paso 3: Configuración del proyecto de Java
>
> 1. Extraiga el archivo ZIP en una carpeta local próxima a la raíz del disco, por ejemplo, *C:\Azure-Samples*.
> 1. Vaya a la subcarpeta **msal-client-credential-secret**.
> 1. Edite el archivo *src\main\resources\application.properties* y reemplace los valores de los campos `AUTHORITY`, `CLIENT_ID` y `SECRET` por el siguiente fragmento de código:
>
>    ```
>    AUTHORITY=https://login.microsoftonline.com/Enter_the_Tenant_Id_Here/
>    CLIENT_ID=Enter_the_Application_Id_Here
>    SECRET=Enter_the_Client_Secret_Here
>    ```
>    Donde:
>    - `Enter_the_Application_Id_Here`: es el **identificador de aplicación (cliente)** de la aplicación que registró.
>    - `Enter_the_Tenant_Id_Here`: sustituya este valor por el **identificador de inquilino** o el **nombre de inquilino** (por ejemplo, contoso.microsoft.com).
>    - `Enter_the_Client_Secret_Here`: sustituya este valor por el secreto de cliente creado en el paso 1.
>
> > [!TIP]
> > Para buscar los valores de **identificador de aplicación (cliente)** e **identificador de directorio (inquilino)** , vaya a la página **Información general** de Azure Portal. Para generar una nueva clave, vaya a la página **Certificates & secrets** (Certificados y secretos).

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-admin-consent"></a>Paso 3: Consentimiento de administrador

> [!div renderon="docs"]
> #### <a name="step-4-admin-consent"></a>Paso 4: Consentimiento de administrador

Si intenta ejecutar la aplicación en este momento, recibirá un error *HTTP 403 - Prohibido*: `Insufficient privileges to complete the operation`. Este error sucede porque cualquier *permiso de solo aplicación* requiere el consentimiento del administrador: un administrador global del directorio debe otorgar su consentimiento a la aplicación. Seleccione una de las opciones siguientes según el rol:

##### <a name="global-tenant-administrator"></a>Administrador de inquilinos global

> [!div renderon="docs"]
> Si es administrador de inquilinos global, vaya a la página **API Permissions** (Permisos de API) de **Registros de aplicaciones** de Azure Portal (versión preliminar) y seleccione **Grant admin consent for {Tenant Name}** (Conceder consentimiento del administrador para {Tenant Name}), donde el nombre del inquilino es el nombre del directorio.

> [!div renderon="portal" class="sxs-lookup"]
> Si es administrador global, vaya a la página **API Permissions** (Permisos de API), seleccione **Grant admin consent for Enter_the_Tenant_Name_Here** (Conceder consentimiento del administrador para _escribir_aquí_el_nombre_del_inquilino).
> > [!div id="apipermissionspage"]
> > [Ir a la página Permisos de API]()

##### <a name="standard-user"></a>Usuario estándar

Si es usuario estándar de su inquilino, deberá pedir a un administrador global que conceda consentimiento del administrador para su aplicación. Para ello, proporcione la siguiente dirección URL a su administrador:

```url
https://login.microsoftonline.com/Enter_the_Tenant_Id_Here/adminconsent?client_id=Enter_the_Application_Id_Here
```

> [!div renderon="docs"]
>> Donde:
>> * `Enter_the_Tenant_Id_Here`: sustituya este valor por el **identificador de inquilino** o el **nombre de inquilino** (por ejemplo, contoso.microsoft.com).
>> * `Enter_the_Application_Id_Here`: es el **identificador de aplicación (cliente)** de la aplicación que registró.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-4-run-the-application"></a>Paso 4: Ejecución de la aplicación

> [!div renderon="docs"]
> #### <a name="step-5-run-the-application"></a>Paso 5: Ejecución de la aplicación

Puede probar el ejemplo directamente mediante la ejecución del método main de ClientCredentialGrant.Java desde el IDE.

Desde el shell o la línea de comandos, ejecute:

```
$ mvn clean compile assembly:single
```

Se generará el archivo msal-client-credential-secret-1.0.0.jar en el directorio /targets. Ejecútelo con el archivo ejecutable de Java como se indica a continuación:

```
$ java -jar msal-client-credential-secret-1.0.0.jar
```

Después de la ejecución, la aplicación debe mostrar la lista de usuarios del inquilino configurado.


> [!IMPORTANT]
> Esta aplicación de inicio rápido usa un secreto de cliente para identificarse como cliente confidencial. Como el secreto de cliente se agrega como texto sin formato a los archivos del proyecto, por motivos de seguridad, se recomienda que use un certificado en lugar de un secreto de cliente antes de considerar el uso de la aplicación en producción. Para más información sobre cómo usar un certificado, consulte [estas instrucciones](https://github.com/Azure-Samples/ms-identity-java-daemon/tree/master/msal-client-credential-certificate) en el mismo repositorio de GitHub que este ejemplo, pero en la segunda carpeta, **msal-client-credential-certificate**.

## <a name="more-information"></a>Más información

### <a name="msal-java"></a>Java de MSAL

[MSAL para Java](https://github.com/AzureAD/microsoft-authentication-library-for-java) es la biblioteca que se usa para iniciar la sesión de los usuarios y solicitar los tokens que se usan para acceder a una API protegida por la Plataforma de identidad de Microsoft. Como se ha descrito, en este inicio rápido se solicitan tokens mediante la propia identidad de la aplicación, en lugar de permisos delegados. El flujo de autenticación usado en este caso se conoce como *[flujo de OAuth de credenciales de cliente](v2-oauth2-client-creds-grant-flow.md)* . Para más información sobre cómo usar MSAL para Java con aplicaciones de demonio, consulte [este artículo](scenario-daemon-overview.md).

Agregue MSAL4J a la aplicación con Maven o Gradle para administrar las dependencias al realizar los cambios siguientes en el archivo pom.xml (Maven) o build.gradle (Gradle) de la aplicación.

En pom.xml:

```XML
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>msal4j</artifactId>
    <version>1.0.0</version>
</dependency>
```

En build.gradle:

```$xslt
compile group: 'com.microsoft.azure', name: 'msal4j', version: '1.0.0'
```

### <a name="msal-initialization"></a>Inicialización de MSAL

Para agregar una referencia a MSAL for Java, incorpore el código siguiente al principio del archivo en el que va a usar MSAL4J:

```Java
import com.microsoft.aad.msal4j.*;
```

A continuación, realice la inicialización de MSAL con el siguiente código:

```Java
IClientCredential credential = ClientCredentialFactory.createFromSecret(CLIENT_SECRET);

ConfidentialClientApplication cca =
        ConfidentialClientApplication
                .builder(CLIENT_ID, credential)
                .authority(AUTHORITY)
                .build();
```

> | Donde: |Descripción |
> |---------|---------|
> | `CLIENT_SECRET` | Es el secreto de cliente creado para la aplicación en Azure Portal. |
> | `CLIENT_ID` | Es el **Identificador de aplicación (cliente)** de la aplicación registrada en Azure Portal. Puede encontrar este valor en la página **Información general** de la aplicación en Azure Portal. |
> | `AUTHORITY`    | El punto de conexión STS para el usuario que se autenticará. Normalmente `https://login.microsoftonline.com/{tenant}` en la nube pública, donde {tenant} es el nombre o el identificador del inquilino.|

### <a name="requesting-tokens"></a>Solicitud de tokens

Para solicitar un token mediante la identidad de la aplicación, use el método `acquireToken`:

```Java
IAuthenticationResult result;
     try {
         SilentParameters silentParameters =
                 SilentParameters
                         .builder(SCOPE)
                         .build();

         // try to acquire token silently. This call will fail since the token cache does not
         // have a token for the application you are requesting an access token for
         result = cca.acquireTokenSilently(silentParameters).join();
     } catch (Exception ex) {
         if (ex.getCause() instanceof MsalException) {

             ClientCredentialParameters parameters =
                     ClientCredentialParameters
                             .builder(SCOPE)
                             .build();

             // Try to acquire a token. If successful, you should see
             // the token information printed out to console
             result = cca.acquireToken(parameters).join();
         } else {
             // Handle other exceptions accordingly
             throw ex;
         }
     }
     return result;
```

> |Donde:| Descripción |
> |---------|---------|
> | `SCOPE` | Contiene los ámbitos solicitados. Con clientes confidenciales se debe usar un formato similar a `{Application ID URI}/.default` para indicar que los ámbitos que se solicitan son los definidos estáticamente en el objeto de aplicación establecido en Azure Portal (con Microsoft Graph, `{Application ID URI}` apunta a `https://graph.microsoft.com`). Con API web personalizadas, `{Application ID URI}` se define en la sección **Expose an API** (Exponer una API) **Registros de aplicaciones** de Azure Portal.|

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las aplicaciones demonio, consulte la página de aterrizaje del escenario.

> [!div class="nextstepaction"]
> [Aplicación demonio que llama a las API web](scenario-daemon-overview.md)
