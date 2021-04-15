---
title: Administración de directivas personalizadas con PowerShell
titleSuffix: Azure AD B2C
description: Use el cmdlet de PowerShell de Azure Active Directory (Azure AD) para la administración mediante programación de las directivas personalizadas de Azure AD B2C. Cree, lea, actualice y elimine directivas personalizadas con PowerShell.
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/14/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: cbe465c41363e8c0779440affeae78f999b4632d
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107257439"
---
# <a name="manage-azure-ad-b2c-custom-policies-with-azure-powershell"></a>Administración de las directivas personalizadas de Azure AD B2C con Azure PowerShell

Azure PowerShell proporciona varios cmdlets para la administración de directivas personalizadas basadas en scripts y de línea de comandos en el inquilino e Azure AD B2C. Aprenda a usar el módulo de PowerShell de Azure AD para:

* Enumerar las directivas personalizadas en un inquilino de Azure AD B2C
* Descargar una directiva de un inquilino
* Sobrescribir el contenido de una directiva existente para actualizarla
* Cargar una nueva directiva en el inquilino de Azure AD B2C
* Eliminar una directiva personalizada de un inquilino

## <a name="prerequisites"></a>Prerrequisitos

* [Inquilino de Azure AD B2C](tutorial-create-tenant.md) y credenciales para un usuario en el directorio con el rol [Administrador de directivas IEF de B2C](../active-directory/roles/permissions-reference.md#b2c-ief-policy-administrator)
* [Directivas personalizadas](tutorial-create-user-flows.md?pivots=b2c-custom-policy) cargadas en el inquilino
* [**Módulo de versión preliminar** de Azure AD PowerShell for Graph](/powershell/azure/active-directory/install-adv2)

## <a name="connect-powershell-session-to-b2c-tenant"></a>Conexión de la sesión de PowerShell con el inquilino de B2C

Para trabajar con directivas personalizadas en el inquilino de Azure AD B2C,primero debe conectar la sesión de PowerShell al inquilino con el comando [Connect-AzureAD][Connect-AzureAD].

Sustituya `{b2c-tenant-name}` por el nombre del inquilino de Azure AD B2C para ejecutar el siguiente comando. Inicie sesión con una cuenta asignada al rol [Administrador de directivas de B2C IEF](../active-directory/roles/permissions-reference.md#b2c-ief-policy-administrator) en el directorio.

```PowerShell
Connect-AzureAD -Tenant "{b2c-tenant-name}.onmicrosoft.com"
```

El ejemplo de salida del comando muestra un inicio de sesión correcto:

```Console
PS C:\> Connect-AzureAD -Tenant "contosob2c.onmicrosoft.com"

Account               Environment TenantId                             TenantDomain                 AccountType
-------               ----------- --------                             ------------                 -----------
azureuser@contoso.com AzureCloud  00000000-0000-0000-0000-000000000000 contosob2c.onmicrosoft.com   User
```

## <a name="list-all-custom-policies-in-the-tenant"></a>Enumeración de todas las directivas personalizadas del inquilino

La detección de directivas personalizadas permite que un administrador de Azure AD B2C revise, administre y agregue lógica de negocios a las operaciones. Use el comando [Get-AzureADMSTrustFrameworkPolicy][Get-AzureADMSTrustFrameworkPolicy] para devolver una lista de los identificadores de las directivas personalizadas en un inquilino de Azure AD B2C.

```PowerShell
Get-AzureADMSTrustFrameworkPolicy
```

Ejemplo de salida del comando:

```Console
PS C:\> Get-AzureADMSTrustFrameworkPolicy

Id
--
B2C_1A_TrustFrameworkBase
B2C_1A_TrustFrameworkExtensions
B2C_1A_signup_signin
B2C_1A_ProfileEdit
B2C_1A_PasswordReset
```

## <a name="download-a-policy"></a>Descarga de una directiva

Tras revisar la lista de id. de directivas, puede abordar una directiva específica con [Get-AzureADMSTrustFrameworkPolicy][Get-AzureADMSTrustFrameworkPolicy] para descargar su contenido.

```PowerShell
Get-AzureADMSTrustFrameworkPolicy [-Id <policyId>]
```

En este ejemplo, la directiva con id. *B2C_1A_signup_signin* está descargada:

```Console
PS C:\> Get-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin
<TrustFrameworkPolicy xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06" PolicySchemaVersion="0.3.0.0" TenantId="contosob2c.onmicrosoft.com" PolicyId="B2C_1A_signup_signin" PublicPolicyUri="http://contosob2c.onmicrosoft.com/B2C_1A_signup_signin" TenantObjectId="00000000-0000-0000-0000-000000000000">
  <BasePolicy>
    <TenantId>contosob2c.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>
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
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```

Para editar el contenido de la directiva localmente, canalice la salida del comando en un archivo con el argumento `-OutputFilePath` y abra el archivo en el editor que prefiera.

Ejemplo de comando enviando una salida a un archivo:

```PowerShell
# Download and send policy output to a file
Get-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin -OutputFilePath C:\RPPolicy.xml
```

## <a name="update-an-existing-policy"></a>Actualización de una directiva existente

Tras editar un archivo de una directiva que ha creado o descargado, puede publicar la directiva actualizada en Azure AD B2C con el comando [Set-AzureADMSTrustFrameworkPolicy][Set-AzureADMSTrustFrameworkPolicy].

Si emite el comando `Set-AzureADMSTrustFrameworkPolicy` con el id. de una directiva que ya existe en el inquilino de Azure AD B2C, el contenido de dicha directiva se sobrescribe.

```PowerShell
Set-AzureADMSTrustFrameworkPolicy [-Id <policyId>] -InputFilePath <inputpolicyfilePath> [-OutputFilePath <outputFilePath>]
```

Comando de ejemplo:

```PowerShell
# Update an existing policy from file
Set-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin -InputFilePath C:\B2C_1A_signup_signin.xml
```

Para ver más ejemplos, consulte la referencia del comando [Set-AzureADMSTrustFrameworkPolicy][Set-AzureADMSTrustFrameworkPolicy].

## <a name="upload-a-new-policy"></a>Carga de una nueva directiva

Al realizar un cambio en una directiva personalizada que se ejecuta en producción, puede que quiera publicar varias versiones de esta para escenarios de prueba A/B o reserva. Puede que también quiera realizar una copia de una directiva existente, modificarla con un ligero cambio y, luego, cargarla como una nueva directiva para usarla en otra aplicación.

Use el comando [New-AzureADMSTrustFrameworkPolicy][New-AzureADMSTrustFrameworkPolicy] para cargar una nueva directiva:

```PowerShell
New-AzureADMSTrustFrameworkPolicy -InputFilePath <inputpolicyfilePath> [-OutputFilePath <outputFilePath>]
```

Comando de ejemplo:

```PowerShell
# Add new policy from file
New-AzureADMSTrustFrameworkPolicy -InputFilePath C:\SignUpOrSignInv2.xml
```

## <a name="delete-a-custom-policy"></a>Eliminación de una directiva personalizada

Para mantener un ciclo de vida de operaciones limpio, se recomienda quitar periódicamente las directivas personalizadas que no se usen. Por ejemplo, puede que quiera quitar las anteriores versiones de una directiva tras realizar una migración a un nuevo conjunto de directivas y verificar la funcionalidad de las nuevas directivas. Además, si intenta publicar un conjunto de directivas personalizadas y recibe un error, puede que sea conveniente quitar las directivas que se crearon como parte de la versión con errores.

Use el comando [Remove-AzureADMSTrustFrameworkPolicy][Remove-AzureADMSTrustFrameworkPolicy] para eliminar una directiva del inquilino.

```PowerShell
Remove-AzureADMSTrustFrameworkPolicy -Id <policyId>
```

Comando de ejemplo:

```PowerShell
# Delete an existing policy
Remove-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin
```

## <a name="troubleshoot-policy-upload"></a>Solución de problemas de carga de las directivas

Al intentar publicar una nueva directiva personalizada o actualizar una directiva existente, errores y formatos XML inadecuados en la cadena de herencia de los archivos de directiva pueden provocar errores de validación.

Por ejemplo, a continuación se indica un intento de actualizar una directiva con contenido que incluye XML con estructura incorrecta (la salida está truncada para que sea breve):

```Console
PS C:\> Set-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin -InputFilePath C:\B2C_1A_signup_signin.xml
Set-AzureADMSTrustFrameworkPolicy : Error occurred while executing PutTrustFrameworkPolicy
Code: AADB2C
Message: Validation failed: 1 validation error(s) found in policy "B2C_1A_SIGNUP_SIGNIN" of tenant "contosob2c.onmicrosoft.com".Schema validation error found at line
14 col 55 in policy "B2C_1A_SIGNUP_SIGNIN" of tenant "contosob2c.onmicrosoft.com": The element 'OutputClaims' in namespace
'http://schemas.microsoft.com/online/cpim/schemas/2013/06' cannot contain text. List of possible elements expected: 'OutputClaim' in namespace
'http://schemas.microsoft.com/online/cpim/schemas/2013/06'.
...
```

Para información sobre la solución de problemas de directivas personalizadas, consulte [Solución de problemas de directivas personalizadas de Azure AD B2C e Identity Experience Framework](./troubleshoot-custom-policies.md).

## <a name="next-steps"></a>Pasos siguientes

Para información sobre el uso de PowerShell para implementar directivas personalizadas como parte de una canalización de integración o entrega continuas (CI/CD), consulte [Implementación de directivas personalizadas desde una canalización de Azure DevOps](deploy-custom-policies-devops.md).

<!-- LINKS - External -->
[Connect-AzureAD]: /powershell/module/azuread/get-azureadmstrustframeworkpolicy
[Get-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/get-azureadmstrustframeworkpolicy
[New-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/new-azureadmstrustframeworkpolicy
[Remove-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/remove-azureadmstrustframeworkpolicy
[Set-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/set-azureadmstrustframeworkpolicy
