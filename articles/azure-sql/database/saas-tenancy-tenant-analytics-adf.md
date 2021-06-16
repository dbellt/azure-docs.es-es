---
title: Ejecución de consultas de análisis en bases de datos de inquilino
description: Consultas de análisis entre inquilinos con datos extraídos de Azure SQL Database, Azure Synapse Analytics, Azure Data Factory o Power BI.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 12/18/2018
ms.openlocfilehash: e6d06cb63b3fa52e83605abf565afdb7eb9e167b
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110691745"
---
# <a name="explore-saas-analytics-with-azure-sql-database-azure-synapse-analytics-data-factory-and-power-bi"></a>Exploración del análisis de SaaS con Azure SQL Database, Azure Synapse Analytics, Data Factory y Power BI
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial se le guiará a través de un escenario de análisis completo. El escenario muestra cómo los análisis de datos de los inquilinos facilitan la toma de decisiones inteligentes a los proveedores de software. Con los datos extraídos de cada base de datos de inquilino, utilice los análisis para obtener información del comportamiento del inquilino, incluido el uso de la aplicación SaaS de ejemplo Wingtip Tickets. Este escenario engloba tres pasos:

1. **Extraer datos** de cada base de datos de inquilinos en un almacén de análisis, en este caso, un grupo de SQL dedicado.
2. **Optimizar los datos extraídos** para el procesamiento de análisis.
3. Use las herramientas de **Business Intelligence** para extraer información útil que sirva para tomar decisiones.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Crear el almacén de análisis de inquilinos para la carga.
> - Utilizar Azure Data Factory (ADF) para extraer datos de cada base de datos de inquilinos en el almacenamiento de datos de análisis.
> - Optimizar los datos extraídos (reorganizarlos en un esquema de estrella).
> - Consultar el almacenamiento de datos de análisis.
> - Usar Power BI para la visualización de datos para resaltar tendencias en los datos de inquilino y hacer recomendaciones para mejoras.

![architectureOverView](./media/saas-tenancy-tenant-analytics-adf/adf_overview.png)

## <a name="analytics-over-extracted-tenant-data"></a>Análisis en los datos extraídos de los inquilinos

Las aplicaciones SaaS pueden contener gran cantidad de datos de inquilino en la nube. Estos datos pueden suponer una valiosa fuente de información sobre la operación y el uso de la aplicación y el comportamiento de los inquilinos. Esta información puede guiarle a la hora de desarrollar características, mejorar la facilidad de uso y realizar otras inversiones en las aplicaciones y la plataforma.

El acceso a los datos para todos los inquilinos es sencillo si todos los datos se encuentran en una única base de datos multiinquilino. Sin embargo, el acceso es más complejo si se distribuyen a escala entre miles de bases de datos. Una forma de reducir la complejidad consiste en extraer los datos en una base de datos de análisis o en un almacenamiento de datos para la consulta.

En este tutorial se presenta un escenario de análisis completo para la aplicación Wingtip Tickets. En primer lugar, [Azure Data Factory (ADF)](../../data-factory/introduction.md) se utiliza como herramienta de orquestación para extraer los datos de venta de entradas y relacionados de cada base de datos de inquilinos. Estos datos se cargan en tablas de ensayo en un almacenamiento de análisis. El almacén de análisis podría ser un grupo de SQL dedicado o una instancia de SQL Database. En este tutorial se usa [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) como almacén de análisis.

A continuación, los datos extraídos se transforman en un conjunto de tablas en un [esquema de estrella](https://www.wikipedia.org/wiki/Star_schema) y se cargan. Las tablas constan de una tabla de hechos central más tablas de dimensiones relacionadas:

- La tabla de hechos central del esquema de estrella contiene datos de las entradas.
- Las tablas de dimensiones contienen datos sobre lugares, eventos, clientes y fechas de compra.

De forma conjunta, las tablas centrales y de dimensiones permiten realizar un procesamiento analítico eficaz. El esquema de estrella usado en este tutorial se muestra en la siguiente imagen:

![Diagrama que muestra el esquema de estrella que se usa en este tutorial.](./media/saas-tenancy-tenant-analytics-adf/starschematables.JPG)

Por último, se consultan las tablas de un esquema de estrella. Los resultados de la consulta se muestran visualmente con Power BI para resaltar la información sobre el comportamiento de los inquilinos y el uso que estos hacen de la aplicación. Con este esquema de estrella se ejecutan las consultas que exponen:

- Quién compra las entradas y desde qué lugar.
- Patrones y tendencias de la venta de entradas.
- La popularidad relativa de cada lugar.

En este tutorial se proporcionan ejemplos básicos de la información que puede deducirse de los datos de Wingtip Tickets. Entender la manera en que los lugares utilizan este servicio puede hacer que el proveedor de Wingtip Tickets considere distintos planes de servicio para lugares más o menos activos, por ejemplo.

## <a name="setup"></a>Configurar

### <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

- La aplicación SaaS de Wingtip Tickets con una base de datos por inquilino debe estar implementada. Para implementarla en menos de cinco minutos, consulte el artículo sobre la [implementación y exploración de la aplicación SaaS de Wingtip](./saas-dbpertenant-get-started-deploy.md).
- Los scripts de la aplicación SaaS de Wingtip Tickets con una base de datos por inquilino y el [código fuente](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant/) de la aplicación se descargan de GitHub. Consulte las instrucciones de descarga. No olvide *desbloquear el archivo ZIP* antes de extraer su contenido.
- Power BI Desktop debe estar instalado. [Descargue Power BI Desktop](https://powerbi.microsoft.com/downloads/).
- Se debe haber aprovisionado el loto de inquilinos adicionales; vea el [**tutorial de aprovisionamiento de inquilinos**](./saas-dbpertenant-provision-and-catalog.md).

### <a name="create-data-for-the-demo"></a>Creación de datos para la demostración

En este tutorial se explora el análisis de los datos de venta de entradas. En este paso se generan los datos sobre las entradas para todos los inquilinos. Estos datos se extraen para el análisis en un paso posterior. _Asegúrese de aprovisionar el lote de inquilinos_ (tal como se describe anteriormente) para tener suficientes datos para exponer un intervalo de diferentes patrones de compra de entradas.

1. En PowerShell ISE, abra *…\Learning Modules\Operational Analytics\Tenant Analytics DW\Demo-TenantAnalyticsDW.ps1* y defina el siguiente valor:
    - **$DemoScenario** = **1** Purchase tickets for events at all venues
2. Presione **F5** para ejecutar el script y crear un historial de compra de entradas para todos los lugares. Con 20 inquilinos, el script genera decenas de miles de entradas y puede tardar 10 minutos o más.

### <a name="deploy-azure-synapse-analytics-data-factory-and-blob-storage"></a>Implementación de Azure Synapse Analytics, Data Factory y Blob Storage

En la aplicación Wingtip Tickets, los datos transaccionales de los inquilinos se distribuyen en muchas bases de datos. Azure Data Factory (ADF) se usa para coordinar la extracción, la carga y la transformación (ETL) de estos datos en el almacenamiento de datos. Para cargar datos en Azure Synapse Analytics de forma más eficaz, ADF extrae los datos en archivos de blob intermedios y usa [PolyBase](../../synapse-analytics/sql-data-warehouse/design-elt-data-loading.md) para cargar los datos en el almacenamiento de datos.

En este paso se implementan los recursos adicionales para el tutorial: un grupo de SQL dedicado denominado _tenantanalytics_, una instancia de Azure Data Factory denominada _dbtodwload-\<user\>_ y una cuenta de almacenamiento de Azure denominada _wingtipstaging\<user\>_ . La cuenta de almacenamiento se usa para almacenar temporalmente los archivos de datos extraídos como blobs antes de que se carguen en el almacenamiento de datos. En este paso también se implementa el esquema de almacenamiento de datos y se definen las canalizaciones de ADF que orquestan el proceso de extracción, carga y transformación.

1. En PowerShell ISE, abra *…\Learning Modules\Operational Analytics\Tenant Analytics DW\Demo-TenantAnalyticsDW.ps1* y defina:
    - **$DemoScenario** = **2** Implementación del almacenamiento de datos de análisis de inquilinos, del almacenamiento en blobs y la factoría de datos
1. Presione **F5** para ejecutar el script de demostración e implementar los recursos de Azure.

Ahora, revise los recursos de Azure que ha implementado:

#### <a name="tenant-databases-and-analytics-store"></a>Bases de datos de inquilinos y almacén de análisis

Use [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) para conectarse a los servidores **tenants1-dpt-&lt;usuario&gt;** y **catalog-dpt-&lt;usuario&gt;** . Reemplace &lt;usuario&gt; por el valor usado al implementar la aplicación. Utilice el inicio de sesión = *developer* y la contraseña = *P\@ssword1*. Consulte el [tutorial de introducción](./saas-dbpertenant-wingtip-app-overview.md) para obtener más orientación.

![Conexión a SQL Database desde SSMS](./media/saas-tenancy-tenant-analytics-adf/ssmsSignIn.JPG)

En el Explorador de objetos:

1. Expanda el servidor *tenants1-dpt-&lt;usuario&gt;* .
1. Expanda el nodo Bases de datos y vea la lista de bases de datos de inquilino.
1. Expanda el servidor *catalog-dpt-&lt;usuario&gt;* .
1. Verifique que ve el almacén de análisis que contiene los objetos siguientes:
    1. La tablas **raw_Tickets**, **raw_Customers**, **raw_Events** y **raw_Venues** tienen los datos sin procesar extraídos de las bases de datos de inquilinos.
    1. Las tablas del esquema de estrella son **fact_Tickets**, **dim_Customers**, **dim_Venues**, **dim_Events** y **dim_Dates**.
    1. Se utiliza el procedimiento almacenado, **sp_transformExtractedData** para transformar los datos y cargarlos en las tablas con esquema de estrella.

![La captura de pantalla muestra el Explorador de objetos con las tablas expandidas para mostrar varios objetos de base de datos.](./media/saas-tenancy-tenant-analytics-adf/DWtables.JPG)

#### <a name="blob-storage"></a>Blob Storage

1. En [Azure Portal](https://ms.portal.azure.com), vaya al grupo de recursos que utilizó para implementar la aplicación. Verifique que se ha agregado una cuenta de almacenamiento denominada **wingtipstaging\<user\>** .

   ![DWtables](./media/saas-tenancy-tenant-analytics-adf/adf-staging-storage.PNG)

1. Haga clic en la cuenta de almacenamiento **wingtipstaging\<user\>** para explorar los objetos presentes.
1. Haga clic en el icono de **Blobs**
1. Haga clic en el contenedor **configfile**
1. Verifique que **configfile** contiene un archivo JSON denominado **TableConfig.json**. Este archivo contiene los nombres de tabla de origen y de destino, los nombres de columna y el nombre de la columna de seguimiento.

#### <a name="azure-data-factory-adf"></a>Azure Data Factory (ADF)

En [Azure Portal](https://ms.portal.azure.com), en el grupo de recursos, verifique que se ha agregado una instancia de Azure Data Factory denominada _dbtodwload-\<user\>_ .

 ![adf_portal](./media/saas-tenancy-tenant-analytics-adf/adf-data-factory-portal.png)

En esta sección se explora la factoría de datos creada.
Siga los pasos siguientes para iniciar la factoría de datos:

1. En el portal, haga clic en la factoría de datos denominada **dbtodwload-\<user\>** .
2. Haga clic en el icono **Author & Monitor** (Creación y supervisión) para iniciar el diseñador de Azure Data Factory en una pestaña independiente.

## <a name="extract-load-and-transform-data"></a>Extracción, carga y transformación de datos

Azure Data Factory se usa para orquestar la extracción, la carga y la transformación de datos. En este tutorial se extraen datos de cuatro vistas diferentes de SQL a partir de cada una de las bases de datos de inquilinos: **rawTickets**, **rawCustomers**, **rawEvents** y  **rawVenues**. Estas vistas incluyen el identificador del lugar, por lo que en el almacenamiento de datos se distinguen los datos de cada lugar. Los datos se cargan en las tablas de almacenamiento provisional correspondientes del almacenamiento de datos: **raw_Tickets**, **raw_customers**, **raw_Events** y **raw_Venue**. Después, un procedimiento almacenado transforma los datos sin procesar y rellena las tablas con esquema de estrella: **fact_Tickets**, **dim_Customers**, **dim_Venues**, **dim_Events**  y **dim_Dates**.

En la sección anterior se implementaron y se inicializaron los recursos de Azure necesarios, incluida la factoría de datos. La factoría de datos implementada incluye canalizaciones, los conjuntos de datos, los servicios vinculados, etc., necesarios para extraer, cargar y transformar los datos de inquilino. Vamos a examinar estos objetos con más detalle y a desencadenar la canalización para mover datos desde las bases de datos de inquilinos al almacenamiento de datos.

### <a name="data-factory-pipeline-overview"></a>Introducción a la canalización de la factoría de datos

En esta sección se exploran los objetos creados en la factoría de datos. La ilustración siguiente describe el flujo de trabajo general de la canalización de ADF de este tutorial. Si desea explorar la canalización después y ver los resultados primero, vaya a la sección **Desencadenamiento de la ejecución de la canalización** a continuación.

![adf_overview](./media/saas-tenancy-tenant-analytics-adf/adf-data-factory.PNG)

En la página de información general, cambie a la pestaña **Autor** del panel izquierdo y observe que se han creado tres [canalizaciones](../../data-factory/concepts-pipelines-activities.md) y tres [conjuntos de datos](../../data-factory/concepts-datasets-linked-services.md).
![adf_author](./media/saas-tenancy-tenant-analytics-adf/adf_author_tab.JPG)

Las tres canalizaciones anidadas son: SQLDBToDW, DBCopy y TableCopy.

**Canalización 1: SQLDBToDW** busca los nombres de las bases de datos de inquilinos que se almacenan en la base de datos de catálogo (nombre de la tabla: [__ShardManagement].[ShardsGlobal]) y, para cada base de datos de inquilinos, ejecuta la canalización **DBCopy**. Al finalizar se ejecuta el esquema de procedimiento almacenado **sp_TransformExtractedData**. Este procedimiento almacenado transforma los datos cargados en las tablas de ensayo y rellena las tablas con esquema de estrella.

**Canalización 2: DBCopy** busca los nombres de las tablas y las columnas de origen de un archivo de configuración almacenado en Blob Storage.  La canalización **TableCopy** se ejecuta para cada una de las cuatro tablas: TicketFacts, CustomerFacts, EventFacts y VenueFacts. La actividad **[ForEach](../../data-factory/control-flow-for-each-activity.md)** se ejecuta en paralelo para todas las 20 bases de datos. ADF permite un máximo de 20 iteraciones en bucle para la ejecución en paralelo. Considere la posibilidad de crear varias canalizaciones para más bases de datos.

**Canalización 3: TableCopy** utiliza los números de versión de fila de SQL Database (_rowversion_) para identificar las filas que se hayan cambiado o actualizado. Esta actividad busca la versión de fila inicial y final para extraer filas de las tablas de origen. La tabla **CopyTracker** que se almacena en cada base de datos de inquilinos realiza un seguimiento de la última fila que se extrajo de cada tabla de origen en cada ejecución. Las filas nuevas o cambiadas se copian en las tablas de almacenamiento provisional correspondientes del almacenamiento de datos: **raw_Tickets**, **raw_Customers**, **raw_Events** y **raw_Venue**. Por último, la última versión de fila se guarda en la tabla **CopyTracker** para usarse como versión de fila inicial para la extracción siguiente.

Hay también tres servicios relacionados con parámetros que vinculan la factoría de datos con las bases de datos de origen de SQL Database, el grupo de SQL dedicado de destino y el almacenamiento de blobs intermedio. En la pestaña **Autor**, haga clic en **Conexiones** para explorar los servicios vinculados, tal como se muestra en la siguiente imagen:

![adf_linkedservices](./media/saas-tenancy-tenant-analytics-adf/linkedservices.JPG)

A los tres servicios vinculados les corresponden tres conjuntos de datos que hacen referencia a los datos que se utilizan en las actividades de la canalización como entradas o salidas. Explore cada uno de los conjuntos de datos para observar las conexiones y los parámetros utilizados. _AzureBlob_ apunta al archivo de configuración que contiene las columnas y tablas de origen y destino, así como la columna de seguimiento de cada origen.
  
### <a name="data-warehouse-pattern-overview"></a>Introducción al patrón de almacenamiento de datos

Azure Synapse se utiliza como almacenamiento de análisis que lleva a cabo la agregación con los datos de inquilino. En este ejemplo, se cargan los datos en el almacenamiento de datos con PolyBase. Los datos sin procesar se cargan en tablas de almacenamiento provisional que tienen una columna de identidad para realizar un seguimiento de las filas que se han transformado en tablas con esquema de estrella. En la imagen siguiente se muestra el modelo de carga: ![En el diagrama se muestra el modelo de carga de las tablas de base de datos.](./media/saas-tenancy-tenant-analytics-adf/loadingpattern.JPG)

En este ejemplo se usan tablas de dimensiones de tipo 1 que varían lentamente. Cada dimensión tiene una clave suplente que se define mediante una columna de identidad. Como procedimiento recomendado, la tabla de dimensiones de fecha se rellena automáticamente para ahorrar tiempo. Para las demás tablas de dimensiones se utiliza una instrucción CREATE TABLE AS SELECT... (CTAS) para crear una tabla temporal con las filas existentes modificadas y sin modificar, junto con las claves suplentes. Esto se realiza con IDENTITY_INSERT=ON. A continuación, las filas nuevas se insertan en la tabla con IDENTITY_INSERT=OFF. Para facilitar la reversión, se cambia el nombre de la tabla de dimensiones existente y se cambia el nombre de la tabla temporal para convertirla en la nueva tabla de dimensiones. Antes de cada ejecución, se elimina la tabla de dimensiones antigua.

Las tablas de dimensiones se cargan antes de la tabla de hechos. Esta secuencia garantiza que todas las dimensiones de referencia ya existen para cada hecho que llega. Cuando se cargan los hechos, se hacen coincidir con la clave empresarial de la dimensión correspondiente y se agregan las claves suplentes correspondiente a cada hecho.

En el último paso de la transformación se eliminan los datos de almacenamiento provisionales para la siguiente ejecución de canalización.

### <a name="trigger-the-pipeline-run"></a>Desencadenamiento de la ejecución de la canalización

Siga los pasos siguientes para ejecutar la canalización de extracción, carga y transformación completa para todas las bases de datos de inquilinos:

1. En la pestaña **Author** (Autor) de la interfaz de usuario de ADF, seleccione la canalización **SQLDBToDW** del panel izquierdo.
1. Haga clic en **Trigger** (Desencadenar) y, en el menú desplegable, haga clic en **Trigger now** (Desencadenar ahora). Esta acción ejecuta la canalización inmediatamente. En un escenario de producción, tendría que definir un horario de ejecución de la canalización para actualizar los datos el programa.
  ![En la captura de pantalla se muestran los recursos de fábrica para una canalización denominada SQLDBToDW con la opción Trigger (Desencadenar) ampliada y Trigger Now (Desencadenar ahora) seleccionada.](./media/saas-tenancy-tenant-analytics-adf/adf_trigger.JPG)
1. En la página **Pipeline Run** (Ejecución de canalización), haga clic en **Finish** (Finalizar).

### <a name="monitor-the-pipeline-run"></a>Supervisión de la ejecución de la canalización

1. En la interfaz de usuario de ADF, cambie a la pestaña **Monitor** (Supervisar) del menú de la izquierda.
1. Haga clic en **Refresh** (Actualizar) hasta que el estado de la canalización SQLDBToDW sea **Succeeded** (Correcto).
  ![La captura de pantalla muestra la canalización SQLDBToDW con un estado correcto.](./media/saas-tenancy-tenant-analytics-adf/adf_monitoring.JPG)
1. Conéctese al almacenamiento de datos con SSMS y consulte las tablas con esquema de estrella para verificar que los datos se cargaron en estas tablas.

Una vez completada la canalización, la tabla de hechos contiene datos de la venta de entrada para todos los lugares y las tablas de dimensiones se rellenan con los lugares, eventos y clientes correspondientes.

## <a name="data-exploration"></a>Exploración de datos

### <a name="visualize-tenant-data"></a>Visualización de datos de inquilino

Los datos de las tablas con esquema de estrella proporcionan todos los datos de la venta de entradas necesarios para el análisis. La representación gráfica de los datos facilita la visualización de las tendencias en conjuntos de datos grandes. En esta sección se aprende a usar **Power BI** para manipular y visualizar los datos de inquilino en el almacenamiento de datos.

Siga estos pasos para conectarse a Power BI e importar las vistas creadas anteriormente:

1. Lance Power BI Desktop.
2. En la cinta de opciones de Inicio, seleccione **Obtener datos** y, después, seleccione **Más…** en el menú.
3. En la ventana **Obtener datos**, seleccione **Azure SQL Database**.
4. En la ventana de inicio de sesión de la base de datos, escriba el nombre del servidor (**catalog-dpt-&lt;usuario&gt;.database.windows.net**). Seleccione **Importar** en **Modo Conectividad de datos** y haga clic en **Aceptar**.

    ![Inicio de sesión en Power BI](./media/saas-tenancy-tenant-analytics-adf/powerBISignIn.PNG)

5. Seleccione **Base de datos** en el panel izquierdo y escriba los valores de user name = *developer* y password = *P\@ssword1*. Haga clic en **Conectar**.  

    ![Inicio de sesión en la base de datos](./media/saas-tenancy-tenant-analytics-adf/databaseSignIn.PNG)

6. En el panel **Navegador**, debajo de la base de datos de análisis, seleccione las tablas con esquema de estrella: **fact_Tickets**, **dim_Events**, **dim_Venues**, **dim_Customers** y **dim_Dates**. Después seleccione **Cargar**.

Felicidades. Ha cargado los datos correctamente en Power BI. Ahora puede explorar visualizaciones interesantes para obtener información sobre los inquilinos. Esta es una orientación sobre cómo los análisis permiten proporcionar recomendaciones basadas en los datos al equipo empresarial de Wingtip Tickets. Las recomendaciones pueden ayudarle a optimizar el modelo de negocio y la experiencia del cliente.

Lo primero es analizar los datos de ventas de entradas para ver la variación del uso entre los distintos lugares. Seleccione las opciones que aparecen en Power BI para trazar un gráfico de barras del número total de entradas vendidas en cada lugar. (Debido a la variación aleatoria del generador de entradas, los resultados pueden ser diferentes).

![TotalTicketsByVenues](./media/saas-tenancy-tenant-analytics-adf/TotalTicketsByVenues-DW.PNG)

El gráfico anterior confirma que el número de entradas vendidas en cada lugar varía. Los lugares que venden más entradas usan su servicio con mayor frecuencia que aquellos que venden menos. En este caso, puede haber una oportunidad de adaptar la asignación de recursos a las diferentes necesidades de los inquilinos.

Puede analizar aún más los datos para ver cómo varían las ventas de entradas con el paso del tiempo. Seleccione las opciones que aparecen en la siguiente imagen en Power BI para trazar un gráfico de barras del número total de entradas vendidas cada día durante un período de sesenta días.

![SaleVersusDate](./media/saas-tenancy-tenant-analytics-adf/SaleVersusDate-DW.PNG)

En el gráfico anterior se muestra que las ventas de entradas presentan picos en algunos lugares. Estos picos refuerzan la idea de que algunos lugares pueden estar consumiendo recursos del sistema de manera desproporcionada. Hasta ahora no hay ningún patrón obvio de cuándo se producen los picos.

Después, podemos indagar más en la importancia de los días en que se registran los picos de ventas. ¿Cuándo se producen estos picos después de que las entradas salen a la venta? Para obtener un gráfico de las entradas vendidas al día, seleccione las opciones que aparecen en la siguiente imagen en Power BI.

![SaleDayDistribution](./media/saas-tenancy-tenant-analytics-adf/SaleDistributionPerDay-DW.PNG)

En el gráfico se refleja que algunos lugares venden muchas entradas el primer día de venta. En cuanto las entradas salen a la venta en estos lugares, parece haber una desenfrenada demanda. Tan intensa actividad en algunos lugares puede repercutir en el servicio de otros inquilinos.

Puede profundizar en los datos de nuevo para ver si tal desenfrenada demanda se produce para todos los eventos celebrados en estos lugares. En los gráficos anteriores, observó que Contoso Concert Hall vende muchas entradas y que Contoso también registra un pico de ventas de entradas en determinados días. Practique con las opciones de Power BI para trazar un gráfico de las ventas de entradas acumulativas de Contoso Concert Hall, centrándose en las tendencias de ventas de cada uno de sus eventos. ¿Todos los eventos siguen el mismo patrón de venta? Intente generar un gráfico como el siguiente.

![ContosoSales](./media/saas-tenancy-tenant-analytics-adf/EventSaleTrends.PNG)

Este gráfico de ventas de entradas acumulativo a lo largo del tiempo para Contoso Concert Hall y cada evento muestra que el pico de ventas no se produce para todos los eventos. Practique con las opciones de filtro para explorar las tendencias de venta de otros lugares.

La información de los patrones de venta de entradas pueden permitir a Wingtip Tickets optimizar su modelo de negocio. En lugar de aplicar los mismos cargos a todos los inquilinos, quizá Wingtip debería introducir niveles de servicio con distintos tamaños de proceso. A los lugares más grandes que necesitan vender más entradas al día se les podría ofrecer un nivel superior con un contrato de nivel de servicio (SLA) de categoría superior. Estos lugares podrían tener sus bases de datos agrupadas con límites de recursos por base de datos más altos. Cada nivel de servicio podría tener una asignación de ventas por hora, con tarifas adicionales por exceder la asignación. Los lugares más grandes que tienen intensas actividades de ventas se beneficiarían de los niveles más altos, y Wingtip Tickets puede monetizar su servicio con mayor eficacia.

Mientras tanto, algunos clientes de Wingtip Tickets se quejan de que tienen dificultades para vender las suficientes entradas como para cubrir el costo del servicio. Quizá en esta información se ofrece la oportunidad de impulsar las ventas de entradas para los lugares que presentan déficit de rendimiento. Un aumento de las ventas aumentaría el valor percibido del servicio. Haga clic con el botón derecho en fact_Tickets y seleccione **Nueva medida**. Escriba la siguiente expresión para la nueva medida denominada **AverageTicketsSold**:

```sql
AverageTicketsSold = DIVIDE(DIVIDE(COUNTROWS(fact_Tickets),DISTINCT(dim_Venues[VenueCapacity]))*100, COUNTROWS(dim_Events))
```

Seleccione las siguientes opciones de visualización para obtener un gráfico del porcentaje de entradas vendidas en cada lugar para determinar su éxito relativo.

![AvgTicketsByVenues](./media/saas-tenancy-tenant-analytics-adf/AvgTicketsByVenues-DW.PNG)

En el gráfico anterior se refleja que, aunque la mayoría de los lugares venden más del 80 % de las entradas, algunos se esfuerzan por llegar a más de la mitad del aforo. Practique con los valores Well para seleccionar los porcentajes máximo y mínimo de las entradas vendidas en cada lugar.

## <a name="embedding-analytics-in-your-apps"></a>Inserción de análisis en las aplicaciones

Este tutorial se centra en análisis entre inquilinos que sirve que el proveedor de software comprenda mejor a sus inquilinos. Los análisis también pueden proporcionar información detallada para los _inquilinos_, para ayudarles a administrar su negocio de la forma más eficaz por ellos mismos.

En el ejemplo de Wingtip Tickets anterior se detectó que las ventas de entradas tienden a seguir los patrones predecibles. Esta información podría utilizarse para mejorar el rendimiento de los lugares y que aumenten sus ventas de entradas. Quizá sea una oportunidad para emplear técnicas de aprendizaje automático para predecir las ventas de entradas de cada evento. Los efectos de los cambios de precio también se podrían modelar para la predicción del impacto de los descuentos. Power BI Embedded puede integrarse en una aplicación de administración de eventos para visualizar las ventas previstas, incluido el impacto de los descuentos en las plazas totales vendidas y los ingresos en los eventos con pocas ventas. Con Power BI Embedded, puede integrar incluso la aplicación real del descuento en el precio de las entradas, justo en la experiencia de visualización.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
>
> - Crear el almacén de análisis de inquilinos para la carga.
> - Utilizar Azure Data Factory (ADF) para extraer datos de cada base de datos de inquilinos en el almacenamiento de datos de análisis.
> - Optimizar los datos extraídos (reorganizarlos en un esquema de estrella).
> - Consultar el almacenamiento de datos de análisis.
> - Usar Power BI para la visualización de datos para resaltar tendencias en los datos de inquilino y hacer recomendaciones para mejoras.

Felicidades.

## <a name="additional-resources"></a>Recursos adicionales

- Otros [tutoriales basados en la aplicación SaaS de Wingtip](./saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials).