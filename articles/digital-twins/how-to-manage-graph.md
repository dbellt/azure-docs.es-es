---
title: Administración del grafo de gemelos con relaciones
titleSuffix: Azure Digital Twins
description: Consulte cómo administrar un grafo de gemelos digitales conectándolos con relaciones.
author: baanders
ms.author: baanders
ms.date: 11/03/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 31c99fbac936ab6ec96d6e24bc824d9cb2069e0f
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110078785"
---
# <a name="manage-a-graph-of-digital-twins-using-relationships"></a>Administración de un grafo de gemelos digitales con relaciones

El corazón de Azure Digital Twins es el [grafo de gemelos](concepts-twins-graph.md) que representa todo el entorno. El grafo de gemelos se compone de gemelos digitales individuales conectados mediante **relaciones**. 

Una vez que tenga una [instancia de Azure Digital Twins](how-to-set-up-instance-portal.md) en funcionamiento y que haya configurado el código de [autenticación](how-to-authenticate-client.md) en la aplicación cliente, puede crear, modificar y eliminar gemelos digitales y sus relaciones en una instancia de Azure Digital Twins.

Este artículo está centrado en la administración de las relaciones y el grafo en conjunto; para trabajar con gemelos digitales individuales, consulte [Procedimiento: Administración de Digital Twins](how-to-manage-twin.md).

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [digital-twins-prereq-instance.md](../../includes/digital-twins-prereq-instance.md)]

[!INCLUDE [digital-twins-developer-interfaces.md](../../includes/digital-twins-developer-interfaces.md)]

También puede realizar cambios en el grafo mediante el ejemplo de Azure Digital Twins Explorer, que permite visualizar los gemelos y el grafo, y hace uso del SDK en segundo plano. En la sección siguiente se describe este ejemplo en detalle.

[!INCLUDE [visualizing with Azure Digital Twins explorer](../../includes/digital-twins-visualization.md)]

## <a name="create-relationships"></a>Crear relaciones

Las relaciones describen cómo los diferentes gemelos digitales se conectan entre sí, lo que constituye la base del grafo de gemelos.

Las relaciones se crean mediante la llamada a `CreateOrReplaceRelationshipAsync()`. 

Para crear una relación, debe especificar:
* El identificador del gemelo de origen (`srcId` en el ejemplo de código siguiente): identificador del gemelo en el que se origina la relación.
* El identificador del gemelo de destino (`targetId` en el ejemplo de código siguiente): identificador del gemelo al que llega la relación.
* Un nombre de relación (`relName` en el ejemplo de código siguiente): tipo genérico de relación, algo como _contains_ (contiene).
* Un identificador de relación (`relId` en el ejemplo de código siguiente): nombre específico de esta relación, algo como _Relationship1_.

El id. de relación debe ser único dentro del gemelo de origen especificado. No es necesario que sea único globalmente.
Por ejemplo, para el gemelo Foo, cada identificador de relación específico debe ser único. Sin embargo, otro gemelo llamado Bar puede tener una relación de salida que coincida con el mismo identificador de una relación de Foo.

En el ejemplo de código siguiente se muestra cómo crear una relación en la instancia de Azure Digital Twins. Usa la llamada de SDK (resaltada) de un método personalizado que puede aparecer en el contexto de un programa más grande.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="CreateRelationshipMethod" highlight="13":::

Ahora se puede llamar a esta función personalizada para crear una relación _contiene_ como esta: 

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseCreateRelationship":::

Si desea crear varias relaciones, puede repetir las llamadas al mismo método, con tipos de relación diferentes en el argumento. 

Para más información sobre la clase auxiliar `BasicRelationship`, consulte [Conceptos: Uso de las API y los SDK de Azure Digital Twins](concepts-apis-sdks.md#serialization-helpers).

### <a name="create-multiple-relationships-between-twins"></a>Creación de varias relaciones entre gemelos

Las relaciones se pueden clasificar como: 

* Relaciones de salida: relaciones que pertenecen a este gemelo que apuntan hacia afuera para conectarlo con otros gemelos. El método `GetRelationshipsAsync()` se usa para obtener las relaciones de salida de un gemelo.
* Relaciones de entrada: relaciones que pertenecen a otros gemelos que apuntan hacia este gemelo para crear un vínculo "entrante". El método `GetIncomingRelationshipsAsync()` se usa para obtener las relaciones de entrada de un gemelo.

No hay ninguna restricción en el número de relaciones que puede tener entre dos gemelos: puede tener tantas como desee. 

Esto significa que puede expresar varios tipos diferentes de relaciones entre dos gemelos a la vez. Por ejemplo, el gemelo A puede tener una relación *almacenada* y una relación *creada* con el gemelo B.

También puede crear varias instancias del mismo tipo de relación entre los mismos dos gemelos, si lo desea. En este ejemplo, el gemelo A podría tener dos relaciones *almacenadas* diferentes con el gemelo B, siempre y cuando las relaciones tengan identificadores de relación diferentes.

## <a name="list-relationships"></a>Enumeración de las relaciones

Para acceder a la lista de relaciones de **salida** de un gemelo determinado del grafo, puede usar el método `GetRelationships()` de la siguiente forma:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="GetRelationshipsCall":::

Esto devuelve un elemento `Azure.Pageable<T>` o `Azure.AsyncPageable<T>`, en función de si se usa la versión sincrónica o asincrónica de la llamada.

Este es un ejemplo que recupera una lista de relaciones. Usa la llamada de SDK (resaltada) de un método personalizado que puede aparecer en el contexto de un programa más grande.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="FindOutgoingRelationshipsMethod" highlight="8":::

Ahora puede llamar a este método personalizado para ver las relaciones de salida de los gemelos como se muestra a continuación:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseFindOutgoingRelationships":::

Puede usar las relaciones recuperadas para navegar a otros gemelos del grafo. Para ello, lea el campo `target` de la relación que se devuelve y úselo como id. de la siguiente llamada a `GetDigitalTwin()`.

### <a name="find-incoming-relationships-to-a-digital-twin"></a>Búsqueda de relaciones de entrada para un gemelo digital

Azure Digital Twins también dispone de una API para buscar todas las relaciones de **entrada** para un gemelo determinado. Suele ser útil para la navegación inversa o cuando se elimina un gemelo.

>[!NOTE]
> Las llamadas a `IncomingRelationship` no devuelven todo el cuerpo de la relación. Para más información sobre la clase `IncomingRelationship`, consulte su [documentación de referencia](/dotnet/api/azure.digitaltwins.core.incomingrelationship?view=azure-dotnet&preserve-view=true).

El ejemplo de código de la sección anterior se centraba en buscar relaciones de salida desde un gemelo. El ejemplo siguiente está estructurado de manera similar, pero en su lugar busca relaciones de *entrada* para el gemelo. En este ejemplo también se usa la llamada de SDK (resaltada) de un método personalizado que puede aparecer en el contexto de un programa más grande.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="FindIncomingRelationshipsMethod" highlight="8":::

Ahora puede llamar a este método personalizado para ver las relaciones de entrada de los gemelos como se muestra a continuación:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseFindIncomingRelationships":::

### <a name="list-all-twin-properties-and-relationships"></a>Enumeración de todas las propiedades y relaciones de un gemelo

Con los métodos anteriores para mostrar las relaciones de salida y de entrada de un gemelo, puede crear un método que imprima la información completa del gemelo, incluidas las propiedades del gemelo y ambos tipos de relaciones. Este es un método personalizado de ejemplo que muestra cómo combinar los métodos personalizados anteriores con este fin.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="FetchAndPrintMethod":::

Ahora puede llamar a esta función personalizada de la siguiente manera: 

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseFetchAndPrint":::

## <a name="update-relationships"></a>Actualización de relaciones

Las relaciones se actualizan mediante el método `UpdateRelationship`. 

>[!NOTE]
>Este método se usa para actualizar las **propiedades** de una relación. Si necesita cambiar el gemelo de origen o el de destino de la relación, deberá [eliminar la relación](#delete-relationships) y [volver a crear una](#create-relationships) con los nuevos gemelos.

Los parámetros necesarios para la llamada de cliente son el identificador del gemelo de origen (el gemelo en el que se origina la relación), el identificador de la relación que se va a actualizar y un documento de [revisión de JSON](http://jsonpatch.com/) que contiene las propiedades y los nuevos valores que quiere actualizar.

Este es el código de ejemplo que muestra cómo utilizar este método. En este ejemplo se usa la llamada de SDK (resaltada) de un método personalizado que puede aparecer en el contexto de un programa más grande.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UpdateRelationshipMethod" highlight="6":::

A continuación se muestra un ejemplo de una llamada a este método personalizado que pasa un documento de revisión de JSON con la información para actualizar una propiedad.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseUpdateRelationship":::

## <a name="delete-relationships"></a>Eliminar relaciones

El primer parámetro especifica el gemelo de origen (el gemelo en el que se origina la relación). El otro parámetro es el id. de relación. Necesita tanto el id. de gemelo como el id. de relación, ya que los id. de relación solo son únicos dentro del ámbito de un gemelo.

Este es el código de ejemplo que muestra cómo utilizar este método. En este ejemplo se usa la llamada de SDK (resaltada) de un método personalizado que puede aparecer en el contexto de un programa más grande.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="DeleteRelationshipMethod" highlight="5":::

Ahora puede llamar a este método personalizado para eliminar una relación de la siguiente forma:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseDeleteRelationship":::

## <a name="create-graph-from-a-csv-file"></a>Creación de un grafo a partir de un archivo .csv

En casos de uso prácticos, las jerarquías de gemelos se crean a menudo a partir de los datos almacenados en una base de datos diferente o quizás en una hoja de cálculo o un archivo .csv. En esta sección se ilustra cómo leer datos de un archivo .csv y crear un grafo gemelo con ellos.

Observe la siguiente tabla de datos, que describe un conjunto de gemelos digitales y las relaciones.

|  Id. de modelo    | Identificador del gemelo (debe ser único) | Nombre de relación  | Identificador del gemelo de destino  | Datos de inicialización del gemelo |
| --- | --- | --- | --- | --- |
| dtmi:example:Floor;1    | Floor1 | contiene | Room1 | |
| dtmi:example:Floor;1    | Floor0 | contains | Room0 | |
| dtmi:example:Room;1    | Room1 | | | {"Temperature": 80} |
| dtmi:example:Room;1    | Room0 | | | {"Temperature": 70} |

Una forma de introducir estos datos en Azure Digital Twins es convertir la tabla en un archivo .csv y escribir código que interprete el archivo en comandos para crear gemelos y relaciones. El siguiente código de ejemplo ilustra la lectura de los datos del archivo .csv y la creación de un grafo gemelo en Azure Digital Twins.

En el código siguiente, el archivo .csv se llama *data.csv* y hay un marcador de posición que representa el **nombre de host** de la instancia de Azure Digital Twins. El ejemplo también usa varios paquetes, que se pueden agregar al proyecto para que faciliten este proceso.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graphFromCSV.cs":::

## <a name="runnable-twin-graph-sample"></a>Ejemplo de grafo de gemelos ejecutable

En el siguiente fragmento de código ejecutable se usan las operaciones de relación de este artículo para crear un grafo de gemelos a partir de gemelos digitales y relaciones.

### <a name="set-up-sample-project-files"></a>Configuración de los archivos del proyecto de ejemplo

El fragmento de código usa dos definiciones de modelo de ejemplo, [Room.json](https://raw.githubusercontent.com/Azure-Samples/digital-twins-samples/master/AdtSampleApp/SampleClientApp/Models/Room.json) y [Floor.json](https://raw.githubusercontent.com/Azure-Samples/digital-twins-samples/master/AdtSampleApp/SampleClientApp/Models/Floor.json). Para **descargar los archivos del modelo** para poder usarlos en el código, use estos vínculos para ir directamente a los archivos de GitHub. A continuación, haga clic con el botón derecho en cualquier lugar de la pantalla, seleccione **Guardar como** en el menú contextual del explorador y use la ventana Guardar como para guardar los archivos como **Room.json** y **Floor.json**.

A continuación, cree un **nuevo proyecto de aplicación de consola** en Visual Studio o el editor que prefiera.

Luego **copie el código siguiente** del ejemplo ejecutable en el proyecto:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs":::

### <a name="configure-project"></a>Configurar proyecto

Luego siga estos pasos para configurar el código del proyecto:
1. Agregue los archivos **Room.json** y **Floor.json** que ha descargado anteriormente al proyecto y reemplace los marcadores de posición `<path-to>` en el código para indicar al programa dónde encontrarlos.
1. Reemplace el marcador de posición `<your-instance-hostname>` por el nombre de host de la instancia de Azure Digital Twins.
1. Agregue dos dependencias al proyecto ya que las necesitará para trabajar con Azure Digital Twins. La primera es el paquete del [SDK de Azure Digital Twins para .NET](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true) y la segunda proporciona herramientas para ayudar con la autenticación en Azure.

      ```cmd/sh
      dotnet add package Azure.DigitalTwins.Core
      dotnet add package Azure.Identity
      ```

También necesitará configurar las credenciales locales si desea ejecutar el ejemplo directamente. La siguiente sección le indicará cómo hacerlo.
[!INCLUDE [Azure Digital Twins: local credentials prereq (outer)](../../includes/digital-twins-local-credentials-outer.md)]

### <a name="run-the-sample"></a>Ejecución del ejemplo

Ahora que ha completado la configuración, puede ejecutar el proyecto de código de ejemplo.

Esta es la salida de consola del programa: 

:::image type="content" source="./media/how-to-manage-graph/console-output-twin-graph.png" alt-text="Salida de consola que muestra los detalles de los gemelos y las relaciones entrantes y salientes de los gemelos." lightbox="./media/how-to-manage-graph/console-output-twin-graph.png":::

> [!TIP]
> El grafo de gemelos es un concepto de creación de relaciones entre gemelos. Si desea ver la representación visual del grafo de gemelos, consulte la sección [Visualización](how-to-manage-graph.md#visualization) de este artículo. 

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo consultar un grafo de gemelos de Azure Digital Twins:
* [Conceptos: Lenguaje de consulta](concepts-query-language.md)
* [Procedimiento: Consulta del grafo de gemelos](how-to-query-graph.md)