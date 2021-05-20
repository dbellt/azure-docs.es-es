---
title: Ejemplos de transformación de notificaciones de cadena para directivas personalizadas
titleSuffix: Azure AD B2C
description: Ejemplos de transformación de notificaciones de cadena para el esquema de Identity Experience Framework (IEF) de Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/08/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 176c36ee5c3addf655503e3a371767764e0d9968
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108738060"
---
# <a name="string-claims-transformations"></a>Transformaciones de notificaciones de cadena

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

En este artículo se proporcionan ejemplos para usar las transformaciones de notificaciones de cadena del esquema Identity Experience Framework en Azure Active Directory B2C (Azure AD B2C). Para más información, vea [ClaimsTransformations](claimstransformations.md).

## <a name="assertstringclaimsareequal"></a>AssertStringClaimsAreEqual

Comparar dos notificaciones y emitir una excepción si no son iguales según la comparación especificada inputClaim1, inputClaim2 y stringComparison.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | string | Tipo de la primera notificación, que se va a comparar. |
| InputClaim | inputClaim2 | string | Tipo de la segunda notificación, que se va a comparar. |
| InputParameter | stringComparison | string | comparación de cadenas, uno de los valores: Ordinal, OrdinalIgnoreCase. |

La transformación de notificaciones **AssertStringClaimsAreEqual** siempre se ejecuta desde un [perfil técnico de validación](validation-technical-profile.md) llamado por un [perfil técnico autoafirmado](self-asserted-technical-profile.md) o un elemento [DisplayControl](display-controls.md). Los metadatos `UserMessageIfClaimsTransformationStringsAreNotEqual` de un perfil técnico autoafirmado controlan el mensaje de error que se presenta al usuario. Los mensajes de error se pueden [localizar](localization-string-ids.md#claims-transformations-error-messages).


![Ejecución de AssertStringClaimsAreEqual](./media/string-transformations/assert-execution.png)

Puede usar esta transformación de notificaciones para asegurarse de que dos argumentos ClaimType tienen el mismo valor. Si no es así, se emite un mensaje de error. En el ejemplo siguiente se comprueba que los argumentos ClaimType **strongAuthenticationEmailAddress** e **email** son iguales. De lo contrario, se emite un mensaje de error.

```xml
<ClaimsTransformation Id="AssertEmailAndStrongAuthenticationEmailAddressAreEqual" TransformationMethod="AssertStringClaimsAreEqual">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="strongAuthenticationEmailAddress" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
  </InputParameters>
</ClaimsTransformation>
```


El perfil técnico de validación **login-NonInteractive** llama a la transformación de notificaciones **AssertEmailAndStrongAuthenticationEmailAddressAreEqual**.
```xml
<TechnicalProfile Id="login-NonInteractive">
  ...
  <OutputClaimsTransformations>
    <OutputClaimsTransformation ReferenceId="AssertEmailAndStrongAuthenticationEmailAddressAreEqual" />
  </OutputClaimsTransformations>
</TechnicalProfile>
```

El perfil técnico autoafirmado llama al perfil técnico **login-NonInteractive** de validación.

```xml
<TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
  <Metadata>
    <Item Key="UserMessageIfClaimsTransformationStringsAreNotEqual">Custom error message the email addresses you provided are not the same.</Item>
  </Metadata>
  <ValidationTechnicalProfiles>
    <ValidationTechnicalProfile ReferenceId="login-NonInteractive" />
  </ValidationTechnicalProfiles>
</TechnicalProfile>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
  - **inputClaim1**: someone@contoso.com
  - **inputClaim2**: someone@outlook.com
- Parámetros de entrada:
  - **stringComparison**: ordinalIgnoreCase
- Resultado: aparece un error

## <a name="changecase"></a>ChangeCase

Cambia el uso de mayúsculas y minúsculas en la notificación proporcionada, dependiendo del operador.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | string | ClaimType que se va a cambiar. |
| InputParameter | toCase | string | Puede ser uno de los siguientes valores: `LOWER` o `UPPER`. |
| OutputClaim | outputClaim | string | El valor ClaimType que se genera después de que se haya invocado esta transformación de notificaciones. |

Use esta transformación de notificaciones para cambiar cualquier ClaimType de cadena a minúsculas o mayúsculas.

```xml
<ClaimsTransformation Id="ChangeToLower" TransformationMethod="ChangeCase">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="inputClaim1" />
  </InputClaims>
<InputParameters>
  <InputParameter Id="toCase" DataType="string" Value="LOWER" />
</InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
  - **email**: SomeOne@contoso.com
- Parámetros de entrada:
    - **toCase**: LOWER
- Notificaciones de salida:
  - **email**: someone@contoso.com

## <a name="createstringclaim"></a>CreateStringClaim

Crea una notificación de cadena a partir del parámetro de entrada proporcionado en la transformación.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
|----- | ----------------------- | --------- | ----- |
| InputParameter | value | string | La cadena que va a establecerse. Este parámetro de entrada admite [expresiones de transformación de notificaciones de cadena](string-transformations.md#string-claim-transformations-expressions). |
| OutputClaim | createdClaim | string | El valor ClaimType que se genera después de que se haya invocado esta transformación de notificaciones, con el valor especificado en el parámetro de entrada. |

Use que esta transformación de notificaciones para establecer un valor de ClaimType de cadena.

```xml
<ClaimsTransformation Id="CreateTermsOfService" TransformationMethod="CreateStringClaim">
  <InputParameters>
    <InputParameter Id="value" DataType="string" Value="Contoso terms of service..." />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="TOS" TransformationClaimType="createdClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Parámetro de entrada:
    - **value**: Contoso terms of service...
- Notificaciones de salida:
    - **createdClaim**: el tipo ClaimType TOS contiene el valor "Contoso terms of service…".

## <a name="copyclaimifpredicatematch"></a>CopyClaimIfPredicateMatch

Copie el valor de una notificación en otra si el valor de la notificación de entrada coincide con el predicado de la notificación de salida. 

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | string | Tipo de la notificación que se va a copiar. |
| OutputClaim | outputClaim | string | El tipo de notificación que se genera después de que se haya invocado esta transformación de notificaciones. El valor de la notificación de entrada se comprueba con este predicado de notificación. |

En el ejemplo siguiente se copia el valor de la notificación signInName en la notificación phoneNumber, solo si signInName es un número de teléfono. Para ver el ejemplo completo, consulte la directiva del módulo [Inicio de sesión con número de teléfono o correo electrónico](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/blob/master/scenarios/phone-number-passwordless/Phone_Email_Base.xml).

```xml
<ClaimsTransformation Id="SetPhoneNumberIfPredicateMatch" TransformationMethod="CopyClaimIfPredicateMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="signInName" TransformationClaimType="inputClaim" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example-1"></a>Ejemplo 1

- Notificaciones de entrada:
    - **inputClaim**: bob@contoso.com
- Notificaciones de salida:
    - **outputClaim**: la notificación de salida no se cambiará de su valor original.

### <a name="example-2"></a>Ejemplo 2

- Notificaciones de entrada:
    - **inputClaim**: +11234567890
- Notificaciones de salida:
    - **outputClaim**: +11234567890

## <a name="compareclaims"></a>CompareClaims

Determine si una notificación de cadena es igual a otra. El resultado es un nuevo ClaimType booleano con un valor de `true` o `false`.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | string | Tipo de la primera notificación, que se va a comparar. |
| InputClaim | inputClaim2 | string | Tipo de la segunda notificación, que se va a comparar. |
| InputParameter | operator | string | Valores posibles: `EQUAL` o `NOT EQUAL`. |
| InputParameter | ignoreCase | boolean | Especifica si la comparación distingue entre mayúsculas y minúsculas en las cadenas que se están comparando. |
| OutputClaim | outputClaim | boolean | El valor ClaimType que se genera después de que se haya invocado esta transformación de notificaciones. |

Use esta transformación de notificaciones para comprobar si una notificación es igual a otra notificación. Por ejemplo, la siguiente transformación de notificaciones comprueba si el valor de la notificación **email** es igual al de la notificación **Verified.Email**.

```xml
<ClaimsTransformation Id="CheckEmail" TransformationMethod="CompareClaims">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="Email" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="Verified.Email" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="operator" DataType="string" Value="NOT EQUAL" />
    <InputParameter Id="ignoreCase" DataType="string" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="SameEmailAddress" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
  - **inputClaim1**: someone@contoso.com
  - **inputClaim2**: someone@outlook.com
- Parámetros de entrada:
    - **operator**:  NOT EQUAL
    - **ignoreCase**: true
- Notificaciones de salida:
    - **outputClaim**: true

## <a name="compareclaimtovalue"></a>CompareClaimToValue

Determina si un valor de notificación es igual al valor del parámetro de entrada.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | string | Tipo de la notificación, que se va a comparar. |
| InputParameter | operator | string | Valores posibles: `EQUAL` o `NOT EQUAL`. |
| InputParameter | compareTo | string | comparación de cadenas, uno de los valores: Ordinal, OrdinalIgnoreCase. |
| InputParameter | ignoreCase | boolean | Especifica si la comparación distingue entre mayúsculas y minúsculas en las cadenas que se están comparando. |
| OutputClaim | outputClaim | boolean | El valor ClaimType que se genera después de que se haya invocado esta transformación de notificaciones. |

Puede usar esta transformación de notificaciones para comprobar si una notificación es igual a un valor que haya especificado. Por ejemplo, la siguiente transformación de notificaciones comprueba si el valor de la notificación **termsOfUseConsentVersion** es igual a `v1`.

```xml
<ClaimsTransformation Id="IsTermsOfUseConsentRequiredForVersion" TransformationMethod="CompareClaimToValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="inputClaim1" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="compareTo" DataType="string" Value="V1" />
    <InputParameter Id="operator" DataType="string" Value="not equal" />
    <InputParameter Id="ignoreCase" DataType="string" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentRequired" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo
- Notificaciones de entrada:
    - **inputClaim1**: v1
- Parámetros de entrada:
    - **compareTo**: V1
    - **operator**: EQUAL
    - **ignoreCase**: true
- Notificaciones de salida:
    - **outputClaim**: true

## <a name="createrandomstring"></a>CreateRandomString

Crea una cadena aleatoria mediante el generador de números aleatorios. Si el generador de números aleatorios es de tipo `integer`, opcionalmente se pueden proporcionar un parámetro de inicialización y un número máximo. Un parámetro de formato de cadena opcional permite que la salida tenga ese formato, y un parámetro de base64 opcional especifica si la salida randomGeneratorType [guid, integer] outputClaim (String) está codificada en base64 .

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputParameter | randomGeneratorType | string | Especifica el valor aleatorio que se genera, `GUID` (identificador único global) o `INTEGER` (un número). |
| InputParameter | stringFormat | string | [Opcional] Formato del valor aleatorio. |
| InputParameter | base64 | boolean | [Opcional] Convertir el valor aleatorio en base64. Si se aplica el formato de cadena, el valor después del formato de cadena está codificado en base64. |
| InputParameter | maximumNumber | int | [Opcional] Solo para randomGeneratorType `INTEGER`. Especifique el número máximo. |
| InputParameter | seed  | int | [Opcional] Solo para randomGeneratorType `INTEGER`. Especifique el valor de inicialización del valor aleatorio. Nota: El mismo valor de inicialización genera la misma secuencia de números aleatorios. |
| OutputClaim | outputClaim | string | El valor ClaimType que se genera después de que se haya invocado esta transformación de notificaciones. El valor aleatorio. |

El ejemplo siguiente genera un identificador único global. Esta transformación de notificaciones se usa para crear el UPN (nombre principal del usuario) aleatorio.

```xml
<ClaimsTransformation Id="CreateRandomUPNUserName" TransformationMethod="CreateRandomString">
  <InputParameters>
    <InputParameter Id="randomGeneratorType" DataType="string" Value="GUID" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="upnUserName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Ejemplo

- Parámetros de entrada:
    - **randomGeneratorType**: GUID
- Notificaciones de salida:
    - **outputClaim**: bc8bedd2-aaa3-411e-bdee-2f1810b73dfc

El ejemplo siguiente genera un valor entero aleatorio entre 0 y 1000. El valor pasa a tener el formato OTP_ {valor_aleatorio}.

```xml
<ClaimsTransformation Id="SetRandomNumber" TransformationMethod="CreateRandomString">
  <InputParameters>
    <InputParameter Id="randomGeneratorType" DataType="string" Value="INTEGER" />
    <InputParameter Id="maximumNumber" DataType="int" Value="1000" />
    <InputParameter Id="stringFormat" DataType="string" Value="OTP_{0}" />
    <InputParameter Id="base64" DataType="boolean" Value="false" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="randomNumber" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Parámetros de entrada:
    - **randomGeneratorType**: INTEGER
    - **maximumNumber**: 1000
    - **stringFormat**: OTP_{0}
    - **base64**: false
- Notificaciones de salida:
    - **outputClaim**: OTP_853


## <a name="formatlocalizedstring"></a>FormatLocalizedString

Aplique formato a varias notificaciones de acuerdo con una cadena de formato localizada proporcionada. Esta transformación usa el método `String.Format` de C#.


| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaims |  |string | La colección de notificaciones de entrada que actúa como parámetros de formato de cadena {0}, {1}, {2}. |
| InputParameter | stringFormatId | string |  El valor `StringId` de una [cadena localizada](localization.md).   |
| OutputClaim | outputClaim | string | El valor ClaimType que se genera después de que se haya invocado esta transformación de notificaciones. |

> [!NOTE]
> El formato de cadena máximo permitido es 4000.

Para usar la transformación de notificaciones FormatLocalizedString:

1. Defina una [cadena de localización](localization.md) y asóciela con un [perfil técnico autoafirmado](self-asserted-technical-profile.md).
1. El `ElementType` del elemento `LocalizedString` debe establecerse en `FormatLocalizedStringTransformationClaimType`.
1. `StringId` es un identificador único que se define y se usa más adelante en la transformación de notificaciones `stringFormatId`.
1. En la transformación de notificaciones, especifique la lista de notificaciones que se van a establecer con la cadena localizada. A continuación, establezca `stringFormatId` en el valor `StringId` del elemento de cadena localizado. 
1. En un [perfil técnico autoafirmado](self-asserted-technical-profile.md) o una transformación de notificaciones de entrada o salida de [control de visualización](display-controls.md), haga una referencia a la transformación de notificaciones.


En el ejemplo siguiente se genera un mensaje de error cuando una cuenta ya está en el directorio. En el ejemplo se definen cadenas localizadas para inglés (valor predeterminado) y español.

```xml
<Localization Enabled="true">
  <SupportedLanguages DefaultLanguage="en" MergeBehavior="Append">
    <SupportedLanguage>en</SupportedLanguage>
    <SupportedLanguage>es</SupportedLanguage>
   </SupportedLanguages>

  <LocalizedResources Id="api.localaccountsignup.en">
    <LocalizedStrings>
      <LocalizedString ElementType="FormatLocalizedStringTransformationClaimType" StringId="ResponseMessge_EmailExists">The email '{0}' is already an account in this organization. Click Next to sign in with that account.</LocalizedString>
      </LocalizedStrings>
    </LocalizedResources>
  <LocalizedResources Id="api.localaccountsignup.es">
    <LocalizedStrings>
      <LocalizedString ElementType="FormatLocalizedStringTransformationClaimType" StringId="ResponseMessge_EmailExists">Este correo electrónico "{0}" ya es una cuenta de esta organización. Haga clic en Siguiente para iniciar sesión con esa cuenta.</LocalizedString>
    </LocalizedStrings>
  </LocalizedResources>
</Localization>
```

La transformación de notificaciones crea un mensaje de respuesta basado en la cadena localizada. El mensaje contiene la dirección de correo electrónico del usuario insertada en la cadena localizada *ResponseMessge_EmailExists*.

```xml
<ClaimsTransformation Id="SetResponseMessageForEmailAlreadyExists" TransformationMethod="FormatLocalizedString">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringFormatId" DataType="string" Value="ResponseMessge_EmailExists" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="responseMsg" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **inputClaim**: sarah@contoso.com
- Parámetros de entrada:
    - **stringFormat**:  ResponseMessge_EmailExists
- Notificaciones de salida:
  - **outputClaim**: el correo electrónico "sarah@contoso.com" ya es una cuenta de esta organización. Haga clic en Siguiente para iniciar sesión con esa cuenta.


## <a name="formatstringclaim"></a>FormatStringClaim

Da formato a una notificación de acuerdo con la cadena de formato proporcionada. Esta transformación usa el método `String.Format` de C#.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim |string |ClaimType que actúa como el parámetro {0} de formato de cadena. |
| InputParameter | stringFormat | string | El formato de cadena, incluido el parámetro {0}. Este parámetro de entrada admite [expresiones de transformación de notificaciones de cadena](string-transformations.md#string-claim-transformations-expressions).  |
| OutputClaim | outputClaim | string | El valor ClaimType que se genera después de que se haya invocado esta transformación de notificaciones. |

> [!NOTE]
> El formato de cadena máximo permitido es 4000.

Use esta transformación de notificaciones para dar formato a cualquier cadena con un parámetro {0}. El ejemplo siguiente crea un **userPrincipalName**. Todos los perfiles técnicos de proveedores de las identidades de redes sociales, como `Facebook-OAUTH` llaman a **CreateUserPrincipalName** para generar un **userPrincipalName**.

```xml
<ClaimsTransformation Id="CreateUserPrincipalName" TransformationMethod="FormatStringClaim">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="upnUserName" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringFormat" DataType="string" Value="cpim_{0}@{RelyingPartyTenantId}" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="userPrincipalName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **inputClaim**: 5164db16-3eee-4629-bfda-dcc3326790e9
- Parámetros de entrada:
    - **stringFormat**: cpim_{0}@{RelyingPartyTenantId}
- Notificaciones de salida:
  - **outputClaim**: cpim_5164db16-3eee-4629-bfda-dcc3326790e9@b2cdemo.onmicrosoft.com

## <a name="formatstringmultipleclaims"></a>FormatStringMultipleClaims

Da formato a dos notificaciones de acuerdo con la cadena de formato proporcionada. Esta transformación usa el método `String.Format` de C#.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim |string | ClaimType que actúa como el parámetro {0} de formato de cadena. |
| InputClaim | inputClaim | string | ClaimType que actúa como el parámetro {1} de formato de cadena. |
| InputParameter | stringFormat | string | El formato de cadena, incluidos los parámetros {0} y {1}. Este parámetro de entrada admite [expresiones de transformación de notificaciones de cadena](string-transformations.md#string-claim-transformations-expressions).   |
| OutputClaim | outputClaim | string | El valor ClaimType que se genera después de que se haya invocado esta transformación de notificaciones. |

> [!NOTE]
> El formato de cadena máximo permitido es 4000.

Use esta transformación de notificaciones para dar formato a cualquier cadena con dos parámetros, {0} y {1}. En el ejemplo siguiente se crea un **displayName** con el formato especificado:

```xml
<ClaimsTransformation Id="CreateDisplayNameFromFirstNameAndLastName" TransformationMethod="FormatStringMultipleClaims">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="givenName" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="surName" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringFormat" DataType="string" Value="{0} {1}" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="displayName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **inputClaim1**: Joe
    - **inputClaim2**: Fernando
- Parámetros de entrada:
    - **stringFormat**: {0} {1}
- Notificaciones de salida:
    - **outputClaim**: Joe Fernando

## <a name="getlocalizedstringstransformation"></a>GetLocalizedStringsTransformation

Copia las cadenas localizadas en las notificaciones.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| OutputClaim | Nombre de la cadena localizada. | string | Lista de tipos de notificaciones que se genera después de que se haya invocado esta transformación de notificaciones. |

Para usar la transformación de notificaciones GetLocalizedStringsTransformation:

1. Defina una [cadena de localización](localization.md) y asóciela con un [perfil técnico autoafirmado](self-asserted-technical-profile.md).
1. El `ElementType` del elemento `LocalizedString` debe establecerse en `GetLocalizedStringsTransformationClaimType`.
1. `StringId` es un identificador único que se define y se usa más adelante en la transformación de notificaciones.
1. En la transformación de notificaciones, especifique la lista de notificaciones que se van a establecer con la cadena localizada. `ClaimTypeReferenceId` es una referencia a ClaimType ya definida en la sección ClaimsSchema de la directiva. `TransformationClaimType` es el nombre de la cadena localizada tal como se define en `StringId` del elemento `LocalizedString`.
1. En un [perfil técnico autoafirmado](self-asserted-technical-profile.md) o una transformación de notificaciones de entrada o salida de [control de visualización](display-controls.md), haga una referencia a la transformación de notificaciones.

![GetLocalizedStringsTransformation](./media/string-transformations/get-localized-strings-transformation.png)

En el siguiente ejemplo se busca el asunto, el cuerpo, el mensaje de código y la firma del correo electrónico en las cadenas localizadas. Estas notificaciones se usan posteriormente en la plantilla de comprobación de correo electrónico personalizada.

Defina cadenas localizadas para inglés (predeterminado) y español.

```xml
<Localization Enabled="true">
  <SupportedLanguages DefaultLanguage="en" MergeBehavior="Append">
    <SupportedLanguage>en</SupportedLanguage>
    <SupportedLanguage>es</SupportedLanguage>
   </SupportedLanguages>

  <LocalizedResources Id="api.localaccountsignup.en">
    <LocalizedStrings>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Contoso account email verification code</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Thanks for verifying your account!</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Your code is</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sincerely</LocalizedString>
     </LocalizedStrings>
   </LocalizedResources>
   <LocalizedResources Id="api.localaccountsignup.es">
     <LocalizedStrings>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Código de verificación del correo electrónico de la cuenta de Contoso</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Gracias por comprobar la cuenta de </LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Su código es</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Atentamente</LocalizedString>
    </LocalizedStrings>
  </LocalizedResources>
</Localization>
```

La transformación de notificaciones establece el valor del tipo de notificación *subject* con el valor de `StringId`email_subject *de*.

```xml
<ClaimsTransformation Id="GetLocalizedStringsForEmail" TransformationMethod="GetLocalizedStringsTransformation">
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="subject" TransformationClaimType="email_subject" />
    <OutputClaim ClaimTypeReferenceId="message" TransformationClaimType="email_message" />
    <OutputClaim ClaimTypeReferenceId="codeIntro" TransformationClaimType="email_code" />
    <OutputClaim ClaimTypeReferenceId="signature" TransformationClaimType="email_signature" />
   </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de salida:
  - **subject**: Código de verificación del correo electrónico de la cuenta de Contoso
  - **message**: Gracias por comprobar la cuenta
  - **codeIntro**: Su código es
  - **signature**: Atentamente


## <a name="getmappedvaluefromlocalizedcollection"></a>GetMappedValueFromLocalizedCollection

Buscar un elemento en una colección **Restriction** de notificaciones.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | mapFromClaim | string | La notificación que contiene el texto que se va a buscar en las notificaciones **restrictionValueClaim** con la colección **Restriction**.  |
| OutputClaim | restrictionValueClaim | string | La notificación que contiene la colección **Restriction**. Una vez que se ha invocado la transformación de notificaciones, el valor de esta notificación contiene el valor del elemento seleccionado. |

En el ejemplo siguiente se busca la descripción del mensaje de error según la clave de error. La notificación **responseMsg** contiene una colección de mensajes de error para presentar al usuario final o a enviar al usuario de confianza.

```xml
<ClaimType Id="responseMsg">
  <DisplayName>Error message: </DisplayName>
  <DataType>string</DataType>
  <UserInputType>Paragraph</UserInputType>
  <Restriction>
    <Enumeration Text="B2C_V1_90001" Value="You cannot sign in because you are a minor" />
    <Enumeration Text="B2C_V1_90002" Value="This action can only be performed by gold members" />
    <Enumeration Text="B2C_V1_90003" Value="You have not been enabled for this operation" />
  </Restriction>
</ClaimType>
```
La transformación de notificaciones busca el texto del elemento y devuelve su valor. Si la restricción se localiza mediante `<LocalizedCollection>`, la transformación de notificaciones devuelve el valor localizado.

```xml
<ClaimsTransformation Id="GetResponseMsgMappedToResponseCode" TransformationMethod="GetMappedValueFromLocalizedCollection">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="responseCode" TransformationClaimType="mapFromClaim" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="responseMsg" TransformationClaimType="restrictionValueClaim" />        
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **mapFromClaim**: B2C_V1_90001
- Notificaciones de salida:
    - **restrictionValueClaim**: No puede iniciar sesión porque es menor de edad.

## <a name="lookupvalue"></a>LookupValue

Buscar un valor de notificación en una lista de valores en función del valor de otra notificación.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputParameterId | string | La notificación que contiene el valor de búsqueda. |
| InputParameter | |string | Colección de inputParameters. |
| InputParameter | errorOnFailedLookup | boolean | Controlar si se devuelve un error cuando una búsqueda no tiene resultados. |
| OutputClaim | inputParameterId | string | El valor ClaimType que se genera después de que se haya invocado esta transformación de notificaciones. Valor del `Id` coincidente. |

En el ejemplo siguiente se busca el nombre de dominio en una de las colecciones inputParameters. La transformación de notificaciones busca el nombre de dominio en el identificador y devuelve su valor (un identificador de aplicación).

```xml
 <ClaimsTransformation Id="DomainToClientId" TransformationMethod="LookupValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="domainName" TransformationClaimType="inputParameterId" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="contoso.com" DataType="string" Value="13c15f79-8fb1-4e29-a6c9-be0d36ff19f1" />
    <InputParameter Id="microsoft.com" DataType="string" Value="0213308f-17cb-4398-b97e-01da7bd4804e" />
    <InputParameter Id="test.com" DataType="string" Value="c7026f88-4299-4cdb-965d-3f166464b8a9" />
    <InputParameter Id="errorOnFailedLookup" DataType="boolean" Value="false" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="domainAppId" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **inputParameterId**: test.com
- Parámetros de entrada:
    - **contoso.com**: 13c15f79-8fb1-4e29-a6c9-be0d36ff19f1
    - **microsoft.com**: 0213308f-17cb-4398-b97e-01da7bd4804e
    - **test.com**: c7026f88-4299-4cdb-965d-3f166464b8a9
    - **errorOnFailedLookup**: false
- Notificaciones de salida:
    - **outputClaim**: c7026f88-4299-4cdb-965d-3f166464b8a9

Cuando el parámetro de entrada `errorOnFailedLookup` está establecido en `true`, la transformación de notificaciones **LookupValue** siempre se ejecuta desde un [perfil técnico de validación](validation-technical-profile.md) llamado por un [perfil técnico autofirmado](self-asserted-technical-profile.md) o un elemento [DisplayControl](display-controls.md). Los metadatos `LookupNotFound` de un perfil técnico autoafirmado controlan el mensaje de error que se presenta al usuario.

![Ejecución de AssertStringClaimsAreEqual](./media/string-transformations/assert-execution.png)

En el ejemplo siguiente se busca el nombre de dominio en una de las colecciones inputParameters. La transformación de notificaciones busca el nombre de dominio en el identificador y devuelve su valor (un identificador de aplicación), o bien genera un mensaje de error.

```xml
 <ClaimsTransformation Id="DomainToClientId" TransformationMethod="LookupValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="domainName" TransformationClaimType="inputParameterId" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="contoso.com" DataType="string" Value="13c15f79-8fb1-4e29-a6c9-be0d36ff19f1" />
    <InputParameter Id="microsoft.com" DataType="string" Value="0213308f-17cb-4398-b97e-01da7bd4804e" />
    <InputParameter Id="test.com" DataType="string" Value="c7026f88-4299-4cdb-965d-3f166464b8a9" />
    <InputParameter Id="errorOnFailedLookup" DataType="boolean" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="domainAppId" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **inputParameterId**: live.com
- Parámetros de entrada:
    - **contoso.com**: 13c15f79-8fb1-4e29-a6c9-be0d36ff19f1
    - **microsoft.com**: 0213308f-17cb-4398-b97e-01da7bd4804e
    - **test.com**: c7026f88-4299-4cdb-965d-3f166464b8a9
    - **errorOnFailedLookup**: true
- Error:
    - No se encontró ninguna coincidencia para el valor de notificaciones de entrada en la lista de identificadores de parámetro de entrada y errorOnFailedLookup es true.


## <a name="nullclaim"></a>NullClaim

Limpiar el valor de una notificación determinada.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| OutputClaim | claim_to_null | string | El valor de la notificación está establecido en NULL. |

Use esta transformación de notificaciones para quitar datos innecesarios de la bolsa de propiedades de notificaciones, para que la cookie de la sesión sea menor. En el ejemplo siguiente se quita el valor del tipo de notificación `TermsOfService`.

```xml
<ClaimsTransformation Id="SetTOSToNull" TransformationMethod="NullClaim">
  <OutputClaims>
  <OutputClaim ClaimTypeReferenceId="TermsOfService" TransformationClaimType="claim_to_null" />
  </OutputClaims>
</ClaimsTransformation>
```

- Notificaciones de entrada:
    - **outputClaim**: Bienvenidos a Contoso App. Si continúa explorando y usando este sitio web, acepta las siguientes condiciones generales, que regirán su uso del sitio web...
- Notificaciones de salida:
    - **outputClaim**: NULL

## <a name="parsedomain"></a>ParseDomain

Obtiene la parte de dominio de una dirección de correo electrónico.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | emailAddress | string | ClaimType que contiene la dirección de correo electrónico. |
| OutputClaim | dominio | string | El valor ClaimType que se genera después de que se haya invocado esta transformación de notificaciones: el dominio. |

Use esta transformación de notificaciones para analizar el nombre de dominio después el símbolo @ del usuario. La siguiente transformación de notificaciones muestra cómo analizar el nombre de dominio desde una notificación de **email**.

```xml
<ClaimsTransformation Id="SetDomainName" TransformationMethod="ParseDomain">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="emailAddress" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="domainName" TransformationClaimType="domain" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
  - **emailAddress**: joe@outlook.com
- Notificaciones de salida:
    - **domain**: outlook.com

## <a name="setclaimsifregexmatch"></a>SetClaimsIfRegexMatch

Comprueba que una notificación de cadena `claimToMatch` y el parámetro de entrada `matchTo` son iguales, y establece las notificaciones de salida con el valor presente en el parámetro de entrada `outputClaimIfMatched`, junto con la notificación de salida del resultado de la comparación, que debe establecerse en `true` o `false` según el resultado de la comparación.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| inputClaim | claimToMatch | string | Tipo de la notificación que se va a comparar. |
| InputParameter | matchTo | string | La expresión regular con la que debe coincidir. |
| InputParameter | outputClaimIfMatched | string | Valor que debe establecerse si las cadenas son iguales. |
| InputParameter | extractGroups | boolean | [Opcional] Especifica si la coincidencia de Regex debe extraer los valores de los grupos. Valores posibles: `true` o `false` (valor predeterminado). | 
| OutputClaim | outputClaim | string | Si la expresión regular coincide, esta notificación de salida contiene el valor del parámetro de entrada `outputClaimIfMatched`. Si no hay coincidencia, el valor es NULL. |
| OutputClaim | regexCompareResultClaim | boolean | Tipo de la notificación de salida del resultado de la coincidencia de la expresión regular, que debe establecerse en `true` o `false` en función del resultado de la coincidencia. |
| OutputClaim| El nombre de la notificación| string | Si el parámetro de entrada extractGroups se establece en true, la lista de tipos de notificación que se produce después de que se haya invocado esta transformación de notificaciones. El nombre del tipo de reclamación debe coincidir con el nombre del grupo Regex. | 

### <a name="example-1"></a>Ejemplo 1

Comprueba si el número de teléfono indicado es válido, en función del patrón de expresión regular de número de teléfono.

```xml
<ClaimsTransformation Id="SetIsPhoneRegex" TransformationMethod="SetClaimsIfRegexMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="phone" TransformationClaimType="claimToMatch" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="^[0-9]{4,16}$" />
    <InputParameter Id="outputClaimIfMatched" DataType="string" Value="isPhone" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="validationResult" TransformationClaimType="outputClaim" />
    <OutputClaim ClaimTypeReferenceId="isPhoneBoolean" TransformationClaimType="regexCompareResultClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

- Notificaciones de entrada:
    - **claimToMatch**: "64854114520"
- Parámetros de entrada:
    - **matchTo**: "^[0-9]{4,16}$"
    - **outputClaimIfMatched**: "isPhone"
- Notificaciones de salida:
    - **outputClaim**: "isPhone"
    - **regexCompareResultClaim**: true

### <a name="example-2"></a>Ejemplo 2

Comprueba si la dirección de correo electrónico proporcionada es válida y devuelve el alias de correo electrónico.

```xml
<ClaimsTransformation Id="GetAliasFromEmail" TransformationMethod="SetClaimsIfRegexMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="claimToMatch" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="(?&lt;mailAlias&gt;.*)@(.*)$" />
    <InputParameter Id="outputClaimIfMatched" DataType="string" Value="isEmail" />
    <InputParameter Id="extractGroups" DataType="boolean" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="validationResult" TransformationClaimType="outputClaim" />
    <OutputClaim ClaimTypeReferenceId="isEmailString" TransformationClaimType="regexCompareResultClaim" />
    <OutputClaim ClaimTypeReferenceId="mailAlias" />
  </OutputClaims>
</ClaimsTransformation>
```

- Notificaciones de entrada:
    - **claimToMatch**: "emily@contoso.com"
- Parámetros de entrada:
    - **matchTo**: `(?&lt;mailAlias&gt;.*)@(.*)$`
    - **outputClaimIfMatched**:  "isEmail"
    - **extractGroups**: true
- Notificaciones de salida:
    - **outputClaim**: "isEmail"
    - **regexCompareResultClaim**: true
    - **mailAlias**: emily
    
## <a name="setclaimsifstringsareequal"></a>SetClaimsIfStringsAreEqual

Comprueba que una notificación de cadena y el parámetro de entrada `matchTo` son iguales, y establece las notificaciones de salida con el valor presente en los parámetros de entrada `stringMatchMsg` y `stringMatchMsgCode`, junto con la notificación de salida del resultado de la comparación, que debe establecerse en `true` o `false` según el resultado de la comparación.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | string | Tipo de la notificación que se va a comparar. |
| InputParameter | matchTo | string | Cadena que se va a comparar con `inputClaim`. |
| InputParameter | stringComparison | string | Valores posibles: `Ordinal` o `OrdinalIgnoreCase`. |
| InputParameter | stringMatchMsg | string | Primer valor que debe establecerse si las cadenas son iguales. |
| InputParameter | stringMatchMsgCode | string | Segundo valor que debe establecerse si las cadenas son iguales. |
| OutputClaim | outputClaim1 | string | Si las cadenas son iguales, esta notificación de salida contiene el valor del parámetro de entrada `stringMatchMsg`. |
| OutputClaim | outputClaim2 | string | Si las cadenas son iguales, esta notificación de salida contiene el valor del parámetro de entrada `stringMatchMsgCode`. |
| OutputClaim | stringCompareResultClaim | boolean | Tipo de la notificación de salida del resultado de la comparación, que debe establecerse en `true` o `false` en función del resultado de la comparación. |

Puede usar esta transformación de notificaciones para comprobar si una notificación es igual a un valor que haya especificado. Por ejemplo, la siguiente transformación de notificaciones comprueba si el valor de la notificación **termsOfUseConsentVersion** es igual a `v1`. En caso afirmativo, cambie el valor a `v2`.

```xml
<ClaimsTransformation Id="CheckTheTOS" TransformationMethod="SetClaimsIfStringsAreEqual">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="v1" />
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
    <InputParameter Id="stringMatchMsg" DataType="string" Value="B2C_V1_90005" />
    <InputParameter Id="stringMatchMsgCode" DataType="string" Value="The TOS is upgraded to v2" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="outputClaim1" />
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersionUpgradeCode" TransformationClaimType="outputClaim2" />
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersionUpgradeResult" TransformationClaimType="stringCompareResultClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **inputClaim**: v1
- Parámetros de entrada:
    - **matchTo**: V1
    - **stringComparison**: ordinalIgnoreCase
    - **stringMatchMsg**:  B2C_V1_90005
    - **stringMatchMsgCode**:  The TOS is upgraded to v2
- Notificaciones de salida:
    - **outputClaim1**: B2C_V1_90005
    - **outputClaim2**: The TOS is upgraded to v2
    - **stringCompareResultClaim**: true

## <a name="setclaimsifstringsmatch"></a>SetClaimsIfStringsMatch

Comprueba que una notificación de cadena y el parámetro de entrada `matchTo` son iguales, y establece las notificaciones de salida con el valor presente en el parámetro de entrada `outputClaimIfMatched`, junto con la notificación de salida del resultado de la comparación, que debe establecerse en `true` o `false` según el resultado de la comparación.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | claimToMatch | string | Tipo de la notificación que se va a comparar. |
| InputParameter | matchTo | string | Cadena que se va a comparar con inputClaim. |
| InputParameter | stringComparison | string | Valores posibles: `Ordinal` o `OrdinalIgnoreCase`. |
| InputParameter | outputClaimIfMatched | string | Valor que debe establecerse si las cadenas son iguales. |
| OutputClaim | outputClaim | string | Si las cadenas son iguales, esta notificación de salida contiene el valor del parámetro de entrada `outputClaimIfMatched`. O null, si las cadenas no coinciden. |
| OutputClaim | stringCompareResultClaim | boolean | Tipo de la notificación de salida del resultado de la comparación, que debe establecerse en `true` o `false` en función del resultado de la comparación. |

Por ejemplo, la siguiente transformación de notificaciones comprueba si el valor de la notificación **ageGroup** es igual a `Minor`. En caso afirmativo, devuelve el valor para `B2C_V1_90001`.

```xml
<ClaimsTransformation Id="SetIsMinor" TransformationMethod="SetClaimsIfStringsMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="ageGroup" TransformationClaimType="claimToMatch" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="Minor" />
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
    <InputParameter Id="outputClaimIfMatched" DataType="string" Value="B2C_V1_90001" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isMinor" TransformationClaimType="outputClaim" />
    <OutputClaim ClaimTypeReferenceId="isMinorResponseCode" TransformationClaimType="stringCompareResultClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **claimToMatch**: Minor
- Parámetros de entrada:
    - **matchTo**: Minor
    - **stringComparison**: ordinalIgnoreCase
    - **outputClaimIfMatched**:  B2C_V1_90001
- Notificaciones de salida:
    - **isMinorResponseCode**: B2C_V1_90001
    - **isMinor**: true


## <a name="stringcontains"></a>StringContains

Determina si una subcadena especificada aparece dentro de la notificación de entrada. El resultado es un nuevo ClaimType booleano con un valor de `true` o `false`. `true` si el parámetro de valor aparece dentro de esta cadena; en caso contrario, `false`.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | string | Tipo de la notificación que se va a buscar. |
|InputParameter|contains|string|Valor que se va a buscar.|
|InputParameter|ignoreCase|string|Especifica si la comparación distingue entre mayúsculas y minúsculas en las cadenas que se están comparando.|
| OutputClaim | outputClaim | string | El valor ClaimType que se genera después de que se haya invocado esta ClaimsTransformation. Indicador booleano si la subcadena aparece dentro de la notificación de entrada. |

Use esta transformación de notificaciones para comprobar si un tipo de notificación de cadena contiene una subcadena. En el siguiente ejemplo se comprueba si el tipo de notificación de cadena `roles` contiene el valor **admin**.

```xml
<ClaimsTransformation Id="CheckIsAdmin" TransformationMethod="StringContains">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="roles" TransformationClaimType="inputClaim"/>
  </InputClaims>
  <InputParameters>
    <InputParameter  Id="contains" DataType="string" Value="admin"/>
    <InputParameter  Id="ignoreCase" DataType="string" Value="true"/>
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isAdmin" TransformationClaimType="outputClaim"/>
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **inputClaim**: "Admin, Approver, Editor"
- Parámetros de entrada:
    - **contains**: "admin,"
    - **ignoreCase**: true
- Notificaciones de salida:
    - **outputClaim**: true

## <a name="stringsubstring"></a>StringSubstring

Extrae partes de un tipo de notificación de cadena, comenzando por el carácter que se encuentra en la posición especificada, y devuelve el número de caracteres especificado.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | string | Tipo de notificación que contiene la cadena. |
| InputParameter | startIndex | int | Posición de carácter inicial de base cero de una subcadena en la instancia. |
| InputParameter | length | int | Número de caracteres de la subcadena. |
| OutputClaim | outputClaim | boolean | Cadena equivalente a la subcadena de longitud que comienza en el valor de startIndex de esta instancia, o bien, un valor vacío si el valor de startIndex es igual a la longitud de esta instancia y length es cero. |

Por ejemplo, obtiene el prefijo de país o región del número de teléfono.


```xml
<ClaimsTransformation Id="GetPhonePrefix" TransformationMethod="StringSubstring">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="inputClaim" />
  </InputClaims>
<InputParameters>
  <InputParameter Id="startIndex" DataType="int" Value="0" />
  <InputParameter Id="length" DataType="int" Value="2" />
</InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="phonePrefix" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **inputClaim**: "+1644114520"
- Parámetros de entrada:
    - **startIndex**: 0
    - **length**:  2
- Notificaciones de salida:
    - **outputClaim**: "+1"

## <a name="stringreplace"></a>StringReplace

Busca un valor especificado en una notificación de tipo cadena y devuelve una nueva notificación de tipo cadena en la que todas las apariciones de una cadena especificada en la cadena actual se reemplazan por otra cadena especificada.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | string | Tipo de notificación que contiene la cadena. |
| InputParameter | oldValue | string | Cadena que se va a buscar. |
| InputParameter | newValue | string | Cadena que va a reemplazar todas las apariciones de `oldValue`. |
| OutputClaim | outputClaim | boolean | Cadena que es equivalente a la cadena actual salvo en que todas las instancias de oldValue se reemplazan por el valor de newValue. Si no se encuentra el valor de oldValue en la instancia actual, el método devuelve la instancia actual sin modificar. |

Por ejemplo, normaliza un número de teléfono quitando los caracteres `-`.


```xml
<ClaimsTransformation Id="NormalizePhoneNumber" TransformationMethod="StringReplace">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="inputClaim" />
  </InputClaims>
<InputParameters>
  <InputParameter Id="oldValue" DataType="string" Value="-" />
  <InputParameter Id="newValue" DataType="string" Value="" />
</InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **inputClaim**: "+164-411-452-054"
- Parámetros de entrada:
    - **oldValue**: "-"
    - **newValue**:  ""
- Notificaciones de salida:
    - **outputClaim**: "+164411452054"

## <a name="stringjoin"></a>StringJoin

Concatena los elementos de un tipo de notificación de colección de cadenas especificado, usando el separador indicado entre todos los elementos o miembros.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | stringCollection | Colección que contiene las cadenas que se van a concatenar. |
| InputParameter | delimiter | string | Cadena que se va a usar como separador; por ejemplo, coma `,`. |
| OutputClaim | outputClaim | string | Cadena que consta de los miembros de la colección de cadenas `inputClaim`, delimitadas por el parámetro de entrada `delimiter`. |

En el ejemplo siguiente se toma una colección de cadenas de roles de usuario y se convierte en una cadena con delimitador de comas. Puede usar este método para almacenar una colección de cadenas en la cuenta de usuario de Azure AD. Más adelante, cuando lea la cuenta desde el directorio, podrá usar `StringSplit` para volver a convertir la cadena con delimitador de coma en una colección de cadenas.

```xml
<ClaimsTransformation Id="ConvertRolesStringCollectionToCommaDelimiterString" TransformationMethod="StringJoin">
  <InputClaims>
   <InputClaim ClaimTypeReferenceId="roles" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter DataType="string" Id="delimiter" Value="," />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="rolesCommaDelimiterConverted" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
  - **inputClaim**: [ "Admin", "Author", "Reader" ]
- Parámetros de entrada:
  - **delimiter**: ","
- Notificaciones de salida:
  - **outputClaim**: "Admin,Author,Reader"


## <a name="stringsplit"></a>StringSplit

Devuelve una matriz de cadenas que contiene las subcadenas de esta instancia que están delimitadas por elementos de una cadena especificada.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | string | Tipo de notificación de cadena que contiene las subcadenas que se van a dividir. |
| InputParameter | delimiter | string | Cadena que se va a usar como separador; por ejemplo, coma `,`. |
| OutputClaim | outputClaim | stringCollection | Colección de cadenas cuyos elementos contienen las subcadenas de esta cadena que están delimitadas por el parámetro de entrada `delimiter`. |

En el siguiente ejemplo se toma una cadena con delimitador de coma de roles de usuario y se convierte en una colección de cadenas.

```xml
<ClaimsTransformation Id="ConvertRolesToStringCollection" TransformationMethod="StringSplit">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="rolesCommaDelimiter" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
  <InputParameter DataType="string" Id="delimiter" Value="," />
    </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="roles" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
  - **inputClaim**: "Admin,Author,Reader"
- Parámetros de entrada:
  - **delimiter**: ","
- Notificaciones de salida:
  - **outputClaim**: [ "Admin", "Author", "Reader" ]

## <a name="string-claim-transformations-expressions"></a>Expresiones de transformaciones de notificaciones de cadena
Las expresiones de transformaciones de notificaciones de las directivas personalizadas de Azure AD B2C proporcionan información contextual sobre el identificador de inquilino y el identificador de perfil técnico.

  | Expression | Descripción | Ejemplo |
 | ----- | ----------- | --------|
 | `{TechnicalProfileId}` | Nombre identificador del perfil técnico. | Facebook-OAUTH |
 | `{RelyingPartyTenantId}` | El identificador de inquilino de la directiva del usuario de confianza. | your-tenant.onmicrosoft.com |
 | `{TrustFrameworkTenantId}` | El identificador de inquilino del marco de confianza. | your-tenant.onmicrosoft.com |
