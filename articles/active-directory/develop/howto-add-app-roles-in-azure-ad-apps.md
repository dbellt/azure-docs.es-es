---
title: Incorporación de roles de aplicación y su obtención desde un token | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo agregar roles de aplicación en una aplicación registrada en Azure Active Directory, asignar usuarios y grupos a estos roles y recibirlos en la notificación "roles" del token.
services: active-directory
author: kalyankrishna1
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 05/06/2021
ms.author: kkrishna
ms.reviewer: marsma, kkrishna, jmprieur
ms.custom: aaddev
ms.openlocfilehash: cf1ca72d7afaace847fb1b8321723f57fbe133f6
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108802184"
---
# <a name="how-to-add-app-roles-to-your-application-and-receive-them-in-the-token"></a>Procedimiento: Incorporación de roles de aplicación a una aplicación y su recepción en el token

El control de acceso basado en rol (RBAC) es un mecanismo popular para exigir la autorización en las aplicaciones. Cuando se usa RBAC, un administrador concede permisos a roles y no a usuarios individuales o grupos. El administrador, a continuación, puede asignar roles a diferentes usuarios y grupos para controlar quién tiene acceso a qué contenido y funcionalidad.

Mediante el uso de RBAC con roles de aplicación y notificaciones de rol, los desarrolladores pueden exigir de forma segura la autorización en las aplicaciones con menos esfuerzo.

Otro enfoque consiste en usar grupos de Azure AD y notificaciones de grupo, como se muestra en el ejemplo de código [active-directory-aspnetcore-webapp-openidconnect-v2](https://aka.ms/groupssample) en GitHub. Los grupos de Azure AD y los roles de aplicación no son mutuamente excluyentes y se pueden usar en conjunto para proporcionar un control de acceso aún más preciso.

## <a name="declare-roles-for-an-application"></a>Declaración de roles para una aplicación

Los roles de aplicación se definen en [Azure Portal](https://portal.azure.com). Los roles de aplicación se definen normalmente en un registro de aplicación que representa un servicio, una aplicación o una API. Cuando un usuario inicia sesión en la aplicación, Azure AD emite una notificación `roles` por cada rol que se ha concedido al usuario o entidad de servicio de manera individual y por su pertenencia a grupos. Se puede usar para implementar una autorización basada en notificaciones. Los roles de aplicación se pueden asignar [a un usuario o a un grupo de usuarios](../manage-apps/add-application-portal-assign-users.md#assign-users-to-an-app). Los roles de aplicación también se pueden asignar a la entidad de servicio de otra aplicación o [a la entidad de servicio de una identidad administrada](../managed-identities-azure-resources/how-to-assign-app-role-managed-identity-powershell.md).

> [!IMPORTANT]
> Actualmente, si agrega una entidad de servicio a un grupo y, a continuación, asigna un rol de aplicación a ese grupo, Azure AD no agrega la notificación `roles` a los tokens que emite.

Hay dos maneras de declarar los roles de aplicación mediante Azure Portal:

- [Interfaz de usuario de roles de aplicación](#app-roles-ui)
- [Editor de manifiestos de la aplicación](#app-manifest-editor)

La cantidad de roles que agrega está restringida por los límites de los manifiestos de la aplicación que impone Azure Active Directory. Para obtener información sobre estos límites, consulte la sección [Límites de manifiesto](./reference-app-manifest.md#manifest-limits) de la referencia del [manifiesto de aplicación de Azure Active Directory](reference-app-manifest.md).

### <a name="app-roles-ui"></a>Interfaz de usuario de roles de aplicación

Para crear un rol de aplicación con la interfaz de usuario de Azure Portal:

1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
1. En el menú superior, seleccione el filtro **Directorio + suscripción** y, a continuación, elija el inquilino de Azure Active Directory que contiene el registro de la aplicación al que desea agregar un rol de aplicación.
1. Busque y seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Registros de aplicaciones** y, a continuación, seleccione la aplicación en la que desea definir los roles de aplicación.
1. Seleccione **Roles de aplicación** y, a continuación, seleccione **Crear rol de aplicación**.

   :::image type="content" source="media/howto-add-app-roles-in-azure-ad-apps/app-roles-overview-pane.png" alt-text="Panel de roles de aplicación durante el registro de una aplicación en Azure Portal":::

1. En el panel **Crear rol de aplicación**, escriba la configuración del rol. La tabla que sigue a la imagen describe cada configuración y sus parámetros.

   :::image type="content" source="media/howto-add-app-roles-in-azure-ad-apps/app-roles-create-context-pane.png" alt-text="Panel contextual de creación de roles de aplicación durante el registro de una aplicación en Azure Portal":::

   | Campo                                    | Descripción                                                                                                                                                                                                                                                                                                       | Ejemplo                       |
   | ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- |
   | **Nombre para mostrar**                         | Nombre para mostrar para el rol de aplicación que aparece en las experiencias de asignación de la aplicación y de consentimiento del administrador. Este valor puede incluir espacios.                                                                                                                                                                                    | `Survey Writer`               |
   | **Tipos de miembros permitido**                 | Especifica si este rol de aplicación puede asignarse a usuarios, aplicaciones o ambos.<br/><br/>Cuando está disponible para `applications`, los roles de aplicación aparecen como permisos de aplicación en la sección **Administrar** > **Permisos de API > Agregar un permiso > Mis API > Elegir una API > Permisos de la aplicación** de un registro de aplicación. | `Users/Groups`                |
   | **Valor**                                | Especifica el valor de la notificación de roles que debería esperar la aplicación en los tokens de acceso y autenticación. El valor debe coincidir exactamente con la cadena a la que se hace referencia en el código de la aplicación. El valor no puede contener espacios.                                                                                                          | `Survey.Create`               |
   | **Descripción**                          | Una descripción más detallada del rol de aplicación que se muestra durante la asignación de aplicaciones de administración y las experiencias de consentimiento.                                                                                                                                                                                                        | `Writers can create surveys.` |
   | **¿Quiere habilitar este rol de aplicación?** | Especifica si está habilitado el rol de aplicación. Para eliminar un rol de aplicación, desactive esta casilla y aplique el cambio antes de intentar la operación de eliminación.                                                                                                                                                             | _Activada_                     |

1. Seleccione **Aplicar** para guardar los cambios.

### <a name="app-manifest-editor"></a>Editor de manifiestos de la aplicación

Para agregar roles mediante la edición directa del manifiesto:

1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
1. En el menú superior, seleccione el filtro **Directorio + suscripción** y, a continuación, elija el inquilino de Azure Active Directory que contiene el registro de la aplicación al que desea agregar un rol de aplicación.
1. Busque y seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Registros de aplicaciones** y, a continuación, seleccione la aplicación en la que desea definir los roles de aplicación.
1. Nuevamente en **Administrar**, seleccione **Manifiesto**.
1. Para editar el manifiesto de la aplicación, localice la opción de configuración `appRoles` y agregue sus roles de aplicación. Puede definir roles de aplicación dirigidos a `users`, `applications` o ambos. Los fragmentos de código JSON siguientes muestran ejemplos de ambas opciones.
1. Guarde el manifiesto.

Cada definición de rol de aplicación en el manifiesto debe tener un GUID único para su valor `id`.

La propiedad `value` de cada definición de rol de la aplicación debe coincidir exactamente con las cadenas que se usan en el código de la aplicación. La propiedad `value` no puede contener espacios. Si es así, recibirá un error al guardar el manifiesto.

#### <a name="example-user-app-role"></a>Ejemplo: Rol de aplicación de un usuario

En este ejemplo se define un rol de aplicación denominado `Writer` que puede asignar a un `User`:

```json
"appId": "8763f1c4-0000-0000-0000-158e9ef97d6a",
"appRoles": [
    {
      "allowedMemberTypes": [
        "User"
      ],
      "displayName": "Writer",
      "id": "d1c2ade8-0000-0000-0000-6d06b947c66f",
      "isEnabled": true,
      "description": "Writers Have the ability to create tasks.",
      "value": "Writer"
    }
  ],
"availableToOtherTenants": false,
```

#### <a name="example-application-app-role"></a>Ejemplo: Rol de aplicación de una aplicación

Cuando está disponible para `applications`, los roles de aplicación aparecen como permisos de aplicación en la sección **Administrar** > **Permisos de API > Agregar un permiso > Mis API > Elegir una API > Permisos de la aplicación** de un registro de aplicación.

En este ejemplo se muestra un rol de aplicación dirigido a una `Application`:

```json
"appId": "8763f1c4-0000-0000-0000-158e9ef97d6a",
"appRoles": [
    {
      "allowedMemberTypes": [
        "Application"
      ],
      "displayName": "ConsumerApps",
      "id": "47fbb575-0000-0000-0000-0f7a6c30beac",
      "isEnabled": true,
      "description": "Consumer apps have access to the consumer data.",
      "value": "Consumer"
    }
  ],
"availableToOtherTenants": false,
```

## <a name="assign-users-and-groups-to-roles"></a>Asignación de usuarios y grupos a roles

Una vez que haya agregado los roles de aplicación en la aplicación, puede asignar usuarios y grupos a estos roles. La asignación de usuarios y grupos a los roles se puede realizar mediante la interfaz de usuario del portal o mediante programación con [Microsoft Graph](/graph/api/user-post-approleassignments). Cuando los usuarios asignados a los distintos roles de aplicación inicien sesión en la aplicación, los tokens tendrán sus roles asignados en la notificación `roles`.

Para asignar usuarios y grupos a roles con Azure Portal:

1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
1. En **Azure Active Directory**, seleccione **Aplicaciones empresariales** en el menú de navegación de la izquierda.
1. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones. Si la aplicación no aparece en la lista, use los filtros de la parte superior de la lista **Todas las aplicaciones** para restringir la lista o desplácese hacia abajo en la lista para localizar la aplicación.
1. Seleccione la aplicación en la que desea asignar usuarios o grupos de seguridad a los roles.
1. En **Administrar**, seleccione **Usuarios y grupos**.
1. Seleccione **Agregar miembro** para abrir el panel **Agregar asignación**.
1. Seleccione el selector **Usuarios y grupos** del panel **Agregar asignación**. Se muestra una lista de usuarios y grupos de seguridad. Puede buscar un usuario o grupo determinados y seleccionar varios usuarios y grupos que aparecen en la lista.
1. Una vez que haya seleccionado usuarios y grupos, seleccione el botón **Seleccionar** para continuar.
1. Elija **Seleccionar un rol** en el panel **Agregar asignación**. Se muestran todos los roles que ha definido para la aplicación.
1. Elija un rol y seleccione el botón **Seleccionar**.
1. Seleccione el botón **Asignar** para finalizar las asignaciones de usuarios y grupos a la aplicación.

Confirme que los usuarios y grupos que agregó aparecen en la lista **Usuarios y grupos**.

## <a name="assign-app-roles-to-applications"></a>Asignación de roles de aplicación a aplicaciones

Una vez que haya agregado roles de aplicación en la aplicación, puede asignar un rol de aplicación a una aplicación cliente a través de Azure Portal o mediante programación con [Microsoft Graph](/graph/api/user-post-approleassignments).

Cuando asigna roles de aplicación a una aplicación, crea _permisos de aplicación_. Los permisos de aplicación se suelen usar en aplicaciones de demonio o servicios de back-end que necesitan autenticarse y hacer llamadas API autorizadas como ellos mismos, sin la interacción de un usuario.

Para asignar roles de aplicación a una aplicación mediante Azure Portal:

1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
1. En **Azure Active Directory**, seleccione **Registros de aplicaciones** en el menú de navegación de la izquierda.
1. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones. Si la aplicación no aparece en la lista, use los filtros de la parte superior de la lista **Todas las aplicaciones** para restringir la lista o desplácese hacia abajo en la lista para localizar la aplicación.
1. Seleccione la aplicación a la que quiera asignar un rol de aplicación.
1. Seleccione **Permisos de API** > **Agregar un permiso**.
1. Seleccione la pestaña **Mis API** y, luego, la aplicación para la que definió los roles de aplicación.
1. Seleccione **Permisos de aplicación**.
1. Seleccione los roles que quiere asignar.
1. Seleccione el botón **Agregar permisos** para terminar de agregar los roles.

Los roles que acaba de agregar debe aparecer en el panel **Permisos de API** del registro de la aplicación.

#### <a name="grant-admin-consent"></a>Concesión de consentimiento del administrador

Dado que se trata de _permisos de aplicación_ y no de permisos delegados, un administrador debe dar su consentimiento para usar los roles de aplicación asignados a la aplicación.

1. En el panel **Permisos de API** del registro de la aplicación, seleccione **Conceder consentimiento del administrador para \<tenant name\>** .
1. Seleccione **Sí** cuando se le pida conceder consentimiento para los permisos solicitados.

La columna **Estado** debe reflejar que hay consentimiento **Concedido para \<tenant name\>** .

## <a name="use-app-roles-in-your-web-api"></a>Uso de roles de aplicación en la API web

Una vez que define los roles de aplicación y los asigne a un usuario, grupo o aplicación, el paso siguiente es agregar código a la API web que comprueba esos roles cuando se llama a la API. Es decir, cuando una aplicación cliente solicita una operación de API que decidió que requiere autorización, el código de la API debe comprobar que los ámbitos están en el token de acceso que se presenta en la llamada de la aplicación cliente.

Para obtener información sobre cómo agregar autorización a la API web, consulte [API web protegida: Comprobación de ámbitos y roles de aplicación](scenario-protected-web-api-verification-scope-app-roles.md).

## <a name="app-roles-vs-groups"></a>Roles de aplicación frente a grupos

Si bien puede usar roles de aplicación o grupos para la autorización, las diferencias clave entre ambos pueden influir en la opción que decida usar para el escenario en cuestión.

| Roles de aplicación                                                                                                    | Grupos                                                      |
| ------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| Son específicos de una aplicación y se definen en el registro de la aplicación. Se mueven con la aplicación. | No son específicos de una aplicación, sino de un inquilino de Azure AD. |
| Los roles de aplicación se quitan cuando se quita el registro de la aplicación.                                                | Los grupos permanecen intactos incluso si se quita la aplicación.            |
| Se proporcionan en la notificación `roles`.                                                                               | Se proporcionan en la notificación `groups`.                                 |

Los desarrolladores pueden usar los roles de aplicación para controlar si un usuario puede iniciar sesión en una aplicación o si una aplicación puede obtener un token de acceso para una API web. Para extender este control de seguridad a los grupos, los desarrolladores y administradores también pueden asignar grupos de seguridad a los roles de aplicación.

Los desarrolladores prefieren los roles de aplicación cuando quieren describir y controlar los parámetros de autorización en sus propias aplicaciones. Por ejemplo, una aplicación que usa grupos para la autorización se interrumpirá en el inquilino siguiente, ya que tanto el identificador como el nombre del grupo podrían ser distintos. Una aplicación que usa roles de aplicación sigue siendo segura. De hecho, la asignación de grupos a roles de aplicación se usa mucho con las aplicaciones SaaS por las mismas razones.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre los roles de aplicación en los recursos siguientes.

- Ejemplos de código en GitHub
  - [Incorporación de autorización mediante grupos y notificaciones de grupos a una aplicación web de ASP.NET Core](https://aka.ms/groupssample)
  - [Artículo sobre Llamada a una API web de .NET Core desde una aplicación de página única (SPA) de Angular y uso de roles de aplicación y grupos de seguridad](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-dotnetcore-webapi-roles-groups/blob/master/README.md)
- Documentación de referencia
  - [Manifiesto de la aplicación de Azure AD](./reference-app-manifest.md)
  - [Tokens de acceso de Azure AD](access-tokens.md)
  - [Tokens de identificador de Azure AD](id-tokens.md)
  - [Proporcionar notificaciones opcionales a la aplicación](active-directory-optional-claims.md)
- Vídeo: [Implementación de la autorización en las aplicaciones con la plataforma de identidad de Microsoft](https://www.youtube.com/watch?v=LRoc-na27l0) (1:01:15)
