---
title: Solución de problemas de rendimiento de la actividad de copia
description: Conozca cómo solucionar problemas de rendimiento de la actividad de copia en Azure Data Factory.
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 01/07/2021
ms.openlocfilehash: eee68b8cb533763aff0c1cc6a1ebe19db735461e
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109488580"
---
# <a name="troubleshoot-copy-activity-performance"></a>Solución de problemas de rendimiento de la actividad de copia

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se describe cómo solucionar problemas de rendimiento de la actividad de copia en Azure Data Factory. 

Después de ejecutar una actividad de copia, puede recopilar el resultado de la ejecución y estadísticas de rendimiento en la vista de [supervisión de la actividad de copia](copy-activity-monitoring.md). A continuación se muestra un ejemplo.

![Detalles de la supervisión de la ejecución de la actividad de copia](./media/copy-activity-overview/monitor-copy-activity-run-details.png)

## <a name="performance-tuning-tips"></a>Sugerencias de optimización del rendimiento

En algunos casos, cuando se ejecuta una actividad de copia en Data Factory, se ve el mensaje **"Performance tuning tips"** (Sugerencias para la optimización del rendimiento) en la parte superior, como se muestra en el ejemplo anterior. Las sugerencias señalan el cuello de botella identificado por ADF para la ejecución específica de la copia, junto con recomendaciones sobre cómo aumentar el rendimiento de la copia. Intente llevar a cabo el cambio sugerido y ejecutar la copia de nuevo.

Como referencia, actualmente las sugerencias de optimización del rendimiento proporcionan recomendaciones para los siguientes casos:

| Category              | Sugerencias de optimización del rendimiento                                      |
| --------------------- | ------------------------------------------------------------ |
| Específica del almacén de datos   | Carga de datos en **Azure Synpase Analytics**: se recomienda usar PolyBase o la instrucción COPY si no se usa. |
| &nbsp;                | Copia de datos de o a **Azure SQL Database**: cuando la DTU tiene un uso elevado, se recomienda actualizar a un nivel superior. |
| &nbsp;                | Copia de datos de o a **Azure Cosmos DB**: cuando la RU tiene un uso elevado, se recomienda actualizar a una RU mayor. |
|                       | Copia de datos de una **tabla de SAP**: al copiar una gran cantidad de datos, se recomienda aprovechar la opción de partición del conector SAP para permitir la carga paralela y aumentar el número máximo de particiones. |
| &nbsp;                | Ingesta de datos de **Amazon Redshift**: se recomienda el uso de UNLOAD si no se usa. |
| Limitación del almacén de datos | Si el almacén de datos limita un número de operaciones de lectura o escritura durante la copia, se recomienda realizar una comprobación y aumentar la tasa de solicitudes permitida para el almacén de datos o reducir la carga de trabajo simultánea. |
| Entorno de ejecución de integración  | Si usa un **entorno de ejecución de integración (IR) autohospedado** y la actividad de copia espera mucho tiempo en la cola hasta que el IR tiene disponible un recurso para la ejecución, se recomienda escalar horizontal o verticalmente el entorno de ejecución de integración. |
| &nbsp;                | Si usa una instancia de **Azure Integration Runtime** que se encuentra en una región no óptima, lo que da lugar a una lectura y escritura lentas, se recomienda cambiar la configuración para usar el entorno de ejecución de integración de otra región. |
| Tolerancia a errores       | Si configura la tolerancia a errores y la omisión de filas incompatibles provoca un rendimiento lento, se recomienda asegurarse de que los datos de origen y receptor son compatibles. |
| copia almacenada provisionalmente           | Si la copia almacenada provisionalmente está configurada pero no resulta útil para el par origen-receptor, se recomienda quitarla. |
| Reanudación                | Tenga en cuenta que, cuando la actividad de copia se reanuda desde el último punto de error, pero se cambió la configuración de la unidad de integración de datos (DIU) después de la ejecución original, la nueva configuración de DIU no surte efecto. |

## <a name="understand-copy-activity-execution-details"></a>Descripción de los detalles de ejecución de la actividad de copia

Los detalles de ejecución y las duraciones en la parte inferior de la vista de supervisión de la actividad de copia describen las fases clave por las que pasa la actividad de copia (consulte el ejemplo al principio de este artículo), lo que resulta especialmente útil para solucionar problemas de rendimiento de la copia. El cuello de botella de la ejecución de copia corresponde a la actividad con la mayor duración. Consulte en la tabla siguiente la definición de cada fase y aprenda a [solucionar problemas de la actividad de copia en Azure IR](#troubleshoot-copy-activity-on-azure-ir) y a [solucionar problemas de la actividad de copia en el IR autohospedado](#troubleshoot-copy-activity-on-self-hosted-ir) con dicha información.

| Fase           | Descripción                                                  |
| --------------- | ------------------------------------------------------------ |
| Cola           | Tiempo transcurrido hasta que la actividad de copia se inicia realmente en el entorno de ejecución de integración. |
| Pre-copy script (Script anterior a la copia) | Tiempo transcurrido entre la actividad de copia que comienza en IR y la actividad de copia que finaliza la ejecución del script anterior a la copia en el almacén de datos receptor. Se aplica al configurar el script anterior a la copia para los receptores de base de datos; por ejemplo, al escribir datos en Azure SQL Database, limpiar antes de copiar los datos nuevos. |
| Transferencia        | Tiempo transcurrido entre el final del paso anterior y el IR que transfiere todos los datos del origen al receptor. <br/>Tenga en cuenta que los subpasos en transferencia se ejecutan en paralelo y algunas operaciones no se muestran ahora, por ejemplo, para analizar o generar el formato de archivo.<br><br/>- **Time to first byte** (Tiempo hasta el primer byte): tiempo transcurrido entre el final del paso anterior y el momento en que la instancia de IR recibe el primer byte del almacén de datos de origen. Se aplica a un origen no basado en archivos.<br>- **Listing source** (Lista de orígenes): tiempo empleado en la enumeración de los archivos de origen o las particiones de datos. El último se aplica cuando se configuran las opciones de partición para los orígenes de base de datos; por ejemplo, cuando se copian datos de bases de datos como Oracle, SAP HANA, Teradata, Netezza, etc.<br/>-**Reading from source** (Lectura desde origen): tiempo empleado en la recuperación de datos desde el almacén de datos de origen.<br/>- **Writing to sink** (Escritura en el receptor): tiempo empleado en la escritura de datos en el almacén de datos receptor. Tenga en cuenta que algunos conectores no tienen esta métrica en este momento, como Azure Cognitive Search, Azure Data Explorer, Azure Table Storage, Oracle, SQL Server, Common Data Service, Dynamics 365, Dynamics CRM, Salesforce o Salesforce Service Cloud. |

## <a name="troubleshoot-copy-activity-on-azure-ir"></a>Solución de problemas de la actividad de copia en Azure IR

Siga los [pasos de optimización del rendimiento](copy-activity-performance.md#performance-tuning-steps) para planear y realizar la prueba de rendimiento de su escenario. 

Si el rendimiento de la actividad de copia no satisface sus expectativas, quiere solucionar problemas de una única actividad de copia que se ejecuta en Azure Integration Runtime y aparecen [sugerencias para optimizar el rendimiento](#performance-tuning-tips) en la vista de supervisión de la copia, aplique la recomendación e inténtelo de nuevo. Si no aparecen estas sugerencias, [analice los detalles de ejecución de la actividad de copia](#understand-copy-activity-execution-details), compruebe qué fase presenta la duración **más larga** y aplique las instrucciones siguientes para aumentar el rendimiento de la copia:

- **La fase de script anterior a la copia muestra una larga duración**: esto significa que el script anterior a la copia que se ejecuta en la base de datos receptora tarda mucho tiempo en finalizar. Ajuste la lógica del script anterior a la copia especificado para mejorar el rendimiento. Si necesita más ayuda para mejorar el script, póngase en contacto con el equipo de base de datos.

- **En la fase de transferencia, el tiempo hasta el primer byte muestra una larga duración de trabajo**: significa que la consulta de origen tarda mucho tiempo en devolver datos. Compruebe y optimice la consulta o el servidor. Si necesita más ayuda, póngase en contacto con el equipo del almacén de datos.

- **En la fase de transferencia, el listado de orígenes muestra una larga duración de trabajo**: significa que la enumeración de los archivos de origen o de las particiones de datos de la base de datos de origen es lenta.
  - Al copiar datos desde el origen basado en archivos, si usa el **filtro de comodín** en la ruta de acceso de la carpeta o en el nombre de archivo (`wildcardFolderPath` o `wildcardFileName`) o usa el **filtro de hora de última modificación del archivo** (`modifiedDatetimeStart` o `modifiedDatetimeEnd`), tenga en cuenta que dicho filtro daría lugar a que la actividad de copia enumerase todos los archivos de la carpeta especificada en el lado de cliente y después aplicase el filtro. Esta enumeración de archivos podría convertirse en el cuello de botella, especialmente cuando solo un pequeño conjunto de archivos cumple la regla del filtro.

    - Compruebe si puede [copiar archivos en función del nombre o la ruta de acceso del archivo con particiones de tiempo](tutorial-incremental-copy-partitioned-file-name-copy-data-tool.md). De este modo, no se aporta carga alguna a la enumeración del lado de origen.

    - Compruebe si en su lugar puede usar el filtro nativo del almacén de datos, específicamente "**prefix**" para Amazon S3/Azure Blob/Azure File Storage y "**listAfter/listBefore**" para ADLS Gen1. Estos filtros son del lado servidor del almacén de datos y tendrían un rendimiento mucho mejor.

    - Considere la posibilidad de dividir un conjunto de datos de gran tamaño en varios conjuntos de datos más pequeños y permitir que esos trabajos de copia se ejecuten simultáneamente, abordando cada uno de ellos una parte de los datos. Puede hacerlo con Lookup/GetMetadata + ForEach + Copy. Consulte las plantillas de solución de [Copia de archivos de varios contenedores](solution-template-copy-files-multiple-containers.md) o [Migración de datos de Amazon S3 a ADLS Gen2](solution-template-migration-s3-azure.md) como ejemplo general.

  - Compruebe si ADF notifica algún error de limitación en el origen o si el almacén de datos muestra un estado de uso elevado. Si es así, reduzca las cargas de trabajo en el almacén de datos o intente ponerse en contacto con el administrador del almacén de datos para aumentar el límite o los recursos disponibles.

  - Use una instancia de Azure IR en la misma región que el almacén de datos de origen o en una región próxima.

- **En la fase de transferencia, la lectura desde el origen muestra una larga duración de trabajo**: 

  - Adopte el procedimiento recomendado de carga de datos específico del conector si es aplicable. Por ejemplo, al copiar datos desde [Amazon Redshift](connector-amazon-redshift.md), realice la configuración para que use UNLOAD de Redshift.

  - Compruebe si ADF notifica algún error de limitación en el origen o si el almacén de datos muestra un uso elevado. Si es así, reduzca las cargas de trabajo en el almacén de datos o intente ponerse en contacto con el administrador del almacén de datos para aumentar el límite o los recursos disponibles.

  - Compruebe el patrón del origen y el receptor de la copia: 

    - Si el patrón de copia admite más de cuatro unidades de integración de datos (DIU), consulte [esta sección](copy-activity-performance-features.md#data-integration-units) para obtener detalles; por lo general, puede intentar aumentar las unidades para obtener un mejor rendimiento. 

    - En caso contrario, considere la posibilidad de dividir un conjunto de datos de gran tamaño en varios conjuntos de datos más pequeños y permitir que esos trabajos de copia se ejecuten simultáneamente, abordando cada uno de ellos una parte de los datos. Puede hacerlo con Lookup/GetMetadata + ForEach + Copy. Consulte las plantillas de solución de [Copia de archivos de varios contenedores](solution-template-copy-files-multiple-containers.md), [Migración de datos de Amazon S3 a ADLS Gen2](solution-template-migration-s3-azure.md) o [Copia masiva con una tabla de control](solution-template-bulk-copy-with-control-table.md) como ejemplo general.

  - Use una instancia de Azure IR en la misma región que el almacén de datos de origen o en una región próxima.

- **En la fase de transferencia, la escritura en el receptor muestra una larga duración de trabajo**:

  - Adopte el procedimiento recomendado de carga de datos específico del conector si es aplicable. Por ejemplo, al copiar datos en [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md), use PolyBase o la instrucción COPY. 

  - Compruebe si ADF notifica algún error de limitación en el receptor o si el almacén de datos muestra un uso elevado. Si es así, reduzca las cargas de trabajo en el almacén de datos o intente ponerse en contacto con el administrador del almacén de datos para aumentar el límite o los recursos disponibles.

  - Compruebe el patrón del origen y el receptor de la copia: 

    - Si el patrón de copia admite más de cuatro unidades de integración de datos (DIU), consulte [esta sección](copy-activity-performance-features.md#data-integration-units) para obtener detalles; por lo general, puede intentar aumentar las unidades para obtener un mejor rendimiento. 

    - De lo contrario, ajuste gradualmente las [copias en paralelo](copy-activity-performance-features.md); tenga en cuenta que demasiadas copias en paralelo pueden perjudicar el rendimiento.

  - Use una instancia de Azure IR en la misma región que el almacén de datos receptor o en una región próxima.

## <a name="troubleshoot-copy-activity-on-self-hosted-ir"></a>Solución de problemas de la actividad de copia en IR autohospedado

Siga los [pasos de optimización del rendimiento](copy-activity-performance.md#performance-tuning-steps) para planear y realizar la prueba de rendimiento de su escenario. 

Si el rendimiento de la copia no satisface sus expectativas, quiere solucionar problemas de una única actividad de copia que se ejecuta en Azure Integration Runtime y aparecen [sugerencias para optimizar el rendimiento](#performance-tuning-tips) en la vista de supervisión de la copia, aplique la recomendación e inténtelo de nuevo. Si no aparecen estas sugerencias, [analice los detalles de ejecución de la actividad de copia](#understand-copy-activity-execution-details), compruebe qué fase presenta la duración **más larga** y aplique las instrucciones siguientes para aumentar el rendimiento de la copia:

- **La fase de cola muestra una larga duración**: significa que la actividad de copia espera mucho tiempo en la cola hasta que la instancia de IR autohospedado tiene el recurso para la ejecución. Compruebe la capacidad y el uso de IR y [escale vertical u horizontalmente](create-self-hosted-integration-runtime.md#high-availability-and-scalability) según la carga de trabajo.

- **En la fase de transferencia, el tiempo hasta el primer byte muestra una larga duración de trabajo**: significa que la consulta de origen tarda mucho tiempo en devolver datos. Compruebe y optimice la consulta o el servidor. Si necesita más ayuda, póngase en contacto con el equipo del almacén de datos.

- **En la fase de transferencia, el listado de orígenes muestra una larga duración de trabajo**: significa que la enumeración de los archivos de origen o de las particiones de datos de la base de datos de origen es lenta.

  - Compruebe si la máquina del entorno de ejecución de integración autohospedado tiene una latencia baja al conectarse al almacén de datos de origen. Si el origen está en Azure, puede usar [esta herramienta](http://www.azurespeed.com/Azure/Latency) para comprobar la latencia de la máquina de IR autohospedado en la región de Azure; es mejor un valor menor.

  - Al copiar datos desde el origen basado en archivos, si usa el **filtro de comodín** en la ruta de acceso de la carpeta o en el nombre de archivo (`wildcardFolderPath` o `wildcardFileName`) o usa el **filtro de hora de última modificación del archivo** (`modifiedDatetimeStart` o `modifiedDatetimeEnd`), tenga en cuenta que dicho filtro daría lugar a que la actividad de copia enumerase todos los archivos de la carpeta especificada en el lado de cliente y después aplicase el filtro. Esta enumeración de archivos podría convertirse en el cuello de botella, especialmente cuando solo un pequeño conjunto de archivos cumple la regla del filtro.

    - Compruebe si puede [copiar archivos en función del nombre o la ruta de acceso del archivo con particiones de tiempo](tutorial-incremental-copy-partitioned-file-name-copy-data-tool.md). De este modo, no se aporta carga alguna a la enumeración del lado de origen.

    - Compruebe si en su lugar puede usar el filtro nativo del almacén de datos, específicamente "**prefix**" para Amazon S3/Azure Blob/Azure File Storage y "**listAfter/listBefore**" para ADLS Gen1. Estos filtros son del lado servidor del almacén de datos y tendrían un rendimiento mucho mejor.

    - Considere la posibilidad de dividir un conjunto de datos de gran tamaño en varios conjuntos de datos más pequeños y permitir que esos trabajos de copia se ejecuten simultáneamente, abordando cada uno de ellos una parte de los datos. Puede hacerlo con Lookup/GetMetadata + ForEach + Copy. Consulte las plantillas de solución de [Copia de archivos de varios contenedores](solution-template-copy-files-multiple-containers.md) o [Migración de datos de Amazon S3 a ADLS Gen2](solution-template-migration-s3-azure.md) como ejemplo general.

  - Compruebe si ADF notifica algún error de limitación en el origen o si el almacén de datos muestra un estado de uso elevado. Si es así, reduzca las cargas de trabajo en el almacén de datos o intente ponerse en contacto con el administrador del almacén de datos para aumentar el límite o los recursos disponibles.

- **En la fase de transferencia, la lectura desde el origen muestra una larga duración de trabajo**: 

  - Compruebe si la máquina del entorno de ejecución de integración autohospedado tiene una latencia baja al conectarse al almacén de datos de origen. Si el origen está en Azure, puede usar [esta herramienta](http://www.azurespeed.com/Azure/Latency) para comprobar la latencia de la máquina de IR autohospedado en las regiones de Azure; es mejor un valor menor.

  - Compruebe si la máquina de IR autohospedado tiene suficiente ancho de banda de entrada para leer y transferir los datos de forma eficaz. Si el almacén de datos de origen está en Azure, puede usar [esta herramienta](https://www.azurespeed.com/Azure/Download) para comprobar la velocidad de descarga.

  - Compruebe la tendencia de uso de memoria y CPU del IR autohospedado en Azure Portal -> la factoría de datos -> página de información general. Considere la posibilidad de [escalar vertical u horizontalmente el entorno de ejecución de integración](create-self-hosted-integration-runtime.md#high-availability-and-scalability) si el uso de CPU es alto o hay poca memoria disponible.

  - Adopte el procedimiento recomendado de carga de datos específico del conector si es aplicable. Por ejemplo:

    - \- Cuando copie datos desde [Oracle](connector-oracle.md#oracle-as-source), [Netezza](connector-netezza.md#netezza-as-source), [Teradata](connector-teradata.md#teradata-as-source), [SAP HANA](connector-sap-hana.md#sap-hana-as-source), [SAP Table](connector-sap-table.md#sap-table-as-source) y [SAP Open Hub](connector-sap-business-warehouse-open-hub.md#sap-bw-open-hub-as-source), habilite las opciones de partición de datos para copiar datos en paralelo.

    - Al copiar datos desde [HDFS](connector-hdfs.md), realice la configuración para usar DistCp.

    - Al copiar datos desde [Amazon Redshift](connector-amazon-redshift.md), realice la configuración para usar UNLOAD de Redshift.

  - Compruebe si ADF notifica algún error de limitación en el origen o si el almacén de datos muestra un uso elevado. Si es así, reduzca las cargas de trabajo en el almacén de datos o intente ponerse en contacto con el administrador del almacén de datos para aumentar el límite o los recursos disponibles.

  - Compruebe el patrón del origen y el receptor de la copia: 

    - Si copia datos desde almacenes de datos con la opción de partición habilitada, considere la posibilidad de ajustar gradualmente las [copias en paralelo](copy-activity-performance-features.md); tenga en cuenta que demasiadas copias en paralelo pueden perjudicar al rendimiento.

    - En caso contrario, considere la posibilidad de dividir un conjunto de datos de gran tamaño en varios conjuntos de datos más pequeños y permitir que esos trabajos de copia se ejecuten simultáneamente, abordando cada uno de ellos una parte de los datos. Puede hacerlo con Lookup/GetMetadata + ForEach + Copy. Consulte las plantillas de solución de [Copia de archivos de varios contenedores](solution-template-copy-files-multiple-containers.md), [Migración de datos de Amazon S3 a ADLS Gen2](solution-template-migration-s3-azure.md) o [Copia masiva con una tabla de control](solution-template-bulk-copy-with-control-table.md) como ejemplo general.

- **En la fase de transferencia, la escritura en el receptor muestra una larga duración de trabajo**:

  - Adopte el procedimiento recomendado de carga de datos específico del conector si es aplicable. Por ejemplo, al copiar datos en [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md), use PolyBase o la instrucción COPY. 

  - Compruebe si la máquina del IR autohospedado tiene una latencia baja al conectarse al almacén de datos receptor. Si el receptor está en Azure, puede usar [esta herramienta](http://www.azurespeed.com/Azure/Latency) para comprobar la latencia de la máquina de IR autohospedado en la región de Azure; es mejor un valor menor.

  - Compruebe si la máquina de IR autohospedado tiene suficiente ancho de banda de salida para transferir y escribir los datos de forma eficaz. Si el almacén de datos receptor está en Azure, puede usar [esta herramienta](https://www.azurespeed.com/Azure/UploadLargeFile) para comprobar la velocidad de carga.

  - Compruebe la tendencia de uso de memoria y CPU del IR autohospedado en Azure Portal -> la factoría de datos -> página de información general. Considere la posibilidad de [escalar vertical u horizontalmente el entorno de ejecución de integración](create-self-hosted-integration-runtime.md#high-availability-and-scalability) si el uso de CPU es alto o hay poca memoria disponible.

  - Compruebe si ADF notifica algún error de limitación en el receptor o si el almacén de datos muestra un uso elevado. Si es así, reduzca las cargas de trabajo en el almacén de datos o intente ponerse en contacto con el administrador del almacén de datos para aumentar el límite o los recursos disponibles.

  - Considere la posibilidad de ajustar gradualmente las [copias en paralelo](copy-activity-performance-features.md); tenga en cuenta que demasiadas copias en paralelo pueden perjudicar el rendimiento.


## <a name="connector-and-ir-performance"></a>Rendimiento del conector e IR 

En esta sección se exploran algunas guías de solución de problemas de rendimiento para un tipo de conector o entorno de ejecución de integración concretos.

### <a name="activity-execution-time-varies-using-azure-ir-vs-azure-vnet-ir"></a>El tiempo de ejecución de la actividad varía según se use Azure IR o IR de red virtual de Azure

El tiempo de ejecución de la actividad varía cuando el conjunto de datos se basa en diferentes entornos de ejecución de integración.

- **Síntomas**: El simple cambio de la lista desplegable Servicio vinculado en el conjunto de datos realiza las mismas actividades de canalización, pero tiene tiempos de ejecución radicalmente diferentes. Cuando el conjunto de datos se basa en el entorno de ejecución de integración de la red virtual administrada, tarda más tiempo promedio que la ejecución basada en el entorno de ejecución de integración predeterminado.  

- **Causa**: Al comprobar los detalles de las ejecuciones de canalización, puede observar que la canalización lenta se ejecuta en el entorno de ejecución de integración de la VNET administrada (Virtual Network), mientras que la normal se ejecuta en Azure IR. Por diseño, el entorno de ejecución de integración de la red virtual administrada lleva más tiempo en la cola que Azure IR, ya que no se reserva un nodo de proceso por factoría de datos, por lo que hay una preparación para que se inicie cada actividad de copia y se produce principalmente en la unión a una red virtual y no en Azure IR. 



    
### <a name="low-performance-when-loading-data-into-azure-sql-database"></a>Bajo rendimiento al cargar datos en Azure SQL Database

- **Síntomas**: La copia de datos en Azure SQL Database es lenta.

- **Causa**: La causa principal del problema suele ser un cuello de botella en el lado de Azure SQL Database. Las posibles causas son las siguientes:

    - El nivel de Azure SQL Database no es suficientemente alto.

    - El uso de la DTU de Azure SQL Database está cerca del 100 %. Puede [supervisar el rendimiento](../azure-sql/database/monitor-tune-overview.md) y considerar la posibilidad de actualizar el nivel de Azure SQL Database.

    - Los índices no están configurados correctamente. Quite todos los índices antes de la carga de datos y vuelva a crearlos después de completar la carga.

    - WriteBatchSize no es lo suficientemente grande como para ajustarse al tamaño de fila del esquema. Intente aumentar la propiedad para solucionar el problema.

    - En lugar de Bulk insert, se usa el procedimiento almacenado, cuyo rendimiento es previsiblemente menor. 


### <a name="timeout-or-slow-performance-when-parsing-large-excel-file"></a>Tiempo de espera o rendimiento lento al analizar un archivo de Excel de gran tamaño

- **Síntomas**:

    - Al crear un conjunto de datos de Excel e importar el esquema de la conexión o el almacenamiento, obtener una vista previa de los datos, enumerar o actualizar las hojas de cálculo, es posible que se produzca un error de tiempo de espera si el tamaño del archivo de Excel es grande.

    - Cuando use la actividad de copia para copiar datos de un archivo de Excel de gran tamaño (> = 100 MB) en otro almacén de datos, es posible que experimente un rendimiento lento o un problema de memoria insuficiente.

- **Causa**: 

    - En operaciones como la importación de esquemas, la vista previa de datos y la enumeración de hojas de cálculo en un conjunto de datos de Excel, el tiempo de espera es de 100 s y es estático. Para un archivo de Excel de gran tamaño, es posible que estas operaciones no finalicen dentro del valor del tiempo de espera.

    - La actividad de copia de ADF lee el archivo de Excel completo en la memoria y, luego, busca la hoja de cálculo y las celdas especificadas para leer los datos. Este comportamiento se debe a que usa ADF del SDK subyacente.

- **Solución:** 

    - Para importar el esquema, puede generar un archivo de ejemplo más pequeño que sea un subconjunto del archivo original y elegir "Importar esquema de archivo de ejemplo" en lugar de "Importar esquema desde conexión/almacén".

    - Para mostrar una hoja de cálculo, en la lista desplegable de hoja de cálculo, puede hacer clic en "Editar" e introducir en su lugar el nombre/índice de la hoja.

    - Para copiar un archivo de Excel de gran tamaño (> 100 MB) en otro almacén, puede usar el origen de Excel de Data Flow cuyo streaming de Sport lee y funciona mejor.
    
## <a name="other-references"></a>Otras referencias

Estas son algunas referencias para la supervisión y la optimización del rendimiento para algunos de los almacenes de datos admitidos:

* Azure Blob Storage: [Objetivos de escalabilidad y rendimiento de Blob Storage](../storage/blobs/scalability-targets.md) y [Lista de comprobación de escalabilidad y rendimiento para Blob Storage](../storage/blobs/storage-performance-checklist.md).
* Azure Table Storage: [Objetivos de escalabilidad y rendimiento de Blob Storage](../storage/tables/scalability-targets.md) y [Lista de comprobación de rendimiento y de escalabilidad para Table Storage](../storage/tables/storage-performance-checklist.md).
* Azure SQL Database: puede [supervisar el rendimiento](../azure-sql/database/monitor-tune-overview.md) y comprobar el porcentaje de la unidad de transacción de base de datos (DTU).
* Azure Synapse Analytics: su funcionalidad se mide en unidades de Data Warehouse (DWU). Consulte [Administración de la potencia de proceso en Azure Synapse Analytics (introducción)](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md).
* Azure Cosmos DB: [Niveles de rendimiento en Azure Cosmos DB](../cosmos-db/performance-levels.md).
* SQL Server: [Supervisión y optimización del rendimiento](/sql/relational-databases/performance/monitor-and-tune-for-performance)
* Servidor de archivos local: [Performance tuning for file servers](/previous-versions//dn567661(v=vs.85)) (Ajuste del rendimiento para los servidores de archivos).

## <a name="next-steps"></a>Pasos siguientes
Consulte los restantes artículos acerca de la actividad de copia:

- [Información general de la actividad de copia](copy-activity-overview.md)
- [Guía de escalabilidad y rendimiento de la actividad de copia](copy-activity-performance.md)
- [Características de optimización del rendimiento de la actividad de copia](copy-activity-performance-features.md)
- [Uso de Azure Data Factory para migrar datos del lago de datos y el almacenamiento de datos a Azure](data-migration-guidance-overview.md)
- [Migración de datos de AWS S3 a Azure Storage](data-migration-guidance-s3-azure-storage.md)
