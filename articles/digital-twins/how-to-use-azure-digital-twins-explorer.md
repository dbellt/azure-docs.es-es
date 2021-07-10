---
title: Uso de Azure Digital Twins Explorer
titleSuffix: Azure Digital Twins
description: Aprenda a usar las características de Azure Digital Twins Explorer.
author: baanders
ms.author: baanders
ms.date: 4/20/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 9d029c3392ad5b5472173d17db17a113526d4f3a
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111811513"
---
# <a name="use-azure-digital-twins-explorer-preview"></a>Uso de Azure Digital Twins Explorer (versión preliminar)

[Azure Digital Twins Explorer](concepts-azure-digital-twins-explorer.md) es una herramienta para visualizar y trabajar con instancias de Azure Digital Twins. En este artículo se describen las características de Azure Digital Twins Explorer y cómo usarlas para administrar los datos de la instancia de Azure Digital Twins. 

>[!NOTE]
>Esta herramienta se encuentra actualmente en **versión preliminar pública**.

[!INCLUDE [digital-twins-access-explorer.md](../../includes/digital-twins-access-explorer.md)]

### <a name="switch-contexts-within-the-app"></a>Cambio de contexto dentro de la aplicación

Una vez en la aplicación, también puede cambiar qué instancia está conectada actualmente al explorador; para ello, seleccione el nombre de la instancia en la barra de herramientas superior.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/instance-url-1.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer. Está resaltado el nombre de instancia en la barra de herramientas superior." lightbox="media/how-to-use-azure-digital-twins-explorer/instance-url-1.png":::

Esta acción mostrará el **modal de URL de Azure Digital Twins**, donde puede escribir el nombre de host de otra instancia de Azure Digital Twins después de *https://* para conectarse a esa instancia.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/instance-url-2.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer. El modal de URL de Azure Digital Twins muestra un cuadro editable que contiene https:// y un nombre de host." lightbox="media/how-to-use-azure-digital-twins-explorer/instance-url-2.png":::

>[!NOTE]
>En estos momentos, la capacidad de cambiar de contexto dentro de la aplicación **no** está disponible para las cuentas personales de Microsoft (MSA). Los usuarios de MSA tendrán que acceder al explorador desde la instancia correcta en Azure Portal o bien pueden conectarse a una instancia determinada por medio de un [vínculo directo al entorno](#link-to-your-environment).


## <a name="query-your-digital-twin-graph"></a>Consulta del grafo de gemelos digitales

Puede usar el panel **Query Explorer** (Explorador de consultas) para realizar [consultas](concepts-query-language.md) en el grafo.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/query-explorer-panel.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer. Está resaltado el panel Query Explorer (Explorador de consultas)." lightbox="media/how-to-use-azure-digital-twins-explorer/query-explorer-panel.png":::

Escriba la consulta que desea ejecutar y seleccione el botón **Run Query** (Ejecutar consulta). Esto cargará los resultados de la consulta en el panel **Twin Graph** (Grafo de gemelos).

>[!NOTE]
> Los resultados de la consulta que contienen relaciones solo se pueden representar en el panel **Twin Graph** (Grafo de gemelos) si los resultados incluyen también al menos un gemelo. Aunque las consultas que devuelven solo relaciones son posibles en Azure Digital Twins, solo se pueden ver en Azure Digital Twins Explorer mediante el [panel Output](#advanced-settings) (Salida).

### <a name="overlay-query-results"></a>Superposición de los resultados de la consulta

Puede marcar la casilla **Overlay results** (Superponer resultados) antes de ejecutar la consulta si desea que los resultados se resalten respecto de lo que se muestra actualmente en el panel **Twin Graph** (Grafo de gemelos), en lugar de reemplazar completamente el contenido del panel por los nuevos resultados de la consulta.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/query-explorer-panel-overlay-results.png" alt-text="Captura de pantalla del panel Query Explorer (Explorador de consultas) de Azure Digital Twins Explorer. La casilla Overlay Results (Superponer resultados) está marcada y hay dos gemelos resaltados en el grafo más grande que se muestra en el panel Twin Graph (Grafo de gemelos)." lightbox="media/how-to-use-azure-digital-twins-explorer/query-explorer-panel-overlay-results.png":::

Si el resultado de la consulta incluye algún elemento que no se muestra actualmente en el panel **Twin Graph** (Grafo de gemelos), ese elemento se agregará a la vista existente.

### <a name="save-and-rerun-queries"></a>Guardar y volver a ejecutar consultas

Las consultas se pueden guardar en el almacenamiento local en el explorador, lo que facilita su selección y ejecución.

>[!NOTE]
>El almacenamiento en el explorador local significa que las consultas guardadas no estarán disponibles en otros exploradores distintos de aquel en el que las guardó y que permanecerán en el almacenamiento del explorador indefinidamente hasta que se borre el almacenamiento local.

Para guardar una consulta, escríbala en el cuadro de consulta y seleccione el icono para **guardar** situado a la derecha del panel. Escriba un nombre para la consulta guardada cuando se le solicite.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/query-explorer-panel-save.png" alt-text="Captura de pantalla del panel Query Explorer (Explorador de consultas) de Azure Digital Twins Explorer. El icono para guardar está resaltado." lightbox="media/how-to-use-azure-digital-twins-explorer/query-explorer-panel-save.png":::

Una vez guardada la consulta, está disponible para seleccionarla en el menú desplegable **Saved Queries** (Consultas guardadas) para volver a ejecutarla fácilmente.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/query-explorer-panel-saved-queries.png" alt-text="Captura de pantalla del panel Query Explorer (Explorador de consultas) de Azure Digital Twins Explorer. El menú desplegable Saved Queries (Consultas guardadas) está resaltado y muestra dos consultas de ejemplo." lightbox="media/how-to-use-azure-digital-twins-explorer/query-explorer-panel-saved-queries.png":::

Para eliminar una consulta guardada, seleccione el icono **X** junto al nombre de la consulta cuando el menú desplegable **Saved Queries** (Consultas guardadas) esté abierto.

>[!TIP]
>En el caso de los grafos de gran tamaño, se recomienda consultar solo un subconjunto limitado y después cargar el resto del grafo según sea necesario. Puede hacer doble clic en un gemelo del panel **Twin Graph** (Grafo de gemelos) para recuperar otros nodos relacionados.

## <a name="explore-the-twin-graph"></a>Exploración en el panel Twin Graph (Grafo de gemelos)

El panel **Twin Graph** (Grafo de gemelos) permite explorar los gemelos y las relaciones de la instancia.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer. El panel Twin Graph (Grafo de gemelos) está resaltado." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel.png":::

Puede usar este panel para [ver los gemelos y las relaciones](#view-twins-and-relationships).

El panel Twin Graph (Grafo de gemelos) también ofrece varias opciones para personalizar la experiencia de visualización del grafo:
* [Editar el diseño del grafo de gemelos](#edit-twin-graph-layout)
* [Controlar la expansión del grafo de gemelos](#control-twin-graph-expansion)
* [Mostrar y ocultar elementos del grafo de gemelos](#show-and-hide-twin-graph-elements)
* [Filtrar y resaltar elementos del grafo de gemelos](#filter-and-highlight-twin-graph-elements)

### <a name="view-twins-and-relationships"></a>Visualización de gemelos y relaciones

Ejecute una consulta mediante [Query Explorer](#query-your-digital-twin-graph) (Explorador de consultas) para ver los gemelos y las relaciones del resultado de la consulta que se muestra en el panel **Twin Graph** (Grafo de gemelos).

>[!TIP]
>La consulta para mostrar todos los gemelos y relaciones es `SELECT * FROM digitaltwins`.

#### <a name="view-twin-and-relationship-properties"></a>Visualización de las propiedades de gemelos y relaciones

Para ver los valores de propiedad de un gemelo o una relación, seleccione el gemelo o la relación en el panel **Twin Graph** (Grafo de gemelos) y use el botón **Toggle property inspector** (Alternar inspector de propiedades) para expandir el panel **Properties** (Propiedades). Este panel mostrará todas las propiedades asociadas al elemento, junto con sus valores. También incluye los valores predeterminados de las propiedades que aún no se han establecido.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-highlight-graph-properties.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El gemelo FactoryA está seleccionado y el panel Properties (Propiedades) está expandido, mostrando las propiedades del gemelo." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-highlight-graph-properties.png":::

Las propiedades suelen aparecer con texto blanco, pero también pueden aparecer con los colores siguientes para indicar información adicional:

* **Texto rojo para el modelo**: indica que no se encuentra el modelo para el gemelo. Esto puede ocurrir si se ha eliminado el modelo desde que se creó el gemelo.

    :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/properties-color-red.png" alt-text="Captura de pantalla del panel Properties (Propiedades) de Azure Digital Twins Explorer que muestra las propiedades de un gemelo de ejemplo. El campo $model y su valor se muestran con texto rojo." lightbox="media/how-to-use-azure-digital-twins-explorer/properties-color-red.png":::

* **Texto amarillo para la propiedad**: indica que la propiedad no forma parte de la definición del modelo que usa el gemelo. Esto puede ocurrir si el modelo del gemelo se ha reemplazado o cambiado desde que se creó la propiedad y esta ya no existe en la versión más reciente del modelo. **Los gemelos con propiedades obsoletas no se pueden actualizar, a menos que la actualización también corrija o quite las propiedades obsoletas.**

    :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/properties-color-yellow.png" alt-text="Captura de pantalla del panel Properties (Propiedades) de Azure Digital Twins Explorer que muestra las propiedades de un gemelo de ejemplo. Se muestran varios nombres de propiedad con texto amarillo." lightbox="media/how-to-use-azure-digital-twins-explorer/properties-color-yellow.png":::

#### <a name="view-a-twins-relationships"></a>Visualización de las relaciones de un gemelo

También puede ver rápidamente el código de todas las relaciones que implican a un gemelo determinado (incluidas las relaciones entrantes y salientes).

Para ello, haga clic con el botón derecho en un gemelo del grafo y elija **Get relationships** (Obtener relaciones). Se abre el modal **Relationship Information** (Información de relaciones) que muestra la [representación JSON](concepts-twins-graph.md#relationship-json-format) de todas las relaciones entrantes y salientes.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-get-relationships.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El centro de la pantalla muestra un modal Relationship Information (Información de relaciones) con las relaciones entrantes y salientes." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-get-relationships.png":::

### <a name="edit-twin-graph-layout"></a>Edición del diseño del grafo de gemelos

Para reorganizar los gemelos en distintas configuraciones, haga clic en ellos y arrástrelos alrededor de la pantalla Twin Graph (Grafo de gemelos).

También puede aplicar uno de varios algoritmos de diseño al grafo con las opciones del menú **Run Layout** (Ejecutar diseño). 

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-layout.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El botón Run Layout (Ejecutar diseño) está resaltado y muestra un menú con las opciones de diseño Cola, Dagre, fCoSE y Klay." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-layout.png":::

### <a name="control-twin-graph-expansion"></a>Control de la expansión del grafo de gemelos

Al ver el resultado de una consulta en el panel **Twin Graph** (Grafo de gemelos), puede hacer doble clic en un gemelo para que el grafo obtenga sus relaciones y gemelos relacionados y los muestre si aún no están presentes en la vista. Puede personalizar esta expansión estableciendo un tamaño y una dirección para determinar cuántos gemelos se deben recuperar.

Para establecer el número de niveles que se expandirán, use la opción **Expansion Level** (Nivel de expansión). Este número indica cuántos niveles de relaciones se capturan del gemelo seleccionado.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-expansion-level.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El botón Expansion Level (Nivel de expansión) está resaltado." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-expansion-level.png":::

Para indicar qué tipos de relaciones se deben seguir al expandir, use el botón **Expansion Mode** (Modo de expansión). Esta opción le permite seleccionar entre relaciones entrantes, salientes o entrantes y salientes.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-expansion-mode.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El botón Expansion Modo (Modo de expansión) está resaltado y muestra las opciones de entrada, salida y entrada y salida." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-expansion-mode.png":::

### <a name="show-and-hide-twin-graph-elements"></a>Mostrar y ocultar elementos del grafo de gemelos

Puede alternar la opción para ocultar gemelos o relaciones en la vista del grafo. 

Para ocultar un gemelo o una relación, haga clic con el botón derecho en el elemento en la ventana **Twin Graph** (Grafo de gemelos). Se mostrará un menú con una opción para ocultar el elemento u otros elementos relacionados.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-hide.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El gemelo FactoryA está seleccionado y hay un menú que contiene opciones para ocultar la selección, ocultar la selección y los elementos secundarios, ocultar todos los demás y ocultar elementos no secundarios." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-hide.png":::

También puede ocultar varios gemelos o relaciones a la vez usando las teclas CTRL/CMD o MAYÚS para seleccionar varios elementos del mismo tipo en el grafo. A partir de aquí, siga el mismo proceso de clic con el botón derecho para ver las opciones de ocultación.

Para volver a mostrar todos los gemelos después de ocultar algunos, use el botón **Show All** (Mostrar todo). Para volver a mostrar todas las relaciones, use el botón **Show All Relationships** (Mostrar todas las relaciones).

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-show.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. Los botones Show All (Mostrar todo) y Show All Relationships (Mostrar todas las relaciones) están resaltados." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-show.png":::

### <a name="filter-and-highlight-twin-graph-elements"></a>Filtrado y resaltado de elementos del grafo de gemelos

Puede filtrar los gemelos y las relaciones que aparecen en el grafo por texto; para ello, seleccione este icono de **filtro**:

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-filter-text.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. Está seleccionado el icono de filtro de texto, que muestra la pestaña Filter (Filtro) donde puede escribir un término de búsqueda." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-filter-text.png":::

También puede resaltar los gemelos y los elementos que aparecen en el grafo por texto; para ello, seleccione este icono de **resaltado**:

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-highlight-text.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. Está seleccionado el icono de filtro de texto, que muestra la pestaña Highlight (Resaltar) donde puede escribir un término de búsqueda." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-highlight-text.png":::

## <a name="manage-twins-and-graph"></a>Administración de gemelos y el grafo

Azure Digital Twins Explorer proporciona varias maneras de administrar los [gemelos](concepts-twins-graph.md#digital-twins) y las [relaciones](concepts-twins-graph.md#relationships-a-graph-of-digital-twins) de la instancia.

En esta sección se describe cómo se llevan a cabo las siguientes actividades de administración:
* [Creación de gemelos](#create-twins), con o sin propiedades iniciales
* [Creación de relaciones](#create-relationships) entre gemelos
* [Edición de gemelos y relaciones](#edit-twins-and-relationships)
* [Eliminación de gemelos y relaciones](#delete-twins-and-relationships)

Para obtener información sobre la experiencia de visualización de gemelos y relaciones, consulte [Exploración del panel Twin Graph (Grafo de gemelos)](#explore-the-twin-graph).

### <a name="create-twins"></a>Creación de gemelos

Puede crear un nuevo gemelo digital a partir de su definición de modelo en el panel **Models** (Modelos).

Para crear un gemelo a partir de un modelo, busque ese modelo en la lista y elija el icono **Create a Twin** (Crear un gemelo) junto al nombre del modelo. Se le pedirá que escriba un **nombre** para el nuevo gemelo, que debe ser único. A continuación, guarde el gemelo; esta acción lo agregará al grafo.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/models-panel-create-a-twin.png" alt-text="Captura de pantalla del panel Models (Modelos) de Azure Digital Twins Explorer. Está resaltado el icono Create a Twin (Crear un gemelo) para un único modelo." lightbox="media/how-to-use-azure-digital-twins-explorer/models-panel-create-a-twin.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Para agregar valores de propiedad al gemelo, consulte [Edición de gemelos y relaciones](#edit-twins-and-relationships).

### <a name="create-relationships"></a>Crear relaciones

Para crear una relación entre dos gemelos, empiece por seleccionar el gemelo de origen de la relación en la ventana **Twin Graph** (Grafo de gemelos). A continuación, mantenga presionada la tecla CTRL/CMD o MAYÚS mientras selecciona un segundo gemelo como destino de la relación.

Una vez seleccionados simultáneamente los dos gemelos, haga clic con el botón derecho en cualquiera de ellos. Se mostrará un menú con la opción **Add relationships** (Agregar relaciones) entre ellos.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-add-relationship.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. Los gemelos FactoryA y Consumer están seleccionados y un menú muestra la opción Add relationships (Agregar relaciones)." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-add-relationship.png":::

Se mostrará el cuadro de diálogo **Create Relationship** (Crear relación), que incluye el gemelo de origen y el gemelo de destino de la relación, seguido de un menú desplegable **Relationship** (Relación) que contiene los tipos de relación que puede tener el gemelo de origen (definidos en su modelo DTDL). Seleccione una opción para el tipo de relación y **guarde** la nueva relación.

### <a name="edit-twins-and-relationships"></a>Edición de gemelos y relaciones

Para ver los valores de propiedad de un gemelo o una relación, seleccione el elemento en el panel **Twin Graph** (Grafo de gemelos) y use el botón **Toggle property inspector** (Alternar inspector de propiedades) para expandir el panel **Properties** (Propiedades).

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-highlight-graph-properties.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El gemelo FactoryA está seleccionado y el panel Properties (Propiedades) está expandido, mostrando las propiedades del gemelo." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-highlight-graph-properties.png":::

Puede usar este panel para editar directamente las propiedades que se pueden modificar. Actualice sus valores en línea y haga clic en el botón **Patch twin** (Revisar gemelo) (guardar) de la parte superior del panel para guardar los cambios. Cuando se guarda la actualización, la pantalla muestra una ventana modal con la operación JSON de revisión aplicada por la [API de actualización](/rest/api/azure-digitaltwins/).

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-highlight-graph-properties-save.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El centro de la pantalla muestra un modal de información de revisión con el código JSON de revisión." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-highlight-graph-properties-save.png":::

### <a name="delete-twins-and-relationships"></a>Eliminación de gemelos y relaciones

Para eliminar un gemelo o una relación, haga clic con el botón derecho en el elemento en la ventana **Twin Graph** (Grafo de gemelos). Se mostrará un menú con una opción para eliminarlo.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-delete.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El gemelo FactoryA está seleccionado y hay un menú que contiene una opción para eliminar gemelos." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-delete.png":::

Puede eliminar varios gemelos o varias relaciones a la vez usando las teclas CTRL/CMD o MAYÚS para seleccionar varios elementos del mismo tipo en el grafo. A partir de aquí, siga el mismo proceso de clic con el botón derecho para eliminar los elementos.

También puede eliminar todos los gemelos de la instancia al mismo tiempo, mediante el botón **Delete All Twins** (Eliminar todos los gemelos) de la barra de herramientas superior.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/delete-all-twins.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer. El icono Delete All Twins (Eliminar todos los gemelos) está seleccionado." lightbox="media/how-to-use-azure-digital-twins-explorer/delete-all-twins.png":::

## <a name="explore-models-and-the-model-graph"></a>Exploración de modelos y del panel Model Graph (Grafo de modelos)

Los modelos se pueden ver en el panel **Models** (Modelos) del lado izquierdo de la pantalla Azure Digital Twins Explorer y en el panel **Model Graph** (Grafo de modelos) en el centro de la pantalla.

El panel **Models** (Modelos): :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/models-panel.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer. El panel Models (Modelos) está resaltado." lightbox="media/how-to-use-azure-digital-twins-explorer/models-panel.png":::

El panel **Model Graph** (Grafo de modelos): :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/model-graph-panel.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer. El panel Model Graph (Grafo de modelos) está resaltado." lightbox="media/how-to-use-azure-digital-twins-explorer/model-graph-panel.png":::

Puede usar estos paneles para [ver los modelos](#view-models).

El panel Model Graph (Grafo de modelos) también ofrece varias opciones para personalizar la experiencia de visualización del grafo:
* [Edición del diseño del grafo de modelos](#edit-model-graph-layout)
* [Filtrado y resaltado de elementos del grafo de modelos](#filter-and-highlight-model-graph-elements)
* [Carga de imágenes de modelo](#upload-model-images) para representar modelos en los grafos

### <a name="view-models"></a>Ver modelos

Puede ver una lista sin formato de los modelos de la instancia en el panel **Models** (Modelos). Puede usar la barra de **búsqueda** del panel para buscar en esta lista.

Puede usar el panel **Model Graph** (Grafo de modelos) para ver una representación gráfica de los modelos de la instancia, junto con las relaciones, la herencia y los componentes que los conectan entre sí.

#### <a name="view-model-definition"></a>Visualización de la definición del modelo

Para ver la definición completa de un modelo, busque ese modelo en el panel **Models** (Modelos) y elija el icono **View Model** (Ver modelo) situado junto al nombre del modelo. Se mostrará un modal **Model Information** (Información del modelo) con la definición de DTDL sin formato del modelo.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/models-panel-view.png" alt-text="Captura de pantalla del panel Models (Modelos) de Azure Digital Twins Explorer. Está resaltado el icono View Model (Ver modelo) de un único modelo." lightbox="media/how-to-use-azure-digital-twins-explorer/models-panel-view.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

También puede ver la definición completa de un modelo seleccionándolo en el panel **Model Graph** (Grafo de modelos) y usando el botón **Toggle model details** (Alternar detalles del modelo) para expandir el panel **MODEL DETAIL** (DETALLES DEL MODELO). Este panel también mostrará el código DTDL completo del modelo.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/model-graph-panel-highlight-graph-details.png" alt-text="Captura de pantalla del panel Model Graph (Grafo de modelos) de Azure Digital Twins Explorer. Está seleccionado el modelo Floor y el panel MODEL DETAIL (DETALLES DEL MODELO) está expandido, mostrando el código DTDL del modelo." lightbox="media/how-to-use-azure-digital-twins-explorer/model-graph-panel-highlight-graph-details.png":::

### <a name="edit-model-graph-layout"></a>Edición del diseño del grafo de modelos

Para reorganizar los modelos en distintas configuraciones, haga clic en ellos y arrástrelos por la pantalla Model Graph (Grafo de modelos).

También puede aplicar uno de varios algoritmos de diseño al grafo de modelos con las opciones del menú **Run Layout** (Ejecutar diseño). 

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/model-graph-panel-layout.png" alt-text="Captura de pantalla del panel Model Graph (Grafo de modelos) de Azure Digital Twins Explorer. El botón Run Layout (Ejecutar diseño) está resaltado y muestra un menú con las opciones de diseño Cola, Dagre, fCoSE, Klay y d3Force." lightbox="media/how-to-use-azure-digital-twins-explorer/model-graph-panel-layout.png":::

### <a name="filter-and-highlight-model-graph-elements"></a>Filtrado y resaltado de elementos del grafo de modelos

Puede filtrar los tipos de conexiones que aparecen en el panel Model Graph (Grafo de modelos). Si desactiva uno de los tipos de conexión con los conmutadores de este menú, ese tipo no se mostrará en el grafo.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/model-graph-panel-filter-connections.png" alt-text="Captura de pantalla del panel Model Graph (Grafo de modelos) de Azure Digital Twins Explorer. Está resaltado el menú de filtro con las opciones Relationships (Relaciones), Inheritance (Herencia) y Components (Componentes)." lightbox="media/how-to-use-azure-digital-twins-explorer/model-graph-panel-filter-connections.png":::

También puede filtrar los modelos y las conexiones que aparecen en el grafo por texto; para ello, seleccione este icono de **filtro**:

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/model-graph-panel-filter-text.png" alt-text="Captura de pantalla del panel Model Graph (Grafo de modelos) de Azure Digital Twins Explorer. Está seleccionado el icono de filtro de texto, que muestra la pestaña Filter (Filtro) donde puede escribir un término de búsqueda." lightbox="media/how-to-use-azure-digital-twins-explorer/model-graph-panel-filter-text.png":::

Puede asimismo resaltar los modelos y las conexiones que aparecen en el grafo por texto; para ello, seleccione este icono de **resaltado**:

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/model-graph-panel-highlight-text.png" alt-text="Captura de pantalla del panel Model Graph (Grafo de modelos) de Azure Digital Twins Explorer. Está seleccionado el icono de filtro de texto, que muestra la pestaña Highlight (Resaltar) donde puede escribir un término de búsqueda." lightbox="media/how-to-use-azure-digital-twins-explorer/model-graph-panel-highlight-text.png":::

### <a name="upload-model-images"></a>Carga de imágenes de modelo

Puede cargar imágenes personalizadas para representar diferentes modelos en las vistas Model Graph (Grafo de modelos) y [Twin Graph](#explore-the-twin-graph) (Grafo de gemelos). Puede cargar imágenes para modelos individuales o para varios modelos a la vez.

>[!NOTE]
>Estas imágenes se guardan en el almacenamiento del explorador local. Esto significa que no estarán disponibles en otros exploradores distintos de aquel en el que las guardó y que permanecerán en el almacenamiento del explorador indefinidamente hasta que se borre el almacenamiento local.

Para cargar una imagen para un único modelo, busque ese modelo en el panel **Models** (Modelos) y elija el icono **Upload Model Image** (Cargar imagen de modelo) junto al nombre del modelo. En el cuadro de selección de archivos que aparece, vaya en la máquina al archivo de imagen que desea cargar para ese modelo. Elija **Open** (Abrir) para cargarlo.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/models-panel-upload-one-image.png" alt-text="Captura de pantalla del panel Models (Modelos) de Azure Digital Twins Explorer. Está resaltado el icono Upload Model Image (Cargar imagen de modelo) de un único modelo." lightbox="media/how-to-use-azure-digital-twins-explorer/models-panel-upload-one-image.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

También puede cargar imágenes de modelo de forma masiva.

En primer lugar, use las instrucciones siguientes para establecer los nombres de los archivos de imagen antes de cargarlos. Esto permite que Azure Digital Twins Explorer asigne automáticamente las imágenes a los modelos correctos después de la carga. 
1. Comience con el valor de identificador del modelo (por ejemplo, `dtmi:example:Floor;1`).
1. Reemplace las instancias de ":" por "_" (el ejemplo se convierte en `dtmi_example_Floor;1`).
1. Reemplace las instancias de ";" por "-" (el ejemplo se convierte en `dtmi_example_Floor-1`).
1. Asegúrese de que el archivo tiene una extensión de imagen (el ejemplo se convierte en algo parecido a `dtmi_example_Floor-1.png`).

> [!NOTE]
> Si intenta cargar una imagen que no se asigna a ningún modelo existente con los criterios anteriores, la imagen no se cargará ni se almacenará.

A continuación, para cargar las imágenes al mismo tiempo, use el icono **Upload Model Images** (Cargar imágenes de modelo) en la parte superior del panel Models (Modelos). En el cuadro de selección de archivos, elija los archivos de imagen que desea cargar. 

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/models-panel-upload-images.png" alt-text="Captura de pantalla del panel Models (Modelos) de Azure Digital Twins Explorer. Está resaltado el icono Upload Model Images (Cargar imágenes de modelo)." lightbox="media/how-to-use-azure-digital-twins-explorer/models-panel-upload-images.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

## <a name="manage-models"></a>Administración de modelos

Puede usar el panel **Models** (Modelos) en el lado izquierdo de la pantalla Azure Digital Twins Explorer para realizar actividades de administración en todo el conjunto de modelos o en modelos individuales. 

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/models-panel.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer. El panel Models (Modelos) está resaltado." lightbox="media/how-to-use-azure-digital-twins-explorer/models-panel.png":::

Con este panel, puede completar las siguientes actividades de administración de modelos:
* [Carga de modelos](#upload-models) en la instancia de Azure Digital Twins
* [Eliminación de modelos](#delete-models) de la instancia
* [Actualización de modelos](#refresh-models) para volver a cargar la lista de todos los modelos en este panel

Para obtener información sobre la experiencia de visualización de modelos, consulte [Exploración de modelos y del panel Model Graph (Grafo de modelos)](#explore-models-and-the-model-graph).

### <a name="upload-models"></a>Carga de modelos

Puede cargar modelos de la máquina seleccionándolos individualmente o cargando una carpeta completa de modelos a la vez.

Para cargar uno o varios modelos seleccionados individualmente, seleccione el icono **Upload a model** (Cargar un modelo) que muestra una flecha que apunta hacia una nube.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/models-panel-upload.png" alt-text="Captura de pantalla del panel Models (Modelos) de Azure Digital Twins Explorer. Está resaltado el icono Upload a model (Cargar un modelo)." lightbox="media/how-to-use-azure-digital-twins-explorer/models-panel-upload.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

En el cuadro de selección de archivos que aparece, vaya en la máquina a los modelos que desea cargar. Puede seleccionar uno o varios archivos de modelo JSON; después, seleccione **Open** (Abrir) para cargarlos.

Para cargar una carpeta de modelos, incluido todo lo que contiene, seleccione el icono **Upload a directory of models** (Cargar un directorio de modelos) que muestra una carpeta de archivos.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/models-panel-upload-directory.png" alt-text="Captura de pantalla del panel Models (Modelos) de Azure Digital Twins Explorer. Está resaltado el icono Upload a directory of models (Cargar un directorio de modelos)." lightbox="media/how-to-use-azure-digital-twins-explorer/models-panel-upload-directory.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

En el cuadro de selección de archivos que aparece, vaya en la máquina a una carpeta que contenga archivos de modelo JSON. Seleccione **Open** (Abrir) para cargar esa carpeta de nivel superior y todo su contenido.

>[!IMPORTANT]
>Si un modelo hace referencia a otro modelo en su definición, como cuando se definen relaciones o componentes, el modelo al que se hace referencia debe estar presente en la instancia para cargar el modelo que lo usa. Si va a cargar modelos uno a uno, esto significa que debe cargar el modelo al que se hace referencia **antes** de cargar los modelos que lo usen. Si va a cargar modelos de forma masiva, puede seleccionarlos en la misma importación y Azure Digital Twins deducirá el orden en que se cargarán.

### <a name="delete-models"></a>Eliminación de modelos

Puede usar el panel Models (Modelos) para eliminar modelos individuales o todos los modelos de la instancia a la vez.

Para eliminar un único modelo, busque ese modelo en la lista y elija el icono **Delete Model** (Eliminar modelo) junto al nombre del modelo.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/models-panel-delete-one.png" alt-text="Captura de pantalla del panel Models (Modelos) de Azure Digital Twins Explorer. Está resaltado el icono Delete Model (Eliminar modelo) de un único modelo." lightbox="media/how-to-use-azure-digital-twins-explorer/models-panel-delete-one.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Para eliminar todos los modelos de la instancia a la vez, elija el icono **Delete All Models** (Eliminar todos los modelos) en la parte superior del panel Models (Modelos).

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/models-panel-delete-all.png" alt-text="Captura de pantalla del panel Models (Modelos) de Azure Digital Twins Explorer. Está resaltado el icono Delete All Models (Eliminar todos los modelos)." lightbox="media/how-to-use-azure-digital-twins-explorer/models-panel-delete-all.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

### <a name="refresh-models"></a>Actualización de modelos

Al abrir Azure Digital Twins Explorer, el panel Models (Modelos) debe mostrar automáticamente todos los modelos disponibles en el entorno. 

Sin embargo, puede actualizar manualmente el panel en cualquier momento para volver a cargar la lista de todos los modelos de la instancia de Azure Digital Twins. Para ello, seleccione el icono **Refresh models** (Actualizar modelos) que tiene una flecha que apunta hacia abajo desde una nube. 

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/models-panel-refresh.png" alt-text="Captura de pantalla del panel Models (Modelos) de Azure Digital Twins Explorer. Está resaltado el icono Refresh Models (Actualizar modelos)." lightbox="media/how-to-use-azure-digital-twins-explorer/models-panel-refresh.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

## <a name="importexport-graph"></a>Importación y exportación de grafos

En el panel **Twin Graph** (Grafo de gemelos), tiene opciones para [importar](#import-graph) y [exportar](#export-graph-and-models) características de grafo.

### <a name="import-graph"></a>Importación de grafos

Puede usar la característica de importación para agregar gemelos, relaciones y modelos a la instancia. Esto puede ser útil para crear muchos gemelos, relaciones o modelos a la vez.

#### <a name="create-import-file"></a>Creación de un archivo de importación

El primer paso para importar un grafo es crear un archivo que represente los gemelos y las relaciones que desee agregar.

El archivo de importación puede estar en cualquiera de estos dos formatos:
* El **formato personalizado basado en Excel** que se describe en el resto de esta sección. Este formato le permite cargar gemelos y relaciones.
* El **formato basado en JSON** generado al [exportar grafos](#export-graph-and-models). Este formato puede contener gemelos, relaciones o modelos.

Para crear un grafo personalizado en Excel, use el formato siguiente.

Cada fila representa un elemento que se debe crear: un gemelo, una relación o una combinación de gemelo y la relación correspondiente.
Use las columnas siguientes para estructurar los datos de los gemelos o las relaciones. Los nombres de columna se pueden personalizar, pero deben permanecer en este orden.

| ModelID | ID | Relationship (source) (Relación [origen]) | Relationship Name (Nombre de la relación) | Init Data (Datos de inicialización) |
| --- | --- | --- | --- | --- |
| *Opcional*<br>Identificador del modelo DTMI para un gemelo que se debe crear.<br><br>Puede dejar esta columna en blanco en una fila si desea que esa fila solo cree una relación (sin gemelos). | *Obligatorio*<br>Identificador único de un gemelo.<br><br>Si se crea un gemelo en esta fila, será el identificador del nuevo gemelo.<br>Si hay información de relación en la fila, este identificador se usará como **destino** de la relación. | *Opcional*<br>Identificador del gemelo de **origen** de una nueva relación.<br><br>Puede dejar esta columna en blanco en una fila si desea que esa fila solo cree un gemelo (sin relaciones). | *Opcional*<br>Nombre de la nueva relación que se creará. La dirección de la relación irá **del** gemelo de la columna C **al** gemelo de la columna B. | *Opcional*<br>Cadena JSON que contiene la configuración de propiedades del gemelo que se va a crear. Las propiedades deben coincidir con las definidas en el modelo de la columna A. |

Este es un archivo .xlsx de ejemplo que crea un pequeño grafo de dos plantas y dos salas.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/import-example.png" alt-text="Captura de pantalla de los datos del grafo en Excel. Los encabezados de columna corresponden a los campos anteriores, por orden, y las filas contienen los valores de datos correspondientes." lightbox="media/how-to-use-azure-digital-twins-explorer/import-example.png":::

Puede ver este archivo y otros ejemplos de grafo .xlsx en el [repositorio de Azure Digital Twins Explorer](https://github.com/Azure-Samples/digital-twins-explorer/tree/main/client/examples) en GitHub.

>[!NOTE]
>Las propiedades y relaciones descritas en el archivo .xlsx deben coincidir con lo que se establece en las definiciones de modelo de los gemelos relacionados.

#### <a name="import-file-to-azure-digital-twins-explorer"></a>Importación de un archivo a Azure Digital Twins Explorer

Una vez que tenga un archivo en la máquina que esté listo para importarse, seleccione el icono **Import Graph** (Importar grafo) en el panel Twin Graph (Grafo de gemelos).

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-import.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El botón Import Graph (Importar grafo) está resaltado." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-import.png":::

En el cuadro de selección de archivos que aparece, navegue en la máquina hasta el archivo de grafo (.xlsx o .json) que desea cargar y elija **Open** (Abrir) para cargarlo.

Azure Digital Twins abrirá un panel **Import** (Importar) que muestra una vista previa del grafo que se va a importar. Para confirmarlo, seleccione el icono **Save** (Guardar) en la esquina superior derecha del panel.

Si la importación se realiza correctamente, una ventana modal mostrará el número de modelos, gemelos y relaciones que se cargaron.

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-import-successful.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El centro de la pantalla muestra un modal de importación correcta con 4 gemelos importados y 2 relaciones importadas." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-import-successful.png":::

### <a name="export-graph-and-models"></a>Exportación de grafos y modelos

Puede usar la característica de exportación para exportar grafos parciales o completos, incluidos los modelos, los gemelos y las relaciones. La exportación serializa los gemelos y las relaciones de los resultados de la consulta más recientes, así como de todos los modelos de la instancia, a un formato basado en JSON que puede descargar en la máquina.

Para empezar, use el panel [Query Explorer ](#query-your-digital-twin-graph) (Explorador de consultas) para ejecutar una consulta que seleccione los gemelos y las relaciones que desea descargar. Esto los incluirá en el panel Twin Graph (Grafo de gemelos).

>[!TIP]
>La consulta para mostrar todos los gemelos y relaciones es `SELECT * FROM digitaltwins`.

Una vez que el panel Twin Graph (Grafo de gemelos) muestre la parte del grafo que desea descargar, seleccione el icono **Export Graph** (Exportar grafo).

:::image type="content" source="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-export.png" alt-text="Captura de pantalla del panel Twin Graph (Grafo de gemelos) de Azure Digital Twins Explorer. El botón Export Graph (Exportar grafo) está resaltado." lightbox="media/how-to-use-azure-digital-twins-explorer/twin-graph-panel-export.png":::

Esta acción habilita un vínculo de **descarga** en el cuadro Twin Graph (Grafo de gemelos). Selecciónelo para descargar una representación basada en JSON del resultado de la consulta y todos los modelos de la instancia en la máquina.

>[!TIP]
>Este archivo se puede editar o se puede volver a cargar en Azure Digital Twins con la característica de [importación](#import-graph).

## <a name="link-to-your-environment"></a>Vínculo al entorno

Puede compartir su entorno de Azure Digital Twins Explorer con otros usuarios para colaborar en el trabajo. En esta sección se describe cómo enviar el entorno de Azure Digital Twins Explorer a otra persona y comprobar que tiene los permisos para acceder a él.

Para compartir el entorno, puede enviar un vínculo al destinatario que abrirá una ventana Azure Digital Twins Explorer conectada a la instancia. Use el vínculo siguiente y reemplace los marcadores de posición por el **identificador de inquilino** y el **nombre de host** de su instancia de Azure Digital Twins. 

`https://explorer.digitaltwins.azure.net/?tid=<tenant-ID>&eid=<Azure-Digital-Twins-host-name>`

>[!NOTE]
> El valor del marcador de posición de nombre de host **no** va precedido de *https://* aquí.

Este es un ejemplo de una dirección URL con los valores de marcador de posición rellenados:

`https://explorer.digitaltwins.azure.net/?tid=00a000aa-00a0-00aa-0a0aa000aa00&eid=ADT-instance.api.wcus.digitaltwins.azure.net`

Para que el destinatario vea la instancia en la ventana de Azure Digital Twins Explorer resultante, debe iniciar sesión en su cuenta de Azure y tener acceso con el rol **Azure Digital Twins Data Reader** (Lector de datos de Azure Digital Twins) a la instancia (puede obtener más información sobre los roles de Azure Digital Twins en [Conceptos: seguridad](concepts-security.md)). Para que el destinatario realice cambios en el grafo y los datos, debe tener el rol **Azure Digital Twins Data Owner** (Propietario de datos de Azure Digital Twins) en la instancia.

### <a name="link-with-a-query"></a>Vínculo con una consulta

Es posible que quiera compartir un entorno y especificar una consulta para ejecutarla al acceder a él, para resaltar un subgrafo o una vista personalizada para un compañero de equipo. Para ello, comience con la dirección URL del entorno y agregue el texto de la consulta a ella como parámetro de una cadena de consulta:

`https://explorer.digitaltwins.azure.net/?tid=<tenant-ID>&eid=<Azure-Digital-Twins-host-name>&query=<query-text>`

El texto de la consulta debe estar codificado como URL. 

>[!TIP]
>Puede copiar el texto de la consulta de la ventana **Query Explorer** (Explorador de consultas) y pegarlo en la ventana de URL en el punto correcto al final de la dirección URL. El envío de esta URL debe convertir el texto de la consulta a la codificación de dirección URL adecuada.
>
> También puede usar un codificador de URL independiente para convertir el texto de la consulta.

Este es un ejemplo del parámetro de una consulta para seleccionar con **SELECT * FROM digitaltwins**:

`...&query=SELECT%20*%20FROM%20digitaltwins`

Después puede compartir la URL completa.

## <a name="advanced-settings"></a>Configuración avanzada

Puede habilitar varias opciones de configuración avanzada para Azure Digital Twins Explorer.

Al hacer clic en el engranaje de configuración de la esquina superior derecha, se accede a la configuración de las siguientes características avanzadas:
* **Eager Loading** (Carga diligente): *se accede desde el engranaje **Settings** (Configuración) de la barra de herramientas superior.* Cuando una consulta devuelve gemelos que tienen relaciones con otros gemelos que **no** están incluidos en los resultados de la consulta, esta característica cargará los gemelos "ausentes" antes de representar el grafo.
* **Caching** (Almacenamiento en caché): *se accede desde el engranaje **Settings** (Configuración) de la barra de herramientas superior.* Cuando esta característica está habilitada, Azure Digital Twins Explorer mantiene una caché local de relaciones y modelos en memoria para mejorar el rendimiento de las consultas. Estas memorias caché se borran con cualquier operación de escritura en los elementos correspondientes, así como con la actualización del explorador.
* **Console** (Consola): *se accede desde el engranaje **Settings** (Configuración) de la barra de herramientas superior.* Esta característica permite mostrar una ventana de consola, donde se pueden usar funciones de shell sencillas para trabajar con el grafo.
* **Output** (Salida): *se accede desde el engranaje **Settings** (Configuración) de la barra de herramientas superior.* Esta característica permite mostrar una ventana de salida, que incluye un seguimiento de diagnóstico de las operaciones.
* **Personalizar el diseño del panel**: puede editar la posición de los paneles que conforman Azure Digital Twins Explorer (Query Explorer, Models, Twin Graph, Model Graph [Explorador de consultas, Modelos, Grafo de gemelos, Grafo de modelos]). Para mover un panel a otra ubicación, haga clic y mantenga presionado el nombre del panel y arrástrelo a la nueva posición deseada.

    :::image type="content" source="media/how-to-use-azure-digital-twins-explorer/panels.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer. Están resaltados los nombres de los paneles Query Explorer (Explorador de consultas), Models (Modelos), Twin Graph (Grafo de gemelos) y Model Graph (Grafo de modelos)." lightbox="media/how-to-use-azure-digital-twins-explorer/panels.png":::

    Las posiciones del panel se restablecerán tras la actualización de la ventana del explorador.

## <a name="next-steps"></a>Pasos siguientes 

Obtenga información sobre cómo escribir consultas para el grafo de gemelos de Azure Digital Twins: 
* [Conceptos: Lenguaje de consulta](concepts-query-language.md)
* [Procedimiento: Consulta del grafo de gemelos](how-to-query-graph.md)