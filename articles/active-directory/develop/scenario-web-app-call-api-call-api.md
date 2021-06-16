---
title: Llamada a una API web desde una aplicación web | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo compilar una aplicación web que llame a las API web (llamada a una API web protegida)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/25/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: fadab51318bc41851b6ea74c3a85ee797dfdb8e0
ms.sourcegitcommit: e832f58baf0b3a69c2e2781bd8e32d4f1ae932c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110584932"
---
# <a name="a-web-app-that-calls-web-apis-call-a-web-api"></a>Aplicación web que llama a las API web: Llamada a una API de web

Ahora que dispone de un token, puede llamar a una API web protegida. Normalmente, se llama a una API de nivel inferior desde el controlador o las páginas de la aplicación web.

## <a name="call-a-protected-web-api"></a>Llamada a una API web protegida

La llamada a una API web protegida depende del lenguaje y el marco elegidos:

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Cuando se usa *Microsoft.Identity.Web*, hay tres opciones de uso para llamar a una API:

- [Opción 1: Llamada a Microsoft Graph con el SDK de Microsoft Graph](#option-1-call-microsoft-graph-with-the-sdk)
- [Opción 2: Llamada a una API web de nivel inferior con la clase auxiliar](#option-2-call-a-downstream-web-api-with-the-helper-class)
- [Opción 3: Llamada a una API web de nivel inferior sin la clase auxiliar](#option-3-call-a-downstream-web-api-without-the-helper-class)

#### <a name="option-1-call-microsoft-graph-with-the-sdk"></a>Opción 1: Llamada a Microsoft Graph con el SDK

Desea llamar a Microsoft Graph. En este escenario, ha agregado `AddMicrosoftGraph` en el archivo *Startup.cs* tal y como se especifica en [Configuración del código](scenario-web-app-call-api-app-configuration.md#option-1-call-microsoft-graph) y puede insertar directamente el elemento `GraphServiceClient` en el controlador o en el constructor de la página para su uso en las acciones. En la siguiente página de Razor de ejemplo se muestra la foto del usuario que inició sesión.

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

Desea llamar a una API web que no es Microsoft Graph. En ese caso, ha agregado `AddDownstreamWebApi` en el archivo *Startup.cs* tal y como se especifica en [Configuración del código](scenario-web-app-call-api-app-configuration.md#option-2-call-a-downstream-web-api-other-than-microsoft-graph) y puede insertar directamente el servicio `IDownstreamWebApi` en el controlador o en el constructor de la página y usarlo en las acciones:

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
}
```

`CallWebApiForUserAsync` también tiene invalidaciones genéricas fuertemente tipadas que permiten recibir directamente un objeto. Por ejemplo, el método siguiente recibe una instancia de `Todo`, que es una representación fuertemente tipada del código JSON devuelto por la API web.

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

Ha decidido adquirir un token manualmente mediante el servicio `ITokenAcquisition` y ahora debe usar el token. En ese caso, el código siguiente continúa el código de ejemplo que se muestra en [Aplicación web que llama a las API web: Adquisición de un token para la aplicación](scenario-web-app-call-api-acquire-token.md). Se llama al código en las acciones de los controladores de la aplicación web.

Una vez que haya adquirido el token, úselo como token de portador para llamar a la API de nivel inferior, que en este caso es Microsoft Graph.

```csharp
public async Task<IActionResult> Profile()
{
  // Acquire the access token.
  string[] scopes = new string[]{"user.read"};
  string accessToken = await tokenAcquisition.GetAccessTokenForUserAsync(scopes);

  // Use the access token to call a protected web API.
  HttpClient httpClient = new HttpClient();
  httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

  var response = await httpClient.GetAsync($"{webOptions.GraphApiUrl}/beta/me");

  if (response.StatusCode == HttpStatusCode.OK)
  {
    var content = await response.Content.ReadAsStringAsync();

    dynamic me = JsonConvert.DeserializeObject(content);
    ViewData["Me"] = me;
  }

  return View();
}
```

> [!NOTE]
> Puede usar el mismo principio para llamar a cualquier API web.
>
> La mayoría de las API web de Azure proporcionan un SDK que simplifica la llamada a la API, como en el caso de Microsoft Graph. Consulte, por ejemplo, [Adquisición de un token de Azure AD para la autorización de solicitudes desde una aplicación cliente](../../storage/common/storage-auth-aad-app.md?tabs=dotnet&toc=%2fazure%2fstorage%2fblobs%2ftoc.json) para obtener un ejemplo de una aplicación web que usa Microsoft.Identity.Web y el SDK de Azure Storage.

# <a name="java"></a>[Java](#tab/java)

```java
private String getUserInfoFromGraph(String accessToken) throws Exception {
    // Microsoft Graph user endpoint
    URL url = new URL("https://graph.microsoft.com/v1.0/me");

    HttpURLConnection conn = (HttpURLConnection) url.openConnection();

    // Set the appropriate header fields in the request header.
    conn.setRequestProperty("Authorization", "Bearer " + accessToken);
    conn.setRequestProperty("Accept", "application/json");

    String response = HttpClientHelper.getResponseStringFromConn(conn);

    int responseCode = conn.getResponseCode();
    if(responseCode != HttpURLConnection.HTTP_OK) {
        throw new IOException(response);
    }

    JSONObject responseObject = HttpClientHelper.processResponse(responseCode, response);
    return responseObject.toString();
}
```

# <a name="python"></a>[Python](#tab/python)

```python
@app.route("/graphcall")
def graphcall():
    token = _get_token_from_cache(app_config.SCOPE)
    if not token:
        return redirect(url_for("login"))
    graph_data = requests.get(  # Use token to call downstream service.
        app_config.ENDPOINT,
        headers={'Authorization': 'Bearer ' + token['access_token']},
        ).json()
    return render_template('display.html', result=graph_data)
```

---

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente artículo de este escenario, [Paso a producción](scenario-web-app-call-api-production.md).
