---
title: Ejemplos de transformación de notificaciones de fecha para directivas personalizadas
description: Ejemplos de transformación de notificaciones de fecha para el esquema de Identity Experience Framework (IEF) de Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 02/16/2020
ms.author: mimart
ms.subservice: B2C
ms.custom: b2c-support
ms.openlocfilehash: ccf8c5fceea71c3781ae420c1c36c629ebb97a7b
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110783825"
---
# <a name="date-claims-transformations"></a>Transformaciones de notificaciones de fecha

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

En este artículo se proporcionan ejemplos para usar las transformaciones de notificaciones de fecha del esquema Identity Experience Framework en Azure Active Directory B2C (Azure AD B2C). Para más información, vea [ClaimsTransformations](claimstransformations.md).

## <a name="assertdatetimeisgreaterthan"></a>AssertDateTimeIsGreaterThan

Comprueba que una notificación de fecha y hora (tipo de datos en cadena) es mayor que una segunda notificación de fecha y hora (tipo de datos de cadena) e inicia una excepción.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | leftOperand | string | Tipo de la primera notificación, que debe ser mayor que la segunda notificación. |
| InputClaim | rightOperand | string | Tipo de la segunda notificación, que debe ser menor que la primera notificación. |
| InputParameter | AssertIfEqualTo | boolean | Especifica si esta aserción produce un error si el operando izquierdo es igual al operando derecho. Se producirá un error si el operando izquierdo es igual al operando derecho y el valor se establece en `true`. Valores posibles: `true` (opción predeterminada) o `false`. |
| InputParameter | AssertIfRightOperandIsNotPresent | boolean | Especifica si esta aserción debe pasar si falta el operando derecho. |
| InputParameter | TreatAsEqualIfWithinMillseconds | int | Especifica el número de milisegundos que se permiten entre las dos horas para considerar que las horas son iguales (por ejemplo, para tener en cuenta para el desplazamiento del reloj). |

La transformación de notificaciones **AssertDateTimeIsGreaterThan** siempre se ejecuta desde un [perfil técnico de validación](validation-technical-profile.md) llamado por un [perfil técnico autoafirmado](self-asserted-technical-profile.md). Los metadatos de un perfil técnico autoafirmado **DateTimeGreaterThan** controla el mensaje de error que el perfil técnico presenta al usuario. Los mensajes de error se pueden [localizar](localization-string-ids.md#claims-transformations-error-messages).

![Ejecución de AssertStringClaimsAreEqual](./media/date-transformations/assert-execution.png)

En el ejemplo siguiente se comparan la notificación `currentDateTime` con la notificación `approvedDateTime`. Se produce un error si `currentDateTime` es mayor que `approvedDateTime`. La transformación trata los valores como iguales si están dentro de la diferencia de 5 minutos (30 000 milisegundos). No producirá un error si los valores son iguales porque `AssertIfEqualTo` está establecido en `false`.

```xml
<ClaimsTransformation Id="AssertApprovedDateTimeLaterThanCurrentDateTime" TransformationMethod="AssertDateTimeIsGreaterThan">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="approvedDateTime" TransformationClaimType="leftOperand" />
    <InputClaim ClaimTypeReferenceId="currentDateTime" TransformationClaimType="rightOperand" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="AssertIfEqualTo" DataType="boolean" Value="false" />
    <InputParameter Id="AssertIfRightOperandIsNotPresent" DataType="boolean" Value="true" />
    <InputParameter Id="TreatAsEqualIfWithinMillseconds" DataType="int" Value="300000" />
  </InputParameters>
</ClaimsTransformation>
```

> [!NOTE]
> En el ejemplo anterior, si quita el parámetro de entrada `AssertIfEqualTo` y `currentDateTime` es igual a `approvedDateTime`, se producirá un error. El valor predeterminado de `AssertIfEqualTo` es `true`.
>

El perfil técnico de validación `login-NonInteractive` llama a la transformación de notificaciones `AssertApprovedDateTimeLaterThanCurrentDateTime`.
```xml
<TechnicalProfile Id="login-NonInteractive">
  ...
  <OutputClaimsTransformations>
    <OutputClaimsTransformation ReferenceId="AssertApprovedDateTimeLaterThanCurrentDateTime" />
  </OutputClaimsTransformations>
</TechnicalProfile>
```

El perfil técnico autoafirmado llama al perfil técnico **login-NonInteractive** de validación.

```xml
<TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
  <Metadata>
    <Item Key="DateTimeGreaterThan">Custom error message if the provided left operand is greater than the right operand.</Item>
  </Metadata>
  <ValidationTechnicalProfiles>
    <ValidationTechnicalProfile ReferenceId="login-NonInteractive" />
  </ValidationTechnicalProfiles>
</TechnicalProfile>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **leftOperand**: 2020-03-01T15:00:00.0000000Z
    - **rightOperand**: 2020-03-01T14:00:00.0000000Z
- Resultado: aparece un error

## <a name="convertdatetodatetimeclaim"></a>ConvertDateToDateTimeClaim

Convierte un ClaimType **Date** en un ClaimType **DateTime**. La transformación de notificaciones convierte el formato de hora y agrega 12:00:00 a. m. a la fecha.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | date | ClaimType que se va a convertir. |
| OutputClaim | outputClaim | dateTime | El valor ClaimType que se genera después de que se haya invocado esta ClaimsTransformation. |

En el ejemplo siguiente se muestra la conversión de la notificación `dateOfBirth` (tipo de datos date) en otra notificación `dateOfBirthWithTime` (tipo de datos dateTime).

```xml
  <ClaimsTransformation Id="ConvertToDateTime" TransformationMethod="ConvertDateToDateTimeClaim">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="dateOfBirth" TransformationClaimType="inputClaim" />
    </InputClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="dateOfBirthWithTime" TransformationClaimType="outputClaim" />
    </OutputClaims>
  </ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **inputClaim**: 2020-15-03
- Notificaciones de salida:
    - **outputClaim**: 2020-15-03T00:00:00.0000000Z

## <a name="convertdatetimetodateclaim"></a>ConvertDateTimeToDateClaim

Convierte un ClaimType **DateTime** en un ClaimType **Date**. La transformación de notificaciones quita el formato de hora de la fecha.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | dateTime | ClaimType que se va a convertir. |
| OutputClaim | outputClaim | date | El valor ClaimType que se genera después de que se haya invocado esta ClaimsTransformation. |

En el ejemplo siguiente se muestra la conversión de la notificación `systemDateTime` (tipo de datos dateTime) en otra notificación `systemDate` (tipo de datos date).

```xml
<ClaimsTransformation Id="ConvertToDate" TransformationMethod="ConvertDateTimeToDateClaim">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="systemDateTime" TransformationClaimType="inputClaim" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="systemDate" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
  - **inputClaim**: 2020-15-03T11:34:22.0000000Z
- Notificaciones de salida:
  - **outputClaim**: 2020-15-03

## <a name="getcurrentdatetime"></a>GetCurrentDateTime

Obtenga la fecha y la hora UTC actual y agregue el valor a ClaimType.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| OutputClaim | currentDateTime | dateTime | El valor ClaimType que se genera después de que se haya invocado esta ClaimsTransformation. |

```xml
<ClaimsTransformation Id="GetSystemDateTime" TransformationMethod="GetCurrentDateTime">
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="systemDateTime" TransformationClaimType="currentDateTime" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

* Notificaciones de salida:
    * **currentDateTime**: 2020-15-03T11:40:35.0000000Z

## <a name="datetimecomparison"></a>DateTimeComparison

Determine si un valor dateTime es mayor, menor o igual que otro. El resultado es un nuevo ClaimType booleano con un valor de `true` o `false`.

| Elemento | TransformationClaimType | Tipo de datos | Notas |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | firstDateTime | dateTime | El primer valor de dateTime para comparar si es anterior o posterior al segundo valor de dateTime. Un valor nulo inicia una excepción. |
| InputClaim | secondDateTime | dateTime | El segundo valor de dateTime para comparar si es anterior o posterior al primer valor de dateTime. El valor NULL se trata como el valor de dateTime actual. |
| InputParameter | operator | string | Uno de los siguientes valores: same, later than o earlier than. |
| InputParameter | timeSpanInSeconds | int | Agregue el intervalo de tiempo a la primera fecha y hora. |
| OutputClaim | resultado | boolean | El valor ClaimType que se genera después de que se haya invocado esta ClaimsTransformation. |

Use esta transformación de notificaciones para determinar si dos valores ClaimType son iguales, mayores o menores entre sí. Por ejemplo, puede almacenar la última vez que un usuario ha aceptado los términos del servicio (TOS). Después de 3 meses, puede pedir al usuario que vuelva a acceder a los TOS.
Para ejecutar la transformación de notificaciones, primero deberá obtener el valor dateTime actual y la última vez que el usuario aceptó los TOS.

```xml
<ClaimsTransformation Id="CompareLastTOSAcceptedWithCurrentDateTime" TransformationMethod="DateTimeComparison">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="currentDateTime" TransformationClaimType="firstDateTime" />
    <InputClaim ClaimTypeReferenceId="extension_LastTOSAccepted" TransformationClaimType="secondDateTime" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="operator" DataType="string" Value="later than" />
    <InputParameter Id="timeSpanInSeconds" DataType="int" Value="7776000" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isLastTOSAcceptedGreaterThanNow" TransformationClaimType="result" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Ejemplo

- Notificaciones de entrada:
    - **firstDateTime**: 2020-01-01T00:00:00.100000Z
    - **secondDateTime**: 2020-04-01T00:00:00.100000Z
- Parámetros de entrada:
    - **operator**: mayor que
    - **timeSpanInSeconds**: 7776000 (90 días)
- Notificaciones de salida:
    - **result**: true
