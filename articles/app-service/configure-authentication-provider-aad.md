---
title: Configuración de la autenticación de Azure AD
description: Aprenda a configurar la autenticación de Azure Active Directory como proveedor de identidades para una aplicación de App Service o Azure Functions.
ms.assetid: 6ec6a46c-bce4-47aa-b8a3-e133baef22eb
ms.topic: article
ms.date: 04/14/2020
ms.custom: seodec18, fasttrack-edit, has-adal-ref
ms.openlocfilehash: b1254e7db0e62d08ea2a3d6d30f2abd379675c55
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106078322"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-azure-ad-login"></a>Configuración de una aplicación de App Service o Azure Functions para usar el inicio de sesión de Azure AD

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

En este artículo se muestra cómo configurar la autenticación para Azure App Service o Azure Functions de modo que la aplicación inicie la sesión de los usuarios con la [plataforma de identidad de Microsoft](../active-directory/develop/v2-overview.md) (Azure AD) como proveedor de autenticación.

La característica de autenticación de App Service puede crear automáticamente un registro de aplicaciones con la plataforma de identidad de Microsoft. También es posible usar un registro que el usuario o un administrador de directorios cree por separado.

- [Creación de un nuevo registro de aplicaciones automáticamente](#express)
- [Uso de un registro existente creado por separado](#advanced)

> [!NOTE]
> La opción para crear un nuevo registro no está disponible para las nubes de la administración pública. En su lugar, [defina un registro por separado](#advanced).

## <a name="create-a-new-app-registration-automatically"></a><a name="express"> </a>Creación de un nuevo registro de aplicaciones automáticamente

Esta opción está diseñada para simplificar la autenticación y requiere unos pocos clics.

1. Inicie sesión en [Azure Portal] y vaya a la aplicación.
1. Seleccione **Autenticación** en el menú de la izquierda. Haga clic en **Agregar proveedor de identidades**.
1. Seleccione **Microsoft** en la lista desplegable del proveedor de identidades. La opción para crear un nuevo registro está seleccionada de forma predeterminada. Puede cambiar el nombre del registro o los tipos de cuenta admitidos.

    Se creará un secreto de cliente y se almacenará como una [configuración de aplicación](./configure-common.md#configure-app-settings) con espacios fijos denominada `MICROSOFT_PROVIDER_AUTHENTICATION_SECRET`. Puede actualizar esa configuración más adelante para usar [referencias de Key Vault](./app-service-key-vault-references.md) si desea administrar el secreto en Azure Key Vault.

1. Si este es el primer proveedor de identidades configurado para la aplicación, también se mostrará una sección de **configuración de autenticación de App Service**. De lo contrario, puede pasar al siguiente paso.
    
    Estas opciones determinan el modo en que la aplicación responde a las solicitudes no autenticadas y las selecciones predeterminadas redirigirán todas las solicitudes para iniciar sesión con este nuevo proveedor. Puede cambiar este comportamiento ahora o ajustar esta configuración más adelante desde la pantalla principal **Autenticación**; para ello, elija **Editar** junto a **Configuración de la autenticación**. Para obtener más información acerca de estas opciones, consulte [Flujo de autenticación](overview-authentication-authorization.md#authentication-flow).

1. (Opcional) Haga clic en **Siguiente: Permisos** y agregue los ámbitos que necesite la aplicación. Estos se agregarán al registro de la aplicación, pero también puede cambiarlos más adelante.
1. Haga clic en **Agregar**.

De este modo ya estará listo para usar la plataforma de identidad de Microsoft para realizar la autenticación en la aplicación. El proveedor se mostrará en la pantalla **Autenticación**. Desde allí, puede editar o eliminar esta configuración de proveedor.

Para obtener un ejemplo de configuración de inicio de sesión de Azure AD para una aplicación web con acceso a Azure Storage y Microsoft Graph, consulte [este tutorial](scenario-secure-app-authentication-app-service.md).

## <a name="use-an-existing-registration-created-separately"></a><a name="advanced"> </a>Uso de un registro existente creado por separado

También puede registrar manualmente la aplicación en la plataforma de identidad de Microsoft, personalizando el registro y configurando la autenticación de App Service con los detalles de registro. Esto resulta útil, por ejemplo, si quiere usar un registro de aplicaciones de un inquilino de Azure AD diferente de aquel donde está la aplicación.

### <a name="create-an-app-registration-in-azure-ad-for-your-app-service-app"></a><a name="register"> </a>Creación de un registro de aplicaciones en Azure AD para la aplicación App Service

En primer lugar, creará el registro de la aplicación. Al hacerlo, recopile la siguiente información que necesitará más adelante cuando configure la autenticación en la aplicación App Service:

- Id. de cliente
- Id. de inquilino
- Secreto de cliente (opcional)
- URI de Id. de aplicación

Para registrar la aplicación, lleve a cabo los siguientes pasos:

1. Inicie sesión en [Azure Portal], busque y seleccione **App Services** y luego elija la aplicación. Anote la **Dirección URL** de la aplicación. La usará para configurar el registro de la aplicación de Azure Active Directory.
1. En el menú del portal, seleccione **Azure Active Directory**, vaya a la pestaña **Registros de aplicaciones** y seleccione **Nuevo registro**.
1. En la página **Register an application** (Registrar una aplicación), escriba el **nombre** del registro de aplicaciones.
1. En **URI de redirección**, seleccione **Web** y escriba `<app-url>/.auth/login/aad/callback`. Por ejemplo, `https://contoso.azurewebsites.net/.auth/login/aad/callback`.
1. Seleccione **Registrar**.
1. Una vez creado el registro de la aplicación, copie el **Id. de aplicación (cliente)** y el **Id. de directorio (inquilino)** para usarlos más adelante.
1. Seleccione **Autenticación**. En **Concesión implícita**, habilite **Tokens de id.** para permitir que el usuario de OpenID Connect inicie sesión desde App Service.
1. (Opcional) Seleccione **Personalización de marca**. En **URL de página principal**, escriba la dirección URL de la aplicación de App Service y seleccione **Guardar**.
1. Seleccione **Expose an API (Exponer una API)**  > **Set (Conjunto)** . En el caso de una aplicación de un solo inquilino, pegue la dirección URL de la aplicación de App Service y seleccione **Guardar** y, para la aplicación multiinquilino, pegue la dirección URL que se basa en uno de los dominios comprobados por el inquilino y, a continuación, seleccione **Guardar**.

   > [!NOTE]
   > Este valor es el **URI del identificador de la aplicación** del registro de aplicaciones. Si la aplicación web requiere acceso a una API en la nube, al configurar el recurso de App Service en la nube necesitará el valor de **URI de Id. de aplicación** de la aplicación web. Puede utilizarlo, por ejemplo, si desea que el servicio en la nube conceda acceso explícitamente a la aplicación web.

1. Seleccione **Agregar un ámbito**.
   1. En **Nombre de ámbito**, escriba *user_impersonation*.
   1. En los cuadros de texto, escriba el nombre y la descripción del ámbito de consentimiento que quiere que vean los usuarios en la página de consentimiento. Por ejemplo, escriba *Access my app* (Acceder a mi aplicación).
   1. Seleccione la opción **Agregar un ámbito**.
1. (Opcional) Para crear un secreto de cliente, seleccione **Certificates & secrets (Certificados y secretos)**  > **New client secret (Nuevo secreto de cliente)**  > **Add (Agregar)** . Copie el valor del secreto del cliente que se muestra en la página. No se volverá a mostrar.
1. (Opcional) Para agregar varios valores en **Direcciones URL de respuesta**, seleccione **Autenticación**.

### <a name="enable-azure-active-directory-in-your-app-service-app"></a><a name="secrets"> </a>Habilitación de Azure Active Directory en la aplicación de App Service

1. Inicie sesión en [Azure Portal] y vaya a la aplicación.
1. Seleccione **Autenticación** en el menú de la izquierda. Haga clic en **Agregar proveedor de identidades**.
1. Seleccione **Microsoft** en la lista desplegable del proveedor de identidades.
1. En **App registration type** (Tipo de registro de aplicaciones), puede elegir **Pick an existing app registration in this directory** (Seleccionar in registro de aplicaciones existente en este directorio), que recopilará automáticamente la información necesaria de la aplicación. Si el registro procede de otro inquilino o no tiene permiso para ver el objeto de registro, elija **Proporcione los detalles de un registro de aplicación existente.** . Para esta opción, tendrá que rellenar los siguientes detalles de configuración:

    |Campo|Descripción|
    |-|-|
    |Id. de aplicación (cliente)| Use el **identificador de la aplicación (cliente)** del registro de aplicaciones. |
    |Secreto de cliente (opcional)| Use el secreto de cliente que generó en el registro de la aplicación. Con un secreto de cliente, se usa el flujo híbrido y el App Service devolverá el acceso y los tokens de actualización. Cuando no se establece el secreto de cliente, se usa el flujo implícito y solo se devuelve un token de identificador. Estos tokens los envía el proveedor y se almacenan en el almacén de tokens de EasyAuth.|
    |Dirección URL del emisor| Use `<authentication-endpoint>/<tenant-id>/v2.0` y reemplace *\<authentication-endpoint>* por el [punto de conexión de autenticación del entorno de nube](../active-directory/develop/authentication-national-cloud.md#azure-ad-authentication-endpoints) (por ejemplo, "https://login.microsoftonline.com" para Azure global), y reemplace también *\<tenant-id>* por el **identificador de directorio (inquilino)** en el que se creó el registro de la aplicación. Este valor se usa para redirigir a los usuarios al inquilino de Azure AD correcto, así como para descargar los metadatos adecuados para determinar las claves de firma de tokens y el valor de notificación del emisor del token correspondientes, por ejemplo. En las aplicaciones que usan Azure AD v1 y en las aplicaciones de Azure Functions, debe omitirse `/v2.0` en la dirección URL.|
    |Audiencias de token permitidas| Si se trata de una aplicación en la nube o una aplicación de servidor y quiere permitir tokens de autenticación desde una aplicación web, agregue aquí el valor de **URI de Id. de aplicación** de la aplicación web. De forma implícita, el **Id. de cliente** se considera *siempre* que es un público permitido.|

    El secreto de cliente se almacenará como una [configuración de la aplicación](./configure-common.md#configure-app-settings) con espacios fijos denominada `MICROSOFT_PROVIDER_AUTHENTICATION_SECRET`. Puede actualizar esa configuración más adelante para usar [referencias de Key Vault](./app-service-key-vault-references.md) si desea administrar el secreto en Azure Key Vault.

1. Si este es el primer proveedor de identidades configurado para la aplicación, también se mostrará una sección de **configuración de autenticación de App Service**. De lo contrario, puede pasar al siguiente paso.
    
    Estas opciones determinan el modo en que la aplicación responde a las solicitudes no autenticadas y las selecciones predeterminadas redirigirán todas las solicitudes para iniciar sesión con este nuevo proveedor. Puede cambiar este comportamiento ahora o ajustar esta configuración más adelante desde la pantalla principal **Autenticación**; para ello, elija **Editar** junto a **Configuración de la autenticación**. Para obtener más información acerca de estas opciones, consulte [Flujo de autenticación](overview-authentication-authorization.md#authentication-flow).

1. Haga clic en **Agregar**.

De este modo ya estará listo para usar la plataforma de identidad de Microsoft para realizar la autenticación en la aplicación. El proveedor se mostrará en la pantalla **Autenticación**. Desde allí, puede editar o eliminar esta configuración de proveedor.

## <a name="configure-client-apps-to-access-your-app-service"></a>Configuración de las aplicaciones cliente para acceder a la instancia de App Service

En la sección anterior, registró la instancia de App Service o Azure Functions para autenticar a los usuarios. En esta sección se explica cómo registrar aplicaciones de demonio o de cliente nativas para que puedan solicitar acceso a las API expuestas por la instancia de App Service en nombre de los usuarios o en su propio nombre. No es necesario completar los pasos de esta sección si solo desea autenticar a los usuarios.

### <a name="native-client-application"></a>Aplicación cliente nativa

Puede registrar clientes nativos para solicitar acceso a las API de la aplicación de App Service en nombre de un usuario que ha iniciado sesión.

1. En [Azure Portal], seleccione **Azure Active Directory** > **Registros de aplicaciones** > **Nuevo registro**.
1. En la página **Register an application** (Registrar una aplicación), escriba el **nombre** del registro de aplicaciones.
1. En **URI de redirección**, seleccione **Cliente público (móvil y escritorio)** , y escriba la dirección URL `<app-url>/.auth/login/aad/callback`. Por ejemplo, `https://contoso.azurewebsites.net/.auth/login/aad/callback`.

    > [!NOTE]
    > Para una aplicación de Microsoft Store, use el valor de [SID del paquete](/previous-versions/azure/app-service-mobile/app-service-mobile-dotnet-how-to-use-client-library#package-sid) como URI en su lugar.
1. Seleccione **Crear**.
1. Una vez creado el registro de aplicación, copie el valor de **Id. de aplicación (cliente)** .
1. Seleccione **Permisos de API** > **Agregar permiso** > **Mis API**.
1. Seleccione el registro de aplicaciones que creó anteriormente para la aplicación de App Service. Si no ve el registro de aplicación, compruebe que agregó el ámbito **user_impersonation** en [Creación de un registro de aplicaciones en Azure AD para la aplicación App Service](#register).
1. En **Permisos delegados**, seleccione **user_impersonation** y luego seleccione **Agregar permisos**.

Ahora ha configurado una aplicación cliente nativa que puede solicitar acceso a la aplicación de App Service en nombre de un usuario.

### <a name="daemon-client-application-service-to-service-calls"></a>Aplicación cliente de demonio (llamadas de servicio a servicio)

La aplicación puede adquirir un token para llamar a una API web hospedada en App Service o la aplicación de funciones en nombre propio (no en el nombre de un usuario). Este escenario es útil para las aplicaciones de demonio no interactivas que realizan tareas sin un usuario que ha iniciado sesión. Usa la concesión de [credenciales del cliente](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md) OAuth 2.0 estándar.

1. En [Azure Portal], seleccione **Azure Active Directory** > **Registros de aplicaciones** > **Nuevo registro**.
1. En la página **Register an application** (Registrar una aplicación), escriba el **nombre** del registro de aplicaciones de demonio.
1. En el caso de una aplicación de demonio, no necesita un URI de redireccionamiento para que pueda mantenerlo vacío.
1. Seleccione **Crear**.
1. Una vez creado el registro de aplicación, copie el valor de **Id. de aplicación (cliente)** .
1. Seleccione **Certificados y secretos** > **Nuevo secreto de cliente** > **Agregar**. Copie el valor del secreto del cliente que se muestra en la página. No se volverá a mostrar.

Ahora puede [solicitar un token de acceso mediante el Id. de cliente y el secreto de cliente](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md#first-case-access-token-request-with-a-shared-secret) estableciendo el `resource`parámetro en el **URI del id. de aplicación** de la aplicación de destino. Después, el token de acceso resultante se puede presentar a la aplicación de destino mediante el [encabezado de autorización estándar OAuth 2.0](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md#use-the-access-token-to-access-the-secured-resource), y la autenticación o autorización de App Service validarán y usarán el token como de costumbre para indicar ahora que el autor de llamada (una aplicación en este caso, no un usuario) está autenticado.

En la actualidad, esto permite a _cualquier_ aplicación cliente en el inquilino de Azure AD solicitar un token de acceso y autenticarse en la aplicación de destino. Si también desea aplicar _autorización_ para permitir solo determinadas aplicaciones cliente, debe realizar una configuración adicional.

1. [Defina un rol de aplicación](../active-directory/develop/howto-add-app-roles-in-azure-ad-apps.md) en el manifiesto del registro de la aplicación que representa App Service o la aplicación de función que quiere proteger.
1. En el registro de la aplicación que representa el cliente que debe ser autorizado, seleccione **Permisos de API** > **Agregar un permiso** > **Mis API**.
1. Seleccione el registro de aplicaciones que creó anteriormente. Si no ve el registro de la aplicación, asegúrese de que ha [agregado un rol de aplicación](../active-directory/develop/howto-add-app-roles-in-azure-ad-apps.md).
1. En **Permisos de aplicación**, seleccione el rol de aplicación que creó anteriormente y, a continuación, seleccione **Agregar permisos**.
1. Asegúrese de hacer clic en **Conceder consentimiento del administrador** para autorizar a la aplicación cliente solicitar el permiso.
1. Al igual que en el escenario anterior (antes de que se agregaran los roles), ahora puede [solicitar un token de acceso](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md#first-case-access-token-request-with-a-shared-secret) para el mismo destino`resource`, y el token de acceso incluirá una `roles` solicitud que contiene los roles de aplicación que se autorizaron para la aplicación cliente.
1. En el App Service de destino o el código de la aplicación de función, ahora puede validar que los roles esperados están presentes en el token (esto no se realiza mediante autenticación o autorización de App Service). Para más información, consulte [Access user claims](app-service-authentication-how-to.md#access-user-claims) (Acceso a las notificaciones de usuario).

Ahora ha configurado una aplicación cliente demonio que puede acceder a la aplicación de App Service con su propia identidad.

## <a name="best-practices"></a>Procedimientos recomendados

Independientemente de la configuración que use para la autenticación, los siguientes procedimientos recomendados mantendrán el inquilino y las aplicaciones más seguros:

- Asigne a cada aplicación de App Service sus propios permisos y consentimiento.
- Configure cada aplicación de App Service con su propio registro.
- Evite el uso compartido de permisos entre entornos mediante registros de aplicación independientes para ranuras de implementación independientes. Al probar nuevo código, esta práctica puede ayudar a evitar que los problemas afecten a la aplicación de producción.

## <a name="next-steps"></a><a name="related-content"> </a>Pasos siguientes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]
* [Tutorial: Autenticación y autorización de usuarios en una aplicación web con acceso a Azure Storage y Microsoft Graph](scenario-secure-app-authentication-app-service.md)
* [Tutorial: Autenticación y autorización de usuarios de extremo a extremo en Azure App Service](tutorial-auth-aad.md)
<!-- URLs. -->

[Azure Portal]: https://portal.azure.com/
