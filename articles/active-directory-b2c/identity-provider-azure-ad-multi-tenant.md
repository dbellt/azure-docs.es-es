---
title: Configuración del inicio de sesión para Azure AD multiinquilino mediante directivas personalizadas
titleSuffix: Azure AD B2C
description: Adición de un proveedor de identidades de Azure AD multiinquilino mediante directivas personalizadas en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/15/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 9d9ba7e3e898da8d5b7d1bb4fbc69554ee205147
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107028342"
---
# <a name="set-up-sign-in-for-multi-tenant-azure-active-directory-using-custom-policies-in-azure-active-directory-b2c"></a>Configuración del inicio de sesión para Azure Active Directory multiinquilino mediante directivas personalizadas en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

En este artículo se muestra cómo habilitar el inicio de sesión de los usuarios usando el punto de conexión multiinquilino de Azure Active Directory (Azure AD). Esto permite a los usuarios de varios inquilinos de Azure AD iniciar sesión en Azure AD B2C sin tener que configurar un proveedor de identidades para cada inquilino. Sin embargo, los miembros invitados en cualquiera de estos inquilinos **no** podrán iniciar sesión. Para ello, tendrá que [configurar individualmente cada inquilino](identity-provider-azure-ad-single-tenant.md).

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="register-an-application"></a>Registro de una aplicación

Para habilitar el inicio de sesión para los usuarios con una cuenta de Azure AD en Azure Active Directory B2C (Azure AD B2C), tiene que crear una aplicación en [Azure Portal](https://portal.azure.com). Para más información, consulte [Registro de una aplicación con la plataforma de identidad de Microsoft](../active-directory/develop/quickstart-register-app.md).


1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD de la organización (por ejemplo, contoso.com). Seleccione el **filtro de directorio y suscripción** en el menú superior y elija el directorio que contiene la suscripción.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Registros de aplicaciones**.
1. Seleccione **Nuevo registro**.
1. Escriba el **nombre** de la aplicación. Por ejemplo, `Azure AD B2C App`.
1. Seleccione **Cuentas en cualquier directorio organizativo (cualquier directorio de Azure AD: multiinquilino)** para esa aplicación.
1. Para la **URI de redirección**, acepte el valor de **Web** y escriba la siguiente dirección URL en minúscula, donde `your-B2C-tenant-name` se reemplaza por el nombre del inquilino de Azure AD B2C.

    ```
    https://your-B2C-tenant-name.b2clogin.com/your-B2C-tenant-name.onmicrosoft.com/oauth2/authresp
    ```

    Por ejemplo, `https://fabrikam.b2clogin.com/fabrikam.onmicrosoft.com/oauth2/authresp`.

    Si usa un [dominio personalizado](custom-domain.md), escriba `https://your-domain-name/your-tenant-name.onmicrosoft.com/oauth2/authresp`. Reemplace `your-domain-name` por el dominio personalizado, y `your-tenant-name` por el nombre del inquilino.

1. Seleccione **Registrar**. Anote el **Id. de aplicación (cliente)** para usarlo en un paso posterior.
1. Seleccione **Certificados y secretos** y luego seleccione **Nuevo secreto de cliente**.
1. En **Descripción**, escriba una descripción para el secreto, seleccione una fecha de expiración y seleccione **Agregar**. Registre el valor **Value** del secreto para usarlo en un paso posterior.

## <a name="configuring-optional-claims"></a>Configuración de notificaciones opcionales

Si quiere obtener las notificaciones `family_name` y `given_name` de Azure AD, puede configurar notificaciones opcionales para la aplicación en la interfaz de usuario de Azure Portal o el manifiesto de aplicación. Para obtener más información, consulte [Procedimientos: Proporcionar notificaciones opcionales a la aplicación de Azure AD](../active-directory/develop/active-directory-optional-claims.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Busque y seleccione **Azure Active Directory**.
1. En la sección **Administrar**, seleccione **Registros de aplicaciones**.
1. Seleccione en la lista la aplicación para la que desea configurar notificaciones opcionales.
1. En la sección **Administrar**, seleccione **Configuración del token**.
1. Seleccione **Agregar notificación opcional**.
1. En **Tipo de token**, seleccione **ID**.
1. Seleccione las notificaciones opcionales que va a agregar, `family_name` y `given_name`.
1. Haga clic en **Agregar**.

## <a name="create-a-policy-key"></a>Creación de una clave de directiva

Debe almacenar la clave de la aplicación que creó en el inquilino de Azure AD B2C.

1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. En **Directivas**, seleccione **Identity Experience Framework**.
1. Seleccione **Claves de directiva** y luego **Agregar**.
1. En **Opciones**, elija `Manual`.
1. Escriba un **nombre** para la clave de directiva. Por ejemplo, `AADAppSecret`.  El prefijo `B2C_1A_` se agrega automáticamente al nombre de la clave cuando se crea, por lo que hace referencia a *B2C_1A_AADAAppSecret* en el código XML de la siguiente sección.
1. En **Secreto**, escriba el secreto de cliente que registró previamente.
1. En **Uso de claves**, seleccione `Signature`.
1. Seleccione **Crear**.

## <a name="configure-azure-ad-as-an-identity-provider"></a>Configuración de Azure AD como proveedor de identidades

Para permitir que los usuarios inicien sesión con una cuenta de Azure AD, deberá definir Azure AD como proveedor de notificaciones con el que Azure AD B2C pueda comunicarse a través de un punto de conexión. El punto de conexión proporciona un conjunto de notificaciones que Azure AD B2C usa para comprobar que un usuario concreto se ha autenticado.

Para definir Azure AD como proveedor de notificaciones, agregue el elemento **ClaimsProvider** al archivo de extensión de la directiva.

1. Abra el archivo *TrustFrameworkExtensions.xml*.
1. Busque el elemento **ClaimsProviders**. Si no existe, agréguelo debajo del elemento raíz.
1. Agregue un nuevo elemento **ClaimsProvider** tal como se muestra a continuación:

    ```xml
    <ClaimsProvider>
      <Domain>commonaad</Domain>
      <DisplayName>Common AAD</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="AADCommon-OpenIdConnect">
          <DisplayName>Multi-Tenant AAD</DisplayName>
          <Description>Login with your Contoso account</Description>
          <Protocol Name="OpenIdConnect"/>
          <Metadata>
            <Item Key="METADATA">https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration</Item>
            <!-- Update the Client ID below to the Application ID -->
            <Item Key="client_id">00000000-0000-0000-0000-000000000000</Item>
            <Item Key="response_types">code</Item>
            <Item Key="scope">openid profile</Item>
            <Item Key="response_mode">form_post</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
            <Item Key="DiscoverMetadataByTokenIssuer">true</Item>
            <!-- The key below allows you to specify each of the Azure AD tenants that can be used to sign in. Update the GUIDs below for each tenant. -->
            <Item Key="ValidTokenIssuerPrefixes">https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000,https://login.microsoftonline.com/11111111-1111-1111-1111-111111111111</Item>
            <!-- The commented key below specifies that users from any tenant can sign-in. Uncomment if you would like anyone with an Azure AD account to be able to sign in. -->
            <!-- <Item Key="ValidTokenIssuerPrefixes">https://login.microsoftonline.com/</Item> -->
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_AADAppSecret"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="oid"/>
            <OutputClaim ClaimTypeReferenceId="tenantId" PartnerClaimType="tid"/>
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
            <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" PartnerClaimType="iss" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin"/>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. En el elemento **ClaimsProvider**, actualice el valor de **Domain** a un valor único que pueda usarse para distinguirlo de otros proveedores de identidades.
1. En el elemento **TechnicalProfile**, actualice el valor de **DisplayName**, por ejemplo, `Contoso Employee`. Este valor se muestra en el botón de inicio de sesión de la página de inicio de sesión.
1. Establezca **client_id** en el identificador de aplicación de la aplicación multiinquilino de Azure AD que registró anteriormente.
1. En **CryptographicKeys**, actualice el valor de **StorageReferenceId** con el nombre de la clave de directiva que creó anteriormente. Por ejemplo, `B2C_1A_AADAppSecret`.

### <a name="restrict-access"></a>Restricción del acceso

Al usar `https://login.microsoftonline.com/` como valor para **ValidTokenIssuerPrefixes**, se permite a todos los usuarios de Azure AD iniciar sesión en la aplicación. Actualice la lista de emisores de tokens válidos y restrinja el acceso a la lista específica de usuarios inquilinos de Azure AD que pueden iniciar sesión.

Para obtener los valores, examine los metadatos de descubrimiento de OpenID Connect de cada uno de los inquilinos específicos de Azure AD desde los que le gustaría que los usuarios iniciaran sesión. El formato de la dirección URL de metadatos es similar a `https://login.microsoftonline.com/your-tenant/v2.0/.well-known/openid-configuration`, donde `your-tenant` es el nombre del inquilino de Azure AD. Por ejemplo:

`https://login.microsoftonline.com/fabrikam.onmicrosoft.com/v2.0/.well-known/openid-configuration`

Siga estos pasos para cada inquilino de Azure AD que se deba usar para iniciar sesión:

1. Abra el explorador y vaya a la dirección URL de metadatos de OpenID Connect para el inquilino. Busque el objeto **issuer** y anote su valor. Debe tener un aspecto similar a `https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000/`.
1. Copie y pegue el valor en la clave **ValidTokenIssuerPrefixes**. Separe varios emisores con una coma. Un ejemplo con dos emisores aparece en el ejemplo de código XML `ClaimsProvider` anterior.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="AzureADCommonExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="AzureADCommonExchange" TechnicalProfileReferenceId="AADCommon-OpenIdConnect" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Prueba de la directiva personalizada

1. Seleccione la directiva de usuarios de confianza, por ejemplo `B2C_1A_signup_signin`.
1. En **Aplicación**, seleccione la aplicación web que [registró anteriormente](tutorial-register-applications.md). La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar ahora**.
1. En la página de registro o de inicio de sesión, seleccione **Common AAD** para iniciar sesión con la cuenta de Azure AD.

Para probar la funcionalidad de inicio de sesión de varios inquilinos, realice los dos últimos pasos con las credenciales de un usuario que exista en otro inquilino de Azure AD. Copie el valor de **Ejecutar punto de conexión ahora** y ábralo en una ventana privada del explorador web, por ejemplo, en modo incógnito en Google Chrome o en una ventana de InPrivate en Microsoft Edge. De esta forma, podrá probar el recorrido completo del usuario sin usar ninguna credencial de Azure AD actualmente almacenada en caché.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.

## <a name="next-steps"></a>Pasos siguientes

Al trabajar con directivas personalizadas, en ocasiones es posible que necesite información adicional al solucionar problemas de una directiva durante su desarrollo.

Para ayudar a diagnosticar problemas, puede poner temporalmente la directiva en "modo de programador" y recopilar registros con Azure Application Insights. Descubra cómo en [Azure Active Directory B2C: Recopilación de registros](troubleshoot-with-application-insights.md).

::: zone-end
