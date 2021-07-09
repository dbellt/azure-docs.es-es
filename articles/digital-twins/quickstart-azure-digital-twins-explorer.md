---
title: 'Inicio rápido: introducción a Azure Digital Twins Explorer'
titleSuffix: Azure Digital Twins
description: 'Inicio rápido: Uso del ejemplo de Azure Digital Twins Explorer para visualizar y explorar un escenario precompilado.'
author: baanders
ms.author: baanders
ms.date: 4/27/2021
ms.topic: quickstart
ms.service: digital-twins
ms.openlocfilehash: d4315ff04235e4b555a201e9da7287339b0de125
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110450815"
---
# <a name="quickstart---get-started-with-a-sample-scenario-in-azure-digital-twins-explorer"></a>Inicio rápido: introducción a un escenario de ejemplo de Azure Digital Twins Explorer

Con Azure Digital Twins, puede crear modelos en directo de los entornos del mundo real e interactuar con ellos. En primer lugar, los elementos individuales se modelan como **gemelos digitales**. A continuación, se conectan en un **grafo** de conocimiento que puede dar respuesta a eventos en directo y en el que se puede consultar información.

En este inicio rápido, explorará un grafo de Azure Digital Twins precompilado con la ayuda de [Azure Digital Twins Explorer](concepts-azure-digital-twins-explorer.md). Se trata de una herramienta que permite visualizar e interactuar con los datos Azure Digital Twins en Azure Portal.

Deberá completar los pasos siguientes:

1. Creación de una instancia de Azure Digital Twins y conexión a esta en Azure Digital Twins Explorer.
1. Carga de los modelos precompilados y los datos del grafo para construir el escenario de ejemplo.
1. Exploración del grafo del escenario que se crea.
1. Realización de cambios en el grafo.
1. Revisión de los aprendizajes de la experiencia.

El grafo de ejemplo con el que va a trabajar representa un edificio de dos plantas y dos salas. Floor0 contiene Room0 y Floor1 contiene Room1. El grafo será como el de esta imagen:

:::image type="content" source="media/quickstart-azure-digital-twins-explorer/graph-view-full.png" alt-text="Captura de pantalla de un grafo formado por cuatro nodos circulares conectados por flechas en Azure Digital Twins Explorer.":::

## <a name="prerequisites"></a>Requisitos previos

Necesitará una suscripción de Azure para completar esta guía de inicio rápido. Si aún no tiene una, [cree una gratis](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) .

También deberá descargar los materiales del grafo de ejemplo usado en el inicio rápido. Use los vínculos y las instrucciones siguientes para descargar los tres archivos necesarios desde el repositorio [digital-twins-explorer de GitHub](https://github.com/Azure-Samples/digital-twins-explorer).
* [Room.json](https://raw.githubusercontent.com/Azure-Samples/digital-twins-explorer/main/client/examples/Room.json): vaya al vínculo, haga clic con el botón derecho en cualquier lugar de la pantalla y seleccione **Guardar como** en el menú contextual del explorador. Use la siguiente ventana Guardar como para guardar el archivo en algún lugar del equipo con el nombre **Room.json**.
* [Floor.json](https://raw.githubusercontent.com/Azure-Samples/digital-twins-explorer/main/client/examples/Floor.json): vaya al vínculo, haga clic con el botón derecho en cualquier lugar de la pantalla y seleccione **Guardar como** en el menú contextual del explorador. Use la siguiente ventana Guardar como para guardar el archivo en la misma ubicación que el archivo **Room.json**, con el nombre **Floor.json**.
* [buildingScenario.xlsx](https://github.com/Azure-Samples/digital-twins-explorer/blob/main/client/examples/buildingScenario.xlsx): vaya al vínculo y seleccione el botón **Descargar**. El archivo se descargará en la ubicación de descarga predeterminada.

    :::image type="content" source="media/quickstart-azure-digital-twins-explorer/download-building-scenario.png" alt-text="Captura de pantalla del archivo digital-twins-explorer/client/examples/buildingScenario.xlsx en GitHub. El botón Descargar está resaltado." lightbox="media/quickstart-azure-digital-twins-explorer/download-building-scenario.png":::

## <a name="set-up-azure-digital-twins-and-azure-digital-twins-explorer"></a>Configuración de Azure Digital Twins y Azure Digital Twins Explorer

El primer paso para trabajar con Azure Digital Twins es configurar una instancia de Azure Digital Twins. Después de crear una instancia del servicio, puede conectarse a la instancia de Azure Digital Twins Explorer y rellenarla con los datos de ejemplo más adelante en el inicio rápido.

El resto de esta sección le guía a través de estos pasos.

### <a name="set-up-an-azure-digital-twins-instance"></a>Configurará una instancia de Azure Digital Twins.

Para trabajar con Azure Digital Twins en este artículo, antes es preciso *configurar una instancia de Azure Digital Twins* y los permisos necesarios para usarla.

Siga las instrucciones que se indican en [Configuración de una instancia y autenticación](how-to-set-up-instance-portal.md). Las instrucciones contienen información que le ayudará a comprobar que ha completado cada paso correctamente.

### <a name="open-instance-in-azure-digital-twins-explorer"></a>Abra la instancia en Azure Digital Twins Explorer.

A continuación, abra Azure Digital Twins Explorer para su instancia en [Azure Portal](https://portal.azure.com). 

Para hacerlo, vaya a Azure Portal y escriba el nombre de su nueva instancia de Azure Digital Twins en la barra de búsqueda para encontrarla.

A continuación, seleccione el botón **Azure Digital Twins Explorer (versión preliminar)** .

:::image type="content" source="media/includes/azure-digital-twins-explorer-portal-access.png" alt-text="Captura de pantalla de Azure Portal que muestra la página Información general de una instancia de Azure Digital Twins. Se ha resaltado el contorno del botón Abrir Azure Digital Twins Explorer (versión preliminar)." lightbox="media/includes/azure-digital-twins-explorer-portal-access.png":::

Se abrirá una ventana Azure Digital Twins Explorer conectada a la instancia.

:::image type="content" source="media/quickstart-azure-digital-twins-explorer/explorer-blank.png" alt-text="Captura de pantalla Azure Digital Twins Explorer en un explorador de Internet." lightbox="media/quickstart-azure-digital-twins-explorer/explorer-blank.png":::

## <a name="add-the-sample-data"></a>Adición de los datos de ejemplo

A continuación, importará el escenario de ejemplo y el grafo en Azure Digital Twins Explorer. El escenario de ejemplo se encuentra en la carpeta **digital-twins-explorer-main** que descargó en la sección [Requisitos previos](#prerequisites).

### <a name="models"></a>Modelos

El primer paso en una solución de Azure Digital Twins es definir el vocabulario del entorno. Creará [modelos](concepts-models.md) personalizados que describan los tipos de entidad que existen en el entorno.

Cada modelo se escribe en un lenguaje de tipo JSON-LD, conocido como lenguaje de definición de gemelos digitales (DTDL). Cada modelo describe un único tipo de entidad en lo referente a sus **propiedades**, **telemetría**, **relaciones** y **componentes**. Más adelante, usará estos modelos como base para los gemelos digitales que representan instancias específicas de estos tipos.

Normalmente, cuando se crea un modelo, se realizan tres pasos:

1. Escribir la definición del modelo. En el inicio rápido, este paso ya se ha hecho como parte de la solución de ejemplo.
1. Validarlo para asegurarse de que la sintaxis sea precisa. En el inicio rápido, este paso ya se ha hecho como parte de la solución de ejemplo.
1. Cargarlo en la instancia de Azure Digital Twins.
 
En este inicio rápido, los archivos del modelo ya se han escrito y validado. Están incluidos en la solución que descargó. En esta sección, se cargarán los dos modelos escritos previamente en la instancia para definir estos componentes de un entorno de edificio:

* Floor
* Sala

#### <a name="upload-models"></a>Carga de modelos

Siga estos pasos para cargar los modelos.

1. En el panel **Modelos**, seleccione el icono **Upload a Model** (Cargar un modelo) que muestra una flecha que apunta a una nube.

   :::image type="content" source="media/quickstart-azure-digital-twins-explorer/upload-model.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer, con el panel Modelos resaltado y el icono &quot;Cargar un modelo&quot;." lightbox="media/quickstart-azure-digital-twins-explorer/upload-model.png":::
 
1. En la ventana Abrir que aparece, vaya a la que contiene los archivos **Room.json** y **Floor.json** que descargó anteriormente.
1. Seleccione **Room.json** y **Floor.json**, y seleccione **Abrir** para cargarlos. 

Azure Digital Twins Explorer cargará estos archivos del modelo en su instancia de Azure Digital Twins. Deberían aparecer en el panel **Modelos** con los nombres descriptivos y los identificadores del modelo completos. Puede seleccionar los iconos de información **View Model** (Ver modelo) para ver el código DTDL subyacente.

:::row:::
    :::column:::
        :::image type="content" source="media/quickstart-azure-digital-twins-explorer/model-info.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer que muestra el panel Modelos con dos definiciones de modelo, Floor y Room." lightbox="media/quickstart-azure-digital-twins-explorer/model-info.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

### <a name="twins-and-the-twin-graph"></a>Gemelos y grafo de gemelos

Ahora que se han cargado algunos modelos en la instancia de Azure Digital Twins, puede crear [gemelos digitales](concepts-twins-graph.md) basados en las definiciones de modelo.

Los gemelos digitales representan las entidades reales dentro del entorno empresarial. Pueden ser cosas como sensores de una granja, luces de un coche o, en este inicio rápido, salas de la planta de un edificio. Puede crear muchos gemelos de un tipo de modelo determinado, como varias salas que usen todas el modelo Room. Estos gemelos se conectan mediante relaciones en un **grafo de gemelos** que representa todo el entorno.

En esta sección, cargará los gemelos creados previamente que están conectados en un grafo creado previamente. El grafo contiene dos plantas y dos salas, conectadas en el siguiente diseño:

* Floor0
    - Contains Room0
* Floor1
    - Contains Room1

#### <a name="import-the-graph"></a>Importación de grafo

Siga estos pasos para importar el grafo.

1. En el panel **Twin Graph** (Grafo gemelo), seleccione el icono **Import Graph** (Importar grafo) que muestra una flecha que apunta a una nube.

   :::image type="content" source="media/quickstart-azure-digital-twins-explorer/import-graph.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer muestra el panel Vista gráfico, con el icono &quot;Importar gráfico&quot; resaltado." lightbox="media/quickstart-azure-digital-twins-explorer/import-graph.png":::

2. En la ventana Abrir, vaya al archivo **buildingScenario.xlsx** que descargó anteriormente. Este archivo contiene una descripción del grafo de ejemplo. seleccione **Open**(Abrir).

   Al cabo de unos segundos, Azure Digital Twins Explorer abre una vista **Import** (Importar) que muestra una vista previa del grafo que se va a cargar.

3. Para confirmar la carga del grafo, seleccione el icono **Guardar** situado en la esquina superior derecha del cuadro panel de vista previa del grafo.

   :::row:::
    :::column:::
        :::image type="content" source="media/quickstart-azure-digital-twins-explorer/graph-preview-save.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer con el icono Guardar del panel de vista previa del grafo resaltado." lightbox="media/quickstart-azure-digital-twins-explorer/graph-preview-save.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
   :::row-end:::

4. Azure Digital Twins Explorer usará el archivo cargado para crear los gemelos solicitados y las relaciones entre ellos. Al terminar, aparecerá un cuadro de diálogo. Seleccione **Close** (Cerrar).

   :::row:::
    :::column:::
        :::image type="content" source="media/quickstart-azure-digital-twins-explorer/import-success.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer que muestra un cuadro de diálogo que indica que la importación del grafo se ha realizado correctamente." lightbox="media/quickstart-azure-digital-twins-explorer/import-success.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
   :::row-end:::

5. El grafo ya se ha cargado en Azure Digital Twins Explorer. Vuelva al panel **Twin Graph** (Grafo gemelo).
 
   :::image type="content" source="media/quickstart-azure-digital-twins-explorer/twin-graph-tab.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer con la pestaña Grafo gemelo resaltada." lightbox="media/quickstart-azure-digital-twins-explorer/twin-graph-tab.png"::: 
 
6. Para ver el grafo, seleccione el botón **Ejecutar consulta** en el panel **Explorador de consultas**, cerca de la parte superior de la ventana de Azure Digital Twins Explorer.

   :::image type="content" source="media/quickstart-azure-digital-twins-explorer/run-query.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer con el botón &quot;Ejecutar consulta&quot; resaltado en la esquina superior derecha de la ventana." lightbox="media/quickstart-azure-digital-twins-explorer/run-query.png":::

Esta acción ejecuta la consulta predeterminada para seleccionar y mostrar todos los gemelos digitales. Azure Digital Twins Explorer recupera todos los gemelos y relaciones del servicio, y dibuja el grafo definido por ellos en el panel **Twin Graph** (Grafo gemelo).

## <a name="explore-the-graph"></a>Exploración del grafo

Ahora puede ver el grafo cargado del escenario de ejemplo.

:::image type="content" source="media/quickstart-azure-digital-twins-explorer/graph-view-full.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer que muestra el panel Vista de gráfico con un grafo gemelo.":::

Los círculos ("nodos" del grafo) representan los gemelos digitales. Las líneas representan las relaciones. El gemelo Floor0 contiene Room0, y el gemelo Floor1 contiene Room1.

Si usa un mouse, puede arrastrar trozos del grafo para moverlos.

### <a name="view-twin-properties"></a>Visualización de las propiedades de un gemelo

Puede seleccionar un gemelo para ver una lista de sus propiedades y valores en el panel **Propiedades**.

Estas son las propiedades de Room0:

:::row:::
    :::column:::
        :::image type="content" source="media/quickstart-azure-digital-twins-explorer/properties-room0.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer con el panel Propiedades resaltado, que muestra las propiedades $dtId, Temperatura y Humedad para Room0." lightbox="media/quickstart-azure-digital-twins-explorer/properties-room0.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Room0 tiene una temperatura de 70.

Estas son las propiedades de Room1:

:::row:::
    :::column:::
        :::image type="content" source="media/quickstart-azure-digital-twins-explorer/properties-room1.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer con el panel Propiedades resaltado, que muestra las propiedades $dtId, Temperatura y Humedad para Room1." lightbox="media/quickstart-azure-digital-twins-explorer/properties-room1.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Room1 tiene una temperatura de 80.

### <a name="query-the-graph"></a>Consulta del grafo

Una de las principales características de Azure Digital Twins es la posibilidad de [consultar](concepts-query-language.md) el gráfico de gemelos de forma fácil y eficaz para responder a las preguntas sobre el entorno.

Una forma de consultar los gemelos en el grafo es mediante sus **propiedades**. La consulta basada en propiedades puede ayudar a responder a diversas preguntas. Por ejemplo, puede encontrar valores atípicos en el entorno que necesiten atención.

En esta sección, ejecutará una consulta que responda a la pregunta de cuántos gemelos de su entorno tienen una temperatura superior a 75.

Para ver la respuesta, ejecute la siguiente consulta en el panel **Explorador de consultas**.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="TemperatureQuery":::

Recuerde que, al ver las propiedades de los gemelos anteriores, Room0 tiene una temperatura de 70 y Room1 tiene una temperatura de 80. Por lo tanto, aquí solo aparece Room1 en los resultados.
    
:::image type="content" source="media/quickstart-azure-digital-twins-explorer/result-query-property-before.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer con los resultados de la consulta de propiedades, que muestra solo Room1." lightbox="media/quickstart-azure-digital-twins-explorer/result-query-property-before.png":::

>[!TIP]
> También se admiten otros operadores de comparación (<,>, = o !=) en la consulta anterior. Puede intentar conectar estos operadores, otros valores u otras propiedades de los gemelos a la consulta para tratar de responder a sus propias preguntas.

## <a name="edit-data-in-the-graph"></a>Edición de los datos del grafo

Azure Digital Twins Explorer se puede usar para editar las propiedades de los gemelos representados en el grafo. En esta sección, se va a elevar la temperatura de Room0 a 76.

Para empezar, vuelva a ejecutar la consulta siguiente para seleccionar todos los gemelos digitales. Se mostrará el grafo completo una vez más en el panel **Twin Graph** (Grafo gemelo).

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="GetAllTwins":::

Seleccione **Room0** para mostrar su lista de propiedades en el panel **Propiedades**.

Las propiedades de esta lista son editables. Seleccione el valor de temperatura **70** para habilitar la entrada de un nuevo valor. Escriba **76** y seleccione el icono **Save** (Guardar) para actualizar la temperatura a **76**.

:::row:::
    :::column:::
        :::image type="content" source="media/quickstart-azure-digital-twins-explorer/new-properties-room0.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer con el panel Propiedades resaltado, que muestra las propiedades que se pueden editar para Room0." lightbox="media/quickstart-azure-digital-twins-explorer/new-properties-room0.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Ahora verá la ventana **Patch Information** (Información de revisión) que muestra el código de revisión que se usó en segundo plano con las [API](concepts-apis-sdks.md) de Azure Digital Twins para realizar la actualización. Seleccione **Close** (Cerrar).

### <a name="query-to-see-the-result"></a>Ejecución de la consulta para ver el resultado

Para comprobar que el grafo ha registrado correctamente la actualización de la temperatura de Room0, vuelva a ejecutar la consulta anterior para obtener todos los gemelos del entorno con una temperatura superior a 75.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="TemperatureQuery":::

Ahora que se ha cambiado la temperatura de Room0 de 70 a 76, ambos gemelos deberían aparecer en el resultado.

:::image type="content" source="media/quickstart-azure-digital-twins-explorer/result-query-property-after.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer con los resultados de la consulta de propiedades, que muestra Room0 y Room1." lightbox="media/quickstart-azure-digital-twins-explorer/result-query-property-after.png":::

## <a name="review-and-contextualize-learnings"></a>Revisión y contextualización del aprendizaje

En este inicio rápido, ha creado una instancia de Azure Digital Twins y utilizado Azure Digital Twins Explorer para rellenarla con un escenario de ejemplo.

Después, ha explorado el grafo mediante:

* El uso de una consulta para responder a una pregunta sobre el escenario.
* La edición de una propiedad en un gemelo digital.
* La ejecución de la consulta de nuevo para ver cómo ha cambiado la respuesta como resultado de la actualización.

La intención de este ejercicio es demostrar cómo puede usar el grafo de Azure Digital Twins para responder a preguntas sobre el entorno, incluso a medida que el entorno cambia.

En este inicio rápido, realizó la actualización de la temperatura manualmente. Es habitual en Azure Digital Twins conectar gemelos digitales a dispositivos IoT reales para que reciban actualizaciones automáticamente, en función de los datos de telemetría. De este modo, puede crear un grafo dinámico que siempre refleje el estado real del entorno. Puede usar consultas para obtener información sobre lo que ocurre en su entorno en tiempo real.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para la limpieza después de este inicio rápido, elija los recursos que desea quitar en función de lo que quiera hacer a continuación.

* **Si tiene previsto seguir con los tutoriales de Azure Digital Twins**, puede reutilizar la instancia de este inicio rápido en esos artículos, no es necesario eliminarla.

[!INCLUDE [digital-twins-cleanup-clear-instance.md](../../includes/digital-twins-cleanup-clear-instance.md)]
 
[!INCLUDE [digital-twins-cleanup-basic.md](../../includes/digital-twins-cleanup-basic.md)]

También puede que desee eliminar la carpeta del proyecto de ejemplo de la máquina local.

## <a name="next-steps"></a>Pasos siguientes

A continuación, siga con los tutoriales de Azure Digital Twins para crear su propio escenario de Azure Digital Twins y las herramientas de interacción.

> [!div class="nextstepaction"]
> [Tutorial: Programación de una aplicación cliente](tutorial-code.md)
