---
title: API y SDK de Azure Digital Twins
titleSuffix: Azure Digital Twins
description: Descubra las opciones de SDK y API de Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 04/30/2021
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 801cb9ad60d6dae0c8f69cb3ab004f94a2badacf
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108778413"
---
# <a name="azure-digital-twins-apis-and-sdks"></a>API y SDK de Azure Digital Twins

Azure Digital Twins incluye las **API de plano de control** y las **API de plano de datos** para administrar su instancia y sus elementos. 
* Las API de plano de control son las API de [Azure Resource Manager (ARM)](../azure-resource-manager/management/overview.md) y cubren las operaciones de administración de recursos como la creación y eliminación de la instancia. 
* Las API de plano de datos son las API de Azure Digital Twins y se usan para operaciones de administración de datos, como la administración de modelos, gemelos y el grafo.

En este artículo se proporciona información general sobre las API disponibles y los métodos para interactuar con ellas. Puede usar las API REST directamente con las instancias de Swagger asociadas (mediante una herramienta como [Postman](how-to-use-postman.md)) o mediante un SDK.

## <a name="overview-control-plane-apis"></a>Información general: API de plano de control

Las API del plano de control son las API de [ARM](../azure-resource-manager/management/overview.md) que se usan para administrar la instancia de Azure Digital Twins como un todo, por lo que cubren operaciones como la creación o eliminación de toda la instancia. También se usarán para crear y eliminar puntos de conexión.

La versión más reciente de la API de plano de control es _**2020-12-01**_.

Para usar las API de plano de control:
* Para llamar a las API directamente, haga referencia a la carpeta de Swagger más reciente en la [repositorio de Swagger del plano de control](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/digitaltwins/resource-manager/Microsoft.DigitalTwins/stable). Esta carpeta también incluye una carpeta de ejemplos que muestran el uso.
* Actualmente puede acceder a los SDK para las API de control en...
  - [.NET (C#)](https://www.nuget.org/packages/Microsoft.Azure.Management.DigitalTwins/) ([referencia [generada automáticamente]](/dotnet/api/overview/azure/digitaltwins/management)) ([origen](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Microsoft.Azure.Management.DigitalTwins))
  - [Java](https://search.maven.org/search?q=a:azure-mgmt-digitaltwins) ([referencia [generada automáticamente]](/java/api/overview/azure/digitaltwins)) ([origen](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/digitaltwins))
  - [JavaScript](https://www.npmjs.com/package/@azure/arm-digitaltwins) ([código fuente](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/digitaltwins/arm-digitaltwins))
  - [Python](https://pypi.org/project/azure-mgmt-digitaltwins/) ([código fuente](https://github.com/Azure/azure-sdk-for-python/tree/release/v3/sdk/digitaltwins/azure-mgmt-digitaltwins))
  - [Go](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/services/digitaltwins/mgmt) ([código fuente](https://github.com/Azure/azure-sdk-for-go/tree/master/services/digitaltwins/mgmt))

También puede ejercitar las API de plano de control interactuando con Azure Digital Twins mediante [Azure Portal](https://portal.azure.com) y la [CLI](concepts-cli.md).

## <a name="overview-data-plane-apis"></a>Información general: API de plano de datos

Las API de plano de datos son las API de Azure Digital Twins que se usan para administrar los elementos de la instancia de Azure Digital Twins. Incluyen operaciones como la creación de rutas, la carga de modelos, la creación de relaciones y la administración de gemelos. Se pueden dividir ampliamente en las siguientes categorías:
* **DigitalTwinModels**: la categoría DigitalTwinModels contiene las API para administrar los [modelos](concepts-models.md) en una instancia de Azure Digital Twins. Las actividades de administración incluyen la carga, validación, recuperación y eliminación de modelos creados en DTDL.
* **DigitalTwins**: la categoría DigitalTwins contiene las API que permiten a los desarrolladores crear, modificar y eliminar [gemelos digitales](concepts-twins-graph.md) y sus relaciones en una instancia de Azure Digital Twins.
* **Query**: la categoría Query permite a los desarrolladores [buscar conjuntos de gemelos digitales en el grafo de gemelos](how-to-query-graph.md) entre las relaciones.
* **Event Routes**: la categoría Event Routes contiene las API para [enrutar datos](concepts-route-events.md), en el sistema y a los servicios de bajada.

La versión más actual de la API de plano de datos es _**2020-10-31**_.

Para usar las API de plano de datos:
* Para llamar directamente a las API...
   - haga referencia a la carpeta de Swagger más reciente en el [repositorio de Swagger del plano de datos](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/digitaltwins/data-plane/Microsoft.DigitalTwins). Esta carpeta también incluye una carpeta de ejemplos que muestran el uso. 
   - consulte la [documentación de referencia de la API](/rest/api/azure-digitaltwins/).
* Puede usar el **SDK de .NET (C#)** . Para usar el SDK de .NET...
   - puede ver y agregar el paquete desde NuGet: [Azure.DigitalTwins.Core](https://www.nuget.org/packages/Azure.DigitalTwins.Core). 
   - puede ver la [documentación de referencia del SDK](/dotnet/api/overview/azure/digitaltwins/client).
   - puede encontrar el código fuente del SDK, incluida una carpeta de ejemplos, en GitHub: [Biblioteca de cliente de Digital Twins de Azure IoT para .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core). 
   - puede ver información detallada y ejemplos de uso si continúa con la sección [SDK de .NET (C#) (plano de datos)](#net-c-sdk-data-plane) de este artículo.
* Puede usar el **SDK de Java**. Para usar el SDK de Java…
   - puede ver e instalar el paquete desde Maven: [`com.azure:azure-digitaltwins-core`](https://search.maven.org/artifact/com.azure/azure-digitaltwins-core/1.0.0/jar)
   - puede ver la [documentación de referencia del SDK](/java/api/overview/azure/digitaltwins/client)
   - puede encontrar el código fuente del SDK en GitHub: [Biblioteca cliente de Digital Twins de Azure IoT para Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/digitaltwins/azure-digitaltwins-core)
* Puede usar el **SDK de JavaScript**. Para usar el SDK de JavaScript...
   - puede ver e instalar el paquete desde npm: [Biblioteca cliente de Azure Digital Twins Core para JavaScript](https://www.npmjs.com/package/@azure/digital-twins-core).
   - puede ver la [documentación de referencia del SDK](/javascript/api/@azure/digital-twins-core/).
   - puede encontrar el código fuente del SDK en GitHub: [Biblioteca cliente de Azure Digital Twins Core para JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/digitaltwins/digital-twins-core).
* Puede usar el **SDK para Python**. Para usar el SDK para Python...
   - puede ver e instalar el paquete desde PyPi: [Biblioteca cliente de Azure Digital Twins Core para Python](https://pypi.org/project/azure-digitaltwins-core/).
   - puede ver la [documentación de referencia del SDK](/python/api/azure-digitaltwins-core/azure.digitaltwins.core).
   - puede encontrar el código fuente del SDK en GitHub: [Biblioteca cliente de Azure Digital Twins Core para Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/digitaltwins/azure-digitaltwins-core)

También puede probar las API de plano de fecha interactuando con Azure Digital Twins a través de la [CLI](concepts-cli.md).

## <a name="net-c-sdk-data-plane"></a>SDK de .NET (C#) (plano de datos)

El SDK de .NET (C#) para Azure Digital Twins forma parte del SDK de Azure para .NET. Es de código abierto y se basa en las API del plano de datos de Azure Digital Twins.

> [!NOTE]
> Para más información sobre el diseño del SDK, consulte los [principios de diseño generales para los SDK de Azure](https://azure.github.io/azure-sdk/general_introduction.html) y las [directrices de diseño de .NET](https://azure.github.io/azure-sdk/dotnet_introduction.html) específicas.

Para usar el SDK, incluya el paquete NuGet **Azure.DigitalTwins.Core** con el proyecto. También necesitará la versión más reciente del paquete **Azure.Identity**. En Visual Studio, puede agregar estos paquetes con el administrador de paquetes NuGet (al que se accede a través de *Herramientas > Administrador de paquetes NuGet > Administrar paquetes NuGet para la solución*). O bien, puede usar la herramienta de línea de comandos de .NET con los comandos que se encuentran en los siguientes vínculos de paquetes NuGet para agregarlos al proyecto:
* [Azure.DigitalTwins.Core](https://www.nuget.org/packages/Azure.DigitalTwins.Core). Este es el paquete del [SDK de Azure Digital Twins para .NET](/dotnet/api/overview/azure/digitaltwins/client). 
* [Azure.Identity](https://www.nuget.org/packages/Azure.Identity). Esta biblioteca proporciona herramientas para facilitar la autenticación en Azure.

Para obtener un tutorial detallado sobre el uso de las API en la práctica, consulte [Tutorial: Programación de una aplicación cliente](tutorial-code.md). 

### <a name="net-sdk-usage-examples"></a>Ejemplos de uso del SDK de .NET

Estos son algunos ejemplos de código que ilustran el uso del SDK de .NET.

Autenticación en el servicio:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/authentication.cs" id="DefaultAzureCredential_basic":::

[!INCLUDE [Azure Digital Twins: local credentials note](../../includes/digital-twins-local-credentials-note.md)] 

Carga de un modelo:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/model_operations.cs" id="CreateModel":::

Enumeración de modelos:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/model_operations.cs" id="GetModels":::

Creación de gemelos

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="CreateTwin_withHelper":::

Consultas de gemelos y recorrido en bucle de los resultados:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/queries.cs" id="FullQuerySample":::

Consulte [Tutorial: Programación de una aplicación cliente](tutorial-code.md) para obtener un tutorial de este código de aplicación de ejemplo. 

También puede encontrar más ejemplos en el [repositorio de GitHub para el SDK de .NET (C#)](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core/samples).

#### <a name="serialization-helpers"></a>Asistentes de serialización

Las aplicaciones auxiliares de serialización son funciones auxiliares disponibles en el SDK para crear o deserializar rápidamente datos gemelos para el acceso a información básica. Dado que los métodos principales del SDK devuelven los datos gemelos como JSON de forma predeterminada, puede resultar útil usar estas clases auxiliares para reducir aún más los datos gemelos.

Las clases de asistente disponibles son:
* `BasicDigitalTwin`: representa genéricamente los datos principales de un gemelo digital.
* `BasicDigitalTwinComponent`: representa genéricamente un componente en las propiedades `Contents` de un elemento `BasicDigitalTwin`.
* `BasicRelationship`: representa genéricamente los datos principales de una relación.
* `DigitalTwinsJsonPropertyName`: contiene las constantes de cadena que se usan en la serialización y deserialización JSON para tipos de gemelos digitales personalizados.

##### <a name="deserialize-a-digital-twin"></a>Deserialización de un gemelo digital

Siempre puede deserializar los datos gemelos mediante la biblioteca JSON que prefiera, como `System.Text.Json` o `Newtonsoft.Json`. Las clases auxiliares hacen más cómodo el acceso básico a un gemelo.

La clase de asistente de `BasicDigitalTwin` también proporciona acceso a las propiedades definidas en el gemelo, a través de un `Dictionary<string, object>`. Para enumerar las propiedades del gemelo, puede usar:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="GetTwin":::

> [!NOTE]
> `BasicDigitalTwin` utiliza atributos `System.Text.Json`. Para usar `BasicDigitalTwin` con el elemento [DigitalTwinsClient](/dotnet/api/azure.digitaltwins.core.digitaltwinsclient?view=azure-dotnet&preserve-view=true), debe inicializar el cliente con el constructor predeterminado, o bien, si desea personalizar la opción del serializador, use [JsonObjectSerializer](/dotnet/api/azure.core.serialization.jsonobjectserializer?view=azure-dotnet&preserve-view=true).

##### <a name="create-a-digital-twin"></a>Creación de un gemelo digital

Con la clase `BasicDigitalTwin`, puede preparar los datos para crear una instancia gemela:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="CreateTwin_withHelper":::

El código anterior es equivalente a la siguiente variante "manual":

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="CreateTwin_noHelper":::

##### <a name="deserialize-a-relationship"></a>Deserialización de una relación

Siempre puede deserializar los datos de la relación en un tipo de su elección. Para obtener acceso básico a una relación, utilice el tipo `BasicRelationship`.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="GetRelationshipsCall":::

La clase de asistente `BasicRelationship` también proporciona acceso a las propiedades definidas en la relación, mediante un elemento `IDictionary<string, object>`. Para enumerar las propiedades, puede usar:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_other.cs" id="ListRelationshipProperties":::

##### <a name="create-a-relationship"></a>Creación de una relación

Con la clase `BasicRelationship`, también puede preparar los datos para crear relaciones en una instancia gemela:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_other.cs" id="CreateRelationship_short":::

##### <a name="create-a-patch-for-twin-update"></a>Creación de una revisión para la actualización gemela

Las llamadas de actualización de los gemelos y las relaciones usan la estructura de [revisión de JSON](http://jsonpatch.com/). Para crear listas de operaciones de revisión de JSON, puede usar `JsonPatchDocument`, tal como se muestra a continuación.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="UpdateTwin":::

## <a name="general-apisdk-usage-notes"></a>Notas generales sobre el uso de la API o del SDK

> [!NOTE]
> Tenga en cuenta que Azure Digital Twins no admite actualmente el **uso compartido de recursos entre orígenes (CORS)** . Para más información sobre las estrategias de impacto y resolución, consulte la sección [Uso compartido de recursos entre orígenes (CORS)](concepts-security.md#cross-origin-resource-sharing-cors)  de *Conceptos: Seguridad para las soluciones de Azure Digital Twins*.

En la lista siguiente se proporcionan detalles adicionales y directrices generales para usar las API y los SDK.

* Puede usar una herramienta de prueba de REST de HTTP como Postman para realizar llamadas directas a las API de Azure Digital Twins. Para más información sobre este proceso, consulte [Procedimiento: Realización de solicitudes con Postman](how-to-use-postman.md).
* Para usar el SDK, cree una instancia de la clase `DigitalTwinsClient`. El constructor requiere credenciales que se pueden obtener con una variedad de métodos de autenticación en el paquete de `Azure.Identity`. Para más información sobre `Azure.Identity`, consulte la [documentación del espacio de nombres](/dotnet/api/azure.identity). 
* Es posible que encuentre útil `InteractiveBrowserCredential` al comenzar, pero hay muchas otras opciones, incluidas las credenciales para la [identidad administrada](/dotnet/api/azure.identity.interactivebrowsercredential), que probablemente va a usar para autenticar las [funciones de Azure configuradas con MSI](../app-service/overview-managed-identity.md?tabs=dotnet) en Azure Digital Twins. Para más información sobre `InteractiveBrowserCredential`, consulte la [documentación de la clase](/dotnet/api/azure.identity.interactivebrowsercredential).
* Las solicitudes a las API de Azure Digital Twins requieren un usuario o una entidad de servicio que forme parte del mismo inquilino de [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD) donde reside la instancia de Azure Digital Twins. Para evitar exámenes malintencionados de los puntos de conexión de Azure Digital Twins, las solicitudes con tokens de acceso desde fuera del inquilino de origen recibirán un mensaje de error "404 No se ha encontrado el subdominio". Este error se devolverá *incluso si* al usuario o a la entidad de servicio se le ha concedido un rol de propietario de datos o de lector de datos de Azure Digital Twins mediante la colaboración de [Azure AD B2B](../active-directory/external-identities/what-is-b2b.md). Para obtener información sobre cómo lograr el acceso a través de varios inquilinos, consulte [Procedimiento: Escribir código de autenticación de aplicación](how-to-authenticate-client.md#authenticate-across-tenants).
* Todas las llamadas de API de servicio se exponen como funciones miembro en la clase `DigitalTwinsClient`.
* Todas las funciones de servicio se encuentran en versiones sincrónicas y asincrónicas.
* Todas las funciones de servicio inician una excepción en cualquier estado de devolución de 400 o superior. Asegúrese de ajustar las llamadas en una sección `try` y de capturar al menos `RequestFailedExceptions`. Para más información sobre este tipo de excepción, consulte [este artículo](/dotnet/api/azure.requestfailedexception).
* La mayoría de los métodos de servicio devuelven `Response<T>` o (`Task<Response<T>>` para las llamadas asincrónicas), donde `T` es la clase de objeto de devolución para la llamada de servicio. La clase [`Response`](/dotnet/api/azure.response-1) encapsula la devolución del servicio y presenta los valores devueltos en su campo `Value`.  
* Los métodos de servicio con los resultados paginados devuelven `Pageable<T>` o `AsyncPageable<T>` como resultados. Para más información sobre la clase `Pageable<T>`, consulte [este artículo](/dotnet/api/azure.pageable-1); para más información sobre `AsyncPageable<T>`, consulte [este artículo](/dotnet/api/azure.asyncpageable-1).
* Puede recorrer en iteración los resultados paginados mediante un bucle `await foreach`. Para más información sobre este proceso, consulte [este artículo](/archive/msdn-magazine/2019/november/csharp-iterating-with-async-enumerables-in-csharp-8).
* El SDK subyacente es `Azure.Core`. Consulte la [documentación del espacio de nombres de Azure](/dotnet/api/azure) como referencia sobre los tipos y la infraestructura del SDK.


Los métodos de servicio devuelven objetos fuertemente tipados siempre que sea posible. Sin embargo, dado que Azure Digital Twins se basa en los modelos configurados de forma personalizada por el usuario en tiempo de ejecución (a través de los modelos DTDL cargados en el servicio), muchas API de servicio toman y devuelven datos gemelos en formato JSON.

## <a name="monitor-api-metrics"></a>Supervisión de las métricas de API

Las métricas de API, como las solicitudes, la latencia y la tasa de errores, se pueden ver en [Azure Portal](https://portal.azure.com/). 

En la página principal del portal, busque la instancia de Azure Digital Twins para ver sus detalles. Seleccione la opción **Métricas** en el menú de la instancia de Azure Digital Twins para abrir la página *Métricas*.

:::image type="content" source="media/troubleshoot-metrics/azure-digital-twins-metrics.png" alt-text="Captura de pantalla que muestra la página de métricas de Azure Digital Twins":::

Desde aquí, puede ver las métricas de la instancia y crear vistas personalizadas.

## <a name="next-steps"></a>Pasos siguientes

Vea cómo se hacen solicitudes directas a las API mediante Postman:
* [Procedimiento: Realización de solicitudes con Postman](how-to-use-postman.md)

O bien, practique el uso del SDK de .NET mediante la creación de una aplicación cliente con este tutorial:
* [Tutorial: Programación de una aplicación cliente](tutorial-code.md)
