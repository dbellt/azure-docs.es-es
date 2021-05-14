---
title: Perfiles técnicos de acceso condicional en directivas personalizadas
titleSuffix: Azure AD B2C
description: Referencia de directivas personalizadas para los perfiles técnicos de acceso condicional en Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 04/19/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: ebdc1c9c92f6e3debf08cb640e46424c4ad9d5ad
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107721086"
---
# <a name="define-a-conditional-access-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Definición de un perfil técnico de acceso condicional en una directiva personalizada de Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

El acceso condicional de Azure Active Directory (Azure AD) es la herramienta que usa Azure AD B2C para reunir las señales, tomar decisiones y aplicar las directivas de la organización. La automatización de la evaluación de riesgos con condiciones de directiva significa que los inicios de sesión de riesgo se identifican y corrigen o bloquean.

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

## <a name="protocol"></a>Protocolo

El atributo **Name** del elemento **Protocol** tiene que establecerse en `Proprietary`. El atributo **handler** debe contener el nombre completo del ensamblado del controlador de protocolo que usa Azure AD B2C:

```
Web.TPEngine.Providers.ConditionalAccessProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
```

En el ejemplo siguiente se muestra un perfil técnico de acceso condicional:

```XML
<TechnicalProfile Id="ConditionalAccessEvaluation">
  <DisplayName>Conditional Access Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.ConditionalAccessProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="OperationType">Evaluation</Item>
  </Metadata>
```

## <a name="conditional-access-evaluation"></a>Evaluación del acceso condicional

En cada inicio de sesión, Azure AD B2C evalúa todas las directivas y garantiza que se cumplan todos los requisitos antes de conceder acceso al usuario. "Bloquear el acceso" invalida todas las demás opciones de configuración. El modo **Evaluación** del perfil técnico de acceso condicional evalúa las señales que Azure AD B2C recopila durante el inicio de sesión con una cuenta local. El resultado del perfil técnico de acceso condicional es un conjunto de notificaciones que proceden de la evaluación del acceso condicional. La directiva de Azure AD B2C usa estas notificaciones en un siguiente paso de orquestación para realizar una acción, como bloquear al usuario o desafiarlo con la autenticación multifactor. Pueden configurarse las opciones siguientes para este modo.

### <a name="metadata"></a>Metadatos

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| OperationType | Sí | Debe ser **Evaluation**.  |

### <a name="input-claims"></a>Notificaciones de entrada

El elemento **InputClaims** contiene una lista de notificaciones para enviar al acceso condicional. También puede asignar el nombre de la notificación al nombre definido en el perfil técnico del acceso condicional.

| ClaimReferenceId | Obligatorio | Tipo de datos | Descripción |
| --------- | -------- | ----------- |----------- |
| UserId | Sí | string | Identificador del usuario que inicia sesión. |
| AuthenticationMethodsUsed | Sí |stringCollection | Lista de métodos que el usuario usó para iniciar sesión. Valores posibles: `Password` y `OneTimePasscode`. |
| IsFederated | Sí |boolean | Indica si un usuario inició sesión con una cuenta federada. El valor tiene que ser `false`. |
| IsMfaRegistered | Sí |boolean | Indica si el usuario ya inscribió un número de teléfono para la autenticación multifactor. |


El elemento **InputClaimsTransformations** puede contener una colección de elementos **InputClaimsTransformation** que se usan para modificar las notificaciones de entrada o generar otras nuevas antes de enviarlas al servicio de acceso condicional.

### <a name="output-claims"></a>Notificaciones de salida

El elemento **OutputClaims** contiene una lista de las notificaciones que ConditionalAccessProtocolProvider genera. También puede asignar el nombre de la notificación al nombre que se define a continuación.

| ClaimReferenceId | Obligatorio | Tipo de datos | Descripción |
| --------- | -------- | ----------- |----------- |
| Desafíos | Sí |stringCollection | Lista de acciones para corregir la amenaza identificada. Valores posibles: `block` |
| MultiConditionalAccessStatus | Sí | stringCollection |  |

El elemento **OutputClaimsTransformations** puede contener una colección de elementos **OutputClaimsTransformation** que se usan para modificar las notificaciones de salida o para generar nuevas.

### <a name="example-evaluation"></a>Ejemplo: Evaluación

En el ejemplo siguiente se muestra un perfil técnico de acceso condicional que se usa para evaluar la amenaza de inicio de sesión.

```XML
<TechnicalProfile Id="ConditionalAccessEvaluation">
  <DisplayName>Conditional Access Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.ConditionalAccessProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="OperationType">Evaluation</Item>
  </Metadata>
  <InputClaimsTransformations>
    <InputClaimsTransformation ReferenceId="IsMfaRegisteredCT" />
  </InputClaimsTransformations>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="UserId" />
    <InputClaim ClaimTypeReferenceId="AuthenticationMethodsUsed" />
    <InputClaim ClaimTypeReferenceId="IsFederated" DefaultValue="false" />
    <InputClaim ClaimTypeReferenceId="IsMfaRegistered" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="conditionalAccessClaimCollection" PartnerClaimType="Challenges" />
    <OutputClaim ClaimTypeReferenceId="ConditionalAccessStatus" PartnerClaimType="MultiConditionalAccessStatus" />
  </OutputClaims>
</TechnicalProfile>
```

## <a name="remediation"></a>Corrección

El modo **Remediation** (Corrección) del perfil técnico de acceso condicional informa a Azure AD B2C de que la amenaza que se identificó en el inicio de sesión se remedió. Pueden configurarse las opciones siguientes para el modo de corrección.

### <a name="metadata"></a>Metadatos

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| OperationType | Sí | Debe ser **Remediation** (Corrección).  |

### <a name="input-claims"></a>Notificaciones de entrada

El elemento **InputClaims** contiene una lista de notificaciones para enviar al acceso condicional. También puede asignar el nombre de la notificación al nombre definido en el perfil técnico del acceso condicional.

| ClaimReferenceId | Obligatorio | Tipo de datos | Descripción |
| --------- | -------- | ----------- |----------- |
| ChallengesSatisfied | Sí | stringCollection| La lista de desafíos satisfechos para corregir la amenaza identificada como resultado del modo de evaluación, la notificación de desafíos.|


El elemento **InputClaimsTransformations** puede contener una colección de los elementos **InputClaimsTransformation** que se usan para modificar las notificaciones de entrada o generar otras nuevas antes de llamar al servicio de acceso condicional.

### <a name="output-claims"></a>Notificaciones de salida

El proveedor del protocolo de acceso condicional no devuelve ningún valor **OutputClaims**, por lo que no es necesario especificar las notificaciones de salida. Sin embargo, puede incluir notificaciones que el proveedor del protocolo de acceso condicional no devuelva, siempre que establezca el atributo `DefaultValue`.

El elemento **OutputClaimsTransformations** puede contener una colección de elementos **OutputClaimsTransformation** que se usan para modificar las notificaciones de salida o para generar nuevas.

### <a name="example-remediation"></a>Ejemplo: Corrección

En el ejemplo siguiente se muestra un perfil técnico de acceso condicional que se usa para corregir la amenaza de inicio de sesión.

```xml
<TechnicalProfile Id="ConditionalAccessRemediation">
  <DisplayName>Conditional Access Remediation</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.ConditionalAccessProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"/>
  <Metadata>
    <Item Key="OperationType">Remediation</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="conditionalAccessClaimCollection" PartnerClaimType="ChallengesSatisfied" />
  </InputClaims>
</TechnicalProfile>
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre la [Adición del acceso condicional a los flujos de usuario en Azure AD B2C](conditional-access-user-flow.md).
