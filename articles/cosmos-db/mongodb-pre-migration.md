---
title: Pasos previos a la migración para migraciones de datos a la API de Azure Cosmos DB para MongoDB
description: Este documento proporciona información general sobre los requisitos previos para una migración de datos de MongoDB a Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 05/17/2021
ms.author: anfeldma
ms.openlocfilehash: bb62219b8579fb0e87011ccfcae04b28aa7f0181
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110471136"
---
# <a name="pre-migration-steps-for-data-migrations-from-mongodb-to-azure-cosmos-dbs-api-for-mongodb"></a>Pasos previos a la migración para migraciones de datos de MongoDB a la API de Azure Cosmos DB para MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!IMPORTANT]  
> Lea toda esta guía antes de llevar a cabo los pasos previos a la migración.
>

Esta guía previa a la migración de MongoDB forma parte de la serie sobre la migración de MongoDB. Los pasos críticos de la migración de MongoDB son la migración previa, la migración y la [migración posterior](mongodb-post-migration.md), como se muestra a continuación.

![Diagrama de pasos de migración.](./media/mongodb-pre-migration/overall-migration-steps.png)

## <a name="overview-of-pre-migration"></a>Información general sobre la migración previa

Es fundamental llevar a cabo cierto planeamiento y toma de decisiones sobre la migración por adelantado antes de mover realmente los datos. Este proceso inicial de toma de decisiones es la "migración previa". El objetivo de la migración previa es (1) asegurarse de configurar Azure Cosmos DB para satisfacer los requisitos posteriores a la migración de la aplicación y (2) planear cómo ejecutará la migración.

Siga estos pasos para realizar una migración previa exhaustiva.
* [Descubra los recursos de MongoDB existentes y cree un artefacto para realizar un seguimiento de estos](#pre-migration-discovery).
* [Evalúe la preparación de los recursos de MongoDB existentes para la migración de datos](#pre-migration-assessment).
* [Asigne los recursos de MongoDB existentes a nuevos recursos de Azure Cosmos DB](#pre-migration-mapping).
* [Planee la logística del proceso de migración de un extremo a otro antes de iniciar la migración de datos a escala completa](#execution-logistics).

A continuación, ejecute la migración de acuerdo con el plan previo a la migración.

Por último, realice los [pasos críticos de migración total y optimización posteriores a la migración](mongodb-post-migration.md).

Todos los pasos anteriores son críticos para garantizar una migración correcta.

Al planear una migración, se recomienda que, siempre que sea posible, se planee por recurso.

## <a name="pre-migration-discovery"></a>Detección previa a la migración

El primer paso previo a la migración es la detección de recursos. En este paso, intentará crear una lista completa de los recursos existentes en el patrimonio de datos de MongoDB.

### <a name="create-a-data-estate-migration-spreadsheet"></a>Creación de una hoja de cálculo de migración de patrimonio de datos

Cree una **hoja de cálculo de migración de patrimonio de datos** como documento de seguimiento para la migración mediante el software de productividad que prefiera. 
   * El propósito de esta hoja de cálculo es mejorar la productividad y ayudarle a planear la migración de un extremo a otro.
   * La estructura de la hoja de cálculo es decisión suya. Los siguientes puntos proporcionan algunas recomendaciones.
   * Esta hoja de cálculo debe estructurarse como un registro de los recursos del patrimonio de datos, en forma de lista.
   * Cada fila corresponde a un recurso (base de datos o colección).
   * Cada columna corresponde a una propiedad del recurso; por ahora, debe tener al menos las columnas *nombre* y *tamaño de datos (GB)* , aunque lo ideal es que también pueda recopilar información sobre la versión de MongoDB de cada recurso, en cuyo caso debe agregar también una columna *versión de Mongo*. 
   * Inicialmente, rellenará esta hoja de cálculo con una lista de los recursos existentes en el patrimonio de datos de MongoDB. A medida que avance en esta guía, compilará esta hoja de cálculo en un documento de seguimiento para el planeamiento de la migración de un extremo a otro, y agregará columnas según sea necesario.

### <a name="discover-existing-mongodb-data-estate-resources"></a>Descubrimiento de los recursos existentes del patrimonio de datos de MongoDB

Con una herramienta de detección adecuada, identifique los recursos (bases de datos y colecciones) del patrimonio de datos de MongoDB existente, de la forma más completa posible. 

Estas son algunas herramientas que puede usar para descubrir recursos:
   * [MongoDB Shell](https://www.mongodb.com/try/download/shell)
   * [MongoDB Compass](https://www.mongodb.com/try/download/compass)

## <a name="pre-migration-assessment"></a>Evaluación previa a la migración

En segundo lugar, como preludio para planear la migración, evalúe la preparación de cada recurso del patrimonio de datos para la migración. 

El factor principal que afecta a la preparación es la versión de MongoDB. Azure Cosmos DB admite actualmente las versiones 3.2, 3.6 y 4.0 del protocolo binario de MongoDB. Esperemos que tenga una columna en la hoja de cálculo de planeamiento de la migración para la *versión de MongoDB*. Avance por la hoja de cálculo y resalte los recursos que usan versiones incompatibles de MongoDB para Azure Cosmos DB.

## <a name="pre-migration-mapping"></a>Asignación previa a la migración

Una vez completados los pasos de detección y evaluación, habrá terminado con el lado de la ecuación de MongoDB. Ahora es el momento de planear el de Azure Cosmos DB. ¿Cómo instalará y configurará los recursos de producción de Azure Cosmos DB? Realice el planeamiento *por recurso*, lo que significa que debe agregar las siguientes columnas a la hoja de cálculo de planeamiento: 
* Asignación de Azure Cosmos DB 
* Clave de partición 
* Modelo de datos
* Rendimiento dedicado frente a compartido

En las secciones siguientes se proporciona información más detallada.

### <a name="considerations-when-using-azure-cosmos-dbs-api-for-mongodb"></a>Consideraciones al usar la API de Azure Cosmos DB para MongoDB

Antes de planear el patrimonio de datos de Azure Cosmos DB, asegúrese de comprender los siguientes conceptos de Azure Cosmos DB:

- **Modelo de capacidad**: la capacidad de bases de datos en Azure Cosmos DB se basa en un modelo basado a su vez en el rendimiento. Este modelo se basa en [Unidades de solicitud por segundo](request-units.md), que es una unidad que representa el número de operaciones de base de datos que se pueden ejecutar en una colección por segundo. Esta capacidad se puede asignar en [un nivel de base de datos o de colección](set-throughput.md) y se puede aprovisionar en un modelo de asignación o mediante el [rendimiento aprovisionado de escalabilidad automática](provision-throughput-autoscale.md).

- **Unidades de solicitud**: Cada operación de base de datos tiene un costo de las Unidades de solicitud (RU) en Azure Cosmos DB. Al ejecutarse, se resta del nivel de unidades de solicitud disponible en un segundo dado. Si una solicitud requiere más RU que las RU/s asignadas actualmente, existen dos opciones para resolver el problema: aumentar la cantidad de RU o esperar a que se inicie el próximo segundo y volver a intentar la operación.

- **Capacidad elástica**: la capacidad de una colección o base de datos determinada puede cambiar en cualquier momento. Esto permite a la base de datos adaptarse de forma elástica a los requisitos de rendimiento de su carga de trabajo.

- **Particionamiento automático**: Azure Cosmos DB proporciona un sistema de particionamiento automático que solo requiere una partición (o una clave de partición). El [mecanismo de particionamiento automático](partitioning-overview.md) se comparte entre todas las API de Azure Cosmos DB y permite datos sin problemas y en todo el escalado a través de la distribución horizontal.

### <a name="plan-the-azure-cosmos-db-data-estate"></a>Planeación del patrimonio de datos de Azure Cosmos DB

Descubra qué recursos de Azure Cosmos DB va a crear. Esto significa seguir paso a paso la hoja de cálculo de migración del patrimonio de datos y asignar cada recurso de MongoDB existente a un nuevo recurso de Azure Cosmos DB. 
* Anticipe que cada base de datos de MongoDB se convertirá en una base de datos de Azure Cosmos DB.
* Anticipe que cada colección de MongoDB se convertirá en una colección de Azure Cosmos DB.
* Elija una convención de nomenclatura para sus recursos de Azure Cosmos DB. Bloquear cualquier cambio en la estructura de las bases de datos y colecciones y mantener así los mismos nombres de recursos suele ser una buena opción.
* En MongoDB, las colecciones de particionamiento son opcionales. En Azure Cosmos DB, se particionan todas las colecciones.
* *No suponga que la clave de partición de la colección de MongoDB se convierte en la clave de partición de la colección de Azure Cosmos DB. No suponga que la estructura del documento o modelo de datos de MongoDB existente es la que va a emplear en Azure Cosmos DB.* 
   * La clave de partición es la opción más importante para optimizar la escalabilidad y el rendimiento de Azure Cosmos DB, y la segunda más importante es el modelado de datos. Ambas opciones son inmutables y no se pueden cambiar una vez establecidas; por lo tanto, es muy importante optimizarlas en la fase de planeamiento. Siga las instrucciones de la sección [Decisiones inmutables](#immutable-decisions) para obtener más información.
* Azure Cosmos DB no reconoce ciertos tipos de colección de MongoDB, como las colecciones limitadas. Para estos recursos, solo tiene que crear colecciones de Azure Cosmos DB normales.
* Azure Cosmos DB tiene dos tipos de colección propios: rendimiento compartido y dedicado. El rendimiento compartido frente al rendimiento dedicado es otra decisión crítica e inmutable que es fundamental tomar en la fase de planeamiento. Siga las instrucciones de la sección [Decisiones inmutables](#immutable-decisions) para obtener más información.

### <a name="immutable-decisions"></a>Decisiones inmutables

Las siguientes opciones de configuración de Azure Cosmos DB no se pueden modificar ni deshacer una vez que se ha creado un recurso de Azure Cosmos DB; por lo tanto, es importante definirlas correctamente durante el planeamiento previo a la migración, antes de iniciar las migraciones:
* Siga [esta guía](partitioning-overview.md) para elegir la mejor clave de partición. La creación de particiones, también conocida como particionamiento, es un punto clave de consideración antes de migrar los datos. Azure Cosmos DB usa una creación de particiones totalmente administrada para aumentar la capacidad de una base de datos a fin de cumplir los requisitos de almacenamiento y rendimiento. Esta característica no necesita el hospedaje ni la configuración de los servidores de enrutamiento.   
   * De forma similar, la funcionalidad de creación de particiones agrega capacidad automáticamente y vuelve a equilibrar los datos en consecuencia. Para obtener detalles y recomendaciones sobre cómo elegir la clave de partición correcta para los datos, consulte el [artículo Elección de una clave de partición](partitioning-overview.md#choose-partitionkey). 
* Siga [esta guía](modeling-data.md) para elegir un modelo de datos.
* Siga [esta guía](optimize-cost-throughput.md#optimize-by-provisioning-throughput-at-different-levels) para elegir entre el rendimiento dedicado y compartido para cada recurso que va a migrar.
* [Este](how-to-model-partition-example.md) es un ejemplo real de particionamiento y modelado de datos que le ayudará en el proceso de toma de decisiones.

### <a name="cost-of-ownership"></a>Costo de propiedad

* Calcule el costo de propiedad de los nuevos recursos de Azure Cosmos DB mediante la [calculadora de capacidad de Azure Cosmos DB](https://cosmos.azure.com/capacitycalculator/).

### <a name="estimating-throughput"></a>Estimación del rendimiento

* En Azure Cosmos DB, el rendimiento se aprovisiona de antemano y se mide en Unidades de solicitud (RU) por segundo. A diferencia de las máquinas virtuales o servidores locales, las Unidades de solicitud se pueden escalar y reducir verticalmente fácilmente en cualquier momento. Puede cambiar el número de RU aprovisionadas de forma inmediata. Para más información, consulte [Unidades de solicitud en Azure Cosmos DB](request-units.md).

* Puede usar la [calculadora de capacidad de Azure Cosmos DB](https://cosmos.azure.com/capacitycalculator/) para determinar la cantidad de Unidades de solicitud en función de la configuración de la cuenta de base de datos, la cantidad de datos, el tamaño del documento y las lecturas y escrituras requeridas por segundo.

* Estos son factores clave que afectan al número de Unidades de solicitud necesarias:
   * **Tamaño del documento**: a medida que el tamaño de un elemento o documento aumenta, también lo hace el número de Unidades de solicitud consumidas para leer o escribir el elemento o documento.

   * **Recuento de propiedades del documento**: el número de RU consumidas para crear o actualizar un documento está relacionado con el número, la complejidad y la longitud de sus propiedades. Puede reducir el consumo de Unidades de solicitud para operaciones de escritura [limitando el número de las propiedades indexadas](mongodb-indexing.md).

   * **Patrones de consultas**: la complejidad de una consulta afecta a la cantidad de Unidades de solicitud que una consulta consume. 

* La mejor manera de comprender el costo de las consultas es usar los datos de ejemplo en Azure Cosmos DB [y ejecutar consultas de ejemplo desde el Shell de MongoDB](connect-mongodb-account.md) mediante el comando `getLastRequestStastistics` para obtener el cargo de solicitud, lo que generará el número de Unidades de solicitud consumidas:

    `db.runCommand({getLastRequestStatistics: 1})`

    Este comando generará un documento JSON similar al siguiente:

    ```{  "_t": "GetRequestStatisticsResponse",  "ok": 1,  "CommandName": "find",  "RequestCharge": 10.1,  "RequestDurationInMilliSeconds": 7.2}```

* También puede usar [la configuración de diagnóstico](cosmosdb-monitor-resource-logs.md) para comprender la frecuencia y los patrones de las consultas ejecutadas en Azure Cosmos DB. Los resultados de los registros de diagnóstico se pueden enviar a una cuenta de almacenamiento, una instancia de EventHub o [Azure Log Analytics](../azure-monitor/logs/log-analytics-tutorial.md).  

## <a name="pre-migration-logistics-planning"></a>Planeamiento de la logística previa a la migración

Por último, ahora que tiene una vista del patrimonio de datos existente y un diseño para el nuevo patrimonio de datos de Azure Cosmos DB, está listo para planear cómo ejecutar el proceso de migración de un extremo a otro. Una vez más, realice el planeamiento *por recurso* y agregue columnas a la hoja de cálculo para capturar las dimensiones logísticas siguientes.

### <a name="execution-logistics"></a>Logística de ejecución
* Asigne la responsabilidad de migrar cada recurso existente de MongoDB a Azure Cosmos DB. La forma de aprovechar los recursos del equipo para guiar la migración hasta completarla es decisión suya. En el caso de las migraciones pequeñas, puede hacer que un equipo inicie la migración completa y supervise su progreso. Para migraciones más grandes, podría asignar responsabilidad a los miembros del equipo por recurso para migrar y supervisar ese recurso.
* Una vez que haya asignado la responsabilidad de migrar los recursos, ahora debe elegir las herramientas de migración correctas para la migración. En el caso de migraciones pequeñas, es posible que pueda usar una herramienta de migración, como una herramienta nativa de MongoDB o Azure DMS, para migrar todos los recursos de una sola vez. En el caso de migraciones más grandes o con requisitos especiales, puede elegir herramientas de migración con una granularidad por recurso.
   * Antes de planear qué herramientas de migración usar, se recomienda familiarizarse con las opciones disponibles. [Azure Database Migration Service para la API de Azure Cosmos DB para MongoDB](../dms/tutorial-mongodb-cosmos-db.md) proporciona un mecanismo que simplifica la migración de datos facilitando una plataforma de hospedaje totalmente administrada, opciones de supervisión de la migración y control de limitaciones automático. La lista completa de opciones es la siguiente:

   |**Tipo de migración**|**Solución**|**Consideraciones**|
   |---------|---------|---------|
   |En línea|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)|&bull; Hace uso de la biblioteca BulkExecutor de Azure Cosmos DB <br/>&bull; Adecuada para grandes conjuntos de valores y se encarga de replicar los cambios en directo <br/>&bull; Solo funciona con otros orígenes de MongoDB|
   |Sin conexión|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)|&bull; Hace uso de la biblioteca BulkExecutor de Azure Cosmos DB <br/>&bull; Adecuada para grandes conjuntos de valores y se encarga de replicar los cambios en directo <br/>&bull; Solo funciona con otros orígenes de MongoDB|
   |Sin conexión|[Azure Data Factory](../data-factory/connector-azure-cosmos-db.md)|&bull; Fácil de configurar y admite varios orígenes <br/>&bull; Hace uso de la biblioteca BulkExecutor de Azure Cosmos DB <br/>&bull; Adecuada para grandes conjuntos de datos <br/>&bull; Faltan puntos de comprobación: significa que, si se produce un problema durante el transcurso de la migración, es necesario reiniciar todo el proceso de migración.<br/>&bull; Falta una cola de mensajes con problemas de entrega: significa que algunos archivos erróneos pueden detener todo el proceso de migración. <br/>&bull; Necesita código personalizado para aumentar el rendimiento de lectura de determinados orígenes de datos|
   |Sin conexión|[Herramientas existentes de Mongo (mongodump, mongorestore, Studio3T)](https://azure.microsoft.com/resources/videos/using-mongodb-tools-with-azure-cosmos-db/)|&bull; Fácil de configurar e integrar <br/>&bull; Necesita un control personalizado para las limitaciones|

   * Si el recurso puede tolerar una migración sin conexión, use el diagrama siguiente para elegir la herramienta de migración adecuada:

   ![Herramientas de migración sin conexión.](./media/mongodb-pre-migration/offline-tools.png)

   * Si el recurso requiere una migración en línea, use el diagrama siguiente para elegir la herramienta de migración adecuada:

   ![Herramientas de migración en línea.](./media/mongodb-pre-migration/online-tools.png)

* Una vez que haya elegido las herramientas de migración para cada recurso, el siguiente paso es priorizar los recursos que va a migrar. Una buena priorización puede ayudar a mantener la migración según la programación. Un procedimiento recomendado es priorizar la migración de los recursos que necesitan más tiempo para trasladarse. La migración de estos recursos en primer lugar aportará el mayor progreso hacia la finalización. Además, dado que estas migraciones que consumen mucho tiempo suelen implicar más datos, normalmente suponen un mayor consumo de recursos para la herramienta de migración y, por lo tanto, es más probable que estén expuestas a problemas con la canalización de migración al principio. Esto minimiza la posibilidad de que la programación falle debido a las dificultades con la canalización de migración.
* Planee cómo supervisará el progreso de la migración una vez iniciado. Si está coordinando el trabajo de migración de datos entre un equipo, planee una cadencia regular de sincronizaciones del equipo para tener una visión completa del progreso de las migraciones de alta prioridad.

### <a name="supported-migration-scenarios"></a>Escenarios de migración admitidos

La mejor opción de la herramienta de migración de MongoDB depende del escenario de migración. 

#### <a name="types-of-migrations"></a>Tipos de migraciones

A continuación se muestran las herramientas compatibles para cada escenario de migración:

![Escenarios de migración admitidos.](./media/mongodb-pre-migration/migration-tools-use-case-table.png)

#### <a name="tooling-support-for-mongodb-versions"></a>Compatibilidad con herramientas de las versiones de MongoDB

Dado que va a migrar desde una versión determinada de MongoDB, a continuación se muestran las herramientas compatibles:

![Versiones de MongoDB compatibles con las herramientas de migración.](./media/mongodb-pre-migration/migration-tool-compatibility.png)

### <a name="post-migration"></a>Después de la migración

En la fase previa a la migración, dedique algo de tiempo para planear los pasos que realizará para la migración de aplicaciones y la optimización posterior a la migración.
* En la fase posterior a la migración, ejecutará una migración total de la aplicación para usar Azure Cosmos DB en lugar del patrimonio de datos de MongoDB existente. 
* Haga todo lo posible para planear la indexación, la distribución global, la coherencia y otras propiedades de Azure Cosmos DB *mutables* por recurso; sin embargo, estos valores de configuración de Azure Cosmos DB se *pueden* modificar más adelante, por lo que se pueden esperar ajustes en estos valores en el futuro. No permita que estos aspectos sean una causa de detener el análisis. Aplicará estas configuraciones mutables después de la migración.
* La mejor guía para la migración posterior se puede encontrar [aquí](mongodb-post-migration.md).

## <a name="next-steps"></a>Pasos siguientes
* [Migración de los datos de MongoDB a Cosmos DB mediante Database Migration Service](../dms/tutorial-mongodb-cosmos-db.md) 
* [Aprovisionamiento del rendimiento en contenedores y bases de datos de Azure Cosmos](set-throughput.md)
* [Creación de particiones en Azure Cosmos DB](partitioning-overview.md)
* [Distribución global en Azure Cosmos DB](distribute-data-globally.md)
* [Indexación en Azure Cosmos DB](index-overview.md)
* [Unidades de solicitud en Azure Cosmos DB](request-units.md)
