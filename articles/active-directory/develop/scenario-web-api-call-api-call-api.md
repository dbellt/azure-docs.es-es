---
title: API web que llama a API web | Azure
titleSuffix: Microsoft identity platform
description: Aprenda a compilar una API web que llama a otras API web.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/26/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: d4526446b657b46f97306f51b2db325feefc8eff
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108166494"
---
# <a name="a-web-api-that-calls-web-apis-call-an-api"></a>Una API web que llama a las API web: Llamar a una API

Una vez que disponga de un token, puede llamar a una API web protegida. Normalmente, se llama a las API de nivel inferior desde el controlador o las páginas de la API web.

## <a name="controller-code"></a>Código del controlador

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Cuando se usa *Microsoft.Identity.Web*, hay tres escenarios de uso:

- [Opción 1: Llamada a Microsoft Graph con el SDK](#option-1-call-microsoft-graph-with-the-sdk)
- [Opción 2: Llamada a una API web de nivel inferior con la clase auxiliar](#option-2-call-a-downstream-web-api-with-the-helper-class)
- [Opción 3: Llamada a una API web de nivel inferior sin la clase auxiliar](#option-3-call-a-downstream-web-api-without-the-helper-class)

#### <a name="option-1-call-microsoft-graph-with-the-sdk"></a>Opción 1: Llamada a Microsoft Graph con el SDK

En este escenario, ha agregado `.AddMicrosoftGraph()` en el archivo *Startup.cs* tal y como se especifica en [Configuración del código](scenario-web-api-call-api-app-configuration.md#option-1-call-microsoft-graph) y puede insertar directamente el elemento `GraphServiceClient` en el controlador o en el constructor de la página para su uso en las acciones. En la siguiente página de Razor de ejemplo se muestra la foto del usuario que inició sesión.

```csharp
 [Authorize]
 [AuthorizeForScopes(Scopes = new[] { "user.read" })]
 public class IndexModel : PageModel
 {
     private readonly GraphServiceClient _graphServiceClient;

     public IndexModel(GraphServiceClient graphServiceClient)
     {
         _graphServiceClient = graphServiceClient;
     }

     public async Task OnGet()
     {
         var user = await _graphServiceClient.Me.Request().GetAsync();
         try
         {
             using (var photoStream = await _graphServiceClient.Me.Photo.Content.Request().GetAsync())
             {
                 byte[] photoByte = ((MemoryStream)photoStream).ToArray();
                 ViewData["photo"] = Convert.ToBase64String(photoByte);
             }
             ViewData["name"] = user.DisplayName;
         }
         catch (Exception)
         {
             ViewData["photo"] = null;
         }
     }
 }
```

#### <a name="option-2-call-a-downstream-web-api-with-the-helper-class"></a>Opción 2: Llamada a una API web de nivel inferior con la clase auxiliar

En este escenario, ha agregado `.AddDownstreamWebApi()` en el archivo *Startup.cs* tal y como se especifica en [Configuración del código](scenario-web-api-call-api-app-configuration.md#option-2-call-a-downstream-web-api-other-than-microsoft-graph) y puede insertar directamente el servicio `IDownstreamWebApi` en el controlador o en el constructor de la página para su uso en las acciones:

```csharp
 [Authorize]
 [AuthorizeForScopes(ScopeKeySection = "TodoList:Scopes")]
 public class TodoListController : Controller
 {
     private IDownstreamWebApi _downstreamWebApi;
     private const string ServiceName = "TodoList";

     public TodoListController(IDownstreamWebApi downstreamWebApi)
     {
         _downstreamWebApi = downstreamWebApi;
     }

     public async Task<ActionResult> Details(int id)
     {
         var value = await _downstreamWebApi.CallWebApiForUserAsync(
             ServiceName,
             options =>
             {
                 options.RelativePath = $"me";
             });
         return View(value);
     }
```

El método `CallWebApiForUserAsync` también tiene invalidaciones genéricas fuertemente tipadas que permiten recibir directamente un objeto. Por ejemplo, el método siguiente ha recibido una instancia de `Todo`, que es una representación fuertemente tipada del código JSON devuelto por la API web.

```csharp
 // GET: TodoList/Details/5
 public async Task<ActionResult> Details(int id)
 {
     var value = await _downstreamWebApi.CallWebApiForUserAsync<object, Todo>(
         ServiceName,
         null,
         options =>
         {
             options.HttpMethod = HttpMethod.Get;
             options.RelativePath = $"api/todolist/{id}";
         });
     return View(value);
 }
```

#### <a name="option-3-call-a-downstream-web-api-without-the-helper-class"></a>Opción 3: Llamada a una API web de nivel inferior sin la clase auxiliar

Ha decidido adquirir un token manualmente mediante el servicio `ITokenAcquisition`, ahora debe usar el token. En ese caso, el código siguiente continúa el código de ejemplo que se muestra en [API web que llama a las API web: Adquisición de un token para la aplicación](scenario-web-api-call-api-acquire-token.md). El código se llama en las acciones de los controladores de API. Llama a una API de nivel inferior denominada *todolist*.

 Una vez que haya adquirido el token, úselo como token de portador para llamar a la API de nivel inferior.

```csharp
private async Task CallTodoListService(string accessToken)
{
  // After the token has been returned by Microsoft.Identity.Web, add it to the HTTP authorization header before making the call to access the todolist service.
  _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

  // Call the todolist service.
  HttpResponseMessage response = await _httpClient.GetAsync(TodoListBaseAddress + "/api/todolist");
  // ...
}
```

# <a name="java"></a>[Java](#tab/java)

El código siguiente continúa el código de ejemplo que se muestra en [Una API web que llama a las API web: Adquisición de un token para la aplicación](scenario-web-api-call-api-acquire-token.md). El código se llama en las acciones de los controladores de API. Llama a la API de nivel inferior MS Graph.

Una vez que haya adquirido el token, úselo como token de portador para llamar a la API de nivel inferior.

```java
private String callMicrosoftGraphMeEndpoint(String accessToken){
    RestTemplate restTemplate = new RestTemplate();

    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);

    headers.set("Authorization", "Bearer " + accessToken);

    HttpEntity<String> entity = new HttpEntity<>(null, headers);

    String result = restTemplate.exchange("https://graph.microsoft.com/v1.0/me", HttpMethod.GET,
            entity, String.class).getBody();

    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

Puede encontrar un ejemplo donde se muestra este flujo con MSAL para Python en [ms-identity-python-on-behalf-of](https://github.com/Azure-Samples/ms-identity-python-on-behalf-of).

---

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente artículo de este escenario, [Paso a producción](scenario-web-api-call-api-production.md).
