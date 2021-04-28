---
title: Supervise visualmente Azure Data Factory
description: Aprenda a supervisar visualmente las factorías de datos de Azure
author: minhe-msft
ms.author: hemin
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: bcc10ccde73f5036e50d1717528933a49ccd69cd
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107904953"
---
# <a name="visually-monitor-azure-data-factory"></a>Supervise visualmente Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Una vez que haya creado y publicado una canalización en Azure Data Factory, puede asociarla a un desencadenador o iniciar manualmente una ejecución ad hoc. Puede supervisar todas las ejecuciones de la canalización de forma nativa en la experiencia de usuario de Azure Data Factory. Para abrir la experiencia de supervisión, seleccione el icono **Supervisar y administrar** de la hoja de la factoría de datos de [Azure Portal](https://portal.azure.com/). Si ya está en ADF UX, haga clic en el icono **Supervisar** de la barra lateral izquierda.

De manera predeterminada, todas las ejecuciones de la factoría de datos se muestran en la zona horaria local del explorador. Si cambia de zona horaria, todos los campos de fecha y hora se ajustan al que ha seleccionado.

## <a name="monitor-pipeline-runs"></a>La supervisión de la canalización se ejecuta

La vista de supervisión predeterminada es una lista de ejecuciones de la canalización desencadenadas en el período de tiempo seleccionado. Puede cambiar el intervalo de tiempo y filtrar por estado, nombre de canalización o anotación. Mantenga el puntero sobre la ejecución de canalización específica para ver acciones específicas de la ejecución, como volver a ejecutar y el informe de consumo.

![Vista de lista de supervisión de las ejecuciones de canalización](media/monitor-visually/pipeline-runs.png)

La cuadrícula de ejecución de la canalización contiene las columnas siguientes:

| **Nombre de la columna** | **Descripción** |
| --- | --- |
| Nombre de la canalización | Nombre de la canalización |
| Hora de inicio de la ejecución | Fecha y hora de inicio de la ejecución de la canalización (MM/DD/AAAA, HH: MM: SS a.m./p.m.) |
| Hora de finalización de la ejecución | Fecha y hora de finalización de la ejecución de la canalización (MM/DD/AAAA, HH:MM:SS a. m./p. m.) |
| Duration | Duración de ejecución (HH) |
| Desencadenado por | El nombre del desencadenador que inició la canalización |
| Status | **Error**, **Correcto**, **En curso**, **Cancelado** o **En cola** |
| Anotaciones | Etiquetas filtrables asociadas a una canalización  |
| Parámetros | Parámetros para la ejecución de la canalización (nombre/pares de valor) |
| Error | Si se ha producido un error en la canalización, el error de ejecución |
| Ejecutar identificador | Identificador de la ejecución de canalización |

Debe seleccionar manualmente el botón **Actualizar** para actualizar la lista de ejecuciones de actividad y de la canalización. La actualización automática no se admite actualmente.

![Botón Actualizar](media/monitor-visually/refresh.png)

Para ver los resultados de una ejecución de depuración, seleccione la pestaña **Depurar**.

![Seleccione el icono de vista de ejecuciones de depuraciones activas](media/iterative-development-debugging/view-debug-runs.png)

## <a name="monitor-activity-runs"></a>Supervisión de las ejecuciones de actividad

Para obtener una vista detallada de las ejecuciones de actividades individuales de una ejecución de canalización específica, haga clic en el nombre de la canalización.

![Ver ejecuciones de actividad](media/monitor-visually/view-activity-runs.png)

La vista de lista muestra ejecuciones de actividad correspondiente a cada ejecución de canalización. Mantenga el puntero sobre la ejecución de actividad específica para obtener información específica sobre la ejecución, como la entrada JSON, la salida JSON y las experiencias detalladas de supervisión específicas de la actividad.

![Hay información sobre SalesAnalyticsMLPipeline, seguida de una lista de ejecuciones de actividad.](media/monitor-visually/activity-runs.png)

| **Nombre de la columna** | **Descripción** |
| --- | --- |
| Nombre de actividad | Nombre de la actividad dentro de la canalización |
| Tipo de actividad | Tipo de la actividad, como **Copy**, **ExecuteDataFlow** o **AzureMLExecutePipeline** |
| Acciones | Iconos que le permiten ver información de entrada JSON, información de salida JSON o experiencias de supervisión específicas de la actividad detalladas | 
| Hora de inicio de la ejecución | Fecha y hora de inicio de la ejecución de actividad (MM/DD/AAAA, HH: MM: SS a.m./p.m.) |
| Duration | Duración de ejecución (HH) |
| Status | **Error**, **Correcto**, **En curso** o **Cancelado** |
| Integration Runtime | Instancia de Integration Runtime donde se ejecutó la actividad |
| Propiedades de usuario | Propiedades definidas por el usuario de la actividad |
| Error | Si se ha producido un error en la actividad, el error de ejecución |
| Ejecutar identificador | Identificador de la ejecución de la actividad |

Si se produce un error en una actividad, puede ver el mensaje de error detallado al hacer clic en el icono de la columna Error. 

![Aparece una notificación con los detalles del error, que incluye el código de error, el tipo de error y los detalles del error.](media/monitor-visually/activity-run-error.png)

### <a name="promote-user-properties-to-monitor"></a>Promocionar las propiedades de usuario para supervisar

Promueva cualquier propiedad de actividad de canalización como propiedad de usuario para que se convierta en una entidad que supervise. Por ejemplo, puede promover las propiedades **Origen** y **Destino** de la actividad de copia de la canalización como propiedades de usuario.

> [!NOTE]
> Puede promover un máximo de cinco propiedades de la actividad de canalización como propiedades del usuario.

![Creación de propiedades de usuario](media/monitor-visually/promote-user-properties.png)

Después de crear las propiedades del usuario, puede supervisarlas en las vistas de lista de supervisión.

![Agregar columnas para propiedades del usuario a la lista de ejecuciones de actividad](media/monitor-visually/choose-user-properties.png)

 Si el origen de la actividad de copia es un nombre de tabla, puede supervisar el nombre de la tabla de origen como una columna de la vista de lista en las ejecuciones de actividad.

![Lista de ejecuciones de actividad con columnas de propiedades de usuario](media/monitor-visually/view-user-properties.png)

## <a name="rerun-pipelines-and-activities"></a>Repetición de canalizaciones y actividades

Para volver a ejecutar una canalización que se ejecutó previamente desde el inicio, mantenga el puntero sobre la ejecución de canalización específica y seleccione **Volver a ejecutar**. Si selecciona varias canalizaciones, puede usar el botón **Volver a ejecutar** para ejecutarlas todas.

![Repetición de una canalización](media/monitor-visually/rerun-pipeline.png)

Si quiere volver a ejecutar a partir de un punto concreto, puede hacerlo desde la vista Ejecuciones de actividad. Seleccione la actividad desde la que quiere empezar y seleccione **Volver a ejecutar desde actividad**. 

![Repetición de una ejecución de actividad](media/monitor-visually/rerun-activity.png)

### <a name="rerun-from-failed-activity"></a>Volver a ejecutar desde la actividad con errores

Si se produce un error en una actividad, se agota el tiempo de espera o se cancela, puede seleccionar **Volver a ejecutar desde la actividad con errores** para volver a ejecutar la canalización desde esa actividad con errores.

![Volver a ejecutar la actividad con errores](media/monitor-visually/rerun-failed-activity.png)

### <a name="view-rerun-history"></a>Ver el historial de repetición de ejecuciones

Puede ver el historial de repetición de ejecuciones de todas las ejecuciones de canalización en la vista de lista.

![Visualización del historial](media/monitor-visually/rerun-history-1.png)

También puede ver el historial de repetición de ejecuciones de una ejecución de canalización determinada.

![Ver el historial de ejecuciones de una canalización](media/monitor-visually/view-rerun-history.png)

## <a name="monitor-consumption"></a>Supervisión del consumo

Para ver los recursos utilizados por una ejecución de canalización, haga clic en el icono de consumo junto a la ejecución. 

![Captura de pantalla que muestra dónde puede ver los recursos consumidos por una canalización.](media/monitor-visually/monitor-consumption-1.png)

Al hacer clic en el icono, se abre un informe de consumo de recursos usados por esa canalización. 

![Supervisión del consumo](media/monitor-visually/monitor-consumption-2.png)

Puede conectar estos valores a la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/details/data-factory/) para calcular el costo de la ejecución de la canalización. Para más información sobre los precios de Azure Data Factory, consulte [Descripción de los precios](pricing-concepts.md).

> [!NOTE]
> Estos valores devueltos por la calculadora de precios son una estimación. No refleja el importe exacto en el que se le facturará Azure Data Factory 

## <a name="gantt-views"></a>Vistas de Gantt

Un diagrama de Gantt es una vista que le permite ver el historial de ejecución en un intervalo de tiempo. Al cambiar a una vista de Gantt, notará que todas las ejecuciones de canalización agrupadas por nombre se muestran como barras en relación con cuánto tiempo tardaron en ejecutarse. También puede agruparlas por las anotaciones o etiquetas que haya creado en la canalización. La vista de Gantt también está disponible en el nivel de ejecución de actividad.

![Ejemplo de diagrama de Gantt](media/monitor-visually/select-gantt.png)

La longitud de la barra le informará de la duración de la canalización. Asimismo, puede seleccionar la barra para ver más detalles.

![Duración del diagrama de Gantt](media/monitor-visually/view-gantt-run.png)

## <a name="alerts"></a>Alertas

Puede generar alertas en función de métricas admitidas en Data Factory. Seleccione **Supervisar** > **Alertas y métricas**, en la página de supervisión de Data Factory, para empezar.

![Página de supervisión de Data Factory](media/monitor-visually/start-page.png)

Si desea una demostración y una introducción de siete minutos de esta característica, vea el siguiente vídeo:

> [!VIDEO https://channel9.msdn.com/shows/azure-friday/Monitor-your-Azure-Data-Factory-pipelines-proactively-with-alerts/player]

### <a name="create-alerts"></a>Creación de alertas

1.  Seleccione **Nueva regla de alertas** para crear una nueva alerta.

    ![Botón Nueva regla de alertas](media/monitor-visually/new-alerts.png)

1.  Especifique el nombre de la regla y seleccione la gravedad de la alerta.

    ![Cuadros del nombre de la regla y de la gravedad](media/monitor-visually/name-and-severity.png)

1.  Seleccione los criterios de la alerta.

    ![Cuadro para los criterios de destino](media/monitor-visually/add-criteria-1.png)

    ![Captura de pantalla que muestra dónde se selecciona una métrica para configurar la condición de la alerta.](media/monitor-visually/add-criteria-2.png)

    ![Lista de criterios](media/monitor-visually/add-criteria-3.png)

    Puede crear alertas de varias métricas, incluidas para el tamaño/recuento de entidades de ADF, ejecuciones de actividad/canalización/desencadenador, uso de CPU/memoria/número de nodos/cola de Integration Runtime (IR), así como para ejecuciones de paquetes de SSIS y operaciones de inicio/parada de IR de SSIS.

1.  Configure la lógica de la alerta. Puede crear una alerta de la métrica seleccionada para todas las canalizaciones y actividades correspondientes. También puede seleccionar un tipo de actividad, nombre de actividad, nombre de canalización o tipo de error determinados.

    ![Opciones para configurar la lógica de alerta](media/monitor-visually/alert-logic.png)

1.  Configure las notificaciones push, por correo electrónico, SMS o voz de la alerta. Cree un grupo de acciones para las notificaciones de alerta o elija uno existente.

    ![Opciones para configurar notificaciones](media/monitor-visually/configure-notification-1.png)

    ![Opciones para agregar una notificación](media/monitor-visually/configure-notification-2.png)

1.  Cree la regla de alertas.

    ![Opciones para crear una regla de alertas](media/monitor-visually/create-alert-rule.png)

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la supervisión y la administración de canalizaciones, consulte el artículo [Supervisión y administración de canalizaciones mediante programación](./monitor-programmatically.md).