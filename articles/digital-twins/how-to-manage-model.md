---
title: Administración de modelos DTDL
titleSuffix: Azure Digital Twins
description: Vea cómo crear, editar y eliminar un modelo en Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 4/07/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 3c896e2aad7dae8d03b26e2a16ecb96224ab547b
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107303787"
---
# <a name="manage-azure-digital-twins-models"></a>Administración de modelos de Azure Digital Twins

Puede administrar los [modelos](concepts-models.md) de la instancia de Azure Digital Twins mediante las [**API DigitalTwinModels**](/rest/api/digital-twins/dataplane/models), el [SDK para .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client) o la [CLI de Azure Digital Twins](how-to-use-cli.md). 

Las operaciones de administración incluyen la carga, validación, recuperación y eliminación de modelos. 

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [digital-twins-prereq-instance.md](../../includes/digital-twins-prereq-instance.md)]

## <a name="ways-to-manage-models"></a>Formas de administrar los modelos

[!INCLUDE [digital-twins-ways-to-manage.md](../../includes/digital-twins-ways-to-manage.md)]

## <a name="create-models"></a>Crear modelos

Los modelos de Azure Digital Twins se escriben en DTDL y se guardan como archivos *.json*. También hay una [extensión de DTDL](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-dtdl) disponible para [Visual Studio Code](https://code.visualstudio.com/), que proporciona validación de sintaxis y otras características para facilitar la escritura de documentos DTDL.

Considere un ejemplo en el que un hospital quiere representar digitalmente sus habitaciones. Cada habitación contiene un dispensador de jabón inteligente para supervisar el lavado de manos y sensores para controlar el tráfico en la habitación.

El primer paso para la solución consiste en crear modelos para representar aspectos del hospital. La habitación de un paciente en este escenario podría describirse de la siguiente manera:

:::code language="json" source="~/digital-twins-docs-samples/models/PatientRoom.json":::

> [!NOTE]
> Se trata de un cuerpo de ejemplo de un archivo .json en el que se define y se guarda un modelo, que se va a cargar como parte de un proyecto de cliente. La llamada a la API de REST, por otro lado, adopta una matriz de definiciones de modelo como la anterior (que se asigna a un atributo `IEnumerable<string>` en el SDK de .NET). Por lo tanto, para usar este modelo en la API de REST directamente, inclúyalo entre corchetes.

Este modelo define un nombre y un identificador único para la habitación del paciente, y las propiedades para representar el número de visitantes y el estado del lavado de mano (estos contadores se actualizarán a partir de sensores de movimiento y dispensadores de jabón inteligentes, y se usarán juntos para calcular una propiedad de *porcentaje de lavado de manos*). El modelo también define una relación *hasDevices*, que se usará para conectar cualquier instancia de [Digital Twins](concepts-twins-graph.md) basada este modelo de *Habitación* con los dispositivos reales.

Siguiendo este método, puede continuar con la definición de modelos para las salas y las zonas del hospital, o bien para todo el hospital.

### <a name="validate-syntax"></a>Validación de la sintaxis

[!INCLUDE [Azure Digital Twins: validate models info](../../includes/digital-twins-validate.md)]

## <a name="upload-models"></a>Carga de modelos

Una vez creados los modelos, puede cargarlos en la instancia de Azure Digital Twins.

Cuando esté listo para cargar un modelo, puede usar el fragmento de código siguiente:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/model_operations.cs" id="CreateModel":::

Observe que el método `CreateModels` acepta varios archivos en una sola transacción. Este es un ejemplo ilustrativo:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/model_operations.cs" id="CreateModels_multi":::

Los archivos de modelo pueden contener más de un modelo. En este caso, los modelos deben colocarse en una matriz JSON. Por ejemplo:

:::code language="json" source="~/digital-twins-docs-samples/models/Planet-Moon.json":::

En la carga, el servicio valida los archivos de modelo.

## <a name="retrieve-models"></a>Recuperación de modelos

Puede mostrar y recuperar los modelos almacenados en la instancia de Azure Digital Twins. 

Estas son las opciones para ello:
* Recuperar un único modelo
* Recuperar todos los modelos
* Recuperar los metadatos y las dependencias de los modelos.

A continuación, se muestran algunas llamadas de ejemplo:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/model_operations.cs" id="GetModels":::

Todas las llamadas API para recuperar modelos devuelven todos los objetos `DigitalTwinsModelData`. `DigitalTwinsModelData` contiene metadatos sobre el modelo almacenado en la instancia Azure Digital Twins, como el nombre, la DTMI y la fecha de creación del modelo. El objeto `DigitalTwinsModelData` también incluye opcionalmente el propio modelo. En función de los parámetros, puede usar las llamadas de recuperación para recuperar solo los metadatos (lo que resulta útil en escenarios en los que se desea mostrar una lista de interfaz de usuario de las herramientas disponibles, por ejemplo) o todo el modelo.

La llamada a `RetrieveModelWithDependencies` devuelve no solo el modelo solicitado, sino también todos los modelos de los que depende el modelo solicitado.

Los modelos no se devuelven necesariamente con el formato de documento exacto con el que se cargaron. Azure Digital Twins solo garantiza que el formato devuelvo será equivalente desde el punto de vista semántico. 

## <a name="update-models"></a>Actualización de modelos

Una vez que se carga un modelo en la instancia de Azure Digital Twins, toda la interfaz del modelo es inmutable. Esto significa que no hay ninguna "edición" tradicional de los modelos. Azure Digital Twins tampoco permite volver a cargar el mismo modelo.

En su lugar, si desea realizar cambios en un modelo, como cargar `displayName` o `description`, la manera de hacerlo es cargar una **versión más reciente** del modelo. 

### <a name="model-versioning"></a>Control de versiones de los modelos

Para crear una nueva versión de un modelo existente, empiece con el DTDL del modelo original. Actualice, agregue o quite los campos que quiera cambiar.

A continuación, márquelo como una versión más reciente del modelo actualizando el campo `id` del modelo. La última sección del identificador de modelo, después del `;`, representa el número de modelo. Para indicar que esta es una versión más actualizada de este modelo, incremente el número al final del valor `id` en cualquier número mayor que el número de versión actual.

Por ejemplo, si el identificador de modelo anterior tuviera el siguiente aspecto:

```json
"@id": "dtmi:com:contoso:PatientRoom;1",
```

la versión 2 de este modelo podría tener este aspecto:

```json
"@id": "dtmi:com:contoso:PatientRoom;2",
```

A continuación, cargue la nueva versión del modelo en la instancia. 

Esta versión del modelo estará disponible en su instancia para usarse con gemelos digitales. **No** sobrescriba versiones anteriores del modelo, por lo que varias versiones del modelo coexistirán en la instancia hasta que las [quite](#remove-models).

### <a name="impact-on-twins"></a>Impacto en gemelos

Cuando se crea un gemelo, dado que la nueva versión del modelo y la versión del modelo anterior coexisten, el nuevo gemelo puede usar la nueva versión del modelo o la versión anterior.

Esto también significa que la carga de una nueva versión de un modelo no afecta automáticamente a los gemelos que ya existan. Los gemelos existentes simplemente conservarán las instancias de la versión antigua del modelo.

Puede actualizar estos gemelos a la nueva versión del modelo aplicándole revisiones, tal como se describe en la sección [*Actualización de un modelo de gemelo digital*](how-to-manage-twin.md#update-a-digital-twins-model) de *Procedimientos: Administración de Digital Twins*. Dentro de la misma revisión, debe actualizar los **identificadores de modelo** (a la nueva versión) y los **campos que deben modificarse para que se ajusten al nuevo modelo**.

## <a name="remove-models"></a>Eliminación de modelos

Los modelos también se pueden quitar del servicio, de alguna de estas dos maneras:
* **Retirada**: una vez que se retira un modelo, ya no se puede usar para crear instancias de Digital Twins. Las instancias existentes de Digital Twins que ya usan este modelo no se ven afectadas, por lo que todavía se pueden actualizar con elementos como cambios de propiedades y con la adición o eliminación de relaciones.
* **Eliminación**: el modelo se eliminará definitivamente de la solución. Todas las instancias de Digital Twins que utilicen este modelo dejarán de estar asociadas con cualquier modelo válido, por lo que se tratarán como si no tuvieran ningún modelo. Todavía puede leer estas instancias de Digital Twins, pero no podrá actualizarlas hasta que se reasignen a un modelo diferente.

Se trata de características independientes que no se ven afectadas entre sí, aunque se pueden usar juntas para quitar un modelo gradualmente. 

### <a name="decommissioning"></a>Retirada

Este es el código para retirar un modelo:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/model_operations.cs" id="DecommissionModel":::

El estado de retirada de un modelo se incluye en los registros `ModelData` devueltos por las API de recuperación del modelo.

### <a name="deletion"></a>Eliminación

Puede eliminar todos los modelos de la instancia a la vez o de manera individual.

Para un ejemplo de cómo eliminar todos los modelos, descargue la aplicación de ejemplo que se usa en el [*Tutorial: Exploración de los conceptos básicos con una aplicación cliente de ejemplo*](tutorial-command-line-app.md). El archivo *CommandLoop.cs* realiza esta acción en una función `CommandDeleteAllModels`.

En el resto de esta sección se desglosa la eliminación del modelo para obtener más detalles, y se explica el proceso de eliminación individual.

#### <a name="before-deletion-deletion-requirements"></a>Antes de la eliminación: Requisitos para la eliminación

Por lo general, los modelos se pueden eliminar en cualquier momento.

La excepción son los modelos de los que dependen otros modelos, ya sea con una relación `extends` o como componente. Por ejemplo, si un modelo *ConferenceRoom* extiende un modelo *Room* y tiene un modelo *ACUnit* como componente, no puede eliminar *Room* ni *ACUnit* hasta que *ConferenceRoom* elimina dichas referencias correspondientes. 

Para ello, puede actualizar el modelo dependiente para quitar las dependencias o eliminarlo por completo.

#### <a name="during-deletion-deletion-process"></a>Durante la eliminación: Proceso de eliminación

Incluso si un modelo cumple los requisitos para eliminarlo inmediatamente, es posible que quiera seguir unos cuantos pasos para evitar consecuencias no deseadas en las instancias de Digital Twins dependientes. Estos son algunos de los pasos que pueden facilitar la administración del proceso:
1. En primer lugar, retire el modelo.
2. Espere unos minutos para asegurarse de que el servicio ha procesado las solicitudes de creación gemelas de último minuto enviadas antes de la retirada.
3. Consulte las instancias de Digital Twins por modelo para ver todas las instancias de Digital Twins que usan el modelo que ya se ha retirado.
4. Elimine las instancias de Digital Twins si ya no las necesita o aplíqueles una revisión en un nuevo modelo, si es necesario. También puede optar por dejarlas como independientes, en cuyo caso se convertirán en instancias de Digital Twins sin modelos una vez que se elimine el modelo. Vea la sección siguiente para conocer las implicaciones de este estado.
5. Espere unos minutos para asegurarse de que los cambios se han aplicado.
6. Eliminación del modelo 

Para eliminar un modelo, use esta llamada:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/model_operations.cs" id="DeleteModel":::

#### <a name="after-deletion-twins-without-models"></a>Después de la eliminación: Instancias de Digital Twins sin modelos

Una vez eliminado un modelo, ahora se considerará que todas las instancias de Digital Twins que usaban dicho modelo ya no tendrán ningún modelo. Tenga en cuenta que no hay ninguna consulta que pueda proporcionar una lista de todas las instancias de Digital Twins con este estado, aunque *puede* seguir consultando las instancias de Digital Twins del modelo eliminado para saber qué instancias se ven afectadas.

A continuación, se ofrece información general sobre lo que puede y no puede hacer con instancias de Digital Twins que no tienen ningún modelo.

Cosas que **puede** hacer:
* Consulta de la instancia de Digital Twins.
* Lectura de las propiedades
* Lectura de las relaciones salientes
* Adición y eliminación de las relaciones entrantes (de la misma forma, otras instancias de Digital Twins todavía pueden formar relaciones *con* esta instancia de Digital Twins)
  - El atributo `target` de la definición de la relación todavía puede reflejar el DTMI del modelo eliminado. Una relación sin destino definido también puede funcionar aquí.      
* Eliminar relaciones
* Eliminación de la instancia de Digital Twins

Cosas que **no puede** hacer:
* Editar las relaciones de salida (de la misma forma, las relaciones *de* esta instancia de Digital Twins con otras)
* Modificar propiedades

#### <a name="after-deletion-re-uploading-a-model"></a>Después de la eliminación: volver a cargar un modelo

Una vez eliminado un modelo, puede decidir más tarde si cargar un nuevo modelo con el mismo identificador que el que ha eliminado. Esto es lo que sucede en ese caso.
* Desde la perspectiva del almacén de soluciones, es lo mismo que cargar un modelo completamente nuevo. El servicio no recuerda si el anterior se cargó alguna vez.   
* Si hay otras instancias de Digital Twins en el grafo que hacen referencia al modelo eliminado, dejarán de estar huérfanas; este identificador de modelo vuelve a ser válido con la nueva definición. Sin embargo, si la nueva definición del modelo es diferente de la definición del modelo que se eliminó, estas instancias de Digital Twins pueden tener propiedades y relaciones que coincidan con la definición eliminada y que no sean válidas con la nueva.

Azure Digital Twins no impide este estado, por lo que debe asegurarse de revisar detenidamente las instancias de Digital Twins para comprobar que siguen siendo válidas para el cambio de la definición del modelo.

## <a name="next-steps"></a>Pasos siguientes

Vea cómo crear y administrar instancias de Digital Twins basadas en los modelos:
* [*Procedimiento: Administración de Digital Twins*](how-to-manage-twin.md)