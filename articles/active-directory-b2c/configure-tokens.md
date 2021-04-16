---
title: 'Configuración de tokens: Azure Active Directory B2C | Microsoft Docs'
description: Obtenga información sobre cómo configurar las opciones de compatibilidad y duración de los tokens en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/05/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 0f5586b43143763ebf36adb15d96fdb2a91b5f5c
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106443481"
---
# <a name="configure-tokens-in-azure-active-directory-b2c"></a>Configuración de tokens en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

En este artículo, aprenderá a configurar la [duración y la compatibilidad de un token](tokens-overview.md) en Azure Active Directory B2C (Azure AD B2C).

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="token-lifetime-behavior"></a>Comportamiento de la vigencia de tokens

Puede configurar la duración de los tokens, entre otras cosas:

- **Acceso y duración del token de id. (minutos)** : vigencia del token de portador de OAuth 2.0 y los tokens de id. El valor predeterminado es de 60 minutos (1 hora). El mínimo (incluido) es de 5 minutos. El máximo (incluido) es de 1440 minutos (24 horas).
- **Duración del token de actualización (días)** : período máximo en que un token de actualización se puede utilizar para adquirir un nuevo token de acceso, si se hubiera concedido el ámbito `offline_access` a la aplicación. El valor predeterminado es 14 días. El mínimo (incluido) es de 1 día. El máximo (incluido) es de 90 días.
- **Actualizar duración de ventana deslizante de token**: tipo de ventana deslizante del token de actualización. `Bounded` indica que el token de actualización se puede extender según lo especificado en **Duración (días)** . `No expiry` indica que la vigencia de la ventana deslizante del token de actualización no expira nunca.
- **Duración (días)** : una vez que haya transcurrido este período, el usuario estará obligado a volver a autenticarse, independientemente del período de validez del token de actualización más reciente que haya adquirido la aplicación. El valor debe ser mayor o igual que el valor de **Vigencia del token de actualización**.

En el diagrama siguiente se muestra el comportamiento de duración de la ventana deslizante del token de actualización.

![Vigencia del token de actualización](./media/configure-tokens/refresh-token-lifetime.png)

> [!NOTE]
> >Las aplicaciones de página única que usan el flujo de código de autorización con PKCE siempre tienen una vigencia del token de actualización de 24 horas, mientras que las aplicaciones móviles, las aplicaciones de escritorio y las aplicaciones web no tienen esta limitación. [Más información sobre las consecuencias de seguridad de los tokens de actualización en el explorador](../active-directory/develop/reference-third-party-cookies-spas.md#security-implications-of-refresh-tokens-in-the-browser).


## <a name="configure-token-lifetime"></a>Configuración de la duración de los tokens

::: zone pivot="b2c-user-flow"

Para configurar la duración del token de flujo del usuario:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio y suscripción** del menú superior y el directorio que contiene el inquilino de Azure AD B2C.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. Seleccione **Flujos de usuario (directivas)** .
1. Abra el flujo de usuario que creó anteriormente.
1. Seleccione **Propiedades**.
1. En **Vigencia del token**, ajuste las propiedades para satisfacer las necesidades de su aplicación.
1. Haga clic en **Save**(Guardar).

::: zone-end

::: zone pivot="b2c-custom-policy"

Para cambiar la configuración de la compatibilidad de los tokens, establezca los metadatos del perfil técnico del [Emisor del token](jwt-issuer-technical-profile.md) en la extensión, o el archivo del usuario de confianza de la directiva a la que desea que afecte. El perfil técnico del emisor del token es similar al ejemplo siguiente:

```xml
<ClaimsProviders>
  <ClaimsProvider>
    <DisplayName>Token Issuer</DisplayName>
    <TechnicalProfiles>
      <TechnicalProfile Id="JwtIssuer">
        <Metadata>
          <Item Key="token_lifetime_secs">3600</Item>
          <Item Key="id_token_lifetime_secs">3600</Item>
          <Item Key="refresh_token_lifetime_secs">1209600</Item>
          <Item Key="rolling_refresh_token_lifetime_secs">7776000</Item>
          <!--<Item Key="allow_infinite_rolling_refresh_token">true</Item>-->
          <Item Key="IssuanceClaimPattern">AuthorityAndTenantGuid</Item>
          <Item Key="AuthenticationContextReferenceClaimPattern">None</Item>
        </Metadata>
      </TechnicalProfile>
    </TechnicalProfiles>
  </ClaimsProvider>
</ClaimsProviders>
```

Los valores siguientes se establecen en el ejemplo anterior:

- **token_lifetime_secs**: duración de los tokens de acceso (segundos). El valor predeterminado es de 3600 (1 hora). El mínimo es de 300 (5 minutos). El máximo es de 86 400 (24 horas). 
- **id_token_lifetime_secs**: duración de los tokens de id. (segundos). El valor predeterminado es de 3600 (1 hora). El mínimo es de 300 (5 minutos). El máximo es de 86 400 (24 horas). 
- **refresh_token_lifetime_secs**: duración de los tokens de actualización (segundos). El valor predeterminado es de 120 9600 (14 días). El mínimo es de 86 400 (24 horas). El máximo es de 7 776 000 (90 años). 
- **rolling_refresh_token_lifetime_secs**: duración de la ventana deslizante de los tokens de actualización (segundos). El valor predeterminado es de 7 776 000 (90 días). El mínimo es de 86 400 (24 horas). El máximo es de 31 536 000 (365 días). Si no quiere exigir una duración de la ventana deslizante, establezca el valor de `allow_infinite_rolling_refresh_token` en `true`. 
- **allow_infinite_rolling_refresh_token**: la duración de la ventana deslizante de los token de actualización nunca expira. 

::: zone-end


## <a name="token-compatibility-settings"></a>Configuración de compatibilidad de tokens

Puede configurar la compatibilidad de los tokens, entre otras cosas:

- **Notificación del emisor (ISS)** : formato del emisor del token de acceso y de identificador.
- **Notificación de firmante (sub)** : entidad de seguridad sobre la que el token declara información, por ejemplo, el usuario de una aplicación. Este valor es inmutable y no se puede reasignar ni volver a usar. Se puede usar para realizar comprobaciones de autorización de forma segura, por ejemplo, cuando el token se usa para acceder a un recurso. De manera predeterminada, la notificación del asunto se rellena con el identificador de objeto del usuario del directorio.
- **Notificación que representa el flujo de usuario**: esta notificación identifica el flujo de usuario que se ejecutó. Valores posibles: `tfp` (opción predeterminada) o `acr`.

::: zone pivot="b2c-user-flow"

Para configurar las opciones de compatibilidad del flujo de usuario:

1. Seleccione **Flujos de usuario (directivas)** .
1. Abra el flujo de usuario que creó anteriormente.
1. Seleccione **Propiedades**.
1. En **Configuración de compatibilidad del token**, ajuste las propiedades para satisfacer las necesidades de su aplicación:
1. Haga clic en **Save**(Guardar).

::: zone-end

::: zone pivot="b2c-custom-policy"

Para cambiar la configuración de la compatibilidad de los tokens, establezca los metadatos del perfil técnico del [Emisor del token](jwt-issuer-technical-profile.md) en la extensión, o el archivo del usuario de confianza de la directiva a la que desea que afecte. El perfil técnico del emisor del token es similar al ejemplo siguiente:

```xml
<ClaimsProviders>
  <ClaimsProvider>
    <DisplayName>Token Issuer</DisplayName>
    <TechnicalProfiles>
      <TechnicalProfile Id="JwtIssuer">
        <Metadata>
          ...
          <Item Key="IssuanceClaimPattern">AuthorityAndTenantGuid</Item>
          <Item Key="AuthenticationContextReferenceClaimPattern">None</Item>
        </Metadata>
      </TechnicalProfile>
    </TechnicalProfiles>
  </ClaimsProvider>
</ClaimsProviders>
```

- **Notificación del emisor (iss)** : este valor se establece con el metadato **IssuanceClaimPattern**. Los valores aplicables son `AuthorityAndTenantGuid` y `AuthorityWithTfp`.
- **Configuración de notificación que representa el identificador de directiva**: las opciones para configurar este valor son `TFP` (directiva de plataforma de confianza) y `ACR` (referencia de contexto de autenticación). `TFP` es el valor recomendado. Establezca el valor de **AuthenticationContextReferenceClaimPattern** en `None`.

    En el elemento **ClaimsSchema**, agregue este elemento:

    ```xml
    <ClaimType Id="trustFrameworkPolicy">
      <DisplayName>Trust framework policy name</DisplayName>
      <DataType>string</DataType>
    </ClaimType>
    ```

    En **OutputClaims**, agregue este elemento:

    ```xml
    <OutputClaim ClaimTypeReferenceId="trustFrameworkPolicy" Required="true" DefaultValue="{policy}" />
    ```

    Para ACR, quite **AuthenticationContextReferenceClaimPattern**.

- **Notificación de asunto (sub)** : esta opción se establece de forma predeterminada en ObjectID. Si quiere cambiarla a `Not Supported`, reemplace esta línea:

    ```xml
    <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub" />
    ```

    por esta otra:

    ```xml
    <OutputClaim ClaimTypeReferenceId="sub" />
    ```

::: zone-end

## <a name="provide-optional-claims-to-your-app"></a>Proporcionar notificaciones opcionales a la aplicación

Las notificaciones de aplicación son valores que se devuelven a la aplicación. Actualice el flujo de usuario para que contenga las notificaciones que necesite.

::: zone pivot="b2c-user-flow"

1. Seleccione **Flujos de usuario (directivas)** .
1. Abra el flujo de usuario que creó anteriormente.
1. Seleccione **Notificaciones de aplicación**.
1. Elija las notificaciones y los atributos que quiera enviar de nuevo a la aplicación.
1. Haga clic en **Save**(Guardar).

::: zone-end

::: zone pivot="b2c-custom-policy"

Las notificaciones de salida del [perfil técnico de la directiva del usuario de confianza](relyingparty.md#technicalprofile) son valores que se devuelven a una aplicación. La adición de notificaciones de salida enviará la notificación al token después de un recorrido del usuario correcto y se enviará a la aplicación. Modifique el elemento de perfil técnico en la sección del usuario de confianza para agregar las notificaciones deseadas como una notificación de salida.

1. Abra el archivo de la directiva personalizada. Por ejemplo, SignUpOrSignin.xml.
1. Busque el elemento OutputClaims. Agregue el elemento OutputClaim que desea incluir en el token. 
1. Establezca los atributos de la notificaciones de salida. 

En el siguiente ejemplo se agrega la notificación `accountBalance`. La notificación accountBalance se envía a la aplicación como un saldo. 

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="identityProvider" />
      <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      <!--Add the optional claims here-->
      <OutputClaim ClaimTypeReferenceId="accountBalance" DefaultValue="" PartnerClaimType="balance" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

El elemento OutputClaim contiene los atributos siguientes:

- **ClaimTypeReferenceId**: el identificador de un tipo de notificación que ya se ha definido en la sección [ClaimsSchema](claimsschema.md) del archivo de directiva o del archivo de directiva principal.
- **PartnerClaimType**: permite cambiar el nombre de la notificación en el token. 
- **DefaultValue**: un valor predeterminado. También puede establecer el valor predeterminado en un [solucionador de notificaciones](claim-resolver-overview.md), como el identificador de inquilino.
- **AlwaysUseDefaultValue**: fuerza el uso del valor predeterminado.

::: zone-end

## <a name="authorization-code-lifetime"></a>Vigencia de los códigos de autorización

Al usar el [flujo de código de autorización de OAuth 2.0](authorization-code-flow.md), la aplicación puede usar el código de autorización para solicitar un token de acceso para un recurso de destino. Los códigos de autorización son de corta duración y normalmente expiran después de unos 10 minutos. No se puede configurar la vigencia de los códigos de autorización. Asegúrese de que la aplicación canjee los códigos de autorización en un plazo de 10 minutos. 

## <a name="next-steps"></a>Pasos siguientes

Más información sobre cómo [solicitar tokens de acceso](access-tokens.md).
