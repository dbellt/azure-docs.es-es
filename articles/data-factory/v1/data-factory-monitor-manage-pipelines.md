---
title: Supervisión y administración de canalizaciones mediante Azure Portal y PowerShell
description: Obtenga información sobre el uso de Azure Portal y Azure PowerShell para supervisar y administrar las factorías de datos y las canalizaciones de Azure que haya creado.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 04/30/2018
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2d2f29553d831d80923f569211bc15855b96f591
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110665936"
---
# <a name="monitor-and-manage-azure-data-factory-pipelines-by-using-the-azure-portal-and-powershell"></a>Supervisión y administración de canalizaciones de Azure Data Factory mediante Azure Portal y PowerShell
> [!div class="op_single_selector"]
> * [Uso de Azure Portal/Azure PowerShell](data-factory-monitor-manage-pipelines.md)
> * [Uso de la aplicación de supervisión y administración](data-factory-monitor-manage-app.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el artículo sobre la [supervisión y administración de las canalizaciones de Data Factory](../monitor-visually.md).

En este artículo se describe cómo supervisar, administrar y depurar las canalizaciones mediante Azure Portal y PowerShell.

> [!IMPORTANT]
> La aplicación de supervisión y administración proporciona una mejor compatibilidad con la supervisión y la administración de las canalizaciones de datos y la solución de problemas. Para más información sobre el uso de la aplicación, consulte [Supervisión y administración de canalizaciones de Azure Data Factory mediante la aplicación de supervisión y administración](data-factory-monitor-manage-app.md). 

> [!IMPORTANT]
> La versión 1 de Azure Data Factory emplea ahora la nueva [infraestructura de alerta de Azure Monitor](../../azure-monitor/alerts/alerts-metric.md). La antigua infraestructura de alerta está en desuso. Como resultado, las alertas existentes configuradas para la las factorías de datos de la versión 1 ya no funcionan. Las alertas existentes para las factorías de datos v1 no se migran automáticamente. Deberá volver a crear estas alertas en la nueva infraestructura de alerta. Inicie sesión en Azure Portal y seleccione **Monitor** para crear nuevas alertas sobre métricas (por ejemplo, las ejecuciones erróneas o correctas) para sus factorías de datos de la versión 1.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="understand-pipelines-and-activity-states"></a>Descripción de las canalizaciones y los estados de actividad
Con Azure Portal, puede:

* Ver una factoría de datos como un diagrama.
* Ver las actividades en una canalización.
* Ver conjuntos de datos de entrada y salida.

En esta sección se describen también las transiciones de sectores de un conjunto de datos de un estado a otro.   

### <a name="navigate-to-your-data-factory"></a>Navegación hasta la factoría de datos
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Haga clic en **Factorías de datos** en el menú de la izquierda. Si no ve está opción, haga clic en **Más servicios >** y luego en **Factorías de datos**, en la categoría **INTELIGENCIA Y ANÁLISIS**.

   ![Examinar todo -> Factorías de datos](./media/data-factory-monitor-manage-pipelines/browseall-data-factories.png)
3. En la hoja **Factorías de datos**, seleccione la factoría de datos que le interese.

    ![Selección de la factoría de datos](./media/data-factory-monitor-manage-pipelines/select-data-factory.png)

   Verá la página principal de la factoría de datos.

   ![Hoja Factoría de datos](./media/data-factory-monitor-manage-pipelines/data-factory-blade.png)

#### <a name="diagram-view-of-your-data-factory"></a>Vista de diagrama de la factoría de datos
La vista **Diagrama** de una factoría de datos ofrece un panel único para supervisar y administrar la factoría de datos y sus recursos. Haga clic en **Diagrama** en la página principal de la factoría de datos para ver la vista de **diagrama**.

![Vista Diagrama](./media/data-factory-monitor-manage-pipelines/diagram-view.png)

Puede acercar, alejar, hacer zoom para ajustar, hacer zoom al 100 %, bloquear el diseño del diagrama y colocar automáticamente canalizaciones y conjuntos de datos. También puede ver la información de linaje de datos (es decir, se muestran los elementos ascendentes y descendentes de los elementos seleccionados).

### <a name="activities-inside-a-pipeline"></a>Actividades en una canalización
1. Haga clic con el botón derecho en la canalización y luego en **Abrir canalización** para ver todas las actividades de la canalización junto con los conjuntos de datos de entrada y salida para las actividades. Esta característica resulta útil cuando la canalización incluye más de una actividad y se quiere entender el linaje operativo de una sola canalización.

    ![Menú Abrir canalización](./media/data-factory-monitor-manage-pipelines/open-pipeline-menu.png)     
2. En el ejemplo siguiente, verá una actividad de copia en la canalización con una entrada y una salida. 

    ![Actividades en una canalización](./media/data-factory-monitor-manage-pipelines/activities-inside-pipeline.png)
3. Puede navegar de nuevo a la página principal haciendo clic en el vínculo **Data Factory** situado en la ruta de navegación en la esquina superior izquierda.

    ![Navegación hacia atrás a la factoría de datos](./media/data-factory-monitor-manage-pipelines/navigate-back-to-data-factory.png)

### <a name="view-the-state-of-each-activity-inside-a-pipeline"></a>Visualización del estado de cada actividad dentro de una canalización
Puede ver el estado actual de una actividad viendo el estado de cualquiera de los conjuntos de datos que genera la actividad.

Al hacer doble clic en **OutputBlobTable** en la vista **Diagrama**, puede observar todos los segmentos generados por distintas ejecuciones de actividades dentro de una canalización. Puede ver que la actividad de copia se ejecutó correctamente durante las últimas ocho horas y generó los segmentos en el estado **Listo**.  

![Estado de la canalización](./media/data-factory-monitor-manage-pipelines/state-of-pipeline.png)

Los segmentos de conjunto de datos en una factoría de datos pueden tener uno de los siguientes estados:

<table>
<tr>
    <th align="left">State</th><th align="left">Subestado</th><th align="left">Descripción</th>
</tr>
<tr>
    <td rowspan="8">En espera</td><td>ScheduleTime</td><td>No ha llegado la hora para que se ejecute el segmento.</td>
</tr>
<tr>
<td>DatasetDependencies</td><td>Las dependencias de nivel superior no están listas.</td>
</tr>
<tr>
<td>ComputeResources</td><td>No están disponibles los recursos de proceso.</td>
</tr>
<tr>
<td>ConcurrencyLimit</td> <td>Todas las instancias de actividad están ocupadas con la ejecución de otros sectores.</td>
</tr>
<tr>
<td>ActivityResume</td><td>La actividad está en pausa y no puede ejecutar los segmentos hasta que se reanude.</td>
</tr>
<tr>
<td>Volver a intentar</td><td>Se está volviendo a intentar la ejecución de la actividad.</td>
</tr>
<tr>
<td>Validación</td><td>Aún no ha iniciado la validación.</td>
</tr>
<tr>
<td>ValidationRetry</td><td>La validación está esperando un reintento.</td>
</tr>
<tr>
<tr>
<td rowspan="2">InProgress</td><td>Validating</td><td>Validación en curso.</td>
</tr>
<td>-</td>
<td>El segmento se está procesando.</td>
</tr>
<tr>
<td rowspan="4">Con error</td><td>TimedOut</td><td>La ejecución de la actividad tardó más tiempo del permitido por la actividad.</td>
</tr>
<tr>
<td>Canceled</td><td>El segmento se ha cancelado por una acción del usuario.</td>
</tr>
<tr>
<td>Validación</td><td>Error de validación.</td>
</tr>
<tr>
<td>-</td><td>No se pudo puede generar o validar el segmento.</td>
</tr>
<td>Ready</td><td>-</td><td>El segmento está listo para su uso.</td>
</tr>
<tr>
<td>Omitido</td><td>None</td><td>El segmento se está procesando.</td>
</tr>
<tr>
<td>None</td><td>-</td><td>Un segmento existía con un estado distinto, pero se ha restablecido.</td>
</tr>
</table>



Puede ver los detalles sobre un segmento haciendo clic en la hoja **Segmentos actualizados recientemente**.

![Detalles de segmento](./media/data-factory-monitor-manage-pipelines/slice-details.png)

Si el segmento se ejecutó varias veces, aparecen varias filas en la lista **Ejecuciones de actividad** . Para ver detalles sobre una ejecución de actividad, haga clic en la entrada de la ejecución en la lista **Ejecuciones de actividades** . La lista muestra todos los archivos de registro junto con un mensaje de error, si hubiera alguno. Esta característica resulta muy útil para ver y depurar registros sin tener que salir de la factoría de datos.

![Detalles de ejecución de actividad](./media/data-factory-monitor-manage-pipelines/activity-run-details.png)

Si el segmento no está en el estado **Listo**, puede ver los segmentos ascendentes que no están en estado Listo y bloquean la ejecución del segmento actual en la lista **Segmentos ascendentes que no están listos**. Esta característica resulta muy útil cuando el segmento tiene el estado **En espera** y se quiere saber cuáles son las dependencias ascendentes por las que el segmento está esperando.

![Segmentos ascendentes que no están listos](./media/data-factory-monitor-manage-pipelines/upstream-slices-not-ready.png)

### <a name="dataset-state-diagram"></a>Diagrama de estado del conjunto de datos
Cuando se implementa una factoría de datos y las canalizaciones tienen un período activo válido, los segmentos del conjunto de datos pasan de un estado a otro. Actualmente, el estado del segmento se ajusta al siguiente diagrama de estado:

![Diagrama de estado](./media/data-factory-monitor-manage-pipelines/state-diagram.png)

El flujo de transición de estado del conjunto de datos de la factoría de datos es el siguiente: En espera -> En curso /En curso (Validando) -> Listo/Error.

El segmento se inicia con un estado **En espera**, mientras se espera a que se cumplan las condiciones previas que deben cumplirse antes de su ejecución. Luego, la actividad comienza a ejecutarse y el segmento pasa al estado **En curso**. La ejecución de esta actividad se completará correctamente o dará error. El segmento se marca como **Listo** o **Error** según el resultado de la ejecución.

El usuario puede restablecer el segmento para que vuelva del estado **Listo** o **Error** al estado **En espera**. El usuario también puede marcar el estado del segmento como **Omitir**, lo que impide que la actividad se ejecute, y no se procesa el segmento.

## <a name="pause-and-resume-pipelines"></a>Pausa y reanudación de canalizaciones
Puede administrar las canalizaciones usando Azure PowerShell. Por ejemplo, puede pausar y reanudar canalizaciones ejecutando cmdlets de Azure PowerShell. 

> [!NOTE] 
> La vista de diagrama no admite las funciones de pausa y reanudación de las canalizaciones. Si quiere usar una interfaz de usuario, utilice la aplicación de supervisión y administración. Para más información sobre el uso de la aplicación, consulte el artículo [Supervisión y administración de canalizaciones de Azure Data Factory mediante la aplicación de supervisión y administración](data-factory-monitor-manage-app.md). 

Puede pausar o suspender las canalizaciones mediante el cmdlet **Suspend-AzDataFactoryPipeline** de PowerShell. Este cmdlet es útil cuando no desea ejecutar canalizaciones hasta que se solucione un problema. 

```powershell
Suspend-AzDataFactoryPipeline [-ResourceGroupName] <String> [-DataFactoryName] <String> [-Name] <String>
```
Por ejemplo:

```powershell
Suspend-AzDataFactoryPipeline -ResourceGroupName ADF -DataFactoryName productrecgamalbox1dev -Name PartitionProductsUsagePipeline
```

Tras solucionar el problema con la canalización, se puede reanudar la canalización suspendida mediante el siguiente comando de PowerShell:

```powershell
Resume-AzDataFactoryPipeline [-ResourceGroupName] <String> [-DataFactoryName] <String> [-Name] <String>
```
Por ejemplo:

```powershell
Resume-AzDataFactoryPipeline -ResourceGroupName ADF -DataFactoryName productrecgamalbox1dev -Name PartitionProductsUsagePipeline
```

## <a name="debug-pipelines"></a>Depuración de canalizaciones
Azure Data Factory ofrece amplias funcionalidades a través de Azure Portal y Azure PowerShell para depurar y solucionar problemas de las canalizaciones.

> [!NOTE] 
> Es mucho más fácil solucionar errores mediante la aplicación de supervisión y administración. Para más información sobre el uso de la aplicación, consulte el artículo [Supervisión y administración de canalizaciones de Azure Data Factory mediante la aplicación de supervisión y administración](data-factory-monitor-manage-app.md). 

### <a name="find-errors-in-a-pipeline"></a>Búsqueda de errores en una canalización
Si falla la ejecución de actividad en una canalización, el conjunto de datos generado por la canalización tiene un estado de error debido al fallo. Puede depurar y solucionar los errores en Azure Data Factory con los métodos siguientes.

#### <a name="use-the-azure-portal-to-debug-an-error"></a>Uso de Azure Portal para depurar un error
1. En la hoja **Tabla**, haga clic en el segmento problemático cuyo **Estado** sea **Error**.

   ![Hoja Tabla con segmentos con problemas](./media/data-factory-monitor-manage-pipelines/table-blade-with-error.png)
2. En la hoja **Segmento de datos**, haga clic en la ejecución de actividad que falló.

   ![Segmento de datos con un error](./media/data-factory-monitor-manage-pipelines/dataslice-with-error.png)
3. En la hoja **Detalles de la ejecución de actividad**, puede descargar los archivos asociados al procesamiento de HDInsight. Haga clic en **Descargar** correspondiente a Status/stderr para descargar el archivo de registro de errores que contiene detalles sobre el error.

   ![Hoja Detalles de ejecución de actividad con errores](./media/data-factory-monitor-manage-pipelines/activity-run-details-with-error.png)     

#### <a name="use-powershell-to-debug-an-error"></a>Uso de PowerShell para depurar un error
1. Inicie **PowerShell**.
2. Ejecute el comando **Get-AzDataFactorySlice** para ver los segmentos y sus estados. Debería ver un segmento con el estado: **Error**.        

    ```powershell   
    Get-AzDataFactorySlice [-ResourceGroupName] <String> [-DataFactoryName] <String> [-DatasetName] <String> [-StartDateTime] <DateTime> [[-EndDateTime] <DateTime> ] [-Profile <AzureProfile> ] [ <CommonParameters>]
    ```   
   Por ejemplo:

    ```powershell   
    Get-AzDataFactorySlice -ResourceGroupName ADF -DataFactoryName LogProcessingFactory -DatasetName EnrichedGameEventsTable -StartDateTime 2014-05-04 20:00:00
    ```

   Reemplace **StartDateTime** por la hora de inicio de la canalización. 
3. Ahora, ejecute el cmdlet **Get-AzDataFactoryRun** para obtener detalles sobre la ejecución de actividad para el segmento.

    ```powershell   
    Get-AzDataFactoryRun [-ResourceGroupName] <String> [-DataFactoryName] <String> [-DatasetName] <String> [-StartDateTime]
    <DateTime> [-Profile <AzureProfile> ] [ <CommonParameters>]
    ```

    Por ejemplo:

    ```powershell   
    Get-AzDataFactoryRun -ResourceGroupName ADF -DataFactoryName LogProcessingFactory -DatasetName EnrichedGameEventsTable -StartDateTime "5/5/2014 12:00:00 AM"
    ```

    El valor de StartDateTime es la hora de inicio del segmento con error o con problemas que anotó en el paso anterior. La fecha y hora debe ir encerrada entre comillas dobles.
4. Debería ver una salida con detalles sobre el error similar a la siguiente:

    ```   
    Id                      : 841b77c9-d56c-48d1-99a3-8c16c3e77d39
    ResourceGroupName       : ADF
    DataFactoryName         : LogProcessingFactory3
    DatasetName               : EnrichedGameEventsTable
    ProcessingStartTime     : 10/10/2014 3:04:52 AM
    ProcessingEndTime       : 10/10/2014 3:06:49 AM
    PercentComplete         : 0
    DataSliceStart          : 5/5/2014 12:00:00 AM
    DataSliceEnd            : 5/6/2014 12:00:00 AM
    Status                  : FailedExecution
    Timestamp               : 10/10/2014 3:04:52 AM
    RetryAttempt            : 0
    Properties              : {}
    ErrorMessage            : Pig script failed with exit code '5'. See wasb://        adfjobs@spestore.blob.core.windows.net/PigQuery
                                    Jobs/841b77c9-d56c-48d1-99a3-
                8c16c3e77d39/10_10_2014_03_04_53_277/Status/stderr' for
                more details.
    ActivityName            : PigEnrichLogs
    PipelineName            : EnrichGameLogsPipeline
    Type                    :
    ```
5. Puede ejecutar el cmdlet **Save-AzDataFactoryLog** con el valor de identificador que ve en la salida y descargar los archivos de registro mediante la opción **-DownloadLogsoption** del cmdlet.

    ```powershell
    Save-AzDataFactoryLog -ResourceGroupName "ADF" -DataFactoryName "LogProcessingFactory" -Id "841b77c9-d56c-48d1-99a3-8c16c3e77d39" -DownloadLogs -Output "C:\Test"
    ```

## <a name="rerun-failures-in-a-pipeline"></a>Repetición de la ejecución de errores en una canalización

> [!IMPORTANT]
> Es más fácil solucionar los errores y volver a ejecutar los sectores erróneos con la aplicación de supervisión y administración. Para más información sobre el uso de la aplicación, consulte [Supervisión y administración de canalizaciones de Azure Data Factory mediante la aplicación de supervisión y administración](data-factory-monitor-manage-app.md). 

### <a name="use-the-azure-portal"></a>Uso de Azure Portal
Tras solucionar los problemas y depurar los errores de una canalización, puede volver a ejecutar los elementos con fallos; para ello, vaya al segmento de error y haga clic en el botón **Ejecutar** de la barra de comandos.

![Repetición de ejecución de un segmento con errores](./media/data-factory-monitor-manage-pipelines/rerun-slice.png)

En caso de que el segmento no se valide debido a un error de directiva (por ejemplo que los datos no estén disponibles), puede corregir el error y volver a validarlo haciendo clic en el botón **Validar** de la barra de comandos.

![Corrección de errores y validación](./media/data-factory-monitor-manage-pipelines/fix-error-and-validate.png)

### <a name="use-azure-powershell"></a>Uso de Azure PowerShell
Puede volver a ejecutar errores mediante el cmdlet **Set-AzDataFactorySliceStatus**. Vea el tema [Set-AzDataFactorySliceStatus](/powershell/module/az.datafactory/set-azdatafactoryslicestatus) para obtener información sobre la sintaxis y otros detalles del cmdlet.

**Ejemplo**:

En el siguiente ejemplo, el estado de todos los segmentos de la tabla "DAWikiAggregatedData" se establece en "En espera" en la factoría de datos de Azure "WikiADF".

El valor "UpdateType" se establece en "UpstreamInPipeline", lo que significa que los estados de cada segmento de la tabla y todas las tablas dependientes (ascendentes) se establecen en "En espera". Otro valor posible para este parámetro es "Individual".

```powershell
Set-AzDataFactorySliceStatus -ResourceGroupName ADF -DataFactoryName WikiADF -DatasetName DAWikiAggregatedData -Status Waiting -UpdateType UpstreamInPipeline -StartDateTime 2014-05-21T16:00:00 -EndDateTime 2014-05-21T20:00:00
```
## <a name="create-alerts-in-the-azure-portal"></a>Creación de alertas en Azure Portal

1.  Inicie sesión en Azure Portal y seleccione **Monitor -> Alertas** para abrir la página de alertas.

    ![Abra la página Alertas.](media/data-factory-monitor-manage-pipelines/v1alerts-image1.png)

2.  Haga clic en **+Nueva regla de alertas** para crear una nueva alerta.

    ![Creación de una nueva alerta](media/data-factory-monitor-manage-pipelines/v1alerts-image2.png)

3.  Defina **Alert condition** (Condición de la alerta). (Asegúrese de seleccionar **Factorías de datos** en el campo **Filtrar por tipo de recurso**). También puede especificar valores para **Dimensiones**.

    ![Definir la condición de la alerta: Seleccione el destino](media/data-factory-monitor-manage-pipelines/v1alerts-image3.png)

    ![Definir la condición de la alerta: Agregue criterios de alerta.](media/data-factory-monitor-manage-pipelines/v1alerts-image4.png)

    ![Definir la condición de la alerta: Agregue la lógica de alerta](media/data-factory-monitor-manage-pipelines/v1alerts-image5.png)

4.  Defina los **Detalles de alertas**.

    ![Definir los detalles de la alerta](media/data-factory-monitor-manage-pipelines/v1alerts-image6.png)

5.  Defina el **Grupo de acciones**.

    ![Definir el grupo de acciones: Crear un nuevo grupo de acciones](media/data-factory-monitor-manage-pipelines/v1alerts-image7.png)

    ![Definir el grupo de acciones: Establecer las propiedades](media/data-factory-monitor-manage-pipelines/v1alerts-image8.png)

    ![Definir el grupo de acciones: Nuevo grupo de acciones creado](media/data-factory-monitor-manage-pipelines/v1alerts-image9.png)

## <a name="move-a-data-factory-to-a-different-resource-group-or-subscription"></a>Desplazamiento de una factoría de datos a una suscripción o un grupo de recursos diferente
Puede mover una factoría de datos a un grupo de recursos o una suscripción diferentes con el botón **Mover** de la barra de comandos que aparece en la página principal de su factoría de datos.

![Mover factoría de datos](./media/data-factory-monitor-manage-pipelines/MoveDataFactory.png)

Junto con la factoría de datos, también puede mover todos los recursos relacionados (como las alertas asociadas a la factoría de datos).

![Cuadro de diálogo Mover recursos](./media/data-factory-monitor-manage-pipelines/MoveResources.png)
