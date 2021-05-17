---
title: Uso de identidades administradas con una aplicación en un clúster administrado de Service Fabric
description: Cómo usar identidades administradas en el código de aplicación de Azure Service Fabric para acceder a servicios de Azure en un clúster administrado de Service Fabric.
ms.topic: article
ms.date: 5/10/2021
ms.openlocfilehash: 6bf92a22b347b62ce1be500006d3e00d1fd19cf2
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109689380"
---
# <a name="how-to-leverage-a-service-fabric-applications-managed-identity-to-access-azure-services-on-a-service-fabric-managed-cluster"></a>Aprovechamiento de una identidad administrada de una aplicación de Service Fabric para acceder a servicios de Azure en un clúster administrado de Service Fabric

Las aplicaciones de Service Fabric pueden sacar partido de las identidades administradas para acceder a otros recursos de Azure que admiten la autenticación basada en Azure Active Directory. Una aplicación puede obtener un [token de acceso](../active-directory/develop/developer-glossary.md#access-token) que represente su identidad (y que puede estar asignada por el sistema o por el usuario) y usarlo como token "portador" para autenticarse en otro servicio, también conocido como [servidor de recursos protegido](../active-directory/develop/developer-glossary.md#resource-server). El token representa la identidad asignada a la aplicación de Service Fabric, y solo se emitirá para recursos de Azure (aplicaciones de Service Fabric incluidas) que compartan esa identidad. Vea la documentación de [información general sobre las identidades administradas](../active-directory/managed-identities-azure-resources/overview.md) para obtener una descripción detallada de las identidades administradas, así como saber distinguir las identidades asignadas por el sistema y las asignadas por el usuario. A lo largo de este artículo, haremos referencia a una aplicación de Service Fabric habilitada para identidades administradas como la [aplicación cliente](../active-directory/develop/developer-glossary.md#client-application).

Vea una aplicación de ejemplo complementaria que muestra el uso de [identidades administradas de la aplicación Service Fabric](https://github.com/Azure-Samples/service-fabric-managed-identity) asignadas por el sistema y asignadas por el usuario con Reliable Services y contenedores.

> [!IMPORTANT]
> Una identidad administrada representa la asociación entre un recurso de Azure y una entidad de servicio en el inquilino de Azure AD correspondiente asociado a la suscripción que contiene el recurso en cuestión. Como tal, en el contexto de Service Fabric, las identidades administradas solo se admiten en aplicaciones implementadas como recursos de Azure. 

> [!IMPORTANT]
> Antes de utilizar la identidad administrada de una aplicación de Service Fabric, se debe conceder acceso a la aplicación cliente al recurso protegido. Consulte la lista de [servicios de Azure que admiten la autenticación de Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-managed-identities-for-azure-resources) para saber qué servicios son compatibles y, tras ello, vea la documentación del servicio correspondiente para conocer los pasos específicos para conceder acceso a una identidad a los recursos de interés. 
 

## <a name="leverage-a-managed-identity-using-azureidentity"></a>Utilizar una identidad administrada con Azure.Identity

El SDK de Azure Identity ahora es compatible con Service Fabric. El uso de Azure.Identity facilita la escritura de código para usar identidades administradas por la aplicación Service Fabric, ya que controla la captura de tokens, el almacenamiento en caché de tokens y la autenticación del servidor. Al acceder a la mayoría de los recursos de Azure, el concepto de un token está oculto.

La compatibilidad con Service Fabric está disponible en las siguientes versiones para estos lenguajes: 
- [C# en la versión 1.3.0](https://www.nuget.org/packages/Azure.Identity). Vea un [ejemplo de C#](https://github.com/Azure-Samples/service-fabric-managed-identity).
- [Python en la versión 1.5.0](https://pypi.org/project/azure-identity/). Consulte un [ejemplo de Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/identity/azure-identity/tests/managed-identity-live/service-fabric/service_fabric.md).
- [Java en la versión 1.2.0](/java/api/overview/azure/identity-readme).

Ejemplo de C# de inicialización de las credenciales y uso de estas para capturar un secreto desde Azure Key Vault:

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace MyMIService
{
    internal sealed class MyMIService : StatelessService
    {
        protected override async Task RunAsync(CancellationToken cancellationToken)
        {
            try
            {
                // Load the service fabric application managed identity assigned to the service
                ManagedIdentityCredential creds = new ManagedIdentityCredential();

                // Create a client to keyvault using that identity
                SecretClient client = new SecretClient(new Uri("https://mykv.vault.azure.net/"), creds);

                // Fetch a secret
                KeyVaultSecret secret = (await client.GetSecretAsync("mysecret", cancellationToken: cancellationToken)).Value;
            }
            catch (CredentialUnavailableException e)
            {
                // Handle errors with loading the Managed Identity
            }
            catch (RequestFailedException)
            {
                // Handle errors with fetching the secret
            }
            catch (Exception e)
            {
                // Handle generic errors
            }
        }
    }
}

```

## <a name="acquiring-an-access-token-using-rest-api"></a>Adquisición de un token de acceso mediante la API de REST
En los clústeres habilitados para identidades administradas, el tiempo de ejecución de Service Fabric expone un punto de conexión de host local que las aplicaciones pueden usar para obtener tokens de acceso. El punto de conexión está disponible en todos los nodos del clúster y está accesible para todas las entidades de ese nodo. Los autores de llamadas autorizados pueden obtener tokens de acceso llamando a ese punto de conexión y presentando un código de autenticación. Este código lo genera el tiempo de ejecución de Service Fabric por cada activación de paquete de código de servicio distinta, y durará lo que dure el proceso que hospeda ese paquete de código de servicio.

En concreto, el entorno de un servicio de Service Fabric habilitado para identidades administradas se inicializará con las siguientes variables:
- "IDENTITY_ENDPOINT": el punto de conexión de localhost correspondiente a la identidad administrada del servicio
- "IDENTITY_HEADER": un código de autenticación único que representa el servicio en el nodo actual
- "IDENTITY_SERVER_THUMBPRINT": Huella digital del servidor de identidad administrada de Service Fabric

> [!IMPORTANT]
> El código de aplicación debe considerar el valor de la variable de entorno "IDENTITY_HEADER" como un dato confidencial; no se debe registrar ni difundir en modo alguno. El código de autenticación no tiene ningún valor fuera del nodo local ni una vez finalizado el proceso que hospeda el servicio, pero representa la identidad del servicio de Service Fabric, por lo que se debe tratar con las mismas precauciones que el token de acceso en sí.

Para obtener un token, el cliente realiza los siguientes pasos:
- Forma un URI concatenando el punto de conexión de la identidad administrada (valor de "IDENTITY_ENDPOINT") con la versión de la API y el recurso (audiencia) necesario para el token.
- Crea una solicitud HTTPS GET para el URI especificado.
- Agrega la lógica de validación del certificado de servidor adecuada
- Agrega el código de autenticación (valor de "IDENTITY_HEADER") como un encabezado a la solicitud.
- Envía la solicitud.

Una respuesta correcta contendrá una carga de JSON que representa el token de acceso resultante, así como los metadatos que la describen. Una respuesta incorrecta también incluirá una explicación del error. Consulte a continuación más detalles sobre el tratamiento de errores.

Service Fabric almacena en caché los tokens de acceso en varios niveles (nodo, clúster, servicio del proveedor de recursos), por lo que una respuesta correcta no implica necesariamente que el token se emitió directamente como respuesta a la solicitud de la aplicación de usuario. Los tokens se almacenarán en caché menos tiempo de lo que dura su vigencia, por lo que se garantiza que una aplicación reciba un token válido. Se recomienda que el propio código de la aplicación almacene en caché los tokens de acceso que adquiera; la clave de almacenamiento en caché debe incluir una (derivación de) audiencia. 

Solicitud de ejemplo:
```http
GET 'https://localhost:2377/metadata/identity/oauth2/token?api-version=2019-07-01-preview&resource=https://vault.azure.net/' HTTP/1.1 Secret: 912e4af7-77ba-4fa5-a737-56c8e3ace132
```
donde:

| Elemento | Descripción |
| ------- | ----------- |
| `GET` | El verbo HTTP, indicando que se van a recuperar datos desde el punto de conexión. En este caso, el token de acceso de OAuth. | 
| `https://localhost:2377/metadata/identity/oauth2/token` | Punto de conexión de identidad administrada de las aplicaciones de Service Fabric, facilitado a través de la variable de entorno IDENTITY_ENDPOINT. |
| `api-version` | Parámetro de cadena de consulta que especifica la versión de la API del servicio de token de identidad administrado; actualmente, el único valor aceptado es `2019-07-01-preview` y está sujeto a cambios. |
| `resource` | Un parámetro de cadena de consulta, que indica el URI del identificador de aplicación del recurso de destino. Esto se reflejará como la notificación `aud` (audiencia) del token emitido. En este ejemplo se solicita un token para acceder a Azure Key Vault, cuyo URI de identificador de aplicación es https:\//vault.azure.net/. |
| `Secret` | Campo de encabezado de la solicitud HTTP, que el servicio de token de identidad administrada de Service Fabric necesita para que los servicios de Service Fabric puedan autenticar al autor de la llamada. Este valor lo facilita el tiempo de ejecución de Service Fabric a través de la variable de entorno IDENTITY_HEADER. |


Respuesta de ejemplo:
```json
HTTP/1.1 200 OK
Content-Type: application/json
{
    "token_type":  "Bearer",
    "access_token":  "eyJ0eXAiO...",
    "expires_on":  1565244611,
    "resource":  "https://vault.azure.net/"
}
```
donde:

| Elemento | Descripción |
| ------- | ----------- |
| `token_type` | Tipo de token; en este caso, es un token de acceso de tipo portador ("Bearer"), que significa que el portador de este token es el asunto previsto del token. |
| `access_token` | El token de acceso solicitado. Al llamar a una API de REST protegida, el token se inserta en el campo `Authorization` del encabezado de la solicitud como un token de "portador", lo que permite a la API autenticar el llamador. | 
| `expires_on` | Marca de tiempo de expiración del token de acceso; se representa como el número de segundos de "1970-01-01T0:0:0Z UTC" y corresponde a la notificación `exp` del token. En este caso, el token expira en 2019-08-08T06:10:11+00:00 (3339 en RFC).|
| `resource` | Recurso para el que se emitió el token de acceso, especificado a través del parámetro de cadena de consulta `resource` de la solicitud; corresponde a la notificación "aud" del token. |


## <a name="acquiring-an-access-token-using-c"></a>Adquisición de un token de acceso mediante C#
Lo anterior es lo siguiente en C#:

```C#
namespace Azure.ServiceFabric.ManagedIdentity.Samples
{
    using System;
    using System.Net.Http;
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Web;
    using Newtonsoft.Json;

    /// <summary>
    /// Type representing the response of the SF Managed Identity endpoint for token acquisition requests.
    /// </summary>
    [JsonObject]
    public sealed class ManagedIdentityTokenResponse
    {
        [JsonProperty(Required = Required.Always, PropertyName = "token_type")]
        public string TokenType { get; set; }

        [JsonProperty(Required = Required.Always, PropertyName = "access_token")]
        public string AccessToken { get; set; }

        [JsonProperty(PropertyName = "expires_on")]
        public string ExpiresOn { get; set; }

        [JsonProperty(PropertyName = "resource")]
        public string Resource { get; set; }
    }

    /// <summary>
    /// Sample class demonstrating access token acquisition using Managed Identity.
    /// </summary>
    public sealed class AccessTokenAcquirer
    {
        /// <summary>
        /// Acquire an access token.
        /// </summary>
        /// <returns>Access token</returns>
        public static async Task<string> AcquireAccessTokenAsync()
        {
            var managedIdentityEndpoint = Environment.GetEnvironmentVariable("IDENTITY_ENDPOINT");
            var managedIdentityAuthenticationCode = Environment.GetEnvironmentVariable("IDENTITY_HEADER");
            var managedIdentityServerThumbprint = Environment.GetEnvironmentVariable("IDENTITY_SERVER_THUMBPRINT");
            // Latest api version, 2019-07-01-preview is still supported.
            var managedIdentityApiVersion = Environment.GetEnvironmentVariable("IDENTITY_API_VERSION");
            var managedIdentityAuthenticationHeader = "secret";
            var resource = "https://management.azure.com/";

            var requestUri = $"{managedIdentityEndpoint}?api-version={managedIdentityApiVersion}&resource={HttpUtility.UrlEncode(resource)}";

            var requestMessage = new HttpRequestMessage(HttpMethod.Get, requestUri);
            requestMessage.Headers.Add(managedIdentityAuthenticationHeader, managedIdentityAuthenticationCode);
            
            var handler = new HttpClientHandler();
            handler.ServerCertificateCustomValidationCallback = (httpRequestMessage, cert, certChain, policyErrors) =>
            {
                // Do any additional validation here
                if (policyErrors == SslPolicyErrors.None)
                {
                    return true;
                }
                return 0 == string.Compare(cert.GetCertHashString(), managedIdentityServerThumbprint, StringComparison.OrdinalIgnoreCase);
            };

            try
            {
                var response = await new HttpClient(handler).SendAsync(requestMessage)
                    .ConfigureAwait(false);

                response.EnsureSuccessStatusCode();

                var tokenResponseString = await response.Content.ReadAsStringAsync()
                    .ConfigureAwait(false);

                var tokenResponseObject = JsonConvert.DeserializeObject<ManagedIdentityTokenResponse>(tokenResponseString);

                return tokenResponseObject.AccessToken;
            }
            catch (Exception ex)
            {
                string errorText = String.Format("{0} \n\n{1}", ex.Message, ex.InnerException != null ? ex.InnerException.Message : "Acquire token failed");

                Console.WriteLine(errorText);
            }

            return String.Empty;
        }
    } // class AccessTokenAcquirer
} // namespace Azure.ServiceFabric.ManagedIdentity.Samples
```
## <a name="accessing-key-vault-from-a-service-fabric-application-using-managed-identity"></a>Acceso a Key Vault desde una aplicación de Service Fabric mediante una identidad administrada
Este ejemplo se basa en el anterior para explicar cómo acceder a un secreto almacenado en una instancia de Key Vault usando una identidad administrada.

```C#
        /// <summary>
        /// Probe the specified secret, displaying metadata on success.  
        /// </summary>
        /// <param name="vault">vault name</param>
        /// <param name="secret">secret name</param>
        /// <param name="version">secret version id</param>
        /// <returns></returns>
        public async Task<string> ProbeSecretAsync(string vault, string secret, string version)
        {
            // initialize a KeyVault client with a managed identity-based authentication callback
            var kvClient = new Microsoft.Azure.KeyVault.KeyVaultClient(new Microsoft.Azure.KeyVault.KeyVaultClient.AuthenticationCallback((a, r, s) => { return AuthenticationCallbackAsync(a, r, s); }));

            Log(LogLevel.Info, $"\nRunning with configuration: \n\tobserved vault: {config.VaultName}\n\tobserved secret: {config.SecretName}\n\tMI endpoint: {config.ManagedIdentityEndpoint}\n\tMI auth code: {config.ManagedIdentityAuthenticationCode}\n\tMI auth header: {config.ManagedIdentityAuthenticationHeader}");
            string response = String.Empty;

            Log(LogLevel.Info, "\n== {DateTime.UtcNow.ToString()}: Probing secret...");
            try
            {
                var secretResponse = await kvClient.GetSecretWithHttpMessagesAsync(vault, secret, version)
                    .ConfigureAwait(false);

                if (secretResponse.Response.IsSuccessStatusCode)
                {
                    // use the secret: secretValue.Body.Value;
                    response = String.Format($"Successfully probed secret '{secret}' in vault '{vault}': {PrintSecretBundleMetadata(secretResponse.Body)}");
                }
                else
                {
                    response = String.Format($"Non-critical error encountered retrieving secret '{secret}' in vault '{vault}': {secretResponse.Response.ReasonPhrase} ({secretResponse.Response.StatusCode})");
                }
            }
            catch (Microsoft.Rest.ValidationException ve)
            {
                response = String.Format($"encountered REST validation exception 0x{ve.HResult.ToString("X")} trying to access '{secret}' in vault '{vault}' from {ve.Source}: {ve.Message}");
            }
            catch (KeyVaultErrorException kvee)
            {
                response = String.Format($"encountered KeyVault exception 0x{kvee.HResult.ToString("X")} trying to access '{secret}' in vault '{vault}': {kvee.Response.ReasonPhrase} ({kvee.Response.StatusCode})");
            }
            catch (Exception ex)
            {
                // handle generic errors here
                response = String.Format($"encountered exception 0x{ex.HResult.ToString("X")} trying to access '{secret}' in vault '{vault}': {ex.Message}");
            }

            Log(LogLevel.Info, response);

            return response;
        }

        /// <summary>
        /// KV authentication callback, using the application's managed identity.
        /// </summary>
        /// <param name="authority">The expected issuer of the access token, from the KV authorization challenge.</param>
        /// <param name="resource">The expected audience of the access token, from the KV authorization challenge.</param>
        /// <param name="scope">The expected scope of the access token; not currently used.</param>
        /// <returns>Access token</returns>
        public async Task<string> AuthenticationCallbackAsync(string authority, string resource, string scope)
        {
            Log(LogLevel.Verbose, $"authentication callback invoked with: auth: {authority}, resource: {resource}, scope: {scope}");
            var encodedResource = HttpUtility.UrlEncode(resource);

            // This sample does not illustrate the caching of the access token, which the user application is expected to do.
            // For a given service, the caching key should be the (encoded) resource uri. The token should be cached for a period
            // of time at most equal to its remaining validity. The 'expires_on' field of the token response object represents
            // the number of seconds from Unix time when the token will expire. You may cache the token if it will be valid for at
            // least another short interval (1-10s). If its expiration will occur shortly, don't cache but still return it to the 
            // caller. The MI endpoint will not return an expired token.
            // Sample caching code:
            //
            // ManagedIdentityTokenResponse tokenResponse;
            // if (responseCache.TryGetCachedItem(encodedResource, out tokenResponse))
            // {
            //     Log(LogLevel.Verbose, $"cache hit for key '{encodedResource}'");
            //
            //     return tokenResponse.AccessToken;
            // }
            //
            // Log(LogLevel.Verbose, $"cache miss for key '{encodedResource}'");
            //
            // where the response cache is left as an exercise for the reader. MemoryCache is a good option, albeit not yet available on .net core.

            var requestUri = $"{config.ManagedIdentityEndpoint}?api-version={config.ManagedIdentityApiVersion}&resource={encodedResource}";
            Log(LogLevel.Verbose, $"request uri: {requestUri}");

            var requestMessage = new HttpRequestMessage(HttpMethod.Get, requestUri);
            requestMessage.Headers.Add(config.ManagedIdentityAuthenticationHeader, config.ManagedIdentityAuthenticationCode);
            Log(LogLevel.Verbose, $"added header '{config.ManagedIdentityAuthenticationHeader}': '{config.ManagedIdentityAuthenticationCode}'");

            var response = await httpClient.SendAsync(requestMessage)
                .ConfigureAwait(false);
            Log(LogLevel.Verbose, $"response status: success: {response.IsSuccessStatusCode}, status: {response.StatusCode}");

            response.EnsureSuccessStatusCode();

            var tokenResponseString = await response.Content.ReadAsStringAsync()
                .ConfigureAwait(false);

            var tokenResponse = JsonConvert.DeserializeObject<ManagedIdentityTokenResponse>(tokenResponseString);
            Log(LogLevel.Verbose, "deserialized token response; returning access code..");

            // Sample caching code (continuation):
            // var expiration = DateTimeOffset.FromUnixTimeSeconds(Int32.Parse(tokenResponse.ExpiresOn));
            // if (expiration > DateTimeOffset.UtcNow.AddSeconds(5.0))
            //    responseCache.AddOrUpdate(encodedResource, tokenResponse, expiration);

            return tokenResponse.AccessToken;
        }

        private string PrintSecretBundleMetadata(SecretBundle bundle)
        {
            StringBuilder strBuilder = new StringBuilder();

            strBuilder.AppendFormat($"\n\tid: {bundle.Id}\n");
            strBuilder.AppendFormat($"\tcontent type: {bundle.ContentType}\n");
            strBuilder.AppendFormat($"\tmanaged: {bundle.Managed}\n");
            strBuilder.AppendFormat($"\tattributes:\n");
            strBuilder.AppendFormat($"\t\tenabled: {bundle.Attributes.Enabled}\n");
            strBuilder.AppendFormat($"\t\tnbf: {bundle.Attributes.NotBefore}\n");
            strBuilder.AppendFormat($"\t\texp: {bundle.Attributes.Expires}\n");
            strBuilder.AppendFormat($"\t\tcreated: {bundle.Attributes.Created}\n");
            strBuilder.AppendFormat($"\t\tupdated: {bundle.Attributes.Updated}\n");
            strBuilder.AppendFormat($"\t\trecoveryLevel: {bundle.Attributes.RecoveryLevel}\n");

            return strBuilder.ToString();
        }

        private enum LogLevel
        {
            Info,
            Verbose
        };

        private void Log(LogLevel level, string message)
        {
            if (level != LogLevel.Verbose
                || config.DoVerboseLogging)
            {
                Console.WriteLine(message);
            }
        }

```

## <a name="error-handling"></a>Control de errores
El campo "status code" del encabezado de respuesta HTTP indica un estado correcto de la solicitud; un estado "200 OK" pone de manifiesto que el proceso se ha realizado correctamente, y la respuesta incluirá el token de acceso, tal y como se ha descrito anteriormente. A continuación se muestra una breve lista de las posibles respuestas de error.

| Código de estado | Motivo del error | Realización del control |
| ----------- | ------------ | ------------- |
| 404 No encontrado. | Código de autenticación desconocido, o no se asignó una identidad administrada a la aplicación. | Corrija la configuración de la aplicación o el código de adquisición de token. |
| 429 Demasiadas solicitudes. |  Se alcanzó la limitación impuesta por AAD o Service Fabric. | Vuelva a intentarlo con retroceso exponencial. Consulte las instrucciones siguientes. |
| Error 4xx en la solicitud. | Uno o varios de los parámetros de solicitud eran incorrectos. | No vuelva a intentarlo.  Consulte los detalles del error para obtener más información.  Los errores 4xx son de tiempo de diseño.|
| Error 5xx del servicio. | El subsistema de identidades administradas o Azure Active Directory devolvieron un error transitorio. | Se puede volver a intentar con seguridad transcurrido un tiempo breve. Al reintentarlo, se puede alcanzar una condición de limitación (429).|

Si se produce un error, el cuerpo de respuesta HTTP correspondiente contiene un objeto JSON con estos detalles del error:

| Elemento | Descripción |
| ------- | ----------- |
| código | Código de error. |
| correlationId | Identificador de correlación que se puede usar en la depuración. |
| message | Descripción detallada del error. **Las descripciones de error pueden cambiar en cualquier momento. No se guíe únicamente por el mensaje de error.**|

Error de ejemplo:
```json
{"error":{"correlationId":"7f30f4d3-0f3a-41e0-a417-527f21b3848f","code":"SecretHeaderNotFound","message":"Secret is not found in the request headers."}}
```

Aquí se muestra una lista de los errores de Service Fabric habituales específicos de las identidades administradas:

| Código | Message | Descripción | 
| ----------- | ----- | ----------------- |
| SecretHeaderNotFound | No se encuentra el secreto en los encabezados de la solicitud. | No se ha facilitado el código de autenticación con la solicitud. | 
| ManagedIdentityNotFound | No se encontró la identidad administrada del host de aplicación especificado. | La aplicación no tiene ninguna identidad, o se desconoce el código de autenticación. |
| ArgumentNullOrEmpty | El parámetro "resource" no debe ser nulo ni una cadena vacía. | El recurso (audiencia) no se ha facilitado en la solicitud. |
| InvalidApiVersion | La versión de API no se admite. La versión admitida es "2019-07-01-preview". | Falta la versión de API o la que se ha especificado no se admite en el URI de solicitud. |
| InternalServerError | Se produjo un error. | Se ha encontrado un error en el subsistema de identidades administradas, posiblemente fuera de la pila de Service Fabric. La causa más probable es que se haya especificado un valor incorrecto del recurso (busque caracteres "/" finales). | 

## <a name="retry-guidance"></a>Instrucciones de reintento 

Normalmente, el único código de error que se puede reintentar es el 429 (demasiadas solicitudes); los errores de servidor internos/códigos de error 5xx se pueden reintentar, aunque la causa puede ser permanente. 

Las limitaciones se aplican al número de llamadas realizadas al subsistema de identidades administradas, en concreto, las dependencias de "canal de subida" (el servicio de Azure de identidades administradas o el servicio de token seguro). Service Fabric almacena en caché los tokens en varios niveles dentro de la canalización pero, dada la naturaleza dispersa de los componentes implicados, el autor de la llamada puede experimentar respuestas de limitación incoherentes (es decir, obtener limitaciones en un nodo o una instancia de una aplicación, pero no en otro nodo al solicitar un token para la misma identidad). Cuando se establece la condición de limitación, se puede producir un error en las solicitudes posteriores desde la misma aplicación, con el código de estado HTTP 429 (demasiadas solicitudes), hasta que la condición se borre.  

Se recomienda que las solicitudes que tengan errores debido a la limitación se vuelvan a intentar con un retroceso exponencial, como se indica a continuación: 

| Índice de llamada | Acción al recibir el código 429 | 
| --- | --- | 
| 1 | Esperar 1 segundo y reintentar. |
| 2 | Esperar 2 segundos y reintentar. |
| 3 | Esperar 4 segundos y reintentar. |
| 4 | Esperar 8 segundos y reintentar. |
| 4 | Esperar 8 segundos y reintentar. |
| 5 | Esperar 16 segundos y reintentar. |

## <a name="resource-ids-for-azure-services"></a>Identificadores de recurso para los servicios de Azure
Vea [Servicios de Azure que admiten la autenticación de Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md) para obtener una lista de los recursos que admiten Azure AD y sus respectivos identificadores de recurso.

## <a name="next-steps"></a>Pasos siguientes
* [Implementación de una aplicación de Service Fabric con una identidad administrada asignada por el sistema](./how-to-deploy-service-fabric-application-system-assigned-managed-identity.md)
* [Concesión de acceso a otros recursos de Azure para una aplicación de Azure Service Fabric](./how-to-managed-cluster-grant-access-other-resources.md)
* [Exploración de una aplicación de ejemplo mediante Identidad administrada de Service Fabric](https://github.com/Azure-Samples/service-fabric-managed-identity)