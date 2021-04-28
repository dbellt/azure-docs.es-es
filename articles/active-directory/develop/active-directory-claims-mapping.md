---
title: Personalización de las notificaciones de aplicación del inquilino de Azure AD (PowerShell)
titleSuffix: Microsoft identity platform
description: En esta página se describe la asignación de notificaciones de Azure Active Directory.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: how-to
ms.date: 08/25/2020
ms.author: ryanwi
ms.reviewer: paulgarn, hirsin, jeedes, luleon
ms.openlocfilehash: e77155f8a6efd3916ae90fcb562d688bb5b5126f
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2021
ms.locfileid: "107598897"
---
# <a name="how-to-customize-claims-emitted-in-tokens-for-a-specific-app-in-a-tenant-preview"></a>Procedimientos: Personalizar las notificaciones emitidas en tokens para una determinada aplicación de un inquilino (versión preliminar)

> [!NOTE]
> Esta característica reemplaza a la [personalización de notificaciones](active-directory-saml-claims-customization.md) que se ofrece actualmente a través del portal. Si en la misma aplicación personaliza las notificaciones mediante el portal y usa al mismo tiempo el método de Graph/PowerShell que se detalla en este documento, los tokens emitidos para esa aplicación harán caso omiso de la configuración del portal. Las configuraciones realizadas mediante los métodos que se detallan en este documento no se reflejarán en el portal.

> [!NOTE]
> Esta funcionalidad se encuentra actualmente en versión preliminar pública. Debe estar preparado para deshacer o eliminar los cambios. La característica está disponible en cualquier suscripción de Azure Active Directory (Azure AD) durante el período de versión preliminar pública. Pero cuando ya esté disponible con carácter general, algunos aspectos podrían requerir una suscripción Premium de Azure AD. Esta característica permite configurar directivas de asignación de notificaciones para los protocolos WS-Fed, SAML, OAuth y OpenID Connect.

Los administradores de inquilinos usan esta característica para personalizar las notificaciones que se emiten en tokens para una aplicación específica de su inquilino. Puede usar directivas de asignación de notificaciones para:

- Seleccionar las notificaciones que se incluyen en tokens.
- Crear tipos de notificación que aún no existen.
- Elegir o cambiar el origen de los datos emitidos en notificaciones concretas.

En este artículo se abordan algunos escenarios comunes que pueden ayudarle a entender cómo usar el [tipo de directiva de asignación de notificaciones](reference-claims-mapping-policy-type.md).

Al crear una directiva de asignación de notificaciones, también puede emitir una notificación de un atributo de extensión de esquema de directorio en los tokens. Use *ExtensionID* para el atributo de extensión en lugar de *ID* en el elemento `ClaimsSchema`.  Para más información sobre los atributos de extensión, consulte [Uso de atributos de extensión de esquema de directorio](active-directory-schema-extensions.md).

## <a name="prerequisites"></a>Prerrequisitos

En los ejemplos siguientes, va a crear, actualizar, vincular y eliminar directivas de entidades de servicio. Las directivas de asignación de notificaciones solo se pueden asignar a objetos de entidades de servicio. Si no está familiarizado con Azure AD, es conveniente que [aprenda a obtener un inquilino de Azure AD](quickstart-create-new-tenant.md) antes de continuar con estos ejemplos.

Para comenzar, realice uno de los pasos siguientes:

1. Descargue la [versión preliminar pública más reciente del módulo de PowerShell de Azure AD](https://www.powershellgallery.com/packages/AzureADPreview).
1. Ejecute el comando Connect para iniciar sesión en la cuenta de administrador de Azure AD. Ejecute este comando cada vez que inicie una nueva sesión.

   ``` powershell
   Connect-AzureAD -Confirm
   ```
1. Ejecute el siguiente comando para ver todas las directivas que se han creado en la organización. Se recomienda ejecutar este comando después de la mayoría de las operaciones en los escenarios siguientes, para comprobar que las directivas se van a crear según lo previsto.

   ``` powershell
   Get-AzureADPolicy
   ```

## <a name="omit-the-basic-claims-from-tokens"></a>Omisión de las notificaciones básicas de los tokens

En este ejemplo se crea una directiva que quita el [conjunto de notificaciones básicas](reference-claims-mapping-policy-type.md#claim-sets) de los tokens emitidos para entidades de servicio vinculadas.

1. Cree una directiva de asignación de notificaciones. Esta directiva, que se vincula a entidades de servicio concretas, quita el conjunto de notificaciones básicas de los tokens.
   1. Ejecute este comando para crear la directiva:

      ``` powershell
      New-AzureADPolicy -Definition @('{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"false"}}') -DisplayName "OmitBasicClaims" -Type "ClaimsMappingPolicy"
      ```
   2. Para ver la nueva directiva y obtener su ObjectID, ejecute el siguiente comando:

      ``` powershell
      Get-AzureADPolicy
      ```
1. Asigne la directiva a su entidad de servicio. También necesita obtener el valor de ObjectId de su entidad de servicio.
   1. Para ver todas las entidades de servicio de su organización, puede [consultar la Microsoft Graph API](/graph/traverse-the-graph). O bien, inicie sesión en su cuenta de Azure AD en el [Explorador de Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer).
   2. Cuando tenga el valor de ObjectId de la entidad de servicio, ejecute el siguiente comando:

      ``` powershell
      Add-AzureADServicePrincipalPolicy -Id <ObjectId of the ServicePrincipal> -RefObjectId <ObjectId of the Policy>
      ```

## <a name="include-the-employeeid-and-tenantcountry-as-claims-in-tokens"></a>Inclusión de EmployeeID y TenantCountry como notificaciones en los tokens

En este ejemplo se crea una directiva que agrega EmployeeID y TenantCountry a los tokens emitidos para entidades de servicio vinculadas. EmployeeID se emite como tipo de notificación de nombre en los tokens SAML y JWT. TenantCountry se emite como tipo de notificación de país o región en los tokens SAML y JWT. En este ejemplo se sigue incluyendo el conjunto de notificaciones básicas en los tokens.

1. Cree una directiva de asignación de notificaciones. Esta directiva, que se vincula a entidades de servicio concretas, agrega las notificaciones EmployeeID y TenantCountry a los tokens.
   1. Ejecute el siguiente comando para crear la directiva:

      ``` powershell
      New-AzureADPolicy -Definition @('{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"true", "ClaimsSchema": [{"Source":"user","ID":"employeeid","SamlClaimType":"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/employeeid","JwtClaimType":"name"},{"Source":"company","ID":"tenantcountry","SamlClaimType":"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/country","JwtClaimType":"country"}]}}') -DisplayName "ExtraClaimsExample" -Type "ClaimsMappingPolicy"
      ```

   2. Para ver la nueva directiva y obtener su ObjectID, ejecute el siguiente comando:

      ``` powershell
      Get-AzureADPolicy
      ```
1. Asigne la directiva a su entidad de servicio. También necesita obtener el valor de ObjectId de su entidad de servicio.
   1. Para ver todas las entidades de servicio de su organización, puede [consultar la Microsoft Graph API](/graph/traverse-the-graph). O bien, inicie sesión en su cuenta de Azure AD en el [Explorador de Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer).
   2. Cuando tenga el valor de ObjectId de la entidad de servicio, ejecute el siguiente comando:

      ``` powershell
      Add-AzureADServicePrincipalPolicy -Id <ObjectId of the ServicePrincipal> -RefObjectId <ObjectId of the Policy>
      ```

## <a name="use-a-claims-transformation-in-tokens"></a>Uso de una transformación de notificaciones en los tokens

En este ejemplo se crea una directiva que emite una notificación "JoinedData" personalizada para los tokens JWT emitidos para entidades de servicio vinculadas. Esta notificación contiene un valor creado mediante la combinación de los datos almacenados en el atributo extensionattribute1 en el objeto de usuario que incluye ".sandbox". En este ejemplo se excluye el conjunto de notificaciones básicas en los tokens.

1. Cree una directiva de asignación de notificaciones. Esta directiva, que se vincula a entidades de servicio concretas, agrega las notificaciones EmployeeID y TenantCountry a los tokens.
   1. Ejecute el siguiente comando para crear la directiva:

      ``` powershell
      New-AzureADPolicy -Definition @('{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"true", "ClaimsSchema":[{"Source":"user","ID":"extensionattribute1"},{"Source":"transformation","ID":"DataJoin","TransformationId":"JoinTheData","JwtClaimType":"JoinedData"}],"ClaimsTransformations":[{"ID":"JoinTheData","TransformationMethod":"Join","InputClaims":[{"ClaimTypeReferenceId":"extensionattribute1","TransformationClaimType":"string1"}], "InputParameters": [{"ID":"string2","Value":"sandbox"},{"ID":"separator","Value":"."}],"OutputClaims":[{"ClaimTypeReferenceId":"DataJoin","TransformationClaimType":"outputClaim"}]}]}}') -DisplayName "TransformClaimsExample" -Type "ClaimsMappingPolicy"
      ```

   2. Para ver la nueva directiva y obtener su ObjectID, ejecute el siguiente comando:

      ``` powershell
      Get-AzureADPolicy
      ```
1. Asigne la directiva a su entidad de servicio. También necesita obtener el valor de ObjectId de su entidad de servicio.
   1. Para ver todas las entidades de servicio de su organización, puede [consultar la Microsoft Graph API](/graph/traverse-the-graph). O bien, inicie sesión en su cuenta de Azure AD en el [Explorador de Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer).
   2. Cuando tenga el valor de ObjectId de la entidad de servicio, ejecute el siguiente comando:

      ``` powershell
      Add-AzureADServicePrincipalPolicy -Id <ObjectId of the ServicePrincipal> -RefObjectId <ObjectId of the Policy>
      ```

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Las aplicaciones que reciben tokens se basan en el hecho de que los valores de notificación se emiten de forma autoritaria por parte de Azure AD y no se pueden alterar. Sin embargo, al modificar el contenido del token a través de las directivas de asignación de notificaciones, es posible que estas suposiciones ya no sean correctas. Las aplicaciones deben reconocer explícitamente que el creador de la directiva de asignación de notificaciones ha modificado los tokens para protegerse de las directivas de asignación de notificaciones creadas por actores malintencionados. Esto puede hacerse de las siguientes maneras:

- Configurar una clave de firma personalizada
- Actualizar el manifiesto de aplicación para aceptar notificaciones asignadas.
 
Sin esto, Azure AD devolverá un [código de error `AADSTS50146`](reference-aadsts-error-codes.md#aadsts-error-codes).

### <a name="custom-signing-key"></a>Clave de firma de personalizada

Para agregar una clave de firma personalizada al objeto de la entidad de servicio, puede usar el cmdlet [`New-AzureADApplicationKeyCredential`](/powerShell/module/Azuread/New-AzureADApplicationKeyCredential) de Azure PowerShell para crear una credencial de clave de certificado para el objeto de aplicación.

Las aplicaciones que tienen habilitada la asignación de notificaciones deben validar sus claves de firma de tokens mediante la anexión de `appid={client_id}` a las [solicitudes de metadatos de OpenID Connect](v2-protocols-oidc.md#fetch-the-openid-connect-metadata-document). A continuación se muestra el formato del documento de metadatos de OpenID Connect que se debe usar:

```
https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration?appid={client-id}
```

### <a name="update-the-application-manifest"></a>Actualización del manifiesto de aplicación

Como alternativa, puede establecer la propiedad `acceptMappedClaims` en `true` en el [manifiesto de aplicación](reference-app-manifest.md). Como se documenta en el [tipo de recurso apiApplication](/graph/api/resources/apiapplication#properties), esto permite que una aplicación use la asignación de notificaciones sin especificar una clave de firma personalizada.

Esto requiere que la audiencia de tokens solicitada use un nombre de dominio comprobado del inquilino de Azure AD, lo que significa que debe asegurarse de establecer `Application ID URI` (representado por `identifierUris` en el manifiesto de aplicación), por ejemplo, en `https://contoso.com/my-api` o (simplemente mediante el nombre de inquilino predeterminado) `https://contoso.onmicrosoft.com/my-api`.

Si no usa un dominio comprobado, Azure AD devolverá un código de error `AADSTS501461` con *el mensaje "AcceptMappedClaims solo se admite para una audiencia de tokens que coincida con el GUID de la aplicación o con una audiencia dentro de los dominios comprobados del inquilino. Cambie el identificador de recursos o use una clave de firma específica de la aplicación".*

## <a name="next-steps"></a>Pasos siguientes

- Consulte el artículo de referencia sobre el [tipo de directiva de asignación de notificaciones](reference-claims-mapping-policy-type.md) para más información.
- Para obtener información sobre cómo personalizar las notificaciones emitidas en el token SAML a través de Azure Portal, vea [Procedimientos para: Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales](active-directory-saml-claims-customization.md).
- Para más información sobre los atributos de extensión, consulte [Uso de atributos de extensión de esquema de directorio en las notificaciones](active-directory-schema-extensions.md).
