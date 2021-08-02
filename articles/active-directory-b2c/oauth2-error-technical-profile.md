---
title: Definición de un perfil técnico de error personalizado de OAuth2 en una directiva personalizada
titleSuffix: Azure AD B2C
description: Definición de un perfil técnico de error personalizado de OAuth2 en una directiva personalizada en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 05/26/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 6fc4f9e7be394a8c63bb95969a2928c63b0c122d
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111962334"
---
# <a name="define-an-oauth2-custom-error-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Definición de un perfil técnico de error personalizado de OAuth2 en una directiva personalizada en Azure Active Directory B2C

En este artículo se describe cómo controlar un error personalizado de OAuth2 con Azure Active Directory B2C (Azure AD B2C). Use este perfil técnico si hay algún problema en la lógica de la directiva. El perfil técnico devuelve un error a la aplicación de usuario de confianza de OAuth2 o de OpenId Connect.

Para controlar el mensaje de error de OAuth2 personalizado:

1. Defina un perfil técnico de error de OAuth2.
1. Establezca el código de error y las notificaciones del mensaje de error.
1. Desde el recorrido del usuario, llame al perfil técnico de error de OAuth2.

## <a name="oauth2-error"></a>Error de OAuth2

El error se devuelve con los datos siguientes:

- **error** - `access_denied`
- **error_description:** mensaje de error que usa la convención `AAD_Custom_<errorCode>: <errorMessage>`.
- **Id. de correlación**: visualización del id. de correlación de Azure AD B2C.
- **Marca de tiempo**: marca de tiempo del error.

En el ejemplo siguiente se muestra un mensaje de error personalizado que se devuelve a la aplicación https://jwt.ms:

```http
https://jwt.ms/#error=access_denied&error_description=AAD_Custom_1234%3a+My+custom+error+message%0d%0aCorrelation+ID%3a+233bf9bd-747a-4800-9062-6236f3f69a47%0d%0aTimestamp%3a+2021-03-25+14%3a01%3a23Z%0d%0a
```
  
## <a name="protocol"></a>Protocolo

El atributo **Name** del elemento **Protocol** tiene que establecerse en `None`. Establezca el elemento **OutputTokenFormat** en `OAuth2Error`.

En el ejemplo siguiente se muestra un perfil técnico de `ReturnOAuth2Error`:

```xml
<!--
 <ClaimsProviders> -->
  <ClaimsProvider>
    <DisplayName>Token Issuer</DisplayName>
    <TechnicalProfiles>
      <TechnicalProfile Id="ReturnOAuth2Error">
        <DisplayName>Return OAuth2 error</DisplayName>
        <Protocol Name="None" />
        <OutputTokenFormat>OAuth2Error</OutputTokenFormat>
        <CryptographicKeys>
          <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
        </CryptographicKeys>
        <InputClaims>
          <InputClaim ClaimTypeReferenceId="errorCode" />
          <InputClaim ClaimTypeReferenceId="errorMessage" />
        </InputClaims>
      </TechnicalProfile>
    </TechnicalProfiles>
  </ClaimsProvider>
<!--
</ClaimsProviders> -->
```

## <a name="input-claims"></a>Notificaciones de entrada

El elemento **InputClaims** contiene una lista de notificaciones necesarias para devolver el error de OAuth2. 

| ClaimReferenceId | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| errorCode | Sí | Código de error. | 
| errorMessage | Sí | El mensaje de error. |

## <a name="cryptographic-keys"></a>Claves de cifrado

El elemento CryptographicKeys contiene la clave siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| issuer_secret | Sí  | Un certificado X509 (conjunto de claves RSA). Use la clave `B2C_1A_TokenSigningKeyContainer` que configuró en [Introducción a las directivas personalizadas](./tutorial-create-user-flows.md?pivots=b2c-custom-policy).|
|

## <a name="invoke-the-technical-profile"></a>Llamada al perfil técnico

Puede llamar al perfil técnico de error de OAuth2 desde un recorrido del usuario o desde un subrecorrido. Establezca el tipo de [paso de orquestación](userjourneys.md#orchestrationsteps) en `SendClaims` con una referencia al perfil técnico de error de OAuth2.

Si el recorrido del usuario o el subrecorrido ya tiene otro paso de orquestación `SendClaims`, establezca el atributo `DefaultCpimIssuerTechnicalProfileReferenceId` en el perfil técnico del emisor del token.

En el ejemplo siguiente:

-  El recorrido del usuario `SignUpOrSignIn-Custom` establece `DefaultCpimIssuerTechnicalProfileReferenceId` en el perfil técnico del emisor de tokens `JwtIssuer`. 
- El octavo paso de orquestación comprueba si existe `errorCode`. En caso afirmativo, llame al perfil técnico `ReturnOAuth2Error` para devolver el error.
- Si `errorCode` no existe, el noveno paso de orquestación emite el token.

```xml
<UserJourney Id="SignUpOrSignIn-Custom" DefaultCpimIssuerTechnicalProfileReferenceId="JwtIssuer">
  <OrchestrationSteps>
    ...
    <OrchestrationStep Order="8" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="ReturnOAuth2Error">
      <Preconditions>
        <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
          <Value>errorCode</Value>
          <Action>SkipThisOrchestrationStep</Action>
        </Precondition>
      </Preconditions>
    </OrchestrationStep>

    <OrchestrationStep Order="9" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />

  </OrchestrationSteps>
  <ClientDefinition ReferenceId="DefaultWeb" />
</UserJourney>
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre [UserJourneys](userjourneys.md)