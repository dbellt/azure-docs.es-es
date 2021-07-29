---
title: Identidades administradas en Azure SignalR Service
description: Obtenga información sobre cómo funcionan las identidades administradas en Azure SignalR Service y sobre cómo usar una identidad administrada en escenarios sin servidor.
author: chenyl
ms.service: signalr
ms.topic: article
ms.date: 06/8/2020
ms.author: chenyl
ms.openlocfilehash: 7f39e209cf2f01abaf836924fc25dc64275f5fcb
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110089819"
---
# <a name="managed-identities-for-azure-signalr-service"></a>Identidades administradas para Azure SignalR Service

En este artículo se explica cómo crear una identidad administrada para Azure SignalR Service y cómo usarla en escenarios sin servidor.

> [!Important] 
> Azure SignalR Service solo admite una identidad administrada. Esto significa que puede agregar una identidad asignada por el sistema o asignada por el usuario. 

## <a name="add-a-system-assigned-identity"></a>Adición de una identidad asignada por el sistema

Para configurar una identidad administrada en Azure Portal, primero tiene que crear una instancia de Azure SignalR Service y después habilitar la característica.

1. Cree una instancia de Azure SignalR Service en el portal como lo haría normalmente. Búsquela en el portal.

2. Seleccione **Identidad**.

4. En la pestaña **Asignado por el sistema**, cambie **Estado** a **Activado**. Seleccione **Guardar**.

    :::image type="content" source="media/signalr-howto-use-managed-identity/system-identity-portal.png" alt-text="Adición de una identidad asignada por el sistema en el portal":::

## <a name="add-a-user-assigned-identity"></a>Adición de una identidad asignada por el usuario

La creación de una instancia de Azure SignalR Service con una identidad asignada por el usuario requiere que se cree la identidad y luego se agregue su identificador de recurso al servicio.

1. Cree un recurso de identidad administrada asignada por el usuario según [estas instrucciones](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity).

2. Cree una instancia de Azure SignalR Service en el portal como lo haría normalmente. Búsquela en el portal.

3. Seleccione **Identidad**.

4. En la pestaña **Usuario asignado**, seleccione **Agregar**.

5. Busque la identidad que creó anteriormente y selecciónela. Seleccione **Agregar**.

    :::image type="content" source="media/signalr-howto-use-managed-identity/user-identity-portal.png" alt-text="Adición de una identidad asignada por el usuario en el portal":::

## <a name="use-a-managed-identity-in-serverless-scenarios"></a>Uso de una identidad administrada en escenarios sin servidor

Azure SignalR Service es un servicio totalmente administrado, por lo que no puede usar una identidad administrada para obtener tokens manualmente. En cambio, Azure SignalR Service usa la identidad administrada que se establece para obtener un token de acceso. Después, el servicio establece el token de acceso en un encabezado `Authorization` de una solicitud ascendente en escenarios sin servidor.

### <a name="enable-managed-identity-authentication-in-upstream-settings"></a>Habilitar la autenticación de la identidad administrada en la configuración ascendente

1. Agregue una identidad asignada por el sistema o una asignada por el usuario.

2. Agregue una configuración ascendente y haga clic en cualquier asterisco para entrar en una página detallada, como se muestra a continuación.
    :::image type="content" source="media/signalr-howto-use-managed-identity/pre-msi-settings.png" alt-text="pre-msi-setting":::
    
    :::image type="content" source="media/signalr-howto-use-managed-identity/msi-settings.png" alt-text="msi-setting":::

3. En la configuración de autenticación de la identidad administrada, puede especificar el recurso de destino en **Recurso**. El recurso se convertirá en una notificación `aud` en el token de acceso obtenido, que se puede usar como parte de la validación en los puntos de conexión ascendentes. El recurso puede ser uno de los siguientes:
    - Vacío
    - Identificador de la aplicación (cliente) de la entidad de servicio
    - URI del identificador de la aplicación de la entidad de servicio
    - [Identificador de recursos de un servicio de Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication)

    > [!NOTE]
    > Si valida un token de acceso por sí mismo en el servicio, puede elegir cualquier formato para el recurso. Solo tiene que asegurarse de que el valor **Recurso** de la configuración de **Auth** sea coherente con la validación. Si usa el control de acceso basado en roles de Azure para un plano de datos, debe usar el recurso que solicite el proveedor de servicios.

### <a name="validate-access-tokens"></a>Validación de los tokens de acceso

El token del encabezado `Authorization` es un [token de acceso plataforma de identidades de Microsoft](../active-directory/develop/access-tokens.md#validating-tokens).

Para validar los tokens de acceso, la aplicación también debe validar la audiencia y los tokens de firmas. Deben validarse con los valores del documento de detección de OpenID. Por ejemplo, fijémonos en la [versión independiente del inquilino del documento](https://login.microsoftonline.com/common/.well-known/openid-configuration).

El middleware de Azure Active Directory (Azure AD) tiene capacidades integradas para validar los tokens de acceso. Puede examinar nuestros [ejemplos](../active-directory/develop/sample-v2-code.md) para encontrar uno en el idioma que elija.

Proporcionamos bibliotecas y ejemplos de código que le muestran cómo controlar la validación de tokens. También hay varias bibliotecas de asociados de código abierto disponibles para la validación de JSON Web Token (JWT). Al menos hay una opción para casi cualquier plataforma y lenguaje. Para obtener más información acerca de los ejemplos de código y las bibliotecas de autenticación de Azure AD, vea [Bibliotecas de autenticación de la plataforma de identidad de Microsoft](../active-directory/develop/reference-v2-libraries.md).

#### <a name="authentication-in-function-app"></a>Autenticación en la aplicación de funciones

La configuración de la validación de token de acceso en la aplicación de funciones es fácil y eficaz sin que el código funcione.

1. En la página **Autenticación (clásica)** , cambie **Autenticación de App Service** a **Activado**.

2. Seleccione **Log in with Azure Active Directory** (Iniciar sesión con Azure Active Directory) en **Action to take when request is not authenticated** (Acción necesaria cuando la solicitud no está autenticada).

3. En el proveedor de autenticación, haga clic en **Azure Active Directory**

4. En la nueva página. Seleccione **Rápido** y **Crear nueva aplicación de AD** y, a continuación, haga clic en **Aceptar** :::image type="content" source="media/signalr-howto-use-managed-identity/function-aad.png" alt-text="Aplicación de funciones":::

5. Vaya a SignalR Service y siga los [pasos](howto-use-managed-identity.md#add-a-system-assigned-identity) para agregar una identidad asignada por el sistema o una asignada por el usuario.

6. Adéntrese en la **configuración ascendente** de SignalR Service y elija **Uso de identidad administrada** y **Select from existing Applications** (Seleccionar una de las aplicaciones existentes). Seleccione la aplicación que ha creado anteriormente.

Después de esta configuración, la aplicación de funciones rechazará las solicitudes sin un token de acceso en el encabezado.

> [!Important] 
> Para pasar la autenticación, la dirección *URL del emisor* debe coincidir con la notificación *iss* del token. Actualmente, solo se admite el punto de conexión v1 (vea [v1.0 y v2.0](../active-directory/develop/access-tokens.md#v10-and-v20)), por lo que la dirección *URL del emisor* debe parecerse a `https://sts.windows.net/<tenant-id>/`. Compruebe la dirección *URL del emisor* configurada en la instancia de Azure Functions. En **Autenticación**, vaya a *Proveedor de identidades* -> *Editar* -> *URL del emisor* y en **Autenticación (clásica)** , vaya a *Azure Active Directory* -> *Opciones avanzadas* -> *URL del emisor*.


## <a name="use-a-managed-identity-for-key-vault-reference"></a>Uso de una identidad administrada para la referencia de Key Vault

SignalR Service puede tener acceso a Key Vault para obtener el secreto mediante la identidad administrada.

1. Agregue una identidad asignada por el sistema o una identidad asignada por el usuario para Azure SignalR Service.

2. Conceda el permiso de lectura de secreto para la identidad administrada en las directivas de acceso de Key Vault. Consulte [Asignación de una directiva de acceso de Key Vault mediante Azure Portal](../key-vault/general/assign-access-policy-portal.md)

Actualmente, esta característica se puede usar en los escenarios siguientes:

- [Secreto de referencia en el patrón de dirección URL ascendente](./concept-upstream.md#key-vault-secret-reference-in-url-template-settings)


## <a name="next-steps"></a>Pasos siguientes

- [Desarrollo y configuración de Azure Functions con Azure SignalR Service](signalr-concept-serverless-development-config.md)
