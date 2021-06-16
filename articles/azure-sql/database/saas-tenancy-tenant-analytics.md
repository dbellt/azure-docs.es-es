---
title: Análisis entre inquilinos mediante datos extraídos
description: Consultas de análisis entre inquilinos mediante datos extraídos de varias instancias de base de datos de Azure SQL en una aplicación de un solo inquilino.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 12/18/2018
ms.openlocfilehash: 1c42a11d5d59007f9e97fbc64e854cd96e29410f
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708041"
---
# <a name="cross-tenant-analytics-using-extracted-data---single-tenant-app"></a>Análisis entre inquilinos mediante datos extraídos: aplicación de un solo inquilino
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]
 
En este tutorial, recorrerá un escenario completo de análisis para la implementación de un solo inquilino. En el escenario se explica cómo los análisis pueden permitir a las empresas tomar decisiones inteligentes. Con los datos extraídos de cada base de datos de inquilino, utilice los análisis para obtener información del comportamiento del inquilino, incluido el uso de la aplicación SaaS de ejemplo Wingtip Tickets. Este escenario engloba tres pasos: 

1.  **Extraer** datos de cada base de datos de y **cargar** en un almacén de análisis.
2.  **Transformar los datos extraídos** para el procesamiento de análisis.
3.  Use las herramientas de **inteligencia empresarial** para extraer información útil que sirva para tomar decisiones. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> - Crear el almacén de análisis de inquilino para extraer en él los datos.
> - Utilizar trabajos elásticos para extraer datos de cada base de datos de inquilino en el almacén de análisis.
> - Optimizar los datos extraídos (reorganizarlos en un esquema de estrella).
> - Consultar la base de datos de análisis.
> - Usar Power BI para la visualización de datos para resaltar tendencias en los datos de inquilino y hacer recomendaciones para mejoras.

![En el diagrama se muestra información general sobre la arquitectura usada para este artículo.](./media/saas-tenancy-tenant-analytics/architectureOverview.png)

## <a name="offline-tenant-analytics-pattern"></a>Patrón de análisis de inquilinos sin conexión

Las aplicaciones SaaS multiinquilino tienen normalmente una gran cantidad de datos almacenados en la nube. Estos datos proporcionan una valiosa fuente de información sobre la operación y el uso de la aplicación y el comportamiento de los inquilinos. Esta información puede guiarle a la hora de desarrollar características, mejorar la facilidad de uso y realizar otras inversiones en la aplicación y la plataforma.

El acceso a los datos para todos los inquilinos es sencillo si todos los datos se encuentran en una única base de datos multiinquilino. Sin embargo, el acceso es más complejo si se distribuyen a escala entre miles de bases de datos potenciales. Una manera de dominar la complejidad y minimizar el impacto de las consultas de análisis sobre datos transaccionales es extraer los datos en un almacén de datos o una base de datos de análisis diseñados específicamente para ello.

En este tutorial se presenta un escenario de análisis completo para la aplicación SaaS Wingtip Tickets. En primer lugar, se usan *Trabajos elásticos* para extraer datos de cada base de datos de inquilino y cargarlos en tabla de almacenamiento provisional en un almacén de análisis. El almacén de análisis podría ser un grupo de SQL dedicado o una instancia de SQL Database. Para la extracción de datos a gran escala, se recomienda [Azure Data Factory](../../data-factory/introduction.md).

A continuación, los datos agregados se transforman en un conjunto de tablas en un [esquema de estrella](https://www.wikipedia.org/wiki/Star_schema). Las tablas constan de una tabla de hechos central más tablas de dimensiones relacionadas.  Para Wingtip Tickets:

- La tabla de hechos central del esquema de estrella contiene datos de las entradas.
- Las tablas de dimensiones describen lugares, eventos, clientes y fechas de compra.

De forma conjunta, las tablas de hechos centrales y de dimensiones permiten realizar un procesamiento analítico eficaz. El esquema de estrella usado en este tutorial se muestra en la siguiente imagen:
 
![architectureOverView](./media/saas-tenancy-tenant-analytics/StarSchema.png)

Por último, se consulta el almacén de análisis con **Power BI** para resaltar la información sobre el comportamiento de inquilinos y el uso de la aplicación Wingtip Tickets. Se ejecutan consultas que:
 
- Muestran la popularidad relativa de cada lugar.
- Resaltan los patrones de ventas de entradas para distintos eventos.
- Muestran el éxito relativo de los distintos lugares en las ventas de sus eventos.

La comprensión del uso que hace cada inquilino del servicio es relevante para explorar opciones que sirvan para monetizar el servicio y mejorar el servicio a fin de ayudar a los inquilinos a tener más éxito. En este tutorial se proporcionan ejemplos básicos de los tipos de información que pueden deducirse de los datos de inquilino.

## <a name="setup"></a>Configurar

### <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

- La aplicación SaaS de Wingtip Tickets con una base de datos por inquilino debe estar implementada. Para implementarla en menos de cinco minutos, consulte el artículo sobre la [implementación y exploración de la aplicación SaaS de Wingtip](./saas-dbpertenant-get-started-deploy.md).
- Los scripts de la aplicación SaaS de Wingtip Tickets con una base de datos por inquilino y el [código fuente](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant/) de la aplicación se descargan de GitHub. Consulte las instrucciones de descarga. No olvide *desbloquear el archivo ZIP* antes de extraer su contenido. Consulte las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md) para saber cuáles son los pasos para descargar y desbloquear los scripts SaaS de Wingtip Tickets.
- Power BI Desktop debe estar instalado. [Descargue Power BI Desktop](https://powerbi.microsoft.com/downloads/).
- Se debe haber aprovisionado el loto de inquilinos adicionales; vea el [**tutorial de aprovisionamiento de inquilinos**](./saas-dbpertenant-provision-and-catalog.md).
- Se deben haber creado una cuenta de trabajo y una base de datos para la cuenta de trabajo. Consulte los pasos apropiados en el [**tutorial de administración de esquemas**](./saas-tenancy-schema-management.md#create-a-job-agent-database-and-new-job-agent).

### <a name="create-data-for-the-demo"></a>Creación de datos para la demostración

En este tutorial, se realiza un análisis de los datos de ventas de entradas. En el paso actual, genera datos sobre las entradas para todos los inquilinos.  Más adelante, estos datos se extraen para los análisis. *Asegúrese de haber aprovisionado el lote de los inquilinos como se describió anteriormente, para que tenga una cantidad significativa de datos*. Una cantidad de datos lo suficientemente grande puede exponer un intervalo de patrones de compra de entradas distintos.

1. En PowerShell ISE, abra *…\Learning Modules\Operational Analytics\Tenant Analytics\Demo-TenantAnalytics.ps1* y defina el siguiente valor:
    - **$DemoScenario** = **1** Purchase tickets for events at all venues
2. Presione **F5** para ejecutar el script y crear un historial de compra de entradas para cada evento en cada lugar.  El script se ejecuta durante varios minutos para generar decenas de miles de entradas.

### <a name="deploy-the-analytics-store"></a>Implementación del almacén de análisis
A menudo, hay numerosas bases de datos transaccionales que juntas contienen todos los datos de inquilino. Debe agregar los datos de inquilino de muchas bases de datos transaccionales en un solo almacén de análisis. La agregación permite realizar una consulta eficaz de los datos. En este tutorial, se usa una instancia de Azure SQL Database para almacenar los datos agregados.

En los pasos siguientes, se implementa el almacén de análisis, que se denomina **tenantanalytics**. También va a implementar tablas predefinidas que se rellenan más adelante en el tutorial:
1. En PowerShell ISE, abra *…\Learning Modules\Operational Analytics\Tenant Analytics\Demo-TenantAnalytics.ps1*. 
2. Establezca la variable $DemoScenario en el script para que coincida con el almacén de análisis elegido:
    - Para usar SQL Database sin almacenamiento de columnas, establezca **$DemoScenario** = **2**
    - Para usar SQL Database con almacenamiento de columnas, establezca **$DemoScenario** = **3**  
3. Presione **F5** para ejecutar el script de demostración (que llama al script *Deploy-TenantAnalytics\<XX>.ps1*) que crea el almacén de análisis de inquilino. 

Ahora que ya ha implementado la aplicación y la ha completado con datos de inquilino interesantes, use [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) para conectar los servidores **tenants1-dpt-&lt;Usuario&gt;** y **catalog-dpt-&lt;Usuario&gt;** con las credenciales Login = *developer* y Password = *P\@ssword1*. Consulte el [tutorial de introducción](./saas-dbpertenant-wingtip-app-overview.md) para obtener más orientación.

![Captura de pantalla que muestra la información necesaria para conectarse a SQL Server.](./media/saas-tenancy-tenant-analytics/ssmsSignIn.png)

En el Explorador de objetos, siga estos pasos:

1. Expanda el servidor *tenants1-dpt-&lt;Usuario&gt;* .
2. Expanda el nodo Bases de datos y vea la lista de bases de datos de inquilino.
3. Expanda el servidor *catalog-dpt-&lt;Usuario&gt;* .
4. Verifique que puede ver el almacén de análisis y la base de datos jobaccount.

Expanda el almacén de análisis para ver los siguientes elementos de la base de datos en el Explorador de objetos de SSMS:

- Las tablas **TicketsRawData** y **EventsRawData** contienen datos sin procesar extraídos de las bases de datos de inquilino.
- Las tablas del esquema de estrella son **fact_Tickets**, **dim_Customers**, **dim_Venues**, **dim_Events** y **dim_Dates**.
- El procedimiento almacenado se usa para rellenar las tablas del esquema de estrella a partir de las tablas de datos sin procesar.

![Captura de pantalla de los elementos de base de datos que se muestran en el Explorador de objetos de SSMS.](./media/saas-tenancy-tenant-analytics/tenantAnalytics.png)

## <a name="data-extraction"></a>Extracción de datos 

### <a name="create-target-groups"></a>Creación de grupos de destino 

Antes de continuar, asegúrese de haber implementado la cuenta de trabajo y la base de datos jobaccount. En la siguiente serie de pasos, se usan trabajos elásticos para extraer datos de cada base de datos de inquilino y para almacenar los datos en el almacén de análisis. Después, el segundo trabajo desglosa los datos y los almacena en tablas en el esquema de estrella. Estos dos trabajos se ejecutan en dos grupos de destino distintos, denominados **TenantGroup** y **AnalyticsGroup**. El trabajo de extracción se ejecuta en el grupo TenantGroup, que contiene todas las bases de datos de inquilino. El trabajo de desglose se ejecuta en el grupo AnalyticsGroup, que contiene solo el almacén de análisis. Cree los grupos de destino mediante los pasos siguientes:

1. En SSMS, conéctese a la base de datos **jobaccount** en catalog-dpt-&lt;Usuario&gt;.
2. En SSMS, abra *…\Learning Modules\Operational Analytics\Tenant Analytics\ TargetGroups.sql* 
3. Modifique la variable @User en la parte superior del script y reemplace `<User>` por el valor de usuario utilizado cuando implementó la aplicación SaaS de Wingtip.
4. Presione **F5** para ejecutar el script que crea los dos grupos de destino.

### <a name="extract-raw-data-from-all-tenants"></a>Extracción de datos sin procesar de todos los inquilinos

Las modificaciones de datos extensivas pueden ocurrir con más frecuencia para datos de *entradas y clientes* que para datos de *eventos y lugares*. Por tanto, considere la posibilidad de extraer los datos sobre las entradas y los clientes por separado y con más frecuencia de la que extrae los datos sobre eventos y lugares. En esta sección, se van a definir y programar dos trabajos independientes:

- Extraer los datos de las entradas y los clientes.
- Extraer los datos de los eventos y los lugares.

Cada trabajo extrae sus datos y los introduce en el almacén de análisis. Ahí, un trabajo independiente desglosa los datos extraídos en el esquema de estrella de análisis.

1. En SSMS, conéctese a la base de datos **jobaccount** en el servidor catalog-dpt-&lt;Usuario&gt;.
2. En SSMS, abra *...\Learning Modules\Operational Analytics\Tenant Analytics\ExtractTickets.sql*.
3. Modifique @User en la parte superior del script y reemplace `<User>` por el nombre de usuario utilizado cuando implementó la aplicación SaaS de Wingtip. 
4. Presione F5 para ejecutar el script que crea y ejecuta el trabajo que extrae los datos de las entradas y los clientes de cada base de datos de inquilino. El trabajo guarda los datos en el almacén de análisis.
5. Consulte la tabla TicketsRawData en la base de datos tenantanalytics, para asegurarse de que la tabla se rellena con la información de las entradas de todos los inquilinos.

![En la captura de pantalla se muestra la base de datos de ExtractTickets con el elemento dbo.TicketsRawData seleccionado en el Explorador de objetos.](./media/saas-tenancy-tenant-analytics/ticketExtracts.png)

Repita los pasos anteriores, excepto esta sustitución de tiempo de **\ExtractTickets.sql** por **\ExtractVenuesEvents.sql** en el paso 2.

Si el trabajo se ejecuta correctamente, se rellena la tabla EventsRawData del almacén de análisis con información nueva de los eventos y los lugares de todos los inquilinos. 

## <a name="data-reorganization"></a>Reorganización de datos

### <a name="shred-extracted-data-to-populate-star-schema-tables"></a>Desglose de los datos extraídos para rellenar las tablas de un esquema de estrella

El paso siguiente consiste en desglosar los datos sin procesar extraídos en un conjunto de tablas que se optimizan para las consultas de análisis. Se utiliza un esquema de estrella. Una tabla de hechos central contiene registros de ventas de entradas individuales. Otras tablas se rellenan con datos relacionados sobre lugares, eventos y clientes. Además, hay tablas de dimensiones de tiempo. 

En esta sección del tutorial, va a definir y ejecutar un trabajo que combina los datos sin procesar extraídos con los datos de las tablas de un esquema de estrella. Cuando finaliza el trabajo de combinación, se eliminan los datos sin procesar, dejando las tablas listas para que se rellenen con el próximo trabajo de extracción de datos de inquilino.

1. En SSMS, conéctese a la base de datos **jobaccount** en catalog-dpt-&lt;Usuario&gt;.
2. En SSMS, abra *…\Learning Modules\Operational Analytics\Tenant Analytics\ShredRawExtractedData.sql*.
3. Presione **F5** para ejecutar el script con el que se define un trabajo que llama al procedimiento almacenado sp_ShredRawExtractedData en el almacén de análisis.
4. Deje tiempo suficiente para que el trabajo se ejecute correctamente.
    - Consulte la columna **Lifecycle** de la tabla jobs.jobs_execution para comprobar el estado del trabajo. Asegúrese de que el trabajo se ha completado **correctamente** antes de continuar. Una ejecución satisfactoria muestra datos similares a los del siguiente gráfico:

![desglose](./media/saas-tenancy-tenant-analytics/shreddingJob.PNG)

## <a name="data-exploration"></a>Exploración de datos

### <a name="visualize-tenant-data"></a>Visualización de datos de inquilino

Los datos de las tablas de un esquema de estrella proporcionan todos los datos de ventas de entradas necesarios para el análisis. Para facilitar la visualización de tendencias en grandes conjuntos de datos, necesita una representación gráfica.  En esta sección, va a aprender a usar **Power BI** para manipular y visualizar los datos de inquilino que extrajo y organizó.

Siga estos pasos para conectarse a Power BI e importar las vistas creadas anteriormente:

1. Lance Power BI Desktop.
2. En la cinta de opciones de Inicio, seleccione **Obtener datos** y, después, seleccione **Más…** en el menú.
3. En la ventana **Obtener datos**, seleccione Azure SQL Database.
4. En la ventana de inicio de sesión de la base de datos, escriba el nombre del servidor (catalog-dpt-&lt;Usuario&gt;.database.windows.net). Seleccione **Importar** en **Modo Conectividad de datos** y después haga clic en Aceptar. 

    ![signinpowerbi](./media/saas-tenancy-tenant-analytics/powerBISignIn.PNG)

5. Seleccione **Base de datos** en el panel izquierdo y escriba los valores de user name = *developer* y password = *P\@ssword1*. Haga clic en **Conectar**.  

    ![En la captura de pantalla se muestra el cuadro de diálogo de la base de datos de SQL Server, desde donde puede especificar un nombre de usuario y una contraseña.](./media/saas-tenancy-tenant-analytics/databaseSignIn.PNG)

6. En el panel **Navegador**, debajo de la base de datos de análisis, seleccione las tablas del esquema de estrella: fact_Tickets, dim_Events, dim_Venues, dim_Customers y dim_Dates. Después seleccione **Cargar**. 

Felicidades. Ha cargado los datos correctamente en Power BI. Ahora puede empezar a explorar visualizaciones interesantes para ayudar a obtener información sobre los inquilinos. Después, se ofrece orientación sobre cómo los análisis pueden permitirle proporcionar recomendaciones basadas en datos al equipo empresarial de Wingtip Tickets. Las recomendaciones pueden ayudarle a optimizar el modelo de negocio y la experiencia del cliente.

Primero debe analizar los datos de ventas de entradas para ver la variación del uso entre los distintos lugares. Seleccione las opciones siguientes en Power BI para trazar un gráfico de barras del número total de entradas vendidas en cada lugar. Debido a la variación aleatoria del generador de entradas, los resultados pueden ser diferentes.
 
![En la captura de pantalla se muestra una visualización de Power BI y controles para la visualización de datos a la derecha.](./media/saas-tenancy-tenant-analytics/TotalTicketsByVenues.PNG)

El gráfico anterior confirma que el número de entradas vendidas en cada lugar varía. Los lugares que venden más entradas usan su servicio con mayor frecuencia que aquellos que venden menos. En este caso, puede haber una oportunidad de adaptar la asignación de recursos a las diferentes necesidades de los inquilinos.

Puede analizar aún más los datos para ver cómo varían las ventas de entradas con el paso del tiempo. Seleccione las opciones siguientes en Power BI para trazar un gráfico de barras del número total de entradas vendidas cada día durante un período de sesenta días.
 
![En la captura de pantalla se muestra una visualización de Power BI titulada Ticket Sale Distribution (Distribución de venta de entradas) comparada con Sale Day (Día de ventas).](./media/saas-tenancy-tenant-analytics/SaleVersusDate.PNG)

En el gráfico anterior se muestra que las ventas de entradas presentan picos en algunos lugares. Estos picos refuerzan la idea de que algunos lugares pueden estar consumiendo recursos del sistema de manera desproporcionada. Hasta ahora no hay ningún patrón obvio de cuándo se producen los picos.

Después, puede que quiera indagar más en la importancia de los días en que se registran los picos de ventas. ¿Cuándo se producen estos picos después de que las entradas salen a la venta? Para obtener un gráfico de las entradas vendidas al día, seleccione las siguientes opciones en Power BI.

![SaleDayDistribution](./media/saas-tenancy-tenant-analytics/SaleDistributionPerDay.PNG)

En el gráfico anterior se refleja que algunos lugares venden muchas entradas el primer día de venta. En cuanto las entradas salen a la venta en estos lugares, parece haber una desenfrenada demanda. Tan intensa actividad en algunos lugares puede repercutir en el servicio de otros inquilinos.

Puede profundizar en los datos de nuevo para ver si tal desenfrenada demanda se produce para todos los eventos celebrados en estos lugares. En los gráficos anteriores, observó que Contoso Concert Hall vende muchas entradas y que Contoso también registra un pico de ventas de entradas en determinados días. Practique con las opciones de Power BI para trazar un gráfico de las ventas de entradas acumulativas de Contoso Concert Hall, centrándose en las tendencias de ventas de cada uno de sus eventos. ¿Todos los eventos siguen el mismo patrón de venta?

![ContosoSales](./media/saas-tenancy-tenant-analytics/EventSaleTrends.PNG)

En el gráfico anterior de Contoso Concert Hall, se refleja que la desenfrenada demanda no se registra para todos los eventos. Practique con las opciones de filtro para ver las tendencias de venta de otros lugares.

La información de los patrones de venta de entradas pueden permitir a Wingtip Tickets optimizar su modelo de negocio. En lugar de aplicar los mismos cargos a todos los inquilinos, quizá Wingtip debería introducir niveles de servicio con distintos tamaños de proceso. A los lugares más grandes que necesitan vender más entradas al día se les podría ofrecer un nivel superior con un contrato de nivel de servicio (SLA) de categoría superior. Estos lugares podrían tener sus bases de datos agrupadas con límites de recursos por base de datos más altos. Cada nivel de servicio podría tener una asignación de ventas por hora, con tarifas adicionales por exceder la asignación. Los lugares más grandes que tienen intensas actividades de ventas se beneficiarían de los niveles más altos, y Wingtip Tickets puede monetizar su servicio con mayor eficacia.

Mientras tanto, algunos clientes de Wingtip Tickets se quejan de que tienen dificultades para vender las suficientes entradas como para cubrir el costo del servicio. Quizá en esta información se ofrece la oportunidad de impulsar las ventas de entradas para los lugares que presentan déficit de rendimiento. Un aumento de las ventas aumentaría el valor percibido del servicio. Haga clic con el botón derecho en fact_Tickets y seleccione **Nueva medida**. Escriba la siguiente expresión para la nueva medida denominada **AverageTicketsSold**:

```
AverageTicketsSold = AVERAGEX( SUMMARIZE( TableName, TableName[Venue Name] ), CALCULATE( SUM(TableName[Tickets Sold] ) ) )
```

Seleccione las siguientes opciones de visualización para obtener un gráfico del porcentaje de entradas vendidas en cada lugar para determinar su éxito relativo.

![La captura de pantalla muestra una visualización de Power BI titulada Average Tickets Sold By Each Venue.](./media/saas-tenancy-tenant-analytics/AvgTicketsByVenues.PNG)

En el gráfico anterior se refleja que, aunque la mayoría de los lugares venden más del 80 % de las entradas, algunos se esfuerzan por llegar a más de la mitad del aforo. Practique con los valores Well para seleccionar los porcentajes máximo y mínimo de las entradas vendidas en cada lugar.

Anteriormente, profundizó en sus análisis para detectar que las ventas de entradas tienden a seguir patrones predecibles. Esta conclusión puede permitir a Wingtip Tickets ayudar a los lugares que presentan un déficit de rendimiento a impulsar las ventas de entradas mediante la recomendación de precios dinámicos. Este hecho podría brindar una oportunidad para emplear técnicas de aprendizaje automático para predecir ventas de entradas de cada evento. También se pueden realizar predicciones de la repercusión que la oferta de descuentos en las ventas de entradas puede tener en los ingresos. Power BI Embedded puede integrarse en una aplicación de administración de eventos. La integración de podría ayudar a visualizar las ventas previstas y el efecto de diferentes descuentos. La aplicación podría ayudar a diseñar un descuento óptimo para aplicarlo directamente desde la pantalla de análisis.

Ha observado las tendencias de los datos de inquilino de la aplicación de WingTip. Puede contemplar otras formas en que la aplicación puede informar de las decisiones empresariales a los proveedores de aplicaciones SaaS. De esta forma, los proveedores pueden satisfacer mejor las necesidades de sus inquilinos. Se espera que este tutorial le haya proporcionado todas las herramientas necesarias para realizar análisis de datos de inquilino para permitir que las empresas tomen decisiones basadas en datos.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> - Implementar una base de datos de análisis de inquilino con tablas de un esquema de estrella predefinidas
> - Utilizar trabajos elásticos para extraer datos de todas las bases de datos de inquilinos
> - Combinar los datos extraídos en tablas en un esquema de estrella diseñado para el análisis
> - Consultar una base de datos de análisis 
> - Usar Power BI para la visualización de datos para observar tendencias en los datos de inquilino 

Felicidades.

## <a name="additional-resources"></a>Recursos adicionales

- Otros [tutoriales basados en la aplicación SaaS de Wingtip](./saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials).
- [Trabajos elásticos](./elastic-jobs-overview.md).
- [Análisis entre inquilinos mediante datos extraídos: aplicación multiinquilino](./saas-multitenantdb-tenant-analytics.md)
