---
title: Migración de datos de Log Analytics para Azure HDInsight
description: Obtenga información sobre los cambios en la integración de Azure Monitor y los procedimientos recomendados para usar las nuevas tablas.
ms.service: hdinsight
ms.topic: how-to
ms.author: ali
author: AliciaLiMicrosoft
ms.date: 04/19/2021
ms.openlocfilehash: 5e312941e02c5605132971a55041a0b4f7d6dbc9
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108139100"
---
# <a name="log-analytics-migration-guide-for-azure-hdinsight-clusters"></a>Guía de migración de Log Analytics para clústeres de Azure HDInsight

 Azure HDInsight es un servicio de clústeres administrados, listo para las empresas. Este servicio ejecuta marcos de análisis de código abierto, como Apache Spark, Hadoop, HBase y Kafka en Azure. Azure HDInsight se ha integrado con otros servicios de Azure para permitir a los clientes administrar mejor sus aplicaciones de análisis de macrodatos.

Log Analytics ofrece una herramienta en Azure Portal para editar y ejecutar consultas de registros. Las consultas proceden de los datos recopilados por los registros de Azure Monitor y analizan sus resultados de manera interactiva. Los clientes pueden usar consultas de Log Analytics para recuperar registros que coincidan con criterios específicos. También pueden usar consultas para identificar tendencias, analizar patrones y proporcionar conclusiones sobre sus datos.

Azure HDInsight habilitó la integración en Log Analytics en 2017. Los clientes de HDInsight adoptaron rápidamente esta característica para supervisar sus clústeres de HDInsight y consultar los registros de los clústeres. Aunque la adopción de esta característica ha aumentado, los clientes han enviado comentarios sobre la integración:

- Los clientes no pueden decidir qué registros almacenar, y almacenar todos los registros puede resultar costoso.
- Los registros de esquemas de HDInsight actuales no cumplen convenciones de nomenclatura coherentes, y algunas tablas son repetitivas.
- Los clientes quieren un panel ya integrado para supervisar fácilmente el KPI de sus clústeres de HDInsight.
- Los clientes tienen que saltar a Log Analytics para ejecutar consultas sencillas.

## <a name="solution-overview"></a>Información general de la solución

Teniendo en cuenta los comentarios de los clientes, el equipo de Azure HDInsight invirtió en la integración en Azure Monitor. Esta integración permite:

- Un nuevo conjunto de tablas en el área de trabajo de Log Analytics de los clientes. Las nuevas tablas se entregan a través de una nueva canalización de Log Analytics.
- Mayor confiabilidad.
- Entrega de registros más rápida.
- Agrupación de tablas basadas en recursos y consultas predeterminadas.

## <a name="benefits-of-the-new-azure-monitor-integration"></a>Ventajas de la nueva integración en Azure Monitor

En este documento se describen los cambios en la integración de Azure Monitor y se proporcionan procedimientos recomendados para usar las nuevas tablas.

**Esquemas rediseñados**: el formato de los esquemas para la nueva integración de Azure Monitor está mejor organizado y es fácil de entender. Hay dos tercios menos esquemas, lo que elimina la mayor cantidad posible de ambigüedad en los esquemas heredados.

**Registro selectivo (próximo al lanzamiento)** : hay registros y métricas disponibles a través de Log Analytics. Para ayudarle a ahorrar en los costos de supervisión, publicaremos una nueva característica de registro selectivo. Use esta característica para activar y desactivar diferentes registros y orígenes de métricas. Con esta característica, solo tendrá que pagar por lo que use.

**Integración del portal de clústeres de registros**: el panel **Registros** es nuevo en el portal de clústeres de HDInsight. Cualquier usuario con acceso al clúster puede ir a este panel para consultar cualquier tabla a la que el recurso de clúster envíe registros. Los usuarios ya no necesitan acceso al área de trabajo de Log Analytics para ver los registros de un recurso de clúster específico.

**Integración del portal de clústeres de Insights**: el panel **Insights** también es nuevo en el portal de clústeres de HDInsight. Después de habilitar la nueva integración de Azure Monitor, puede seleccionar el panel **Insights** y un panel integrado de métricas y registros específico del tipo del clúster se rellenará automáticamente. Estos paneles se han renovado a partir de nuestras soluciones de Azure anteriores. Brindan información detallada sobre el rendimiento y el estado del clúster.

**Información a escala**: Puede usar el nuevo libro **At-Scale Insights** en el portal de **Azure Monitor** para supervisar el estado y el rendimiento de los clústeres en distintas suscripciones.

## <a name="customer-scenarios"></a>Escenarios de cliente

En las secciones siguientes se describe cómo los clientes pueden usar la nueva integración de Azure Monitor en distintos escenarios. En la sección [Activación de una nueva integración de Azure Monitor](#activate-a-new-azure-monitor-integration) se describe cómo activar y usar la nueva integración de Azure Monitor. En la sección [Migración de Azure Monitor clásico a la nueva integración de Azure Monitor](#migrate-to-the-new-azure-monitor-integration) se incluye información adicional para los usuarios que dependen de la integración de Azure Monitor anterior.

> [!NOTE]
> Solo los clústeres creados a finales de septiembre de 2020 y después son aptos para la nueva integración de Azure Monitor.

## <a name="activate-a-new-azure-monitor-integration"></a>Activación de una nueva integración de Azure Monitor 

> [!NOTE]
> Tiene que tener un área de trabajo de Log Analytics creada en una suscripción a la que tenga acceso antes de habilitar la nueva integración. Para obtener más información sobre la creación de un área de trabajo de Log Analytics, consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](../azure-monitor/logs/quick-create-workspace.md).

Para activar la nueva integración, vaya a la página del portal del clúster y desplácese hacia abajo en el menú de la izquierda hasta llegar a la sección **Supervisión**. En la sección **Supervisión,** seleccione **Monitor integration** (Integración de Monitor). A continuación, seleccione **Habilitar** y podrá elegir el área de trabajo de Log Analytics a la que quiere que se envíen los registros. Seleccione **Guardar** una vez que haya elegido el área de trabajo. 

### <a name="access-the-new-tables"></a>Acceso a las nuevas tablas

Hay dos maneras de acceder a las nuevas tablas. 

### <a name="approach-1"></a>Enfoque 1:
La primera manera de acceder a las nuevas tablas es a través del área de trabajo de Log Analytics. 

1. Vaya al área de trabajo de Log Analytics que seleccionó al habilitar la integración. 
2. Desplácese hacia abajo en el menú de la izquierda de la pantalla y seleccione **Registros**. Aparecerá un editor Consultas de registros con una lista de todas las tablas en el área de trabajo. 
3. Si las tablas se agrupan por **Solución**, las nuevas tablas de HDI se encuentran en la sección **Administración de registros**. 
4. Si agrupa las tablas por **Tipo de recurso**, las tablas se encuentran en la sección **Clústeres de HDInsight**, como se muestra en la imagen siguiente. 

> [!NOTE]
> En este proceso se describe cómo se accedía a los registros en la integración anterior. Esto requiere que el usuario tenga acceso al área de trabajo.

### <a name="approach-2"></a>Enfoque 2:

La segunda manera de acceder a las nuevas tablas es a través del acceso al portal del clúster.
 
1. Vaya a la página del portal del clúster y desplácese hacia abajo en el menú de la izquierda hasta que vea la sección **Supervisión**. En esta sección, verá el panel **Registros**. 
2. Seleccione **Registros**, y aparecerá un editor Consultas de registros. El editor contiene todos los registros asociados al recurso de clúster. Envió los registros al área de trabajo de Log Analytics al habilitar la integración. Estos registros proporcionan acceso basado en recursos (RBAC). Con RBAC, los usuarios que tienen acceso al clúster, pero no al área de trabajo, pueden ver los registros asociados al clúster.

Para ver la comparación, en las capturas de pantalla siguientes se muestran la vista del área de trabajo de integración heredada y la nueva vista del área de trabajo de integración:

**Vista heredada del área de trabajo de integración**

  :::image type="content" source="./media/log-analytics-migration/legacy-integration-workspace-view.png"  lightbox="./media/log-analytics-migration/legacy-integration-workspace-view.png" alt-text="Captura de pantalla que muestra la vista heredada del área de trabajo de integración." border="false":::

**Nueva vista del área de trabajo de integración**

  :::image type="content" source="./media/log-analytics-migration/new-integration-workspace-view.png" lightbox="./media/log-analytics-migration/new-integration-workspace-view.png" alt-text="Captura de pantalla que muestra la nueva vista del área de trabajo de integración." border="false":::

### <a name="use-the-new-tables"></a>Uso de las nuevas tablas

Estas integraciones pueden ayudarle a usar las nuevas tablas:

#### <a name="default-queries-to-use-with-new-tables"></a>Consultas predeterminadas para usar con las nuevas tablas

En el editor Consultas de registros, establezca el botón de alternancia en **Consultas** encima de la lista de tablas. Asegúrese de agrupar las consultas por **Tipo de recurso** y de que no haya ningún filtro establecido para un tipo de recurso distinto de **Clústeres de HDInsight**. En la imagen siguiente se muestra el aspecto de los resultados cuando se agrupan por **Tipo de recurso** y se filtran para **Clústeres de HDInsight**. Solo tiene que seleccionar uno para que aparezca en el editor Consultas de registros. Asegúrese de leer los comentarios incluidos en las consultas, ya que algunos requieren que escriba información, como el nombre del clúster, para que la consulta se ejecute correctamente.

:::image type="content" source="./media/log-analytics-migration/default-query-results-grouped-resource-type.png" alt-text="Captura de pantalla que muestra los resultados de consulta predeterminados, agrupados por tipo de recurso." border="true":::


#### <a name="create-your-own-queries"></a>Creación de sus propias consultas

Puede escribir sus propias consultas en el editor Consultas de registros. Las consultas usadas en las tablas antiguas no serán válidas en las tablas nuevas, ya que muchas de las nuevas tablas tienen esquemas nuevos y refinados. Las consultas predeterminadas son referencias excelentes para dar forma a las consultas en las nuevas tablas.

#### <a name="insights"></a>Información detallada

En Información se encuentran paneles de visualización específicos del clúster creados mediante [Azure Workbooks](../azure-monitor/visualize/workbooks-overview.md). Estos paneles le dan gráficos y visualizaciones detallados de cómo se ejecuta el clúster. Los paneles tienen secciones para cada tipo de clúster, YARN, métricas del sistema y registros de componentes. Para acceder al panel del clúster, visite la página del clúster en el portal, desplácese hacia abajo hasta la sección **Supervisión** y seleccione el panel **Información**. El panel se carga automáticamente si ha habilitado la nueva integración. Espere unos segundos para que los gráficos se carguen a medida que consulten los registros.

:::image type="content" source="./media/log-analytics-migration/visualization-dashboard.png" lightbox="./media/log-analytics-migration/visualization-dashboard.png" alt-text="Captura de pantalla en la que se muestra el panel de visualización.":::

#### <a name="custom-azure-workbooks"></a>Libros de Azure personalizados

Puede crear sus propios libros de Azure con gráficos y visualizaciones personalizados. En la página del portal del clúster, desplácese hacia abajo hasta la sección **Supervisión** y seleccione el panel **Libros** en el menú de la izquierda. Puede empezar a usar una plantilla en blanco o usar una de las plantillas de la sección **Clústeres de HDInsight**. Hay una plantilla para cada tipo de clúster. Las plantillas son útiles si quiere guardar personalizaciones específicas que la Información de HDInsight predeterminada no proporciona. No dude en enviar solicitudes de nuevas características en Información de HDInsight si cree que carecen de algo.

#### <a name="at-scale-workbooks-for-new-azure-monitor-integrations"></a>Libros a escala para nuevas integraciones de Azure Monitor

Use nuestro nuevo libro a escala para obtener una experiencia de supervisión de varios clústeres para sus clústeres. Nuestro libro a escala muestra cuál de los clústeres tiene habilitada la canalización de supervisión. El libro también ofrece una manera sencilla de comprobar el estado de varios clústeres a la vez. Para ver este libro:

1. Vaya a la página **Azure Monitor** desde la página principal de Azure Portal.
2. Una vez que esté en la página **Azure Monitor**, seleccione **Centro de información** en la sección **Información**.
3. Seleccione **Clústeres de HDInsight** en la sección **Análisis**.

   :::image type="content" source="./media/log-analytics-migration/at-scale-workbook.png" lightbox="./media/log-analytics-migration/at-scale-workbook.png" alt-text="Captura de pantalla que muestra el libro a escala." border="false":::

#### <a name="alerts"></a>Alertas

Puede agregar alertas personalizadas a los clústeres y áreas de trabajo en el editor Consultas de registro. Para ir al editor Consultas de registros, seleccione el panel **Registros** en el portal del clúster o del área de trabajo. Ejecute una consulta y, a continuación, seleccione **Nueva regla de alertas**, como se muestra en la captura de pantalla siguiente. Para obtener más información, lea sobre la [configuración de alertas](../azure-monitor/alerts/alerts-log.md).

:::image type="content" source="./media/log-analytics-migration/new-rule-alert.png" alt-text="Captura de pantalla que muestra la nueva alerta de reglas." border="false":::

## <a name="migrate-to-the-new-azure-monitor-integration"></a>Migración a la nueva integración de Azure Monitor

Si usa la integración clásica de Azure Monitor, debe realizar algunos ajustes en los formatos de las nuevas tablas después de cambiar a la nueva integración de Azure Monitor.

Para habilitar la nueva integración de Azure Monitor, siga los pasos descritos en la sección [Activación de una nueva integración de Azure Monitor](#activate-a-new-azure-monitor-integration).

### <a name="run-queries-in-log-analytics"></a>Ejecución de consultas en Log Analytics

Dado que el formato de las nuevas tablas es diferente del anterior, las consultas deben modificarse para poder usar las nuevas tablas. Una vez que haya habilitado la nueva integración de Azure Monitor, puede examinar las tablas y esquemas para identificar los campos que se usan en las consultas antiguas.

Proporcionamos una [tabla de asignación](#appendix-table-mapping) entre la tabla antigua y la nueva tabla para ayudarle a encontrar rápidamente los nuevos campos que necesita usar para migrar los paneles y las consultas.

**Consultas predeterminadas**: hemos creado consultas predeterminadas que muestran cómo usar las nuevas tablas para situaciones comunes. Las consultas predeterminadas también muestran qué información está disponible en cada tabla. Para acceder a las consultas predeterminadas, puede seguir las instrucciones de la sección [Consultas predeterminadas para usar con las nuevas tablas](#default-queries-to-use-with-new-tables) de este artículo.

### <a name="update-dashboards-for-hdinsight-clusters"></a>Actualización de paneles para clústeres de HDInsight

Si ha creado varios paneles para supervisar los clústeres de HDInsight, tiene que ajustar la consulta detrás de la tabla una vez que habilite la nueva integración de Azure Monitor. El nombre de la tabla o el nombre del campo pueden haber cambiado en la nueva integración, pero se incluye toda la información que tiene en la integración anterior.

Consulte la [tabla de asignación](#appendix-table-mapping) entre la tabla o el esquema anteriores y la tabla o el esquema nuevos para actualizar la consulta detrás de los paneles.

#### <a name="out-of-box-dashboards"></a>Paneles integrados 

También se han mejorado los paneles integrados en el nivel de clúster. Hay un botón en la parte superior derecha de cada gráfico, que le permite ver la consulta subyacente que genera la información. El gráfico es una excelente manera de familiarizarse con cómo se pueden consultar las nuevas tablas de manera eficaz. Para acceder a los paneles integrados, siga las instrucciones que encontrará en las secciones [Información detallada](#insights) y [Libros a escala para nuevas integraciones de Azure Monitor](#at-scale-workbooks-for-new-azure-monitor-integrations).

### <a name="use-an-hdinsight-at-scale-monitoring-dashboard"></a>Uso de un panel de supervisión a escala de HDInsight

Si usa el panel de supervisión integrado para clústeres de HDInsight, como Supervisión de HDInsight Spark y Supervisión de HDInsight Interactive, estamos trabajando para proporcionarle las mismas funcionalidades en el portal de Azure Monitor.

Verá que hay una opción de clústeres de HDInsight en Azure Monitor.

   :::image type="content" source="./media/log-analytics-migration/hdinsight-azure-monitor.png" lightbox="./media/log-analytics-migration/hdinsight-azure-monitor.png" alt-text="Captura de pantalla que muestra la opción HDInsight en Azure Monitor." border="false":::

El Centro de información del portal de Azure Monitor le ofrece la capacidad de supervisar varios clústeres de HDInsight en un solo lugar. Organizamos los clústeres en función del tipo de carga de trabajo, para que vea tipos como Spark, HBase y Hive. En lugar de ir a varios paneles, ahora puede supervisar todos los clústeres de HDInsight en esta vista.

> [!NOTE]
> Para obtener más información, consulte las secciones [Información detallada](#insights) y [Libros a escala para nuevas integraciones de Azure Monitor](#at-scale-workbooks-for-new-azure-monitor-integrations) de este artículo.

## <a name="enable-both-integrations-to-accelerate-the-migration"></a>Habilitación de ambas integraciones para acelerar la migración

Puede activar tanto las integraciones clásicas como nuevas de Azure Monitor al mismo tiempo en un clúster que sea apto para ambas integraciones a los efectos de migrar rápidamente a la nueva integración de Azure Monitor. La nueva integración está disponible para todos los clústeres creados después de mediados de septiembre de 2020.

De esta manera, puede realizar fácilmente una comparación en paralelo de las consultas que está usando.

### <a name="enabling-the-classic-integration"></a>Habilitación de la integración clásica

Si usa un clúster creado después de mediados de septiembre de 2020, verá la nueva experiencia del portal en el portal del clúster. Para habilitar la nueva canalización, siga los pasos descritos en la sección [Activación de una nueva integración de Azure Monitor](#activate-a-new-azure-monitor-integration). Para activar la integración clásica en este clúster, vaya a la página del portal del clúster. Seleccione el panel **Monitor integration** (Supervisión de Monitor) en la sección **Supervisión** del menú a la izquierda de la página del portal del clúster. Seleccione **Configure Azure Monitor for HDInsight clusters integration (classic)** (Configurar la integración de Azure Monitor para clústeres de HDInsight [clásico]). Aparece un menú contextual lateral con un botón de alternancia que puede usar para habilitar y deshabilitar la integración clásica de Azure Monitor. 

> [!NOTE]
> No verá ningún registro ni métrica de la integración clásica a través de la página Información y los registros del portal del clúster. Solo los nuevos registros y métricas de integraciones se presentarán en esas ubicaciones.

   :::image type="content" source="./media/log-analytics-migration/hdinsight-classic-integration.png" alt-text="Captura de pantalla que muestra el vínculo para acceder a la integración clásica." border="false":::

La creación de clústeres con la integración clásica de Azure Monitor no está disponible a partir del 1 de julio de 2021.

## <a name="release-and-support-timeline"></a>Escala de tiempo de lanzamiento y soporte técnico

- La integración clásica de Azure Monitor no estará disponible después del 15 de octubre de 2021. No se puede habilitar la integración clásica de Azure Monitor después de esa fecha.
- Las integraciones clásicas de Azure Monitor existentes seguirán funcionando. El soporte técnico será limitado para la integración clásica de Azure Monitor. 
  - Los problemas se investigarán una vez que los clientes envíen la incidencia de soporte técnico.
  - Si la solución requiere un cambio de imagen, los clientes deben pasarse a la nueva integración.
  - No se van a aplicar revisiones a los clústeres de la integración clásica de Azure Monitor, excepto en el caso de problemas de seguridad críticos.

## <a name="appendix-table-mapping"></a>Apéndice: Asignación de tablas

En los gráficos siguientes se muestran las asignaciones de tablas de la integración clásica de Azure Monitor a la nueva. En la columna **Carga de trabajo** se describe a qué carga de trabajo está asociada cada tabla. En la fila **Nueva tabla** se muestra el nombre de la nueva tabla. En la fila **Descripción** se describe el tipo de registro o métrica que estará disponible en esta tabla. En la fila **Tabla antigua** se encuentra una lista de todas las tablas de la integración clásica de Azure Monitor cuyos datos ahora estarán presentes en la tabla que aparece en la fila **Nueva tabla**.

> [!NOTE]
> Algunas tablas son nuevas y no se basan en tablas antiguas.

## <a name="general-workload-tables"></a>Tablas de cargas de trabajo generales

| Nueva tabla | Detalles |
| --- | --- |
| HDInsightAmbariSystemMetrics | <ul><li>**Descripción**: Esta tabla contiene las métricas del sistema recopiladas de Ambari. Las métricas proceden ahora de cada nodo del clúster (excepto los nodos perimetrales), en lugar de solo los dos nodos principales. Cada métrica es ahora una columna, y cada métrica se notifica una vez por registro.</li><li>**Tabla antigua**: metrics\_cpu\_nice\_cl, metrics\_cpu\_system\_cl, metrics\_cpu\_user\_cl, metrics\_memory\_cache\_CL, metrics\_memory\_swap\_CL, metrics\_memory\_total\_CLmetrics\_memory\_buffer\_CL, metrics\_load\_1min\_CL, metrics\_load\_cpu\_CL, metrics\_load\_nodes\_CL, metrics\_load\_procs\_CL, metrics\_network\_in\_CL, metrics\_network\_out\_CL</li></ul>|
| HDInsightAmbariClusterAlerts | <ul><li>**Descripción**: Esta tabla contiene alertas de clústeres de Ambari de cada nodo del clúster (excepto para los nodos perimetrales). Cada alerta es un registro en esta tabla.</li><li>**Tabla anterior**: metrics\_cluster\_alerts\_CL</li></ul>|
| HDInsightSecurityLogs | <ul><li>**Descripción**: Esta tabla contiene registros de los registros de auditoría y autenticación de Ambari.</li><li>**Tabla antigua**: log\_ambari\_audit\_CL, log\_auth\_CL</li></ul>|
| HDInsightRangerAuditLogs | <ul><li>**Descripción**: Esta tabla contiene todos los registros del registro de Ranger Audit para los clústeres de ESP.</li><li>**Tabla antigua**: ranger\_audit\_logs\_CL</li></ul>|
| HDInsightGatewayAuditLogs\_CL | <ul><li>**Descripción**: Esta tabla contiene la información de auditoría de los nodos de puerta de enlace. Tiene el mismo formato que la tabla de la columna Tablas antiguas. **Todavía se encuentra en la sección Registros personalizados.**</li><li>**Tabla antigua**: log\_gateway\_Audit\_CL</li></ul>|

## <a name="spark-workload"></a>Carga de trabajo de Spark

> [!NOTE]
> Las tablas relacionadas con aplicaciones de Spark se han reemplazado por 11 nuevas tablas de Spark (a partir de HDInsightSpark*) que ofrecerán información más detallada sobre las cargas de trabajo de Spark.


| Nueva tabla | Detalles |
| --- | --- |
| HDInsightSparkLogs | <ul><li>**Descripción**: Esta tabla contiene todos los registros relacionados con Spark y sus componentes relacionado: Livy y Jupyter.</li><li>**Tabla antigua**: log\_livy,\_CL, log\_jupyter\_CL, log\_spark\_CL, log\_sparkappsexecutors\_CL, log\_sparkappsdrivers\_CL</li></ul>|
| HDInsightSparkApplicationEvents | <ul><li>**Descripción**: Esta tabla contiene información de eventos para aplicaciones de Spark, incluidos el tiempo de envío y finalización, el identificador de la aplicación y AppName. Resulta útil para mantener un seguimiento de cuándo se iniciaron y completaron las aplicaciones. </li></ul>|
| HDInsightSparkBlockManagerEvents | <ul><li>**Descripción**: Esta tabla contiene información de eventos relacionada con el Administrador de bloques de Spark. Incluye información como el uso de memoria del ejecutor.</li></ul>|
| HDInsightSparkEnvironmentEvents | <ul><li>**Descripción**: Esta tabla contiene información de eventos relacionada con el entorno en el que se ejecuta una aplicación, incluidos el modo de implementación de Spark, el maestro y la información sobre el ejecutor.</li></ul>|
| HDInsightSparkExecutorEvents | <ul><li>**Descripción**: Esta tabla contiene información de eventos sobre el uso del ejecutor de Spark por parte de una aplicación.</li></ul>|
| HDInsightSparkExtraEvents | <ul><li>**Descripción**: Esta tabla contiene información de eventos que no cabe en ninguna otra tabla de Spark. </li></ul>|
| HDInsightSparkJobEvents | <ul><li>**Descripción**: Esta tabla contiene información sobre los trabajos de Spark, incluidas sus horas de inicio y finalización, el resultado y las fases asociadas.</li></ul>|
| HDInsightSparkSqlExecutionEvents | <ul><li>**Descripción**: Esta tabla contiene información de eventos sobre las consultas SQL de Spark, incluida la información y descripción del plan y las horas de inicio y finalización.</li></ul>|
| HDInsightSparkStageEvents | <ul><li>**Descripción**: Esta tabla contiene información de eventos para las fases de Spark, incluidas las horas de inicio y finalización, el estado de error y la información de ejecución detallada.</li></ul>|
| HDInsightSparkStageTaskAccumulables | <ul><li>**Descripción**: Esta tabla contiene métricas de rendimiento para fases y tareas.</li></ul>|
| HDInsightTaskEvents | <ul><li>**Descripción**: Esta tabla contiene información de eventos para las tareas de Spark, incluidas las horas de inicio y finalización, las fases asociadas, el estado de ejecución y el tipo de tarea.</li></ul>|
| HDInsightJupyterNotebookEvents | <ul><li>**Descripción**: Esta tabla contiene información de eventos para instancias de Jupyter Notebook.</li></ul>|

## <a name="hadoopyarn-workload"></a>Carga de trabajo de Hadoop/YARN

| Nueva tabla | Detalles |
| --- | --- |
| HDInsightHadoopAndYarnMetrics | <ul><li>**Descripción**: Esta tabla contiene métricas de JMX provenientes de los marcos de Hadoop y YARN. Contiene todas las mismas métricas de JMX que las tablas de Registros personalizados anteriores, además de otras métricas que consideramos importantes. Hemos agregado métricas de servidor de escala de tiempo, de administrador de nodos y de servidor de historial de trabajos. Contiene una métrica por registro.</li><li>**Tabla antigua**: metrics\_resourcemanager\_clustermetrics\_CL, metrics\_resourcemanager\_jvm\_CL, metrics\_resourcemanager\_queue\_root\_CL, metrics\_resourcemanager\_queue\_root\_joblauncher\_CL, metrics\_resourcemanager\_queue\_root\_default\_CL, metrics\_resourcemanager\_queue\_root\_thriftsvr\_CL</li></ul>|
| HDInsightHadoopAndYarnLogs | <ul><li>**Descripción**: Esta tabla contiene todos los registros generados a partir de los marcos de Hadoop y YARN.</li><li>**Tabla antigua**: log\_mrjobsummary\_CL, log\_resourcemanager\_CL, log\_timelineserver\_CL, log\_nodemanager\_CL</li></ul>|

 
## <a name="hivellap-workload"></a>Carga de trabajo de Hive/LLAP 

| Nueva tabla | Detalles |
| --- | --- |
| HDInsightHiveAndLLAPMetrics | <ul><li>**Descripción**: Esta tabla contiene métricas de JMX provenientes de los marcos de Hive y LLAP. Contiene todas las mismas métricas de JMX que las tablas de Registros personalizados antiguas. Contiene una métrica por registro.</li><li>**Tabla antigua**: llap\_metrics\_hiveserver2\_CL, llap\_metrics\_hs2\_metrics\_subsystemllap\_metrics\_jvm\_CL, llap\_metrics\_llap\_daemon\_info\_CL, llap\_metrics\_buddy\_allocator\_info\_CL, llap\_metrics\_deamon\_jvm\_CL, llap\_metrics\_io\_CL, llap\_metrics\_executor\_metrics\_CL, llap\_metrics\_metricssystem\_stats\_CL, llap\_metrics\_cache\_CL</li></ul>|
| HDInsightHiveAndLLAPLogs | <ul><li>**Descripción**: Esta tabla contiene registros generados a partir de Hive, LLAP y sus componentes relacionados: WebHCat y Zeppelin.</li><li>**Tabla antigua**: log\_hivemetastore\_CL log\_hiveserver2\_CL, log\_hiveserve2interactive\_CL, log\_webhcat\_CL, log\_zeppelin\_zeppelin\_CL</li></ul>|


## <a name="kafka-workload"></a>Carga de trabajo de Kafka

| Nueva tabla | Detalles |
| --- | --- |
| HDInsightKafkaMetrics | <ul><li>**Descripción**: Esta tabla contiene las métricas de JMX provenientes de Kafka. Contiene todas las mismas métricas de JMX que las tablas de Registros personalizados anteriores, además de otras métricas que consideramos importantes. Contiene una métrica por registro.</li><li>**Tabla antigua**: metrics\_kafka\_CL</li></ul>|
| HDInsightKafkaLogs | <ul><li>**Descripción**: Esta tabla contiene todos los registros generados a partir de los agentes de Kafka.</li><li>**Tabla antigua**: log\_kafkaserver\_CL, log\_kafkacontroller\_CL</li></ul>|

## <a name="hbase-workload"></a>Carga de trabajo de HBase

| Nueva tabla | Detalles |
| --- | --- |
| HDInsightHBaseMetrics | <ul><li>**Descripción**: Esta tabla contiene las métricas de JMX provenientes de HBase. Contiene todas las mismas métricas de JMX a partir de las tablas enumeradas en la columna Esquema antiguo. A diferencia de las tablas antiguas, cada fila contiene una métrica.</li><li>**Tabla antigua**: metrics\_regionserver\_CL, metrics\_regionserver\_wal\_CL, metrics\_regionserver\_ipc\_CL, metrics\_regionserver\_os\_CL, metrics\_regionserver\_replication\_CL, metrics\_restserver\_CL, metrics\_restserver\_jvm\_CL, metrics\_hmaster\_assignmentmanager\_CL, metrics\_hmaster\_ipc\_CL, metrics\_hmaster\_os\_CL, metrics\_hmaster\_balancer\_CL, metrics\_hmaster\_jvm\_CL, metrics\_hmaster\_CL,metrics\_hmaster\_fs\_CL</li></ul>|
| HDInsightHBaseLogs | <ul><li>**Descripción**: Esta tabla contiene registros de HBase y sus componentes relacionados: Phoenix y HDFS.</li><li>**Tabla antigua**: log\_regionserver\_CL, log\_restserver\_CL, log\_phoenixserver\_CL, log\_hmaster\_CL, log\_hdfsnamenode\_CL, log\_garbage\_collector\_CL</li></ul>|

## <a name="storm-workload"></a>Carga de trabajo de Storm

| Nueva tabla | Detalles |
| --- | --- |
| HDInsightStormMetrics | <ul><li>**Descripción**: Esta tabla contiene las mismas métricas de JMX que las tablas de la sección Tablas antiguas. Sus filas contienen una métrica por registro.</li><li>**Tabla antigua**: metrics\_stormnimbus\_CL, metrics\_stormsupervisor\_CL</li></ul>|
| HDInsightStormTopologyMetrics | <ul><li>**Descripción**: Esta tabla contiene métricas de nivel de topología de Storm. Tiene la misma forma que la tabla que aparece en la sección Tablas antiguas.</li><li>**Tabla antigua**: metrics\_stormrest\_CL</li></ul>|
| HDInsightStormLogs | <ul><li>**Descripción**: Esta tabla contiene todos los registros generados a partir de Storm.</li><li>**Tabla antigua**: log\_supervisor\_CL, log\_nimbus\_CL</li></ul>|

## <a name="oozie-workload"></a>Carga de trabajo de Oozie

| Nueva tabla | Detalles |
| --- | --- |
| HDInsightOozieLogs | <ul><li>**Descripción**: Esta tabla contiene todos los registros generados a partir del marco de Oozie.</li><li>**Tabla antigua**: Log\_oozie\_CL</li></ul>|

## <a name="next-steps"></a>Pasos siguientes
[Consulta a los registros de Azure Monitor para supervisar clústeres de HDInsight](hdinsight-hadoop-oms-log-analytics-use-queries.md)