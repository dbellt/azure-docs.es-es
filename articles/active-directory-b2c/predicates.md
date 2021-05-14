---
title: Predicates y PredicateValidations
titleSuffix: Azure AD B2C
description: Impida que se agreguen datos con formato incorrecto al inquilino de Azure AD B2C, mediante el uso de directivas personalizadas en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: d5317edcf7ef00c6d82c1799582ad2a92005b7cb
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108161634"
---
# <a name="predicates-and-predicatevalidations"></a>Predicates y PredicateValidations

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Los elementos **Predicates** y **PredicateValidations** le permiten realizar un proceso de validación para asegurarse de que solo se introducen datos con el formato adecuado en el inquilino de Azure Active Directory B2C (Azure AD B2C).

En el siguiente diagrama se muestra la relación entre los elementos:

![Diagrama que muestra las relaciones de los elementos Predicates y PredicateValidations](./media/predicates/predicates.png)

## <a name="predicates"></a>Predicados

El elemento **Predicate** define una validación básica para comprobar el valor de un tipo de notificación y devuelve `true` o `false`. La validación se realiza mediante el uso de un elemento **Method** especificado y un conjunto de elementos **Parameter** relevantes para el método. Por ejemplo, un predicado puede comprobar si la longitud de un valor de notificación de cadena está dentro del intervalo de parámetros mínimos y máximos especificados, o si un valor de notificación de cadena contiene un juego de caracteres. Si se produce un error en la comprobación, el elemento **UserHelpText** proporciona un mensaje de error para los usuarios. El valor del elemento **UserHelpText** se puede localizar mediante la [personalización de idioma](localization.md).

El elemento **Predicados** debe aparecer directamente después del elemento **ClaimsSchema** en el elemento [BuildingBlocks](buildingblocks.md).

El elemento **Predicates** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| Predicate | 1:n | Una lista de predicados. |

El elemento **Predicate** contiene los siguientes atributos:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Identificador | Sí | Un identificador que se usa para el predicado. Hay otros elementos que pueden usar este identificador en la directiva. |
| Método | Sí | El tipo de método que se usará para la validación. Valores posibles: [IsLengthRange](#islengthrange), [MatchesRegex](#matchesregex), [IncludesCharacters](#includescharacters) o [IsDateRange](#isdaterange).  |
| HelpText | No | S se produce un error en la comprobación, un mensaje de error para los usuarios. Esta cadena se puede localizar con la [personalización de idioma](localization.md). |

El elemento **Predicate** contiene los siguientes elementos:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| UserHelpText | 0:1 | (En desuso) Un mensaje de error para los usuarios si se produce un error en la comprobación. |
| Parámetros | 1:1 | Parámetros para el tipo de método de la validación de cadenas. |

El elemento **Parameters** contiene los siguientes elementos:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| Parámetro | 1:n | Parámetros para el tipo de método de la validación de cadenas. |

El elemento **Parameter** contiene los siguientes atributos:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| Identificador | 1:1 | Identificador del parámetro. |

### <a name="predicate-methods"></a>Métodos de predicado

#### <a name="islengthrange"></a>IsLengthRange

El método IsLengthRange comprueba si la longitud de un valor de notificación de cadena está dentro del intervalo de parámetros mínimos y máximos especificados. El elemento de predicado admite los siguientes parámetros:

| Parámetro | Obligatorio | Descripción |
| ------- | ----------- | ----------- |
| Máxima | Sí | Número máximo de caracteres que se pueden especificar. |
| Mínima | Sí | Número mínimo de caracteres que se deben especificar. |

En el ejemplo siguiente se muestra un método IsLengthRange con los parámetros `Minimum` y `Maximum` que especifican el intervalo de longitud de la cadena:

```xml
<Predicate Id="IsLengthBetween8And64" Method="IsLengthRange" HelpText="The password must be between 8 and 64 characters.">
  <Parameters>
    <Parameter Id="Minimum">8</Parameter>
    <Parameter Id="Maximum">64</Parameter>
  </Parameters>
</Predicate>
```

#### <a name="matchesregex"></a>MatchesRegex

El método MatchesRegex comprueba si un valor de notificación de cadena coincide con una expresión regular. El elemento de predicado admite los siguientes parámetros:

| Parámetro | Obligatorio | Descripción |
| ------- | ----------- | ----------- |
| RegularExpression | Sí | Patrón de expresión regular del que van a buscarse coincidencias. |

En el ejemplo siguiente se muestra un método `MatchesRegex` con el parámetro `RegularExpression` que especifica una expresión regular:

```xml
<Predicate Id="PIN" Method="MatchesRegex" HelpText="The password must be numbers only.">
  <Parameters>
    <Parameter Id="RegularExpression">^[0-9]+$</Parameter>
  </Parameters>
</Predicate>
```

#### <a name="includescharacters"></a>IncludesCharacters

El método IncludesCharacters comprueba si un valor de notificación de cadena contiene un juego de caracteres. El elemento de predicado admite los siguientes parámetros:

| Parámetro | Obligatorio | Descripción |
| ------- | ----------- | ----------- |
| CharacterSet | Sí | El conjunto de caracteres que se pueden especificar. Por ejemplo, caracteres en minúscula `a-z`, caracteres en mayúscula `A-Z`, dígitos `0-9` o una lista de símbolos, como `@#$%^&amp;*\-_+=[]{}|\\:',?/~"();!`. |

En el ejemplo siguiente se muestra un método `IncludesCharacters` con el parámetro `CharacterSet` que especifica un juego de caracteres:

```xml
<Predicate Id="Lowercase" Method="IncludesCharacters" HelpText="a lowercase letter">
  <Parameters>
    <Parameter Id="CharacterSet">a-z</Parameter>
  </Parameters>
</Predicate>
```

#### <a name="isdaterange"></a>IsDateRange

El método IsDateRange comprueba si un valor de notificación de fecha se encuentra dentro de un intervalo de parámetros mínimos y máximos especificados. El elemento de predicado admite los siguientes parámetros:

| Parámetro | Obligatorio | Descripción |
| ------- | ----------- | ----------- |
| Máxima | Sí | La fecha más larga posible que se puede especificar. El formato de la fecha sigue la convención `yyyy-mm-dd` o `Today`. |
| Mínima | Sí | La fecha más corta posible que se puede especificar. El formato de la fecha sigue la convención `yyyy-mm-dd` o `Today`.|

En el ejemplo siguiente se muestra un método `IsDateRange` con los parámetros `Minimum` y `Maximum` que especifican el intervalo de fechas con un formato de `yyyy-mm-dd` y `Today`.

```xml
<Predicate Id="DateRange" Method="IsDateRange" HelpText="The date must be between 1970-01-01 and today.">
  <Parameters>
    <Parameter Id="Minimum">1970-01-01</Parameter>
    <Parameter Id="Maximum">Today</Parameter>
  </Parameters>
</Predicate>
```

## <a name="predicatevalidations"></a>PredicateValidations

Mientras que los predicados definen la validación que se va a comparar con un tipo de notificación, **PredicateValidations** agrupa un conjunto de predicados para formar una validación de entrada de usuario que se puede aplicar a un tipo de notificación. Cada elemento **PredicateValidation** contiene un conjunto de elementos **PredicateGroup** que contienen un conjunto de elementos **PredicateReference** que dirigen a un elemento **Predicate**. Para pasar la validación, el valor de la notificación debe superar todas las pruebas de cualquier predicado en todo el **PredicateGroup** con su conjunto de elementos **PredicateReference**.

El elemento **PredicateValidations** debe aparecer directamente después del elemento **Predicados** en el elemento [BuildingBlocks](buildingblocks.md).

```xml
<PredicateValidations>
  <PredicateValidation Id="">
    <PredicateGroups>
      <PredicateGroup Id="">
        <UserHelpText></UserHelpText>
        <PredicateReferences MatchAtLeast="">
          <PredicateReference Id="" />
          ...
        </PredicateReferences>
      </PredicateGroup>
      ...
    </PredicateGroups>
  </PredicateValidation>
...
</PredicateValidations>
```

El elemento **PredicateValidations** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| PredicateValidation | 1:n | Una lista de validación de predicados. |

El elemento **PredicateValidation** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Identificador | Sí | Un identificador que se usa para la validación del predicado. El elemento **ClaimType** puede usar este identificador en la directiva. |

El elemento **PredicateValidation** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| PredicateGroups | 1:n | Una lista de grupos de predicados. |

El elemento **PredicateGroups** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| PredicateGroup | 1:n | Una lista de predicados. |

El elemento **PredicateGroup** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Identificador | Sí | Un identificador que se usa para el grupo de predicados.  |

El elemento **PredicateGroup** contiene los elementos siguientes:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| UserHelpText | 0:1 |  Una descripción del predicado que puede ser útil para que los usuarios sepan qué valor deben introducir. |
| PredicateReferences | 1:n | Una lista de referencias de predicados. |

El elemento **PredicateReferences** contiene los siguientes atributos:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| MatchAtLeast | No | Especifica que el valor debe coincidir por lo menos con ese número de definiciones de predicado para que la entrada se acepte. Si no se especifica, el valor debe coincidir con todas las definiciones de predicado. |

El elemento **PredicateReferences** contiene los siguientes elementos:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| PredicateReference | 1:n | Una referencia a un predicado. |

El elemento **PredicateReference** contiene los siguientes atributos:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Identificador | Sí | Un identificador que se usa para la validación del predicado.  |

## <a name="configure-password-complexity"></a>Configurar la complejidad de la contraseña

Con **Predicates** y **PredicateValidationsInput** puede controlar los requisitos de complejidad para las contraseñas proporcionadas por un usuario al crear una cuenta. De forma predeterminada, Azure AD B2C utiliza contraseñas seguras. Azure AD B2C también admite opciones de configuración para controlar la complejidad de las contraseñas que los clientes pueden usar. Puede definir la complejidad de la contraseña mediante el uso de estos predicados:

- **IsLengthBetween8And64** con el método `IsLengthRange` valida que la contraseña debe tener entre 8 y 64 caracteres.
- **Lowercase** con el método `IncludesCharacters` valida que la contraseña contiene una letra minúscula.
- **Uppercase** con el método `IncludesCharacters` valida que la contraseña contiene una letra mayúscula.
- **Number** con el método `IncludesCharacters` valida que la contraseña contiene un dígito.
- **Symbol** con el método `IncludesCharacters` valida que la contraseña contiene uno de los diversos caracteres de símbolos.
- **PIN** con el método `MatchesRegex` valida que la contraseña solo contiene números.
- **AllowedAADCharacters** con el método `MatchesRegex` valida que se ha proporcionado el único carácter no válido de la contraseña.
- **DisallowedWhitespace** con el método `MatchesRegex` valida que la contraseña no empiece ni termine con un carácter de espacio en blanco.

```xml
<Predicates>
  <Predicate Id="IsLengthBetween8And64" Method="IsLengthRange" HelpText="The password must be between 8 and 64 characters.">
    <Parameters>
      <Parameter Id="Minimum">8</Parameter>
      <Parameter Id="Maximum">64</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Lowercase" Method="IncludesCharacters" HelpText="a lowercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">a-z</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Uppercase" Method="IncludesCharacters" HelpText="an uppercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">A-Z</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Number" Method="IncludesCharacters" HelpText="a digit">
    <Parameters>
      <Parameter Id="CharacterSet">0-9</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Symbol" Method="IncludesCharacters" HelpText="a symbol">
    <Parameters>
      <Parameter Id="CharacterSet">@#$%^&amp;*\-_+=[]{}|\\:',.?/`~"();!</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="PIN" Method="MatchesRegex" HelpText="The password must be numbers only.">
    <Parameters>
      <Parameter Id="RegularExpression">^[0-9]+$</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="AllowedAADCharacters" Method="MatchesRegex" HelpText="An invalid character was provided.">
    <Parameters>
      <Parameter Id="RegularExpression">(^([0-9A-Za-z\d@#$%^&amp;*\-_+=[\]{}|\\:',?/`~"();! ]|(\.(?!@)))+$)|(^$)</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="DisallowedWhitespace" Method="MatchesRegex" HelpText="The password must not begin or end with a whitespace character.">
    <Parameters>
      <Parameter Id="RegularExpression">(^\S.*\S$)|(^\S+$)|(^$)</Parameter>
    </Parameters>
  </Predicate>
```

Después de definir las validaciones básicas, puede combinarlas y crear un conjunto de directivas de contraseña que puede usar en la directiva:

- **SimplePassword** valida los predicados DisallowedWhitespace, AllowedAADCharacters y IsLengthBetween8And64.
- **StrongPassword** valida los predicados DisallowedWhitespace, AllowedAADCharacters y IsLengthBetween8And64. El último grupo `CharacterClasses` ejecuta un conjunto adicional de predicados con `MatchAtLeast` establecido en 3. La contraseña del usuario debe tener entre 8 y 16 caracteres, y estar formado por los siguientes caracteres: minúsculas, mayúsculas, número o símbolo.
- **CustomPassword** solo valida DisallowedWhitespace y AllowedAADCharacters. Por lo tanto, el usuario puede proporcionar cualquier contraseña de cualquier longitud, siempre y cuando los caracteres sean válidos.

```xml
<PredicateValidations>
  <PredicateValidation Id="SimplePassword">
    <PredicateGroups>
      <PredicateGroup Id="DisallowedWhitespaceGroup">
        <PredicateReferences>
          <PredicateReference Id="DisallowedWhitespace" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="AllowedAADCharactersGroup">
        <PredicateReferences>
          <PredicateReference Id="AllowedAADCharacters" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="LengthGroup">
        <PredicateReferences>
          <PredicateReference Id="IsLengthBetween8And64" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>

  <PredicateValidation Id="StrongPassword">
    <PredicateGroups>
      <PredicateGroup Id="DisallowedWhitespaceGroup">
        <PredicateReferences>
          <PredicateReference Id="DisallowedWhitespace" />
       </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="AllowedAADCharactersGroup">
        <PredicateReferences>
          <PredicateReference Id="AllowedAADCharacters" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="LengthGroup">
        <PredicateReferences>
          <PredicateReference Id="IsLengthBetween8And64" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="CharacterClasses">
        <UserHelpText>The password must have at least 3 of the following:</UserHelpText>
        <PredicateReferences MatchAtLeast="3">
          <PredicateReference Id="Lowercase" />
          <PredicateReference Id="Uppercase" />
          <PredicateReference Id="Number" />
          <PredicateReference Id="Symbol" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>

  <PredicateValidation Id="CustomPassword">
    <PredicateGroups>
      <PredicateGroup Id="DisallowedWhitespaceGroup">
        <PredicateReferences>
          <PredicateReference Id="DisallowedWhitespace" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="AllowedAADCharactersGroup">
        <PredicateReferences>
          <PredicateReference Id="AllowedAADCharacters" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>
</PredicateValidations>
```

En el tipo de notificación, agregue el elemento **PredicateValidationReference** y especifique el identificador como una de las validaciones de predicado, como SimplePassword, StrongPassword o CustomPassword.

```xml
<ClaimType Id="password">
  <DisplayName>Password</DisplayName>
  <DataType>string</DataType>
  <AdminHelpText>Enter password</AdminHelpText>
  <UserHelpText>Enter password</UserHelpText>
  <UserInputType>Password</UserInputType>
  <PredicateValidationReference Id="StrongPassword" />
</ClaimType>
```

A continuación, se muestra cómo se organizan los elementos cuando Azure AD B2C muestra el mensaje de error siguiente:

![Diagrama de ejemplo de la complejidad de la contraseña de los elementos Predicate y PredicateGroup](./media/predicates/predicates-pass.png)

## <a name="configure-a-date-range"></a>Configurar un intervalo de fechas

Con los elementos **Predicates** y **PredicateValidations** puede controlar los valores de fecha mínimos y máximos de **UserInputType** utilizando `DateTimeDropdown`. Para ello, cree un **Predicate** con el método `IsDateRange` y especifique los parámetros mínimos y máximos.

```xml
<Predicates>
  <Predicate Id="DateRange" Method="IsDateRange" HelpText="The date must be between 01-01-1980 and today.">
    <Parameters>
      <Parameter Id="Minimum">1980-01-01</Parameter>
      <Parameter Id="Maximum">Today</Parameter>
    </Parameters>
  </Predicate>
</Predicates>
```

Agregue un **PredicateValidation** con una referencia al predicado `DateRange`.

```xml
<PredicateValidations>
  <PredicateValidation Id="CustomDateRange">
    <PredicateGroups>
      <PredicateGroup Id="DateRangeGroup">
        <PredicateReferences>
          <PredicateReference Id="DateRange" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>
</PredicateValidations>
```

En el tipo de notificación, agregue el elemento **PredicateValidationReference** y especifique el identificador como `CustomDateRange`.

```xml
<ClaimType Id="dateOfBirth">
  <DisplayName>Date of Birth</DisplayName>
  <DataType>date</DataType>
  <AdminHelpText>The user's date of birth.</AdminHelpText>
  <UserHelpText>Your date of birth.</UserHelpText>
  <UserInputType>DateTimeDropdown</UserInputType>
  <PredicateValidationReference Id="CustomDateRange" />
</ClaimType>
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre la [Configuración de la complejidad de contraseñas mediante directivas personalizadas en Azure Active Directory B2C](password-complexity.md) con validaciones de predicado.
