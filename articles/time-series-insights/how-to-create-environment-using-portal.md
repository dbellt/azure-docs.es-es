---
title: 'Configuración de un entorno de Gen2 mediante Azure Portal: Azure Time Series Insights Gen2 | Microsoft Docs'
description: Aprenda a configurar un entorno en Azure Time Series Insights Gen2 con Azure Portal.
author: riserrad
ms.author: riserrad
manager: edett
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: how-to
ms.date: 03/15/2021
ms.custom: seodec18
ms.openlocfilehash: e10685b1b64e8e3cd636245625c13df12f177fbb
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107951849"
---
# <a name="create-an-azure-time-series-insights-gen2-environment-using-the-azure-portal"></a>Creación de un entorno de Azure Time Series Insights Gen2 mediante Azure Portal

En este artículo se describe cómo crear un entorno de Azure Time Series Insights Gen2 con [Azure Portal](https://portal.azure.com/).

El tutorial de aprovisionamiento de entorno le guiará por el proceso. Aprenderá a seleccionar el identificador de serie temporal correcto y verá ejemplos de dos cargas JSON.</br>

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RWzk3P]

## <a name="overview"></a>Información general

Al aprovisionar un entorno de Azure Time Series Insights Gen2, creará estos recursos de Azure:

* Un entorno de Azure Time Series Insights Gen2 que sigue el modelo de precios de pago por uso.
* Una cuenta de Azure Storage.
* Un almacenamiento intermedio opcional para consultas más rápidas e ilimitadas.

> [!TIP]
>
> * Aprenda [a planificar su entorno ](./how-to-plan-your-environment.md).
> * Obtenga información sobre cómo [agregar un origen de centro de eventos](./how-to-ingest-data-event-hub.md) o cómo [agregar un origen de IoT Hub](./how-to-ingest-data-iot-hub.md).

Aprenderá a:

1. Asociar cada entorno de Azure Time Series Insights Gen2 con un origen del evento. También proporcionará una propiedad de identificador de marca de tiempo y un grupo de consumidores único para asegurarse de que el entorno tiene acceso a los eventos correspondientes.

1. Una vez completado el aprovisionamiento, puede modificar sus directivas de acceso y otros atributos del entorno para adaptarse a sus necesidades empresariales.

   > [!NOTE]
   > El primer paso es opcional al aprovisionar un entorno. Si omite este paso, debe adjuntar un origen del evento al entorno más adelante para que los datos puedan empezar a fluir a su entorno y se pueda tener acceso a ellos a través de la consulta.

## <a name="create-the-environment"></a>Creación del entorno

Para crear un entorno de Azure Time Series Insights Gen2:

1. Cree un recurso de Azure Time Series Insights en *Internet de las cosas* en [Azure Portal](https://portal.azure.com/).

1. Seleccione **Gen2(L1)** como el **Nivel**. Proporcione un nombre de entorno y elija el grupo de suscripciones y el grupo de recursos que se va a usar. A continuación, seleccione una ubicación compatible para hospedar el entorno.

   :::image type="content" source="media/how-to-create-environment-using-portal/environment-configuration.png" alt-text="Cree una instancia de Azure Time Series Insights." lightbox="media/how-to-create-environment-using-portal/environment-configuration.png":::

1. Introduzca un id. de serie temporal.

   :::image type="content" source="media/how-to-create-environment-using-portal/environment-configuration-2.png" alt-text="Creación de una configuración del entorno de Azure Time Series Insights, continuación." lightbox="media/how-to-create-environment-using-portal/environment-configuration-2.png":::

   > [!NOTE]
   >
   > * El identificador de serie temporal es *inmutable* y *distingue mayúsculas de minúsculas*. (Una vez establecido, no se puede cambiar).
   > * Los identificadores de serie temporal pueden tener hasta *tres* claves. Considérelo como una clave principal en una base de datos, que representa de forma única cada sensor de dispositivo que enviaría datos a su entorno. Puede ser una propiedad o una combinación de hasta tres propiedades.
   > * Más información sobre [cómo elegir un identificador de serie temporal](./how-to-select-tsid.md)

1. Para crear una cuenta de Azure Storage, seleccione un nombre de cuenta de almacenamiento, un tipo de cuenta y designe una opción de [replicación](../storage/common/redundancy-migration.md?tabs=portal). Al hacerlo, se crea automáticamente una cuenta de Azure Storage. De manera predeterminada, se creará la cuenta de [uso general v2](../storage/common/storage-account-overview.md). La cuenta se crea en la misma región que el entorno de Azure Time Series Insights Gen2 que seleccionó anteriormente.
De manera alternativa, también puede traer su propio almacenamiento (BYOS) a través de una [plantilla de Azure Resource Manager](./time-series-insights-manage-resources-using-azure-resource-manager-template.md) cuando cree un nuevo entorno de Azure Time Series Gen2.

1. **(Opcional)** Habilite el almacenamiento intermedio para el entorno si quiere consultas más rápidas e ilimitadas sobre los datos más recientes del entorno. También puede crear o eliminar un almacenamiento intermedio a través de la opción **Configuración de almacenamiento** en el panel de navegación de la izquierda, después de crear un entorno de Azure Time Series Insights Gen2.

1. **(Opcional)** Ahora puede agregar un origen del evento. También puede esperar hasta que se haya aprovisionado la instancia.

   * Azure Time Series Insights admite [Azure IoT Hub](./how-to-ingest-data-iot-hub.md) y [Azure Event Hubs](./how-to-ingest-data-event-hub.md) como opciones de origen del evento. Si bien solo puede agregar un origen del evento único en el momento de crear el entorno, puede agregar otro origen del evento más adelante.

     Puede seleccionar un grupo de consumidores existente o crear uno nuevo cuando agregue el origen del evento. Asegúrese de que el origen del evento use un único grupo de consumidores para que el entorno lea los datos en él.

   * Elija cuándo empezar a recopilar datos del origen del evento. El valor predeterminado es empezar a recopilar desde el momento en que se crea el origen del evento.

     > [!TIP]
     > Si elige la opción **Todos mis datos** para recopilar datos preexistentes del origen del evento, podría experimentar una latencia inicial alta a medida que el entorno de Azure Time Series Insights Gen2 procesa todos los datos. La latencia debería mitigarse a medida que se indexan los datos.

   * Elija la propiedad Timestamp adecuada. De manera predeterminada, Azure Time Series Insights usa el tiempo de puesta en cola del mensaje para cada origen del evento.

     > [!TIP]
     > Es posible que el tiempo de puesta en cola del mensaje no sea la mejor opción de configuración para usar en escenarios de carga de datos históricos o de eventos por lotes. En tales casos, asegúrese de comprobar correctamente la decisión de usar o no usar una propiedad Timestamp.

   :::image type="content" source="media/how-to-create-environment-using-portal/configure-event-source.png" alt-text="Pestaña Configuración de origen de eventos" lightbox="media/how-to-create-environment-using-portal/configure-event-source.png":::

1. Seleccione **Revisar y crear** para confirmar que el entorno se ha aprovisionado y configurado de la forma deseada.

    :::image type="content" source="media/how-to-create-environment-using-portal/environment-confirmation.png" alt-text="Pestaña Revisar y crear" lightbox="media/how-to-create-environment-using-portal/environment-confirmation.png":::

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los entornos de Gen2 y de disponibilidad general de Azure Time Series Insights, lea [Planeamiento del entorno](./how-to-plan-your-environment.md).
* Obtenga información acerca de los [orígenes de eventos de ingesta de streaming](./concepts-streaming-ingestion-event-sources.md) en el entorno de Azure Time Series Insights Gen2.
* Obtenga más información sobre [cómo administrar el entorno](./how-to-provision-manage.md).
