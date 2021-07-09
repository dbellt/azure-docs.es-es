---
title: Creación de un registro de aplicación (portal)
titleSuffix: Azure Digital Twins
description: Vea cómo se crea un registro de aplicaciones de Azure AD, como opción de autenticación para aplicaciones cliente, mediante Azure Portal.
author: baanders
ms.author: baanders
ms.date: 10/13/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 17f671a945c569d5492f84967a81931606d4cea1
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110989553"
---
# <a name="create-an-app-registration-to-use-with-azure-digital-twins-portal"></a>Creación de un registro de aplicaciones para usar con Azure Digital Twins (portal)

[!INCLUDE [digital-twins-create-app-registration-selector.md](../../includes/digital-twins-create-app-registration-selector.md)]

Al trabajar con una instancia de Azure Digital Twins, es habitual interactuar con ella mediante aplicaciones cliente, como una aplicación cliente personalizada que se compila en el artículo [Tutorial: Programación con las API de Azure Digital Twins](tutorial-code.md). Dichas aplicaciones deben autenticarse con Azure Digital Twins para interactuar con él, y algunos de los [mecanismos de autenticación](how-to-authenticate-client.md) que las aplicaciones pueden usar conllevan un **registro de aplicaciones** de [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md).

Esto no es necesario en todos los escenarios de autenticación. Sin embargo, si va a usar una estrategia de autenticación o un ejemplo de código que requiera un registro de aplicaciones, en este artículo se muestra cómo configurar uno mediante [Azure Portal](https://portal.azure.com). También se explica cómo [recopilar valores importantes](#collect-important-values) necesarios para usar el registro de aplicaciones para realizar la autenticación.

## <a name="azure-ad-app-registrations"></a>Registros de aplicaciones de Azure AD

[Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md) es un servicio de administración de identidades y acceso basado en la nube de Microsoft. La configuración de un **registro de aplicaciones** en Azure AD es una manera de conceder acceso a una aplicación cliente a Azure Digital Twins.

Este registro de la aplicación es donde se configuran los permisos de acceso a las [API de Azure Digital Twins](concepts-apis-sdks.md). Más tarde, las aplicaciones cliente se pueden autenticar en el registro de aplicaciones con los **valores de identificador de cliente y de inquilino** y, como resultado, que se le concedan permisos de acceso a las API.

>[!TIP]
> Es posible que prefiera configurar un nuevo registro de aplicaciones cada vez que lo necesite, *o*, hacer esto solo una vez, y establecer un solo registro de aplicaciones que se compartirá entre todos los escenarios que lo requieran.

## <a name="create-the-registration"></a>Creación del registro

Empiece por ir a [Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) en Azure Portal (puede usar este vínculo o buscar en la barra de búsqueda del portal). Seleccione *Registros de aplicaciones* en el menú del servicio y, luego, *+ Nuevo registro*.

:::image type="content" source="media/how-to-create-app-registration/new-registration.png" alt-text="Captura de pantalla de página del servicio de Azure AD en Azure Portal, en la que se muestran los pasos necesarios para crear un registro en la página &quot;Registros de aplicaciones&quot;.":::

En la página *Registrar una aplicación* que sigue, rellene los valores solicitados:
* **Name**: nombre para mostrar de una aplicación de Azure AD para asociar al registro.
* **Tipos de cuenta admitidos**: seleccione *Cuentas de este directorio organizativo solo (Solo directorio predeterminado: inquilino único)*
* **URI de redirección**: *URL de respuesta de aplicación de Azure AD* para la aplicación de Azure AD. Agregue un URI *Cliente público o nativo (móvil o escritorio)* para `http://localhost`.

Cuando termine, pulse el botón *Registrar*.

:::image type="content" source="media/how-to-create-app-registration/register-an-application.png" alt-text="Captura de pantalla de la página &quot;Registrar una aplicación&quot; de Azure Portal con los valores descritos rellenados.":::

Cuando termine la configuración del registro, el portal lo redirigirá a su página de detalles.

## <a name="collect-important-values"></a>Recopilación de valores importantes

A continuación, recopile algunos valores importantes sobre el registro de la aplicación que necesitará para usar el registro para autenticar una aplicación cliente. Estos valores incluyen:
* **nombre del recurso**
* **identificador del cliente**
* **identificador del inquilino**
* **secreto del cliente**

Para trabajar con Azure Digital Twins, el **nombre del recurso** es `http://digitaltwins.azure.net`.

En las secciones siguientes se describe cómo buscar los restantes valores.

### <a name="collect-client-id-and-tenant-id"></a>Recopilación de los identificadores de cliente y de inquilino

Los valores de **identificador de cliente** e **identificador de inquilino** se pueden recopilar de la página de detalles del registro de la aplicación en Azure Portal:

:::image type="content" source="media/how-to-create-app-registration/client-id-tenant-id.png" alt-text="Captura de pantalla de Azure Portal en la que se muestran los valores importantes para el registro de la aplicación.":::

Tome nota del **Id. de la aplicación (cliente)** y el **Id. de directorio (inquilino)** , como se muestra en **su** página.

### <a name="collect-client-secret"></a>Recopilación del secreto de cliente

Para configurar un **secreto de cliente** para un registro de aplicación, lo primero que hay que hacer es dirigirse a la página del registro de la aplicación en Azure Portal. 

1. Seleccione **Certificados y secretos** en el menú del registro y, después, **+ Nuevo secreto de cliente**.

    :::image type="content" source="media/how-to-create-app-registration/client-secret.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra un registro de aplicación de Azure AD y se resalta &quot;Nuevo secreto de cliente&quot;.":::

1. Escriba los valores que desee en Descripción y Caduca, y seleccione **Agregar**.

    :::image type="content" source="media/how-to-create-app-registration/add-client-secret.png" alt-text="Captura de pantalla de Azure Portal al agregar un secreto de cliente.":::

1. Compruebe que el secreto de cliente está visible en la página **Certificados y secretos** con los campos Caduca y Valor. 

1. Tome nota de los valores de **Secret ID** (Identificador de secreto) y **Value** (Valor) para usarlo más adelante (también puede copiarlos en el Portapapeles con los iconos de Copia).

    :::image type="content" source="media/how-to-create-app-registration/client-secret-value.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra cómo copiar el valor del secreto de cliente.":::

>[!IMPORTANT]
>Asegúrese de copiar los valores ahora y almacenarlos en un lugar seguro, ya que no se pueden volver a recuperar. Si no los encuentra posteriormente, tendrá que crear un secreto.

## <a name="provide-azure-digital-twins-api-permission"></a>Concesión del permiso de API de Azure Digital Twins

A continuación, configure el registro de la aplicación que ha creado con permisos de línea base para las API de Azure Digital Twins.

En la página del portal de registro de aplicación, seleccione *Permisos de API* en el menú. En la siguiente página de permisos, seleccione el botón *+ Agregar un permiso*.

:::image type="content" source="media/how-to-create-app-registration/add-permission.png" alt-text="Captura de pantalla del registro de aplicación en Azure Portal que resalta la opción del menú &quot;Permisos de API&quot; y el botón &quot;Agregar un permiso&quot;.":::

En la página *Solicitud de permisos de API* que sigue, cambie a la pestaña *API usadas en mi organización* y busque *Azure Digital Twins*. Seleccione _**Azure Digital Twins**_ en los resultados de la búsqueda para continuar con la asignación de permisos para las API de Azure Digital Twins.

:::image type="content" source="media/how-to-create-app-registration/request-api-permissions-1.png" alt-text="Captura de pantalla del resultado de la búsqueda de la página &quot;Solicitar permisos de API&quot; Azure Portal que muestra Azure Digital Twins.":::

>[!NOTE]
> Si la suscripción todavía tiene una instancia existente de Azure Digital Twins de la versión preliminar pública anterior del servicio (antes de julio de 2020), deberá en cambio buscar y seleccionar _**Azure Smart Spaces Service**_. Se trata de un nombre anterior para el mismo conjunto de API (observe que el *id. de aplicación (cliente)* es el mismo que en la captura de pantalla anterior) y su experiencia no se modificará más allá de este paso.
> :::image type="content" source="media/how-to-create-app-registration/request-api-permissions-1-smart-spaces.png" alt-text="Captura de pantalla del resultado de la búsqueda de la página &quot;Solicitud de permisos de API&quot; que muestra Azure Smart Spaces Service en Azure Portal.":::

A continuación, seleccione los permisos que quiere conceder para estas API. Expanda el permiso **Lectura (1)** y marque la casilla que indica *Read.Write* para conceder permisos de lectura y escritura para el registro de la aplicación.

:::image type="content" source="media/how-to-create-app-registration/request-api-permissions-2.png" alt-text="Captura de pantalla de la página &quot;Solicitud de permisos de API&quot; y selección de permisos &quot;Read.Write&quot; para las API de Azure Digital Twins en Azure Portal.":::

Cuando termine, seleccione *Agregar permisos*.

### <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

En la página *Permisos de API*, compruebe que haya ahora una entrada para Azure Digital Twins que refleje los permisos de lectura y escritura:

:::image type="content" source="media/how-to-create-app-registration/verify-api-permissions.png" alt-text="Captura de pantalla de los permisos de API para el registro de la aplicación de Azure AD en Azure Portal en la que se muestra &quot;Acceso de lectura y escritura&quot; para Azure Digital Twins.":::

También puede comprobar la conexión a Azure Digital Twins en el archivo *manifest.json* del registro de la aplicación, que se actualizó automáticamente con la información de Azure Digital Twins cuando agregó los permisos de API.

Para ello, seleccione **Manifiesto** en el menú para ver el código del manifiesto del registro de la aplicación. Desplácese hasta la parte inferior de la ventana de código y busque los siguientes campos y valores en `requiredResourceAccess`: 
* `"resourceAppId": "0b07f429-9f4b-4714-9392-cc5e8e80c8b0"`
* `"resourceAccess"` > `"id": "4589bd03-58cb-4e6c-b17f-b580e39652f8"`

Estos valores se muestran en la siguiente captura de pantalla:

:::image type="content" source="media/how-to-create-app-registration/verify-manifest.png" alt-text="Captura de pantalla del manifiesto del registro de la aplicación de Azure AD en Azure Portal.":::

Si faltan estos valores, vuelva a intentar los pasos de la [sección para agregar el permiso de API](#provide-azure-digital-twins-api-permission).

## <a name="other-possible-steps-for-your-organization"></a>Otros posibles pasos para la organización

Es posible que su organización requiera otras acciones por parte de los propietarios o administradores de las suscripciones para configurar correctamente un registro de aplicaciones. Los pasos necesarios pueden variar en función de la configuración específica de su organización.

Estas son algunas posibles actividades comunes que un propietario o administrador de la suscripción puede tener que hacer. Estas y otras operaciones se pueden realizar desde la página [Registros de aplicaciones de Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) en Azure Portal.
* Conceda el consentimiento del administrador para el registro de aplicaciones. Es posible que la organización tenga habilitado globalmente el *Consentimiento del administrador necesario* en Azure AD para todos los registros de aplicaciones dentro de la suscripción. Si es así, el propietario o el administrador tendrán que seleccionar este botón para la empresa en la página *Permisos de API* del registro de aplicaciones para que este sea válido:

    :::image type="content" source="media/how-to-create-app-registration/grant-admin-consent.png" alt-text="Captura de pantalla de Azure Portal que muestra el botón &quot;Conceder consentimiento del administrador&quot; en Permisos de API.":::
  - Si el consentimiento se concedió correctamente, la entrada de Azure Digital Twins debería mostrar un valor *Estado* de _Concedido para **(su empresa)**_
   
    :::image type="content" source="media/how-to-create-app-registration/granted-admin-consent-done.png" alt-text="Captura de pantalla de Azure Portal que muestra el consentimiento del administrador concedida en Permisos de API.":::
* Activar el acceso de cliente público
* Establecer direcciones URL de respuesta específicas para el acceso web y de escritorio
* Permitir flujos de autenticación de OAuth2 implícitos

Para más información sobre el registro de aplicaciones y sus distintas opciones de configuración, consulte [Registro de una aplicación en la plataforma de identidad de Microsoft](/graph/auth-register-app-v2).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha configurado un registro de aplicaciones de Azure AD que se puede usar para autenticar las aplicaciones cliente con las API de Azure Digital Twins.

A continuación, lea sobre los mecanismos de autenticación, tanto los que usan registros de aplicaciones como los que no:
* [Procedimiento: Escritura de código de autenticación de aplicación](how-to-authenticate-client.md)