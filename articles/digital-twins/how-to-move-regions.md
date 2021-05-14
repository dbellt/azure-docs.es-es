---
title: Migración de una instancia a otra región de Azure
titleSuffix: Azure Digital Twins
description: Vea cómo migrar una instancia de Azure Digital Twins de una región de Azure a otra.
author: baanders
ms.author: baanders
ms.date: 08/26/2020
ms.topic: how-to
ms.custom: subject-moving-resources
ms.service: digital-twins
ms.openlocfilehash: b21915b6cf0cd018ccbf7b79fc9a5ab8dc222925
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108076132"
---
# <a name="move-an-azure-digital-twins-instance-to-a-different-azure-region"></a>Migración de una instancia de Azure Digital Twins a otra región de Azure

Si necesita mover una instancia de Azure Digital Twins de una región a otra, el proceso actual consiste en volver a crear los recursos en la región nueva y, después, eliminar los recursos originales. Al final de este proceso, trabajará con una instancia de Azure Digital Twins nueva que es idéntica a la primera, salvo que la ubicación está actualizada.

En este artículo se proporciona una guía para realizar una migración completa y copiar todo lo necesario para que la nueva instancia coincida con la original.

Este proceso incluye los siguientes pasos:

1. Preparación: descargue los modelos, gemelos y grafos originales.
1. Migración: cree una instancia de Azure Digital Twins en una nueva región.
1. Migración: vuelva a rellenar la nueva instancia de Azure Digital Twins.
    - Cargue los modelos, gemelos y grafos originales.
    - Vuelva a crear los puntos de conexión y las rutas.
    - Vuelva a vincular los recursos conectados.
1. Limpie los recursos de origen: Elimine la instancia original.

## <a name="prerequisites"></a>Requisitos previos

Antes de intentar volver a crear una instancia de Azure Digital Twins, revise los componentes de la instancia original para hacerse una idea clara de todas las partes que deben volver a crearse.

Estas son algunas preguntas que debe tener en cuenta:

* ¿Cuáles son los modelos *cargados* en mi instancia? ¿Cuántos hay?
* ¿Cuáles son los *gemelos* de mi instancia? ¿Cuántos hay?
* ¿Cuál es la forma general del *grafo* en mi instancia? ¿Cuántas relaciones hay?
* ¿Qué *puntos de conexión* tengo en mi instancia?
* ¿Qué *rutas* tengo en mi instancia? ¿Tienen filtros?
* ¿Dónde la instancia se *conecta a otros servicios de Azure*? Entre los puntos de integración comunes se incluyen:

    - Azure Event Grid, Azure Event Hubs o Azure Service Bus
    - Azure Functions
    - Azure Logic Apps
    - Azure Time Series Insights
    - Azure Maps
    - Azure IoT Hub Device Provisioning Service
* ¿Qué otras *aplicaciones personales o empresariales* tengo que conectar a mi instancia?

Para recopilar esta información se pueden usar [Azure Portal](https://portal.azure.com), las [API y los SDK de Azure Digital Twins](how-to-use-apis-sdks.md), los [ comandos de la CLI de Azure Digital Twins](how-to-use-cli.md) o el ejemplo de [Azure Digital Twins Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/).

## <a name="prepare"></a>Preparación

En esta sección, se preparará para volver a crear la instancia mediante la descarga de los modelos, gemelos y grafo originales de la instancia original. En este artículo se usa el ejemplo de [Azure Digital Twins Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/) para esta tarea.

>[!NOTE]
>Es posible que ya tenga en la instancia archivos que contengan los modelos o el grafo. Si es así, no es necesario volver a descargar, solo las partes que faltan o aquello que pueda haber cambiado desde que se cargaron originalmente estos archivos. Por ejemplo, puede que tenga gemelos que se hayan actualizado con nuevos datos.

### <a name="limitations-of-azure-digital-twins-explorer"></a>Limitaciones de Azure Digital Twins Explorer

El [ejemplo de Azure Digital Twins Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/) es un ejemplo de aplicación cliente que admite una representación visual del grafo y proporciona interacción visual con la instancia. En este artículo se muestra cómo usarlo para descargar y volver a cargar modelos, gemelos y grafos.

Este ejemplo no es una herramienta completa. No se ha realizado una prueba de esfuerzo y no se ha creado para manejar grafos de gran tamaño. Por tanto, tenga en cuenta las siguientes limitaciones del ejemplo estándar:

* Actualmente, el ejemplo solo se ha probado en tamaños de grafo de hasta 1000 nodos y 2000 relaciones.
* El ejemplo no admite reintentos si se producen errores intermitentes.
* El ejemplo no necesariamente enviará una notificación al usuario si los datos cargados están incompletos.
* El ejemplo no administra los errores resultantes de grafos muy grandes que superan los recursos disponibles, como la memoria.

Si el ejemplo no puede controlar el tamaño del grafo, se pueden usar otras herramientas de desarrollo de Azure Digital Twins para exportarlo e importarlo:

* [Comandos de la CLI de Azure Digital Twins](how-to-use-cli.md)
* [las API y los SDK de Azure Digital Twins](how-to-use-apis-sdks.md)

### <a name="set-up-the-azure-digital-twins-explorer-application"></a>Configuración de la aplicación Azure Digital Twins Explorer

Para continuar con Azure Digital Twins Explorer, descargue el código de la aplicación de ejemplo y prepárelo para ejecutarlo en su máquina.

Para obtener el ejemplo, vaya a [Azure Digital Twins Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/). Seleccione el botón **Browse code** (Examinar código) debajo del título, que le llevará al repositorio de GitHub del ejemplo. Seleccione el botón **Código** y **Descargar archivo ZIP** para descargar el ejemplo como un archivo *.ZIP* en la máquina.

:::image type="content" source="media/how-to-move-regions/download-repo-zip.png" alt-text="Captura de pantalla del repositorio digital-twins-explorer en GitHub. El botón Código está seleccionado y se muestra un pequeño cuadro de diálogo en el que el botón Descargar archivo ZIP está resaltado." lightbox="media/how-to-move-regions/download-repo-zip.png":::

Descomprima el archivo.

A continuación, instale y configure los permisos para Azure Digital Twins Explorer. Siga las instrucciones de la sección [Configuración de Azure Digital Twins y Azure Digital Twins Explorer](quickstart-azure-digital-twins-explorer.md#set-up-azure-digital-twins-and-azure-digital-twins-explorer) del inicio rápido de Azure Digital Twins. Esta sección le lleva por los siguientes pasos:

1. Configure una instancia de Azure Digital Twins. Puede omitir esta parte porque ya tiene una instancia.
1. Configure las credenciales locales de Azure para proporcionar acceso a su instancia.
1. Ejecute Azure Digital Twins Explorer y configúrelo para que se conecte a su instancia. Usará el *nombre de host* de la instancia original de Azure Digital Twins que va a mover.

Ahora tendrá en ejecución la aplicación de ejemplo de Azure Digital Twins Explorer en un explorador en la máquina. El ejemplo debe estar conectado a la instancia original de Azure Digital Twins.

:::image type="content" source="media/how-to-move-regions/explorer-blank.png" alt-text="Ventana del explorador que muestra una aplicación que se ejecuta en localhost:3000. La aplicación se llama Azure Digital Twins Explorer y contiene los cuadros para Query Explorer (Explorador de consultas), Models (Modelos), Twin Graph (Grafo gemelo) y Properties (Propiedades). Todavía no hay datos en pantalla." lightbox="media/how-to-move-regions/explorer-blank.png":::

Para comprobar la conexión, seleccione el botón **Run Query** (Ejecutar consulta) para ejecutar la consulta predeterminada que muestra todos los gemelos y relaciones del grafo en el cuadro **TWIN GRAPH** (GRAFO GEMELO).

:::image type="content" source="media/how-to-move-regions/run-query.png" alt-text="Se resalta el botón Run Query (Ejecutar consulta) de la esquina superior derecha de la ventana." lightbox="media/how-to-move-regions/run-query.png":::

Puede dejar Azure Digital Twins Explorer en ejecución, ya que lo volverá a usar en el artículo para volver a cargar estos elementos en la nueva instancia de la región de destino.

### <a name="download-models-twins-and-graph"></a>Descarga de modelos, gemelos y grafo

A continuación, descargue los modelos, los gemelos y el grafo de la solución en la máquina.

Para descargar todos estos elementos a la vez, asegúrese en primer lugar de que el grafo completo aparece en el cuadro **TWIN GRAPH VIEW** (GRAFO GEMELO). Si no aparece el grafo completo, vuelva a ejecutar la consulta predeterminada de `SELECT * FROM digitaltwins` en el cuadro **QUERY EXPLORER** (EXPLORADOR DE CONSULTAS).
 
Luego, seleccione el icono **Export graph** (Exportar grafo) en el cuadro **TWIN GRAPH** (GRAFO GEMELO).

:::image type="content" source="media/how-to-move-regions/export-graph.png" alt-text="Hay un icono resaltado en el cuadro Twin Graph (Grafo gemelo). Se muestra una flecha que apunta hacia fuera de una nube." lightbox="media/how-to-move-regions/export-graph.png":::

Esta acción habilita un vínculo **Descargar** en el cuadro **TWIN GRAPH** (GRAFO GEMELO). Selecciónelo para descargar una representación basada en JSON del resultado de la consulta, que incluye los modelos, los gemelos y las relaciones. Esta acción debería descargar un archivo .json en la máquina.

>[!NOTE]
>Si el archivo descargado parece tener una extensión de archivo diferente, pruebe a editar la extensión directamente y cámbiela por .json.

## <a name="move"></a>Mover

A continuación, completará el "movimiento" de la instancia. Para ello, creará una instancia en la región de destino. Y, después, la llenará con los datos y componentes de la instancia original.

### <a name="create-a-new-instance"></a>Creación de una instancia

En primer lugar, crear una instancia de Azure Digital Twins en la región de destino. Siga los pasos que se describen en [Procedimientos: configuración de una instancia y autenticación](how-to-set-up-instance-portal.md). Tenga en cuenta los siguientes puntos:

* Puede usar el mismo nombre para la nueva instancia *si* está en otro grupo de recursos. Si necesita usar el mismo grupo de recursos que contiene la instancia original, la nueva instancia necesitará su propio nombre distintivo.
* Cuando se le solicite una ubicación, escriba la nueva región de destino.

Una vez que haya completado este paso, necesitará el nombre de host de la nueva instancia para continuar con la configuración de los datos. Si no ha anotado el nombre de host durante la configuración, siga [estas instrucciones](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values) para obtenerlo ahora desde Azure Portal.

### <a name="repopulate-the-old-instance"></a>Volver a rellenar la instancia anterior

A continuación, configurará la nueva instancia para que sea una copia de la original.

#### <a name="upload-the-original-models-twins-and-graph-by-using-azure-digital-twins-explorer"></a>Carga de los modelos, los gemelos y el grafo originales mediante Azure Digital Twins Explorer

En esta sección, puede volver a cargar los modelos, los gemelos y el grafo en la nueva instancia. Si no tiene ninguno de ellos en la instancia original o no desea migrarlos a la nueva instancia, puede ir directamente a la [siguiente sección](#re-create-endpoints-and-routes).

De lo contrario, ejecute Azure Digital Twins Explorer para volver a la ventana del explorador y siga estos pasos.

##### <a name="connect-to-the-new-instance"></a>Conexión a la nueva instancia

Actualmente, Azure Digital Twins Explorer está conectado a la instancia original de Azure Digital Twins. Cambie la conexión para que apunte a la nueva instancia; para ello, seleccione el botón **Sign in** (Iniciar sesión) en la esquina superior derecha de la ventana.

:::image type="content" source="media/how-to-move-regions/sign-in.png" alt-text="Azure Digital Twins Explorer con el icono de inicio de sesión resaltado en la esquina superior derecha de la ventana. El icono muestra una silueta simple de una persona superpuesta con la silueta de una llave." lightbox="media/how-to-move-regions/sign-in.png":::

Reemplace la **dirección URL de ADT** para que se refleje la nueva instancia. Cambie este valor por *https://{nombre de host de nueva instancia}* .

Seleccione **Conectar**. Es posible que se le pida que vuelva a iniciar sesión con sus credenciales de Azure o que conceda a esta aplicación el consentimiento para la instancia.

##### <a name="upload-models-twins-and-graph"></a>Carga de modelos, gemelos y grafo

A continuación, cargue los componentes de la solución que descargó anteriormente en la nueva instancia.

Para cargar los modelos, los gemelos y el grafo, seleccione el icono **Import Graph** (Importar grafo) en el cuadro **TWIN GRAPH** (GRAFO GEMELO). Esta opción carga los tres componentes a la vez. Incluso carga los modelos que no se están usando actualmente en el grafo.

:::image type="content" source="media/how-to-move-regions/import-graph.png" alt-text="Hay un icono resaltado en el cuadro Twin Graph. Muestra una flecha que apunta a una nube." lightbox="media/how-to-move-regions/import-graph.png":::

En el cuadro se selección de archivos, vaya al grafo descargado. Seleccione el archivo **.json** del grafo y seleccione **Abrir**.

Al cabo de unos segundos, Azure Digital Twins Explorer abre una vista **Import** (Importar) que muestra una vista previa del grafo que se va a cargar.

Para confirmar la carga del grafo, seleccione el icono **Save** (Guardar) situado en la esquina superior derecha del cuadro **TWIN GRAPH** (GRAFO DE GEMELOS).

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-move-regions/graph-preview-save.png" alt-text="Resaltado del icono Save (Guardar) en el panel de vista previa del grafo." lightbox="media/how-to-move-regions/graph-preview-save.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Ahora, Azure Digital Twins Explorer carga los modelos y el grafo (incluidos los gemelos y las relaciones) en la nueva instancia de Azure Digital Twins. Debería verá un mensaje de confirmación que indica cuántos modelos, gemelos y relaciones se han cargado.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-move-regions/import-success.png" alt-text="Cuadro de diálogo que indica que el grafo se ha importado correctamente. Indica &quot;Importación correcta. 2 modelos importados. 4 gemelos importados. 2 relaciones importadas.&quot;" lightbox="media/how-to-move-regions/import-success.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Para comprobar que todo se ha cargado correctamente, vuelva a la pestaña **TWIN GRAPH** (Grafo gemelo) y seleccione el botón **Run Query** (Ejecutar consulta) en el cuadro **QUERY EXPLORER** (EXPLORADOR DE CONSULTAS) para ejecutar la consulta predeterminada que muestra todos los gemelos y las relaciones en el grafo. Esta acción también actualiza la lista de modelos en el cuadro **MODELS** (MODELOS).

:::image type="content" source="media/how-to-move-regions/run-query.png" alt-text="Se resalta el botón Run Query (Ejecutar consulta) en la esquina superior derecha de la ventana." lightbox="media/how-to-move-regions/run-query.png":::

Debería ver el grafo con todos sus gemelos y relaciones en el cuadro **TWIN GRAPH** (GRAFO GEMELO). También verá los modelos enumerados en el cuadro **MODELS** (MODELOS).

:::image type="content" source="media/how-to-move-regions/post-upload.png" alt-text="Una vista de Azure Digital Twins Explorer que muestra dos modelos resaltados en el cuadro Models (Modelos) y un grafo resaltado en el cuadro Twin Graph (Grafo gemelo)." lightbox="media/how-to-move-regions/post-upload.png":::

Estas vistas confirman que los modelos, los gemelos y el grafo se han vuelto a cargar en la nueva instancia de la región de destino.

#### <a name="re-create-endpoints-and-routes"></a>Volver a crear puntos de conexión y rutas

Si tiene puntos de conexión o rutas en la instancia original, deberá volver a crearlos en la nueva instancia. Si no tiene ningún punto de conexión o ruta en la instancia original o no quiere moverlos a la nueva instancia, puede ir directamente a la [siguiente sección](#relink-connected-resources).

De lo contrario, siga los pasos que se describen en [Procedimiento: administración de puntos de conexión y rutas](how-to-manage-routes-portal.md) mediante la nueva instancia. Tenga en cuenta los siguientes puntos:

* *No* necesita volver a crear el recurso Event Grid, Event Hubs o Service Bus que usa para el punto de conexión. Para más información, consulte la sección sobre requisitos previos en las instrucciones del punto de conexión. Solo tiene que volver a crear el punto de conexión en la instancia de Azure Digital Twins.
* Los nombres tanto de punto de conexión como de ruta se pueden volver a utilizar, ya que tienen sus ámbitos son diferentes.
* No olvide agregar los filtros necesarios a las rutas que cree.

#### <a name="relink-connected-resources"></a>Volver a vincular los recursos conectados

Si tiene otras aplicaciones o recursos de Azure que están conectados a la instancia original de Azure Digital Twins, deberá editar la conexión para que se comuniquen con la nueva instancia. Estos recursos pueden incluir tanto otros servicios de Azure como aplicaciones personales o empresariales que haya configurado para trabajar con Azure Digital Twins.

Si no tiene ningún otro recurso conectado a la instancia original o no desea migrarlos a la nueva instancia, puede ir directamente a la [siguiente sección](#verify).

De lo contrario, tenga en cuenta los recursos conectados en su escenario. No es necesario eliminar y volver a crear los recursos conectados. En su lugar, solo tiene que editar los puntos en los que se conectan a una instancia de Azure Digital Twins a través de su nombre de host. Luego, actualice estos puntos para usar el nombre de host de la nueva instancia, en lugar del original.

Los recursos exactos que necesite editar dependen del escenario, pero estos son algunos puntos de integración comunes:

* Azure Functions. Si tiene una función de Azure cuyo código incluya el nombre de host de la instancia original, debe actualizar este valor al nombre de host de la nueva instancia y volver a publicar la función.
* Event Grid, Event Hubs o Service Bus.
* Logic Apps.
* Time Series Insights.
* Azure Maps.
* IoT Hub Device Provisioning Service.
* Aplicaciones personales o empresariales fuera de Azure, como la aplicación cliente creada en el [Tutorial: Programación de una aplicación cliente](tutorial-code.md), que se conecta a la instancia y llama a las API de Azure Digital Twins.
* *No* es preciso volver a crear los registros de aplicaciones de Azure AD. Si usa un [registro de aplicaciones](how-to-create-app-registration.md) para conectarse a las API de Azure Digital Twins, puede volver a usar el mismo registro de aplicaciones con la nueva instancia.

Una vez que finalice este paso, la nueva instancia de la región de destino debe ser una copia de la instancia original.

## <a name="verify"></a>Comprobar

Para comprobar que la nueva instancia se ha configurado correctamente, use las siguientes herramientas:

* [Azure Portal](https://portal.azure.com). El portal es adecuado para comprobar que la nueva instancia existe y se encuentra en la región de destino correcta. También lo es para comprobar los puntos de conexión y las rutas y conexiones con otros servicios de Azure.
* [Comandos de la CLI de Azure Digital Twins](how-to-use-cli.md). Estos comandos son adecuados para comprobar que la nueva instancia existe y se encuentra en la región de destino correcta. También se pueden usar para comprobar los datos de la instancia.
* [Azure Digital Twins Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/). Azure Digital Twins Explorer es adecuado para comprobar los datos de la instancia, como modelos, gemelos y grafos.
* [API y SDK de Azure Digital Twins](how-to-use-apis-sdks.md). Estos recursos son adecuados para comprobar los datos de la instancia, como modelos, gemelos y grafos. También lo son para comprobar los puntos de conexión y las rutas.

También puede intentar ejecutar cualquier aplicación personalizada o flujo de un extremo a otro que haya ejecutado con la instancia original, con el fin de ayudarle a comprobar que funcionan correctamente con la nueva instancia.

## <a name="clean-up-source-resources"></a>Limpieza de los recursos de origen

Ahora que la nueva instancia está configurada en la región de destino con una copia de los datos y las conexiones de la instancia original, puede eliminar la instancia original.

Puede usar [Azure Portal](https://portal.azure.com), la [CLI de Azure](how-to-use-cli.md) o las [API del plano de control](how-to-use-apis-sdks.md#overview-control-plane-apis).

Para eliminar la instancia mediante Azure Portal, [abra el portal](https://portal.azure.com) en una ventana del explorador y vaya a la instancia original de Azure Digital Twins; para ello, busque el nombre en la barra de búsqueda del portal.

Seleccione el botón **Eliminar** y siga las indicaciones para finalizar la eliminación.

:::image type="content" source="media/how-to-move-regions/delete-instance.png" alt-text="Vista de los detalles de la instancia de Azure Digital Twins en Azure Portal, en la pestaña Información general. El botón Delete (Eliminar) está resaltado.":::