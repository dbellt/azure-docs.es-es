---
title: Uso de las API habilitadas para la evaluación continua de acceso en las aplicaciones | Azure
titleSuffix: Microsoft identity platform
description: Cómo aumentar la seguridad y la resistencia de las aplicaciones al agregar compatibilidad con la evaluación de acceso continuo para habilitar tokens de acceso de larga duración que se pueden revocar en función de eventos críticos y evaluación de directivas.
services: active-directory
author: knicholasa
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/06/2020
ms.author: nichola
ms.reviewer: ''
ms.openlocfilehash: 970985193245a4d7482979c2fc753c2c0b67834b
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111406740"
---
# <a name="how-to-use-continuous-access-evaluation-enabled-apis-in-your-applications"></a>Uso de las API habilitadas para la evaluación continua de acceso en las aplicaciones

La [evaluación continua de acceso](../conditional-access/concept-continuous-access-evaluation.md) (CAE) es una característica de Azure AD que permite revocar los tokens de acceso en función de [eventos críticos](../conditional-access/concept-continuous-access-evaluation.md#critical-event-evaluation) y la [evaluación de directivas](../conditional-access/concept-continuous-access-evaluation.md#conditional-access-policy-evaluation-preview) en lugar de depender de la expiración del token en función de la duración. En el caso de algunas API de recursos, ya que el riesgo y la directiva se evalúan en tiempo real, esto puede aumentar considerablemente la duración del token hasta 28 horas. Estos tokens de larga duración se actualizarán de manera proactiva mediante la biblioteca de autenticación de Microsoft (MSAL), lo que aumenta la resistencia de las aplicaciones.

En este artículo se muestra cómo usar las API habilitadas para CAE en sus aplicaciones. Las aplicaciones que no usan MSAL pueden agregar compatibilidad con [desafíos de notificaciones, solicitudes de notificaciones y funcionalidades de cliente](claims-challenge.md) para usar CAE.

## <a name="implementation-considerations"></a>Consideraciones de implementación

Para usar la evaluación continua de acceso, tanto la aplicación como la API de recursos a la que se tiene acceso deben estar habilitadas para CAE. Sin embargo, preparar el código para usar un recurso habilitado para CAE no le impedirá usar las API que no están habilitadas para tal fin.

Si una API de recursos implementa CAE y la aplicación declara que puede controlar CAE, la aplicación obtendrá tokens de CAE para ese recurso. Por esta razón, si declara que la aplicación está lista para CAE, la aplicación debe controlar el desafío de notificaciones CAE para todas las API de recursos que acepten tokens de acceso de identidad de Microsoft. Si no controla las respuestas CAE en estas llamadas API, la aplicación podría acabar en un bucle de reintento de una llamada API con un token que todavía está usando la duración devuelta del token, pero se ha revocado debido a CAE.

## <a name="the-code"></a>El código.

El primer paso consiste en agregar código para controlar una respuesta de la API de recursos que rechaza la llamada debido a CAE. Con CAE, las API devolverán un estado 401 y un encabezado WWW-Authenticate cuando el token de acceso se haya revocado o la API detecte un cambio en la dirección IP usada. El encabezado WWW-Authenticate contiene un desafío de notificaciones que la aplicación puede usar para adquirir un nuevo token de acceso.

Por ejemplo:

```console
HTTP 401; Unauthorized
WWW-Authenticate=Bearer
  authorization_uri="https://login.windows.net/common/oauth2/authorize",
  error="insufficient_claims",
  claims="eyJhY2Nlc3NfdG9rZW4iOnsibmJmIjp7ImVzc2VudGlhbCI6dHJ1ZSwgInZhbHVlIjoiMTYwNDEwNjY1MSJ9fX0="
```

La aplicación comprobaría:

- la llamada API que devuelve el estado 401
- la existencia de un encabezado WWW-Authenticate que contenga:
  - un parámetro "error" con el valor "insufficient_claims"
  - un parámetro "claims"

Cuando se cumplen estas condiciones, la aplicación puede extraer y descodificar el desafío de notificaciones.

```csharp
if (APIresponse.IsSuccessStatusCode)
{
    // ...
}
else
{
    if (APIresponse.StatusCode == System.Net.HttpStatusCode.Unauthorized
        && APIresponse.Headers.WwwAuthenticate.Any())
    {
        AuthenticationHeaderValue bearer = APIresponse.Headers.WwwAuthenticate.First
            (v => v.Scheme == "Bearer");
        IEnumerable<string> parameters = bearer.Parameter.Split(',').Select(v => v.Trim()).ToList();
        var error = GetParameter(parameters, "error");

        if (null != error && "insufficient_claims" == error)
        {
            var claimChallengeParameter = GetParameter(parameters, "claims");
            if (null != claimChallengeParameter)
            {
                var claimChallengebase64Bytes = System.Convert.FromBase64String(claimChallengeParameter);
                var claimChallenge = System.Text.Encoding.UTF8.GetString(claimChallengebase64Bytes);
                var newAccessToken = await GetAccessTokenWithClaimChallenge(scopes, claimChallenge);
```

A continuación, la aplicación usará el desafío de notificaciones para adquirir un nuevo token de acceso para el recurso.

```csharp
try
{
    authResult = await _clientApp.AcquireTokenSilent(scopes, firstAccount)
        .WithClaims(claimChallenge)
        .ExecuteAsync()
        .ConfigureAwait(false);
}
catch (MsalUiRequiredException)
{
    try
    {
        authResult = await _clientApp.AcquireTokenInteractive(scopes)
            .WithClaims(claimChallenge)
            .WithAccount(firstAccount)
            .ExecuteAsync()
            .ConfigureAwait(false);
    }
    // ...
```

Una vez que la aplicación esté lista para controlar el desafío de notificaciones que devuelve un habilitado para CAE, puede indicar a la identidad de Microsoft que la aplicación está lista. Para hacerlo desde la aplicación MSAL, incluya las funcionalidades de cliente "cp1" al crear su cliente público.

```csharp
_clientApp = PublicClientApplicationBuilder.Create(App.ClientId)
    .WithDefaultRedirectUri()
    .WithAuthority(authority)
    .WithClientCapabilities(new [] {"cp1"})
    .Build();
```

Para probar la aplicación, puede iniciar la sesión de un usuario en la aplicación mediante Azure Portal para revocar las sesiones del usuario. La próxima vez que la aplicación llame a la API habilitada para CAE, se le pedirá al usuario que vuelva a autenticarse.

## <a name="next-steps"></a>Pasos siguientes

- Introducción conceptual a la [evaluación de acceso continua](../conditional-access/concept-continuous-access-evaluation.md)
- [Desafíos de notificaciones, solicitudes de notificaciones y funcionalidades de cliente](claims-challenge.md)
