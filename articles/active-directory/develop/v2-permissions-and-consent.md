---
title: Ámbitos, permisos y consentimiento de la Plataforma de identidad de Microsoft
description: Aprenda sobre la autorización en el punto de conexión de la plataforma de identidad de Microsoft, incluidos los ámbitos, los permisos y el consentimiento.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 04/14/2021
ms.author: ryanwi
ms.reviewer: hirsin, jesakowi, jmprieur, marsma
ms.custom: aaddev, fasttrack-edit, contperf-fy21q1, identityplatformtop40
ms.openlocfilehash: 2a975a0aba06ecfd010fe328ef6c8cda75290f2b
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107515590"
---
# <a name="permissions-and-consent-in-the-microsoft-identity-platform"></a>Permisos y consentimiento en la plataforma de identidad de Microsoft

Las aplicaciones que se integran con la Plataforma de identidad de Microsoft siguen un modelo de autorización que permite a los usuarios y los administradores controlar el modo en que se puede acceder a los datos. La implementación del modelo de autorización se ha actualizado en la Plataforma de identidad de Microsoft. Cambia el modo en que una aplicación debe interactuar con la Plataforma de identidad de Microsoft. En este artículo se tratan los conceptos básicos de este modelo de autorización, incluidos los ámbitos, los permisos y el consentimiento.

## <a name="scopes-and-permissions"></a>Permisos y ámbitos

La plataforma de identidad de Microsoft implementa el protocolo de autorización [OAuth 2.0](active-directory-v2-protocols.md). OAuth 2.0 es un método a través del cual una aplicación de terceros puede acceder a recursos hospedados en Web en nombre de un usuario. Cualquier recurso hospedado en Web que se integre con la Plataforma de identidad de Microsoft tiene un identificador de recursos o un *URI de identificador de aplicación*. 

Estos son algunos ejemplos de recursos hospedados en Web de Microsoft:

* Microsoft Graph: `https://graph.microsoft.com`
* Mail API de Microsoft 365: `https://outlook.office.com`
* `https://vault.azure.net`Azure Key Vault:

Ocurre lo mismo con cualquier recurso de terceros integrado en la plataforma de identidad de Microsoft. Cualquiera de estos recursos también puede definir un conjunto de permisos que se puede usar para dividir la funcionalidad de ese recurso en fragmentos menores. Por ejemplo, [Microsoft Graph](https://graph.microsoft.com) tiene permisos definidos para realizar, entre otras, las tareas siguientes:

* Leer el calendario de un usuario
* Escribir en el calendario de un usuario
* Enviar correo como un usuario

Debido a estas definiciones de tipos de permisos, el recurso tiene control específico sobre sus datos y el modo en que la funcionalidad de la API se expone. Una aplicación de terceros puede solicitar estos permisos a los usuarios y administradores, quienes deben aprobar la solicitud para que la aplicación pueda acceder a los datos o actuar en nombre de un usuario. 

Al fragmentar la funcionalidad de un recurso en conjuntos de permisos pequeños, se pueden crear aplicaciones de terceros para solicitar solo los permisos que necesitan para realizar sus tareas. Los usuarios y administradores pueden saber a qué datos puede acceder la aplicación. Además, pueden estar más seguros de que la aplicación no se comporte con una intención malintencionada. Los desarrolladores deben cumplir siempre con el principio del mínimo privilegio y pedir solo los permisos que necesitan para que sus aplicaciones funcionen.

En OAuth 2.0, estos tipos de conjuntos de permisos se denominan *ámbitos*. A menudo se hace referencia a ellos como *permisos*. En la Plataforma de identidad de Microsoft, un permiso se representa como un valor de cadena. Una aplicación solicita los permisos que necesita al especificar el permiso en el parámetro de consulta `scope`. La plataforma de identidad admite varios [ámbitos de OpenID Connect](#openid-connect-scopes) bien definidos, así como permisos basados en recursos (cada permiso se indica al anexar el valor de permiso al identificador del recurso o al URI del identificador de aplicación). Por ejemplo, la cadena de permiso `https://graph.microsoft.com/Calendars.Read` se usa para solicitar permiso para leer los calendarios de los usuarios en Microsoft Graph.

Una aplicación suele solicitar estos permisos, para lo que especifica los ámbitos en las solicitudes al punto de conexión de la autorización de la Plataforma de identidad de Microsoft. Sin embargo, algunos permisos con privilegios elevados solo se pueden conceder con el consentimiento del administrador. Se pueden solicitar o conceder con el [punto de conexión de consentimiento del administrador](#admin-restricted-permissions). Siga leyendo para obtener más información.

## <a name="permission-types"></a>Tipos de permisos

La Plataforma de identidad de Microsoft admite dos tipos de permisos: *permisos delegados* y *permisos de aplicación*.

* **Permisos delegados**: se utilizan en aplicaciones que tienen un usuario con la sesión iniciada. Para estas aplicaciones, el usuario o un administrador dan su consentimiento para los permisos que la aplicación requiere. A la aplicación se le delega el permiso para actuar como el usuario que inició sesión al realizar llamadas al recurso de destino. 

    Quienes no son administradores pueden dar su consentimiento para algunos permisos delegados. Sin embargo, algunos permisos con privilegios elevados requieren el [consentimiento de un administrador](#admin-restricted-permissions). Para información sobre qué roles de administrador pueden dar su consentimiento para los permisos delegados, vea [Permisos de roles de administrador en Azure Active Directory (Azure AD)](../roles/permissions-reference.md).

* Los **permisos de aplicación** los usan las aplicaciones que se ejecutan sin la presencia de un usuario con la sesión iniciada; por ejemplo, las aplicaciones que se ejecutan como demonios o servicios en segundo plano. Solo [un administrador puede dar su consentimiento para](#requesting-consent-for-an-entire-tenant) los permisos de aplicación.

Los _permisos efectivos_ son los que la aplicación tiene al realizar solicitudes al recurso de destino. Es importante comprender la diferencia entre, por un lado, los permisos delegados y los permisos de la aplicación que se conceden a una aplicación y, por otro, los permisos efectivos que se conceden a la aplicación cuando realiza llamadas al recurso de destino.

- En el caso de los permisos delegados, los _permisos efectivos_ de la aplicación son la intersección con menos privilegios de los permisos delegados que se le han concedido a la aplicación (mediante consentimiento) y los privilegios del usuario que tiene iniciada la sesión actualmente. La aplicación nunca puede tener más privilegios que el usuario que tiene la sesión iniciada. 

   Dentro de las organizaciones, los privilegios del usuario que tiene la sesión iniciada pueden determinarse mediante directivas o pertenencia a uno o varios roles de administrador. Para información sobre qué roles de administrador pueden consentir los permisos delegados, consulte [Permisos de roles de administrador en Azure AD](../roles/permissions-reference.md).

   Por ejemplo, suponga que a su aplicación se le ha concedido el permiso delegado _User.ReadWrite.All_ en Microsoft Graph. Este permiso concede a su aplicación de forma nominal un permiso para leer y actualizar el perfil de cada usuario de una organización. Si el usuario que inició sesión es un administrador global, la aplicación puede actualizar el perfil de cada usuario de la organización. Aun así, si el usuario con la sesión iniciada no tiene un rol de administrador, la aplicación puede actualizar solo el perfil del usuario que tiene la sesión iniciada. No puede actualizar los perfiles de otros usuarios de la organización, porque el usuario para el que tiene permiso para actuar en su nombre no tiene tales privilegios.

- Para los permisos de aplicación, los _permisos efectivos_ de la aplicación son el nivel completo de privilegios que concede el permiso. Por ejemplo, una aplicación que tiene el permiso de aplicación _User.ReadWrite.All_ puede actualizar el perfil de cada usuario de la organización.

## <a name="openid-connect-scopes"></a>Ámbitos de OpenID Connect

La implementación de la Plataforma de identidad de Microsoft de OpenID Connect tiene unos cuantos ámbitos bien definidos que también se hospedan en Microsoft Graph: `openid`, `email`, `profile` y `offline_access`. Los ámbitos de OpenID Connect `address` y `phone` no son compatibles.

Si solicita los ámbitos de OpenID Connect y un token, obtendrá un token para llamar al [punto de conexión de UserInfo](userinfo.md).

### <a name="openid"></a>openid

Si una aplicación inicia sesión mediante [OpenID Connect](active-directory-v2-protocols.md), debe solicitar el ámbito `openid`. El ámbito `openid` aparece en la página de consentimiento de la cuenta profesional como el permiso **Iniciar sesión en su nombre**. En la página de consentimiento de la cuenta personal de Microsoft, aparece como el permiso **Ver el perfil y conectarse a las aplicaciones y servicios con la cuenta de Microsoft**. 

Con este permiso, una aplicación puede recibir un identificador único para el usuario en forma de la notificación `sub`. El permiso también ofrece acceso de la aplicación al punto de conexión de UserInfo. El ámbito `openid` puede usarse en el punto de conexión del token de la Plataforma de identidad de Microsoft para adquirir tokens de identificador. La aplicación puede usar estos tokens para la autenticación.

### <a name="email"></a>email

El ámbito `email` puede usarse con el ámbito `openid` y con cualquier otro. Proporciona acceso de la aplicación a la dirección de correo electrónico principal del usuario en forma de la notificación `email`. 

La notificación `email` solo se incluye en los tokens si hay una dirección de correo electrónico asociada a la cuenta de usuario, que no siempre es el caso. Si la aplicación usa el ámbito `email`, la aplicación debe ser capaz de controlar un caso en el que no existe ninguna notificación `email` en el token.

### <a name="profile"></a>perfil

El ámbito `profile` puede usarse con el ámbito `openid` y con cualquier otro. Proporciona acceso de la aplicación a una cantidad elevada de información sobre el usuario. Por ejemplo, el nombre propio del usuario, el apellido, el nombre de usuario preferido o el id. de objeto, entre otros datos. 

Para ver una lista completa de las notificaciones `profile` disponibles en el parámetro `id_tokens` para un usuario específico, vea la [referencia `id_tokens`](id-tokens.md).

### <a name="offline_access"></a>offline_access

El ámbito [`offline_access`](https://openid.net/specs/openid-connect-core-1_0.html#OfflineAccess) proporciona acceso de la aplicación a recursos en nombre del usuario durante un periodo de tiempo prolongado. En la página de consentimiento, este ámbito aparece como el permiso **Mantener el acceso a los datos a los que le ha dado acceso**. 

Cuando un usuario aprueba el ámbito `offline_access`, la aplicación puede recibir tokens de actualización del punto de conexión del token de la Plataforma de identidad de Microsoft. Los tokens de actualización tienen una duración larga. La aplicación puede obtener nuevos tokens de acceso cuando expiren los antiguos.

> [!NOTE]
> Este permiso aparece actualmente en todas las páginas de consentimiento, incluso en el caso de los flujos que no proporcionan un token de actualización (como el [flujo implícito](v2-oauth2-implicit-grant-flow.md)). Esta configuración abarca los escenarios en los que un cliente puede comenzar dentro del flujo implícito y después pasar al flujo de código, donde sí se espera un token de actualización.

En la plataforma de identidad de Microsoft (solicitudes realizadas al punto de conexión v2.0), la aplicación debe solicitar explícitamente el ámbito de `offline_access` para recibir tokens de actualización. Por tanto, cuando se canjea un código de autorización del [flujo del código de autorización de OAuth 2.0](active-directory-v2-protocols.md), solo se recibirá un token de acceso del punto de conexión `/token`. 

El token de acceso es válido durante un breve período de tiempo. Suele expirar al cabo de una hora. En ese momento, la aplicación tiene que redirigir al usuario de vuelta al punto de conexión `/authorize` para que obtenga un nuevo código de autorización. Durante esta redirección y, en función del tipo de aplicación, puede que el usuario tenga que volver a escribir sus credenciales o dar de nuevo el consentimiento a permisos.

Para más información sobre cómo obtener y usar tokens de actualización, consulte la [referencia del protocolo de la Plataforma de identidad de Microsoft](active-directory-v2-protocols.md).

## <a name="requesting-individual-user-consent"></a>Solicitud de consentimiento de usuario individual

En una solicitud de autorización de [OpenID Connect o OAuth 2.0](active-directory-v2-protocols.md), una aplicación puede solicitar los permisos que necesita mediante el parámetro de consulta `scope`. Por ejemplo, cuando un usuario inicia sesión en una aplicación, la aplicación envía una solicitud como la del ejemplo siguiente. (Se han agregado saltos de línea para mejorar la legibilidad).

```HTTP
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=
https%3A%2F%2Fgraph.microsoft.com%2Fcalendars.read%20
https%3A%2F%2Fgraph.microsoft.com%2Fmail.send
&state=12345
```

El parámetro `scope` es una lista separada por espacios que incluye los permisos delegados que la aplicación solicita. Cada permiso se indica anexando el valor del permiso al identificador del recurso (URI del identificador de aplicación). En la solicitud de ejemplo, la aplicación necesita permiso para leer el calendario del usuario y enviar correo electrónico en nombre del usuario.

Después de que el usuario escriba sus credenciales, la Plataforma de identidad de Microsoft buscará un registro que coincida con el *consentimiento del usuario*. Si el usuario no dio su consentimiento a ninguno de los permisos solicitados en el pasado, y si el administrador no dio su consentimiento a estos permisos en nombre de toda la organización, la Plataforma de identidad de Microsoft solicita al usuario que conceda los permisos solicitados.

En este momento, los permisos `offline_access` ("Mantener el acceso a los datos a los que se le ha dado acceso") y `user.read` ("Iniciar sesión y leer su perfil") se incluyen automáticamente en el consentimiento inicial para una aplicación.  Estos permisos suelen ser necesarios para que la aplicación funcione correctamente. El permiso `offline_access` concede a la aplicación acceso a los tokens de actualización que son críticos para aplicaciones nativas y aplicaciones web. El permiso `user.read` concede acceso a la notificación `sub`. Permite al cliente o a la aplicación identificar correctamente al usuario con el tiempo y acceder a información de usuario rudimentaria.

![Captura de pantalla de ejemplo que muestra el consentimiento de la cuenta profesional](./media/v2-permissions-and-consent/work_account_consent.png)

Cuando el usuario aprueba la solicitud del permiso, el consentimiento se registra. No es necesario que el usuario dé su consentimiento de nuevo cuando inicie sesión posteriormente en la aplicación.

## <a name="requesting-consent-for-an-entire-tenant"></a>Solicitud de consentimiento para un inquilino al completo

Cuando una organización adquiere una licencia o una suscripción de una aplicación, la organización suele desear configurar de forma proactiva la aplicación para que la usen todos sus miembros. Como parte de este proceso, el administrador puede conceder consentimiento a la aplicación para que actúe en nombre de cualquier usuario del inquilino. Si el administrador concede el consentimiento del inquilino entero, los usuarios de la organización no verán una página de consentimiento para la aplicación.

Para solicitar el consentimiento de los permisos delegados para todos los usuarios de un inquilino, la aplicación puede usar el punto de conexión de consentimiento del administrador.

Además, las aplicaciones deben usar el punto de conexión de consentimiento del administrador para solicitar los permisos de la aplicación.

## <a name="admin-restricted-permissions"></a>Permisos restringidos por el administrador

Algunos permisos de privilegios elevados de los recursos de Microsoft se pueden establecer en *restringido para los administradores*. Estos son algunos ejemplos de estos tipos de permisos:

* Leer los perfiles completos de todos los usuarios con `User.Read.All`
* Escribir datos en el directorio de una organización mediante `Directory.ReadWrite.All`
* Leer todos los grupos de seguridad del directorio de una organización con: `Groups.Read.All`

Aunque un usuario consumidor podría conceder acceso de la aplicación a este tipo de datos, los usuarios de la organización no pueden conceder acceso al mismo conjunto de datos confidenciales de la empresa. Si la aplicación solicita acceso a uno de estos permisos desde un usuario de la organización, el usuario recibe un mensaje de error que indica que no está autorizado para dar el consentimiento a los permisos de la aplicación.

Si la aplicación requiere ámbitos para permisos restringidos por el administrador, el administrador de una organización debe dar su consentimiento a esos ámbitos en nombre de los usuarios de la organización. Para evitar que se muestren a los usuarios solicitudes de consentimiento de los permisos que no pueden conceder, la aplicación puede usar el punto de conexión de consentimiento del administrador. El punto de conexión de consentimiento del administrador se aborda en la sección siguiente.

Si la aplicación solicita permisos delegados con privilegios elevados y un administrador los concede a través del punto de conexión de consentimiento del administrador, el consentimiento se concede a todos los usuarios del inquilino.

Si la aplicación solicita permisos de aplicación y un administrador los concede a través del punto de conexión de consentimiento del administrador, esta concesión no se realiza en nombre de un usuario específico. En su lugar, a la aplicación cliente se le conceden los permisos *directamente*. Estos tipos de permisos solo se usan en servicios de demonio y otras aplicaciones no interactivas que se ejecutan en segundo plano.

## <a name="using-the-admin-consent-endpoint"></a>Uso del punto de conexión de consentimiento del administrador

Después de usar el punto de conexión de consentimiento del administrador para conceder el consentimiento del administrador, ya ha terminado. No es necesarios que los usuarios realicen ninguna acción adicional. Una vez concedido el consentimiento del administrador, los usuarios pueden obtener un token de acceso mediante un flujo de autenticación típico. El token de acceso resultante tiene los permisos consentidos.

Cuando un administrador global usa la aplicación y se dirige al punto de conexión de autorización, la Plataforma de identidad de Microsoft detecta el rol de usuario. Pregunta si el administrador global desea dar su consentimiento en nombre de todo el inquilino para los permisos solicitados. En su lugar, podría usar un punto de conexión de consentimiento del administrador dedicado para solicitar proactivamente a un administrador que conceda permiso en nombre de todo el inquilino. Este punto de conexión también es necesario para solicitar permisos de aplicación. No se pueden solicitar permisos de aplicación mediante el punto de conexión de autorización.

Si sigue estos pasos, la aplicación puede recopilar los permisos para todos los usuarios de un inquilino, incluso en los ámbitos restringidos para los administradores. Esta operación tiene privilegios elevados. Use la operación solo si es necesario para su escenario.

Para ver un ejemplo de código que implementa lo pasos, consulte el [ejemplo de ámbitos restringidos para los administradores](https://github.com/Azure-Samples/active-directory-dotnet-admin-restricted-scopes-v2) en GitHub.

### <a name="request-the-permissions-in-the-app-registration-portal"></a>Solicitud de los permisos en el portal de registro de aplicaciones

En el portal de registro de aplicaciones, las aplicaciones pueden enumerar los permisos que necesitan, incluidos los permisos delegados y los permisos de aplicación. Esta configuración permite el uso del ámbito `/.default` y la opción **Conceder consentimiento de administrador** de Azure Portal.  

En general, los permisos se deben definir estáticamente para una aplicación determinada. Deben formar un superconjunto de los permisos que la aplicación solicitará de manera dinámica o incremental.

> [!NOTE]
>Los permisos de aplicación solo se pueden solicitar con [`/.default`](#the-default-scope). Por tanto, si la aplicación necesita permisos de aplicación, asegúrese de que se muestran en el portal de registro de aplicaciones.

Para configurar la lista de permisos solicitados estáticamente para una aplicación:

1. Vaya a su aplicación en la experiencia de inicio rápido <a href="https://go.microsoft.com/fwlink/?linkid=2083908" target="_blank">Azure Portal: Registros de aplicaciones</a>.
1. Seleccione o [cree una aplicación](quickstart-register-app.md) si aún no lo ha hecho.
1. En la página **Información general** de la aplicación, en **Administrar**, seleccione **Permisos de API** > **Agregar un permiso**.
1. Seleccione **Microsoft Graph** en la lista de API disponibles. A continuación, agregue los permisos que requiere la aplicación.
1. Seleccione **Agregar permisos**.

### <a name="recommended-sign-the-user-in-to-your-app"></a>Recomendación: inicie la sesión del usuario en la aplicación

Normalmente, cuando se compila una aplicación que usa el punto de conexión de consentimiento del administrador, la aplicación necesita una página o una vista en la que el administrador pueda aprobar los permisos de la aplicación. Esta página puede ser:

* parte del flujo de registro de la aplicación,
* parte de la configuración de la aplicación o
* un flujo de "conexión" dedicado. 

En muchos casos, tiene sentido que la aplicación muestre esta vista de "conexión" solo después de que un usuario haya iniciado sesión con una cuenta de Microsoft profesional o educativa.

Cuando inicia la sesión del usuario en la aplicación, puede identificar a la organización a la que pertenece el administrador antes de pedirle que apruebe los permisos necesarios. Aunque este paso no es estrictamente necesario, puede ayudarle a crear una experiencia más intuitiva para los usuarios de la organización. 

Para iniciar la sesión del usuario, siga los [tutoriales del protocolo de la Plataforma de identidad de Microsoft](active-directory-v2-protocols.md).

### <a name="request-the-permissions-from-a-directory-admin"></a>Solicitud de los permisos de un administrador de directorios

Cuando esté listo para solicitar permisos al administrador de la organización, puede redirigir al usuario al punto de conexión de consentimiento del administrador de la Plataforma de identidad de Microsoft.

```HTTP
// Line breaks are for legibility only.
GET https://login.microsoftonline.com/{tenant}/v2.0/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
&scope=
https://graph.microsoft.com/calendars.read
https://graph.microsoft.com/mail.send
```


| Parámetro        | Condición        | Descripción                                                                                |
|:--------------|:--------------|:-----------------------------------------------------------------------------------------|
| `tenant` | Obligatorio | El inquilino de directorio al que quiere solicitar permiso. Se puede proporcionar en un formato de nombre descriptivo o GUID. También se puede hacer referencia a él de forma genérica con las organizaciones, como se observa en el ejemplo. No use "común", porque las cuentas personales no pueden proporcionar consentimiento del administrador, salvo en el contexto de un inquilino. Para garantizar la mejor compatibilidad con las cuentas personales que administran los inquilinos, use el identificador de inquilino cuando sea posible. |
| `client_id` | Obligatorio | El identificador de aplicación (cliente) que la experiencia [Azure Portal: Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) asignó a la aplicación. |
| `redirect_uri` | Obligatorio |El URI de redireccionamiento adonde desea que se envíe la respuesta para que la controle la aplicación. Debe coincidir exactamente con uno de los identificadores URI de redirección que registró el Portal de registro de aplicaciones. |
| `state` | Recomendado | Un valor incluido en la solicitud que se devolverá también en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. Use el estado para codificar información sobre el estado del usuario en la aplicación antes de que se produzca la solicitud de autenticación, por ejemplo, la página o vista en la que estaba. |
|`scope`        | Obligatorio        | Define el conjunto de permisos que la aplicación solicita. Los ámbitos pueden ser estáticos (mediante [`/.default`](#the-default-scope)) o dinámicos.  Este conjunto puede incluir los ámbitos de OpenID Connect (`openid`, `profile` o `email`). Si necesita permisos de aplicación, debe usar `/.default` para solicitar la lista de permisos configurada de forma estática.  |


En este momento, Azure AD requiere que un administrador de inquilinos inicie sesión para completar la solicitud. Se solicita al administrador que apruebe todos los permisos solicitados en el parámetro `scope`.  Si ha usado un valor estático (`/.default`), funcionará como el punto de conexión de consentimiento del administración de la versión v1.0 y el consentimiento de solicitud para todos los ámbitos que se encuentran en los permisos necesarios para la aplicación.

#### <a name="successful-response"></a>Respuesta correcta

Si el administrador aprueba los permisos para la aplicación, la respuesta correcta tendrá un aspecto similar al siguiente:

```HTTP
GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True
```

| Parámetro | Descripción |
| --- | --- |
| `tenant` | El inquilino del directorio que concedió los permisos solicitados, en formato GUID. |
| `state` | Un valor incluido en la solicitud que también se devolverá en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. El estado se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, por ejemplo, la página o vista en la que estaban. |
| `admin_consent` | Se establecerá en `True`. |

#### <a name="error-response"></a>Respuesta de error

Si el administrador no aprueba los permisos de la aplicación, la respuesta de error tendrá el aspecto siguiente:

```HTTP
GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request
```

| Parámetro | Descripción |
| --- | --- |
| `error` | Una cadena de código de error que puede usarse para clasificar los tipos de errores que se producen. También se puede usar para reaccionar ante errores. |
| `error_description` | Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error. |

Cuando haya recibido una respuesta correcta del punto de conexión de consentimiento del administrador, la aplicación habrá obtenido los permisos solicitados. Ahora puede solicitar un token para el recurso que desee.

## <a name="using-permissions"></a>Uso de permisos

Después de que el usuario da su consentimiento para los permisos de la aplicación, esta puede obtener tokens de acceso que representan los permisos de la aplicación para acceder a un recurso de alguna manera. Un token de acceso solo se puede usar para un único recurso. Sin embargo, dentro del token de acceso están todos los permisos codificados que se han concedido a la aplicación para ese recurso. Para adquirir un token de acceso, la aplicación puede realizar una solicitud al punto de conexión del token de la Plataforma de identidad de Microsoft, como esta:

```HTTP
POST common/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/json

{
    "grant_type": "authorization_code",
    "client_id": "6731de76-14a6-49ae-97bc-6eba6914391e",
    "scope": "https://outlook.office.com/mail.read https://outlook.office.com/mail.send",
    "code": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...",
    "redirect_uri": "https://localhost/myapp",
    "client_secret": "zc53fwe80980293klaj9823"  // NOTE: Only required for web apps
}
```

Puede usar el token de acceso resultante en las solicitudes HTTP al recurso. Dicho token indica al recurso de forma confiable que la aplicación tiene el permiso apropiado para realizar una tarea específica.

Para más información sobre el protocolo OAuth 2.0 y cómo obtener tokens de acceso, consulte la [referencia del protocolo del punto de conexión de la Plataforma de identidad de Microsoft](active-directory-v2-protocols.md).

## <a name="the-default-scope"></a>Ámbito /.default

Puede usar el ámbito `/.default` para ayudar a migrar las aplicaciones del punto de conexión v1.0 al punto de conexión de la Plataforma de identidad de Microsoft. El ámbito `/.default` está integrado en cada aplicación que hace referencia a la lista estática de los permisos configurados en el registro de la aplicación. 

Un valor `scope` de `https://graph.microsoft.com/.default` tiene la misma funcionalidad que `resource=https://graph.microsoft.com` en el punto de conexión v1.0. Al especificar el ámbito `https://graph.microsoft.com/.default` en su solicitud, la aplicación solicita un token de acceso que incluye los ámbitos para cada permiso de Microsoft Graph que ha seleccionado para la aplicación en el portal de registro de aplicaciones. El ámbito se construye mediante el URI del recurso y `/.default`. Por tanto, si el URI del recurso es `https://contosoApp.com`, el ámbito solicitado es `https://contosoApp.com/.default`.  En los casos en los que debe incluir una segunda barra diagonal para solicitar correctamente el token, vea la [sección sobre barras diagonales finales](#trailing-slash-and-default).

El ámbito `/.default` se puede usar en cualquier flujo de OAuth 2.0. Pero es necesario en el [flujo con derechos delegados](v2-oauth2-on-behalf-of-flow.md) y en el [flujo de credenciales de cliente](v2-oauth2-client-creds-grant-flow.md). También es necesario cuando se usa el punto de conexión de consentimiento del administrador v2 para solicitar los permisos de la aplicación.

Los clientes no pueden combinar el consentimiento estático (`/.default`) y dinámico en una única solicitud. Por lo tanto, `scope=https://graph.microsoft.com/.default+mail.read` genera un error porque combina tipos de ámbitos.

### <a name="default-and-consent"></a>/.default y consentimiento

El ámbito `/.default` también desencadena el comportamiento del punto de conexión v1.0 para `prompt=consent`. Solicita el consentimiento para todos los permisos que la aplicación registró, con independencia del recurso. Si se incluye como parte de la solicitud, el ámbito `/.default` devuelve un token que contiene los ámbitos para el recurso solicitado.

### <a name="default-when-the-user-has-already-given-consent"></a>/.default cuando el usuario ya ha dado el consentimiento

La funcionalidad del ámbito `/.default` es idéntica al comportamiento del punto de conexión v1.0 basado en `resource`. También aporta el comportamiento de consentimiento del punto de conexión v1.0. Esto quiere decir que `/.default` solo desencadena una petición de consentimiento si el usuario no ha concedido ningún permiso entre el cliente y el recurso. 

Si existe este consentimiento, el token devuelto contiene todos los ámbitos que el usuario ha concedido para ese recurso. Sin embargo, si no se ha concedido ningún permiso o si se ha proporcionado el parámetro `prompt=consent`, se muestra una petición de consentimiento para todos los ámbitos que la aplicación cliente registró.

#### <a name="example-1-the-user-or-tenant-admin-has-granted-permissions"></a>Ejemplo 1: El usuario o administrador de inquilinos han concedido permisos

En este ejemplo, el usuario o un administrador de inquilino ha concedido al cliente los permisos de Microsoft Graph `mail.read` y `user.read`. 

Si el cliente realiza una solicitud a `scope=https://graph.microsoft.com/.default`, no se muestra ninguna petición de consentimiento, independientemente del contenido de los permisos registrados por las aplicaciones cliente para Microsoft Graph. El token devuelto contiene los ámbitos `mail.read` y `user.read`.

#### <a name="example-2-the-user-hasnt-granted-permissions-between-the-client-and-the-resource"></a>Ejemplo 2: El usuario no ha concedido permisos entre el cliente y el recurso

En este ejemplo, el usuario no ha dado su consentimiento entre el cliente y Microsoft Graph. El cliente se ha registrado para los permisos `user.read` y `contacts.read`. También se ha registrado para el ámbito `https://vault.azure.net/user_impersonation` de Azure Key Vault. 

Cuando el cliente solicita un token para `scope=https://graph.microsoft.com/.default`, el usuario ve una página de consentimiento para los ámbitos `user.read` y `contacts.read`, y los ámbitos `user_impersonation` de Key Vault. El token devuelto solo contiene los ámbitos `user.read` y `contacts.read`. Solo se puede usar en Microsoft Graph.

#### <a name="example-3-the-user-has-consented-and-the-client-requests-more-scopes"></a>Ejemplo 3: El usuario ha dado su consentimiento y el cliente solicita más ámbitos

En este ejemplo, el usuario ya ha dado su consentimiento para `mail.read` al cliente. El cliente se ha registrado para el ámbito `contacts.read`. 

Cuando el cliente solicita un token con `scope=https://graph.microsoft.com/.default` y solicita el consentimiento mediante `prompt=consent`, el usuario ve una página de consentimiento únicamente con todos los permisos registrados por la aplicación. El ámbito `contacts.read` está en la página de consentimiento, pero `mail.read` no. El token devuelto es para Microsoft Graph. Contiene `mail.read` y `contacts.read`.

### <a name="using-the-default-scope-with-the-client"></a>Uso del ámbito /.default con el cliente

En algunos casos, un cliente puede solicitar su propio ámbito `/.default`. En el siguiente ejemplo se muestra este escenario.

```HTTP
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
response_type=token            //Code or a hybrid flow is also possible here
&client_id=9ada6f8a-6d83-41bc-b169-a306c21527a5
&scope=9ada6f8a-6d83-41bc-b169-a306c21527a5/.default
&redirect_uri=https%3A%2F%2Flocalhost
&state=1234
```

Este ejemplo de código genera una página de consentimiento para todos los permisos registrados si las descripciones anteriores de consentimiento y `/.default` se aplican al escenario. A continuación, el código devuelve un `id_token`, en lugar de un token de acceso.  

Este comportamiento se ajusta a algunos clientes heredados que se mueven de la Biblioteca de autenticación de Azure AD (ADAL) a la biblioteca de autenticación de Microsoft (MSAL). Los nuevos clientes que tienen como destino la Plataforma de identidad de Microsoft *no deben* utilizar esta configuración.

### <a name="client-credentials-grant-flow-and-default"></a>Flujo de concesión de credenciales de cliente y ./default  

`/.default` también se utiliza para solicitar permisos de aplicación o *roles* en una aplicación no interactiva, como una aplicación de demonio que usa el flujo de concesión de [credenciales de cliente](v2-oauth2-client-creds-grant-flow.md) para llamar a una API web.

Para crear permisos de aplicación (roles) para una API web, vea [Incorporación de roles de aplicación a una aplicación y su recepción en el token](howto-add-app-roles-in-azure-ad-apps.md).

Las solicitudes de credenciales de cliente en la aplicación cliente *deben* incluir `scope={resource}/.default`. Aquí, `{resource}` es la API web a la que la aplicación tiene previsto llamar. *No* se admite la emisión de una solicitud de credenciales de cliente con permisos de aplicación individuales (roles). En el token de acceso devuelto se incluyen todos los permisos de aplicación (roles) que se han concedido para esa API web.

Para conceder acceso a los permisos de aplicación que defina, incluida la concesión de consentimiento de administrador para la aplicación, vea [Configuración de una aplicación cliente para acceder a las API web](quickstart-configure-app-access-web-apis.md).

### <a name="trailing-slash-and-default"></a>Barra diagonal final y /.default

Algunos identificadores URI de recursos tienen una barra diagonal final, por ejemplo, `https://contoso.com/` en lugar de `https://contoso.com`. La barra diagonal final puede producir problemas con la validación de tokens. Los problemas se producen principalmente cuando se solicita un token para Azure Resource Manager (`https://management.azure.com/`). En este caso, una barra diagonal final en el identificador URI del recurso significa que la barra diagonal debe estar presente cuando se solicita el token.  Por lo tanto, cuando se solicita un token para `https://management.azure.com/` y se usa `/.default`, debe solicitar `https://management.azure.com//.default` (tenga en cuenta la barra diagonal doble). En general, si verifica que se está emitiendo el token y la API que debe aceptarlo lo está rechazando, considere la posibilidad de agregar una segunda barra diagonal y vuelva a intentarlo. 

## <a name="troubleshooting-permissions-and-consent"></a>Solución de problemas con permisos y consentimientos

Para aplicar los pasos de solución de problemas, vea [Error inesperado al otorgar consentimiento a una aplicación](../manage-apps/application-sign-in-unexpected-user-consent-error.md).

## <a name="next-steps"></a>Pasos siguientes

* [Tokens de id. de la Plataforma de identidad de Microsoft](id-tokens.md)
* [Tokens de acceso de la Plataforma de identidad de Microsoft](access-tokens.md)
