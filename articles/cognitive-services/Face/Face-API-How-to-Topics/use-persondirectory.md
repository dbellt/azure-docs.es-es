---
title: 'Ejemplo: Uso de la estructura PersonDirectory - Face'
titleSuffix: Azure Cognitive Services
description: Aprenda a usar la estructura de datos PersonDirectory para almacenar datos de caras y personas con una mayor capacidad y con otras características nuevas.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 04/22/2021
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 8d7238aea62b3532461cd40404b683e18227386f
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111950982"
---
# <a name="use-the-persondirectory-structure"></a>Uso de la estructura de PersonDirectory

A fin de realizar operaciones de reconocimiento facial como Identificar y Buscar similares, los clientes de Face API deben crear una lista de objetos **Person**. El nuevo objeto **PersonDirectory** es una estructura de datos que contiene identificadores únicos, cadenas de nombre opcionales y cadenas de metadatos de usuario opcionales para cada objeto de identidad **Person** agregada al directorio.

Actualmente, Face API ofrece la estructura **LargePersonGroup**, que tiene una funcionalidad similar, pero está limitada a un millón de identidades. La estructura **PersonDirectory** se puede escalar verticalmente hasta 75 millones de identidades.

Otra diferencia importante entre **PersonDirectory** y las estructuras de datos anteriores es que ya no necesitará realizar una llamada a Train después de agregar caras a un objeto **Person**; &mdash;el proceso de actualización se produce automáticamente.

## <a name="prerequisites"></a>Prerrequisitos
* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Una vez que tenga la suscripción de Azure, [cree un recurso de Face](https://portal.azure.com/#create/Microsoft.CognitiveServicesFace) en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
  * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Face API. Pegará la clave y el punto de conexión en el código siguiente.
  * Puede usar el plan de tarifa gratis (F0) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="add-persons-to-the-persondirectory"></a>Adición de objetos Person a un PersonDirectory
Los objetos **Person** son las unidades de inscripción base de **PersonDirectory**. Una vez que haya agregado un objeto **Person** al directorio, podrá agregar hasta 248 imágenes de cara a ese objeto **Person**, por cada modelo de reconocimiento. De este modo, podrá identificar caras en ellas según distintos ámbitos.

### <a name="create-the-person"></a>Creación del objeto Person
Para crear un objeto **Person**, debe llamar a la **CreatePerson** API y proporcionar un nombre o un valor de propiedad userData.

```csharp
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Threading.Tasks;

var client = new HttpClient();

// Request headers
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "{subscription key}");

var addPersonUri = "https:// {endpoint}/face/v1.0-preview/persons";

HttpResponseMessage response;

// Request body
var body = new Dictionary<string, object>();
body.Add("name", "Example Person");
body.Add("userData", "User defined data");
byte[] byteData = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(body));

using (var content = new ByteArrayContent(byteData))
{
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    response = await client.PostAsync(addPersonUri, content); 
}
```

La llamada a CreatePerson devolverá un identificador generado para el objeto **Person** y una ubicación de operación. Los datos del objeto **Person** se procesarán de forma asincrónica, por lo que usará la ubicación de la operación para capturar los resultados. 

### <a name="wait-for-asynchronous-operation-completion"></a>Espera hasta la finalización de la operación asincrónica
Tendrá que consultar el estado de la operación asincrónica con la cadena de ubicación de la operación devuelta para comprobar el progreso. 

En primer lugar, debe definir un modelo de datos como el siguiente para controlar la respuesta de estado.

```csharp
[Serializable]
public class AsyncStatus
{
    [DataMember(Name = "status")]
    public string AsyncStatus { get; set; }

    [DataMember(Name = "createdTime")]
    public DateTime CreatedTime { get; set; }

    [DataMember(Name = "lastActionTime")]
    public DateTime? LastActionTime { get; set; }

    [DataMember(Name = "finishedTime", EmitDefaultValue = false)]
    public DateTime? FinishedTime { get; set; }

    [DataMember(Name = "resourceLocation", EmitDefaultValue = false)]
    public string ResourceLocation { get; set; }

    [DataMember(Name = "message", EmitDefaultValue = false)]
    public string Message { get; set; }
}
```

Con el elemento HttpResponseMessage anterior, puede sondear la dirección URL y esperar los resultados.

```csharp
string operationLocation = response.Headers.GetValues("Operation-Location").FirstOrDefault();

Stopwatch s = Stopwatch.StartNew();
string status = "notstarted";
do
{
    if (status == "succeeded")
    {
        await Task.Delay(500);
    }

    var operationResponseMessage = await client.GetAsync(operationLocation);

    var asyncOperationObj = JsonConvert.DeserializeObject<AsyncStatus>(await operationResponseMessage.Content.ReadAsStringAsync());
    status = asyncOperationObj.Status;

} while ((status == "running" || status == "notstarted") && s.Elapsed < TimeSpan.FromSeconds(30));
```


Una vez que el estado se haya devuelto como "succeeded", el objeto **Person** se considera agregado al directorio.

> [!NOTE]
> La operación asincrónica de la llamada a Create **Person** no tiene que mostrar el estado "succeeded" antes de que se puedan agregar caras a ella, pero debe completarse antes de que el objeto **Person** se pueda agregar a un elemento **DynamicPersonGroup** (consulte la sección Creación y actualización de un elemento **DynamicPersonGroup**) o compararlo durante una llamada a Identify. Compruebe que las llamadas funcionarán inmediatamente después de que las caras se hayan agregado correctamente a **Person**.


### <a name="add-faces-to-persons"></a>Incorporación de caras a objetos Person

Una vez que tenga el identificador del objeto **Person** de la llamada a Create Person, puede agregar hasta 248 imágenes de caras a un objeto **Person** por cada modelo de reconocimiento. Especifique el modelo de reconocimiento (y, opcionalmente, el modelo de detección) que se usará en la llamada, ya que los datos de cada modelo de reconocimiento se procesarán por separado dentro de la estructura **PersonDirectory**.

Los modelos de reconocimiento admitidos actualmente son:
* `Recognition_02`
* `Recognition_03`
* `Recognition_04`

Además, si la imagen contiene varias caras, deberá especificar el rectángulo delimitador para la cara que es el destino deseado. El código siguiente agrega caras a un objeto **Person**.

```csharp
var client = new HttpClient();

// Request headers
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "{subscription key}");

// Optional query strings for more fine grained face control
var queryString = "userData={userDefinedData}&targetFace={left,top,width,height}&detectionModel={detectionModel}";
var uri = "https://{endpoint}/face/v1.0-preview/persons/{personId}/recognitionModels/{recognitionModel}/persistedFaces?" + queryString;

HttpResponseMessage response;

// Request body
var body = new Dictionary<string, object>();
body.Add("url", "{image url}");
byte[] byteData = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(body));

using (var content = new ByteArrayContent(byteData))
{
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    response = await client.PostAsync(uri, content);
}
```

Después de la llamada a Add Faces, los datos de la cara se procesarán de forma asincrónica y deberá esperar a que la operación finalice correctamente de la misma manera que antes.

Cuando haya finalizado la operación para la adición de caras, los datos estarán listos para las llamadas a Identify.

## <a name="create-and-update-a-dynamicpersongroup"></a>Creación y actualización de un elemento **DynamicPersonGroup**

Los elementos **DynamicPersonGroup** son colecciones de referencias a objetos **Person** dentro de una estructura **PersonDirectory**; se usan para crear subconjuntos del directorio. Un uso común es cuando quiere obtener menos falsos positivos y una mayor precisión en una operación de Identify al limitar el ámbito solo a los objetos **Person** que se espera que coincidan. Los casos de uso prácticos incluyen directorios para el acceso específico a edificios en un campus u organización más grande. El directorio de la organización podría contener cinco millones de personas, pero solo necesita buscar 800 personas en un edificio concreto. De modo que tendría que crear un elemento **DynamicPersonGroup** que contenga a esas personas específicas. 

Si ha usado un elemento **PersonGroup** antes, tome nota de las dos diferencias principales: 
* Cada objeto **Person** dentro de un elemento **DynamicPersonGroup** es una referencia al objeto **Person** real en la estructura **PersonDirectory**, lo que significa que no es necesario volver a crear un objeto **Person** en cada grupo.
* Como se mencionó en las secciones anteriores, no es necesario realizar llamadas a Train, ya que los datos faciales se procesan automáticamente en el nivel de directorio.

### <a name="create-the-group"></a>Creación del grupo

Para crear un elemento **DynamicPersonGroup**, debe proporcionar un identificador de grupo con caracteres alfanuméricos o guiones. Dicho identificador funcionará como identificador único para todos los fines del grupo.

Hay dos maneras de inicializar una colección de grupos. Al inicio puede crear un grupo vacío y rellenarlo más adelante:

```csharp
var client = new HttpClient();

// Request headers
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "{subscription key}");

var uri = "https://{endpoint}/face/v1.0-preview/dynamicpersongroups/{dynamicPersonGroupId}";

HttpResponseMessage response;

// Request body
var body = new Dictionary<string, object>();
body.Add("name", "Example DynamicPersonGroup");
body.Add("userData", "User defined data");
byte[] byteData = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(body));

using (var content = new ByteArrayContent(byteData))
{
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    response = await client.PutAsync(uri, content);
}
```

Este proceso es inmediato y no es necesario esperar a que las operaciones asincrónicas finalicen correctamente.

También puede crearlo con un conjunto de identificadores de objeto **Person** para contener esas referencias desde el principio; para ello, especifique el conjunto en el argumento _AddPersonIds_:

```csharp
var client = new HttpClient();

// Request headers
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "{subscription key}");

var uri = "https://{endpoint}/face/v1.0-preview/dynamicpersongroups/{dynamicPersonGroupId}";

HttpResponseMessage response;

// Request body
var body = new Dictionary<string, object>();
body.Add("name", "Example DynamicPersonGroup");
body.Add("userData", "User defined data");
body.Add("addPersonIds", new List<string>{"{guid1}", "{guid2}", …});
byte[] byteData = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(body));

using (var content = new ByteArrayContent(byteData))
{
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    response = await client.PutAsync(uri, content);

    // Async operation location to query the completion status from
    var operationLocation = response.Headers.Get("Operation-Location");
}
```

> [!NOTE]
> En cuanto se devuelva la llamada, el elemento **DynamicPersonGroup** creado estará listo para usarse en una llamada a Identify, con las referencias de objetos **Person** proporcionadas en el proceso. Por otro lado, el estado de finalización del identificador de operación devuelto indica el estado de actualización de la relación entre la persona y el grupo.

### <a name="update-the-dynamicpersongroup"></a>Actualización del elemento DynamicPersonGroup

Después de la creación inicial, puede agregar y quitar referencias a objetos **Person** del elemento **DynamicPersonGroup** con Update Dynamic Person Group API. Para agregar objetos **Person** al grupo, enumere los identificadores de **Person** en el argumento _addPersonsIds_. Para quitar objetos **Person**, enumérelos en el argumento _removePersonIds_. Tanto la adición como la eliminación se pueden realizar en una sola llamada:

```csharp
var client = new HttpClient();

// Request headers
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "{subscription key}");

var uri = "https://{endpoint}/face/v1.0-preview/dynamicpersongroups/{dynamicPersonGroupId}";

HttpResponseMessage response;

// Request body
var body = new Dictionary<string, object>();
body.Add("name", "Example Dynamic Person Group updated");
body.Add("userData", "User defined data updated");
body.Add("addPersonIds", new List<string>{"{guid1}", "{guid2}", …});
body.Add("removePersonIds", new List<string>{"{guid1}", "{guid2}", …});
byte[] byteData = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(body));

using (var content = new ByteArrayContent(byteData))
{
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    response = await client.PatchAsync(uri, content);

    // Async operation location to query the completion status from
    var operationLocation = response.Headers.Get("Operation-Location");
}
```

Una vez que se devuelve la llamada, las actualizaciones de la colección se reflejarán al consultar el grupo. Al igual que con la API de creación, la operación devuelta indica el estado de actualización de la relación entre la persona y el grupo para cualquier objeto **Person** implicada en la actualización. No es necesario esperar a que finalice la operación antes de realizar más llamadas de actualización al grupo.

## <a name="identify-faces-in-a-persondirectory"></a>Identificación de caras en un objeto PersonDirectory

La manera más común de usar los datos faciales en una estructura **PersonDirectory** es comparar los objetos **Person** inscritos con una cara específica e identificar a cuál de los candidatos es más probable que pertenezca. Se pueden proporcionar varias caras en la solicitud y cada una de ellas recibirá su propio conjunto de resultados de comparación en la respuesta.

En la estructura **PersonDirectory**, hay tres tipos de ámbitos con los que se puede identificar cada cara:

### <a name="scenario-1-identify-against-a-dynamicpersongroup"></a>Escenario 1: Identificación con DynamicPersonGroup
 
Si se especifica la propiedad _dynamicPersonGroupId_ en la solicitud, se compara la cara con cada objeto **Person** al que se hace referencia en el grupo. Solo se puede identificar un elemento **DynamicPersonGroup** único en una llamada. 

```csharp
var client = new HttpClient();

// Request headers
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "{subscription key}");

// Optional query strings for more fine grained face control
var uri = "https://{endpoint}/face/v1.0-preview/identify";

HttpResponseMessage response;

// Request body
var body = new Dictionary<string, object>();
body.Add("faceIds", new List<string>{"{guid1}", "{guid2}", …});
body.Add("dynamicPersonGroupId", "{dynamicPersonGroupIdToIdentifyIn}");
byte[] byteData = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(body));

using (var content = new ByteArrayContent(byteData))
{
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    response = await client.PostAsync(uri, content);
}
```

### <a name="scenario-2-identify-against-a-specific-list-of-persons"></a>Escenario 2: Identificación en una lista específica de personas

También puede especificar una lista de identificadores de **Person** en la propiedad _personIds_ para comparar la cara con cada uno de ellos.

```csharp
var client = new HttpClient();

// Request headers
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "{subscription key}");
            
var uri = "https://{endpoint}/face/v1.0-preview/identify";

HttpResponseMessage response;

// Request body
var body = new Dictionary<string, object>();
body.Add("faceIds", new List<string>{"{guid1}", "{guid2}", …});
body.Add("personIds", new List<string>{"{guid1}", "{guid2}", …});
byte[] byteData = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(body));

using (var content = new ByteArrayContent(byteData))
{
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    response = await client.PostAsync(uri, content);
}
```

### <a name="scenario-3-identify-against-the-entire-persondirectory"></a>Escenario 3: Identificación en toda la estructura **PersonDirectory**

Si se proporciona un único asterisco en la propiedad _personIds_ de la solicitud, la cara se compara con todos los elementos **Person** inscritos en **PersonDirectory**. 
 

```csharp
var client = new HttpClient();

// Request headers
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "{subscription key}");
            
var uri = "https://{endpoint}/face/v1.0-preview/identify";

HttpResponseMessage response;

// Request body
var body = new Dictionary<string, object>();
body.Add("faceIds", new List<string>{"{guid1}", "{guid2}", …});
body.Add("personIds", new List<string>{"{guid1}", "{guid2}", …});
byte[] byteData = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(body));

using (var content = new ByteArrayContent(byteData))
{
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    response = await client.PostAsync(uri, content);
}
```
En los tres escenarios, la identificación solo compara la cara entrante con las caras cuya llamada a AddPersonFace se ha devuelto con una respuesta "succeeded".

## <a name="verify-faces-against-persons-in-the-persondirectory"></a>Comprobación de caras con personas en la estructura **PersonDirectory**

Con un identificador de cara devuelto por una llamada de detección, puede comprobar si la cara pertenece a un objeto **Person** específico inscrito en la estructura **PersonDirectory**. Especifique el objeto **Person** mediante la propiedad _personId_.

```csharp
var client = new HttpClient();

// Request headers
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "{subscription key}");

var uri = "https://{endpoint}/face/v1.0-preview/verify";

HttpResponseMessage response;

// Request body
var body = new Dictionary<string, object>();
body.Add("faceId", "{guid1}");
body.Add("personId", "{guid1}");
var jsSerializer = new JavaScriptSerializer();
byte[] byteData = Encoding.UTF8.GetBytes(jsSerializer.Serialize(body));

using (var content = new ByteArrayContent(byteData))
{
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    response = await client.PostAsync(uri, content);
}
```

La respuesta contendrá un valor booleano que indicará si el servicio considera que la nueva cara pertenece al mismo objeto **Person** y una puntuación de confianza para la predicción.

## <a name="next-steps"></a>Pasos siguientes

En esta guía, ha aprendido a usar la estructura **PersonDirectory** para almacenar datos de caras y personas para la aplicación de Face. A continuación, aprenderá los procedimientos recomendados para agregar los datos faciales de los usuarios.

* [Procedimientos recomendados para agregar usuarios](../enrollment-overview.md)