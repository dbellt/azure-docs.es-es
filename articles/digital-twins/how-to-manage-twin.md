---
title: Administración de Digital Twins
titleSuffix: Azure Digital Twins
description: Vea cómo recuperar, actualizar y eliminar relaciones y gemelos individuales.
author: baanders
ms.author: baanders
ms.date: 10/21/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 68127a7603db2d408ed1bbdb244f49de4eb21031
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108208498"
---
# <a name="manage-digital-twins"></a>Administración de Digital Twins

Las entidades de su entorno se representan mediante [gemelos digitales](concepts-twins-graph.md). La administración de los gemelos digitales puede incluir las operaciones de creación, modificación y eliminación. Para realizar estas operaciones, puede usar las [API de DigitalTwins](/rest/api/digital-twins/dataplane/twins), el [SDK de .NET ( C# )](/dotnet/api/overview/azure/digitaltwins/client) o la [CLI de Azure Digital Twins](how-to-use-cli.md).

Este artículo se centra en la administración de gemelos digitales; para trabajar con relaciones y con el [grafo de gemelos](concepts-twins-graph.md) en conjunto, consulte [Procedimiento: Administración del grafo de gemelos con relaciones](how-to-manage-graph.md).

> [!TIP]
> Todas las funciones del SDK cuentan con versiones sincrónicas y asincrónicas.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [digital-twins-prereq-instance.md](../../includes/digital-twins-prereq-instance.md)]

## <a name="ways-to-manage-twins"></a>Formas de administrar gemelos

[!INCLUDE [digital-twins-ways-to-manage.md](../../includes/digital-twins-ways-to-manage.md)]

## <a name="create-a-digital-twin"></a>Creación de un gemelo digital

Para crear un gemelo, use el método `CreateOrReplaceDigitalTwinAsync()` en el cliente de servicio de la siguiente manera:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="CreateTwinCall":::

Para crear un gemelo digital, debe proporcionar lo siguiente:
* El identificador deseado para el gemelo digital.
* El [modelo](concepts-models.md) que quiere usar.

Opcionalmente, puede proporcionar los valores iniciales de todas las propiedades del gemelo digital. Las propiedades se tratan como opcionales y se pueden establecer más adelante, pero **no aparecerán como parte de un gemelo hasta que se hayan establecido.**

>[!NOTE]
>Aunque no es necesario inicializar las propiedades del gemelo, los [componentes](concepts-models.md#elements-of-a-model) del gemelo **deben** establecerse cuando se crea este. Aunque pueden ser objetos vacíos, los componentes en sí deben existir.

El modelo y los valores iniciales de las propiedades se proporcionan mediante el parámetro `initData`, que es una cadena JSON que contiene los datos pertinentes. Para más información sobre cómo estructurar este objeto, vaya a la sección siguiente.

> [!TIP]
> Después de crear o actualizar un gemelo, puede haber una latencia de hasta 10 segundos antes de que los cambios se reflejen en las [consultas](how-to-query-graph.md). La API `GetDigitalTwin` (que se describe [más adelante en este artículo](#get-data-for-a-digital-twin)) no experimenta este retraso, por lo que debe usar la llamada a la API en lugar de realizar una consulta para ver los gemelos recién creados si necesita una respuesta instantánea. 

### <a name="initialize-model-and-properties"></a>Inicialización del modelo y las propiedades

Puede inicializar las propiedades de un gemelo en el momento en que este se crea. 

La API de creación de gemelos acepta un objeto serializado en una descripción JSON válida de las propiedades gemelas. Consulte [Conceptos: Gemelos digitales y el grafo de gemelos](concepts-twins-graph.md) para obtener una descripción del formato JSON de un gemelo. 

En primer lugar, puede crear un objeto de datos para representar el gemelo y los datos de sus propiedades. Puede crear un objeto de parámetro manualmente o mediante una clase auxiliar proporcionada. A continuación se muestra un ejemplo de cada opción.

#### <a name="create-twins-using-manually-created-data"></a>Creación de gemelos con datos creados manualmente

Sin el uso de ninguna clase auxiliar personalizada, puede representar las propiedades de un gemelo en un elemento `Dictionary<string, object>`, donde `string` es el nombre de la propiedad y `object` es un objeto que representa la propiedad y su valor.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="CreateTwin_noHelper":::

#### <a name="create-twins-with-the-helper-class"></a>Creación de gemelos con la clase auxiliar

La clase auxiliar `BasicDigitalTwin` permite almacenar los campos de propiedades directamente en un objeto "gemelo". Aún puede crear la lista de propiedades mediante un elemento `Dictionary<string, object>`, que luego se puede agregar al objeto gemelo como su `CustomProperties` directamente.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="CreateTwin_withHelper":::

>[!NOTE]
> Los objetos `BasicDigitalTwin` contienen un campo `Id`. Puede dejar este campo vacío, pero si agrega un valor de id., debe coincidir con el parámetro de id. pasado a la llamada `CreateOrReplaceDigitalTwinAsync()`. Por ejemplo:
>
>```csharp
>twin.Id = "myRoomId";
>```

## <a name="get-data-for-a-digital-twin"></a>Obtención de datos para un gemelo digital

Puede acceder a los detalles de cualquier gemelo digital mediante una llamada al método `GetDigitalTwin()`, como se describe a continuación:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="GetTwinCall":::

Esta llamada devuelve los datos del gemelo en forma de un tipo de objeto fuertemente tipado, como `BasicDigitalTwin`. `BasicDigitalTwin` es una clase auxiliar de serialización que se incluye con el SDK, que devolverá los metadatos y las propiedades de gemelos principales en formato analizado previamente. Este es un ejemplo de cómo se usa para ver los detalles del gemelo:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="GetTwin" highlight="2":::

Solo se devuelven las propiedades que se han establecido al menos una vez cuando se recupera un gemelo con el método `GetDigitalTwin()`.

>[!TIP]
>El `displayName` para un gemelo es parte de sus metadatos del modelo, por lo que no se mostrará al obtener los datos de la instancia gemela. Para ver este valor, puede [recuperarlo del modelo](how-to-manage-model.md#retrieve-models).

Para recuperar varios gemelos mediante una única llamada API, consulte los ejemplos de la API de consulta en [Procedimiento: consultar el grafo de gemelos](how-to-query-graph.md).

Tenga en cuenta el siguiente modelo, escrito en el [lenguaje de definición de gemelos digitales (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl/tree/master/DTDL), que define un objeto *Moon*:

:::code language="json" source="~/digital-twins-docs-samples/models/Moon.json":::

El resultado de llamar a `object result = await client.GetDigitalTwinAsync("my-moon");` en un gemelo de tipo *Moon* puede ser similar al siguiente:

```json
{
  "$dtId": "myMoon-001",
  "$etag": "W/\"e59ce8f5-03c0-4356-aea9-249ecbdc07f9\"",
  "radius": 1737.1,
  "mass": 0.0734,
  "$metadata": {
    "$model": "dtmi:example:Moon;1",
    "radius": {
      "desiredValue": 1737.1,
      "desiredVersion": 5,
      "ackVersion": 4,
      "ackCode": 200,
      "ackDescription": "OK"
    },
    "mass": {
      "desiredValue": 0.0734,
      "desiredVersion": 8,
      "ackVersion": 8,
      "ackCode": 200,
      "ackDescription": "OK"
    }
  }
}
```

Las propiedades definidas del gemelo digital se devuelven como propiedades de nivel superior en el gemelo digital. Los metadatos o la información del sistema que no forman parte de la definición de DTDL se devuelven con un prefijo `$`. Las propiedades de metadatos incluyen los valores siguientes:
* `$dtId`: el identificador del gemelo digital en esta instancia de Azure Digital Twins
* `$etag`: un campo HTTP estándar asignado por el servidor web. Esto se actualiza a un nuevo valor cada vez que se actualiza el gemelo, lo que puede resultar útil para determinar si los datos del gemelo se han actualizado en el servidor desde una comprobación anterior. Puede usar `If-Match` para realizar las actualizaciones y eliminaciones que solo se completan si el valor ETag de la entidad coincide con el valor ETag proporcionado. Para obtener más información sobre estas operaciones, consulte la documentación de [DigitalTwins Update](/rest/api/digital-twins/dataplane/twins/digitaltwins_update) y [DigitalTwins Delete](/rest/api/digital-twins/dataplane/twins/digitaltwins_delete).
* `$metadata`: un conjunto de otras propiedades, entre las que se incluyen:
  - DTMI del modelo del gemelo digital.
  - Estado de sincronización de cada propiedad grabable. Esto es útil principalmente para los dispositivos, donde es posible que el servicio y el dispositivo tengan estados divergentes (por ejemplo, cuando un dispositivo está sin conexión). Actualmente, esta propiedad solo se aplica a dispositivos físicos conectados a IoT Hub. Los datos de la sección de metadatos permiten comprender el estado completo de una propiedad, así como las últimas marcas de tiempo modificadas. Para obtener más información sobre el estado de sincronización, consulte [este tutorial de IoT Hub](../iot-hub/tutorial-device-twins.md) sobre la sincronización del estado del dispositivo.
  - Metadatos específicos del servicio, como de IoT Hub o Azure Digital Twins. 

Para más información sobre las clases auxiliares de serialización como `BasicDigitalTwin`, consulte [Procedimiento: Uso de las API y los SDK de Azure Digital Twins](how-to-use-apis-sdks.md).

## <a name="view-all-digital-twins"></a>Visualización de todos los gemelos digitales

Para ver todos los gemelos digitales de la instancia, use una [consulta](how-to-query-graph.md). Puede ejecutar una consulta con las [API de consulta](/rest/api/digital-twins/dataplane/query) o los [comandos de la CLI](how-to-use-cli.md).

Este es el cuerpo de la consulta básica que devolverá una lista de todos los gemelos digitales en la instancia:

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="GetAllTwins":::

## <a name="update-a-digital-twin"></a>Actualización de un gemelo digital

Para actualizar las propiedades de un gemelo digital, escriba la información que quiere reemplazar en formato de [revisión de JSON](http://jsonpatch.com/). De esta manera, puede reemplazar varias propiedades a la vez. A continuación, pase el documento de revisión de JSON a un método `UpdateDigitalTwin()`:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="UpdateTwinCall":::

Una llamada de revisión puede actualizar tantas propiedades de un solo gemelo como se quiera (incluso todas ellas). Si necesita actualizar las propiedades en varios gemelos, necesitará una llamada de actualización independiente para cada uno.

> [!TIP]
> Después de crear o actualizar un gemelo, puede haber una latencia de hasta 10 segundos antes de que los cambios se reflejen en las [consultas](how-to-query-graph.md). La API de `GetDigitalTwin` (que se describe [anteriormente en este artículo](#get-data-for-a-digital-twin)) no experimenta este retraso, por lo que debe usar la llamada API en lugar de realizar una consulta para ver los gemelos recién actualizados si necesita una respuesta instantánea. 

Este es un ejemplo de código de revisión de JSON. En este documento se reemplazan los valores de las propiedades *mass* y *radius* del gemelo digital al que se aplica.

:::code language="json" source="~/digital-twins-docs-samples/models/patch.json":::

Puede crear revisiones mediante [JsonPatchDocument](/dotnet/api/azure.jsonpatchdocument) del SDK para .NET de Azure. A continuación se muestra un ejemplo:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="UpdateTwin":::

### <a name="update-properties-in-digital-twin-components"></a>Actualización de propiedades en componentes de gemelos digitales

Recuerde que un modelo puede contener componentes, lo que permite que esté formado por otros modelos. 

Para revisar las propiedades de los componentes de un gemelo digital, puede usar la sintaxis de ruta de acceso en la revisión de JSON:

:::code language="json" source="~/digital-twins-docs-samples/models/patch-component.json":::

### <a name="update-a-digital-twins-model"></a>Actualización de un modelo de gemelo digital

La función `UpdateDigitalTwin()` también puede usarse para migrar un gemelo digital a un modelo diferente. 

Por ejemplo, tenga en cuenta el siguiente documento de revisión de JSON que reemplaza el campo `$model` de los metadatos del gemelo digital:

:::code language="json" source="~/digital-twins-docs-samples/models/patch-model-1.json":::

Esta operación solo se realizará correctamente si el gemelo digital que la revisión está modificando se ajusta al nuevo modelo. 

Considere el ejemplo siguiente:
1. Imagine un gemelo digital con un modelo de *foo_old*. *foo_old* define una propiedad *mass* necesaria.
2. El nuevo modelo *foo_new* define una propiedad mass y agrega una nueva propiedad *temperature* necesaria.
3. Después de la revisión, el gemelo digital debe tener ambas propiedades mass y temperature. 

La revisión para esta situación debe actualizar tanto el modelo como la propiedad temperature del gemelo, de la siguiente manera:

:::code language="json" source="~/digital-twins-docs-samples/models/patch-model-2.json":::

### <a name="handle-conflicting-update-calls"></a>Control de las llamadas de actualización en conflicto

Azure Digital Twins garantiza que todas las solicitudes entrantes se procesan una tras otra. Esto significa que, incluso para el caso de que varias funciones intenten actualizar la misma propiedad en un gemelo al mismo tiempo, **no es necesario** que escriba código de bloqueo explícito para controlar el conflicto.

Este comportamiento se basa en cada gemelo. 

Por ejemplo, imagine un escenario en el que llegan estas tres llamadas al mismo tiempo: 
*   Escribir propiedad A en *Gemelo1*
*   Escribir propiedad B en *Gemelo1*
*   Escribir propiedad A en *Gemelo2*

Las dos llamadas que modifican *Gemelo1* se ejecutan una tras otra, y se generan mensajes para cada cambio. La llamada para modificar *Gemelo2* puede ejecutarse simultáneamente sin ningún conflicto, en cuanto llega.

## <a name="delete-a-digital-twin"></a>Eliminación de un gemelo digital

Puede eliminar gemelos con el método `DeleteDigitalTwin()`. Sin embargo, solo puede eliminar un gemelo si este no tiene más relaciones. Por lo tanto, elimine primero las relaciones de entrada y salida del gemelo.

Este es un ejemplo de código para eliminar gemelos y sus relaciones. La llamada al SDK `DeleteDigitalTwin` está resaltada para aclarar dónde se encuentra en el contexto de un ejemplo más amplio.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="DeleteTwin" highlight="7":::

### <a name="delete-all-digital-twins"></a>Eliminación de todos los gemelos digitales

Para obtener un ejemplo de cómo eliminar todos los gemelos a la vez, descargue la aplicación de ejemplo que se usa en el [Tutorial: Exploración de los conceptos básicos con una aplicación cliente de ejemplo](tutorial-command-line-app.md). El archivo *CommandLoop.cs* realiza esta acción en una función `CommandDeleteAllTwins()`.

## <a name="runnable-digital-twin-code-sample"></a>Ejemplo de código de gemelo digital ejecutable

Puede usar el ejemplo de código ejecutable siguiente para crear un gemelo, actualizar sus detalles y eliminar el gemelo. 

### <a name="set-up-the-runnable-sample"></a>Configuración del ejemplo ejecutable

En el fragmento de código se usa la definición de modelo [Room.json](https://github.com/Azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Room.json) de [Tutorial: Exploración de Azure Digital Twins con una aplicación cliente de ejemplo](tutorial-command-line-app.md). Puede usar este vínculo para ir directamente al archivo o descargarlo como parte del [proyecto de ejemplo completo de un extremo a otro](/samples/azure-samples/digital-twins-samples/digital-twins-samples/).

Antes de ejecutar el ejemplo, haga lo siguiente:
1. Descargue el archivo del modelo, colóquelo en el proyecto y reemplace el marcador de posición `<path-to>` en el código siguiente para indicar al programa dónde encontrarlos.
2. Reemplace el marcador de posición `<your-instance-hostname>` por el nombre de host de la instancia de Azure Digital Twins.
3. Agregue dos dependencias al proyecto ya que las necesitará para trabajar con Azure Digital Twins. El primero es el paquete para el [SDK de Azure Digital Twins para .NET](/dotnet/api/overview/azure/digitaltwins/client), el segundo proporciona herramientas para ayudar con la autenticación en Azure.

      ```cmd/sh
      dotnet add package Azure.DigitalTwins.Core
      dotnet add package Azure.Identity
      ```

También necesitará configurar las credenciales locales si desea ejecutar el ejemplo directamente. La siguiente sección le indicará cómo hacerlo.
[!INCLUDE [Azure Digital Twins: local credentials prereq (outer)](../../includes/digital-twins-local-credentials-outer.md)]

### <a name="run-the-sample"></a>Ejecución del ejemplo

Después de completar los pasos anteriores, puede ejecutar directamente el siguiente código de ejemplo.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs":::

Esta es la salida de consola del programa anterior: 

:::image type="content" source="./media/how-to-manage-twin/console-output-manage-twins.png" alt-text="Salida de consola que muestra que el gemelo se crea, se actualiza y se elimina" lightbox="./media/how-to-manage-twin/console-output-manage-twins.png":::

## <a name="next-steps"></a>Pasos siguientes

Consulte cómo crear y administrar relaciones entre los gemelos digitales:
* [Procedimiento: Administración del grafo de gemelos con relaciones](how-to-manage-graph.md)