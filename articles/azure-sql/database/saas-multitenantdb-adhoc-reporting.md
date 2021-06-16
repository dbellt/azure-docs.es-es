---
title: Ejecución de consultas ad hoc en varias bases de datos
description: Ejecute consultas de notificaciones ad hoc entre varias bases de datos de Azure SQL en un ejemplo de aplicación multiinquilino.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 10/30/2018
ms.openlocfilehash: 1853c37241f9748f8870b27fe76df51d3c0f4b0f
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699158"
---
# <a name="run-ad-hoc-analytics-queries-across-multiple-databases-azure-sql-database"></a>Ejecución de consultas de análisis ad hoc entre varias bases de datos (Azure SQL Database)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial, se ejecutan consultas distribuidas en todo el conjunto de bases de datos de inquilino para habilitar las notificaciones ad hoc interactivas. Estas consultas pueden extraer información incluida en los datos operativos diarios de la aplicación SaaS Wingtip Tickets. Para que se realicen estas extracciones, puede implementar una base de datos de análisis adicional en el servidor de catálogo y usar una consulta elástica para habilitar las consultas distribuidas.


En este tutorial, obtendrá información:

> [!div class="checklist"]
> 
> * Implementación de una base de datos de notificaciones ad hoc
> * Acerca de cómo ejecutar consultas distribuidas en todas las bases de datos de inquilinos


Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

* La aplicación SaaS de base de datos multiinquilino Wingtip Tickets está implementada. Para implementarla en menos de cinco minutos, consulte el artículo sobre la [implementación y exploración de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets](saas-multitenantdb-get-started-deploy.md)
* Azure PowerShell está instalado. Para más información, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps)
* SQL Server Management Studio (SSMS) está instalado. Para descargar e instalar SSMS, consulte [Descarga de SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).


## <a name="ad-hoc-reporting-pattern"></a>Patrón de notificación ad hoc

![patrón de notificación ad hoc](./media/saas-multitenantdb-adhoc-reporting/adhocreportingpattern_shardedmultitenantDB.png)

Las aplicaciones SaaS pueden analizar la ingente cantidad de datos del inquilino que se almacenan de forma centralizada en la nube. Los análisis muestran información sobre el funcionamiento y el uso de la aplicación. Esta información puede guiarle en el desarrollo de características, mejoras de facilidad de uso y otras inversiones en sus aplicaciones y servicios.

Es fácil obtener acceso a estos datos en una sola base de datos multiinquilino, pero no es tan fácil cuando se distribuyen a escala en miles de bases de datos. Un enfoque es usar la [consulta elástica](elastic-query-overview.md), que permite consultas en un conjunto distribuido de bases de datos con un esquema común. Estas bases de datos se pueden distribuir en distintos grupos de recursos y suscripciones. No obstante, un mismo inicio de sesión debe tener acceso para extraer datos de todas las bases de datos. La consulta elástica usa una única base de datos *principal* en la que se definen tablas externas que reflejan tablas o vistas en las bases de datos (de inquilino) distribuidas. Las consultas que se envían a esta base de datos principal se compilan para producir un plan de consulta distribuido, con partes de la consulta aplicadas en las bases de datos de inquilino según sea necesario. La consulta elástica usa el mapa de particiones de la base de datos del catálogo para determinar la ubicación de todas las bases de datos de inquilino. La configuración y la consulta son sencillas y utilizan [Transact-SQL](/sql/t-sql/language-reference) estándar; además, admiten consultas ad hoc desde herramientas como Power BI y Excel.

Mediante la distribución de las consultas en las bases de datos de inquilino, la consulta elástica proporciona una visión inmediata de los datos de producción activos. Sin embargo, como la consulta elástica puede extraer los datos de muchas bases de datos, la latencia de las consultas a veces puede ser mayor que las consultas equivalentes enviadas a una sola base de datos multiinquilino. Asegúrese de diseñar las consultas para minimizar los datos que se devuelven. La consulta elástica a menudo resulta más apropiada para consultar pequeñas cantidades de datos en tiempo real, en lugar de generar informes o consultas de análisis complejos o usados con frecuencia. Si las consultas no funcionan correctamente, examine el [plan de ejecución](/sql/relational-databases/performance/display-an-actual-execution-plan) para ver qué parte de la consulta se ha transferido a la base de datos remota. Y evalúe cuántos datos se devuelven. Las consultas que requieren un procesamiento analítico complejo se pueden atender mejor si se guardan los datos de inquilino extraídos en una base de datos optimizada para las consultas de análisis. SQL Database y Azure Synapse Analytics podrían hospedar este tipo de base de datos de análisis.

Este patrón de análisis se explica en el [tutorial sobre el análisis de inquilinos](saas-multitenantdb-tenant-analytics.md).

## <a name="get-the-wingtip-tickets-saas-multi-tenant-database-application-source-code-and-scripts"></a>Obtención del código fuente y los scripts de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets

Los scripts y el código fuente de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets están disponibles en el repositorio de GitHub [WingtipTicketsSaaS-MultitenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB). Consulte las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md) para saber cuáles son los pasos para descargar y desbloquear los scripts SaaS de Wingtip Tickets.

## <a name="create-ticket-sales-data"></a>Creación de datos de ventas de entradas

Para ejecutar consultas en un conjunto de datos más interesante, ejecute el generador de entradas para crear datos de ventas de entradas.

1. En *PowerShell ISE*, abra el script ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\*Demo-AdhocReporting.ps1* y establezca los valores siguientes:
   * **$DemoScenario** = 1, **Purchase tickets for events at all venues** (comprar entradas para eventos de todas las ubicaciones).
2. Presione **F5** para ejecutar el script y generar datos de ventas de entradas. Mientras se ejecuta el script, siga con los pasos de este tutorial. Los datos de entradas se consultan en la sección *Ejecución de consultas distribuidas ad hoc*, de modo que espere a que el generador de entradas finalice.

## <a name="explore-the-tenant-tables"></a>Exploración de las tablas de inquilino 

En la aplicación SaaS de base de datos multiinquilino Wingtip Tickets, los inquilinos se almacenan en un modelo de administración de inquilinos híbrido, en el que los datos del inquilino se almacenan en una base de datos multiinquilino o en una base de datos de un único inquilino y se pueden mover entre las dos opciones. Al consultar en todas las bases de datos de inquilino, es importante que la consulta elástica pueda tratar los datos como si formaran parte de una sola base de datos lógica con particiones por inquilino. 

Para lograr este patrón, todas las tablas de inquilinos incluyen una columna *VenueId* que identifica a qué inquilino pertenecen los datos. El identificador *VenueId* se ha calculado como un valor hash del nombre de Venue, pero se puede usar cualquier estrategia para introducir un valor único para esta columna. Esta estrategia es similar a cómo se calcula la clave de inquilino para usarse en el catálogo. La consulta elástica usa las tablas que contienen *VenueId* para usar consultas en paralelo e insertarlas en la base de datos remota de inquilinos. Esto reduce considerablemente la cantidad de datos que se devuelven y da como resultado un aumento del rendimiento, especialmente cuando hay varios inquilinos cuyos datos se almacenan en bases de datos de un solo inquilino.

## <a name="deploy-the-database-used-for-ad-hoc-distributed-queries"></a>Implementación de la base de datos usada para las consultas distribuidas ad hoc

En este ejercicio se implementa la base de datos *adhocreporting*. Esta es la base de datos principal que contendrá el esquema utilizado para realizar consultas en todas las bases de datos de inquilino. La base de datos se implementa en el servidor de catálogo existente, que es el utilizado para todas las bases de datos relacionadas con la administración en la aplicación de ejemplo.

1. Abra ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\*Demo-AdhocReporting.ps1* en *PowerShell ISE* y establezca los valores siguientes:
   * **$DemoScenario** = 2, **Implementación de una base de datos de análisis ad hoc**.

2. Presione **F5** para ejecutar el script y crear la base de datos *adhocreporting*.

En la siguiente sección, se agrega el esquema a la base de datos de modo que se pueda usar para ejecutar consultas distribuidas.

## <a name="configure-the-head-database-for-running-distributed-queries"></a>Configuración de la base de datos "principal" para ejecutar consultas distribuidas

En este ejercicio se agrega el esquema (el origen de datos externo y las definiciones de tabla externas) a la base de datos de notificaciones ad hoc que permite realizar consultas en todas las bases de datos de inquilino.

1. Abra SQL Server Management Studio y conéctese a la base de datos de notificaciones ad hoc que creó en el paso anterior. El nombre de la base de datos será *adhocreporting*.
2. Abra ...\Learning Modules\Operational Analytics\Adhoc Reporting\ *Initialize-AdhocReportingDB.sql* en SSMS.
3. Revise el script SQL y tenga en cuenta lo siguiente:

   La consulta elástica usa una credencial de ámbito de base de datos para acceder a cada una de las bases de datos de inquilino. Esta credencial debe estar disponible en todas las bases de datos y normalmente se le deberían conceder los derechos mínimos necesarios para permitir estas consultas ad hoc.

    ![crear credencial](./media/saas-multitenantdb-adhoc-reporting/create-credential.png)

   Al utilizar la base de datos del catálogo como origen de datos externo, las consultas se distribuyen a todas las bases de datos registradas en el catálogo cuando se ejecuta la consulta. Dado que los nombres de servidor son diferentes para cada implementación, este script de inicialización obtiene la ubicación de la base de datos de catálogo mediante la recuperación del servidor actual (@@servername) donde se ejecuta el script.

    ![crear origen de datos externos](./media/saas-multitenantdb-adhoc-reporting/create-external-data-source.png)

   Las tablas externas que hacen referencia a tablas de inquilino se definen con **DISTRIBUTION = SHARDED(VenueId)** . Esto enruta una consulta para un determinado *VenueId* a la base de datos adecuada y mejora el rendimiento en muchos escenarios como se muestra en la sección siguiente.

    ![crear tablas externas](./media/saas-multitenantdb-adhoc-reporting/external-tables.png)

   La tabla local *VenueTypes* que se crea y rellena. Esta tabla de datos de referencia es común en todas las bases de datos de inquilino, por lo que se puede representar como una tabla local y rellenarse con los datos comunes. En algunas consultas, así se puede reducir la cantidad de datos movidos entre las bases de datos de inquilino y la base de datos *adhocreporting*.

    ![crear tabla](./media/saas-multitenantdb-adhoc-reporting/create-table.png)

   Si incluye tablas de referencia de esta manera, asegúrese de actualizar el esquema de tabla y los datos cada vez que actualice las bases de datos de inquilino.

4. Presione **F5** para ejecutar el script e inicializar la base de datos *adhocreporting*. 

Ahora puede ejecutar consultas distribuidas y recopilar información de todos los inquilinos.

## <a name="run-ad-hoc-distributed-queries"></a>Ejecución de consultas distribuidas ad hoc

Ahora que la base de datos *adhocreporting* está configurada, continúe y ejecute algunas consultas distribuidas. Incluya el plan de ejecución para comprender mejor dónde sucede el procesamiento de las consultas. 

Al inspeccionar el plan de ejecución, mantenga el mouse sobre los iconos de plan para obtener más información. 

1. En *SSMS*, abra ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\*Demo-AdhocReportingQueries.sql*.
2. Asegúrese de que está conectado a la base de datos **adhocreporting**.
3. Seleccione el menú **Consulta** y haga clic en **Incluir plan de ejecución real**.
4. Resalte la consulta *Which venues are currently registered?* (¿Qué ubicaciones están registradas actualmente?) y presione **F5**.

   La consulta devuelve la lista de ubicaciones completa e ilustra lo rápido y fácil que es realizar consultas en todos los inquilinos y devolver los datos de cada uno.

   Inspeccione el plan y compruebe que el costo íntegro es la consulta remota porque simplemente vamos a cada base de datos de inquilino y seleccionamos la información de las ubicaciones.

   ![SELECT * FROM dbo.Venues](./media/saas-multitenantdb-adhoc-reporting/query1-plan.png)

5. Seleccione la siguiente consulta y presione **F5**.

   Esta consulta combina datos de las bases de datos de inquilino y de la tabla local *VenueTypes* (local porque es una tabla de la base de datos *adhocreporting*).

   Inspeccione el plan y compruebe que la mayor parte del costo es la consulta remota, ya que se consulta la información de ubicación de cada inquilino (dbo.Venues) y, a continuación, se realiza una combinación rápida local con la tabla local *VenueTypes* para mostrar el nombre descriptivo.

   ![Combinación de datos remotos y locales](./media/saas-multitenantdb-adhoc-reporting/query2-plan.png)

6. Ahora seleccione la consulta *On which day were the most tickets sold?* (¿Qué día se vendieron más entradas?) y presione **F5**.

   Esta consulta hace una combinación y una agregación un poco más complejas. Es importante tener en cuenta que la mayor parte del procesamiento se realiza de forma remota y, una vez más, se devuelven solo las filas que se necesitan, una única fila para el recuento total diario de ventas de entradas de cada ubicación.

   ![Query](./media/saas-multitenantdb-adhoc-reporting/query3-plan.png)


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo:

> [!div class="checklist"]
> 
> * Ejecutar consultas distribuidas en todas las bases de datos de inquilinos
> * Implementar una base de datos de notificaciones ad hoc y agregarle el esquema para ejecutar consultas distribuidas.

Ahora pruebe el [tutorial de análisis de inquilinos](saas-multitenantdb-tenant-analytics.md) para explorar la extracción de datos en una base de datos de análisis independiente para llevar a cabo un procesamiento de análisis más complejo.

## <a name="additional-resources"></a>Recursos adicionales

<!-- ??
* Additional [tutorials that build upon the Wingtip Tickets SaaS Multi-tenant Database application](saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
-->

* [Consulta elástica](elastic-query-overview.md)