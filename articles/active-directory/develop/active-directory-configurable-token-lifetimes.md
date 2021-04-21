---
title: Vigencia de tokens configurable
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo establecer la vigencia de los tokens de acceso, SAML y de identificador emitidos por la Plataforma de identidad de Microsoft.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 04/08/2021
ms.author: ryanwi
ms.custom: aaddev, identityplatformtop40, content-perf, FY21Q1, contperf-fy21q1
ms.reviewer: hirsin, jlu, annaba
ms.openlocfilehash: e1753391c7b61b6e9bd9e6ac0d142b4ee94502d8
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107363978"
---
# <a name="configurable-token-lifetimes-in-the-microsoft-identity-platform-preview"></a>Vigencia de tokens configurable en la Plataforma de identidad de Microsoft (versión preliminar)

Puede especificar la vigencia de un token de acceso, identificador o SAML emitido por la Plataforma de identidad de Microsoft. La vigencia de los tokens se puede configurar para todas las aplicaciones de una organización, para una aplicación multiinquilino (multiorganización) o para una entidad de servicio específica de una organización. No obstante, actualmente no se admite la configuración de la vigencia de los tokens para las [entidades de servicio de identidad administrada](../managed-identities-azure-resources/overview.md).

En Azure AD, un objeto de directiva representa un conjunto de reglas que se exigen en algunas o todas las aplicaciones de una organización. Cada tipo de directiva tiene una estructura única con un conjunto de propiedades que luego se aplican a los objetos a los que están asignadas.

Puede designar una directiva como la directiva predeterminada para su organización. La directiva se aplicará a cualquier aplicación que resida dentro de esa organización, siempre y cuando no se haya reemplazado por una directiva con una prioridad más alta. También puede asignar una directiva a aplicaciones específicas. El orden de prioridad varía según el tipo de directiva.

Para obtener ejemplos, lea [Configuración de la vigencia de los tokens](configure-token-lifetimes.md).

> [!NOTE]
> La directiva de vigencia del token configurable solo se aplica a los clientes móviles y de escritorio que tienen acceso a los recursos SharePoint Online y OneDrive para la Empresa, y no se aplica a las sesiones del explorador web.
> Para administrar la vigencia de las sesiones del explorador web para SharePoint Online y OneDrive para la Empresa, use la característica de [vigencia de sesiones de acceso condicional](../conditional-access/howto-conditional-access-session-lifetime.md). Consulte el [blog de SharePoint Online](https://techcommunity.microsoft.com/t5/SharePoint-Blog/Introducing-Idle-Session-Timeout-in-SharePoint-and-OneDrive/ba-p/119208) para más información sobre cómo configurar los tiempos de expiración de sesiones inactivas.

## <a name="license-requirements"></a>Requisitos de licencia

Necesita una licencia de Azure AD Premium P1 para usar esta característica. Para obtener la licencia correcta para sus requisitos, consulte [Comparación de las características con disponibilidad general de las ediciones Gratis y Prémium](https://azure.microsoft.com/pricing/details/active-directory/).

Los clientes con [licencias de Microsoft 365 Empresa](/office365/servicedescriptions/microsoft-365-service-descriptions/microsoft-365-business-service-description) también tienen acceso a características de acceso condicional.

## <a name="token-lifetime-policies-for-access-saml-and-id-tokens"></a>Directivas de vigencia para los tokens de acceso, SAML e identificador

Las directivas de vigencia de tokens se pueden establecer para los tokens de acceso, SAML y de identificador.

### <a name="access-tokens"></a>Tokens de acceso

Los clientes utilizan tokens de acceso para acceder a un recurso protegido. Solo se puede utilizar un token de acceso para una combinación específica de usuario, cliente y recurso. Los tokens de acceso no se pueden revocar y son válidos hasta que caducan. Un individuo maltintencionado que haya obtenido un token de acceso puede usarlo durante toda su vigencia. El ajuste de la vigencia de un token de acceso es un balance entre la mejora del rendimiento del sistema y el aumento de la cantidad de tiempo que el cliente conserva el acceso después de que la cuenta de usuario está deshabilitada. Se consigue un rendimiento mejorado del sistema al reducir el número de veces que un cliente necesita adquirir un token de acceso nuevo.  El valor predeterminado varía en función de la aplicación cliente que solicite el token. Por ejemplo, los clientes compatibles con la evaluación de acceso continuo (CAE) que negocian sesiones compatibles con CAE, tendrán una vigencia de token de larga duración (hasta 28 horas). Una vez expire el token, el cliente debe usar el token de actualización para adquirir un nuevo token de actualización (normalmente en modo silencioso) y un token de acceso.

### <a name="saml-tokens"></a>Tokens de SAML

Muchas aplicaciones SaaS basadas en web usan tokens SAML, que se obtienen mediante el punto de conexión del protocolo SAML2 de Azure Active Directory. También las usan las aplicaciones mediante WS-Federation. La duración predeterminada del token es de 1 hora. Desde la perspectiva de una aplicación, el período de validez del token se especifica mediante el valor NotOnOrAfter del elemento `<conditions …>` en el token. Una vez finalizado el período de validez del token, el cliente debe iniciar una nueva solicitud de autenticación, que a menudo se satisfará sin iniciar sesión de forma interactiva como resultado del token de sesión de inicio de sesión único (SSO).

El valor de NotOnOrAfter se puede cambiar mediante el parámetro `AccessTokenLifetime` en un `TokenLifetimePolicy`. Se establecerá en la duración configurada en la directiva, si la hay, más un factor de sesgo de reloj de cinco minutos.

Tenga en cuenta que la confirmación de asunto NotOnOrAfter especificada en el elemento `<SubjectConfirmationData>` no se ve afectada por la configuración de la vigencia del token. 

### <a name="id-tokens"></a>Tokens de identificador

Los tokens de identificador se pasan a los clientes nativos y de sitios web. Los tokens de identificador contienen información de perfil de un usuario. Un token de identificador se enlaza a una combinación específica de usuario y cliente. Los tokens de identificador se consideran válidos hasta que expiran. Normalmente, una aplicación web relaciona la vigencia de la sesión de un usuario en la aplicación con la vigencia del token de identificador emitido para el usuario. Puede ajustar la vigencia de un token de identificador para controlar la frecuencia con la que la aplicación web expira la sesión de la aplicación y requiere que el usuario se vuelva a autenticar en la Plataforma de identidad de Microsoft (de forma silenciosa o interactiva).

## <a name="token-lifetime-policies-for-refresh-tokens-and-session-tokens"></a>Directivas de vigencia para los tokens de actualización y los tokens de sesión

Las directivas de vigencia del token no se pueden establecer para los tokens de actualización y los tokens de sesión.

> [!IMPORTANT]
> A partir del 30 de enero de 2021, no puede configurar la vigencia de los tokens de actualización ni de sesión. Azure Active Directory ya no respeta la configuración de los tokens de actualización y de sesión en las directivas existentes.  Los nuevos tokens emitidos después de que los tokens existentes hayan expirado ahora se establecen en la [configuración predeterminada](#configurable-token-lifetime-properties). Después de la retirada de la configuración de los tokens de actualización y de sesión, todavía podrá configurar la vigencia de los tokens de acceso, SAML y de identificador.
>
> La duración del token existente no se cambiará. Una vez expirado, se emitirá un nuevo token basado en el valor predeterminado.
>
> Si necesita seguir definiendo el período de tiempo antes de que se pida al usuario que vuelva a iniciar sesión, configure la frecuencia de inicio de sesión en el acceso condicional. Para obtener más información sobre el acceso condicional, consulte [Configuración de la administración de las sesiones de autenticación con el acceso condicional](../conditional-access/howto-conditional-access-session-lifetime.md).

## <a name="configurable-token-lifetime-properties"></a>Propiedades de vigencia de tokens configurables
Una directiva de vigencia del token es un tipo de objeto de directiva que contiene reglas de vigencia del token. Esta directiva controla cuánto tiempo se consideran válidos los tokens de acceso, SAML y de identificador para este recurso. Las directivas de vigencia del token no se pueden establecer en tokens de actualización y de sesión. Si no se establece ninguna directiva, el sistema aplica el valor de vigencia predeterminado.

### <a name="access-id-and-saml2-token-lifetime-policy-properties"></a>Propiedades de la directiva de duración del token de SAML2, de id. y de acceso

Reducir la vigencia de los tokens de acceso disminuye el riesgo de que un individuo malintencionado use un token de acceso o de identificador durante un período de tiempo prolongado. (Estos tokens no se pueden revocar). El inconveniente es que afecta negativamente al rendimiento, ya que los tokens tendrán que reemplazarse con más frecuencia.

Para obtener un ejemplo, consulte [Creación de una directiva para inicio de sesión web](configure-token-lifetimes.md#create-a-policy-for-web-sign-in).

La configuración de los tokens de SAML2, de id. y de acceso se ve afectada por las siguientes propiedades y sus correspondientes valores establecidos:

- **Propiedad**: duración del token de acceso
- **Cadena de propiedad de directiva:** AccessTokenLifetime
- **Afecta a**: tokens de acceso, de id. y de SAML2
- **Predeterminado**:
    - Tokens de acceso: el valor predeterminado varía en función de la aplicación cliente que solicite el token. Por ejemplo, los clientes compatibles con la evaluación de acceso continuo (CAE) que negocian sesiones compatibles con CAE, tendrán una vigencia de token de larga duración (hasta 28 horas).
    - Tokens de id., tokens de SAML2: 1 hora
- **Mínimo:** 10 minutos
- **Máximo:** 1 día

### <a name="refresh-and-session-token-lifetime-policy-properties"></a>Propiedades de las directivas de vigencia de los tokens de actualización y de sesión

La configuración de los tokens de actualización y sesión se ve afectada por las siguientes propiedades y sus correspondientes valores establecidos. Después de la retirada de la configuración de los tokens de actualización y de sesión el 30 de enero de 2021, Azure AD solo aceptará los valores predeterminados que se describen a continuación. Si decide no usar el [acceso condicional](../conditional-access/howto-conditional-access-session-lifetime.md) para administrar la frecuencia de inicio de sesión, los tokens de actualización y de sesión se establecerán en la configuración predeterminada en esa fecha y ya no podrá cambiar su vigencia.  

|Propiedad   |Cadena de propiedad de directiva    |Afecta a |Valor predeterminado |
|----------|-----------|------------|------------|
|Tiempo máximo de inactividad del token de actualización |MaxInactiveTime  |Tokens de actualización |90 días  |
|Antigüedad máxima del token de actualización (un solo factor)  |MaxAgeSingleFactor  |Tokens de actualización (para los usuarios)  |Hasta que se revoca  |
|Antigüedad máxima del token de actualización (varios factores)  |MaxAgeMultiFactor  |Tokens de actualización (para los usuarios) |Hasta que se revoca  |
|Antigüedad máxima del token de sesión (un solo factor)  |MaxAgeSessionSingleFactor |Tokens de sesión (persistentes y no persistentes)  |Hasta que se revoca |
|Antigüedad máxima del token de sesión (varios factores)  |MaxAgeSessionMultiFactor  |Tokens de sesión (persistentes y no persistentes)  |Hasta que se revoca |

Puede usar PowerShell para buscar las directivas que se verán afectadas por la retirada.  Use los [cmdlets de PowerShell](configure-token-lifetimes.md#get-started) para ver todas las directivas creadas en su organización o para averiguar qué aplicaciones y entidades de servicio están vinculadas a una directiva específica.

## <a name="policy-evaluation-and-prioritization"></a>Evaluación y prioridades de directivas
Puede crear y, a continuación, asignar una directiva de vigencia del token a una aplicación específica, a su organización y a las entidades de servicio. Se pueden aplicar varias directivas a una aplicación específica. La directiva de vigencia del token que entra en vigor sigue estas reglas:

* Si una directiva se asigna explícitamente a la entidad de servicio, se aplicará.
* Si no hay ninguna directiva que se asigne explícitamente a la entidad de servicio, se aplicará una directiva asignada explícitamente a la organización primaria de la entidad de servicio.
* Si no hay ninguna directiva que se asigne explícitamente a la entidad de servicio o a la organización, se aplicará la directiva asignada a la aplicación.
* Si no se ha asignado ninguna directiva a la entidad de servicio, la organización o el objeto de aplicación, se aplican los valores predeterminados. (Consulte la tabla que aparece en [Propiedades de vigencia de tokens configurables](#configurable-token-lifetime-properties)).

Para más información sobre la relación entre objetos de aplicación y objetos de entidad de servicio, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory](app-objects-and-service-principals.md).

La validez de un token se evalúa en el momento en el que se usa. La directiva con la prioridad más alta en la aplicación a la que se accede es la que se aplica.

Todos los intervalos de tiempo usados aquí tienen formato según el objeto [TimeSpan](/dotnet/api/system.timespan) de C#: D.HH:MM:SS.  Por lo tanto, 80 días y 30 minutos sería `80.00:30:00`.  La D inicial puede eliminarse si es cero, por lo que 90 minutos sería `00:90:00`.  

## <a name="cmdlet-reference"></a>Referencia de cmdlets

Estos son los cmdlets del [módulo de versión preliminar de PowerShell para Graph de Azure Active Directory](/powershell/module/azuread/?view=azureadps-2.0-preview&preserve-view=true#service-principals).

### <a name="manage-policies"></a>Administración de directivas

Los cmdlets siguientes se pueden usar para administrar directivas.

| Cmdlet | Descripción | 
| --- | --- |
| [New-AzureADPolicy](/powershell/module/azuread/new-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) | Crea una nueva directiva. |
| [Get-AzureADPolicy](/powershell/module/azuread/get-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) | Obtiene todas las directivas de AzureAD o una directiva especificada. |
| [Get-AzureADPolicyAppliedObject](/powershell/module/azuread/get-azureadpolicyappliedobject?view=azureadps-2.0-preview&preserve-view=true) | Obtiene todas las aplicaciones y entidades de servicio vinculadas a una directiva. |
| [Set-AzureADPolicy](/powershell/module/azuread/set-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) | Actualiza una directiva existente. |
| [Remove-AzureADPolicy](/powershell/module/azuread/remove-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) | Elimina la directiva especificada. |

### <a name="application-policies"></a>Directivas de aplicación
Los cmdlets siguientes se pueden usar para directivas de aplicación.</br></br>

| Cmdlet | Descripción | 
| --- | --- |
| [Add-AzureADApplicationPolicy](/powershell/module/azuread/add-azureadapplicationpolicy?view=azureadps-2.0-preview&preserve-view=true) | Vincula la directiva especificada a una aplicación. |
| [Get-AzureADApplicationPolicy](/powershell/module/azuread/get-azureadapplicationpolicy?view=azureadps-2.0-preview&preserve-view=true) | Obtiene la directiva asignada a una aplicación. |
| [Remove-AzureADApplicationPolicy](/powershell/module/azuread/remove-azureadapplicationpolicy?view=azureadps-2.0-preview&preserve-view=true) | Quita una directiva de una aplicación. |

### <a name="service-principal-policies"></a>Directivas de la entidad de servicio
Los cmdlets siguientes se pueden usar para las directivas de entidad de servicio.

| Cmdlet | Descripción | 
| --- | --- |
| [Add-AzureADServicePrincipalPolicy](/powershell/module/azuread/add-azureadserviceprincipalpolicy?view=azureadps-2.0-preview&preserve-view=true) | Vincula la directiva especificada a una entidad de servicio. |
| [Get-AzureADServicePrincipalPolicy](/powershell/module/azuread/get-azureadserviceprincipalpolicy?view=azureadps-2.0-preview&preserve-view=true) | Obtiene cualquier directiva vinculada a la entidad de servicio especificada.|
| [Remove-AzureADServicePrincipalPolicy](/powershell/module/azuread/remove-azureadserviceprincipalpolicy?view=azureadps-2.0-preview&preserve-view=true) | Quita la directiva de la entidad de servicio especificada.|

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, lea los [ejemplos configuración de la vigencia de los tokens](configure-token-lifetimes.md).
