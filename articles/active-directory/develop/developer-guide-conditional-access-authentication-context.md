---
title: Instrucciones para desarrolladores sobre el contexto de autenticación del acceso condicional de Azure AD
description: Instrucciones para desarrolladores y escenarios del contexto de autenticación del acceso condicional de Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.date: 05/18/2021
ms.author: joflore
author: MicrosoftGuyJFlo
manager: CelesteDG
ms.reviewer: kkrishna
ms.workload: identity
ms.custom: aaddev
ms.openlocfilehash: c632b19daf52fd2af4d2c2920c3a61519da6c85c
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111408072"
---
# <a name="developers-guide-to-conditional-access-authentication-context"></a>Instrucciones para desarrolladores sobre el contexto de autenticación del acceso condicional

El [acceso condicional](../conditional-access/overview.md) es el plano de control de Confianza cero que permite establecer directivas de destino para el acceso a todas las aplicaciones: antiguas o nuevas, privadas o públicas, locales o en varias nubes. Con el [contexto de autenticación del acceso condicional](../conditional-access/concept-conditional-access-cloud-apps.md#authentication-context-preview), puede aplicar diferentes directivas dentro de esas aplicaciones. 

El contexto de autenticación del acceso condicional permite aplicar directivas específicas a datos y acciones confidenciales en lugar de solo en el nivel de la aplicación. Puede ajustar las directivas de Confianza cero para el acceso con privilegios mínimos al tiempo que minimiza las desavenencias entre los usuarios y hace que estos sean más productivos y que sus recursos estén más protegidos. En la actualidad, las aplicaciones que usan [OpenId Connect](https://openid.net/specs/openid-connect-core-1_0.html) pueden usarlo para la autenticación desarrollada por su empresa con el fin de proteger recursos confidenciales, como transacciones de alto valor o la visualización de datos personales de empleados.

Utilice la nueva característica de contexto de autenticación del motor del acceso condicional de Azure AD para desencadenar una demanda de autenticación por pasos desde la aplicación y los servicios. Los desarrolladores ahora tienen la capacidad de exigir una autenticación más sólida y mejorada, de forma selectiva, como MFA, a los usuarios finales desde sus aplicaciones. Esta característica ayuda a los desarrolladores a crear experiencias de usuario más fluidas para la mayoría de los componentes de su aplicación, mientras que el acceso a operaciones y datos más protegidos permanece detrás de controles de autenticación más sólidos.

## <a name="problem-statement"></a>Declaración del problema

A menudo, los administradores y reguladores de TI intentan buscar el equilibrio entre solicitar a sus usuarios factores adicionales de autenticación con demasiada frecuencia, y lograr una seguridad y un cumplimiento de directivas adecuados para aplicaciones y servicios con componentes que contienen datos y operaciones confidenciales. Puede que haya que decidir entre una directiva fuerte que afecte a la productividad de los usuarios cuando acceden a la mayoría de los datos y acciones, o una directiva que no sea lo suficientemente segura para los recursos confidenciales. 

Por lo tanto, ¿qué ocurriría si las aplicaciones pudieran combinar ambas cosas? Un escenario en el que podrían funcionar con una seguridad relativamente menor y solicitudes menos frecuentes para la mayoría de los usuarios y operaciones y, sin embargo, aumentar condicionalmente el requisito de seguridad cuando los usuarios accedieran a componentes más confidenciales. 

## <a name="common-scenarios"></a>Escenarios frecuentes

Por ejemplo, aunque los usuarios pueden iniciar sesión en SharePoint mediante la autenticación multifactor, el acceso a la colección de sitios de SharePoint que contiene documentos confidenciales puede requerir un dispositivo compatible y solo ser accesible desde intervalos IP de confianza. 

## <a name="steps"></a>Pasos

Estos son los requisitos previos y los pasos si desea usar el contexto de autenticación del acceso condicional. 

### <a name="prerequisites"></a>Prerrequisitos

**En primer lugar**, la aplicación debe integrarse con Microsoft Identity Platform mediante el uso de los protocolos [OpenID Connect](v2-protocols-oidc.md)/ [OAuth 2.0](v2-oauth2-auth-code-flow.md) para la autenticación y autorización. Se recomienda usar las [bibliotecas de autenticación de la plataforma de identidad de Microsoft](reference-v2-libraries.md) para integrar y proteger la aplicación con Azure Active Directory. La [documentación de la plataforma de identidad de Microsoft](index.yml) es un buen lugar para aprender a integrar las aplicaciones con dicha plataforma. La compatibilidad con las características de contexto de autenticación del acceso condicional se basa en las extensiones de protocolo proporcionadas por el protocolo estándar [OpenID Connect](v2-protocols-oidc.md) del sector. Los desarrolladores usan un **valor** de [referencia para el contexto de autenticación del acceso condicional](/graph/api/resources/authenticationcontextclassreference) con el parámetro [Solicitud de notificaciones](claims-challenge.md) para proporcionar a las aplicaciones una manera de desencadenar y cumplir con la directiva.

**En segundo lugar**, el [acceso condicional](../conditional-access/overview.md) requiere licencias de Azure AD Premium P1. Para más información sobre las licencias, consulte la [página de precios de Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

**En tercer lugar**, actualmente solo está disponible para las aplicaciones en las que inician sesión los usuarios. No se admiten las aplicaciones que se autentican por sí mismas. Use la [guía Flujos de autenticación y escenarios de aplicaciones](authentication-flows-app-scenarios.md) para más información sobre los tipos y flujos de aplicaciones de autenticación admitidos en la plataforma de identidad de Microsoft.

### <a name="integration-steps"></a>Pasos de integración

Una vez que la aplicación se integra mediante los protocolos de autenticación admitidos y se registra en un inquilino de Azure AD que tiene la característica de acceso condicional disponible para su uso, puede iniciar el proceso de integración de esta característica en las aplicaciones en las que inician sesión los usuarios.

> [!NOTE]
> También hay disponible un tutorial detallado de esta característica en una sesión grabada de [Uso del contexto de autenticación del acceso condicional en la aplicación para la autenticación por pasos](https://www.youtube.com/watch?v=_iO7CfoktTY).

**Primero**, declare y haga que los contextos de autenticación estén disponibles en el inquilino. Para más información, consulte [Configurar contextos de autenticación](../conditional-access/concept-conditional-access-cloud-apps.md#configure-authentication-contexts).

Los valores **C1-C25** están disponibles para su uso como **identificadores de los contextos de autenticación** de un inquilino. Estos son algunos ejemplos de contexto de autenticación:

- **C1**: requiere autenticación segura
- **C2**: requiere dispositivos compatibles
- **C3**: requiere ubicaciones de confianza

Cree o modifique las directivas de acceso condicional para usar los contextos de autenticación del acceso condicional. Estos son algunos ejemplos de directivas:

- Todos los usuarios que inician sesión en esta aplicación web deben haber completado correctamente la autenticación en dos fases para el identificador de contexto de autenticación **C1**.
- Todos los usuarios que inician sesión en esta aplicación web deben haber completado correctamente la autenticación en dos fases y también tener acceso a la aplicación web desde un determinado intervalo de direcciones IP para el identificador de contexto de autenticación **C3**.

> [!NOTE]
> Los valores del contexto de autenticación del acceso condicional se declaran y mantienen de manera independiente de las aplicaciones. No es aconsejable que las aplicaciones tengan una dependencia fuerte de los identificadores del contexto de autenticación. Los administradores de TI suelen crear las directivas de acceso condicional, ya que tienen un mayor conocimiento de los recursos disponibles para aplicar directivas. Por ejemplo, para un inquilino de Azure AD, los administradores de TI conocen cuántos usuarios del inquilino están equipados para usar la autenticación en dos fases como autenticación multifactor y, por tanto, pueden garantizar que las directivas de acceso condicional que requieren la autenticación en dos fases se limitan a estos usuarios equipados. De forma similar, si la aplicación se usa en varios inquilinos, los identificadores de contextos de autenticación en uso podrían ser diferentes y, en algunos casos, no estar disponibles en absoluto.

**Segundo**: se recomienda a los desarrolladores de una aplicación que planeen usar el contexto de autenticación del acceso condicional que proporcionen primero a los administradores de aplicaciones o de TI un medio para asignar posibles acciones confidenciales a identificadores de contextos de autenticación. Estos son, a grandes rasgos, los pasos:

1. Acciones de identidad del código que están disponibles para su asignación a identificadores de contextos de autenticación.
1. Cree una pantalla en el portal de administración de la aplicación (o una funcionalidad equivalente) que los administradores de TI puedan usar para asignar acciones confidenciales a un identificador de contexto de autenticación disponible.
1. Consulte el ejemplo de código [Uso del contexto de autenticación del acceso condicional para realizar la autenticación por pasos](https://github.com/Azure-Samples/ms-identity-ca-auth-context/blob/main/README.md) para obtener un ejemplo sobre cómo se realiza.

Estos pasos son los cambios que debe llevar a cabo en el código base. A grandes rasgos, los pasos son:

- Consulte MS Graph para que [muestre todos los contextos de autenticación disponibles](/graph/api/conditionalaccessroot-list-authenticationcontextclassreferences).
- Permita a los administradores de TI seleccionar operaciones confidenciales o con privilegios elevados, y asignarlas a los contextos de autenticación disponibles mediante directivas de CA. 
- Guarde esta información de asignación en la base de datos o almacén local.

:::image type="content" source="media/developer-guide-conditional-access-authentication-context/configure-conditional-access-authentication-context.png" alt-text="Flujo de configuración para crear un contexto de autenticación":::

**Tercero**: la aplicación que, en este ejemplo, se supone que es una API web, posteriormente, necesita evaluar las llamadas a la asignación guardada y, en consecuencia, plantear desafíos de notificación para sus aplicaciones cliente. Para prepararse para esta acción, se deben realizar los pasos siguientes:

1. En una operación confidencial y protegida mediante el contexto de autenticación, evalúe los valores de la notificación **acrs** en comparación con las asignaciones de identificador del contexto de autenticación guardadas anteriormente y genere un [desafío de notificaciones](claims-challenge.md) como se ofrece en el fragmento de código siguiente. 

1. En el diagrama siguiente se muestra la interacción entre el usuario, la aplicación cliente y la API web.

   :::image type="content" source="media/developer-guide-conditional-access-authentication-context/authentication-context-application-flow.png" alt-text="Diagrama que muestra la interacción del usuario, la aplicación web, la API y Azure AD":::

   El fragmento de código siguiente pertenece al ejemplo de código [Uso del contexto de autenticación del acceso condicional para realizar la autenticación por pasos](https://github.com/Azure-Samples/ms-identity-ca-auth-context/blob/main/README.md). El primer método, `CheckForRequiredAuthContext()` en la API 

      - Comprueba si la acción de la aplicación a la que se llama requiere autenticación por pasos. Para ello, compruebe en su base de datos una asignación guardada para este método.
      - Si esta acción requiere realmente un contexto de autenticación con privilegios elevados, se busca la notificación **acrs** de un identificador de contexto de autenticación existente que coincida.
      - Si no se encuentra un identificador de contexto de autenticación que coincida, se genera un [desafío de notificaciones](claims-challenge.md#claims-challenge-header-format).

      ```
      public void CheckForRequiredAuthContext(string method)
      {
          string authType = _commonDBContext.AuthContext.FirstOrDefault(x => x.Operation == method 
                      && x.TenantId == _configuration["AzureAD:TenantId"])?.AuthContextId;

          if (!string.IsNullOrEmpty(authType))
          {
              HttpContext context = this.HttpContext;
              string authenticationContextClassReferencesClaim = "acrs";

              if (context == null || context.User == null || context.User.Claims == null 
                  || !context.User.Claims.Any())
              {
                  throw new ArgumentNullException("No Usercontext is available to pick claims from");
              }

              Claim acrsClaim = context.User.FindAll(authenticationContextClassReferencesClaim).FirstOrDefault(x 
                  => x.Value == authType);

              if (acrsClaim == null || acrsClaim.Value != authType)
              {
                  if (IsClientCapableofClaimsChallenge(context))
                  {
                      string clientId = _configuration.GetSection("AzureAd").GetSection("ClientId").Value;
                      var base64str = Convert.ToBase64String(Encoding.UTF8.GetBytes("{\"access_token\":{\"acrs\":{\"essential\":true,\"value\":\"" + authType + "\"}}}"));

                      context.Response.Headers.Append("WWW-Authenticate", $"Bearer realm=\"\", authorization_uri=\"https://login.microsoftonline.com/common/oauth2/authorize\", client_id=\"" + clientId + "\", error=\"insufficient_claims\", claims=\"" + base64str + "\", cc_type=\"authcontext\"");
                      context.Response.StatusCode = (int)HttpStatusCode.Unauthorized;
                      string message = string.Format(CultureInfo.InvariantCulture, "The presented access tokens had insufficient claims. Please request for claims requested in the WWW-Authentication header and try again.");
                      context.Response.WriteAsync(message);
                      context.Response.CompleteAsync();
                      throw new UnauthorizedAccessException(message);
                  }
                  else
                  {
                      throw new UnauthorizedAccessException("The caller does not meet the authentication  bar to carry our this operation. The service cannot allow this operation");
                  }
              }
          }
      }
      ```

   > [!NOTE]
   > El formato del desafío de notificaciones se describe en el artículo [Desafío de notificaciones en la plataforma de identidad de Microsoft](claims-challenge.md). 

1. En la aplicación cliente, intercepte el desafío de notificaciones y redirija al usuario a Azure AD para una evaluación de directiva adicional. El fragmento de código siguiente pertenece al ejemplo de código [Uso del contexto de autenticación del acceso condicional para realizar la autenticación por pasos](https://github.com/Azure-Samples/ms-identity-ca-auth-context/blob/main/README.md).

   ```
   internal static string ExtractHeaderValues(WebApiMsalUiRequiredException response)
   {
       if (response.StatusCode == System.Net.HttpStatusCode.Unauthorized && response.Headers.WwwAuthenticate.Any())
       {
           AuthenticationHeaderValue bearer = response.Headers.WwwAuthenticate.First(v => v.Scheme == "Bearer");
           IEnumerable<string> parameters = bearer.Parameter.Split(',').Select(v => v.Trim()).ToList();
           var errorValue = GetParameterValue(parameters, "error");

           try
           {
               // read the header and checks if it conatins error with insufficient_claims value.
               if (null != errorValue && "insufficient_claims" == errorValue)
               {
                   var claimChallengeParameter = GetParameterValue(parameters, "claims");
                   if (null != claimChallengeParameter)
                   {
                       var claimChallenge = ConvertBase64String(claimChallengeParameter);

                       return claimChallenge;
                   }
               }
           }
           catch (Exception ex)
           {
               throw ex;
           }
       }
       return null;
   }
   ```

   Controle la excepción en la llamada a la API web; si se presenta un desafío de notificaciones, redirija al usuario a Azure AD para un procesamiento adicional.

   ```
   try
   {
       // Call the API 
       await _todoListService.AddAsync(todo);
   }
   catch (WebApiMsalUiRequiredException hex)
   {
       // Challenges the user if exception is thrown from Web API.
       try
       {
           var claimChallenge =ExtractHeaderValues(hex);
           _consentHandler.ChallengeUser(new string[] { "user.read" }, claimChallenge);

           return new EmptyResult();
       }
       catch (Exception ex)
       {
           _consentHandler.HandleException(ex);
       }

       Console.WriteLine(hex.Message);
   }
   return RedirectToAction("Index");
   ```
   
1. (Opcional) Declare la funcionalidad del cliente. Las funcionalidades de cliente ayudan a los proveedores de recursos, como nuestra API web anterior, a detectar si la aplicación cliente que realiza la llamada entiende el desafío de notificaciones y puede, a continuación, personalizar su respuesta en consecuencia. Esta funcionalidad podría ser útil en aquellos casos en los que no todos los clientes API son capaces de controlar los desafíos de notificaciones y algunos anteriores todavía esperan una respuesta diferente. Para más información, consulte la sección [Funcionalidades de cliente](claims-challenge.md#client-capabilities).

## <a name="caveats-and-recommendations"></a>Advertencias y recomendaciones

No codifique de forma rígida los valores del contexto de autenticación de la aplicación. Las aplicaciones deben leer y aplicar el contexto de autenticación [mediante llamadas de MS Graph](/graph/api/resources/authenticationcontextclassreference). Este procedimiento es fundamental para las [aplicaciones multiinquilino](howto-convert-app-to-be-multi-tenant.md). Los valores de contexto de autenticación variarán entre los inquilinos de Azure AD y no estarán disponibles en la edición gratuita de Azure AD. Para más información sobre cómo una aplicación debe consultar, configurar y usar un contexto de autenticación en su código, consulte el ejemplo de código de [Uso del contexto de autenticación del acceso condicional para realizar la autenticación por pasos](https://github.com/Azure-Samples/ms-identity-ca-auth-context/blob/main/README.md). 

No use el contexto de autenticación en aquellos casos en los que la propia aplicación va a ser un destino de las directivas de acceso condicional. La característica funciona mejor cuando los componentes de la aplicación requieren que el usuario cumpla un nivel mayor de autenticación.

## <a name="next-steps"></a>Pasos siguientes

- [Acceso condicional específico para acciones y datos confidenciales (blog)](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/granular-conditional-access-for-sensitive-data-and-actions/ba-p/1751775)
- [Confianza cero con la plataforma de identidad de Microsoft](/security/zero-trust/identity-developer)
- [Creación de aplicaciones listas para la Confianza cero con la Plataforma de identidad de Microsoft](/security/zero-trust/identity-developer)
- [Uso del contexto de autenticación del acceso condicional para realizar la autenticación por pasos para operaciones con privilegios elevados en una API web](https://github.com/Azure-Samples/ms-identity-ca-auth-context/blob/main/README.md)
- [Contexto de autenticación del acceso condicional](../conditional-access/concept-conditional-access-cloud-apps.md#authentication-context-preview)
- [Tipo de recurso authenticationContextClassReference: MS Graph](/graph/api/conditionalaccessroot-list-authenticationcontextclassreferences)
- [Desafío de notificaciones, solicitud de notificaciones y funcionalidades de cliente en la plataforma de identidad de Microsoft](claims-challenge.md)
- [Uso del contexto de autenticación con Microsoft Information Protection y SharePoint](/microsoft-365/compliance/sensitivity-labels-teams-groups-sites#more-information-about-the-dependencies-for-the-authentication-context-option)
- [Flujos de autenticación y escenarios de aplicaciones](authentication-flows-app-scenarios.md)
- [Uso de las API habilitadas para la evaluación continua de acceso en las aplicaciones](app-resilience-continuous-access-evaluation.md)
