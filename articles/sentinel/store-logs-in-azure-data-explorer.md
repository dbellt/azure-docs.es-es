---
title: Integración con Azure Data Explorer para conservar registros a largo plazo | Microsoft Docs
description: Envíe registros de Azure Sentinel a Azure Data Explorer para conservarlos a largo plazo y reducir los costos de almacenamiento de datos.
services: sentinel
documentationcenter: na
author: batamig
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/25/2021
ms.author: bagol
ms.openlocfilehash: 584f108ea534b8703b941d91b7d03b6275ea6134
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958353"
---
# <a name="integrate-azure-data-explorer-for-long-term-log-retention"></a>Integración con Azure Data Explorer para conservar registros a largo plazo

De forma predeterminada, los registros ingeridos en Azure Sentinel se almacenan en Azure Monitor Log Analytics. En este artículo se explica cómo reducir los costos de retención de Azure Sentinel enviándolos a Azure Data Explorer para conservarlos a largo plazo.

Almacenar registros en Azure Data Explorer reduce los costos y, al mismo tiempo, permite seguir consultando los datos. Además, es especialmente útil a medida que los datos crecen. Por ejemplo, aunque es posible que los datos de seguridad pierdan valor con el tiempo, es posible que deba conservar los registros para cumplir con los requisitos normativos o realizar investigaciones periódicas en los datos más antiguos.

## <a name="about-azure-data-explorer"></a>Acerca de Azure Data Explorer

Azure Data Explorer es una plataforma de análisis de macrodatos que está altamente optimizada para el análisis de registros y datos. Puesto que Azure Data Explorer usa el lenguaje de consulta Kusto (KQL), es una buena alternativa para almacenar datos de Azure Sentinel. Al usar Azure Data Explorer para el almacenamiento de datos, podrá ejecutar consultas multiplataforma y visualizar datos tanto en Azure Data Explorer como en Azure Sentinel.

Para obtener más información, consulte la [documentación](/azure/data-explorer/) y el [blog](https://azure.microsoft.com/en-us/blog/tag/azure-data-explorer/) de Azure Data Explorer.

### <a name="when-to-integrate-with-azure-data-explorer"></a>Cuándo se debe integrar con Azure Data Explorer

Azure Sentinel proporciona funcionalidades completas de SIEM y SOAR, implementación y configuración rápidas, así como características de seguridad avanzadas e integradas para los equipos del centro de operaciones de seguridad (SOC). Sin embargo, el valor que se obtiene por almacenar datos de seguridad en Azure Sentinel puede disminuir después de unos meses, una vez que los usuarios del SOC no necesiten acceder a ellos con tanta frecuencia como con los datos más recientes.

Si solo necesita acceder ocasionalmente a tablas específicas, como para las investigaciones o auditorías periódicas, podría llegar a la conclusión de que conservar los datos en Azure Sentinel ya no es rentable. En este punto, se recomienda almacenar datos en Azure Data Explorer, lo que cuesta menos, pero permite explorar usando las mismas consultas KQL que se ejecutan en Azure Sentinel.

Puede acceder a los datos de Azure Data Explorer directamente desde Azure Sentinel mediante la [característica de proxy de Azure Data Explorer en Log Analytics](../azure-monitor/logs/azure-monitor-data-explorer-proxy.md). Para ello, use consultas entre clústeres en la búsqueda de registros o en los libros.

> [!IMPORTANT]
> Las funcionalidades principales de SIEM, incluidas las reglas de análisis, UEBA y el gráfico de investigación, no admiten los datos almacenados en Azure Data Explorer.
>

> [!NOTE]
> La integración con Azure Data Explorer también puede permitirle tener control y granularidad de los datos. Para obtener más información, consulte [Consideraciones de diseño](#design-considerations).
>
## <a name="send-data-directly-to-azure-sentinel-and-azure-data-explorer-in-parallel"></a>Envío de datos directamente a Azure Sentinel y Azure Data Explorer en paralelo

Es posible que desee conservar los datos *con valor de seguridad* en Azure Sentinel para usarlos en detecciones, investigaciones de incidentes, búsqueda de amenazas, UEBA, etc. Mantener estos datos en Azure Sentinel beneficia principalmente a los usuarios del SOC, donde normalmente son suficientes entre 3 y 12 meses de almacenamiento.

También puede configurar todos los datos, *independientemente de su valor de seguridad,* para que se envíen a Azure Data Explorer al mismo tiempo, donde puede almacenar los datos durante más tiempo. Aunque el envío de datos a Azure Sentinel y Azure Data Explorer al mismo produce alguna duplicación, el ahorro de costos puede ser significativo a medida que se reducen los costos de retención de Azure Sentinel.

> [!TIP]
> Esta opción también permite poner en correlación los datos distribuidos entre almacenes de datos (como enriquecer los datos de seguridad almacenados en Azure Sentinel con los datos operativos o almacenados a largo plazo almacenados en Azure Data Explorer). Para obtener más información, consulte [Consulta entre recursos en Azure Data Explorer mediante Azure Monitor](../azure-monitor/logs/azure-monitor-data-explorer-proxy.md).
>

En la imagen siguiente se muestra cómo puede conservar todos los datos en Azure Data Explorer, al tiempo que se envían solo los datos de seguridad a Azure Sentinel para su uso diario.

:::image type="content" source="media/store-logs-in-adx/store-data-in-sentinel-and-adx-in-parallel.png" alt-text="Almacenamiento de datos en Azure Data Explorer y Azure Sentinel en paralelo.":::

Para obtener más información sobre cómo implementar esta opción de arquitectura, consulte [Supervisión de Azure Data Explorer](/azure/architecture/solution-ideas/articles/monitor-azure-data-explorer).

## <a name="export-data-from-log-analytics-into-azure-data-explorer"></a>Exportación de datos de Log Analytics a Azure Data Explorer

En lugar de enviar los datos directamente a Azure Data Explorer, puede elegir exportar los datos de Log Analytics a Azure Data Explorer mediante un centro de eventos de Azure o Azure Data Factory.

### <a name="data-export-architecture"></a>Arquitectura de Exportación segura

En la imagen siguiente se muestra un flujo de ejemplo de datos exportados mediante la canalización de ingesta de Azure Monitor. Los datos se dirigen a Log Analytics de forma predeterminada, pero también puede configurarlos para que se exporten una cuenta de Azure Storage o a un centro de eventos.

:::image type="content" source="media/store-logs-in-adx/export-data-from-azure-monitor.png" alt-text="Exportación de datos desde Azure Monitor: arquitectura.":::

Al configurar las reglas de exportación de datos, seleccione los tipos de registros que desea exportar. Una vez configurados, los nuevos datos que llegan al punto de conexión de ingesta de Log Analytics y que están destinados al área de trabajo de las tablas seleccionadas se exportan a la cuenta de almacenamiento o al centro de eventos.

Al configurar los datos para la exportación, tenga en cuenta lo siguiente:

|Consideración  | Detalles |
|---------|---------|
|**Ámbito de los datos exportados**     |  Una vez configurada la exportación para una tabla específica, se exportan todos los datos enviados a esa tabla, sin excepción. No se puede exportar un subconjunto filtrado de los datos ni limitar la exportación a eventos específicos.       |
|**Requisitos de ubicación**     |   Tanto el área de trabajo de Azure Monitor y de Azure Sentinel como la ubicación de destino (una cuenta de Azure Storage o un centro de eventos) deben encontrarse en la misma región geográfica.      |
|**Tablas admitidas**     | No todas las tablas se pueden exportar, como es el caso de las tablas de registros personalizadas. <br><br>Para obtener más información, consulte [Exportación de datos del área de trabajo de Log Analytics en Azure Monitor](../azure-monitor/logs/logs-data-export.md) y la [lista de tablas admitidas](../azure-monitor/logs/logs-data-export.md#supported-tables).         |
|     |         |

### <a name="data-export-methods-and-procedures"></a>Métodos y procedimientos de exportación de datos

Use uno de los procedimientos siguientes para exportar datos de Azure Sentinel a Azure Data Explorer:

- **Un centro de eventos de Azure**. Exporte datos de Log Analytics a un centro de eventos, donde puede ingerirlos en Azure Data Explorer. Este método almacena algunos datos (los primeros X meses) en Azure Sentinel y Azure Data Explorer.

- **Azure Storage y Azure Data Factory**. Exporte los datos de Log Analytics a Azure Blob Storage; a continuación, Azure Data Factory se usa para ejecutar un trabajo de copia periódico para exportar aún más datos a Azure Data Explorer. Este método permite copiar datos de Azure Data Factory solo cuando se acerca a su límite de retención de Azure Sentinel o Log Analytics, lo que evita la duplicación.

### <a name="azure-event-hub"></a>[Centro de eventos de Azure](#tab/adx-event-hub)

En esta sección se describe cómo exportar datos de Azure Sentinel desde Log Analytics a un centro de eventos, donde pueden ingerirse a Azure Data Explorer. De forma similar al [envío de datos directamente a Azure Sentinel y Azure Data Explorer en paralelo](#send-data-directly-to-azure-sentinel-and-azure-data-explorer-in-parallel), este método incluye produce cierta duplicación de los datos a medida que se transmiten a Azure Data Explorer cuando llegan a Log Analytics.

En la imagen siguiente se muestra un flujo de ejemplo de datos exportados a un centro de eventos, desde donde se ingieren a Azure Data Explorer.

:::image type="content" source="media/store-logs-in-adx/ingest-data-to-adx-via-event-hub.png" alt-text="Exportación de datos a Azure Data Explorer con un centro de eventos de Azure.":::

La arquitectura que se muestra en la imagen anterior proporciona toda la experiencia SIEM de Azure Sentinel (como la administración de incidentes, las investigaciones visuales, la búsqueda de amenazas, las visualizaciones avanzadas, UEBA, etc.) para los datos a los que se debe acceder con frecuencia, cada *X* meses. Al mismo tiempo, esta arquitectura también permite consultar datos a largo plazo accediendo a ellos directamente en Azure Data Explorer o mediante Azure Sentinel gracias a la característica de proxy de Azure Data Explorer. Las consultas al almacenamiento de datos a largo plazo de Azure Data Explorer se pueden portar sin cambios de Azure Sentinel a Azure Data Explorer.

> [!NOTE]
> Al exportar varias tablas de datos a Azure Data Explorer mediante centros de eventos, tenga en cuenta que la exportación de datos de Log Analytics tiene limitaciones en lo que respecta al número máximo de centros de eventos por espacio de nombres. Para obtener más información sobre la exportación de datos, consulte [Exportación de datos del área de trabajo de Log Analytics en Azure Monitor](../azure-monitor/logs/logs-data-export.md?tabs=portal).
>
> Para la mayoría de los clientes, se recomienda usar el nivel estándar de Event Hubs. Dependiendo del número de tablas que necesite exportar y la cantidad de tráfico a esas tablas, puede que tenga que usar el nivel Dedicado de Event Hubs. Para obtener más información, consulte la [documentación de Event Hubs](../event-hubs/event-hubs-quotas.md).
>

> [!TIP]
> Si quiere consultar más detalles sobre este procedimiento, vea [Tutorial: Ingesta y consulta de datos de supervisión en Azure Data Explorer](/azure/data-explorer/ingest-data-no-code).
>

**Para exportar datos a Azure Data Explorer mediante centro de eventos**:

1. **Configure la exportación de datos de Log Analytics a un centro de eventos**. Para obtener más información, consulte [Exportación de datos del área de trabajo de Log Analytics en Azure Monitor](../azure-monitor/logs/logs-data-export.md).

1. **Cree de un clúster y de la base de datos de Azure Data Explorer**. Para más información, consulte:

    - [Creación de un clúster y de la base de datos de Azure Data Explorer](/azure/data-explorer/create-cluster-database-portal)
    - [Selección de la SKU de proceso correcta para el clúster de Azure Data Explorer](/azure/data-explorer/manage-cluster-choose-sku)

1. **Cree tablas de destino**. Los datos sin procesar se ingieren primero en una tabla intermedia, donde los datos sin procesar se almacenan, manipulan y expanden.

    Una directiva de actualización, que es similar a una función aplicada a todos los datos nuevos, se usa para ingerir los datos expandidos en la tabla final, que tiene el mismo esquema que la tabla original en Azure Sentinel.

    Establezca la retención de la tabla sin formato en **0** días. Los datos solo se almacenan en la tabla con el formato correcto, y se eliminan en la tabla sin formato en cuanto se transforman.

    Para obtener más información, consulte [Ingesta y consulta de datos de supervisión en Azure Data Explorer](/azure/data-explorer/ingest-data-no-code?tabs=diagnostic-metrics).

1. <a name="mapping"></a>**Cree la asignación de tablas**. Asigne las tablas JSON para definir cómo llegan los registros a la tabla de eventos sin procesar a medida que lleguen de un centro de eventos. Para obtener más información, consulte [Creación de la directiva de actualización para datos de métricas y registros](/azure/data-explorer/ingest-data-no-code?tabs=diagnostic-metrics).

1. **Cree una directiva de actualización y asóciela a la tabla de registros sin formato**. En este paso, cree una función, denominada "directiva de actualización", y asóciela a la tabla de destino para que los datos se transformen en el momento de la ingesta.

    > [!NOTE]
    > Este paso solo es necesario si desea tener tablas de datos en Azure Data Explorer con el mismo esquema y formato que en Azure Sentinel.
    >

    Para obtener más información, consulte [Conexión de un centro de eventos a Azure Data Explorer](/azure/data-explorer/ingest-data-no-code?tabs=activity-logs).

1. **Cree una conexión de datos entre el centro de eventos y la tabla de datos sin procesar en Azure Data Explorer**. Configure Azure Data Explorer con los detalles sobre cómo exportar los datos al centro de eventos.

    Siga las instrucciones de la [documentación Azure Data Explorer](/azure/data-explorer/ingest-data-no-code?tabs=activity-logs) y especifique los detalles siguientes:

    - **Destino**. Indique la tabla específica con los datos sin procesar.
    - **Formato**. Especifique `.json` como formato de tabla.
    - **Asignación para aplicar**. Especifique la tabla de asignación creada en el [paso 4](#mapping) anterior.


1. **Modifique la retención de la tabla de destino**. La [directiva de retención predeterminada de Azure Data Explorer](/azure/data-explorer/kusto/management/retentionpolicy) puede ser mucho más larga de lo necesario.

    Use el siguiente comando para actualizar la directiva de retención a un año:

    ```kusto
    .alter-merge table <tableName> policy retention softdelete = 365d recoverability = disabled
    ```
### <a name="azure-storage--azure-data-factory"></a>[Azure Storage/Azure Data Factory](#tab/azure-storage-azure-data-factory)

En esta sección se describe cómo exportar datos de Azure Sentinel desde Log Analytics a Azure Storage, donde Azure Data Factory puede ejecutar un trabajo normal para exportar los datos a Azure Data Explorer.

Con Azure Storage y Azure Data Factory se pueden copiar datos de Azure Storage solo cuando se está cerca del límite de retención en Azure Sentinel o Log Analytics. No se produce ninguna duplicación de datos, y Azure Data Explorer se usa *solo* para acceder a datos que son anteriores al límite de retención de Azure Sentinel.

> [!TIP]
> Aunque la arquitectura para usar Azure Storage y Azure Data Factory en los datos heredados es más compleja, este método puede ahorrar más costos en general.
>
En la imagen siguiente se muestra un flujo de ejemplo de datos exportados a Azure Storage; desde ahí, Azure Data Factory ejecuta un trabajo normal para exportar aún más datos a Azure Data Explorer.

:::image type="content" source="media/store-logs-in-adx/ingest-data-to-adx-via-azure-storage-azure-data-factory.png" alt-text="Exportación de datos a Azure Data Explorer mediante Azure Storage y Azure Data Factory.":::

**Para exportar datos a Azure Data Explorer mediante Azure Storage y Azure Data Factory**:

1. **Configure la exportación de datos de Log Analytics a un centro de eventos**. Para obtener más información, consulte [Exportación de datos del área de trabajo de Log Analytics en Azure Monitor](../azure-monitor/logs/logs-data-export.md?tabs=portal#enable-data-export).

1. **Cree de un clúster y de la base de datos de Azure Data Explorer**. Para más información, consulte:

    - [Creación de un clúster y de la base de datos de Azure Data Explorer](/azure/data-explorer/create-cluster-database-portal)
    - [Selección de la SKU de proceso correcta para el clúster de Azure Data Explorer](/azure/data-explorer/manage-cluster-choose-sku)

1. **Cree tablas de destino**. Los datos sin procesar se ingieren primero en una tabla intermedia, donde los datos sin procesar se almacenan, manipulan y expanden.

    Una directiva de actualización, que es similar a una función aplicada a todos los datos nuevos, se usa para ingerir los datos expandidos en la tabla final, que tiene el mismo esquema que la tabla original en Azure Sentinel.

    Establezca la retención de la tabla sin formato en **0** días. Los datos solo se almacenan en la tabla con el formato correcto, y se eliminan en la tabla sin formato en cuanto se transforman.

    Para obtener más información, consulte [Ingesta y consulta de datos de supervisión en Azure Data Explorer](/azure/data-explorer/ingest-data-no-code?tabs=diagnostic-metrics).

1. <a name="mapping"></a>**Cree la asignación de tablas**. Asigne las tablas JSON para definir cómo llegan los registros a la tabla de eventos sin procesar a medida que lleguen de un centro de eventos. Para obtener más información, consulte [Creación de la directiva de actualización para datos de métricas y registros](/azure/data-explorer/ingest-data-no-code?tabs=diagnostic-metrics).

1. **Cree una directiva de actualización y asóciela a la tabla de registros sin formato**. En este paso, cree una función, denominada "directiva de actualización", y asóciela a la tabla de destino para que los datos se transformen en el momento de la ingesta.

    > [!NOTE]
    > Este paso solo es necesario si desea tener tablas de datos en Azure Data Explorer con el mismo esquema y formato que en Azure Sentinel.
    >

    Para obtener más información, consulte [Conexión de un centro de eventos a Azure Data Explorer](/azure/data-explorer/ingest-data-no-code?tabs=activity-logs).

1. **Cree una conexión de datos entre el centro de eventos y la tabla de datos sin procesar en Azure Data Explorer**. Configure Azure Data Explorer con los detalles sobre cómo exportar los datos al centro de eventos.

    Siga las instrucciones de la [documentación Azure Data Explorer](/azure/data-explorer/ingest-data-no-code?tabs=activity-logs) y especifique los detalles siguientes:

    - **Destino**. Indique la tabla específica con los datos sin procesar.
    - **Formato**. Especifique `.json` como formato de tabla.
    - **Asignación para aplicar**. Especifique la tabla de asignación creada en el [paso 4](#mapping) anterior.

1. **Configure la canalización de Azure Data Factory**:

    - Cree servicios vinculados para Azure Storage y Azure Data Explorer. Para más información, consulte:

        - [Copia y transformación de datos en Azure Blob Storage mediante Azure Data Factory](../data-factory/connector-azure-blob-storage.md)
        - [Copie datos con Azure Data Explorer como origen o destino mediante Azure Data Factory](../data-factory/connector-azure-data-explorer.md).

    - Cree un conjunto de datos desde Azure Storage. Para obtener más información, consulte [Conjuntos de datos de Azure Data Factory](../data-factory/concepts-datasets-linked-services.md).

    - Cree una canalización de datos con una operación de copia basada en las propiedades **LastModifiedDate**.

        Para obtener más información, consulte [Copia de archivos nuevos y cambiados por **LastModifiedDate** con Azure Data Factory](../data-factory/solution-template-copy-new-files-lastmodifieddate.md).

---

## <a name="design-considerations"></a>Consideraciones de diseño

Al almacenar los datos de Azure Sentinel en Azure Data Explorer, tenga en cuenta los siguientes elementos:

|Consideración  |Description  |
|---------|---------|
|**SKU y tamaño del clúster**     | Planee cuidadosamente el número de nodos y la SKU de máquina virtual del clúster. Estos factores determinarán la cantidad de potencia de procesamiento y el tamaño de la memoria caché activa (SSD y memoria). Cuanto mayor sea la caché, más datos podrá consultar con un mayor rendimiento. <br><br>Le recomendamos que visite la [calculadora de dimensionamiento de Azure Data Explorer](https://dataexplorer.azure.com/AzureDataExplorerCostEstimator.html), donde puede experimentar con diferentes configuraciones y ver el costo resultante. <br><br>Azure Data Explorer también tiene una funcionalidad de escalabilidad automática que toma decisiones inteligentes para agregar o quitar nodos según sea necesario en función de la carga del clúster. Para obtener más información, consulte [Administración del escalado horizontal de clústeres en Azure Data Explorer para ajustarse a los cambios en la demanda](/azure/data-explorer/manage-cluster-horizontal-scaling).      |
|**Caché activa/inactiva**     | Azure Data Explorer proporciona control sobre las tablas de datos que están en la caché activa y devuelve resultados más rápido. Si tiene grandes cantidades de datos en el clúster de Azure Data Explorer, puede que desee dividir las tablas por mes, de modo que tenga una mayor granularidad en los datos que están presentes en la caché activa. <br><br>Para obtener más información, consulte [Directiva de caché (caché activa e inactiva)](/azure/data-explorer/kusto/management/cachepolicy).       |
|**Retención**     |   En Azure Data Explorer, puede configurar cuándo se quitan los datos de una base de datos o una tabla específica, lo que también constituye una parte importante de la limitación de los costos de almacenamiento. <br><br> Para más información, consulte [Directiva de retención](/azure/data-explorer/kusto/management/retentionpolicy).       |
|**Seguridad**     |  Varias opciones de configuración de Azure Data Explorer pueden ayudarle a proteger los datos, como la administración de identidades o el cifrado. Específicamente para el control de acceso basado en rol (RBAC), Azure Data Explorer puede configurarse para restringir el acceso a bases de datos, tablas o incluso filas dentro de una tabla. Para obtener más información, vea [Seguridad en Azure Data Explorer](/azure/data-explorer/security) y [Seguridad de nivel de fila](/azure/data-explorer/kusto/management/rowlevelsecuritypolicy).|
|**Uso compartido de datos**     |   Azure Data Explorer permite que los datos estén disponibles para otras partes, como asociados o proveedores, e incluso comprar datos de otras partes. Para obtener más información, consulte [Uso de Azure Data Share para compartir datos con Azure Data Explorer](/azure/data-explorer/data-share).      |
| **Otros componentes de costos** | Tenga en cuenta estos otros componentes de costos para los métodos siguientes: <br><br>**Exportación de datos mediante un centro de eventos de Azure:** <br>- Costos de exportación de datos de Log Analytics; se cobra por los GB exportados. <br>- Costos del centro de eventos; se cobra por unidad de procesamiento.  <br><br>**Exportación de datos mediante Azure Storage y Azure Data Factory**: <br>- Exportación de datos de Log Analytics; se cobra por los GB exportados. <br>- Azure Storage; se cobra por los GB almacenados. <br>- Azure Data Factory; se cobra por copia de las actividades ejecutadas.
|     |         |

## <a name="next-steps"></a>Pasos siguientes

Independientemente de dónde almacene los datos, siga buscando e investigando mediante Azure Sentinel.

Para más información, consulte:

- [Tutorial: Investigación de incidentes con Azure Sentinel](tutorial-investigate-cases.md)
- [Búsqueda de amenazas con Azure Sentinel](hunting.md)