---
title: Actividad Data Flow
description: Cómo ejecutar flujos de datos desde una canalización de factoría de datos.
author: kromerm
ms.service: data-factory
ms.topic: conceptual
ms.author: makromer
ms.date: 05/20/2021
ms.openlocfilehash: 3793fb3495ca9df9ab8ed408090a8f285f6488b0
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110464653"
---
# <a name="data-flow-activity-in-azure-data-factory"></a>Actividad de Data Flow en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Utilice la actividad de Data Flow para transformar y trasladar datos a través de flujos de datos de asignación. Si no está familiarizado con los flujos de datos, consulte esta [introducción a los flujos de datos de asignación](concepts-data-flow-overview.md).

## <a name="syntax"></a>Sintaxis

```json
{
    "name": "MyDataFlowActivity",
    "type": "ExecuteDataFlow",
    "typeProperties": {
      "dataflow": {
         "referenceName": "MyDataFlow",
         "type": "DataFlowReference"
      },
      "compute": {
         "coreCount": 8,
         "computeType": "General"
      },
      "traceLevel": "Fine",
      "runConcurrently": true,
      "continueOnError": true,      
      "staging": {
          "linkedService": {
              "referenceName": "MyStagingLinkedService",
              "type": "LinkedServiceReference"
          },
          "folderPath": "my-container/my-folder"
      },
      "integrationRuntime": {
          "referenceName": "MyDataFlowIntegrationRuntime",
          "type": "IntegrationRuntimeReference"
      }
}

```

## <a name="type-properties"></a>Propiedades de tipo

Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | --------
dataflow | Referencia al flujo de datos que se está ejecutando. | DataFlowReference | Sí
integrationRuntime | Entorno de proceso en el que se ejecuta el flujo de datos. Si no se especifica, se usará la resolución automática del entorno de Azure Integration Runtime. | IntegrationRuntimeReference | No
compute.coreCount | Número de núcleos utilizados en el clúster de Spark. Solo se puede especificar si se usa la resolución automática de Azure Integration Runtime | 8, 16, 32, 48, 80, 144, 272 | No
compute.computeType | Tipo de proceso utilizado en el clúster de Spark. Solo se puede especificar si se usa la resolución automática de Azure Integration Runtime | "General", "ComputeOptimized", "MemoryOptimized" | No
staging.linkedService | Si utiliza un origen o un receptor de Azure Synapse Analytics, especifique la cuenta de almacenamiento que se utilizará como almacenamiento provisional de PolyBase.<br/><br/>Si Azure Storage está configurado con el punto de conexión de servicio de red virtual, tiene que utilizar la autenticación de identidad administrada con la opción para permitir el servicio de Microsoft de confianza habilitada en la cuenta de almacenamiento; consulte [Efectos del uso de puntos de conexión de servicio de la red virtual con Azure Storage](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage). Obtenga también información sobre las configuraciones necesarias para [Azure Blob](connector-azure-blob-storage.md#managed-identity) y [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#managed-identity) respectivamente.<br/> | LinkedServiceReference | Solo si el flujo de datos lee o escribe en una instancia de Azure Synapse Analytics.
staging.folderPath | Si usa un origen o un receptor de Azure Synapse Analytics, es la ruta de la carpeta de la cuenta de almacenamiento de blobs que se utiliza como almacenamiento provisional de PolyBase. | String | Solo si el flujo de datos lee o escribe en una instancia de Azure Synapse Analytics.
traceLevel | Establecimiento del nivel de registro de la ejecución de actividades de flujo de datos | Fino, Grueso, Ninguno | No

![Ejecución de flujo de datos](media/data-flow/activity-data-flow.png "Ejecución de flujo de datos")

### <a name="dynamically-size-data-flow-compute-at-runtime"></a>Ajuste dinámico del tamaño del proceso de flujo de datos en tiempo de ejecución

Las propiedades Recuento de núcleos y Tipo de proceso se pueden configurar dinámicamente para ajustarse al tamaño de los datos de origen entrantes en tiempo de ejecución. Use actividades de canalización como Búsqueda u Obtener metadatos para averiguar el tamaño de los datos del conjunto de datos de origen. Tras ello, use Agregar contenido dinámico en las propiedades de la actividad del flujo de datos.

> [!NOTE]
> Al elegir núcleos de nodo de trabajo y controlador en flujos de datos de Synapse, siempre se utilizará un mínimo de 3 nodos.

![Flujo de datos dinámicos](media/data-flow/dyna1.png "Flujo de datos dinámicos")

[Este es un breve tutorial en formato de vídeo en el que se explica esta técnica.](https://www.youtube.com/watch?v=jWSkJdtiJNM)

### <a name="data-flow-integration-runtime"></a>Entorno de ejecución de integración de Data Flow

Seleccione el entorno de ejecución de integración que desee usar con la ejecución de la actividad de Data Flow. De forma predeterminada, Data Factory usará la función de resolución automática de Azure Integration Runtime con cuatro núcleos de trabajo. Este entorno de ejecución de integración tendrá un tipo de proceso de uso general y se ejecutará en la misma región que la fábrica. En el caso de canalizaciones operativizadas, se recomienda encarecidamente crear sus propias instancias de Azure Integration Runtime que definan regiones específicas, el tipo de proceso, los recuentos de núcleos y el TTL para la ejecución de actividades del flujo de datos.

Un tipo de proceso mínimo de uso general (no se recomienda la optimización para proceso en cargas de trabajo grandes) con una configuración de 8+8 (16 núcleos virtuales en total) y de 10 minutos es la recomendación mínima para la mayoría de las cargas de trabajo de producción. Al establecer un valor de TTL pequeño, Azure IR puede mantener un clúster semiactivo que no empleará los minutos de tiempo de inicio que un clúster en frío necesita. Puede acelerar aún más la ejecución de los flujos de datos si selecciona "Quick re-use" (Reutilización rápida) en las configuraciones de flujo de datos de Azure IR. Para más información, consulte [Azure Integration Runtime](concepts-integration-runtime.md).

![Azure Integration Runtime](media/data-flow/ir-new.png "Azure Integration Runtime")

> [!IMPORTANT]
> La selección del entorno de ejecución de integración en la actividad de Data Flow solo se aplica a las *ejecuciones activadas* de su canalización. La canalización con flujos de datos se depurará en el clúster que se haya especificado en la sesión de depuración.

### <a name="polybase"></a>PolyBase

Si utiliza Azure Synapse Analytics como origen o receptor, debe elegir una ubicación de almacenamiento provisional para la carga por lotes de PolyBase. PolyBase utiliza la carga por lotes en lugar de cargar los datos fila a fila. PolyBase reduce considerablemente el tiempo de carga en Azure Synapse Analytics.

## <a name="logging-level"></a>Nivel de registro

Si no es necesario que cada ejecución de canalización de las actividades de flujo de datos anote completamente todos los registros de telemetría detallados, tiene la opción de establecer el nivel de registro en "básico" o "ninguno". Al ejecutar los flujos de datos en modo "detallado" (valor predeterminado), está solicitando a ADF que registre la actividad por completo en cada nivel de partición individual durante la transformación de los datos. Esta puede ser una operación costosa; por tanto, habilitar solo el modo detallado al solucionar problemas puede mejorar el flujo de datos y el rendimiento de la canalización en general. El modo "básico" solo registrará las duraciones de las transformaciones, mientras que "ninguno" solo proporcionará un resumen de las duraciones.

![Nivel de registro](media/data-flow/logging.png "Establecimiento del nivel de registro")

## <a name="sink-properties"></a>Propiedades del receptor

La característica de agrupación de los flujos de datos permite establecer el orden de ejecución de los receptores y agruparlos con el mismo número de grupo. Para facilitar la administración de los grupos, puede pedir a ADF que ejecute los receptores, en el mismo grupo, en paralelo. También puede establecer que el grupo de receptores continúe incluso después de que uno de los receptores encuentre un error.

El comportamiento predeterminado de los receptores de flujo de datos es ejecutar cada receptor de forma secuencial, en serie, y producir un error en el flujo de datos cuando se encuentra un error en el receptor. Además, todos los receptores se establecen de forma predeterminada en el mismo grupo, a menos que vaya a las propiedades del flujo de datos y establezca otras prioridades para los receptores.

### <a name="first-row-only"></a>Solo la primera fila

Esta opción solo está disponible para los flujos de datos que tienen los receptores de caché habilitados para la "salida a la actividad". La salida del flujo de datos que se inserta directamente en la canalización está limitada a 2 MB. Establecer "solo la primera fila" le ayuda a limitar la salida de datos del flujo de datos al insertar la salida de la actividad de flujo de datos directamente en la canalización.

![Propiedades del receptor](media/data-flow/sink-properties.png "Establecimiento de propiedades del receptor")

## <a name="parameterizing-data-flows"></a>Flujos de datos con parámetros

### <a name="parameterized-datasets"></a>Conjuntos de datos con parámetros

Si el flujo de datos utiliza conjuntos de datos con parámetros, establezca los valores de los parámetros en la pestaña **Configuración**.

![Parámetros de ejecución de flujo de datos](media/data-flow/params.png "Parámetros")

### <a name="parameterized-data-flows"></a>Flujos de datos con parámetros

Si el flujo de datos tiene parámetros, establezca los valores dinámicos de los parámetros de flujo de datos en la pestaña **Parámetros**. Puede usar el lenguaje de expresiones de canalización de ADF o el lenguaje de expresión de Data Flow para asignar valores dinámicos o literales a los parámetros. Para más información, consulte [Parámetros de Data Flow](parameters-data-flow.md).

### <a name="parameterized-compute-properties"></a>Propiedades de proceso con parámetros.

Puede parametrizar el número de núcleos o el tipo de proceso si usa la resolución automática de Azure Integration Runtime y especifica valores para compute.coreCount y compute.computeType.

![Ejemplo de parámetros de ejecución de flujo de datos](media/data-flow/parameterize-compute.png "Ejemplo de parámetro")

## <a name="pipeline-debug-of-data-flow-activity"></a>Depuración de la canalización de actividades de Data Flow

Para ejecutar una canalización de depuración con una actividad de Data Flow, debe activar el modo de depuración del flujo de datos a través del control deslizante **Data Flow Debug** (Depuración de flujo de datos) en la barra superior. El modo de depuración permite ejecutar el flujo de datos en un clúster de Spark activo. Para más información, consulte [Modo de depuración](concepts-data-flow-debug-mode.md).

![Captura de pantalla que muestra dónde está el botón Depurar](media/data-flow/debug-button-3.png)

La canalización de depuración se ejecuta en el clúster de depuración activo, no en el entorno de ejecución de integración especificado en la configuración de la actividad de Data Flow. Puede elegir el entorno de proceso de depuración al iniciar el modo de depuración.

## <a name="monitoring-the-data-flow-activity"></a>Supervisión de la actividad de Data Flow

La supervisión de la actividad de Data Flow se realiza de forma especial, ya que permite ver información sobre las particiones, el tiempo de cada fase y el linaje de datos. Abra el panel supervisión utilizando el icono con forma de gafas que encontrará en **Acciones**. Para más información, consulte [Supervisión de flujos de datos](concepts-data-flow-monitoring.md).

### <a name="use-data-flow-activity-results-in-a-subsequent-activity"></a>Uso de los resultados de actividad de Data Flow en una actividad posterior

La actividad de flujo de datos genera métricas con respecto al número de filas escritas en cada receptor y las filas leídas de cada origen. Estos resultados se devuelven en la sección `output` del resultado de ejecución de la actividad. Las métricas devueltas están en el formato del siguiente código JSON.

``` json
{
    "runStatus": {
        "metrics": {
            "<your sink name1>": {
                "rowsWritten": <number of rows written>,
                "sinkProcessingTime": <sink processing time in ms>,
                "sources": {
                    "<your source name1>": {
                        "rowsRead": <number of rows read>
                    },
                    "<your source name2>": {
                        "rowsRead": <number of rows read>
                    },
                    ...
                }
            },
            "<your sink name2>": {
                ...
            },
            ...
        }
    }
}
```

Por ejemplo, para obtener el número de filas escritas en un receptor denominado "sink1" en una actividad denominada "dataflowActivity", use `@activity('dataflowActivity').output.runStatus.metrics.sink1.rowsWritten`.

Para obtener el número de filas leídas de un origen denominado "source1" que se usó en ese receptor, use `@activity('dataflowActivity').output.runStatus.metrics.sink1.sources.source1.rowsRead`.

> [!NOTE]
> Si un receptor tiene cero filas escritas, no se mostrará en las métricas. Se puede comprobar la existencia con la función `contains`. Por ejemplo, `contains(activity('dataflowActivity').output.runStatus.metrics, 'sink1')` comprobará si se escribieron filas en sink1.

## <a name="next-steps"></a>Pasos siguientes

Consulte las actividades de flujo de control compatibles con Data Factory: 

- [Actividad If Condition](control-flow-if-condition-activity.md)
- [Actividad de ejecución de canalización](control-flow-execute-pipeline-activity.md)
- [Para cada actividad](control-flow-for-each-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
- [Actividad Lookup](control-flow-lookup-activity.md)
- [Actividad web](control-flow-web-activity.md)
- [Actividad Until](control-flow-until-activity.md)
