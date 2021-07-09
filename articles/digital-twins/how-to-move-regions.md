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
ms.openlocfilehash: 9b4d896edea86d85b650325ac5efb7f3cf439b17
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111953943"
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

Para recopilar esta información se pueden usar [Azure Portal](https://portal.azure.com), las [API y los SDK de Azure Digital Twins](concepts-apis-sdks.md), los [ comandos de la CLI de Azure Digital Twins](/cli/azure/dt?view=azure-cli-latest&preserve-view=true) o [Azure Digital Twins Explorer](concepts-azure-digital-twins-explorer.md).

## <a name="prepare"></a>Preparación

En esta sección, se preparará para volver a crear la instancia mediante la descarga de los modelos, gemelos y grafo originales de la instancia original. En este artículo se usa [Azure Digital Twins Explorer](concepts-azure-digital-twins-explorer.md) para esta tarea.

>[!NOTE]
>Es posible que ya tenga en la instancia archivos que contengan los modelos o el grafo. Si es así, no es necesario volver a descargar, solo las partes que faltan o aquello que pueda haber cambiado desde que se cargaron originalmente estos archivos. Por ejemplo, puede que tenga gemelos que se hayan actualizado con nuevos datos.

### <a name="download-models-twins-and-graph-with-azure-digital-twins-explorer"></a>Descarga de modelos, gemelos y grafos con Azure Digital Twins Explorer

En primer lugar, abra **Azure Digital Twins Explorer** para su instancia de Azure Digital Twins en [Azure Portal](https://portal.azure.com). Para hacerlo, vaya a la instancia de Azure Digital Twins en el portal y busque su nombre en la barra de búsquedas. A continuación, seleccione el botón **Ir al Explorador (versión preliminar)** . 

:::image type="content" source="media/includes/azure-digital-twins-explorer-portal-access.png" alt-text="Captura de pantalla de Azure Portal que muestra la página Información general de una instancia de Azure Digital Twins. Se ha resaltado el botón Ir al explorador (versión preliminar)." lightbox="media/includes/azure-digital-twins-explorer-portal-access.png":::

Se abrirá una ventana de Azure Digital Twins Explorer conectada a esta instancia.

:::image type="content" source="media/quickstart-azure-digital-twins-explorer/explorer-blank.png" alt-text="Captura de pantalla de Azure Portal en un explorador de Internet. El portal muestra Azure Digital Twins Explorer sin datos." lightbox="media/quickstart-azure-digital-twins-explorer/explorer-blank.png":::

Siga las instrucciones de Azure Digital Twins Explorer que se indican en [Exportación de grafos y modelos](how-to-use-azure-digital-twins-explorer.md#export-graph-and-models). Esto descargará un archivo JSON en la máquina que contiene el código de los modelos, gemelos y relaciones (incluidos los modelos que no se usan actualmente en el grafo).

## <a name="move"></a>Mover

A continuación, completará el "movimiento" de la instancia. Para ello, creará una instancia en la región de destino. Y, después, la llenará con los datos y componentes de la instancia original.

### <a name="create-a-new-instance"></a>Creación de una instancia

En primer lugar, crear una instancia de Azure Digital Twins en la región de destino. Siga los pasos que se describen en [Procedimientos: configuración de una instancia y autenticación](how-to-set-up-instance-portal.md). Tenga en cuenta los siguientes puntos:

* Puede usar el mismo nombre para la nueva instancia *si* está en otro grupo de recursos. Si necesita usar el mismo grupo de recursos que contiene la instancia original, la nueva instancia necesitará su propio nombre distintivo.
* Cuando se le solicite una ubicación, escriba la nueva región de destino.

Una vez que haya completado este paso, necesitará el nombre de host de la nueva instancia para continuar con la configuración de los datos. Si no ha anotado el nombre de host durante la configuración, siga [estas instrucciones](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values) para obtenerlo ahora desde Azure Portal.

A continuación, configurará los datos de la nueva instancia para que sea una copia de la instancia original.

#### <a name="upload-models-twins-and-graph-with-azure-digital-twins-explorer"></a>Carga de modelos, gemelos y grafos con Azure Digital Twins Explorer

En esta sección, puede volver a cargar los modelos, los gemelos y el grafo en la nueva instancia. Si no tiene ninguno de ellos en la instancia original o no desea migrarlos a la nueva instancia, puede ir directamente a la [siguiente sección](#re-create-endpoints-and-routes).

En primer lugar, vaya a **Azure Digital Twins Explorer** para ver la nueva instancia en [Azure Portal](https://portal.azure.com). 

Importe el [archivo JSON que descargó](#download-models-twins-and-graph-with-azure-digital-twins-explorer) anteriormente en este artículo a la nueva instancia de siguiendo los pasos descritos en las instrucciones de [Importación de un archivo a Azure Digital Twins Explorer](how-to-use-azure-digital-twins-explorer.md#import-file-to-azure-digital-twins-explorer). Esto cargará todos los modelos, gemelos y relaciones de la instancia original en la nueva instancia.

Para comprobar que todo se ha cargado correctamente, vuelva a la pestaña **Twin Graph** (Grafo gemelo) y seleccione el botón **Run Query** (Ejecutar consulta) en el panel **Query Explorer** (Explorador de consultas) para ejecutar la consulta predeterminada que muestra todos los gemelos y las relaciones en el grafo. Esta acción también actualiza la lista de modelos del panel **Models** (Modelos).

:::image type="content" source="media/how-to-move-regions/run-query.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer con el botón &quot;Ejecutar consulta&quot; resaltado en la esquina superior derecha de la ventana." lightbox="media/how-to-move-regions/run-query.png":::

Debería ver el grafo con todos sus gemelos y relaciones en el cuadro **Twin Graph** (Grafo gemelo). También verá los modelos enumerados en el panel **Models** (Modelos).

:::image type="content" source="media/how-to-move-regions/post-upload.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer que muestra dos modelos resaltados en el cuadro Models (Modelos) y un grafo resaltado en el cuadro Twin Graph (Grafo gemelo)." lightbox="media/how-to-move-regions/post-upload.png":::

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
* *No* es preciso volver a crear los registros de aplicaciones de Azure AD. Si usa un [registro de aplicaciones](./how-to-create-app-registration-portal.md) para conectarse a las API de Azure Digital Twins, puede volver a usar el mismo registro de aplicaciones con la nueva instancia.

Una vez que finalice este paso, la nueva instancia de la región de destino debe ser una copia de la instancia original.

## <a name="verify"></a>Comprobar

Para comprobar que la nueva instancia se ha configurado correctamente, use las siguientes herramientas:

* [Azure Portal](https://portal.azure.com). El portal es adecuado para comprobar que la nueva instancia existe y se encuentra en la región de destino correcta. También lo es para comprobar los puntos de conexión y las rutas y conexiones con otros servicios de Azure.
* [Comandos de la CLI de Azure Digital Twins](/cli/azure/dt?view=azure-cli-latest&preserve-view=true). Estos comandos son adecuados para comprobar que la nueva instancia existe y se encuentra en la región de destino correcta. También se pueden usar para comprobar los datos de la instancia.
* [Azure Digital Twins Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/). Azure Digital Twins Explorer es adecuado para comprobar los datos de la instancia, como modelos, gemelos y grafos.
* [API y SDK de Azure Digital Twins](concepts-apis-sdks.md). Estos recursos son adecuados para comprobar los datos de la instancia, como modelos, gemelos y grafos. También lo son para comprobar los puntos de conexión y las rutas.

También puede intentar ejecutar cualquier aplicación personalizada o flujo de un extremo a otro que haya ejecutado con la instancia original, con el fin de ayudarle a comprobar que funcionan correctamente con la nueva instancia.

## <a name="clean-up-source-resources"></a>Limpieza de los recursos de origen

Ahora que la nueva instancia está configurada en la región de destino con una copia de los datos y las conexiones de la instancia original, puede eliminar la instancia original.

Puede usar [Azure Portal](https://portal.azure.com), la [CLI de Azure](/cli/azure/dt?view=azure-cli-latest&preserve-view=true) o las [API del plano de control](concepts-apis-sdks.md#overview-control-plane-apis).

Para eliminar la instancia mediante Azure Portal, [abra el portal](https://portal.azure.com) en una ventana del explorador y vaya a la instancia original de Azure Digital Twins; para ello, busque el nombre en la barra de búsqueda del portal.

Seleccione el botón **Eliminar** y siga las indicaciones para finalizar la eliminación.

:::image type="content" source="media/how-to-move-regions/delete-instance.png" alt-text="Captura de pantalla de los detalles de la instancia de Azure Digital Twins en Azure Portal, en la pestaña Información general. El botón Delete (Eliminar) está resaltado.":::