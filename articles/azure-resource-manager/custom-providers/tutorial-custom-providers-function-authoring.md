---
title: Creación de un punto de conexión RESTful
description: En este tutorial se muestra cómo crear un punto de conexión de RESTful para proveedores personalizados. Detalla cómo tratar las solicitudes y respuestas para los métodos HTTP de RESTful admitidos.
author: jjbfour
ms.topic: tutorial
ms.date: 01/13/2021
ms.author: jobreen
ms.openlocfilehash: 49a2d242ef5cdb9304a9b94d29328c8379a05f46
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108755628"
---
# <a name="author-a-restful-endpoint-for-custom-providers"></a>Creación de un punto de conexión de RESTful para proveedores personalizados

Un proveedor personalizado es un contrato entre Azure y un punto de conexión. Los proveedores personalizados le permiten personalizar los flujos de trabajo en Azure. En este tutorial se muestra cómo crear un punto de conexión de RESTful para proveedores personalizados. Si no está familiarizado con los proveedores personalizados de Azure, consulte [la introducción a los proveedores de recursos personalizados](overview.md).

> [!NOTE]
> Este tutorial se basa en el artículo [Configuración de Azure Functions para los proveedores personalizados de Azure](./tutorial-custom-providers-function-setup.md). Algunos de los pasos del tutorial solo funcionan si se ha configurado una aplicación de funciones en Azure Functions para que funcione con proveedores personalizados.

## <a name="work-with-custom-actions-and-custom-resources"></a>Trabajo con las acciones y recursos personalizados

En este tutorial, va a actualizar la aplicación de funciones para que sirve de punto de conexión de RESTful para el proveedor personalizado. Los recursos y las acciones de Azure se modelan en función de la especificación de RESTful básica:

- **PUT**: crear un recurso.
- **GET (instancia)** : recuperar un recurso existente.
- **DELETE**: quitar un recurso existente.
- **POST**: desencadenar una acción.
- **GET (colección)** : enumerar todos los recursos existentes.

 En este tutorial, va a utilizar Azure Table Storage. Sin embargo, cualquier servicio o de base de datos o de almacenamiento puede funcionar.

## <a name="partition-custom-resources-in-storage"></a>Creación de particiones de recursos personalizados en el almacenamiento

Como va a crear un servicio RESTful, es necesario almacenar los recursos creados. Para Azure Table Storage, es necesario generar las claves de partición y de fila de los datos. En el caso de los proveedores personalizados, los datos deben particionarse en el proveedor personalizado. Cuando se envía una solicitud entrante al proveedor personalizado, este agregará el encabezado `x-ms-customproviders-requestpath` a la solicitud saliente al punto de conexión.

En el ejemplo siguiente se muestra un encabezado `x-ms-customproviders-requestpath` de un recurso personalizado:

```
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/{myResourceType}/{myResourceName}
```

Según el encabezado `x-ms-customproviders-requestpath` del ejemplo, puede crear los parámetros *partitionKey* y *rowKey* para el almacenamiento, tal como se muestra en la tabla siguiente:

Parámetro | Plantilla | Descripción
---|---|---
*partitionKey* | `{subscriptionId}:{resourceGroupName}:{resourceProviderName}` | El parámetro *partitionKey* especifica cómo se particionan los datos. Por lo general, la instancia del proveedor personalizado se encarga de particionar los datos.
*rowKey* | `{myResourceType}:{myResourceName}` | El parámetro *rowKey* especifica el identificador individual de los datos. Normalmente, el identificador es el nombre del recurso.

También necesita crear una nueva clase para modelar el recurso personalizado. En este tutorial, va a agregar la siguiente clase **CustomResource** a la aplicación de funciones:

```csharp
// Custom Resource Table Entity
public class CustomResource : TableEntity
{
    public string Data { get; set; }
}
```
**CustomResource** es una clase genérica simple que acepta cualquier dato de entrada. Se basa en el parámetro **TableEntity**, que se usa para almacenar los datos. La clase **CustomResource** hereda dos propiedades de **TableEntity**: **partitionKey** y **rowKey**.

## <a name="support-custom-provider-restful-methods"></a>Compatibilidad con los métodos de RESTful del proveedor personalizado

> [!NOTE]
> Si no está copiando el código directamente del tutorial, el contenido de la respuesta debe ser un JSON válido que establece el encabezado `Content-Type` en `application/json`.

Ahora que ha configurado la partición de datos, cree el CRUD básico y desencadene métodos para los recursos y acciones personalizados. Dado que los proveedores personalizados actúan como servidores proxy, el punto de conexión RESTful debe modelar y controlar la solicitud y la respuesta. Los fragmentos de código siguientes muestran cómo controlar las operaciones de RESTful básicas.

### <a name="trigger-a-custom-action"></a>Desencadenamiento de la acción personalizada

En el caso de los proveedores personalizados, se desencadena una acción personalizada mediante las solicitudes POST. Una acción personalizada puede aceptar opcionalmente un cuerpo de la solicitud que contenga un conjunto de parámetros de entrada. La acción entonces devuelve una respuesta que indique el resultado de la acción y si se realizado correctamente o ha producido un error.

Agregue el método **TriggerCustomAction** siguiente a la aplicación de funciones:

```csharp
/// <summary>
/// Triggers a custom action with some side effects.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <returns>The HTTP response result of the custom action.</returns>
public static async Task<HttpResponseMessage> TriggerCustomAction(HttpRequestMessage requestMessage)
{
    var myCustomActionRequest = await requestMessage.Content.ReadAsStringAsync();

    var actionResponse = requestMessage.CreateResponse(HttpStatusCode.OK);
    actionResponse.Content = myCustomActionRequest != string.Empty ? 
        new StringContent(JObject.Parse(myCustomActionRequest).ToString(), System.Text.Encoding.UTF8, "application/json") :
        null;
    return actionResponse;
}
```

El método **TriggerCustomAction** acepta una solicitud entrante y simplemente devuelve la respuesta con un código de estado correcto.

### <a name="create-a-custom-resource"></a>Creación de un recurso personalizado

En el caso de los proveedores personalizados, se crea un recurso personalizado mediante las solicitudes PUT. El proveedor personalizado acepta un cuerpo de la solicitud JSON, que contiene un conjunto de propiedades para el recurso personalizado. Los recursos de Azure siguen un modelo de RESTful. Puede usar la misma dirección URL de solicitud para crear, recuperar o eliminar un recurso.

Agregue el método **CreateCustomResource** siguiente para crear nuevos recursos:

```csharp
/// <summary>
/// Creates a custom resource and saves it to table storage.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <param name="azureResourceId">The parsed Azure resource ID.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="rowKey">The row key for storage. This is '{resourceType}:{customResourceName}'.</param>
/// <returns>The HTTP response containing the created custom resource.</returns>
public static async Task<HttpResponseMessage> CreateCustomResource(HttpRequestMessage requestMessage, CloudTable tableStorage, ResourceId azureResourceId, string partitionKey, string rowKey)
{
    // Adds the Azure top-level properties.
    var myCustomResource = JObject.Parse(await requestMessage.Content.ReadAsStringAsync());
    myCustomResource["name"] = azureResourceId.Name;
    myCustomResource["type"] = azureResourceId.FullResourceType;
    myCustomResource["id"] = azureResourceId.Id;

    // Save the resource into storage.
    var insertOperation = TableOperation.InsertOrReplace(
        new CustomResource
        {
            PartitionKey = partitionKey,
            RowKey = rowKey,
            Data = myCustomResource.ToString(),
        });
    await tableStorage.ExecuteAsync(insertOperation);

    var createResponse = requestMessage.CreateResponse(HttpStatusCode.OK);
    createResponse.Content = new StringContent(myCustomResource.ToString(), System.Text.Encoding.UTF8, "application/json");
    return createResponse;
}
```

El método **CreateCustomResource** actualiza la solicitud entrante para incluir los campos específicos de Azure **id**, **name** y **type**. Estos campos son propiedades de nivel superior que utilizan los servicios en Azure. Permiten que el proveedor personalizado se integre con otros servicios como Azure Policy, plantillas de Azure Resource Manager y el registro de actividades de Azure.

Propiedad | Ejemplo | Descripción
---|---|---
**name** | {myCustomResourceName} | El nombre del recurso personalizado
**type** | Microsoft.CustomProviders/resourceProviders/{resourceTypeName} | El espacio de nombres del tipo de recurso
**identificador** | /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/<br>providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/<br>{resourceTypeName}/{myCustomResourceName} | El identificador del recurso

Además de agregar las propiedades, también guardamos el documento JSON en Azure Table Storage.

### <a name="retrieve-a-custom-resource"></a>Recuperación de un recurso personalizado

En el caso de los proveedores personalizados, se recupera un recurso personalizado mediante las solicitudes GET. El proveedor personalizado *no* acepta un cuerpo de la solicitud JSON. En las solicitudes GET, el punto de conexión utiliza el encabezado `x-ms-customproviders-requestpath` para devolver el recurso ya creado.

Agregue el método **RetrieveCustomResource** siguiente para recuperar los recursos existentes:

```csharp
/// <summary>
/// Retrieves a custom resource.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="rowKey">The row key for storage. This is '{resourceType}:{customResourceName}'.</param>
/// <returns>The HTTP response containing the existing custom resource.</returns>
public static async Task<HttpResponseMessage> RetrieveCustomResource(HttpRequestMessage requestMessage, CloudTable tableStorage, string partitionKey, string rowKey)
{
    // Attempt to retrieve the Existing Stored Value
    var tableQuery = TableOperation.Retrieve<CustomResource>(partitionKey, rowKey);
    var existingCustomResource = (CustomResource)(await tableStorage.ExecuteAsync(tableQuery)).Result;

    var retrieveResponse = requestMessage.CreateResponse(
        existingCustomResource != null ? HttpStatusCode.OK : HttpStatusCode.NotFound);

    retrieveResponse.Content = existingCustomResource != null ?
            new StringContent(existingCustomResource.Data, System.Text.Encoding.UTF8, "application/json"):
            null;
    return retrieveResponse;
}
```

En Azure, los recursos siguen un modelo de RESTful. La dirección URL de la solicitud que crea un recurso también devuelve el recurso si se realiza una solicitud GET.

### <a name="remove-a-custom-resource"></a>Eliminación de un recurso personalizado

En los proveedores personalizados, se quita un recurso personalizado mediante las solicitudes DELETE. El proveedor personalizado *no* acepta un cuerpo de la solicitud JSON. En las solicitudes DELETE, el punto de conexión utiliza el encabezado `x-ms-customproviders-requestpath` para eliminar el recurso ya creado.

Agregue el método **RemoveCustomResource** siguiente para quitar los recursos existentes:

```csharp
/// <summary>
/// Removes an existing custom resource.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure storage account table.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="rowKey">The row key for storage. This is '{resourceType}:{customResourceName}'.</param>
/// <returns>The HTTP response containing the result of the deletion.</returns>
public static async Task<HttpResponseMessage> RemoveCustomResource(HttpRequestMessage requestMessage, CloudTable tableStorage, string partitionKey, string rowKey)
{
    // Attempt to retrieve the Existing Stored Value
    var tableQuery = TableOperation.Retrieve<CustomResource>(partitionKey, rowKey);
    var existingCustomResource = (CustomResource)(await tableStorage.ExecuteAsync(tableQuery)).Result;

    if (existingCustomResource != null) {
        var deleteOperation = TableOperation.Delete(existingCustomResource);
        await tableStorage.ExecuteAsync(deleteOperation);
    }

    return requestMessage.CreateResponse(
        existingCustomResource != null ? HttpStatusCode.OK : HttpStatusCode.NoContent);
}
```

En Azure, los recursos siguen un modelo de RESTful. La dirección URL de la solicitud que crea un recurso también elimina el recurso si se realiza una solicitud DELETE.

### <a name="list-all-custom-resources"></a>Enumeración de recursos personalizados

Para los proveedores personalizados, se puede enumerar una lista de los recursos personalizados existentes mediante las solicitudes GET de la colección. El proveedor personalizado *no* acepta un cuerpo de la solicitud JSON. Para las solicitudes GET de la colección, el punto de conexión debe utilizar el encabezado `x-ms-customproviders-requestpath` para enumerar los recursos ya creados.

Agregue el método **EnumerateAllCustomResources** siguiente para enumerar los recursos existentes:

```csharp
/// <summary>
/// Enumerates all the stored custom resources for a given type.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="resourceType">The resource type of the enumeration.</param>
/// <returns>The HTTP response containing a list of resources stored under 'value'.</returns>
public static async Task<HttpResponseMessage> EnumerateAllCustomResources(HttpRequestMessage requestMessage, CloudTable tableStorage, string partitionKey, string resourceType)
{
    // Generate upper bound of the query.
    var rowKeyUpperBound = new StringBuilder(resourceType);
    rowKeyUpperBound[rowKeyUpperBound.Length - 1]++;

    // Create the enumeration query.
    var enumerationQuery = new TableQuery<CustomResource>().Where(
        TableQuery.CombineFilters(
            TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, partitionKey),
            TableOperators.And,
            TableQuery.CombineFilters(
                TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.GreaterThan, resourceType),
                TableOperators.And,
                TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, rowKeyUpperBound.ToString()))));
    
    var customResources = (await tableStorage.ExecuteQuerySegmentedAsync(enumerationQuery, null))
        .ToList().Select(customResource => JToken.Parse(customResource.Data));

    var enumerationResponse = requestMessage.CreateResponse(HttpStatusCode.OK);
    enumerationResponse.Content = new StringContent(new JObject(new JProperty("value", customResources)).ToString(), System.Text.Encoding.UTF8, "application/json");
    return enumerationResponse;
}
```

> [!NOTE]
> Los métodos RowKey QueryComparisons.GreaterThan y QueryComparisons.LessThan es la sintaxis de Azure Table Storage para realizar una consulta "StartsWith" para las cadenas.

Para enumerar todos los recursos existentes, se genera una consulta de Azure Table Storage que garantiza que los recursos existen bajo nuestra partición de proveedor personalizado. La consulta comprueba entonces que la clave de fila comienza con el mismo valor `{myResourceType}`.

## <a name="integrate-restful-operations"></a>Integración de las operaciones de RESTful

Cuando todos los métodos RESTful se agregan a la aplicación de funciones, se actualiza el método **Run** principal para llamar a las funciones y tratar las diferentes solicitudes REST:

```csharp
/// <summary>
/// Entry point for the function app webhook that acts as the service behind a custom provider.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="log">The logger.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <returns>The HTTP response for the custom Azure API.</returns>
public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger log, CloudTable tableStorage)
{
    // Get the unique Azure request path from request headers.
    var requestPath = req.Headers.GetValues("x-ms-customproviders-requestpath").FirstOrDefault();

    if (requestPath == null)
    {
        var missingHeaderResponse = req.CreateResponse(HttpStatusCode.BadRequest);
        missingHeaderResponse.Content = new StringContent(
            new JObject(new JProperty("error", "missing 'x-ms-customproviders-requestpath' header")).ToString(),
            System.Text.Encoding.UTF8, 
            "application/json");
    }

    log.LogInformation($"The Custom Provider Function received a request '{req.Method}' for resource '{requestPath}'.");

    // Determines if it is a collection level call or action.
    var isResourceRequest = requestPath.Split('/').Length % 2 == 1;
    var azureResourceId = isResourceRequest ? 
        ResourceId.FromString(requestPath) :
        ResourceId.FromString($"{requestPath}/");

    // Create the Partition Key and Row Key
    var partitionKey = $"{azureResourceId.SubscriptionId}:{azureResourceId.ResourceGroupName}:{azureResourceId.Parent.Name}";
    var rowKey = $"{azureResourceId.FullResourceType.Replace('/', ':')}:{azureResourceId.Name}";

    switch (req.Method)
    {
        // Action request for a custom action.
        case HttpMethod m when m == HttpMethod.Post && !isResourceRequest:
            return await TriggerCustomAction(
                requestMessage: req);

        // Enumerate request for all custom resources.
        case HttpMethod m when m == HttpMethod.Get && !isResourceRequest:
            return await EnumerateAllCustomResources(
                requestMessage: req,
                tableStorage: tableStorage,
                partitionKey: partitionKey,
                resourceType: rowKey);

        // Retrieve request for a custom resource.
        case HttpMethod m when m == HttpMethod.Get && isResourceRequest:
            return await RetrieveCustomResource(
                requestMessage: req,
                tableStorage: tableStorage,
                partitionKey: partitionKey,
                rowKey: rowKey);

        // Create request for a custom resource.
        case HttpMethod m when m == HttpMethod.Put && isResourceRequest:
            return await CreateCustomResource(
                requestMessage: req,
                tableStorage: tableStorage,
                azureResourceId: azureResourceId,
                partitionKey: partitionKey,
                rowKey: rowKey);

        // Remove request for a custom resource.
        case HttpMethod m when m == HttpMethod.Delete && isResourceRequest:
            return await RemoveCustomResource(
                requestMessage: req,
                tableStorage: tableStorage,
                partitionKey: partitionKey,
                rowKey: rowKey);

        // Invalid request received.
        default:
            return req.CreateResponse(HttpStatusCode.BadRequest);
    }
}
```

El método actualizado **Run** ahora incluye el enlace de entrada *tableStorage* que agregó para Azure Table Storage. La primera parte del método lee el encabezado `x-ms-customproviders-requestpath` y usará la biblioteca `Microsoft.Azure.Management.ResourceManager.Fluent` para analizar el valor como un identificador de recurso. El encabezado `x-ms-customproviders-requestpath` lo envía el proveedor personalizado y especifica la ruta de acceso de la solicitud entrante.

Con el identificador del recurso analizado, ahora se puede generar los valores **partitionKey** y **rowKey** para que los datos busquen o almacenen recursos personalizados.

Después de agregar los métodos y las clases, hay que actualizar los métodos **using** de la aplicación de funciones. Agregue el código siguiente al principio del archivo de C#:

```csharp
#r "Newtonsoft.Json"
#r "Microsoft.WindowsAzure.Storage"
#r "../bin/Microsoft.Azure.Management.ResourceManager.Fluent"

using System;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Configuration;
using System.Text;
using System.Threading;
using System.Globalization;
using System.Collections.Generic;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.WindowsAzure.Storage.Table;
using Microsoft.Azure.Management.ResourceManager.Fluent.Core;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
```

Si se perdió en algún punto de este tutorial, puede encontrar el ejemplo de código completo en la [referencia de punto de conexión RESTful de C# del proveedor personalizado](./reference-custom-providers-csharp-endpoint.md). Una vez finalizada la aplicación de funciones, guarde la dirección URL de la aplicación de funciones. Se puede usar para desencadenar la aplicación de funciones en tutoriales posteriores.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado un punto de conexión de RESTful para trabajar con un punto de conexión de proveedor personalizado de Azure. Para aprender a crear un proveedor personalizado, vaya al artículo [Tutorial: Creación de un proveedor personalizado](./tutorial-custom-providers-create.md).
